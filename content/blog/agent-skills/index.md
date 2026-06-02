---
title: 聊聊 Agent Skills：把"能力"模块化地装进 Agent
date: 2026-06-02T15:00:00+08:00
tags: [Agent, Skills, LLM]
series: []
featured: false
description: "Skills 不是 prompt 也不是工具，而是介于二者之间的一层可复用能力封装。它解决了什么问题，又该如何设计？"
draft: true
ShowToc: true
TocOpen: true
---

## 前言

<!-- 写在这里：为什么聊 Skills？它解决了哪些 prompt / tool / MCP 都没解决好的问题？ -->


## 什么是 Agent Skill

<!-- Skill 的定义、与 prompt / tool / subagent / MCP 的边界 -->


## Skill 的组成

<!-- frontmatter（name / description / triggers）+ 正文指令 + 可选的脚本资源 -->


## Skill 的加载与触发

<!-- 何时被发现、何时被注入上下文、触发条件如何写得准 -->


## 设计原则

<!-- 单一职责、可组合、可发现、可降级；写 description 的几个反例 -->


## 案例

<!-- 选 1~2 个实际 skill 拆解：触发场景 / 提示词 / 配套脚本 -->

![alt 文本：示意图客观描述](skill-architecture.png)
*图 1：Skill 在 Agent 体系中的位置*


## 与 MCP / Tool 的关系

<!-- 三层心智模型：Tool 提供能力，MCP 提供能力的传输协议，Skill 提供使用能力的"知识" -->


## 局限与坑

<!-- description 写不好就召不出来；skill 之间相互覆盖；版本演进的兼容问题 -->


## 总结

<!-- 一句话收束：Skill 让 Agent 的能力从"写死在系统提示词里"变成"按需加载的模块" -->


## 参考

- <!-- 官方文档 / 相关讨论链接 -->
