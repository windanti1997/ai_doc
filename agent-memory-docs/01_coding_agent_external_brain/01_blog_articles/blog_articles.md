# 博客文章合集：给 Coding Agent 装一个外置大脑

## 01 别再把 Agent 记忆做成向量库了

### 副标题

人类记忆不是硬盘，Agent Memory 也不该只是 RAG。

### 核心观点

人类记忆不是为了保存过去，而是为了指导未来行动。Agent Memory 也不应该只是存储聊天记录、摘要和向量索引，而应该是一套让 Agent 从经历中形成经验、习惯和工作风格的系统。

### 正文提纲

1. 开头：为什么一个每天重复踩坑的工程师不算聪明？
2. 常见误区：把 Agent Memory 做成数据库或 RAG。
3. 人类记忆生命周期：注意、编码、加权、重复、巩固、召回、更新、遗忘、习惯化、自我叙事。
4. 工程映射：Memory Gate、Episodic Recorder、Importance Scorer、Dream、Curator、Skill Promoter、Self Model。
5. 钉子句：记忆的价值不是保存过去，而是改变下一次行动。

### 可发布正文摘要

现在很多 Coding Agent 能写代码、改文件、跑测试，但仍然经常像新人一样重复踩坑。问题不只是上下文不够，而是缺少经验系统。

RAG 能帮 Agent 查到过去资料，但不能保证这些资料仍然有效，也不能自动把失败经验升级成规则和技能。人类记忆的关键不是存储容量，而是生命周期管理：什么值得注意，什么值得编码，什么需要加权，什么需要巩固，什么应该遗忘，什么应该变成习惯。

因此，Agent Memory 的目标不应该是“记住更多”，而应该是“让下一次行动更好”。

---

## 02 为什么你的 Agent 总像第一次见这个项目？

### 副标题

Coding Agent 需要的不是更长上下文，而是记忆生命周期。

### 核心观点

一个真正有记忆的 Coding Agent 至少需要四层记忆：Working、Episodic、Semantic、Procedural，并配套 Write、Retrieve、Consolidate、Forget、Promote 五个治理机制。

### 正文提纲

1. 为什么 RAG 不等于 Agent Memory。
2. 为什么长上下文不等于长期记忆。
3. 四层记忆：当前任务、历史经历、项目知识、可复用技能。
4. 五个机制：写入、召回、巩固、遗忘、晋升。
5. Memory Schema：scope、evidence、confidence、last_verified、invalidated_by。
6. 钉子句：Agent Memory 不是 RAG，而是经历到技能的生命周期系统。

### 可发布正文摘要

很多 Agent 之所以总像第一次见项目，是因为它只有上下文，没有经验。它能看到当前文件，但不知道哪些历史坑点仍然有效；它能总结对话，但不知道哪些事实经过验证；它能保存规则，但不会把重复流程升级成技能。

真正的 Agent Memory 应该把当前任务状态、历史执行轨迹、项目长期知识和可复用流程分开管理。更重要的是，它必须知道什么值得写入、什么时候召回、如何巩固、如何遗忘，以及什么时候把经验升级成 skill。

---

## 03 我对比了十几个 Coding Agent，发现大多数还不会长经验

### 副标题

Codex、Claude Code、OpenCode、Hermes、MiMo 谁真正开始做记忆系统？

### 核心观点

主流 Coding Agent 可以分成五类：规则型、上下文型、自动记忆型、生命周期型、进化型。大多数还停在记录和整理，少数开始进入抽象和技能化。

### 正文提纲

1. 成长路径：经历 → 记录 → 整理 → 抽象 → 固化。
2. 规则型：Aider、Gemini CLI、Cursor Rules、Trae Rules。
3. 上下文型：Cursor、Windsurf、Cline、Roo Code。
4. 自动记忆型：Claude Code、Codex、Cascade、Devin、Qwen Code。
5. 生命周期型：Codex、Claude Code、OpenCode、Kiro。
6. 进化型：Hermes、MiMo Code、Qwen Code。
7. 结论：真正分水岭是有没有 Skill Promotion。

### 可发布正文摘要

如果按“会不会长经验”来评价，现在大多数 Coding Agent 还只是会读规则、会维持上下文、会自动总结。它们能记住发生过什么，但很少能把重复经验升级成稳定能力。

Codex、Claude Code、OpenCode 这类工具的价值在于提供了 Hook、Skills、Subagents 等生命周期入口。Hermes 和 MiMo Code 的价值在于开始把经历升级成 skill、workflow 和 memory consolidation。这意味着竞争点正在从模型能力转向 Agent Runtime。

---

## 04 给 Codex 装一个海马体

### 副标题

用 Hook、Checkpoint 和 Verifier 做 Agent Memory Runtime。

### 核心观点

Hook 是给 Coding Agent 外挂记忆系统的最佳切入口。它能在 SessionStart、UserPrompt、PostToolUse、PreCompact、Stop 等关键节点记录、召回、验证和巩固记忆。

### 正文提纲

1. 为什么 Hook 比事后总结更可靠。
2. SessionStart：加载最小必要记忆。
3. UserPrompt：根据任务召回相关经验。
4. PostToolUse：把工具结果写成情景记忆。
5. PreCompact：上下文压缩前写 checkpoint。
6. Stop：防止 Agent 假完成。
7. Dream / Distill / Curator：离线巩固、技能提炼、过期清理。
8. 最小目录设计。

### 可发布正文摘要

记忆不是任务结束后的 summary，而是发生在 Agent 工作过程中的证据采集和状态维护。Hook 的价值在于，它能在工具调用、测试失败、上下文压缩、任务停止这些关键时刻捕捉事实。

我们可以用 Hook 让 Agent 启动时加载项目记忆，用户输入时召回历史坑点，工具调用后记录测试结果，压缩前写 checkpoint，停止前检查是否真的完成。这样不需要训练新模型，也能给 Codex / Claude Code / OpenCode 装上一套外置记忆运行时。

---

## 05 从踩坑到技能：让 Agent 不再重复犯错

### 副标题

经验如何升级成 Skill，并塑造 Agent 的工作风格。

### 核心观点

Agent Memory 的终点不是 project_memory.md，而是可复用 Skill、Workflow、Hook、Command 和 Subagent Playbook。长期记忆还会塑造 Agent 的默认工作风格。

### 正文提纲

1. 为什么只会记笔记的 Agent 仍然像新人。
2. Episode → Fact → Rule → Skill → Automation。
3. 什么经验值得升级成 Skill。
4. 错误经验如何污染 Skill Library。
5. Dream、Distill、Curator 的分工。
6. 记忆如何塑造严谨型、探索型、安全型、架构型 Agent。
7. 个人版到团队版的伏笔。

### 可发布正文摘要

一个成熟工程师不是记住了更多日志，而是形成了稳定工作流程。Agent 也一样。一次失败只是 episode，多次验证才是 fact，长期有效才是 rule，反复执行才应该升级成 skill。

真正强的 Agent，不是会写更多代码，而是会把经验变成稳定工作方式。你给它什么记忆，它就会逐渐变成什么样的工程师。
