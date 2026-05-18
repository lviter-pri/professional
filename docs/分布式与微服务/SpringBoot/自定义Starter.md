# SpringBoot自定义Starter

## 一、Starter的概念

Starter是SpringBoot的核心特性，它提供了：
- 自动配置
- 依赖管理
- 约定优于配置

## 二、创建自定义Starter的步骤

### 2.1 创建Maven项目

**pom.xml配置：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
    <name>my-spring-boot-starter</name>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starters</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

### 2.2 创建配置属性类

```java
@ConfigurationProperties(prefix = "my.starter")
public class MyStarterProperties {
    
    private String name = "default";
    private int timeout = 5000;
    private boolean enabled = true;
    
    // getters and setters
}
```

### 2.3 创建自动配置类

```java
@Configuration
@EnableConfigurationProperties(MyStarterProperties.class)
@ConditionalOnClass(MyService.class)
@ConditionalOnProperty(prefix = "my.starter", name = "enabled", havingValue = "true")
public class MyStarterAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyStarterProperties properties) {
        return new MyService(properties);
    }
}
```

### 2.4 创建业务类

```java
public class MyService {
    
    private final MyStarterProperties properties;
    
    public MyService(MyStarterProperties properties) {
        this.properties = properties;
    }
    
    public String sayHello() {
        return "Hello from " + properties.getName();
    }
}
```

### 2.5 创建spring.factories文件

在`src/main/resources/META-INF`目录下创建：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.mystarter.MyStarterAutoConfiguration
```

## 三、使用自定义Starter

### 3.1 添加依赖

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 3.2 配置属性

```yaml
my:
  starter:
    name: MyApp
    timeout: 10000
    enabled: true
```

### 3.3 使用服务

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        ConfigurableApplicationContext context = 
            SpringApplication.run(Application.class, args);
        
        MyService service = context.getBean(MyService.class);
        System.out.println(service.sayHello());
    }
}
```

## 四、常用条件注解

| 注解 | 说明 |
|------|------|
| `@ConditionalOnClass` | 当类存在时生效 |
| `@ConditionalOnMissingClass` | 当类不存在时生效 |
| `@ConditionalOnBean` | 当Bean存在时生效 |
| `@ConditionalOnMissingBean` | 当Bean不存在时生效 |
| `@ConditionalOnProperty` | 当配置属性满足条件时生效 |
| `@ConditionalOnWebApplication` | 当是Web应用时生效 |

## 五、最佳实践

1. **命名规范**：artifactId使用`xxx-spring-boot-starter`格式
2. **提供默认配置**：确保Starter可以开箱即用
3. **文档说明**：提供清晰的使用文档
4. **可配置性**：支持通过配置属性定制行为
5. **条件装配**：使用条件注解避免不必要的Bean创建
