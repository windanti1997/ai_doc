# 05 从踩坑到技能：让 Agent 不再重复犯错

副标题：经验如何升级成 Skill，并塑造 Agent 的工作风格。

建议封面文案：真正的记忆，是变成下一次的工作方式。

---

记录经历只是第一步。

真正的成长，是把经历变成技能。

更进一步，是让这些技能和记忆塑造 Agent 的工作风格。

也就是说：

```text
Episode -> Fact -> Rule -> Skill -> Style
```

很多 memory 系统到 project_memory.md 就停了。它会记录项目使用 pnpm、payment tests require Redis、generated files 不要手改。

这些有用，但它们只是事实。

一个新人也可以看项目文档。真正有经验的工程师，不只是知道事实，而是形成了做事方法。

## 只会记笔记的 Agent 仍然像新人

一个工程师真正有经验，不是因为他记住了更多日志。

而是因为他遇到问题时，会自然按一套稳定路径行动：

先复现问题。

再定位边界。

再看最近变更。

再写失败测试。

再做最小修改。

再跑验证。

再检查 diff。

最后提交 PR。

他不需要每次重新思考。

这就是程序记忆。

对应到 Agent，就是 skill、workflow、hook、command、subagent playbook。

## Episode -> Fact -> Rule -> Skill -> Automation

一个成熟的 Agent Memory 系统，应该有清晰的晋升链。

Episode 是一次经历。

比如某次 payment webhook test 失败，错误是 Redis 连接不上。

Fact 是被验证事实。

比如启动 Redis 后测试通过，因此可以记录 payment integration tests require Redis。

Rule 是项目规则。

比如运行 payment integration tests 前必须检查 Redis。

Skill 是可复用流程。

比如 payment-test-setup.skill，包含检查服务、启动依赖、运行测试、确认日志的完整步骤。

Automation 是自动化动作。

比如运行相关测试前，hook 自动检查 Redis 状态，失败时提醒 Agent 先启动依赖。

这条链路越往后，影响越大，也越需要谨慎。

## 什么经验值得升级成 Skill

不是所有经验都应该升级成 skill。

适合升级的经验通常有几个特征：

重复出现。

流程稳定。

结果可验证。

未来复用概率高。

错误成本高。

步骤明确。

作用域清晰。

比如 CI 排查、API 变更、migration 检查、权限 review、发布验证，就很适合 skill 化。

因为它们重复、可验证、风险高，而且每次重新摸索都很浪费。

不适合升级的内容包括：

一次性临时命令。

未验证猜测。

偶然 bug 修法。

旧架构 workaround。

用户某次临时偏好。

没有作用域的经验。

这些东西可以先作为 candidate memory 保留，但不要急着变成 skill。

## 错误经验也可能变成坏技能

Skill Promotion 最大风险，是把错误自动化。

一条错误事实，如果只是记在日志里，影响有限。

一条错误规则，如果进入 project_memory.md，会误导后续任务。

一个错误 skill，如果被 Agent 反复执行，就会持续制造问题。

更糟的是，如果这个 skill 被 hook 自动触发，错误会被放大。

所以升级前必须检查：

证据是否充分？

有没有通过测试验证？

作用域是否明确？

有没有反例？

是否会过度泛化？

是否有过期条件？

谁有权限修改或废弃它？

Agent Memory 如果没有 Curator，迟早会长出坏习惯。

## 记忆如何塑造 Agent 的工作风格

长期记忆不只是知识库，它还会塑造 Agent 的工作风格。

如果 memory 总是强化“不要冒险、先问用户、测试失败很危险”，Agent 会变得保守。

如果 memory 强化“小步迭代、先跑通、失败后复盘”，Agent 会变得探索。

如果 memory 强化“先读架构、先写测试、必须 verifier 通过”，Agent 会变得严谨。

如果 memory 强化“先看攻击面、检查权限、保护数据”，Agent 会变得安全敏感。

这意味着，Agent 的“性格”不一定只来自模型本身，也来自长期规则、失败复盘、verifier、skill library 和团队审查。

你给 Agent 什么记忆，它就会变成什么样的工程师。

## 给 Agent 什么记忆，它就会成为什么样的工程师

如果你只给它片段，它会变成资料检索器。

如果你只给它规则，它会变成手册执行器。

如果你给它经历和证据，它会开始理解项目。

如果你给它 skill 和 verifier，它会形成稳定工作方式。

如果你给它治理机制，它才可能进入真实团队。

第一系列讲到这里，其实已经完成一个闭环：

Agent Memory 不是 RAG。

Coding Agent 需要四层记忆。

Hook 是运行时入口。

证据和遗忘决定记忆质量。

Skill Promotion 决定 Agent 能否真正成长。

## 总结

真正强的 Agent，不是会写更多代码，而是会把经验变成稳定的工作方式。

Memory 的终点不是长期笔记，而是可复用技能。

一句话总结：

> Agent Memory 的价值不是保存过去，而是改变下一次行动。

第一系列到这里完成闭环：单个 Agent 如何从经历中成长。

后续可以继续讲团队版：个人经验如何升级成团队记忆，错误记忆如何防止污染组织。

