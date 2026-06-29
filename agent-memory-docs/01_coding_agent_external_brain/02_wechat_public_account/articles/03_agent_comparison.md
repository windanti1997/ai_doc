# 03 我对比了十几个 Coding Agent，发现大多数还不会长经验

副标题：Codex、Claude Code、OpenCode、Hermes、MiMo 谁真正开始做记忆系统？

建议封面文案：大多数 Agent 只是在记录，还不会长经验。

---

现在几乎所有 Coding Agent 都在说自己有上下文、有规则、有 memory。

但它们做的是同一件事吗？

不是。

有些只是会读项目规则，有些只是维护当前上下文，有些会自动总结，少数开始把经历变成技能。

如果按照“会不会长经验”来分，主流 Coding Agent 大致可以分成五类：规则型、上下文型、自动记忆型、生命周期型、进化型。

## 第一类：规则型

规则型工具依赖 AGENTS.md、CLAUDE.md、GEMINI.md、.cursor/rules、copilot-instructions.md 等文件。

它们像新人入职时读手册。

优点很明显：简单、透明、可控、适合团队共享。你把项目规范写进去，Agent 每次启动都能读到。

但它也有明显边界：它不会自己从失败中学习。

如果某次测试因为 Redis 没启动失败，规则型系统不会自然把这次经历沉淀成“payment integration tests require Redis”。除非你手动写进规则文件。

所以规则型不是 memory 的终点，而是 memory 的底座。

## 第二类：上下文型

上下文型工具强在 IDE 和 workspace 感知。

Cursor、Windsurf、Cline、Roo Code 这类工具可以读代码、理解文件、跟随当前编辑状态、在一次任务里保持较强连续性。

它们让当前任务体验很好。

但很多经验在 session 结束后不会沉淀。你这次纠正它不要改 generated 文件，下次换一个任务，它未必还记得。它这次发现某个测试命令需要特殊环境，下次也未必主动想起来。

上下文型 Agent 很像短期记忆很强的新人：当前沟通很顺，但长期成长还不稳定。

## 第三类：自动记忆型

自动记忆型工具开始保存用户偏好、项目习惯、测试命令、常见坑点。

Claude Code、Codex、Cascade、Devin、Qwen Code 都在往这个方向走。

这是重要进步。

但自动记忆不等于正确记忆。

它为什么记这条？

证据是什么？

有没有作用域？

还能不能删除、降权、修正？

会不会把一次临时偏好当成长期规则？

会不会把模型猜测当成项目事实？

如果没有治理机制，自动 memory 可能从帮助变成污染。

## 第四类：生命周期型

生命周期型工具开始提供 hooks、skills、subagents、commands、session、compaction 等机制。

Codex、Claude Code、OpenCode、Kiro、Qwen Code 都能看到这个方向。

这很关键。

因为记忆不是任务结束后写一句 summary，而是应该发生在 Agent 工作过程中。

SessionStart 时加载最小必要记忆。

UserPrompt 时按任务召回相关经验。

PostToolUse 时记录命令结果、错误栈、diff、测试验证。

PreCompact 时写 checkpoint，避免长任务断线。

Stop 时阻止假完成。

AfterStop 时生成 memory candidates。

Hook 让 memory 不再只是文件，而成为运行时的一部分。

这就是从“记笔记”走向“Memory Runtime”的分水岭。

## 第五类：进化型

进化型工具最值得关注。

Hermes Agent、MiMo Code 这类系统开始讨论 Dream、Distill、Curator、Skill Learning。

它们关注的不只是存储，而是如何把经历整理成知识，把知识提炼成技能，把技能纳入未来工作流程。

这对应人类工程师真正的成长方式。

一个工程师不是因为看过更多日志才厉害，而是因为他形成了稳定的排查路径、测试习惯、风险判断和 review 标准。

Agent 也一样。

如果一次 CI 排查能被沉淀成 ci-debug.skill，如果一次 migration 事故能变成 migration-checklist，如果一次安全 review 能变成 auth-change-verifier，那么 Agent 才开始进入“会长经验”的阶段。

## 真正的分水岭：有没有 Skill Promotion

我认为真正的分水岭不是有没有 memory，而是有没有 Skill Promotion。

一个 Agent 如果只能记录事实，它最多像会做笔记的新人。

如果它能把一次成功经验变成以后自动执行的能力，它才开始成长。

可以粗略分成五层：

L1：能读规则。

L2：能维持上下文。

L3：能自动记忆。

L4：能生命周期治理。

L5：能技能进化。

大多数工具还在 L1 到 L3。

Codex、Claude Code、OpenCode、Kiro、Qwen Code 正在进入 L4。

Hermes 和 MiMo Code 最接近 L5。

## 为什么 Codex、Claude Code、OpenCode 值得改造

现有工具还不完美，但它们已经给了我们足够入口。

Codex 有 AGENTS.md、shell、workspace、任务上下文、技能体系、工具调用记录。

Claude Code 有 hooks、skills、commands、subagents。

OpenCode 有开放的 CLI 和可插拔工作流。

这些入口意味着：我们不一定要训练新模型，才能让 Agent 记得更好。

我们可以在运行时外接一套 Memory Runtime。

把关键生命周期事件接出来，把工具结果变成证据，把用户纠正变成高权重记忆，把重复流程升级成 skill。

这比单纯加长上下文更现实。

## 总结

很多 Coding Agent 都在说自己有 memory，但层次差异很大。

会读规则，不等于会长经验。

会保持上下文，不等于有长期记忆。

会自动总结，不等于记忆可信。

真正值得关注的是：它有没有生命周期治理，有没有证据链，有没有遗忘机制，有没有 Skill Promotion。

一句话总结：

> 大多数 Coding Agent 只是在记录，还没有真正长经验。

下一篇，我们就进入实现层：如何给 Codex 装一个“海马体”，用 Hook、Checkpoint 和 Verifier 做 Memory Runtime。

