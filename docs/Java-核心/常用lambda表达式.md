# Lambda表达式常用示例汇总

Lambda表达式是Java 8引入的核心特性，极大简化了函数式编程。本文汇总了日常开发中常用的Lambda表达式场景。

---

## 一、集合基础操作

### 1.1 过滤（Filter）

```java
// 过滤出年龄大于18的用户
List<User> adults = users.stream()
    .filter(user -> user.getAge() > 18)
    .collect(Collectors.toList());

// 过滤非空元素
List<String> nonNullList = list.stream()
    .filter(Objects::nonNull)
    .collect(Collectors.toList());

// 多条件过滤
List<Product> filtered = products.stream()
    .filter(p -> p.getPrice() > 100 && p.getStock() > 0)
    .collect(Collectors.toList());
```

### 1.2 映射（Map）

```java
// 对象转属性列表
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());

// 类型转换
List<Integer> lengths = strings.stream()
    .map(String::length)
    .collect(Collectors.toList());

// 多级映射
List<String> roles = users.stream()
    .map(User::getProfile)
    .map(Profile::getRole)
    .collect(Collectors.toList());
```

### 1.3 排序（Sort）

```java
// 自然排序
List<String> sorted = list.stream()
    .sorted()
    .collect(Collectors.toList());

// 自定义排序（升序）
List<User> sortedByAge = users.stream()
    .sorted(Comparator.comparingInt(User::getAge))
    .collect(Collectors.toList());

// 自定义排序（降序）
List<User> sortedByAgeDesc = users.stream()
    .sorted(Comparator.comparingInt(User::getAge).reversed())
    .collect(Collectors.toList());

// 多字段排序
List<User> sorted = users.stream()
    .sorted(Comparator.comparing(User::getDepartment)
        .thenComparing(User::getAge))
    .collect(Collectors.toList());
```

### 1.4 查找（Find）

```java
// 查找第一个匹配元素
Optional<User> first = users.stream()
    .filter(u -> "admin".equals(u.getRole()))
    .findFirst();

// 查找任意匹配元素
Optional<User> any = users.stream()
    .filter(u -> u.getAge() > 30)
    .findAny();

// 判断是否存在匹配元素
boolean hasAdmin = users.stream()
    .anyMatch(u -> "admin".equals(u.getRole()));

// 判断所有元素是否满足条件
boolean allAdults = users.stream()
    .allMatch(u -> u.getAge() >= 18);

// 判断是否没有元素满足条件
boolean noChildren = users.stream()
    .noneMatch(u -> u.getAge() < 18);
```

### 1.5 去重（Distinct）

```java
// 简单去重
List<String> unique = list.stream()
    .distinct()
    .collect(Collectors.toList());

// 根据对象属性去重（使用TreeSet）
List<User> uniqueUsers = users.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(User::getId))),
        ArrayList::new
    ));

// 根据多个属性去重
List<User> uniqueByMultiple = users.stream()
    .filter(distinctByKey(u -> Arrays.asList(u.getName(), u.getEmail())))
    .collect(Collectors.toList());

// 辅助方法
private static <T> Predicate<T> distinctByKey(Function<? super T, ?> keyExtractor) {
    Set<Object> seen = ConcurrentHashMap.newKeySet();
    return t -> seen.add(keyExtractor.apply(t));
}
```

---

## 二、集合聚合操作

### 2.1 分组（Grouping）

```java
// 按字段分组
Map<String, List<User>> groupedByDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment));

// 按字段分组并统计数量
Map<String, Long> countByDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment, Collectors.counting()));

// 多级分组
Map<String, Map<String, List<User>>> grouped = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment, 
        Collectors.groupingBy(User::getRole)));

// 分组后映射
Map<String, List<String>> namesByDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment,
        Collectors.mapping(User::getName, Collectors.toList())));
```

### 2.2 聚合统计（Aggregation）

```java
// 求和
int totalAge = users.stream()
    .mapToInt(User::getAge)
    .sum();

// 求平均值
double avgAge = users.stream()
    .mapToInt(User::getAge)
    .average()
    .orElse(0);

// 求最大值
OptionalInt maxAge = users.stream()
    .mapToInt(User::getAge)
    .max();

// 求最小值
Optional<User> youngest = users.stream()
    .min(Comparator.comparingInt(User::getAge));

// 统计信息
IntSummaryStatistics stats = users.stream()
    .mapToInt(User::getAge)
    .summaryStatistics();
System.out.println("Count: " + stats.getCount());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Avg: " + stats.getAverage());
```

