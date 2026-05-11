# Synchronized锁详解

## 一、Synchronized基本用法

### 1.1 Synchronized的三种用法

Synchronized是Java中最基本的同步机制，可以修饰**实例方法**、**静态方法**或**代码块**。

#### 1.1.1 修饰实例方法

```java
public class SynchronizedDemo {
    
    /**
     * 修饰实例方法
     * 锁对象是当前实例 this
     */
    public synchronized void instanceMethod() {
        // 临界区代码
        System.out.println("实例方法锁: " + Thread.currentThread().getName());
    }
}
```

#### 1.1.2 修饰静态方法

```java
public class SynchronizedDemo {
    
    /**
     * 修饰静态方法
     * 锁对象是当前类的Class对象
     */
    public static synchronized void staticMethod() {
        // 临界区代码
        System.out.println("静态方法锁: " + Thread.currentThread().getName());
    }
}
```

#### 1.1.3 修饰代码块

```java
public class SynchronizedDemo {
    private final Object lock = new Object();
    
    /**
     * 修饰代码块 - 使用自定义锁对象
     */
    public void blockMethod() {
        synchronized (lock) {
            // 临界区代码
            System.out.println("代码块锁: " + Thread.currentThread().getName());
        }
    }
    
    /**
     * 修饰代码块 - 使用this作为锁对象
     */
    public void blockMethodThis() {
        synchronized (this) {
            // 临界区代码
            System.out.println("this锁: " + Thread.currentThread().getName());
        }
    }
    
    /**
     * 修饰代码块 - 使用Class对象作为锁对象
     */
    public void blockMethodClass() {
        synchronized (SynchronizedDemo.class) {
            // 临界区代码
            System.out.println("Class锁: " + Thread.currentThread().getName());
        }
    }
}
```

### 1.2 关键注意点

| 特性       | 说明                     |
| -------- | ---------------------- |
| **互斥性**  | 一把锁只能同时被一个线程获取，其他线程需等待 |
| **锁粒度**  | 实例方法锁是实例级别，静态方法锁是类级别   |
| **自动释放** | 方法正常执行完或抛出异常时，都会自动释放锁  |
| **可重入性** | 同一线程可以多次获取同一把锁         |

***

## 二、Java对象头结构

### 2.1 对象头组成

Java对象在内存中由三部分组成：

```mermaid
flowchart TD
    A[对象内存布局] --> B[对象头]
    A --> C[实例数据]
    A --> D[对齐填充]
    
    B --> E[Mark Word]
    B --> F[类型指针]
    B --> G[数组长度]
    
    E --> H[锁状态]
    E --> I[线程ID]
    E --> J[Epoch]
    E --> K[hash]
```

### 2.2 Mark Word结构

Mark Word是对象头中存储锁信息的部分，其结构随锁状态变化：

| 锁状态      | 占用位 | 存储内容                                                         |
| -------- | --- | ------------------------------------------------------------ |
| **无锁**   | 64位 | 对象哈希码(25位) + 分代年龄(4位) + 是否偏向锁(1位) + 锁标志位(2位=01)              |
| **偏向锁**  | 64位 | 线程ID(54位) + Epoch(2位) + 分代年龄(4位) + 是否偏向锁(1位=1) + 锁标志位(2位=01) |
| **轻量级锁** | 64位 | 指向栈帧中锁记录的指针(62位) + 锁标志位(2位=00)                               |
| **重量级锁** | 64位 | 指向Monitor对象的指针(62位) + 锁标志位(2位=10)                            |

### 2.3 锁标志位说明

| 锁标志位 | 锁状态                |
| ---- | ------------------ |
| `01` | 无锁或偏向锁（根据是否偏向锁位判断） |
| `00` | 轻量级锁               |
| `10` | 重量级锁               |
| `11` | GC标记               |

***

## 三、锁升级过程

### 3.1 锁升级方向

```mermaid
flowchart LR
    A[无锁] -->|第一次获取| B[偏向锁]
    B -->|竞争发生| C[轻量级锁]
    C -->|自旋失败| D[重量级锁]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#9f9,stroke:#333,stroke-width:2px
    style C fill:#ff9,stroke:#333,stroke-width:2px
    style D fill:#f99,stroke:#333,stroke-width:2px
```

**注意**：锁升级是**单向的**，只能从低级向高级升级，不能降级。但偏向锁可以被重置为无锁状态。

### 3.2 各阶段详细说明

#### 3.2.1 无锁 → 偏向锁

**触发条件**：第一次有线程获取锁，且无其他线程竞争。

**执行过程**：

```mermaid
sequenceDiagram
    participant Thread as 线程T1
    participant Object as 锁对象
    
    Thread->>Object: 尝试获取锁
    Note over Object: 检查锁标志位=01且偏向锁位=0
    Object->>Object: 将线程ID写入Mark Word
    Object->>Object: 设置偏向锁位=1
    Object-->>Thread: 获取偏向锁成功
```

**特点**：

- 偏向锁不会主动释放锁
- 后续同一线程再次获取锁时，只需比较线程ID即可
- 如果其他线程尝试获取锁，才会考虑撤销偏向锁

#### 3.2.2 偏向锁 → 轻量级锁

**触发条件**：有其他线程尝试获取已偏向的锁。

**执行过程**：

```mermaid
sequenceDiagram
    participant T1 as 线程T1(持有偏向锁)
    participant T2 as 线程T2(尝试获取)
    participant Object as 锁对象
    
    T2->>Object: 尝试获取锁
    Note over Object: 发现Mark Word中线程ID≠T2
    Object->>Object: 检查T1是否存活
    alt T1已退出或不使用锁
        Object->>Object: 重置为无锁状态
        Object-->>T2: T2获取偏向锁
    else T1仍在执行
        Object->>Object: 撤销偏向锁，升级为轻量级锁
        Note over T1,T2: 双方使用CAS竞争
    end
```

