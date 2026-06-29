# 微信公众号完整稿：给 Coding Agent 装一个外置大脑

> 说明：本文件是第一系列公众号完整发布稿合集。原 `wechat_article_plan.md` 保留为选题与排版计划。

---

# 01 别再把 Agent 记忆做成向量库了

如果一个工程师每天都在重复踩同一个坑，你会觉得他聪明吗？

现在很多 Coding Agent 就处在这个阶段。

它能写代码，能改文件，能跑测试，能读日志，甚至能连续执行几十步任务。但它仍然经常像新人一样：刚踩过的坑，下次还会踩；刚确认过的测试命令，下次又忘；刚理解的项目约束，过几轮就丢。

所以问题不是 Agent 不够会写代码，而是它没有真正的经验系统。

很多人一提 Agent Memory，第一反应就是存聊天记录、存摘要、建向量库、下次检索出来。这当然有用，但它更像资料库，不是完整记忆。

真正值得借鉴的人类记忆，不是“大脑能存多少东西”，而是：大脑如何让过去经历影响未来行动。

人类记忆不是硬盘。它不是为了完整保存过去，而是为了预测未来、指导行动、形成习惯、塑造判断。

所以 Agent Memory 也不应该只回答“过去发生过什么”，而应该回答：什么值得记？这条记忆有多重要？它现在还成立吗？它会如何影响下一次行动？它能不能变成技能？

这就是 RAG 和 Memory 的区别。

RAG 是查资料。

Memory 是长经验。

一个成熟的 Agent Memory 系统至少需要几个机制：

第一，筛选。不是所有日志都值得存，只有高风险、反复出现、被用户纠正、被测试验证的信息才值得进入长期记忆。

第二，证据。好的记忆不是一句“测试需要 Redis”，而应该记录任务、命令、失败现象、根因、验证结果和相关文件。

第三，加权。Agent 没有情绪，但它需要重要性评分。线上事故、测试失败、用户明确纠正，权重应该高于普通提示。

第四，巩固。主 Agent 不应该边写代码边维护长期记忆。更合理的是用 Memory Writer、Dream、Distill、Curator 分工整理。

第五，遗忘。不会遗忘的 Agent，会被旧经验误导。代码库会变，旧测试命令、旧架构、旧 workaround 都可能过期。

第六，技能化。最高级的记忆不是 project_memory.md，而是 tdd-loop.skill、ci-debug.workflow、api-change.checklist。

所以，Agent Memory 的目标不是让 Agent 记住更多，而是让它下一次做得更好。

一句话总结：

> 别再把 Agent 记忆做成向量库了。Agent 需要的不是搜索系统，而是经验系统。

---

# 02 为什么你的 Agent 总像第一次见这个项目？

你明明告诉过它这个项目用 pnpm，不要用 npm。

你明明纠正过它不要改 generated 文件。

你明明让它跑过测试，也解释过这个模块的历史坑点。

但下一次任务开始，它还是像第一次见这个项目。

这不是单个工具的问题，而是现在很多 Coding Agent 的共同问题：它有上下文，但没有经验。

长上下文能让 Agent 看到更多，但看到更多不等于记得更好。把所有历史都塞进上下文，只会让重要信息被淹没，让临时猜测和稳定事实混在一起，让旧规则污染新任务。

真正有效的记忆，不是把更多东西放进去，而是把正确的东西，在正确的时机，用正确的方式召回。

一个 Coding Agent 至少需要四层记忆。

第一层是 Working Memory，当前任务状态。它包括当前目标、用户约束、todo、正在修改的文件、最近失败的测试、下一步计划和 checkpoint。它的目标是让任务不断线。

第二层是 Episodic Memory，过去发生过什么。它记录 session、tool call、命令结果、测试失败、错误栈、代码 diff、用户纠正和最终验证结果。它是经验的原材料。

第三层是 Semantic Memory，抽象出的项目知识。比如项目使用 pnpm，generated 文件不要手改，payment integration tests require Redis。这些知识比日志更稳定，但必须带 scope、evidence、confidence、last_verified。

