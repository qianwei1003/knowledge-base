---
来源: Sematext 技术博客
原始链接: https://sematext.com/blog/postgresql-performance-tuning/
---

# PostgreSQL 性能调优与优化指南

> 来源：Sematext — PostgreSQL Performance Tuning and Optimization Guide
> 整理时间：2026-04-19

## 概述

PostgreSQL 是一个高度灵活可靠的开源关系数据库，提供丰富的功能集。但由于可以以不同方式部署，它开箱即用时只根据部署环境提供基本的性能调优。部署数据库后，必须根据用例进行调优以获得最佳性能。

调优 PostgreSQL 不类似于调优其他数据库——可以为每个 schema 设置不同的性能指标，例如频繁写入或频繁读取。

## 一、数据库设计

设计数据库是优化任何数据库性能最重要的步骤之一。

### 数据分区

利用 PostgreSQL 的关系数据库特性，将数据分区到多个逻辑分离的表中，而不是一个大型表。这通常会立即显著改善查询性能。

### 索引策略

创建足够的索引，在常用过滤条件的列上建立索引，尽可能多地过滤数据。

**注意**：过度使用索引会产生反效果——创建和维护索引是昂贵的操作，过多索引会降低整体性能。

## 二、硬件调优

### CPU

复杂操作（聚合、连接、哈希、分组、排序等）需要 CPU 时间。升级 CPU 成本较高，需在优化查询和硬件升级之间进行权衡。

### RAM（内存）

查询提交时，数据首先加载到内存中。增加内存可以提高磁盘缓存并减少磁盘 I/O 操作，显著提升查询性能。

- 内存不足 → 查询可能被 "out of memory" 异常终止
- 内存充足 → 减少 I/O 操作，提升响应速度

### 磁盘 I/O

- 快速的读写时间大幅提升查询性能
- 大量 I/O 操作时，考虑将每个表空间存储在不同磁盘驱动器上
- 使用 SSD 替代 HDD 可显著改善性能

### 网络

数据量增长后，网络延迟成为常见性能瓶颈。确保使用可靠、高速、高带宽的网络卡，节点间高速连接同样关键。

## 三、操作系统优化

操作系统是数据库软件和底层硬件之间的通信层。

### TCP Keepalive

常见连接丢失错误：
- `could not receive data from client: Connection reset by peer`
- `server closed the connection unexpectedly`

Windows 和 Linux 默认 keepalive 空闲时间为 2 小时：
- Linux 每 75 秒发送 keepalive 信号
- Windows 每 1 秒发送 keepalive 信号

根据需求调整这些配置参数。

## 四、数据库配置参数调优

### 连接配置

```properties
# 最大连接数计算公式
max_connections = max(4 * CPU核心数, 100)
```

确保 CPU 不会同时被过多活跃连接过载，同时有足够的硬件资源支持并行连接。

### 内存配置

> 所有内存相关配置总和不应超过可用内存总量。

| 参数 | 说明 | 建议值 |
|------|------|--------|
| `shared_buffers` | 共享内存缓冲区 | 可用内存的 ~25% |
| `temp_buffers` | 每个会话的临时缓冲区 | 按活跃会话数 × 单个会话需求计算 |
| `wal_buffers` | WAL 数据缓冲区 | 默认 -1（约 shared_buffers 的 3%） |
| `work_mem` | 单个查询可用内存 | 默认 4MB，根据用例调整 |
| `maintenance_work_mem` | 维护操作内存 | 建议至少 1GB |

### Write-Ahead Log（WAL）配置

| 参数 | 说明 |
|------|------|
| `fsync` | 确保所有更新先写入磁盘（影响性能但保证数据完整性） |
| `commit_delay` | WAL 刷新前的等待时间，允许多个事务一起刷新 |
| `checkpoint_timeout` | 两个 WAL 检查点之间的最大时间（30秒~1天，默认5分钟） |
| `checkpoint_completion_target` | 检查点写入在 checkpoint_timeout 内完成的进度（默认0.9） |
| `min_wal_size` / `max_wal_size` | 替代已废弃的 checkpoint_segments（PG 9.5+） |

