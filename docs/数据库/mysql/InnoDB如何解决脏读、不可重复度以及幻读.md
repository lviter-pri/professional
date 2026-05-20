# InnoDB 如何解决脏读、不可重复读以及幻读

## 目录

1. [前置概念](#1-前置概念)
2. [问题定义](#2-问题定义)
3. [事务隔离级别](#3-事务隔离级别)
4. [InnoDB 解决方案](#4-innodb-解决方案)
5. [实战示例](#5-实战示例)

***

## 1. 前置概念

### 1.1 事务特性（ACID）

```mermaid
flowchart TD
    A[事务] --> B[Atomic<br/>原子性]
    A --> C[Consistency<br/>一致性]
    A --> D[Isolation<br/>隔离性]
    A --> E[Durability<br/>持久性]
    
    B --> B1[要么全部成功<br/>要么全部失败]
    C --> C1[事务前后<br/>数据状态一致]
    D --> D1[并发事务<br/>互不干扰]
    E --> E1[提交后<br/>永久保存]
```

### 1.2 并发事务问题

```mermaid
flowchart LR
    A[事务T1] --> B((读取))
    B --> C[数据]
    C --> D((读取))
    D --> E[事务T2]
    
    style C fill:#ff9999
```

***

## 2. 问题定义

### 2.1 脏读（Dirty Read）

**定义**：一个事务读取了另一个事务未提交的数据。

```mermaid
sequenceDiagram
    participant T1 as 事务A
    participant T2 as 事务B
    participant DB as 数据库
    
    T1->>DB: UPDATE account SET money = 800 WHERE id = 1
    T2->>DB: SELECT money FROM account WHERE id = 1
    DB-->>T2: 800 (脏数据!)
    T1->>DB: ROLLBACK (撤销操作)
    Note over T2: 读取到了不存在的数据!
```

**危害**：读到不存在的数据，导致业务逻辑错误。

***

### 2.2 不可重复读（Non-Repeatable Read）

**定义**：同一事务内，两次读取同一数据，结果不一致。

```mermaid
sequenceDiagram
    participant T1 as 事务A
    participant T2 as 事务B
    participant DB as 数据库
    
    T1->>DB: SELECT money FROM account WHERE id = 1
    DB-->>T1: money = 1000
    T2->>DB: UPDATE account SET money = 800 WHERE id = 1
    T2->>DB: COMMIT
    T1->>DB: SELECT money FROM account WHERE id = 1
    DB-->>T1: money = 800
    Note over T1: 同一事务内两次读取<br/>结果不一致!
```

**危害**：同一查询结果不同，影响业务判断。

***

### 2.3 幻读（Phantom Read）

**定义**：同一事务内，两次查询返回的记录数不同，像出现"幻影"。

```mermaid
sequenceDiagram
    participant T1 as 事务A
    participant T2 as 事务B
    participant DB as 数据库
    
    T1->>DB: SELECT * FROM users WHERE age > 18
    DB-->>T1: 5条记录
    T2->>DB: INSERT INTO users VALUES (6, 'new', 20)
    T2->>DB: COMMIT
    T1->>DB: SELECT * FROM users WHERE age > 18
    DB-->>T1: 6条记录
    Note over T1: 出现了"幻影"记录!
```

**危害**：统计数据不准确，如库存数量错误。

***

## 3. 事务隔离级别

### 3.1 四种隔离级别

| 隔离级别                 | 脏读  | 不可重复读 | 幻读  | 说明       |
| -------------------- | --- | ----- | --- | -------- |
| **READ UNCOMMITTED** | 可能  | 可能    | 可能  | 最低级别     |
| **READ COMMITTED**   | 不可能 | 可能    | 可能  | 大多数数据库默认 |
| **REPEATABLE READ**  | 不可能 | 不可能   | 可能  | MySQL默认  |
| **SERIALIZABLE**     | 不可能 | 不可能   | 不可能 | 最高级别     |

### 3.2 隔离级别对比

```mermaid
flowchart TD
    A[隔离级别] --> B[READ UNCOMMITTED]
    A --> C[READ COMMITTED]
    A --> D[REPEATABLE READ]
    A --> E[SERIALIZABLE]
    
    B --> B1[允许所有并发问题]
    C --> C1[禁止脏读]
    D --> D1[禁止脏读<br/>禁止不可重复读]
    E --> E1[禁止所有并发问题]
    
    B1 --- B2[性能最高]
    C1 --- C2[性能较好]
    D1 --- D2[性能适中]
    E1 --- E2[性能最低]
    
    style B fill:#ffcccc
    style C fill:#ffddcc
    style D fill:#ffffcc
    style E fill:#ccffcc
```

***

## 4. InnoDB 解决方案

### 4.1 整体架构

```mermaid
flowchart TD
    subgraph InnoDB[InnoDB 存储引擎]
        A[事务管理器] --> B[锁管理器]
        A --> C[MVCC控制器]
        B --> D[行锁]
        B --> E[表锁]
        C --> F[ReadView]
        C --> G[Undo Log]
    end
    
    subgraph 问题解决
        H[脏读] --> I[MVCC + 锁]
        J[不可重复读] --> K[MVCC + 锁]
        L[幻读] --> M[Next-Key Lock]
    end
```

***

### 4.2 解决方案一：MVCC（多版本并发控制）

#### 4.2.1 MVCC 工作原理

```mermaid
flowchart TD
    subgraph 数据行
        A[主键] --> B[id: 1]
        C[事务ID] --> D[trx_id: 100]
        E[版本号] --> F[row_version: 2]
        G[数据] --> H[name: 'Tom', age: 20]
    end
    
    subgraph Undo Log
        I[历史版本1] --> J[trx_id: 99, name: 'Jack']
        K[历史版本2] --> L[trx_id: 98, name: 'Mary']
    end
```

#### 4.2.2 ReadView 机制

```mermaid
flowchart TD
    A[事务开启] --> B[创建ReadView]
    B --> C[活跃事务列表]
    C --> D[m_ids: 100, 101, 102]
    D --> E[最小事务ID]
    E --> F[min_trx_id: 100]
    D --> G[创建时最大事务ID]
    G --> H[max_trx_id: 103]
    
    C --> I[读取数据时]
    I --> J{trx_id < min_trx_id?}
    J -->|是| K[可见]
    J -->|否| L{trx_id in m_ids?}
    L -->|是| M[不可见,查undo]
    L -->|否| N[可见]
```

#### 4.2.3 MVCC 解决脏读和不可重复读

```mermaid
flowchart TD
    subgraph T1事务[事务T1 - REPEATABLE READ]
        A1[T1开始] --> B1[创建ReadView m_ids=100]
        B1 --> C1[第一次读取< 看到v1]
        C1 --> D1[T1继续持有 同一个ReadView]
        D1 --> E1[第二次读取 仍看到v1]
    end
    
    subgraph T2事务[事务T2]
        A2[T2开始] --> B2[T2修改v1→v2 trx_id=100]
        B2 --> C2[T2提交]
    end
```

***

### 4.3 解决方案二：锁机制

#### 4.3.1 锁类型

| 锁类型     | 英文 | 作用   | 兼容性         |
| ------- | -- | ---- | ----------- |
| **共享锁** | S锁 | 允许读取 | 与S锁兼容，与X锁互斥 |
| **排他锁** | X锁 | 允许修改 | 与所有锁互斥      |

#### 4.3.2 锁粒度

```mermaid
flowchart TD
    A[锁] --> B[行锁]
    A --> C[表锁]
    
    B --> B1[只锁定某一行]
    B1 --> B2[并发度高<br/>锁开销大]
    
    C --> C1[锁定整张表]
    C1 --> C2[并发度低<br/>锁开销小]
    
    style B fill:#e8f5e9
    style C fill:#fff8e1
```

#### 4.3.3 记录锁（Record Lock）

```sql
-- 只锁定 id = 1 这一行
SELECT * FROM users WHERE id = 1 FOR UPDATE;
```

```mermaid
flowchart TD
    A[users表] --> B[id=1 锁定]
    A --> C[id=2 未锁定]
    A --> D[id=3 未锁定]
    
    style B fill:#ffcdd2
    style C fill:#c8e6c9
    style D fill:#c8e6c9
```

***

### 4.4 解决方案三：Next-Key Lock（临键锁）

#### 4.4.1 解决幻读的核心机制

```mermaid
flowchart TD
    subgraph Next-Key Lock
        A[Next-Key Lock] --> B[Record Lock<br/>记录锁]
        A --> C[Gap Lock<br/>间隙锁]
    end
    
    B --> B1[锁定索引本身]
    C --> C1[锁定索引间间隙]
```

#### 4.4.2 临键锁示例

假设 `id` 索引有值：1, 3, 5, 7, 9

```mermaid
flowchart TD
    subgraph 索引区间
        A[-∞, 1] --> B[1]
        B --> C[1, 3]
        C --> D[3]
        D --> E[3, 5]
        E --> F[5]
        F --> G[5, 7]
        G --> H[7]
        H --> I[7, 9]
        I --> J[9]
        J --> K[9, +∞]
    end
    
    subgraph 临键锁覆盖
        L[SELECT * FROM t WHERE id = 5 FOR UPDATE] --> M[锁定: 5 + 5,7]
    end
    
    style M fill:#ffcdd2
```

#### 4.4.3 临键锁解决幻读

```mermaid
sequenceDiagram
    participant T1 as 事务A
    participant T2 as 事务B
    participant DB as 数据库
    
    T1->>DB: SELECT * FROM users<br/>WHERE id > 3 FOR UPDATE
    Note over DB: 锁定 (3,5], (5,7], (7,9], (9,+∞)
    T2->>DB: INSERT INTO users<br/>VALUES (6, 'new')
    T2-->>DB: 等待锁...
    T1->>DB: SELECT * FROM users<br/>WHERE id > 3
    DB-->>T1: 5, 7, 9 (无幻影!)
    T1->>DB: COMMIT
    T2->>DB: 获取锁, INSERT成功
```

***

### 4.5 InnoDB 隔离级别实现

```mermaid
flowchart TD
    subgraph READ UNCOMMITTED
        A1[不使用MVCC]
        A2[直接读取最新版本]
        A3[可能脏读]
    end
    
    subgraph READ COMMITTED
        B1[使用MVCC]
        B2[每次读取创建新ReadView]
        B3[解决脏读]
        B4[可能出现不可重复读]
    end
    
    subgraph REPEATABLE READ
        C1[使用MVCC]
        C2[事务开始创建ReadView]
        C3[解决脏读和不可重复读]
        C4[使用Next-Key Lock]
        C5[解决幻读]
    end
    
    subgraph SERIALIZABLE
        D1[使用MVCC + 锁]
        D2[所有读取加S锁]
        D3[完全串行化]
    end
```

***

## 5. 实战示例

### 5.1 设置隔离级别

```sql
-- 查看当前隔离级别
SHOW VARIABLES LIKE 'transaction_isolation';

-- 设置隔离级别
SET SESSION transaction_isolation = 'REPEATABLE-READ';
SET GLOBAL transaction_isolation = 'READ-COMMITTED';
```

### 5.2 验证脏读

```sql
-- 终端1：设置未提交读
SET SESSION transaction_isolation = 'READ-UNCOMMITTED';

-- 终端1：开启事务
START TRANSACTION;
UPDATE account SET balance = 800 WHERE id = 1;

-- 终端2：查询（会看到未提交的数据！）
SELECT * FROM account WHERE id = 1;  -- balance = 800

-- 终端1：回滚
ROLLBACK;

-- 终端2：再次查询
SELECT * FROM account WHERE id = 1;  -- balance = 1000
```

### 5.3 验证不可重复读

```sql
-- 终端1：设置读已提交
SET SESSION transaction_isolation = 'READ-COMMITTED';

-- 终端1：开启事务
START TRANSACTION;
SELECT * FROM account WHERE id = 1;  -- balance = 1000

-- 终端2：更新并提交
UPDATE account SET balance = 800 WHERE id = 1;
COMMIT;

-- 终端1：再次查询（结果不同！）
SELECT * FROM account WHERE id = 1;  -- balance = 800
COMMIT;
```

### 5.4 验证幻读（REPEATABLE READ）

```sql
-- 终端1：设置可重复读
SET SESSION transaction_isolation = 'REPEATABLE-READ';

-- 终端1：开启事务
START TRANSACTION;
SELECT * FROM users WHERE age > 18;  -- 5条记录

-- 终端2：插入新记录
INSERT INTO users VALUES (6, 'new', 20);
COMMIT;

-- 终端1：再次查询（结果相同，无幻读！）
SELECT * FROM users WHERE age > 18;  -- 仍为5条记录

-- 终端1：插入相同条件的记录（会失败！）
INSERT INTO users VALUES (7, 'another', 22);
-- ERROR: Duplicate entry 或 锁等待超时
COMMIT;
```

### 5.5 验证 Next-Key Lock

```sql
-- 终端1：
START TRANSACTION;
SELECT * FROM users WHERE id BETWEEN 3 AND 5 FOR UPDATE;
-- 锁定范围： (1,3], (3,5], (5,7]

-- 终端2：以下操作会被阻塞
INSERT INTO users VALUES (4, 'test', 20);  -- 阻塞！
INSERT INTO users VALUES (6, 'test', 20);  -- 阻塞！
INSERT INTO users VALUES (2, 'test', 20);  -- 可以（不在范围内）
```

***

## 总结

```mermaid
flowchart TD
    A[并发问题] --> B[脏读]
    A --> C[不可重复读]
    A --> D[幻读]
    
    B --> E[MVCC]
    C --> E
    C --> F[MVCC + ReadView]
    D --> G[Next-Key Lock]
    
    E --> H[READ COMMITTED]
    F --> I[REPEATABLE READ]
    G --> I
    
    style H fill:#fff3e0
    style I fill:#e8f5e9
```

### InnoDB 解决方案一览

| 问题        | 隔离级别            | 解决方案                   |
| --------- | --------------- | ---------------------- |
| **脏读**    | READ COMMITTED+ | MVCC 读取已提交版本           |
| **不可重复读** | REPEATABLE READ | MVCC + 事务级 ReadView    |
| **幻读**    | REPEATABLE READ | Next-Key Lock（间隙锁+记录锁） |

***

## 参考资料

1. [MySQL 官方文档 - InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
2. [MySQL 官方文档 - MVCC](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)
3. [MySQL 事务隔离级别](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)

