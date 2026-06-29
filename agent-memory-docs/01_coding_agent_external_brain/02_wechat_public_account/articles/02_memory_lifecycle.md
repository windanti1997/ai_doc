# 02 为什么你的 Agent 总像第一次见这个项目？

副标题：Coding Agent 需要的不是更长上下文，而是记忆生命周期。

建议封面文案：长上下文不等于长期记忆。

---

你明明告诉过它这个项目用 pnpm，不要用 npm。

你明明纠正过它不要改 generated 文件。

你明明让它跑过测试，也解释过这个模块的历史坑点。

但下一次任务开始，它还是像第一次见这个项目。

这不是单个工具的问题，而是很多 Coding Agent 的共同问题：它有上下文，但没有经验。

## 长上下文不是长期记忆

长上下文能让 Agent 看到更多，但看到更多不等于记得更好。

把所有历史都塞进上下文，只会带来三个问题。

第一，重要信息会被淹没。一次关键用户纠正，可能和几十段普通对话混在一起。

第二，临时信息和稳定事实会混在一起。某个临时 workaround 只适用于当天环境，却可能被 Agent 当成长期规则。

第三，旧经验会污染新任务。项目升级后，旧命令、旧目录、旧接口可能已经失效。

所以长期记忆不是“把窗口变大”，而是“把经历变成可治理的经验”。

## Coding Agent 至少需要四层记忆

一个真正能长经验的 Coding Agent，至少需要四层记忆。

第一层是 Working Memory。

这是当前任务状态：用户目标、活跃约束、todo、正在修改的文件、最近失败的测试、下一步计划、checkpoint。它的作用是让任务不断线。

第二层是 Episodic Memory。

这是过去发生过什么：session、tool call、命令结果、测试失败、错误栈、代码 diff、用户纠正、最终验证结果。它是经验的原材料。

第三层是 Semantic Memory。

这是抽象出的项目知识：项目使用 pnpm，generated 文件不要手改，payment integration tests require Redis，某个 API 改动必须同步 schema。这些知识比日志更稳定，但必须带作用域、证据、可信度和最后验证时间。

第四层是 Procedural Memory。

这是可复用技能：ci-debug.skill、api-change.workflow、migration-checklist、release-verify.playbook。它不是“知道什么”，而是“怎么做”。

很多 Agent 总像第一次见项目，就是因为它最多有上下文和摘要，没有这四层结构。

## 记忆必须带证据、作用域和状态

同一句“不要改 generated 文件”，如果没有作用域，就会出问题。

是所有 generated 文件都不能改？

还是某个目录不能手改？

有没有例外？

这条规则来自用户提醒、项目文档、失败测试，还是模型猜测？

什么时候最后验证过？

有没有被新流程替代？

一条可用的 Agent memory，至少应该包含：

scope：适用于用户、项目、模块、目录、命令还是团队。

evidence：来源是什么，有没有命令结果、文件路径、PR review、测试日志。

confidence：可信度多高。

status：active、candidate、stale、deprecated、invalidated。

last_verified：最后一次验证时间。

without evidence 的记忆很危险。它看起来像经验，实际上可能只是模型把几段上下文混合后的自我感觉。

## 没有遗忘机制的记忆，迟早会污染任务

很多人担心 Agent 忘东西。

但在真实工程里，记错比忘记更危险。

一个旧测试命令如果一直被召回，会让 Agent 反复跑错。

一个旧架构 workaround 如果一直被高权重保留，会让 Agent 在新架构下做出错误修改。

一个个人偏好如果被当成团队规则，会影响其他人。

所以 Agent Memory 必须有遗忘机制。

遗忘不是删除一切，而是降权、标记过期、等待重新验证、被新证据覆盖。

这和工程里的缓存很像。缓存不是永远正确，必须有 TTL、失效条件、刷新机制和回滚策略。

Memory 也是一样。

## Promote：从经历升级成技能

如果一个 Agent 每次调 CI 都重新摸索，它只是有日志。

如果它能把反复出现的 CI 排查流程变成 ci-debug.skill，它才开始有经验。

Promote 是记忆生命周期里最关键的一步。

一次经历只是 episode。

多次重复后，可以变成 fact。

被验证、被限定作用域后，可以变成 rule。

流程稳定、可复用后，可以变成 skill。

甚至可以进一步变成 hook 或 automation。

但 Promote 必须谨慎。错误经验升级成 skill，会把错误自动化。临时修法升级成 workflow，会污染未来任务。

所以每次升级都要问：

它重复出现了吗？

它有清晰步骤吗？

它有验证结果吗？

它的作用域明确吗？

有没有反例？

未来复用概率高吗？

错误成本高吗？

如果答案成立，才值得升级。

## 一个最小可行的记忆生命周期

给 Coding Agent 做 memory，不需要一开始就建复杂平台。

最小可行版本可以很简单：

任务开始时，加载项目规则和近期 checkpoint。

用户输入时，根据任务检索相关 memory。

工具调用后，记录命令、结果、错误栈、修改文件。

上下文压缩前，写入 checkpoint。

任务停止前，检查是否真的验证完成。

任务结束后，从日志里生成候选记忆。

离线阶段，把候选记忆整理成事实、规则、skill。

这就是从执行到记忆、从记忆到经验、从经验到技能的闭环。

## 总结

你的 Agent 总像第一次见这个项目，不是因为它没有看见足够多的信息，而是因为它没有把信息沉淀成经验。

长上下文解决不了长期成长。

Coding Agent 需要的是四层记忆：Working、Episodic、Semantic、Procedural。

还需要五个治理动作：Write、Retrieve、Consolidate、Forget、Promote。

一句话总结：

> Agent Memory 不是 RAG，而是经历到技能的生命周期系统。

下一篇，我们就用这个框架去看主流 Coding Agent：到底谁只是在记录，谁真的开始长经验？

