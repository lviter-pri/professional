## ConcurrentHashMap

Q:KEY/VALUE都不允许为空，为什么？

A:为了消除两种语义。在单线程 HashMap 中，可以用 `containsKey` 判断；但在并发下，如果 `get(key)` 返回 `null`，无法确定是"值本身就是 null"还是"在判断的过程中值被删除了"

## 一、 ConcurrentHashMap：从“分段锁”到“行级锁”的进化

在 Java 8 中，CHM 抛弃了 Java 7 那套厚重的 `Segment`（分段锁）设计，转而采用了更细粒度的控制。

### 1. 核心原理：CAS + synchronized

Java 8 的 CHM 看起来和 HashMap 结构相似（数组+链表+红黑树），但在并发控制上非常精妙：

- **比较并插入CAS (Compare And Swap)**：在插入新节点时，如果桶（Bucket）为空，直接通过 CAS 原子操作放入，**无锁执行**，效率极高。

- **synchronized**：如果桶不为空（即发生了碰撞），则只锁定**当前的头节点（或树的根节点）**。

  > **经验必谈点**：为什么不用 `ReentrantLock`？因为 Java 6 之后 `synchronized` 引入了偏向锁、轻量级锁和锁消除优化（锁升级）。在大量细粒度锁的场景下，`synchronized` 的内存开销比 `ReentrantLock` 更小。

### 2. 扩容

这是 CHM 最牛的地方。当一个线程发现 Map 正在扩容时，它不会坐视不管，而是会**主动参与协助扩容**。

- 它利用 `ForwardingNode`（转发节点）来标记已经处理过的桶。
- 多个线程可以同时搬运不同的桶，极大地缩短了扩容导致的停顿时间。

### 3. 读取的“弱一致性”

CHM 的 `get` 方法是**完全无锁**的。它通过 `volatile` 关键字保证了节点的 `val` 和 `next` 指针的可见性。

> **提醒**：既然无锁，那读到的是最新的吗？答案是：CHM 保证的是**弱一致性**，即它不保证能立刻读到其他线程刚写入的数据，但保证不会读到乱码或破坏结构的数据。



## 二、核心分析

**Java 8 对比java 7：** 它废弃了 Segment，直接在 **Node（桶的头节点）** 上加锁。

- **物理结构**：依然是数组 + 链表 + 红黑树。
- **并发策略**：
  - 如果桶是空的：使用 **CAS-自旋** 插入，完全无锁。
  - 如果桶不为空：使用 **synchronized** 锁住头节点。

> 为什么 Java 8 又捡回了 `synchronized`？ 因为 JVM 团队对 `synchronized` 做了大量底层优化（偏向锁、锁消除、锁粗化）。相比 `ReentrantLock`，它不需要额外的 AQS 节点内存开销，且在锁竞争不激烈时几乎零损耗。

### 1. 计数器之谜：size() 是怎么统计出来的？

在多线程下，`HashMap` 的 `size++` 是不准的。那 CHM 怎么办？ 它没有简单地用 `AtomicInteger`，因为在极高并发下，所有线程去竞争一个原子变量会导致大量的 CPU 自旋。

**CHM 的方案：`CounterCell`（借鉴了 LongAdder）**

- 它维护了一个 `baseCount` 和一个 `CounterCell` 数组。
- 线程先尝试 CAS 更新 `baseCount`。
- 如果失败（说明竞争激烈），它会随机选一个 `CounterCell` 进行 CAS 更新。
- **最后 `size()` 的结果 = `baseCount` + `CounterCell` 数组所有值的累加。**

> 这是一种 分散压力 的设计思想，用空间换取了高并发下的计数性能。

### 2. 读取数据：为什么可以“完全无锁”？

**既然写操作加了锁，为什么 get 操作不需要加锁也能保证线程安全？**

答案藏在两个关键点：

1. **Node 的 val 和 next 都用了 `volatile` 修饰**：保证了多线程之间的内存可见性。一旦某个线程修改了值，其他线程能立刻看到。
2. **数组引用也具有可见性**：CHM 内部使用了 `Unsafe` 类的 `getObjectVolatile` 来确保获取最新的桶头节点。