第四层是 Procedural Memory，可复用技能。它不是“知道什么”，而是“怎么做”。比如 ci-debug.skill、api-change.workflow、migration-checklist。

有了四层还不够，还需要五个治理机制。

Write：什么值得写入。

Retrieve：什么时候召回。

Consolidate：如何从经历巩固成知识。

Forget：过期记忆如何降权或归档。

Promote：经验如何升级成 skill。

很多 Agent 总像第一次见项目，就是因为它最多有上下文和摘要，没有完整生命周期。它可能能检索，但不会巩固；能总结，但不会遗忘；能记住事实，但不会把流程变成技能。

一句话总结：

> Agent Memory 不是 RAG，而是经历到技能的生命周期系统。

---

# 03 我对比了十几个 Coding Agent，发现大多数还不会长经验

现在几乎所有 Coding Agent 都在说自己有上下文、有规则、有 memory。

但它们做的是同一件事吗？不是。

如果按照“会不会长经验”来分，主流工具可以分成五类：规则型、上下文型、自动记忆型、生命周期型、进化型。

规则型工具依赖 AGENTS.md、CLAUDE.md、GEMINI.md、.cursor/rules、copilot-instructions.md 等文件。它们像新人看手册，简单、可控、适合团队共享，但不会自己从失败中学习。

上下文型工具强在 IDE 和 workspace 感知，比如 Cursor、Windsurf、Cline、Roo Code。它们当前任务体验很好，但 session 结束后，很多经验没有沉淀。

自动记忆型工具开始自动总结用户偏好、项目习惯、测试命令和常见坑点。Claude Code、Codex、Cascade、Devin、Qwen Code 都在往这个方向走。问题是：自动记忆不等于正确记忆。它为什么记这条？证据是什么？是否还能删除、降权、修正？这些都需要治理。

生命周期型工具更进一步。Codex、Claude Code、OpenCode、Kiro、Qwen Code 提供 hooks、skills、subagents、session、compaction、commands 等机制。它们不只是存文件，而是能在 Agent 工作过程中插入逻辑。

这很重要，因为记忆不是任务结束后写一句 summary，而是应该发生在 SessionStart、UserPrompt、PostToolUse、PreCompact、Stop 等节点。

进化型工具最值得关注，比如 Hermes Agent、MiMo Code。它们开始做 Dream、Distill、Curator、Skill Learning，也就是把经历整理成知识，把知识提炼成技能。

真正的分水岭不是有没有 memory，而是有没有 Skill Promotion。

一个 Agent 如果不能把一次成功经验变成以后自动执行的能力，它最多是会记笔记。

如果能，它才开始成长。

可以粗略分成五层：

L1：能读规则。

L2：能维持上下文。

L3：能自动记忆。

L4：能生命周期治理。

L5：能技能进化。

大多数工具还在 L1 到 L3。Codex、Claude Code、OpenCode、Kiro、Qwen Code 正在进入 L4。Hermes 和 MiMo Code 最接近 L5。

但 L5 也不是终点。真正成熟的系统还需要团队级共享记忆、证据链、失效机制、权限控制、技能审查和行为风格管理。

一句话总结：

> 大多数 Coding Agent 只是在记录，还没有真正长经验。

---

# 04 给 Codex 装一个海马体

我们不一定要训练新模型，才能让 Agent 有更好的记忆。

更现实的方法，是用 Hook 给 Coding Agent 外挂一套 Memory Runtime。

这就像给 Codex 装一个海马体。

主 Agent 负责执行任务，Memory Runtime 负责记录、召回、验证、巩固和技能化。

为什么 Hook 是最佳切入口？

因为记忆不是一个文件。记忆发生在 Agent 工作过程里。

如果只在任务结束后让 Agent 总结一句“本次任务修复了 payment webhook 测试”，你会丢掉大量关键信息：它跑过哪些命令，哪些测试失败，哪些方案试过但失败，哪个错误栈是关键，哪个文件改动导致结果变化，它是否真的验证通过。

Hook 的价值在于，它能在 Agent 行为发生的现场记录证据。

一个最小 Memory Runtime 可以这样设计：

SessionStart：启动时加载最小必要记忆，比如项目规则、近期 checkpoint、已验证事实、高风险模块坑点。

