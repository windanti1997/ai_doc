# 我对比了十几个 Coding Agent，发现大多数还不会长经验

> Codex、Claude Code、OpenCode、Hermes、MiMo 谁真正开始做记忆系统？

上一篇我们提出了一个判断框架：

```text
Working Memory：当前任务状态
Episodic Memory：历史经历
Semantic Memory：项目知识
Procedural Memory：可复用技能
```

再加上五个治理机制：

```text
Write / Retrieve / Consolidate / Forget / Promote
```

现在我们用这套框架回头看主流 Coding Agent。

它们到底做到了哪一步？

谁只是“看起来有记忆”？

谁开始真正把经历变成知识和技能？

## 一、先给结论：大多数工具还停在“经历 → 记录”

如果把 Agent 成长拆成五步：

```text
经历
  ↓
记录
  ↓
整理
  ↓
抽象
  ↓
固化
```

那么大多数 Coding Agent 还停在前两步。

它们能记录上下文。

能保存规则。

能读项目文件。

能维护当前 session。

但还没有稳定做到：

- 从失败中抽象规律；
- 从规律中形成技能；
- 定期清理过期记忆；
- 用证据判断记忆是否仍然有效；
- 把个人经验沉淀为团队资产。

所以，今天很多工具的 memory 更像：

```text
上下文增强
规则注入
历史摘要
```

还不是完整的经验系统。

## 二、五类 Coding Agent 记忆系统

为了避免做成工具百科，可以把主流工具分成五类：

```text
规则型
上下文型
自动记忆型
生命周期型
进化型
```

它们不是绝对分类，而是成熟度阶段。

## 三、规则型：会读手册，但不会长经验

代表工具：

```text
Aider
Gemini CLI
Cursor Rules
Trae Rules
GitHub Copilot instructions
```

规则型记忆的核心是：

> 把项目规则写进文件，让 Agent 每次读取。

常见形式包括：

- AGENTS.md
- CLAUDE.md
- GEMINI.md
- QWEN.md
- .cursor/rules
- .trae/rules
- copilot-instructions.md
- CONVENTIONS.md

它解决的问题是：项目怎么启动、测试怎么跑、代码风格是什么、哪些文件不要改、PR 要满足什么要求、团队开发规范是什么。

这类方式很实用。

优点是简单、可控、可版本管理、适合团队共享、不依赖复杂系统。

但问题也明显：

- 需要人工维护；
- 容易过期；
- 不知道哪些规则真的有效；
- 不会从失败中自动学习；
- 不会把重复流程升级成 skill。

所以规则型 Agent 像什么？

像一个新人。

它会看手册。

但它不会自己长经验。

> 规则文件是 Agent Memory 的地基，但不是完整记忆系统。

## 四、上下文型：短期记忆很好，但不一定长期成长

代表工具：

```text
Cursor
Windsurf / Cascade
Trae
Cline
Roo Code
```

这类工具强在 IDE 体验和上下文感知。

它们能看到当前文件、打开的 tab、代码索引、当前对话、最近编辑、项目结构、用户选中的代码。

这让它们在当前任务里表现很好。

它们能连续改几个文件，能追踪当前上下文，能理解当前 workspace。

但问题是：

> session 结束后，很多经验没有沉淀。

上下文型 Agent 更像一个短期记忆很好的人。

它知道你刚刚说了什么。

知道你刚刚改了什么。

但不一定会把这次经验变成长期知识。

当然，一些工具已经加入 Memories 和 Rules。

这说明它们正在从上下文型向自动记忆型靠近。

但普遍仍然缺少证据链、失效机制、skill promotion、定期巩固、团队级记忆治理。

> 上下文能让 Agent 连续工作，但不等于让 Agent 积累经验。

## 五、自动记忆型：会做笔记，但不一定会治理

代表工具：

```text
Claude Code
Codex
Windsurf / Cascade
Devin
Qwen Code
```

自动记忆型的特点是：

> Agent 会自动抽取一些长期信息。

比如用户偏好、项目习惯、测试命令、常见坑点、代码风格、工作方式。

这已经比规则型更进一步。

因为用户不需要每次手动更新规则文件。

系统会尝试从使用中学习。

Claude Code 有 CLAUDE.md 和 auto memory。

Codex 有 AGENTS.md、Memories 和 Skills。

Cascade 有自动 memories 和 rules。

Devin 强调团队知识和项目知识。

Qwen Code 也开始强调 Auto-Memory、Auto-Skills、SubAgents、Agent Teams。

这些方向都说明一个趋势：

> Agent 开始从“只读规则”进入“自动总结”。

但自动记忆也带来新问题：

- 它为什么记这条？
- 这条记忆证据是什么？
- 是否仍然有效？
- 是否会污染下次任务？
- 用户是否知道它记了什么？
- 是否能删除、修正、降权？
- 是否能升级成 skill？

所以自动记忆的关键不是“自动”。

而是“可治理”。

> 会自动记，不代表会正确记。

## 六、生命周期型：开始管理任务过程

代表工具：

```text
Codex
Claude Code
OpenCode
Kiro
Qwen Code
```

生命周期型 Agent 的关键能力不是 memory 本身，而是 runtime。

