# Trae + OpenSpec 实践教程

> 掌握 Trae 与 OpenSpec 的深度集成，实现 AI 规范驱动的可预测编程

> **核心理念**：OpenSpec 通过在编码前建立人与 AI 的规范共识，将"模糊提示 -> 不可预测结果"的 AI 开发模式，升级为"规范对齐 -> 可预测交付"的工程化开发流程。


---

## 学习目标

- 掌握 Trae 的安装、配置与使用
- 熟悉 Trae 项目规则（rules）和技能（skills）机制
- 掌握 OpenSpec 的安装与项目初始化
- 理解 OpenSpec 核心概念：规范、变更、产物
- 熟练使用 OpenSpec 常用命令
- 能够将 OpenSpec 工作流与 Trae 深度集成
- 建立 AI 规范驱动的开发思维

---

## 环境准备

### 前置要求

| 软件                  | 版本要求 | 说明              |
| --------------------- | -------- | ----------------- |
| **Node.js**           | 20.19.0+ | OpenSpec 运行依赖 |
| **npm/pnpm/yarn/bun** | 最新版   | 包管理器          |
| **Trae**              | 最新版   | AI 编程助手       |

### 安装 Node.js

官网下载安装包安装

[Nodejs官网](https://nodejs.org/zh-cn)

```bash
# 验证 Node.js 版本
node --version
# 确保版本 >= 20.19.0
```

---

## Trae 配置

### 项目目录结构

在工程目录下创建 `.trae` 目录，包含规则和技能文件夹：

- .trae/rules 项目规则目录
- .trae/skills 项目技能目录

```
your-project/
├── .trae/                   # CodeBuddy 元文件目录
│   ├── rules/                    # 项目规则目录
│   │   ├── ai_rule.md            # 服务端开发规范
│   │   └── company_rule.md           # 跨越速运服务端开发规范
│   └── skills/                   # 项目技能目录
│       └── <skill-name>/         # 技能文件夹
│           └── SKILL.md          # 技能描述文件
└── ...（你原有的代码）
```

### 安装后端规则&技能

将仓库中rules和skills下的文件复制到项目对应目录下

[基础服务后端技能库]()


## OpenSpec 环境安装

### 全局安装 OpenSpec

```bash
# 使用 npm（推荐）
npm install -g @fission-ai/openspec@latest

# 使用 pnpm
pnpm add -g @fission-ai/openspec@latest

# 使用 yarn
yarn global add @fission-ai/openspec@latest
```

### 验证安装

```bash
# 检查版本
openspec version

# 查看帮助
openspec --help
```

### 配置扩展工作流能力

OpenSpec默认提供4个标准命令：`/opsx:propose`、`/opsx:explore`、`/opsx:apply`、`/opsx:archive`

> **提示**：配置向导选择 `Delivery and workflows` 可获得更多扩展命令（`/opsx:new`、`/opsx:continue`、`/opsx:verify` 等）

```bash
# 启动配置向导
openspec config profile

# 交互式选择配置：
# 1. Delivery and workflows - 基础命令
# 2. Both (skills + commands) - 创建技能和命令
# 3. Custom Profile - 自定义配置（全选）
```


### 项目初始化

```bash
# 进入项目目录
cd your-project

# 初始化 OpenSpec
openspec init
```

### 项目初始化产物

- OpenSpec命令相关的commands和skills文件
- `openspec/` OpenSpec主目录
  - `changes/` 【变更工作目录】正在进行的修改
  - `specs/`   【存量规格目录】系统规格的完整描述
  - `config.yaml` 项目配置


### 快速配置

修改 `config.yaml` 项目配置

```yaml
schema: spec-driven

context: |
  语言：中文（简体）
  所有产出物必须用简体中文撰写。

# 产出物规则 - 针对特定产物的额外指导
rules:
  proposal:
    - 思考过程遵循项目规则
    - 说明对现有功能的影响
    - 列出需要依赖外部功能
  specs:
    - 思考过程遵循项目规则
    - 使用 Given/When/Then 格式编写场景
    - 优先引用现有模式，避免重复发明
    - 包含边界条件和异常场景，注重代码健壮性
  design:
    - 思考过程遵循项目规则
    - 设计规范参考相关技能
    - 考虑性能和安全影响
  tasks:
    - 任务粒度适中
    - 按模块分组，使用层级编号
    - 包含必要的测试任务
    - 任务执行后需要确保代码可以编译通过
```


### 更新 OpenSpec

```bash
# 1. 升级包
npm install -g @fission-ai/openspec@latest

# 2. 在每个项目中刷新代理指令
openspec update
```

---

## OpenSpec 核心概念

### 1. 需求规格（Specs）

规范是系统的"真相来源"，描述系统的当前行为和需求。

```
openspec/
└── specs/                  # 需求规格目录
    └── <domain>/           # 按领域组织
        └── spec.md         # 规格文件
```

### 2. 变更（Changes）

每个功能/需求对应一个独立的变更单元。

```
openspec/
└── changes/
    └── <change-name>/      # 变更名称（使用 kebab-case）
        ├── proposal.md     # 提案：为什么变更，变更的范围是什么
        ├── specs/          # 需求规格，场景描述
        │   └── <domain>/
        │       └── spec.md
        ├── design.md       # 技术实现方案
        └── tasks.md        # 实现任务清单
```

### 3. 产物（Artifacts）

变更目录下的四类文档，构成完整的开发上下文：

| 产物文件      | 内容说明                                  |
| ------------- | ----------------------------------------- |
| `proposal.md` | **提案**：为什么做、做什么、范围界定      |
| `specs/`      | **增量规范**：ADDED/MODIFIED/REMOVED 需求 |
| `design.md`   | **设计方案**：技术架构、API 设计、数据库  |
| `tasks.md`    | **任务清单**：带复选框的实现步骤          |

### 4. 归档机制（Archive）

变更完成后自动归档，保持项目整洁：

```
openspec/
└── changes/
    └── archive/
        └── <date>-<change-name>/   # 归档目录
            ├── proposal.md
            ├── specs/
            ├── design.md
            └── tasks.md
```

### 5. 工作流程模式

OpenSpec 提供两种工作流：

#### 快速路径（Core Profile - 默认）

```shell
/opsx:propose --> /opsx:apply --> /opsx:sync --> /opsx:archive
```

| 步骤 | 命令            | 说明                       |
| ---- | --------------- | -------------------------- |
| 1    | `/opsx:propose` | 创建变更提案，生成所有产物 |
| 2    | `/opsx:apply`   | 按任务清单实现             |
| 3    | `/opsx:sync`    | 同步规范状态               |
| 4    | `/opsx:archive` | 归档变更                   |

#### 扩展路径（Delivery Profile）

```
/opsx:new --> /opsx:ff 或 /opsx:continue --> /opsx:apply --> /opsx:verify --> /opsx:archive
```

| 步骤 | 命令             | 说明                     |
| ---- | ---------------- | ------------------------ |
| 1    | `/opsx:new`      | 创建新变更（仅初始化）   |
| 2    | `/opsx:ff`       | 快进（快速完成所有工件） |
| 2    | `/opsx:continue` | 逐步继续（增量完成工件） |
| 3    | `/opsx:apply`    | 应用变更，实施任务       |
| 4    | `/opsx:verify`   | 验证变更是否符合规范     |
| 5    | `/opsx:archive`  | 归档变更                 |


---

## 与 Trae 深度集成

### 集成架构

```
                      Trae
  +------------+  +------------+  +--------------------+
  |   Rules    |  |   Skills   |  |  Slash Commands    |
  |  (规范约束) |  |  (技能扩展) |  |   (opsx 工作流)    |
  +------------+  +------------+  +--------------------+
                           |
                           v
                       OpenSpec
  +------------+  +------------+  +--------------------+
  |   Specs    |  |  Changes   |  |   Artifact System  |
  | (真相来源)  |  |  (变更管理) |  |    (产物驱动)      |
  +------------+  +------------+  +--------------------+
```

### OpenSpec 完整项目结构

执行 `openspec init` 后的完整目录结构：

```
your-project/
├── openspec/                      # OpenSpec 主目录
│   ├── specs/                     # 真相来源
│   │   ├── auth/                  # 认证模块规范
│   │   │   └── spec.md
│   │   ├── users/                 # 用户模块规范
│   │   │   └── spec.md
│   │   └── <domain>/              # 其他领域规范
│   │       └── spec.md
│   ├── changes/                   # 变更目录
│   │   ├── <active-change>/       # 活动变更
│   │   │   ├── proposal.md
│   │   │   ├── specs/
│   │   │   │   └── <domain>/
│   │   │   │       └── spec.md
│   │   │   ├── design.md
│   │   │   └── tasks.md
│   │   └── archive/               # 已归档变更
│   │       └── <date>-<name>/
│   │           ├── proposal.md
│   │           ├── specs/
│   │           ├── design.md
│   │           └── tasks.md
│   └── config.yaml                # 项目配置（可选）
├── .trae/                    # trae 配置
│   ├── rules/
│   │   └── openspec-workflow.md
│   └── skills/
│       └── openspec/
│           └── SKILL.md
└── ...（你原有的代码）
```

---

## 完整开发流程实战

### 实战案例：story#2026

#### 场景描述

- 业务需求

#### 第一步：提案

```
You: /opsx:propose

开发需求：zentao:story-view-2026.html

设计思路：
1 接口：/api/v1 回调接口，在此基础上扩展逻辑
2 动态配置，参考SystemConfigTools类
3 使用 getEndTime 方法获取需要的数据
4 使用 list 方法获取车辆经纬度
```

AI 自动创建变更目录并生成产物：

```
openspec/changes/sign-time/
├── proposal.md
├── specs/
│   └── check/
│       └── spec.md
├── design.md
└── tasks.md
```

review产物文件，阅读顺序 proposal.md -> design.md -> specs/ -> tasks.md

```
You: review调整了artifact，补充信息：

1.某某方法已存在，可以调整为直接使用此方法
```


#### 第二步：实施

```
You: /opsx:apply
```

AI 按任务清单逐项实现

```
## 1. 某某方法已存在，可以调整为直接使用此方法
## 3. 日志与监控
```

review代码存在依赖import缺失问题，手动修复


#### 第三步：归档

```
You: /opsx:archive
```

归档结果：

```
Archive Complete
变更: sign-time Schema: spec-driven 归档至: openspec/changes/archive/2026-05-07-sign-time/ Specs: 已同步至主 specs（新建 openspec/specs/check/spec.md）

所有 artifacts 完成。所有 tasks 完成。
```

AI 归档后产物：

```
openspec/
├── changes/
│   └── archive/
│       └── 2026-05-07-sign-time/
│           ├── proposal.md
│           ├── specs/
│           │   └── check/
│           │       └── spec.md
│           ├── design.md
│           └── tasks.md
└── specs/check/
    └── spec.md
```

## 常见问题

### Q1: 如何处理大型变更？

```markdown
# 将大型变更拆分为多个小变更
openspec/changes/user-auth/
openspec/changes/user-profile/
openspec/changes/user-notifications/

# 而不是
openspec/changes/complete-user-system/
```

### Q2: 如何回滚已归档的变更？

```markdown
# 归档只是移动目录，可以手动恢复
mv openspec/changes/archive/<date>-<change-name>/ \
   openspec/changes/<change-name>/
```

### Q3: 多分支开发如何处理？

```markdown
# 每个分支维护独立的 changes/ 目录
# 合并时合并 changes/ 目录
# 注意：不要合并 archive/ 目录（保留历史）
```

### Q4: OpenSpec 与 Git 如何配合？

```markdown
# .gitignore 示例
# 归档目录可选择是否纳入版本控制
openspec/changes/archive/
```

### Q5: 如何禁用遥测？

```bash
# 方法 1：环境变量
export OPENSPEC_TELEMETRY=0

# 方法 2：备选方式
export DO_NOT_TRACK=1
```

---

## 相关资源

| 资源             | 链接                                   |
| ---------------- | -------------------------------------- |
| 官方 GitHub      | https://github.com/Fission-AI/OpenSpec |
| OpenSpec中文文档 | https://lzw.me/docs/OpenSpec-Docs-zh   |
| Discord 社区     | https://discord.gg/YctCnvvshC          |