UserPrompt：根据用户任务召回相关经验。比如用户说修 payment webhook test，就召回 webhook signature、integration test、Redis 依赖等相关记忆。

PostToolUse：把工具结果写成情景记忆。命令是否成功、测试是否通过、错误栈是什么、diff 改了什么，这些都是真实证据。

PreCompact：长任务压缩前写 checkpoint，保存用户目标、当前状态、活跃约束、修改文件、测试结果、失败尝试、下一步动作。

Stop：防止 Agent 假完成。如果改了代码但没跑测试，测试失败却准备总结，修改 API 但没更新 schema，就不允许停止。

AfterStop：任务结束后生成 memory candidates。哪些是已验证事实，哪些是失败路径，哪些是 known pitfalls，哪些可能升级成 skill。

然后再用 Dream、Distill、Curator 做离线整理。

Dream 负责从日志堆里长出知识。

Distill 负责把重复流程提炼成 skill。

Curator 负责清理过期记忆和坏 skill。

一个 MVP 目录可以这样：

```text
memory/
  working/checkpoint.md
  episodic/events.jsonl
  semantic/project_memory.md
  semantic/known_pitfalls.jsonl
  procedural/skills/
  user/preferences.md
```

先实现五件事就够了：启动加载项目记忆、用户输入检索相关经验、工具调用后记录结果、压缩前写 checkpoint、停止前 verifier 防假完成。

一句话总结：

> Hook 是 Agent Memory 的生命周期触发器。

---

# 05 从踩坑到技能：让 Agent 不再重复犯错

记录经历只是第一步。

真正的成长，是把经历变成技能。

更进一步，是让这些技能和记忆塑造 Agent 的工作风格。

也就是说：

```text
Episode → Fact → Rule → Skill → Style
```

很多 memory 系统到 project_memory.md 就停了。它会记录项目使用 pnpm、payment tests require Redis、generated files 不要手改。

这些有用，但它们只是事实。

一个新人也可以看项目文档。真正有经验的工程师，不只是知道事实，而是形成了做事方法。

比如遇到 bug，他会自然按这个路径走：复现问题、定位边界、找最近变更、写失败测试、最小修改、跑验证、检查 diff、提交 PR。

他不需要每次重新思考。

这就是程序记忆。

对应到 Agent，就是 skill、workflow、hook、command、subagent playbook。

一个成熟的 Agent Memory 系统，应该有明确的晋升链：

Episode：一次经历。

Fact：被验证事实。

Rule：项目规则。

Skill：可复用流程。

Automation：自动化动作。

比如一次 payment webhook test 因 Redis 未启动失败，只是 episode。多次验证后，可以形成 fact：payment integration tests require Redis。长期有效后，可以写成 rule。流程稳定后，可以升级成 payment-test-setup.skill。再进一步，可以变成运行 integration test 前自动检查 Redis 的 hook。

但不是所有经验都应该升级成 skill。

适合升级的经验通常重复出现、流程稳定、结果可验证、未来复用概率高、错误成本高、步骤明确。

不适合升级的内容包括一次性临时命令、未验证猜测、偶然 bug 修法、旧架构 workaround、用户临时偏好。

更要注意：错误经验一旦变成 skill，污染会被放大。

所以 Skill Promotion 前必须检查证据是否充分、是否有明确作用域、是否有反例、是否经过测试验证、是否会过度泛化。

长期记忆还会塑造 Agent 的工作风格。

如果 memory 总是强化“不要冒险、先问用户、测试失败很危险”，它会变成保守型 Agent。

如果强化“小步迭代、先跑通、失败后复盘”，它会变成探索型 Agent。

如果强化“先读架构、先写测试、必须 verifier 通过”，它会变成严谨型 Agent。

如果强化“先看攻击面、检查权限、保护数据”，它会变成安全型 Agent。

所以，Agent 的“性格”不一定来自模型本身，也可能来自长期规则、长期记忆、失败复盘、verifier 和 skill library。

一句话总结：

> 真正强的 Agent，不是会写更多代码，而是会把经验变成稳定的工作方式。