它们提供了 hooks、skills、subagents、session、compaction、commands 等机制，让开发者可以在 Agent 工作过程中插入逻辑。

这很重要。

因为记忆不是任务结束后写一句 summary。

记忆应该发生在整个生命周期里。

比如：

```text
SessionStart：加载相关记忆
UserPrompt：召回历史经验
PostToolUse：记录工具结果
PreCompact：写 checkpoint
Stop：检查是否真的完成
SubagentStop：收集子 agent 结论
```

这类工具开始具备搭建 memory runtime 的基础。

Codex 很适合做 hook-based memory harness。

Claude Code 的 hooks、skills、subagents 生态也很成熟。

OpenCode 胜在开放，适合自定义 plugin。

Kiro 的 spec、steering、hook 路线，把项目记忆和工程流程绑定得比较紧。

Qwen Code 则在开源路线里强调 auto-memory、auto-skills 和 agent teams，值得持续观察。

生命周期型的关键价值是：

> 它不再把记忆当成一个文件，而是把记忆嵌入 Agent 的工作过程。

它还没有天然解决所有问题。

但它给了我们改造入口。

> Hook 是从“会记录”走向“会治理”的关键接口。

## 七、进化型：开始把经验变成技能

代表工具：

```text
Hermes Agent
MiMo Code
Qwen Code
```

进化型 Agent 最值得关注。

因为它们不只做 memory。

它们开始做：

```text
Dream
Distill
Curator
Skill Learning
Auto-Skills
Self-improvement
```

Hermes Agent 的特点是：有持久记忆、有用户与项目记忆文件、能搜索过去会话、会从经验中创建 skill、会改进 skill、有 curator 整理技能库。

它更像一个会长期成长的个人 Agent。

MiMo Code 的特点是：面向 long-horizon coding，有 session checkpoint、project memory、global memory、history trace、Dream 做记忆整理、Distill 把重复流程提炼为 skill / command / subagent。

它更像一个专门为长任务 coding 设计的 memory runtime。

这类系统真正跨过了一个分水岭：

> 从“记住发生过什么”，走向“把发生过的事变成以后怎么做”。

它们最接近这个系列说的“会成长的 Agent”。

但也要注意：

进化型 memory 风险更高。

因为一旦错误经验被提炼成 skill，污染会被放大。

所以进化型系统必须配套证据链、验证机制、失效机制、用户审查、skill curator、安全边界、权限控制。

> 经验能变成能力，也可能把错误变成习惯。

## 八、横向对比：谁做到哪一层？

| 工具 | 主要阶段 | 强项 | 短板 |
|---|---|---|---|
| Aider | 规则型 | 简单、稳定、可控 | 长期经验弱 |
| Gemini CLI | 规则 + 长上下文 | 大上下文和工具调用 | 记忆生命周期弱 |
| Cursor | 上下文型 | IDE 体验和项目索引强 | 技能化弱 |
| Windsurf / Cascade | 上下文 + 自动记忆 | memories + rules | 记忆治理仍有限 |
| Trae | 上下文 + rules | 工具链和多轮任务 | 长期语义/技能层弱 |
| Cline / Roo Code | Memory Bank | 结构化笔记实用 | 自动验证和遗忘弱 |
| Claude Code | 自动记忆 + 生命周期 | hooks/skills/subagents 完整 | memory 仍容易 markdown 堆积 |
| Codex | 生命周期型 | AGENTS.md、Memories、Skills、Hooks 组合强 | 需要外接治理系统 |
| OpenCode | 开放底座 | plugins/skills 可扩展 | 默认长期 memory 弱 |
| Kiro | spec/steering/hook | 把流程和记忆绑定 | 仍偏工程规范 |
| Devin | 云端工程师 | 团队知识和项目上下文 | 可审计和迁移问题 |
| Hermes | 进化型 | self-improving skills | 需要防止错误技能化 |
| MiMo Code | 进化型 | checkpoint + Dream + Distill | 机制复杂，治理要求高 |
| Qwen Code | 进化型潜力 | Auto-Memory / Auto-Skills / Agent Teams | 新，需要更多验证 |

## 九、真正的分水岭：有没有 Skill Promotion

判断一个 Agent 是否真的会长经验，可以问一个问题：

> 它能不能把一次成功经验变成以后自动执行的能力？

如果不能，它最多是会记笔记。

如果能，它才开始成长。

可以分成五个层次：

```text
L1：能读规则
L2：能维持上下文
L3：能自动记忆
L4：能生命周期治理
L5：能技能进化
```

大多数工具在 L1-L3。

Codex、Claude Code、OpenCode、Kiro、Qwen Code 正在进入 L4。

Hermes 和 MiMo Code 最接近 L5。

但 L5 不是终点。

真正成熟的系统还需要团队级共享记忆、证据链、失效机制、权限控制、技能审查、行为风格管理。

这一层目前还没有完全成熟的产品。

这也是机会所在。

## 十、结语：大多数 Agent 还没真正长经验

这一篇真正想回答的不是：哪个工具最强？

而是：谁已经开始成长？

答案很清楚：

- 大多数工具还在记录；
- 少数工具开始整理；
- 更少数工具开始抽象；
- 极少数工具开始技能化。

真正的竞争点不是更长上下文，也不是更多工具。

而是：

> 谁能把经验变成能力。
