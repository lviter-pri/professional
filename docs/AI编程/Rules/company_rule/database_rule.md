---
description: 数据库标准
alwaysApply: true
---

## 核心设计原则

**去逻辑化**：禁止使用外键约束、存储过程、视图、触发器，所有业务逻辑必须在应用层（Service 层）实现。

## 数据库使用规范

- **主键定义**：每张表必须有且仅有一个主键，使用 `bigint` 类型的 ID，例如 id bigint not null comment '主键ID' primary key
- **字段注释**：所有字段必须包含 `COMMENT` 描述其业务含义及取值范围。
- 表名采用小写，多个单词用下划线连接，例如 user_info
- 字段名采用小写，多个单词用下划线连接，例如 user_name
- 单表索引数量建议控制在 **5 个**以内。
- **金额**：必须使用 `decimal(18, 2)`，禁止使用 `float` 或 `double`。
- 字段尽可能声明为 `NOT NULL` 并提供默认值（如空字符串或 0）。
- 所有业务表必须包含以下运维审计字段，以支持全链路追踪和逻辑删。

| 字段            | 类型     | 长度 | 默认值            | 说明                    |
| --------------- | -------- | ---- | ----------------- | ----------------------- |
| `trace_id`      | varchar  | 64   | ''                | 日志跟踪id              |
| `created_by_id` | bigint   | 32   | 0                 | 创建人                  |
| `created_time`  | datetime | -    | CURRENT_TIMESTAMP | 创建时间                |
| `updated_by_id` | bigint   | 32   | 0                 | 最后更新人              |
| `enabled_flag`  | tinyint  | 1    | 1                 | 是否有效(1:有效,0:无效) |
| `updated_time`  | datetime | -    | CURRENT_TIMESTAMP | 最后更新时间            |
- 禁止使用外键约束；禁止使用存储过程、视图、触发器