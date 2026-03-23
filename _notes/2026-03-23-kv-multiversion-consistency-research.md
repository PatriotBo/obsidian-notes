---
title: "业界存储系统多版本能力与多副本一致性调研报告"
source: "http://localhost:8976/kv-multiversion-research.html"
author: "内部调研"
date: 2026-03-23
tags: [分布式存储, MVCC, 多版本, 一致性, Redis, HBase, Cassandra, TiKV, FoundationDB, Raft, KV存储]
category: "01-AI与技术"
---

# 业界存储系统多版本能力与多副本一致性调研报告

> 来源：内部调研文档 | 阅读日期：2026-03-23

## 📌 核心要点

- 对比 **Redis、HBase、Cassandra、TiKV、FoundationDB** 五大存储系统的多版本能力和多副本一致性机制
- HBase 和 TiKV 是原生多版本的典范，Redis 仅通过 COW 实现进程级快照，Cassandra 用 LWW 解决冲突而非保留历史
- 一致性从弱到强：Redis（异步复制）< Cassandra（可调一致性）< HBase（单 Region 强一致）< TiKV（Raft 分布式事务）≈ FoundationDB（严格串行化）
- 自研 KV 系统建议：**借鉴 HBase + TiKV 的混合方案**，Key 编码后缀版本号（逆序），使用 HLC 作为版本号，渐进式引入一致性保障

## 📝 详细笔记

### 一、五大系统多版本能力全景对比

| 系统 | 多版本支持 | 版本粒度 | 版本号类型 | 版本数控制 | 主要用途 |
|------|-----------|---------|-----------|-----------|---------|
| **Redis** | ❌ 不支持 | 进程级（COW 快照） | 无 | 无 | 纯缓存，RDB/AOF 持久化 |
| **HBase** | ✅ 原生 | Cell 级 | Timestamp | MAX_VERSIONS + TTL | 多版本数据存储的典范 |
| **Cassandra** | ⚠️ 有限 | Cell 级 | Mutation Timestamp | Compaction 时仅保留最新 | LWW 冲突解决，非版本查询 |
| **TiKV** | ✅ 原生 | Key 级 | 逻辑时间戳 (TSO) | GC SafePoint 清理 | 分布式事务 MVCC |
| **FoundationDB** | ✅ 原生 | Key 级 | 版本号（逻辑时钟） | 5s 事务窗口 | 严格串行化隔离 |

### 二、各系统深度剖析

#### 2.1 Redis — 无原生多版本

**COW 快照机制**：
- `BGSAVE` / `BGREWRITEAOF` 触发 `fork()` 创建子进程
- 父子进程共享物理内存页（页表级共享）
- 写操作时 OS 执行 COW（Copy-On-Write）复制被修改的页面
- 子进程看到的是 fork 瞬间的数据快照

**局限性**：
- 不支持 Key 级多版本，无法查询某 Key 的历史值
- 快照粒度为全库，无法按 Key 或前缀做增量版本
- 写入密集时内存最高膨胀 2x
- 大实例 fork 可能造成毫秒级甚至秒级卡顿

**应用层模拟多版本的三种方案**：
1. **Key 编码版本号**：`SET user:1001:v3 "{...}"`
2. **Sorted Set 按时间戳排序**：`ZADD user:1001:versions 1711180800 "{...}"`
3. **Stream 作为版本日志**：`XADD user:1001:log * name Alice age 30`

> 💡 KeyDB（Redis 兼容分支）实现了真正的 MVCC

#### 2.2 HBase — 原生多版本的典范

**数据模型**：同一个 Cell（RowKey + CF:Qualifier）可保存多个版本，每个版本以 Timestamp 标识。

**物理存储格式（KeyValue）**：
```
┌─────────┬──────────────┬────────────┬───────────┬──────────┬───────┐
│ RowKey  │ ColumnFamily │ Qualifier  │ Timestamp │   Type   │ Value │
└─────────┴──────────────┴────────────┴───────────┴──────────┴───────┘
```

**关键设计**：
- Timestamp **降序排列**：最新版本排在最前，加速读取
- Type 字段标识操作类型（Put / Delete / DeleteColumn / DeleteFamily）
- Delete 不是真删除，而是写入删除标记

**版本管理参数**：
| 参数 | 说明 | 默认值 |
|------|------|--------|
| `MAX_VERSIONS` | 每个 Cell 最多保留的版本数 | 1 |
| `MIN_VERSIONS` | 配合 TTL，至少保留的版本数 | 0 |
| `TTL` | 数据存活时间（秒） | FOREVER |

**版本清理机制**：
- **Minor Compaction**：合并小 HFile → 大 HFile，❌ 不清理过期版本
- **Major Compaction**：合并所有 HFile → 单个 HFile，✅ 清理旧版本 + TTL 过期数据 + Delete 标记

**MVCC 并发控制**（Region 内读写一致性）：
1. 获取 Row Lock（行级锁）
2. 从 MVCC 获取 WriteNumber（单调递增）
3. 数据写入 MemStore（此时读不可见）
4. 数据写入 WAL 并 sync
5. 释放 Row Lock
6. `MVCC.completeAndWait()` → 推进 ReadPoint → 数据对读可见

