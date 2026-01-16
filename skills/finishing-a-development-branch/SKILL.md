---
name: finishing-a-development-branch
description: 当实施完成、所有测试通过，你需要决定如何集成工作时使用 - 通过为合并、PR 或清理提供结构化选项来引导开发工作完成
---

# 完成开发分支

## 概述

通过展示清晰选项和处理所选工作流来引导开发工作完成。

**核心原则：** 验证测试 → 展示选项 → 执行选择 → 清理。

**开始时声明："我正在使用 finishing-a-development-branch 技能完成此工作。"**

## 流程

### 步骤 1：验证测试

**在展示选项之前，验证测试通过：**

```bash
# 运行项目的测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 失败）。必须在完成前修复：

[展示失败]

在测试通过之前无法继续合并/PR。
```

停止。不要进入步骤 2。

**如果测试通过：** 继续步骤 2。

### 步骤 2：确定基础分支

```bash
# 尝试常见基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："此分支从 main 分离 - 那正确吗？"

### 步骤 3：展示选项

准确展示这 4 个选项：

```
实施完成。你想做什么？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 按原样保留分支（我稍后处理）
4. 丢弃此工作

哪个选项？
```

**不要添加解释** - 保持选项简洁。

### 步骤 4：执行选择

#### 选项 1：本地合并

```bash
# 切换到基础分支
git checkout <base-branch>

# 拉取最新
git pull

# 合并功能分支
git merge <feature-branch>

# 验证合并结果的测试
<test command>

# 如果测试通过
git branch -d <feature-branch>
```

然后：清理 worktree（步骤 5）

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<2-3 个变更要点>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

然后：清理 worktree（步骤 5）

#### 选项 3：按原样保留

报告："保留分支 <name>。Worktree 保留在 <path>。"

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- <path> 处的 worktree

输入 'discard' 确认。
```

等待确切确认。

如果确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（步骤 5）

### 步骤 5：清理 Worktree

**对于选项 1、2、4：**

检查是否在 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

如果是：
```bash
git worktree remove <worktree-path>
```

**对于选项 3：** 保留 worktree。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 按原样保留 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓ (强制) |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏代码，创建失败的 PR
- **修复：** 始终在提供选项前验证测试

**开放式问题**
- **问题：** "我接下来应该做什么？" → 模糊
- **修复：** 准确展示 4 个结构化选项

**自动 worktree 清理**
- **问题：** 在可能需要时删除 worktree（选项 2、3）
- **修复：** 仅清理选项 1 和 4

**丢弃无确认**
- **问题：** 意外删除工作
- **修复：** 要求输入"discard"确认

## 危险信号

**永远不要：**
- 在测试失败时继续
- 在不验证结果测试的情况下合并
- 在无确认的情况下删除工作
- 在无明确请求的情况下强制推送

**始终：**
- 在提供选项前验证测试
- 准确展示 4 个选项
- 获取选项 4 的输入确认
- 仅清理选项 1 和 4 的 worktree

## 集成

**被调用者：**
- **subagent-driven-development**（步骤 7）- 所有任务完成后
- **executing-plans**（步骤 5）- 所有批次完成后

**配对者：**
- **using-git-worktrees** - 清理该技能创建的 worktree
