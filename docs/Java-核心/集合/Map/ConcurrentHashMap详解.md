# ConcurrentHashMap 详解

---

## 一、概述与核心特性

### 1.1 为什么需要 ConcurrentHashMap？

在多线程环境下，普通 HashMap 存在以下问题：

| 问题 | 说明 |
|------|------|
| 线程不安全 | 并发修改可能导致数据丢失、环链表、死循环 |
| 并发读性能差 | 单一锁机制导致所有线程排队 |

ConcurrentHashMap（简称 CHM）应运而生，提供**线程安全且高性能**的 Key-Value 存储。

### 1.2 核心特性一览

| 特性 | 说明 |
|------|------|
| **线程安全** | 通过 CAS + synchronized 实现高效并发控制 |
| **高性能** | 读操作完全无锁，写操作锁粒度细化到单个桶 |
| **Key/Value 不允许为 null** | 消除并发下的二义性 |
| **弱一致性** | get() 可能读到旧值，但不破坏数据结构 |
| **扩容优化** | 支持多线程协助扩容，减少停顿时间 |

### 1.3 核心参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| 初始容量 | 16 | 数组大小 |
| 负载因子 | 0.75 | 扩容阈值 = 容量 × 负载因子 |
| 并发级别（已废弃） | 16 | Java 7 的分段锁数量，Java 8 已废弃 |
| 树化阈值 | 8 | 链表长度超过 8 时转为红黑树 |
| 退化阈值 | 6 | 树节点数 ≤ 6 时退化为链表 |
| 最小树化容量 | 64 | 数组长度 < 64 时优先扩容而非树化 |

---

## 二、数据结构解析

### 2.1 演进历程

| 版本 | 数据结构 | 并发控制 |
|------|----------|----------|
| Java 7 | 数组 + Segment + 链表 | Segment 继承 ReentrantLock，分段锁 |
| Java 8 | 数组 + 链表/红黑树 | CAS + synchronized（锁住头节点） |

### 2.2 Java 8 的物理结构

```
┌─────────────────────────────────────────────────────────────┐
│                     ConcurrentHashMap                        │
├─────────────────────────────────────────────────────────────┤
│  Node<K,V>[] table                                          │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐               │
│  │ V   │     │ V   │     │     │ V   │     │               │
│  │head │     │head │     │     │head │     │  ...          │
│  └──┬──┴─────┴──┬──┴─────┴──┬──┴─────┴──┬──┴─────┐         │
│     │           │           │           │        │         │
│     ▼           ▼           ▼           ▼        ▼         │
│   Node        Node       TreeNode    Node     TreeNode      │
│   (链表)      (链表)      (红黑树)    (链表)    (红黑树)     │
└─────────────────────────────────────────────────────────────┘
```

**核心结构**：
- **Node**：基础节点，包含 `hash`、`key`、`value`、`next`
- **TreeNode**：红黑树节点，继承自 Node
- **TreeBin**：包装 TreeNode 的容器，管理红黑树的读写锁
- **ForwardingNode**：扩容时的转发节点，标记已迁移的桶

---

## 三、核心原理：CAS + synchronized

### 3.1 为什么 Java 8 放弃了分段锁？

Java 7 的 Segment 分段锁存在以下问题：

- **锁粒度粗**：每个 Segment 包含若干桶，锁竞争激烈时效率下降
- **内存开销大**：每个 Segment 都是一个独立对象
- **统计 size 复杂**：需要累加所有 Segment 的计数

Java 8 采用**锁细化**策略，直接在单个桶的头节点上加锁。

### 3.2 CAS：无锁插入

当桶为空时，使用 `Unsafe.compareAndSwapObject` 进行无锁插入：

```java
// 伪代码示意
if (tab[i] == null) {
    // CAS 操作：期望为 null，设置为新节点
    cas = U.compareAndSwapObject(tab, i, null, new Node(hash, key, value));
}
```

**优点**：桶为空时无需加锁，并发性能极高

### 3.3 synchronized：桶级锁

当桶不为空（发生 Hash 冲突）时，使用 `synchronized` 锁住头节点：