### 2.3 归约（Reduce）

```java
// 字符串拼接
String allNames = users.stream()
    .map(User::getName)
    .reduce("", (a, b) -> a + ", " + b);

// 带初始值的归约
int total = numbers.stream()
    .reduce(0, Integer::sum);

// 自定义归约
Optional<User> richest = users.stream()
    .reduce((u1, u2) -> u1.getBalance() > u2.getBalance() ? u1 : u2);
```

### 2.4 分区（Partitioning）

```java
// 按条件分区（只有两个分区：true/false）
Map<Boolean, List<User>> partitioned = users.stream()
    .collect(Collectors.partitioningBy(u -> u.getAge() >= 18));

List<User> adults = partitioned.get(true);
List<User> children = partitioned.get(false);
```

---

## 三、Optional操作

### 3.1 空值处理

```java
// 传统空值检查
String name = null;
if (user != null) {
    Profile profile = user.getProfile();
    if (profile != null) {
        name = profile.getName();
    }
}

// 使用Optional简化
String name = Optional.ofNullable(user)
    .map(User::getProfile)
    .map(Profile::getName)
    .orElse("Unknown");

// orElse与orElseGet的区别
// orElse总是执行，orElseGet仅在值为空时执行
String result = optional.orElse(getDefaultValue());  // 总是调用getDefaultValue()
String result = optional.orElseGet(() -> getDefaultValue());  // 仅在空时调用

// orElseThrow
String required = optional.orElseThrow(() -> new IllegalArgumentException("值不能为空"));
```

### 3.2 条件操作

```java
// ifPresent
optional.ifPresent(value -> System.out.println("值存在: " + value));

// ifPresentOrElse（Java 9+）
optional.ifPresentOrElse(
    value -> System.out.println("值存在: " + value),
    () -> System.out.println("值不存在")
);

// filter
Optional<User> admin = optionalUser.filter(u -> "admin".equals(u.getRole()));

// flatMap
Optional<String> role = optionalUser
    .flatMap(u -> Optional.ofNullable(u.getProfile()))
    .flatMap(p -> Optional.ofNullable(p.getRole()));
```

---

## 四、函数式接口

### 4.1 Consumer（消费型接口）

```java
// 基本使用
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello");

// 链式调用
Consumer<String> first = s -> System.out.print("First: ");
Consumer<String> second = s -> System.out.println(s);
first.andThen(second).accept("Lambda");

// 实际应用
users.forEach(user -> System.out.println(user.getName()));
```

### 4.2 Supplier（供给型接口）

```java
// 基本使用
Supplier<LocalDateTime> now = LocalDateTime::now;
LocalDateTime currentTime = now.get();

// 延迟初始化
Supplier<HeavyObject> lazyObject = () -> createHeavyObject();
HeavyObject obj = lazyObject.get();  // 仅在调用时创建

// 配合Optional
Optional<String> optional = Optional.ofNullable(null);
String value = optional.orElseGet(() -> "default");
```

### 4.3 Predicate（断言型接口）

```java
// 基本使用
Predicate<Integer> isEven = n -> n % 2 == 0;
boolean result = isEven.test(4);

// 组合使用
Predicate<String> isNotEmpty = s -> s != null && !s.isEmpty();
Predicate<String> isLongerThan5 = s -> s.length() > 5;

Predicate<String> condition = isNotEmpty.and(isLongerThan5);
boolean valid = condition.test("Hello World");

// 否定
Predicate<String> isEmpty = isNotEmpty.negate();
```

### 4.4 Function（函数型接口）

```java
// 基本使用
Function<String, Integer> length = String::length;
int len = length.apply("Hello");

// 链式调用
Function<String, String> toUpperCase = String::toUpperCase;
Function<String, Integer> lengthOfUpper = toUpperCase.andThen(String::length);
int result = lengthOfUpper.apply("hello");

// 组合调用
Function<Integer, String> intToString = Object::toString;
Function<String, Integer> stringToInt = Integer::parseInt;
Function<Integer, Integer> twice = intToString.andThen(stringToInt).andThen(n -> n * 2);

// BiFunction
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
int sum = add.apply(3, 5);
```

---

## 五、并发编程

### 5.1 CompletableFuture

