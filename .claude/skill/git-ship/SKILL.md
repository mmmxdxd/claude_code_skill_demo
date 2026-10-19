---
name: git-ship
description: >
  将当前修改提交并推送到 GitHub 远程仓库，自动创建 Pull Request。
  当用户说"提交代码"、"推送到GitHub"、"创建PR"、"ship it"时触发。
---

# Git Ship — 一键提交推送并创建 PR

执行以下步骤，任何一步失败都停   下来报告错误，不要继续：

## 步骤 1：检查环境
- 运行 `gh auth status` 确认 GitHub CLI 已认证
- 运行 `git status` 检查是否有未提交的修改
- 如果没有修改，告诉用户"没有需要提交的内容"并停止

## 步骤 2：暂存与提交
- 运行 `git add -A` 暂存所有修改
- 分析 `git diff --cached --stat` 的输出，理解修改了哪些文件
- 根据修改内容生成符合 Conventional Commits 规范的提交信息：
  - feat: 新功能
  - fix: 修复 bug
  - docs: 文档变更
  - refactor: 重构
  - chore: 杂项
- 提交格式示例：`feat(auth): 添加用户登录的邮箱验证`
- 执行 `git commit -m "生成的提交信息"`

## 步骤 3：推送到远程
- 获取当前分支名：`git branch --show-current`
- 如果是 main 或 master 分支，**警告用户**并询问是否要先创建新分支
- 推送：`git push origin <当前分支名>`
- 如果远程没有该分支，使用 `git push -u origin <当前分支名>`

## 步骤 4：创建 Pull Request
- 使用 `gh pr create` 创建 PR
- PR 标题 = commit 信息
- PR 描述自动生成，包含：
  - **变更摘要**：一句话概括
  - **修改列表**：列出所有变更的文件和修改内容
  - **测试说明**：建议的测试方式
- 目标分支默认为 main

## 安全规则
- **绝对不要** force push（`--force`）
- **绝对不要** 直接推送到 main/master（除非用户明确要求）
- 发现 .env、密钥文件、token 等敏感文件时**立即停止**并警告
