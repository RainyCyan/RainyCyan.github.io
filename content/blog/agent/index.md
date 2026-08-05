---
title: Agent
date: 2026-06-30T15:00:00+08:00
tags: [Agent, LLM, Git, Workflow]
series: []
featured: false
description: "主流 Agent 工具与多 Agent 协作时代的 Git 规范"
draft: false
ShowToc: true
TocOpen: true
---

## 主流 Agent 工具

- Claude Code
- OpenClaw
- OpenCode
- Hermes
- DeepAgents/DeerFlow

## 多 Agent 协作时代的 Git 规范

> 这不是 Git 入门教程。这是团队协作、单人多 Agent 在产品迭代时总结出来的一套"防互相踩踏"协作规约。

AI 写代码比人快好几个数量级，犯错也是。多 Agent 并发的典型失控场景：

- Claude Code 在 worktree A 里改 schema，Cursor 在 worktree B 里改 consumer，Codex 在主目录里"顺手"动了 contract file
- 回头一看 `main` 上多了三个本地 commit，没人记得是谁推的
- 某个 agent 一句 `git stash` 把另一个工程师的未提交工作藏了起来

Git 规范的目标已经从"历史好看"变成：**让所有并发工作都能被定位、隔离、审查、验证、回滚**。

### 工作区隔离：从 `main` 到 worktree

**`main` 只做集成，不做开发。** `main` 是 integration branch，代表团队当前认可的集成状态，不是任何 Agent 的临时草稿区。

```bash
git config pull.rebase true
git config pull.ff only
git config branch.main.rebase false
```

**一个任务 = 一个 branch + 一个 worktree + 一个 owner + 一份 scope。**

```
repo/
  .claude/worktrees/   // 留给 Claude Code
  .cursor/worktrees/   // 留给 Cursor
  .worktrees/          // Codex 或通用任务
```

分支命名携带来源和意图：

```
codex/<scope>-<task>
claude/<scope>-<task>
docs/<topic>
feat/<topic>
fix/<topic>
```

任务分配契约至少明确：branch name、owned files、out-of-scope、test command、can commit/push、dependency PRs。**限制 AI 的写入范围，给 reviewer 提供判断依据。**

### 三道关卡：编辑前、commit 前、PR 前都要有可复现证据

**编辑前：** 从 `origin/main` 创建 worktree，`git status` 必须 clean。

**commit 前：** `git diff --name-only` 检查是否只动了 scope 内的文件。AI 经常会"顺手"动到不在 scope 里的文件。

**PR 前：** `git log --oneline origin/main..HEAD` + `git diff --name-only origin/main...HEAD` + `git diff --check`。

PR 必须小到一个人类 reviewer 真的能读完。合并策略默认 squash merge——一个 PR 对应 mainline 上一个 commit，更容易 revert 和生成 release notes。

### 破坏性动作必须慢动作

AI 写代码越快，破坏性 Git 操作就越要慢：

| 动作 | 风险 | 必须先做的事 |
|------|------|------------|
| `git stash` | 把别人的未提交变更藏起来 | `git status` + `git diff --stat` 确认变更归属 |
| `git reset --hard` | 丢失本地工作 | 先 stash 或 backup branch |
| `git push --force` | 覆盖上游 | 确认 owner，旧状态有 backup |
| 删 worktree | 丢失本地未推送工作 | clean + 无 unpushed commits + 无 open PR |

### AI Context 才是真正的"起跑线"

Git 隔离解决工作区问题，AI context 解决理解问题。很多 AI 事故不是模型不会写代码，而是它**基于错误的现实写出了看起来很合理的代码**。

每个仓库提供轻量上下文，让所有工具从同一组事实开始：

| 文件 | 作用 |
|------|------|
| `AGENTS.md` | 工具中立的 agent 指令和仓库边界 |
| `docs/ARCHITECTURE.md` | implemented / partial / deleted / planned 的单一事实源 |
| `.cursor/rules/*.mdc` | Cursor 专属规则 |

跨仓库改动按顺序：dependency PRs → product integration PRs → docs → cleanup。让依赖方向在 Git 历史里也清晰可见。

### 总结

多 Agent 协作的 Git 规范，本质就是一句话——**给工具划清责任边界**。

| 阶段 | 必做检查 |
|------|---------|
| 开新任务前 | `git fetch --prune` → 从 `origin/main` 创建 worktree → `git status` clean |
| 启动 AI 前 | 写下 branch / scope / out-of-scope / test command / can-push? |
| commit 前 | `git diff --name-only` 看是否只动了 scope 内文件 |
| PR 前 | `git diff --check` + 跑一次真实测试 + 写明 agent / owner / dependency |
| stash / reset / force push | 确认 owner、备份旧状态、人类确认 |
| 删 worktree 前 | clean + 无 unpushed commit + 无 open PR + 无人在用 |

这些规则没有一条是"AI 才需要"的——它们本来就是好工程实践。AI 只是把它们从"建议"变成了"必须"。因为人犯错，commit 之前还有几秒钟的犹豫；AI 犯错，从思考到写盘可能不到一秒。

参考：[Multi-Agent Git Workflow](https://fz.cool/Multi-Agent-Git-Workflow/)