# Groovy 基础语法详解

***

## 一、概述

### 1.1 什么是 Groovy？

Groovy 是一种基于 JVM（Java Virtual Machine）的动态编程语言，它：

- **完全兼容 Java**：可以直接调用 Java 类和库
- **语法简洁**：相比 Java 更加简洁灵活
- **动态特性**：支持动态类型、闭包、元编程等
- **脚本语言特性**：适合快速开发和脚本编写

### 1.2 Groovy 主要特性

| 特性         | 说明                             |
| ---------- | ------------------------------ |
| **动态类型**   | 使用 `def` 关键字声明变量，类型推断          |
| **闭包**     | 强大的函数式编程支持                     |
| **语法糖**    | 简化的语法，减少样板代码                   |
| **集合操作**   | 强大的集合处理方法（each、collect、find 等） |
| **字符串处理**  | 支持多行字符串、字符串插值                  |
| **DSL 支持** | 便于构建领域特定语言                     |

### 1.3 与 Java 的关系

```
┌─────────────────────────────────────────────┐
│              JVM 虚拟机                      │
├─────────────────────────────────────────────┤
│  Java 字节码                                │
├─────────────────────────────────────────────┤
│  ┌─────────┐    ┌─────────┐               │
│  │  Java   │    │ Groovy  │               │
│  └────┬────┘    └────┬────┘               │
│       │              │                     │
│       └──────┬───────┘                     │
│              ▼                             │
│       共享 JDK 类库                         │
└─────────────────────────────────────────────┘
```

**关键点**：

- Groovy 代码编译为 Java 字节码
- 可以直接使用所有 Java 类和库
- Java 代码也可以调用 Groovy 类

***

## 二、变量与数据类型

### 2.1 def 关键字

Groovy 使用 `def` 关键字声明变量，支持类型推断：

```groovy
// 基本用法
def name = "Groovy"    // String 类型
def age = 20           // int 类型
def pi = 3.14159       // double 类型
def flag = true        // boolean 类型

// 也可以显式声明类型
String greeting = "Hello"
int count = 100
```

### 2.2 基本数据类型

Groovy 支持所有 Java 基本类型的包装类：

| 类型      | 示例                | 说明     |
| ------- | ----------------- | ------ |
| int     | `def i = 10`      | 整数     |
| long    | `def l = 100L`    | 长整数    |
| double  | `def d = 3.14`    | 双精度浮点数 |
| float   | `def f = 3.14f`   | 单精度浮点数 |
| boolean | `def b = true`    | 布尔值    |
| String  | `def s = "hello"` | 字符串    |

### 2.3 字符串操作

Groovy 提供了丰富的字符串操作：

```groovy
// 单引号字符串（普通字符串）
def str1 = 'Hello World'

// 双引号字符串（支持字符串插值）
def name = "Groovy"
def str2 = "Hello ${name}"  // 结果: "Hello Groovy"

// 三引号字符串（多行字符串）
def multiLine = """
    This is a
    multi-line
    string
"""

// 字符串方法
def str = "Hello Groovy"
println str.length()        // 12
println str.toUpperCase()   // HELLO GROOVY
println str.contains("Groovy")  // true
println str.split(" ")[0]   // Hello

// 字符串乘法
def repeated = "ab" * 3    // "ababab"
```

***

## 三、控制结构

### 3.1 条件判断

```groovy
// if-else
def score = 85
if (score >= 90) {
    println "优秀"
} else if (score >= 80) {
    println "良好"
} else {
    println "继续努力"
}

// switch 语句（支持多种类型）
def x = "Groovy"
switch (x) {
    case "Java": println "Java 语言"
        break
    case "Groovy": println "Groovy 语言"
        break
    default: println "未知"
}

// 简化的三元运算符
def result = score > 60 ? "及格" : "不及格"
```

### 3.2 循环结构

```groovy
// for 循环（传统方式）
for (int i = 0; i < 5; i++) {
    println i
}

// for-in 循环（遍历集合）
def list = [1, 2, 3, 4, 5]
for (num in list) {
    println num
}

// while 循环
def count = 0
while (count < 5) {
    println count
    count++
}

// 增强的循环（结合闭包）
list.each { println it }
```

### 3.3 异常处理