```java
synchronized (tabAt(tab, i)) {
    // 对链表或红黑树进行操作
    if (tabAt(tab, i) == f) {
        // 处理链表
    } else if (f instanceof TreeBin) {
        // 处理红黑树
    }
}
```

### 3.4 为什么选择 synchronized 而不是 ReentrantLock？

> **面试高频点**：Java 8 为什么重新选择 synchronized？

| 对比维度 | synchronized | ReentrantLock |
|----------|--------------|---------------|
| 内存开销 | 只需对象头 | 需要额外的 AQS 节点 |
| 锁升级 | 支持偏向锁、轻量级锁、重量级锁 | 不可升级 |
| 低竞争场景 | 偏向锁几乎零开销 | 需要额外同步 |
| JVM 优化 | JIT 编译器持续优化 | 依赖 JDK 版本 |

**结论**：在大量细粒度锁的高并发场景，`synchronized` 的综合性能更优。

---

## 四、核心操作详解

### 4.1 put 操作流程

```
┌─────────────────────────────────────────────────────────────┐
│                      put() 执行流程                          │
├─────────────────────────────────────────────────────────────┤
│  1. 计算 key 的 hash 值                                     │
│     └─> spread()：高16位与低16位异或，减少碰撞              │
│                                                             │
│  2. 检查数组是否初始化                                       │
│     └─> 未初始化：调用 initTable() 进行懒初始化             │
│                                                             │
│  3. 定位桶位置                                               │
│     └─> i = (n - 1) & hash                                  │
│                                                             │
│  4. 插入逻辑                                                │
│     ├─> 桶为空：CAS 无锁插入                                │
│     ├─> 桶不为空（链表）：synchronized 锁头节点后插入       │
│     ├─> 桶不为空（红黑树）：synchronized 锁 TreeBin 后插入  │
│                                                             │
│  5. 检查是否需要树化                                        │
│     └─> 链表长度 > 8 且数组长度 ≥ 64 → 转为红黑树          │
│                                                             │
│  6. 增加计数                                                │
│     └─> 调用 addCount() 更新 size                           │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 get 操作：无锁读取

get 操作完全无锁，依靠 `volatile` 保证可见性：

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 检查头节点
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        // 树节点
        else if (eh < 0) {
            return (p = e.find(h, key)) != null ? p.val : null;
        }
        // 链表遍历
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

**关键点**：
- `tabAt()` 使用 `Unsafe.getObjectVolatile` 保证读取最新值
- Node 的 `val` 和 `next` 都用 `volatile` 修饰

### 4.3 remove 操作

remove 操作与 put 类似，在 synchronized 保护下：

1. 定位桶和目标节点
2. 删除节点（链表：跳过该节点；红黑树：平衡调整）
3. 更新计数

---

## 五、扩容机制

### 5.1 ForwardingNode：迁移标记

CHM 使用 `ForwardingNode` 标记已迁移的桶：

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
    // find 方法会转发到新数组
}
```

**特点**：
- `hash = MOVED (-1)`，标识该桶已迁移
- 包含指向新数组的引用
- 其他线程访问时自动协助扩容

### 5.2 多线程协助扩容

这是 CHM 相比 HashMap 最大的优势：**并发扩容**。

**工作原理**：
1. 线程发现 map 正在扩容（检测到 ForwardingNode）
2. 该线程**主动参与**扩容，而不是等待
3. 每个线程负责一段桶的迁移
4. 迁移完成后继续处理其他任务

```
线程A: ──> [桶0-7迁移] ──> [桶24-31迁移] ──> 继续工作
线程B: ──> [桶8-15迁移] ──> [桶32-39迁移] ──> 继续工作
线程C: ──> [桶16-23迁移] ──> [桶40-47迁移] ──> 继续工作
```

**优势**：
- 扩容期间仍有线程可以执行 put/get 操作
- 多线程并行迁移，极大缩短扩容停顿时间

