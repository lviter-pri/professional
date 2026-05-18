# HashMap 容量设置与扩容机制详解（JDK 1.8）

---

## 一、概述

HashMap 的性能与其容量设置和扩容机制密切相关。理解这些底层原理，能够帮助我们在实际开发中做出更优的设计决策，避免性能隐患。

**核心要点：**

| 概念 | 说明 |
|------|------|
| **容量（Capacity）** | HashMap 底层数组的大小，默认为 16 |
| **负载因子（Load Factor）** | 触发扩容的阈值比例，默认为 0.75 |
| **阈值（Threshold）** | `capacity × loadFactor`，超过此值触发扩容 |
| **2 的幂次方** | 容量必须是 2 的幂次方，用于高效的位运算 |

---

## 二、容量设计原理：为什么必须是 2 的幂次方？

### 2.1 位运算 vs 取模运算

HashMap 通过哈希算法确定元素在数组中的位置，核心公式为：

```
index = hash(key) % capacity
```

当 `capacity` 是 2 的幂次方时：

```
capacity = 2^n
capacity - 1 = 2^n - 1 = 二进制全 1（如 15 = 1111）
```

此时，取模运算可以转换为位与运算：

```
index = hash(key) & (capacity - 1)
```

**性能对比：**

| 运算类型 | 原理 | 性能 |
|----------|------|------|
| **取模运算（%）** | 除法运算，CPU 开销大 | 较慢 |
| **位与运算（&）** | 直接操作二进制位 | 极快 |

> **结论**：位运算比取模运算效率高一个数量级以上。

### 2.2 散列均匀性分析

当 `capacity - 1` 的二进制全为 1 时，`hash & (capacity - 1)` 能充分利用 hash 值的所有位：

```
示例：capacity = 16 (10000), capacity - 1 = 15 (01111)

hash = 10101100 (172)
      & 00001111 (15)
      ----------
        00001100 (12)  → index = 12

hash = 11001010 (202)
      & 00001111 (15)
      ----------
        00001010 (10)  → index = 10
```

如果 `capacity` 不是 2 的幂次方：

```
capacity = 10 (1010), capacity - 1 = 9 (1001)

hash = 10101100 (172)
      & 00001001 (9)
      ----------
        00001000 (8)  → index = 8

hash = 10101010 (170)
      & 00001001 (9)
      ----------
        00001000 (8)  → index = 8  ← 冲突！
```

**结论**：非 2 的幂次方会导致某些桶位永远无法被命中，增加 Hash 冲突概率。

---

## 三、tableSizeFor 方法：向上取整到 2 的幂次方

### 3.1 方法作用

`tableSizeFor` 方法的作用是：**给定一个容量请求 `cap`，返回大于等于 `cap` 的最小 2 的幂次方**。

### 3.2 源码解析（JDK 1.8）

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;      // 步骤1：防止 cap 本身就是 2 的幂次方
    n |= n >>> 1;         // 步骤2：把最高位的 1 向右扩散 1 位
    n |= n >>> 2;         // 步骤3：继续扩散 2 位
    n |= n >>> 4;         // 步骤4：继续扩散 4 位
    n |= n >>> 8;         // 步骤5：继续扩散 8 位
    n |= n >>> 16;        // 步骤6：继续扩散 16 位
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### 3.3 位运算详解

**示例：cap = 10（二进制：0000 1010）**

```
步骤1：n = cap - 1 = 9 → 0000 1001

步骤2：n |= n >>> 1
       n = 0000 1001
       n >>> 1 = 0000 0100
       结果：0000 1101

步骤3：n |= n >>> 2
       n = 0000 1101
       n >>> 2 = 0000 0011
       结果：0000 1111

步骤4：n |= n >>> 4 → 0000 1111（无变化）
步骤5：n |= n >>> 8 → 0000 1111（无变化）
步骤6：n |= n >>> 16 → 0000 1111（无变化）

最终：n + 1 = 16 → 返回 16
```

**核心逻辑**：通过位移操作将最高位的 1 扩散到后面所有位，最后 `+1` 得到最小的 2 的幂次方。

### 3.4 边界情况处理

```java
// cap = 0 时
n = 0 - 1 = -1  // 二进制全为 1（补码表示）
// 位移操作后仍为全 1
return (n < 0) ? 1 : ...  // 返回 1

// cap = 1 时
n = 1 - 1 = 0
// 位移操作后仍为 0
return n + 1 = 1

// cap = 16（本身就是 2 的幂次方）
n = 16 - 1 = 15 → 0000 1111
// 位移操作后仍为 15
return 15 + 1 = 16  // 正确返回 16，没有翻倍
```