```java
// 异步执行
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 耗时操作
    return fetchData();
});

// 链式处理
future.thenApply(data -> processData(data))
    .thenAccept(result -> System.out.println("结果: " + result))
    .exceptionally(e -> {
        System.err.println("错误: " + e.getMessage());
        return null;
    });

// 组合多个异步任务
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> fetchData1());
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> fetchData2());

CompletableFuture<String> combined = future1.thenCombine(future2, (data1, data2) -> {
    return data1 + " " + data2;
});

// 等待所有任务完成
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2);
all.join();

// 任意任务完成
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);
```

---

## 六、Map操作

### 6.1 遍历Map

```java
// 遍历entry
map.forEach((key, value) -> System.out.println(key + ": " + value));

// 遍历key
map.keySet().forEach(key -> System.out.println(key));

// 遍历value
map.values().forEach(value -> System.out.println(value));
```

### 6.2 转换Map

```java
// List转Map（注意重复key）
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// 处理重复key
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(
        User::getId, 
        Function.identity(), 
        (existing, replacement) -> existing  // 保留第一个
    ));

// Map转换
Map<String, Integer> newMap = oldMap.entrySet().stream()
    .filter(e -> e.getValue() > 10)
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        e -> e.getValue() * 2
    ));
```

---

## 七、集合转换

### 7.1 List与数组互转

```java
// List转数组
List<String> list = Arrays.asList("a", "b", "c");
String[] array = list.toArray(new String[0]);

// 数组转List
String[] array = {"a", "b", "c"};
List<String> list = Arrays.stream(array).collect(Collectors.toList());
```

### 7.2 集合类型转换

```java
// List转Set
Set<String> set = list.stream().collect(Collectors.toSet());

// List转Map
Map<String, User> map = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// Set转List（保持插入顺序）
List<String> list = set.stream().collect(Collectors.toList());
```

---

## 八、Stream进阶技巧

### 8.1 并行流

```java
// 使用并行流
List<User> result = users.parallelStream()
    .filter(u -> u.getAge() > 18)
    .collect(Collectors.toList());

// 注意事项：并行流不一定更快，需要根据数据量和操作复杂度判断
```

### 8.2 无限流

```java
// 生成无限流
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);

// 限制数量
List<Integer> first10 = infinite.limit(10).collect(Collectors.toList());

// 生成随机数
Stream<Double> randoms = Stream.generate(Math::random);
List<Double> randomList = randoms.limit(5).collect(Collectors.toList());
```

### 8.3 自定义收集器

```java
// 自定义收集器示例：收集到不可变List
Collector<String, ?, List<String>> immutableListCollector = Collector.of(
    ArrayList::new,
    List::add,
    (left, right) -> { left.addAll(right); return left; },
    Collections::unmodifiableList
);

List<String> immutable = strings.stream()
    .collect(immutableListCollector);
```

---

## 九、注意事项

### 9.1 Lambda中的变量捕获

```java
// 局部变量必须是final或effectively final
int limit = 100;  // effectively final
List<Product> filtered = products.stream()
    .filter(p -> p.getPrice() < limit)  // 允许
    .collect(Collectors.toList());

// 不能在Lambda中修改外部变量
int counter = 0;
list.forEach(item -> {
    // counter++;  // 编译错误
});
```

### 9.2 方法引用的使用场景

```java
// 优先使用方法引用，代码更简洁
users.stream().map(User::getName)  // 优于 users.stream().map(u -> u.getName())

// 方法引用类型
// 1. 静态方法引用：ClassName::staticMethod
// 2. 实例方法引用：instance::method
// 3. 对象方法引用：ClassName::method
// 4. 构造器引用：ClassName::new
```

### 9.3 性能考虑

```java
// 避免在中间操作中执行昂贵操作
list.stream()
    .filter(item -> expensiveOperation(item))  // 每次过滤都执行
    .collect(Collectors.toList());

// 考虑提前计算或缓存
Map<Item, Boolean> cache = new HashMap<>();
list.stream()
    .filter(item -> cache.computeIfAbsent(item, this::expensiveOperation))
    .collect(Collectors.toList());
```

---

## 总结

Lambda表达式极大提升了代码的简洁性和可读性，尤其是在处理集合数据时。掌握这些常用模式可以显著提高开发效率。建议结合Stream API、Optional、CompletableFuture等一起使用，构建更加优雅的Java代码。