### 5.3 扩容流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    帮助扩容（helpTransfer)                   │
├─────────────────────────────────────────────────────────────┤
│  1. 检查是否需要扩容                                        │
│     └─> size > threshold && table 未被占用 && nextTable 空  │
│                                                             │
│  2. 计算 stride（每个线程处理的桶数）                        │
│     └─> 最小为 16，根据 CPU 核数动态调整                     │
│                                                             │
│  3. 初始化 nextTable（新数组）                               │
│     └─> 容量 = 旧容量 × 2                                    │
│                                                             │
│  4. 分配迁移任务区间                                         │
│     └─> CAS 设置 transferIndex                              │
│                                                             │
│  5. 循环处理自己区间的桶                                     │
│     └─> 创建 ForwardingNode 标记                            │
│     └─> 迁移链表或红黑树到 nextTable                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 六、计数器设计：CounterCell + LongAdder 思想

### 6.1 为什么不用 AtomicInteger？

高并发下，所有线程竞争单一原子变量会导致大量 CAS 自旋：

```java
// HashMap 的做法（有线程安全问题）
size++;

// 如果用 AtomicInteger
AtomicInteger size;  // 线程1: cas成功
                     // 线程2: cas失败，自旋重试
                     // 线程3: cas失败，自旋重试
                     // ... 大量线程竞争
```

### 6.2 CounterCell 分散计数

CHM 借鉴了 LongAdder 的设计思想：

```
┌─────────────────────────────────────────────────────────────┐
│                    分散计数设计                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   baseCount = 55                                           │
│                                                             │
│   CounterCell[] cells                                      │
│   ┌─────┬─────┬─────┬─────┐                                │
│   │ 12  │  8  │  0  │  3  │                                │
│   └─────┴─────┴─────┴─────┘                                │
│                                                             │
│   size = baseCount + sum(cells) = 55 + 12 + 8 + 0 + 3 = 78 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**工作原理**：

1. **先尝试更新 baseCount**：`U.compareAndSwapLong(this, BASECOUNT, b, b + x)`
2. **失败则尝试更新 CounterCell**：随机选择一个 Cell 进行 CAS 更新
3. **size() 累加**：`baseCount + Σ CounterCell[i].value`

### 6.3 LongAdder 的关系

CounterCell 数组的思想与 LongAdder 完全一致：

| 组件 | LongAdder | ConcurrentHashMap |
|------|-----------|-------------------|
| 基础值 | base | baseCount |
| 分散数组 | cells | CounterCell[] |
| 累加方式 | 空间换时间 | 空间换时间 |

---

## 七、HashMap vs HashTable vs ConcurrentHashMap

### 7.1 核心对比表

| 对比维度 | HashMap | HashTable | ConcurrentHashMap |
|----------|---------|-----------|-------------------|
| **线程安全性** | 不安全 | 安全（synchronized） | 安全（CAS + synchronized） |
| **性能** | 单线程最高 | 所有操作加锁，并发性能差 | 锁粒度细化，高并发性能好 |
| **Null 支持** | Key/Value 都可为 null | Key/Value 都不允许 null | Key/Value 都不允许 null |
| **底层结构** | 数组 + 链表/红黑树 | 数组 + 链表 | 数组 + 链表/红黑树 |
| **并发控制** | 无 | 锁整个 map 对象 | 锁单个桶（Java 8） |
| **迭代器** | 快速失败（Fail-Fast） | 安全迭代 | 弱一致性迭代 |
| **扩容方式** | 单线程扩容 | 单线程扩容 | 多线程协助扩容 |
| **初始化** | 懒初始化 | 构造函数初始化 | 懒初始化 |

### 7.2 为什么 HashTable 不允许 null？

```java
// HashTable 的 get + containsKey 语义
if (value == null) {
    // 无法区分：是值本身为 null，还是键不存在
}
```

CHM 同理：**消除二义性**，让并发场景下的判断更清晰。

### 7.3 适用场景分析

| 场景 | 推荐选择 | 原因 |
|------|----------|------|
| **单线程** | HashMap | 无锁开销，性能最优 |
| **低并发** | HashMap + Collections.synchronizedMap | 简单包装够用 |
| **高并发** | ConcurrentHashMap | 锁细化，性能最优 |
| **需要全表锁** | HashTable | 简单粗暴，兼容性考虑 |

### 7.4 选型建议

```
                    ┌─────────────────┐
                    │  需要线程安全？  │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │ Yes                          │ No
              ▼                              ▼
    ┌─────────────────┐            ┌─────────────────┐
    │  并发度高？      │            │   HashMap       │
    └────────┬────────┘            └─────────────────┘
             │
    ┌────────┴────────┐
    │ Yes             │ No
    ▼                 ▼