> 关键保证：**WAL 先于 MemStore 写入（日志先行）**，未提交数据对读不可见

#### 2.3 Cassandra — 时间戳版本 + Last-Write-Wins

**核心特点**：每个 Cell 携带微秒级 mutation timestamp，但 **不保留历史版本供用户查询**。

**冲突解决 — LWW**：
```
Replica A: SET name = "Alice" @ ts=100
Replica B: SET name = "Bob" @ ts=200
读取合并 → 比较 timestamp: 200 > 100 → 返回 "Bob"
```

**删除机制 — 墓碑（Tombstone）**：
- 删除不是真删除，而是写入一个带时间戳的墓碑标记
- 墓碑在 `gc_grace_seconds`（默认 10 天）后由 Compaction 清理
- 过早清理墓碑可能导致"数据复活"（已删除的数据从未同步的副本回来）

**可调一致性级别**：

| 级别 | 含义 | 适用场景 |
|------|------|---------|
| `ONE` | 1 个副本确认 | 低延迟优先 |
| `QUORUM` | 多数派确认 | 平衡一致性和性能 |
| `ALL` | 所有副本确认 | 最强一致（牺牲可用性） |
| `LOCAL_QUORUM` | 本地 DC 多数派 | 跨 DC 部署 |

> W + R > N 时可保证强一致性（如 QUORUM 写 + QUORUM 读）

#### 2.4 TiKV — Raft + MVCC 分布式事务

**Multi-Raft 架构**：
- 数据分成多个 Region（约 96MB），每个 Region 是一个独立的 Raft 复制组
- Leader 分散在不同 TiKV 节点（PD 调度负载均衡）
- 每个 Raft Group 独立选举、独立复制

**MVCC 实现**：
- Key 编码：`{user_key}_{start_ts}` → Lock + Default（数据）
- 提交后：`{user_key}_{commit_ts}` → Write（指向数据）
- 版本号来自 PD 的 TSO（Timestamp Oracle），全局唯一递增

**Raft Leader 选举**：
- **PreVote 优化**：正式选举前先预投票（不增加 Term），防止网络分区恢复后的选举风暴
- 投票规则：Term ≥ 自己的 + 日志至少和自己一样新 + 本 Term 未投过票

**日志复制流程**：
```
Propose → Append（本地 RaftDB）→ Replicate（并行发送 AppendEntries）
→ 多数派 ACK → Commit → Apply（写入 RocksDB）→ 返回客户端
```

**关键概念**：
| 概念 | 含义 |
|------|------|
| Term | 逻辑时钟，每次选举递增 |
| Log Index | 日志条目的全局序号 |
| Commit Index | 已被多数派确认的最大 Log Index |
| Apply Index | 已 Apply 到状态机的最大 Log Index |

**三种一致性读方案**：

| 方案 | 机制 | 一致性 | 延迟 |
|------|------|--------|------|
| **Raft Log Read** | 读也走 Raft 流程 | 绝对一致 | 最慢 |
| **ReadIndex**（默认） | Leader 确认自己仍是 Leader（心跳多数派） | 强一致 | +1 RTT |
| **Lease Read** | Leader 维护租约，租约内无需确认 | 强一致 | 最低 |

#### 2.5 FoundationDB — 严格串行化

- 5 秒事务限制（强制短事务）
- OCC（乐观并发控制）+ MVCC
- 冲突检测在 Resolver 层完成
- 适合最强一致性核心系统（如 Apple CloudKit、Snowflake）

### 三、多副本一致性机制对比

| 系统 | 复制模型 | 一致性级别 | 故障恢复 |
|------|---------|-----------|---------|
| **Redis** | 异步复制（主从） | 最终一致 | WAIT 可选半同步，但不保证 |
| **HBase** | WAL → HDFS 3 副本 + ZK 协调 | Region 内强一致 | WAL Splitting + Region 重分配 |
| **Cassandra** | 无中心 P2P + 可调一致性 | 可调（ONE~ALL） | Anti-Entropy Repair + 墓碑 |
| **TiKV** | Multi-Raft（多数派写入） | 强一致 | Raft 选举 + 日志回放 |
| **FDB** | Paxos 变体 | 严格串行化 | 自动故障转移 |

### 四、HBase 一致性保障的三大支柱

**① 数据不丢**：WAL 存在 HDFS 上（3 副本），RS 崩溃后 WAL 仍然安全可读

**② 不脏读**：MVCC 控制可见性，未提交的数据对读不可见

**③ 不脑裂**：ZooKeeper Session + Fencing 保证同一 Region 同一时刻只有一个 Writer

**④ 恢复后一致**：WAL 回放重建 MemStore + MVCC 状态，数据完全一致

**一句话类比**：
- **HDFS** = 保险柜（数据的物理安全，3 把钥匙分别保管）
- **WAL + MVCC** = 记账本 + 出纳规则（先记账再动钱，只看已签字的账目）
- **ZooKeeper** = 物业管家（监控谁在值班，不在就换人 + 换锁）

