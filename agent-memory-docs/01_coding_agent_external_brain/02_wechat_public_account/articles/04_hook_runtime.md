# 04 给 Codex 装一个海马体

副标题：用 Hook、Checkpoint 和 Verifier 做 Agent Memory Runtime。

建议封面文案：Hook 是 Agent Memory 的生命周期触发器。

---

我们不一定要训练新模型，才能让 Agent 有更好的记忆。

更现实的方法，是用 Hook 给 Coding Agent 外挂一套 Memory Runtime。

这就像给 Codex 装一个海马体。

主 Agent 负责执行任务，Memory Runtime 负责记录、召回、验证、巩固和技能化。

## 为什么 Hook 比事后总结更可靠

很多 memory 系统喜欢在任务结束后总结：

“本次任务修复了 payment webhook 测试。”

这当然有用，但远远不够。

因为真正有价值的信息，往往发生在过程里。

它跑过哪些命令？

哪些测试失败？

错误栈是什么？

哪个方案试过但失败？

哪个文件改动导致结果变化？

用户在哪一步纠正了它？

最终有没有验证通过？

如果只靠事后总结，这些证据很容易丢失，或者被模型压缩成看似合理但不够可信的结论。

Hook 的价值在于：它能在 Agent 行为发生的现场记录证据。

## SessionStart：启动时加载最小必要记忆

任务开始时，不应该把所有 memory 都塞给 Agent。

更好的方式是加载最小必要记忆。

比如：

项目级规则。

用户长期偏好。

最近 checkpoint。

高风险模块坑点。

和当前任务强相关的已验证事实。

Memory 的召回应该像工程里的依赖加载：够用、准确、低噪音。

如果一开始塞进几十条无关经验，Agent 反而更容易被误导。

## UserPrompt：按任务召回相关经验

用户输入是第二个关键节点。

当用户说“修 payment webhook test”，Memory Runtime 应该召回 webhook signature、integration test、Redis 依赖、最近失败日志、相关 skill。

当用户说“改 API”，应该召回 schema、SDK、兼容性、文档、测试 checklist。

当用户说“做 migration”，应该召回 rollback、数据校验、线上风险、灰度流程。

这不是简单关键词搜索，而是带作用域和风险权重的召回。

真正有用的 memory，不是一直存在，而是在正确时机出现。

## PostToolUse：工具结果是最可靠的证据

对 Coding Agent 来说，工具调用是最重要的记忆来源。

命令是否成功，测试是否通过，错误栈是什么，diff 改了什么，这些都是真实证据。

PostToolUse 可以把每次工具调用写成 episodic memory：

命令：pnpm test payment-webhook。

结果：失败。

错误：ECONNREFUSED Redis。

相关文件：tests/payment/webhook.spec.ts。

后续动作：启动 Redis 后重跑通过。

验证状态：passed。

这比一句“payment test 需要 Redis”可靠得多。

因为它知道这条记忆从哪里来。

## PreCompact：长任务压缩前必须写 checkpoint

长任务最怕上下文压缩。

压缩一旦丢掉关键状态，Agent 就会出现奇怪行为：重复尝试已失败方案、忘记用户约束、忘记哪些文件改过、忘记下一步该做什么。

所以 PreCompact 应该写 checkpoint。

一个好的 checkpoint 至少包含：

用户目标。

当前状态。

活跃约束。

修改文件。

已运行命令。

测试结果。

失败尝试。

下一步计划。

未解决风险。

这不是为了写漂亮 summary，而是为了让任务不断线。

## Stop：防止 Agent 假完成

Stop hook 的作用是防止 Agent 在不该结束时结束。

比如：

改了代码但没跑测试。

测试失败却准备总结。

改了 API 但没更新 schema。

改了迁移但没检查 rollback。

修改了配置但没确认文档。

这些情况都应该被 verifier 拦住。

Agent 很容易因为“看起来完成”而停止，但工程任务需要“被验证完成”。

Stop verifier 就是把这个标准固化下来。

## Dream / Distill / Curator：离线巩固和技能化

主 Agent 不应该一边写代码一边维护长期记忆。

更好的方式是分工：

Memory Writer 负责把事件写成结构化记录。

Dream 负责从日志堆里整理出候选知识。

Distill 负责把重复流程提炼成 skill。

Curator 负责清理过期记忆、冲突记忆和坏 skill。

这和人类学习也很像。白天做事，晚上整理；先有经历，再有抽象；先形成规则，再变成习惯。

## 一个 MVP 目录

一个最小 Memory Runtime 可以从文件系统开始：

```text
memory/
  working/checkpoint.md
  episodic/events.jsonl
  semantic/project_memory.md
  semantic/known_pitfalls.jsonl
  procedural/skills/
  user/preferences.md
```

先不用上复杂数据库。

先把生命周期跑通：启动加载、输入召回、工具记录、压缩 checkpoint、停止 verifier、结束后候选记忆、离线整理。

等流程稳定，再考虑索引、向量检索、权限、多 Agent 共享。

## 总结

Agent Memory 不是一个文件，也不是一个向量库。

它应该是一套运行时。

Hook 是最好的切入口，因为它连接了 Agent 的真实行为过程。

一句话总结：

> Hook 是 Agent Memory 的生命周期触发器。

下一篇，我们讲最后一环：如何把踩坑经验升级成 Skill，让 Agent 不再重复犯错。

