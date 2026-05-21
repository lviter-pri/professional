# MySQL 中的锁

## 目录

1. [锁的概述](#1-锁的概述)
2. [锁的类型](#2-锁的类型)
3. [锁的粒度](#3-锁的粒度)
4. [InnoDB 锁机制](#4-innodb-锁机制)
5. [锁的兼容性](#5-锁的兼容性)
6. [死锁问题](#6-死锁问题)
7. [实战示例](#7-实战示例)

***

## 1. 锁的概述

### 1.1 什么是锁

**锁**是数据库用于控制并发访问的机制，确保数据的一致性和完整性。

```mermaid
flowchart TD
    A[并发事务] --> B[锁机制]
    B --> C[数据访问控制]
    C --> D[数据一致性]
```

### 1.2 锁的作用

| 作用        | 说明               |
| --------- | ---------------- |
| **互斥访问**  | 防止多个事务同时修改同一数据   |
| **数据一致性** | 保证事务的原子性和隔离性     |
| **并发控制**  | 在保证一致性的前提下提高并发性能 |

***

## 2. 锁的类型

### 2.1 共享锁（Shared Lock，S锁）

**定义**：多个事务可以同时持有同一资源的共享锁。

```mermaid
flowchart TD
    subgraph 共享锁
        A[事务T1] --> B[获取S锁]
        C[事务T2] --> B
        D[事务T3] --> B
        
        B --> E[数据资源]
        
        F[允许] --> G[读取操作]
        H[禁止] --> I[写入操作]
    end
```

**语法**：

```sql
SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE;
```

### 2.2 排他锁（Exclusive Lock，X锁）

**定义**：只有一个事务可以持有资源的排他锁。

```mermaid
flowchart TD
    subgraph 排他锁
        A[事务T1] --> B[获取X锁]
        C[事务T2] --> D[等待...]
        
        B --> E[数据资源]
        
        F[允许] --> G[读取操作]
        F --> H[写入操作]
        I[禁止] --> J[其他事务操作]
    end
```

**语法**：

```sql
SELECT * FROM table_name WHERE ... FOR UPDATE;
UPDATE table_name SET ... WHERE ...;
DELETE FROM table_name WHERE ...;
INSERT INTO table_name ...;
```

***

## 3. 锁的粒度

### 3.1 表级锁（Table Lock）

**定义**：锁定整张表，粒度最大。

```mermaid
flowchart TD
    A[表级锁] --> B[锁定整张表]
    B --> C[并发度低]
    B --> D[锁开销小]
    
    E[适用场景] --> F[批量操作]
    E --> G[DDL操作]
```

**特点**：

- **优点**：锁开销小，获取和释放快
- **缺点**：并发度低，容易发生锁等待

**示例**：

```sql
-- 手动加表锁
LOCK TABLES users READ;
LOCK TABLES users WRITE;

-- 解锁
UNLOCK TABLES;
```

### 3.2 行级锁（Row Lock）

**定义**：只锁定需要操作的行，粒度最小。

```mermaid
flowchart TD
    A[行级锁] --> B[只锁定目标行]
    B --> C[并发度高]
    B --> D[锁开销大]
    
    E[适用场景] --> F[OLTP系统]
    E --> G[高并发写入]
```

**特点**：

- **优点**：并发度高，不同行可以同时操作
- **缺点**：锁开销大，管理复杂

**示例**：

```sql
-- InnoDB 自动加行锁
UPDATE users SET name = 'new' WHERE id = 1;  -- 只锁 id=1 的行
```

### 3.3 页级锁（Page Lock）

**定义**：锁定数据页（16KB）。

```mermaid
flowchart TD
    A[页级锁] --> B[锁定数据页]
    B --> C[粒度介于表锁和行锁之间]
    
    D[适用引擎] --> E[BDB]
```

***

## 4. InnoDB 锁机制

### 4.1 InnoDB 锁类型

```mermaid
flowchart TD
    subgraph InnoDB锁类型
        A[行锁] --> B[记录锁]
        A --> C[间隙锁]
        A --> D[临键锁]
        A --> E[意向锁]
    end
```

### 4.2 记录锁（Record Lock）

**定义**：锁定索引记录本身。

```mermaid
flowchart TD
    subgraph 记录锁
        A[id=1] --> B[锁定]
        C[id=2] --> D[未锁定]
        E[id=3] --> F[未锁定]
        
        style B fill:#ffcdd2
        style D fill:#c8e6c9
        style F fill:#c8e6c9
    end
```

**示例**：

```sql
SELECT * FROM users WHERE id = 1 FOR UPDATE;
-- 只锁定 id=1 的行
```

### 4.3 间隙锁（Gap Lock）

**定义**：锁定索引之间的间隙，防止插入新记录。

```mermaid
flowchart TD
    subgraph 间隙锁
        A[-∞] --> B[(1,3)]
        B --> C[(3,5)]
        C --> D[(5,7)]
        D --> E[(7,+∞)]
        
        style B fill:#ffe0b2
        style C fill:#ffe0b2
        style D fill:#ffe0b2
        style E fill:#ffe0b2
    end
    
    F[防止在间隙中插入新记录]
```

**示例**：

```sql
SELECT * FROM users WHERE id BETWEEN 1 AND 5 FOR UPDATE;
-- 锁定间隙：(-∞,1), (1,3), (3,5), (5,+∞)
```

### 4.4 临键锁（Next-Key Lock）

**定义**：记录锁 + 间隙锁的组合。

```mermaid
flowchart TD
    subgraph 临键锁
        A[Record Lock] --> B[锁定索引记录]
        C[Gap Lock] --> D[锁定间隙]
        
        E[Next-Key Lock] --> F[Record Lock + Gap Lock]
    end
```

**工作原理**：

```mermaid
flowchart LR
    subgraph 索引区间
        A[-∞,1] --> B[1] --> C[1,3] --> D[3] --> E[3,5]
    end
    
    F[SELECT * WHERE id = 3 FOR UPDATE] --> G[锁定: 1,3 + 3,5]
    
    style G fill:#ffcdd2
```

### 4.5 意向锁（Intention Lock）

**定义**：表明事务打算在表中的某行加锁。

```mermaid
flowchart TD
    subgraph 意向锁
        A[意向共享锁 IS] --> B[打算加S锁]
        C[意向排他锁 IX] --> D[打算加X锁]
        
        E[表级锁] --> F[快速判断]
    end
```

**作用**：

- **IS锁**：事务打算对表中的某些行加共享锁
- **IX锁**：事务打算对表中的某些行加排他锁

***

## 5. 锁的兼容性

### 5.1 锁兼容性矩阵

| 请求锁     | S锁 | X锁 | IS锁 | IX锁 |
| ------- | -- | -- | --- | --- |
| **S锁**  | 兼容 | 冲突 | 兼容  | 兼容  |
| **X锁**  | 冲突 | 冲突 | 冲突  | 冲突  |
| **IS锁** | 兼容 | 冲突 | 兼容  | 兼容  |
| **IX锁** | 兼容 | 冲突 | 兼容  | 兼容  |

### 5.2 兼容性图解

```mermaid
flowchart TD
    subgraph 兼容关系
        A[S锁] --> B[与S锁兼容]
        A --> C[与IS锁兼容]
        A --> D[与IX锁兼容]
        A --> E[与X锁冲突]
        
        F[X锁] --> G[与所有锁冲突]
    end
```

***

## 6. 死锁问题

### 6.1 死锁定义

**死锁**：两个或多个事务互相等待对方释放锁。

```mermaid
sequenceDiagram
    participant T1 as 事务A
    participant T2 as 事务B
    participant DB as 数据库
    
    T1->>DB: 获取行1的X锁
    T2->>DB: 获取行2的X锁
    T1->>DB: 尝试获取行2的X锁
    T2->>DB: 尝试获取行1的X锁
    
    Note over T1,T2: 死锁! 双方互相等待
```

### 6.2 死锁产生条件

```mermaid
flowchart TD
    A[死锁条件] --> B[互斥条件]
    A --> C[请求保持条件]
    A --> D[不可剥夺条件]
    A --> E[循环等待条件]
    
    B --> B1[资源只能被一个事务占用]
    C --> C1[持有锁的同时请求新锁]
    D --> D1[锁不能被强制剥夺]
    E --> E1[事务形成等待循环]
```

### 6.3 死锁检测与处理

```mermaid
flowchart TD
    A[InnoDB死锁检测] --> B[定时检测]
    B --> C[检测到死锁]
    C --> D[选择牺牲者]
    D --> E[回滚代价最小的事务]
    E --> F[释放锁]
    F --> G[其他事务继续]
```

### 6.4 死锁预防策略

| 策略           | 说明                         |
| ------------ | -------------------------- |
| **固定加锁顺序**   | 所有事务按相同顺序获取锁               |
| **减少事务大小**   | 事务尽量短，减少锁持有时间              |
| **使用较低隔离级别** | 减少锁的范围                     |
| **设置锁等待超时**  | `innodb_lock_wait_timeout` |

***

## 7. 实战示例

### 7.1 共享锁与排他锁

```sql
-- 终端1：开启事务，获取共享锁
START TRANSACTION;
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE;

-- 终端2：可以获取共享锁
START TRANSACTION;
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE;  -- 成功

-- 终端3：尝试获取排他锁（等待）
START TRANSACTION;
SELECT * FROM users WHERE id = 1 FOR UPDATE;  -- 阻塞

-- 终端1：提交事务
COMMIT;

-- 终端3：获取锁成功
```

### 7.2 行级锁示例

```sql
-- 终端1：更新 id=1 的行
START TRANSACTION;
UPDATE users SET name = 'A' WHERE id = 1;

-- 终端2：更新 id=2 的行（不受影响）
START TRANSACTION;
UPDATE users SET name = 'B' WHERE id = 2;  -- 成功

-- 终端3：更新 id=1 的行（等待）
START TRANSACTION;
UPDATE users SET name = 'C' WHERE id = 1;  -- 阻塞

-- 终端1：提交
COMMIT;

-- 终端3：获取锁成功
```

### 7.3 间隙锁示例

```sql
-- 创建测试表
CREATE TABLE test (
    id INT PRIMARY KEY
);

-- 插入测试数据
INSERT INTO test VALUES (1), (3), (5);

-- 终端1：开启事务，使用范围查询
START TRANSACTION;
SELECT * FROM test WHERE id BETWEEN 1 AND 5 FOR UPDATE;
-- 锁定范围：(-∞,1], (1,3], (3,5], (5,+∞)

-- 终端2：尝试插入新记录（阻塞）
START TRANSACTION;
INSERT INTO test VALUES (2);  -- 阻塞！在间隙 (1,3) 中
INSERT INTO test VALUES (4);  -- 阻塞！在间隙 (3,5) 中
INSERT INTO test VALUES (6);  -- 阻塞！在间隙 (5,+∞) 中

-- 终端1：提交
COMMIT;

-- 终端2：插入成功
```

### 7.4 死锁示例

```sql
-- 终端1：开启事务，锁定行1
START TRANSACTION;
UPDATE users SET name = 'T1-1' WHERE id = 1;

-- 终端2：开启事务，锁定行2
START TRANSACTION;
UPDATE users SET name = 'T2-2' WHERE id = 2;

-- 终端1：尝试锁定行2（等待）
UPDATE users SET name = 'T1-2' WHERE id = 2;

-- 终端2：尝试锁定行1（死锁！）
UPDATE users SET name = 'T2-1' WHERE id = 1;

-- InnoDB检测到死锁，回滚其中一个事务
```

### 7.5 查看锁状态

```sql
-- 查看当前事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;

-- 查看锁等待
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

-- 查看锁详情
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

-- 查看进程列表
SHOW PROCESSLIST;
```

***

## 总结

```mermaid
flowchart TD
    subgraph MySQL锁体系
        A[锁类型] --> B[S锁]
        A --> C[X锁]
        
        D[锁粒度] --> E[表锁]
        D --> F[行锁]
        
        G[InnoDB锁] --> H[记录锁]
        G --> I[间隙锁]
        G --> J[临键锁]
        G --> K[意向锁]
    end
```

### 锁选择建议

| 场景       | 推荐锁类型          |
| -------- | -------------- |
| **读多写少** | 使用S锁或不加锁（MVCC） |
| **写操作**  | 使用X锁           |
| **批量更新** | 考虑表锁或分批处理      |
| **范围查询** | 注意间隙锁的影响       |

***

## 参考资料

1. [MySQL 官方文档 - InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
2. [MySQL 官方文档 - Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)
3. [MySQL 官方文档 - Deadlocks](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html)