---

## 四、扩容机制详解：resize 方法

### 4.1 扩容触发条件

当 HashMap 满足以下条件时触发扩容：

```java
size > threshold  // threshold = capacity × loadFactor
```

**默认参数：**
- 初始容量 = 16
- 负载因子 = 0.75
- 初始阈值 = 16 × 0.75 = 12

### 4.2 resize 方法核心流程

```
┌─────────────────────────────────────────────────────────────┐
│                    resize() 执行流程                         │
├─────────────────────────────────────────────────────────────┤
│  第一阶段：计算新容量和新阈值                                │
│     ├─> 旧容量 >= MAXIMUM_CAPACITY (2^30)?               │
│     │     └─> 是：阈值设为 Integer.MAX_VALUE，不再扩容      │
│     │                                                      │
│     └─> 否：新容量 = 旧容量 × 2                            │
│           新阈值 = 旧阈值 × 2                               │
│                                                             │
│  第二阶段：创建新数组                                        │
│     └─> Node<K,V>[] newTab = new Node[newCap]             │
│                                                             │
│  第三阶段：数据迁移                                          │
│     └─> 遍历旧数组每个桶                                   │
│           ├─> 桶为空：跳过                                  │
│           ├─> 桶只有一个节点：直接计算新位置               │
│           ├─> 桶是红黑树：调用 split() 方法拆分            │
│           └─> 桶是链表：按高低位拆分为两个链表后迁移       │
│                                                             │
│  第四阶段：更新引用                                          │
│     └─> table = newTab                                    │
│           threshold = newThr                               │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Java 8 扩容优化：无需重算 Hash

这是 Java 8 最重要的优化之一！

**数学原理：**

当容量从 `oldCap` 翻倍到 `newCap = oldCap × 2` 时：
- 新掩码 = `newCap - 1 = (oldCap × 2) - 1`
- 相比旧掩码 `oldCap - 1`，新掩码多了一位最高位的 1

**位置判断：**

元素的新位置只有两种可能：
1. **原位置**：`(e.hash & oldCap) == 0`
2. **原位置 + 旧容量**：`(e.hash & oldCap) != 0`

```java
// 示例：oldCap = 16, newCap = 32
// oldCap = 10000 (二进制)
// newCap - 1 = 11111

// hash = 01010100
// (hash & oldCap) = 01010100 & 010000 = 0 → 位置不变

// hash = 11010100
// (hash & oldCap) = 11010100 & 010000 = 10000 ≠ 0 → 位置 = 原位置 + 16
```

### 4.4 链表拆分源码解析

```java
// Java 8 链表迁移核心代码
Node<K,V> loHead = null, loTail = null;  // 低位链表（原位置）
Node<K,V> hiHead = null, hiTail = null;  // 高位链表（原位置 + oldCap）
Node<K,V> next;

do {
    next = e.next;
    // 判断元素应该放在低位还是高位
    if ((e.hash & oldCap) == 0) {
        // 低位
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    } else {
        // 高位
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);

// 把低位链表放入原位置
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}

// 把高位链表放入新位置
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

**优点：**
- 无需重新计算 hash 值
- 使用尾插法保持元素顺序
- 避免了 Java 7 中的环形链表问题

---

## 五、负载因子的作用与选择

### 5.1 负载因子的作用

负载因子（Load Factor）决定了 HashMap 何时扩容：

```
threshold = capacity × loadFactor
```

| 负载因子值 | 特点 |
|-----------|------|
| **较小（如 0.5）** | 扩容频繁，内存利用率低，但 Hash 冲突少，查询效率高 |
| **较大（如 0.9）** | 扩容少，内存利用率高，但 Hash 冲突多，查询效率低 |
| **默认（0.75）** | 平衡内存与性能的折衷选择 |

### 5.2 选择建议

| 场景 | 推荐负载因子 | 原因 |
|------|-------------|------|
| **CPU 密集型** | 0.75（默认） | 平衡性能与内存 |
| **内存紧张** | 0.9 | 提高内存利用率 |
| **读多写少** | 0.75 或更低 | 减少冲突，提高读性能 |
| **写多读少** | 0.75 或更高 | 减少扩容次数 |

---

## 六、容量预估最佳实践

### 6.1 初始容量计算公式