```groovy
try {
    def result = 10 / 0
} catch (ArithmeticException e) {
    println "算术异常: ${e.message}"
} catch (Exception e) {
    println "其他异常"
} finally {
    println "无论如何都会执行"
}
```

***

## 四、函数与闭包

### 4.1 函数定义与调用

```groovy
// 基本函数定义
def sayHello(String name) {
    return "Hello, ${name}!"
}

// 调用函数
println sayHello("World")  // Hello, World!

// 简化语法（return 可省略）
def add(int a, int b) {
    a + b  // 最后一行表达式作为返回值
}

// 无参数函数
def greet() {
    "Hello from Groovy"
}

// 默认参数
def introduce(String name, String title = "工程师") {
    "${name} 是一名 ${title}"
}
println introduce("张三")        // 张三 是一名 工程师
println introduce("李四", "设计师")  // 李四 是一名 设计师
```

### 4.2 闭包基础

闭包是 Groovy 的核心特性：

```groovy
// 定义闭包
def square = { num -> num * num }

// 调用闭包
println square(5)  // 25

// 闭包作为参数
def process(int num, Closure operation) {
    operation(num)
}

process(10, { it * 2 })  // 20

// 使用 it 作为默认参数
def doubleIt = { it * 2 }
println doubleIt(5)  // 10
```

### 4.3 闭包进阶用法

```groovy
// 闭包与集合
def numbers = [1, 2, 3, 4, 5]

// each 遍历
numbers.each { println it }

// collect 转换
def doubled = numbers.collect { it * 2 }  // [2, 4, 6, 8, 10]

// find 查找
def found = numbers.find { it > 3 }  // 4

// findAll 查找所有
def evens = numbers.findAll { it % 2 == 0 }  // [2, 4]

// inject 累积计算
def sum = numbers.inject(0) { acc, num -> acc + num }  // 15

// 闭包捕获外部变量
def multiplier = 3
def triple = { it * multiplier }
println triple(5)  // 15
```

***

## 五、集合操作

### 5.1 List 操作

```groovy
// 创建 List
def list = [1, 2, 3, 4, 5]

// 访问元素
println list[0]      // 1
println list[-1]     // 5（倒数第一个）
println list[1..3]   // [2, 3, 4]（范围操作）

// 添加元素
list.add(6)
list << 7            // 快捷添加
println list         // [1, 2, 3, 4, 5, 6, 7]

// 删除元素
list.remove(0)       // 删除第一个元素
list -= 5            // 删除指定元素

// List 方法
println list.size()          // 6
println list.contains(3)     // true
println list.sort()          // 排序
println list.reverse()       // 反转
```

### 5.2 Map 操作

```groovy
// 创建 Map
def map = [name: "张三", age: 25, city: "北京"]

// 访问元素
println map.name     // 张三
println map["age"]   // 25

// 添加元素
map.email = "zhangsan@example.com"
map["phone"] = "123456789"

// 遍历 Map
map.each { key, value ->
    println "${key}: ${value}"
}

// Map 方法
println map.keySet()     // [name, age, city, email, phone]
println map.values()     // [张三, 25, 北京, zhangsan@example.com, 123456789]
println map.isEmpty()   // false
```

### 5.3 集合方法汇总

| 方法        | 说明           | 示例                                     |
| --------- | ------------ | -------------------------------------- |
| `each`    | 遍历集合         | `list.each { println it }`             |
| `collect` | 转换集合         | `list.collect { it * 2 }`              |
| `find`    | 查找第一个匹配元素    | `list.find { it > 5 }`                 |
| `findAll` | 查找所有匹配元素     | `list.findAll { it % 2 == 0 }`         |
| `inject`  | 累积计算         | `list.inject(0) { acc, x -> acc + x }` |
| `every`   | 判断所有元素是否满足条件 | `list.every { it > 0 }`                |
| `any`     | 判断是否有元素满足条件  | `list.any { it > 10 }`                 |

***

## 六、面向对象

### 6.1 类定义

```groovy
// 简单类定义
class Person {
    // 属性（自动生成 getter/setter）
    String name
    int age
    
    // 构造函数
    Person(String name, int age) {
        this.name = name
        this.age = age
    }
    
    // 方法
    def introduce() {
        "我叫 ${name}，今年 ${age} 岁"
    }
}

// 使用类
def person = new Person("张三", 25)
println person.introduce()  // 我叫 张三，今年 25 岁
```

### 6.2 简化的类定义

