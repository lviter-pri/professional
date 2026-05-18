---
name: "git-commit"
description: "提交并推送所有变更（排除.trae目录）。当用户需要提交代码变更或完成文档修改后调用。"
---

# Git 提交工具

## 概述

此工具自动化执行 Git 提交和推送操作，自动排除包含临时工作空间文件的 `.trae` 目录。

## 使用场景

在以下情况调用此工具：
- 完成文档或代码修改后
- 需要提交除 `.trae` 目录外的所有变更
- 需要将变更推送到远程仓库

## 执行的命令

该工具执行以下 Git 命令：

```bash
# 添加所有文件（排除.trae目录）
git add --all
git reset HEAD .trae/

# 检查是否有变更
if ! git diff --cached --quiet; then
    # 提交（带时间戳消息）
    git commit -m "Auto commit: $(date)"
    
    # 推送到远程仓库（使用 -q 静默模式，检测 stderr）
    git push -q 2>&1 || {
        echo "Error: Git push failed. Please check your credentials and push manually."
        exit 1
    }
else
    echo "No changes to commit."
fi
```

## 功能特性

1. **自动排除**: 自动从提交中排除 `.trae` 目录
2. **时间戳提交信息**: 生成带当前时间戳的提交消息
3. **一键操作**: 按顺序执行添加、提交和推送
4. **变更检测**: 如果没有变更，跳过提交步骤
5. **错误处理**: 当 git push 失败（如权限问题、需要手动确认等），中断操作并提示用户手动处理

## 权限问题处理

当执行 `git push` 时，如果遇到以下情况，工具会中断并提示用户手动处理：

| 场景 | 表现 | 处理方式 |
|------|------|----------|
| SSH 主机密钥验证 | 提示 "Are you sure you want to continue connecting?" | 中断，用户手动执行 git push |
| 凭据过期 | 提示输入用户名密码 | 中断，用户手动执行 git push |
| 远程仓库权限不足 | 提示 permission denied | 中断，用户手动执行 git push |
| 分支冲突 | 提示 push 被拒绝 | 中断，用户手动解决冲突后 push |

## 注意事项

- 确保已正确配置 Git 凭据
- 工具假设您已设置好远程仓库
- `.trae` 目录外的所有变更都会被提交
- 如果没有检测到变更，工具会跳过提交
- 如果 git push 失败，工具会中断并提示您手动执行 `git push`