---
description: GIT分支管理规范 - AI编码前强制检测
alwaysApply: true
---

## 核心原则

**安全优化：**禁止在test生产分支直接修改代码

## 强制触发条件

AI执行以下操作前，**必须**先完成分支检测：
- 新建/修改 Java 文件或生成代码（Model、Service、Mapper、Controller、XML等）
- 写入或编辑 src/ 目录下的文件

## 分支检测流程（编码前必执行）
1. 执行 `git branch --show-current` 获取当前分支
2. 判断：
   - **ai_YYYYMMDD_* 分支** → ✅ 可继续编码
   - **test 分支** → 自动创建需求分支（见下方自动建分支流程）
   - **其他分支** → ✅ 可继续编码

## 分支规范
- 生产分支：test（只读）
- 需求分支：`ai_YYYYMMDD_功能描述`（日期取当天，功能英文小写+连字符，如 `ai_20260419_area-code`）
- 禁止代码直接合并到 test，合并前必须 CodeReview
- 已在合规 ai 分支上则无需重复创建

## 自动建分支流程（仅test分支触发）
当前在test分支时，AI **必须自动执行**以下命令创建需求分支：
1. `git checkout test && git pull` — 切换到test并拉取最新代码
2. `git checkout -b ai_YYYYMMDD_功能描述` — 创建并切换到新分支
3. 分支名中 `功能描述` 根据当前任务语义自动生成（英文小写+连字符），日期取当天
4. 建分支成功后，方可继续编码