┌─────────────┐  ┌─────────────┐
│ Concurrent  │  │ HashTable   │
│ HashMap     │  │ (不推荐)    │
└─────────────┘  └─────────────┘
```

---

## 八、常见面试问题

### Q1: ConcurrentHashMap 的 get 操作是否需要加锁？

**答案**：**不需要**。

**原因**：
1. Node 的 `val` 和 `next` 都用 `volatile` 修饰
2. 使用 `Unsafe.getObjectVolatile` 读取最新值
3. 保证**内存可见性**，无需加锁

**注意**：虽然无锁，但不保证能立刻读到其他线程刚写入的值（弱一致性），不过**不会读到脏数据或破坏结构**。

---

### Q2: put 操作如果发现桶为空，会发生什么？

**答案**：分两种情况

1. **CAS 成功**：直接插入，流程结束
2. **CAS 失败**：说明其他线程已经插入了，触发自旋重试

```java
// 简化逻辑
do {
    if (tabAt(tab, i) == null) {
        // 尝试 CAS 插入
        if (U.compareAndSwapObject(tab, i, null, f)) {
            break;  // 成功插入
        }
    }
} while (true);  // 继续重试
```

---

### Q3: ConcurrentHashMap 的 size() 为什么不是精确的？

**答案**：为了高性能，size() 是**估算值**。

**原因**：
1. 计数分散在 baseCount + CounterCell[] 中
2. 计算 size() 时只能保证**最终一致性**
3. 迭代过程中可能漏计或重复计

**替代方案**：如果需要精确计数，使用 `mappingCount()` 方法（返回 long 类型）。

---

### Q4: ConcurrentHashMap 1.7 和 1.8 有什么区别？

| 方面 | Java 7 | Java 8 |
|------|--------|--------|
| **数据结构** | Segment + HashEntry | Node + TreeNode |
| **并发控制** | ReentrantLock 分段锁 | CAS + synchronized |
| **锁粒度** | Segment 级别（多个桶） | 桶级别（单个桶） |
| **内存开销** | 每个 Segment 都是对象 | 更紧凑的结构 |
| **扩容** | 单线程扩容 | 多线程协助扩容 |
| **size 计算** | 累加 Segment.count | baseCount + CounterCell |

---

### Q5: ConcurrentHashMap 的 key 和 value 为什么不能为 null？

**答案**：**消除并发场景下的二义性**。

```java
// 场景分析
V value = map.get(key);
if (value == null) {
    // 问题：value 是 null，还是 key 根本不存在？
    // 在并发环境下，key 可能刚被删除
}
```

**HashMap** 可以用 `containsKey` 区分，但 **ConcurrentHashMap** 不提供这个方法（已废弃），所以直接禁止 null。

---

### Q6: ConcurrentHashMap 是如何实现多线程协助扩容的？

**答案**：通过 `ForwardingNode` 和 `transferIndex` 实现。

1. 扩容线程创建 `ForwardingNode` 标记已迁移的桶
2. 其他线程访问到 `ForwardingNode` 时，尝试获取 `transferIndex` 区间
3. 线程迁移自己区间的桶后继续处理业务
4. **扩容不再是阻塞操作**，而是后台协作完成

---

## 九、总结

| 核心要点 | 说明 |
|----------|------|
| **并发控制** | CAS（桶空）+ synchronized（桶非空） |
| **锁粒度** | 细化到单个桶的头节点 |
| **读操作** | 完全无锁，依靠 volatile 保证可见性 |
| **扩容** | 多线程协助，非阻塞式扩容 |
| **计数** | baseCount + CounterCell[] 分散压力 |
| **适用场景** | 高并发 Key-Value 存储 |