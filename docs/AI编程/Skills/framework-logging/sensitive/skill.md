---
name: "framework-logging-sensitive"
description: "提供日志敏感信息脱敏规范，确保数据安全合规。"
---

# 日志敏感信息脱敏规范

## 使用时机
- 需要记录用户个人信息时
- 需要记录财务相关数据时
- 涉及合规要求的日志记录

## 一、需要脱敏的信息

| 类型 | 示例 | 脱敏方式 |
|------|------|---------|
| **手机号** | 13812345678 | 138****5678 |
| **身份证号** | 1101011990XXXX1234 | 110101********1234 |
| **银行卡号** | 622202XXXXXXXX1234 | 622202******1234 |
| **邮箱** | user@example.com | u***r@example.com |
| **密码** | - | 禁止记录 |

## 二、脱敏工具类

```java
public class SensitiveUtils {
    // 手机号脱敏
    public static String maskPhone(String phone) {
        if (phone == null || phone.length() < 11) return phone;
        return phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
    }
    
    // 身份证脱敏
    public static String maskIdCard(String idCard) {
        if (idCard == null || idCard.length() < 18) return idCard;
        return idCard.replaceAll("(\\d{6})\\d{8}(\\d{4})", "$1********$2");
    }
    
    // 邮箱脱敏
    public static String maskEmail(String email) {
        if (email == null || !email.contains("@")) return email;
        return email.replaceAll("(.)[^@]*(@.*)", "$1***$2");
    }
    
    // 银行卡号脱敏
    public static String maskBankCard(String cardNo) {
        if (cardNo == null || cardNo.length() < 16) return cardNo;
        return cardNo.replaceAll("(\\d{6})\\d{8}(\\d{4})", "$1******$2");
    }
}
```

## 三、使用示例

```java
// 正确 ✅ - 脱敏后记录
log.info("[UserService.register] 用户注册: phone={}, email={}", 
    SensitiveUtils.maskPhone(phone), 
    SensitiveUtils.maskEmail(email));

// 错误 ❌ - 记录敏感信息
log.info("[UserService.register] 用户注册: phone={}, password={}", phone, password);
```

## 四、日志审计检查

### 正则检查规则

```java
public class LogAudit {
    // 检查是否包含未脱敏手机号
    public static boolean containsPhone(String logContent) {
        return Pattern.matches(".*1[3-9]\\d{9}.*", logContent);
    }
    
    // 检查是否包含身份证号
    public static boolean containsIdCard(String logContent) {
        return Pattern.matches(".*\\d{18}.*", logContent);
    }
}
```

## 五、最佳实践

1. **统一脱敏**：使用统一的脱敏工具类
2. **禁止记录密码**：任何情况下都不要记录密码
3. **审计检查**：定期检查日志是否包含敏感信息
4. **团队规范**：制定团队日志规范并培训

---

**参考文档**：
- [基础日志规范](../basic/skill.md)
- [MDC结构化日志](../mdc/skill.md)
- [日志配置最佳实践](../config/skill.md)
- [日志排查技巧](../troubleshooting/skill.md)