如果已知要存储的元素数量 `n`，建议按以下公式设置初始容量：

```java
initialCapacity = (int) (n / loadFactor) + 1
```

**示例：**

```java
// 计划存储 100 个元素，负载因子使用默认 0.75
int n = 100;
int initialCapacity = (int) (100 / 0.75) + 1;  // = 134

// 由于 tableSizeFor 会向上取整到 2 的幂次方
// 实际容量 = tableSizeFor(134) = 256
```

### 6.2 常见误区

**误区 1：直接使用元素数量作为初始容量**

```java
// 错误
new HashMap<>(100);  // tableSizeFor(100) = 128
// threshold = 128 × 0.75 = 96
// 当第 97 个元素插入时触发扩容！

// 正确
new HashMap<>(134);  // tableSizeFor(134) = 256
// threshold = 256 × 0.75 = 192
// 存储 100 个元素不会触发扩容
```

**误区 2：使用 2 的幂次方作为初始容量**

```java
// 虽然结果正确，但可读性差
new HashMap<>(256);

// 推荐：明确表达意图
new HashMap<>(134);  // 我需要存储约 100 个元素
```

**误区 3：忽视负载因子的影响**

```java
// 如果修改了负载因子，计算公式也要相应调整
int loadFactor = 0.9;
int initialCapacity = (int) (100 / 0.9) + 1;  // = 112
```

### 6.3 使用 Guava 工具类

```java
import com.google.common.collect.Maps;

// 推荐：让工具类帮你计算
Map<String, Object> map = Maps.newHashMapWithExpectedSize(100);
```

Guava 的 `newHashMapWithExpectedSize` 内部会自动计算合适的初始容量：

```java
// Guava 源码简化版
public static <K, V> HashMap<K, V> newHashMapWithExpectedSize(int expectedSize) {
    return new HashMap<>(capacity(expectedSize));
}

static int capacity(int expectedSize) {
    if (expectedSize < 3) {
        return expectedSize + 1;
    }
    if (expectedSize < Ints.MAX_POWER_OF_TWO) {
        // 当 expectedSize 较小时，使用 50% 的负载因子
        return (int) ((float) expectedSize / 0.75F + 1.0F);
    }
    return Integer.MAX_VALUE;
}
```

---

## 七、完整示例

### 7.1 容量预估示例

```java
public class HashMapCapacityDemo {
    public static void main(String[] args) {
        // 计划存储 1000 个元素
        int expectedSize = 1000;
        
        // 计算初始容量
        int loadFactor = 0.75;
        int initialCapacity = (int) (expectedSize / loadFactor) + 1;
        // initialCapacity = 1334
        
        // 创建 HashMap
        Map<String, Object> map = new HashMap<>(initialCapacity);
        // 实际容量 = tableSizeFor(1334) = 2048
        // threshold = 2048 × 0.75 = 1536
        
        // 存储 1000 个元素不会触发扩容
        for (int i = 0; i < 1000; i++) {
            map.put("key" + i, "value" + i);
        }
    }
}
```

### 7.2 扩容流程示例

```java
public class HashMapResizeDemo {
    public static void main(String[] args) {
        // 创建一个容量为 4 的 HashMap
        Map<String, Integer> map = new HashMap<>(4);
        // threshold = 4 × 0.75 = 3
        
        // 插入 3 个元素，不会扩容
        map.put("A", 1);
        map.put("B", 2);
        map.put("C", 3);
        
        // 插入第 4 个元素，触发扩容
        // newCap = 8, newThr = 6
        map.put("D", 4);
        
        // 此时容量为 8，阈值为 6
    }
}
```

---

## 八、总结

| 核心要点 | 说明 |
|----------|------|
| **容量设计** | 必须是 2 的幂次方，用于高效的位运算 `hash & (n - 1)` |
| **tableSizeFor** | 将任意容量请求向上取整到最小的 2 的幂次方 |
| **扩容触发** | `size > threshold`（threshold = capacity × loadFactor） |
| **Java 8 优化** | 扩容时无需重算 hash，只需判断 `(hash & oldCap) == 0` |
| **初始容量计算** | `(int) (expectedSize / loadFactor) + 1` |
| **负载因子选择** | 默认 0.75 是性能与内存的平衡点 |

**最佳实践口诀：**
> 容量预估要牢记，除以因子再加一；
> 2 的幂方不用管，tableSizeFor 会处理；
> 默认因子点七五，读多写少可降低；
> 写多读少可提高，内存紧张零点九。