**轻量级锁原理**：

1. 线程获取锁时，将Mark Word复制到栈帧的**Displaced Mark Word**
2. 使用CAS将对象头替换为指向Displaced Mark Word的指针
3. 如果CAS成功，获取锁；失败则自旋等待

#### 3.2.3 轻量级锁 → 重量级锁

**触发条件**：

- 自旋次数达到阈值（默认10次）
- 自旋期间有新线程加入竞争
- 自旋线程数超过CPU核心数的一半

**执行过程**：

```mermaid
sequenceDiagram
    participant T1 as 线程T1(持有轻量级锁)
    participant T2 as 线程T2(自旋等待)
    participant T3 as 线程T3(新竞争线程)
    participant Object as 锁对象
    
    T2->>Object: 自旋CAS尝试获取锁
    Note over T2: 自旋10次仍未成功
    T3->>Object: 尝试获取锁
    Object->>Object: 膨胀为重量级锁
    Object->>Object: Mark Word指向Monitor对象
    Object-->>T2: 阻塞T2
    Object-->>T3: 阻塞T3
```

**重量级锁原理**：

- 使用操作系统的互斥量实现
- 未获取锁的线程进入阻塞状态
- 锁释放时唤醒等待线程

***

## 四、各种锁的特点对比

| 特性       | 无锁       | 偏向锁      | 轻量级锁     | 重量级锁        |
| -------- | -------- | -------- | -------- | ----------- |
| **锁标志位** | 01       | 01       | 00       | 10          |
| **适用场景** | 无竞争      | 单线程重复获取  | 多线程交替执行  | 高并发竞争       |
| **性能开销** | 无        | 极低       | 低（自旋）    | 高（系统调用）     |
| **线程状态** | RUNNABLE | RUNNABLE | RUNNABLE | BLOCKED（阻塞） |
| **锁释放**  | 无需释放     | 不会主动释放   | 执行完毕释放   | 执行完毕释放      |
| **可重入性** | -        | 支持       | 支持       | 支持          |

***

## 五、锁的核心机制

### 5.1 Monitor（管程）

重量级锁依赖Monitor对象实现：

```mermaid
flowchart TD
    A[Monitor对象] --> B[Owner]
    A --> C[EntryList]
    A --> D[WaitSet]
    
    B --> E[持有锁的线程]
    C --> F[等待获取锁的线程队列]
    D --> G[调用wait的线程队列]
```

**Monitor工作原理**：

1. `Owner`：指向当前持有锁的线程
2. `EntryList`：等待获取锁的线程队列（阻塞状态）
3. `WaitSet`：调用`wait()`后等待的线程队列

### 5.2 synchronized注意点

```java
public class SynchronizedNotes {
    
    /**
     * 注意点1：不同实例的锁互不影响
     */
    public synchronized void method1() {
        // 锁是当前实例 this
    }
    
    /**
     * 注意点2：异常会自动释放锁
     */
    public synchronized void method2() {
        try {
            // 业务逻辑
            throw new RuntimeException("异常");
        } catch (Exception e) {
            // 锁已自动释放
        }
    }
    
    /**
     * 注意点3：避免锁粒度过大
     */
    public void method3() {
        // 不需要同步的代码
        synchronized (this) {
            // 需要同步的临界区
        }
        // 不需要同步的代码
    }
}
```

***

## 六、使用场景与最佳实践

### 6.1 适用场景

| 场景        | 说明        | 推荐锁类型          |
| --------- | --------- | -------------- |
| **单线程环境** | 无竞争       | 偏向锁（默认开启）      |
| **低并发**   | 多线程交替执行   | 轻量级锁           |
| **高并发**   | 频繁竞争      | 重量级锁           |
| **方法级同步** | 整个方法需要同步  | synchronized方法 |
| **代码块同步** | 仅部分代码需要同步 | synchronized块  |

### 6.2 最佳实践

```mermaid
flowchart TD
    A[使用synchronized] --> B{锁粒度过大?}
    B -->|是| C[缩小同步范围]
    B -->|否| D{锁对象是否正确?}
    D -->|否| E[选择合适的锁对象]
    D -->|是| F{是否需要细粒度控制?}
    F -->|是| G[考虑使用ReentrantLock]
    F -->|否| H[使用synchronized]
```

### 6.3 使用建议

| 原则          | 说明                                      |
| ----------- | --------------------------------------- |
| **减小锁粒度**   | 只同步必要的代码块，避免锁住整个方法                      |
| **避免锁嵌套**   | 减少死锁风险                                  |
| **使用专用锁对象** | 避免使用this或Class作为锁对象，提高可维护性              |
| **注意锁的可见性** | 锁对象应为final，避免被意外修改                      |
| **考虑并发工具类** | 对于复杂场景，考虑使用`java.util.concurrent`包中的工具类 |

***

## 七、总结

Synchronized是Java并发编程的基础，其锁升级机制是性能优化的关键：

| 核心要点          | 说明                     |
| ------------- | ---------------------- |
| **锁升级**       | 无锁→偏向锁→轻量级锁→重量级锁（单向升级） |
| **Mark Word** | 存储锁状态信息，是锁升级的核心数据结构    |
| **偏向锁**       | 适用于单线程重复获取锁的场景         |
| **轻量级锁**      | 使用CAS+自旋实现，避免系统调用      |
| **重量级锁**      | 使用操作系统互斥量，线程进入阻塞状态     |
| **自动释放**      | 方法执行完毕或抛出异常时自动释放锁      |