**WAL 大小计算公式**（从旧的 checkpoint_segments 迁移）：
```
max_wal_size = (3 * checkpoint_segments) * 16MB
```

### 查询成本配置

| 参数 | 说明 | 默认值 | 调优方向 |
|------|------|--------|----------|
| `random_page_cost` | 非顺序获取磁盘页面的成本 | 4 | 减小 → 偏好索引扫描；增大 → 索引扫描更昂贵 |
| `effective_cache_size` | 每个查询可用磁盘缓存的估计大小 | 4GB | 根据实际可用内存调整 |

## 五、日志配置

| 参数 | 说明 |
|------|------|
| `logging_collector` | 将所有日志消息重定向到日志文件 |
| `log_statement` | 控制记录的查询类型：none / ddl / mod / all |
| `log_min_error_statement` | 记录产生指定级别错误的 SQL 查询 |
| `log_line_prefix` | 每行日志的前缀格式（printf 风格） |
| `log_lock_waits` | 在 deadlock_timeout 后记录锁等待信息 |
| `log_checkpoints` | 记录检查点和重启点信息 |

## 六、Vacuum（清理）

当行被更新或删除时，PostgreSQL 不会物理删除记录，而是在磁盘上留下废弃记录，消耗空间并影响查询性能。

### Vacuum 命令变体

| 命令 | 说明 |
|------|------|
| `VACUUM` | 删除废弃记录，释放的空间保留给表未来使用 |
| `VACUUM ANALYZE` | 删除废弃记录 + 刷新表统计信息 |
| `VACUUM FULL` | 重写整个表到新磁盘文件，释放空间给操作系统（需独占访问） |

### Autovacuum

PostgreSQL 自带的自动 Vacuum 机制，定期运行清理命令。

- 可配置运行时间和频率
- 在操作较少的时段调度，减少对并行操作的影响
- 管理员也可随时手动对特定表运行 Vacuum

## 七、查询性能调优

### ANALYZE

收集表统计信息，供查询规划器制定执行计划。

- 统计信息会随时间过时
- 定期运行 `ANALYZE` 保持统计信息最新
- 查询规划器使用这些统计信息优化执行路径

### EXPLAIN

查看查询的执行计划，了解数据库将如何执行查询。

- 是否使用索引扫描还是表扫描
- 基于执行计划优化查询或更新统计信息
- 使用 `EXPLAIN ANALYZE` 获取实际执行时间和行数

### 索引性能

索引对查询性能至关重要，但需注意：
- 索引过多会增加写入开销（每次数据更新都需更新索引）
- 根据表大小、数据访问模式、索引算法等因素综合考虑
- 彻底理解表的所有使用场景再决定索引策略

## 八、监控工具建议

优化 PostgreSQL 性能需要持续关注以下问题：
- 是否需要添加更多物理服务器资源？
- 是否有慢查询影响性能？
- 是否需要更多索引来加速常用查询？
- 是否需要调整 max_connections 配置？

建议使用专业的 PostgreSQL 监控工具来识别瓶颈并决定适当的操作。

## 关键配置速查表

```properties
# 基础性能配置模板（16GB RAM, 4核 CPU）

# 连接
max_connections = 100

# 内存
shared_buffers = 4GB          # 25% of RAM
work_mem = 64MB               # 根据并发查询数调整
maintenance_work_mem = 1GB
effective_cache_size = 12GB   # 75% of RAM

# WAL
wal_buffers = 64MB
checkpoint_timeout = 10min
checkpoint_completion_target = 0.9

# 查询规划器
random_page_cost = 1.1        # SSD 环境降低此值
effective_io_concurrency = 200  # SSD 环境

# 自动清理
autovacuum = on
autovacuum_max_workers = 3
```