Groovy 提供了更简洁的语法：

```groovy
// 简化写法（自动生成构造函数和 getter/setter）
class Person {
    String name
    int age
    
    // 额外方法
    def introduce() {
        "我叫 ${name}，今年 ${age} 岁"
    }
}

// 使用位置参数构造
def p1 = new Person("张三", 25)

// 使用命名参数构造
def p2 = new Person(name: "李四", age: 30)
```

### 6.3 继承与接口

```groovy
// 定义父类
class Animal {
    String name
    
    def speak() {
        "${name} 发出声音"
    }
}

// 定义接口
interface Flyable {
    def fly()
}

// 继承并实现接口
class Bird extends Animal implements Flyable {
    @Override
    def speak() {
        "${name} 叽叽喳喳叫"
    }
    
    @Override
    def fly() {
        "${name} 在飞翔"
    }
}

def bird = new Bird(name: "麻雀")
println bird.speak()  // 麻雀 叽叽喳喳叫
println bird.fly()    // 麻雀 在飞翔
```

***

## 七、与 Java 互操作

### 7.1 调用 Java 类

Groovy 可以直接使用 Java 类：

```groovy
// 导入 Java 类
import java.util.ArrayList
import java.time.LocalDate

// 使用 Java 集合
def arrayList = new ArrayList<String>()
arrayList.add("Hello")
arrayList.add("Groovy")
println arrayList

// 使用 Java 8 日期时间 API
def today = LocalDate.now()
println today.toString()  // 2024-01-15

// 使用 Java 字符串方法
def str = "Hello World"
println str.substring(0, 5)  // Hello
```

### 7.2 Groovy 特性 vs Java

| 特性    | Groovy                        | Java                                           |
| ----- | ----------------------------- | ---------------------------------------------- |
| 变量声明  | `def x = 10`                  | `int x = 10;`                                  |
| 字符串插值 | `"Hello ${name}"`             | `String.format("Hello %s", name)`              |
| 集合创建  | `def list = [1, 2, 3]`        | `List<Integer> list = Arrays.asList(1, 2, 3);` |
| 方法调用  | `obj.method()` 或 `obj.method` | `obj.method();`                                |
| 闭包    | 支持                            | Java 8+ 支持 Lambda                              |
| 可选分号  | 可选                            | 必须                                             |

***

## 八、实战示例

### 8.1 日期处理示例

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter

def formatDate(String dateStr) {
    def formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd")
    def date = LocalDate.parse(dateStr, formatter)
    return date.format(DateTimeFormatter.ofPattern("yyyy年MM月dd日"))
}

// 使用示例
println formatDate("2024-01-15")  // 2024年01月15日
```

### 8.2 数据处理示例

```groovy
// 数据转换和过滤
def users = [
    [name: "张三", age: 25, city: "北京"],
    [name: "李四", age: 30, city: "上海"],
    [name: "王五", age: 22, city: "北京"],
    [name: "赵六", age: 35, city: "深圳"]
]

// 找出北京的用户
def beijingUsers = users.findAll { it.city == "北京" }

// 提取姓名列表
def names = beijingUsers.collect { it.name }
println names  // [张三, 王五]

// 计算平均年龄
def avgAge = users.inject(0) { sum, user -> sum + user.age } / users.size()
println "平均年龄: ${avgAge}"  // 平均年龄: 28.0
```

### 8.3 脚本入口

```groovy
// 定义主函数
def main() {
    println "Groovy 脚本开始执行"
    
    // 调用其他函数
    def result = calculate(10, 20)
    println "计算结果: ${result}"
    
    processList([1, 2, 3, 4, 5])
}

def calculate(int a, int b) {
    a + b
}

def processList(List list) {
    list.each { println "元素: ${it}" }
}

// 执行主函数
return main()
```

***

## 九、总结

| 核心要点          | 说明                           |
| ------------- | ---------------------------- |
| **动态类型**      | 使用 `def` 声明变量，类型推断           |
| **闭包**        | 强大的函数式编程支持，简化集合操作            |
| **语法简洁**      | 相比 Java 减少大量样板代码             |
| **完全兼容 Java** | 可以直接使用所有 Java 类和库            |
| **集合操作**      | 提供丰富的方法（each、collect、find 等） |
| **字符串处理**     | 支持插值、多行字符串等特性                |