**ZooKeeper 的三大职责**：
1. **Master 选举**：`/hbase/master` 临时节点
2. **RegionServer 存活检测**：Session 超时 → 节点消失 → Master 感知 RS 下线
3. **Meta Region 定位**：`/hbase/meta-region-server`

**Fencing 防脑裂流程**：
```
RS-A 发生 Full GC → 无法发送 ZK 心跳 → Session 超时 → ZK 删除临时节点
→ HMaster 执行 WAL Splitting + Region 重分配给 RS-B
→ RS-A 恢复后发现 Session 过期 → 自杀（abort）
→ 不会出现两个 RS 同时服务同一个 Region
```

### 五、自研 KV 系统设计建议

#### 5.1 多版本能力设计（借鉴 HBase + TiKV）

**① Key 编码方案**（参考 TiKV）：
```
存储 Key = {user_key} + {version_id (逆序)}
```
- 同一 user_key 的所有版本物理相邻
- version 逆序 → seek 直接命中最新版本

**② 版本号设计 — 推荐 HLC**：
```
version = {physical_ts (48bit)} + {logical (16bit)}
```
- 单调递增，不受时钟回拨影响
- 含物理时间信息，可排序

**③ 版本管理**（参考 HBase）：
- `max_versions`：每个 Key 最多保留 N 个版本（默认 1）
- `ttl`：版本过期时间
- `min_versions`：配合 TTL 保证最少保留数
- 版本清理：后台 GC/Compaction 时执行

**④ 读取 API 设计**：
```
GET key                      → 最新版本
GET key VERSION vid          → 精确版本
GET key VERSIONS n           → 最近 n 个版本
GET key VERSION_RANGE v1 v2  → 版本范围查询
```

#### 5.2 多副本一致性设计 — 渐进式方案

```
Phase 1: 异步复制（快速上线）
→ Phase 2: 支持 WAIT 语义（半同步可选）
→ Phase 3: 引入 Raft（核心数据强一致）
```

| 场景 | 推荐方案 | 优点 | 代价 |
|------|---------|------|------|
| 缓存为主 | Redis 异步复制 + WAIT 半同步 | 低延迟，架构简单 | 故障时可能丢最新写入 |
| 缓存+存储混合 | TiKV Raft 复制 | 强一致，已提交不丢 | 写入延迟 +1~2 RTT |
| 多 DC 部署 | Cassandra 可调一致性 | 灵活，按需调整 | 需 Anti-Entropy 修复 |

#### 5.3 关键设计决策清单

| 决策项 | 建议 | 理由 |
|--------|------|------|
| 版本号类型 | HLC | 单调递增+可排序+含物理时间信息 |
| Key 编码 | 后缀（key+version 逆序） | 最新版本查询 O(1) seek |
| 版本数上限 | 可配置（默认 1） | 兼容现有单版本行为 |
| 版本清理时机 | 后台 GC | 不影响读写热路径 |
| 删除策略 | 墓碑 + GC | 多副本场景必须用墓碑 |
| 一致性模型 | 可调（推荐） | 灵活适配不同业务场景 |
| 副本协议 | 分层：异步 + 可选 Raft | 缓存场景异步，存储场景 Raft |

### 六、各系统适用场景总结

| 系统 | 适合 | 不适合 |
|------|------|--------|
| **Redis** | 缓存、会话、限流、实时排行 | 需要多版本或强一致的场景 |
| **HBase** | 海量数据多版本存储、时序数据、日志 | 低延迟缓存 |
| **Cassandra** | 高写入吞吐、跨 DC 多活、IoT 数据 | 强一致或复杂查询 |
| **TiKV** | 分布式事务 OLTP、替代 MySQL 分库分表 | 纯缓存场景 |
| **FDB** | 最强一致性核心系统（如 Apple CloudKit） | 5s 事务限制 |

## 💡 个人思考

- 这份报告对于理解分布式存储的核心概念（MVCC、Raft、一致性模型）非常有价值，是后端开发者的必备知识
- **HBase 的 MVCC + WAL + ZooKeeper 三角关系** 是理解分布式一致性的经典案例，可以作为学习分布式系统的入门切入点
- TiKV 的 Multi-Raft + ReadIndex/Lease Read 优化思路，展示了在强一致性和低延迟之间的权衡艺术
- 自研 KV 系统的**渐进式方案**（异步 → 半同步 → Raft）是非常务实的工程策略，值得在实际项目中借鉴
- Cassandra 的可调一致性思想（W + R > N）在很多场景下是最实用的选择

## 🏷️ 关键词

`MVCC` `多版本控制` `Raft` `HBase` `TiKV` `Redis` `Cassandra` `FoundationDB` `分布式一致性` `WAL` `COW` `HLC` `多副本` `KV存储`

## 🔗 相关笔记

- 本文涉及的分布式系统设计理念与 AI Agent 的多角色协作有异曲同工之妙，参见 2026-03-20-garry-tan-gstack-claude-code-virtual-team
