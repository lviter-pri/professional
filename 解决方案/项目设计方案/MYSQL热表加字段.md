# MYSQL热表加字段

场景：生产mysql一张千万数据热表，新加字段加不上，是什么原因，以及有什么解决方案

分析：这种情况的核心矛盾在于：**DDL（数据定义语言）操作需要锁表，而高并发的热表由于长事务或大量查询，导致锁等待超时或直接造成数据库瘫痪。**

### 核心原因排查

### 1. MDL（元数据锁）冲突

这是最常见的原因。MySQL 在对表进行 DDL 操作时，需要获取 **MDL 写锁**。

- **现象：** 如果此时表上有未提交的长事务，或者有大量的查询正在运行，DDL 就会被阻塞（Waiting for table metadata lock）。
- **连锁反应：** 更有趣（也更危险）的是，DDL 申请写锁会排在队列里，它会阻塞后续所有对该表的读写请求，导致整个业务线瞬间“卡死”。

### 2. 锁超时 (Lock Wait Timeout)

如果你的 `innodb_lock_wait_timeout` 设置得较短，而 DDL 一直拿不到锁，操作就会直接报错失败。

### 3. Copy Table 机制导致的 I/O 瓶颈

虽然 MySQL 5.6 以后引入了 **Online DDL**，但在某些情况下（如修改字段类型、全表重建），MySQL 依然会采用 `COPY` 算法。

- **现象：** 系统会创建一个临时表，把千万行数据一行行拷过去。在热表上，这会消耗极高的 CPU 和 I/O，可能导致磁盘空间不足或主从延迟爆炸。

### 查询分析：开启 `performance_schema` 的相关监控项

1. 查询当前的锁阻塞关系：MySql5.7之后，详细的元数据锁监控

   ```sql
   SELECT
       waiting_thread_id AS waiting_tid,
       waiting_pid AS waiting_query_id,
       waiting_query AS waiting_sql,
       blocking_thread_id AS blocking_tid,
       blocking_pid AS blocking_query_id,
       blocking_query AS blocking_sql
   FROM sys.schema_table_lock_waits
   WHERE waiting_query LIKE '%ALTER TABLE%'; -- 替换成你的表名或操作
   ```

   

2. 查看活跃线程状态（最常用）:如果在加字段时发现卡住了，直接运行这个命令。重点关注 `State` 列为 `Waiting for table metadata lock` 的记录。

   ```sql
   SELECT 
       ID, USER, HOST, DB, COMMAND, TIME, STATE, INFO 
   FROM information_schema.processlist 
   WHERE STATE = 'Waiting for table metadata lock' 
      OR INFO LIKE '%ALTER TABLE%';
   ```

   **如何处理：** 找到那个 `Time` 很大（运行很久）且状态为 `Sleep` 或执行查询的 `ID`，然后根据业务情况考虑是否使用 `KILL [ID];` 释放锁。

3. 精准查找持有锁的 Session (底层排查)：

------

## 解决方案

### 方案一：使用专业工具 (推荐)

对于生产环境的热表，直接执行 `ALTER TABLE` 是极具风险的。推荐使用“影子表”方案：

- **gh-ost (GitHub 开源):** * **原理：** 通过模拟从库解析 Binlog，将增量数据同步到新表，不需要触发器，对性能影响最小，支持随时暂停。
  - **优点：** 极其安全，不会产生 MDL 锁死。
- **pt-online-schema-change (Percona Toolkit):**
  - **原理：** 创建触发器，在原表进行增删改时同步到新表。
  - **优点：** 经典稳定，但触发器在极高并发下会有一定的性能损耗。

### 方案二：优化 Online DDL 执行策略

如果你一定要用 SQL 执行，请务必遵循以下步骤：

1. **检查长事务：** 执行前先杀掉所有活跃的长查询或事务。

2. **显式指定算法：** ```sql ALTER TABLE your_table ADD COLUMN new_col INT, ALGORITHM=INPLACE, LOCK=NONE;

   ```
   * `ALGORITHM=INPLACE`：尽量原地修改。
   * `LOCK=NONE`：确保不阻塞读写（如果该变更不支持此模式，SQL 会直接报错，而不是锁表）。
   ```

### 方案三：分时段与小步快跑

- **低峰期操作：** 避开业务高峰。
- **设置锁超时：** 在当前 Session 修改 `lock_wait_timeout`，防止长时间阻塞。
- **主从切换：** 在从库加字段，然后进行主从切换（这种方法操作复杂，风险较高，通常作为备选）。