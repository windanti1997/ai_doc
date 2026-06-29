# 为什么你的 Agent 总像第一次见这个项目？

> Coding Agent 需要的不是更长上下文，而是记忆生命周期。

上一篇我们讲了一个核心判断：

> Agent Memory 不是 RAG，也不是向量库，而是一套让经历影响未来行动的经验系统。

那问题来了：

如果把这个判断落到 Coding Agent 上，一个真正有记忆的 Agent 应该长什么样？

很多人现在用 Coding Agent，会有一种很强的割裂感。

它在单轮任务里越来越强。

能读代码，能改文件，能跑测试，能分析报错。

但一旦换个 session，或者任务变长，它又像第一次见这个项目。

你已经告诉过它：

- 这个项目用 pnpm，不要用 npm；
- integration test 要先启动 Redis；
- generated 文件不要直接改；
- backend 改接口后要同步更新前端类型；
- 这个模块之前踩过 webhook signature 的坑。

但下次它可能还是忘。

于是很多人说：给它加 RAG。

把聊天记录存起来，把项目文档存起来，把历史摘要塞进向量库。

这当然能缓解问题。

但它仍然不够。

因为 Coding Agent 缺的不是一个“能搜索历史的资料库”。

它缺的是一套能让经验持续生效的 memory lifecycle。

---

## 一、RAG 不等于 Agent Memory

RAG 解决的是：

```text
我需要时，能不能找到相关资料？
```

Agent Memory 解决的是：

```text
过去的经历，能不能改变我下一次怎么做？
```

这两个问题差很远。

RAG 更适合静态知识问答。

比如：

- 某个 API 怎么用；
- 某个文档里写了什么；
- 某个历史记录里有没有提过某件事。

但 Coding Agent 面对的是动态工程环境。

代码会变。

测试会变。

依赖会变。

架构会重构。

过去正确的经验，未来可能过期。

所以，Coding Agent 的记忆不能只问“能不能检索出来”。

它还必须问：

- 这条记忆适用于哪个 repo？
- 适用于哪个模块？
- 证据是什么？
- 最后一次验证是什么时候？
- 现在是否还成立？
- 有没有被新代码推翻？
- 是否应该升级成规则或 skill？

这就是 RAG 和 Agent Memory 的分界线。

> RAG 是查资料。Agent Memory 是长经验。

---

## 二、长上下文也不是完整记忆

还有一种常见想法是：只要上下文足够长，Agent 就不会失忆。

这个判断也不完全对。

长上下文能让 Agent “看到更多”。

但看到更多，不等于记得更好。

把所有历史都塞进上下文，会带来几个问题：

- 重要信息被淹没；
- 旧约束和新约束混在一起；
- 临时猜测被误当成事实；
- token 成本上升；
- 模型注意力被稀释；
- 过期信息污染当前决策。

这就像让一个工程师每次开工前读完整个项目历史群聊。

信息很多。

但不一定有用。

真正有效的记忆，应该是经过筛选、压缩、验证和组织的。

也就是说：

```text
原始历史 ≠ 记忆
长上下文 ≠ 经验
摘要 ≠ 知识
```

记忆不是“把更多东西放进去”。

而是“把正确的东西，在正确的时机，用正确的方式召回”。

---

## 三、Coding Agent 至少需要四层记忆

如果参考人类记忆，一个 Coding Agent 至少需要四层：

```text
Working Memory
Episodic Memory
Semantic Memory
Procedural Memory
```

这四层分别回答四个问题：

```text
我现在在做什么？
过去发生过什么？
我从过去抽象出了什么稳定知识？
我已经学会了哪些可复用流程？
```

它们不能混成一份大文档。

因为它们的生命周期完全不同。

---

## 四、Working Memory：当前任务不断线

Working Memory 负责当前正在做什么。

对 Coding Agent 来说，它包括：

- 当前目标；
- 用户约束；
- 当前 todo；
- 正在修改的文件；
- 最近工具结果；
- 当前失败的测试；
- 下一步计划；
- 长任务 checkpoint。

比如：

```text
当前目标：修复 payment webhook integration test
当前约束：不能改 public API
当前状态：已修改 webhook parser，测试仍失败
最近失败：Stripe signature verification failed
下一步：检查 raw body 是否被提前 JSON parse
```

这类信息不一定长期有效。

但在当前任务里极其重要。

如果上下文压缩后丢掉这些，Agent 就会漂。

所以 Working Memory 的核心目标是：

> 保持当前任务不断线。

对应工程实现可以是：

```text
current_goal.md
active_constraints.md
todo.md
checkpoint.md
recent_tool_results
```

---

## 五、Episodic Memory：记录真实经历

Episodic Memory 负责记录具体经历。

对 Coding Agent 来说，它包括：

- session log；
- tool call；
- 命令结果；
- 测试失败；
- 错误栈；
- 代码 diff；
- 用户纠正；
- 任务复盘；
- 失败路径；
- 最终验证结果。

比如：

```text
在某次任务中，payment webhook test 失败。
命令：pnpm test payment-webhook.test.ts
现象：Redis connection refused
处理：启动 Redis 后测试通过
结论：该 integration test 依赖 Redis
```

Episodic Memory 是原材料。

它不一定每次都注入上下文，但它必须保留。

因为很多后续知识都来自它。

如果没有情景记忆，Agent 就只能靠当前上下文重新推理。

这就是为什么它会重复踩坑。

> 情景记忆记录发生过什么，语义记忆才总结它意味着什么。

---

## 六、Semantic Memory：沉淀项目知识

Semantic Memory 负责把多次经历压缩成稳定知识。

对 Coding Agent 来说，它包括：

- 项目架构；
- 技术栈；
- 运行命令；
- 测试规则；
- 代码规范；
- 已验证事实；
- 已知坑点；
- 用户偏好；
- 架构决策；
- 禁止事项。

比如：

```text
本项目使用 pnpm，不使用 npm。
payment integration tests require Redis。
generated API files 不要直接手改。
后端接口变更后必须更新 OpenAPI schema。
```

这类记忆应该比 episodic memory 更短、更稳定、更高价值。

但它不能只是 markdown 里的几句话。

它必须有 metadata：

```text
scope
evidence
confidence
last_verified
invalidated_by
```

否则它很容易变成“看起来正确的旧规则”。

Semantic Memory 的目标是：

> 让 Agent 不用每次都从日志中重新总结项目经验。

---

## 七、Procedural Memory：把经验变成可复用技能

Procedural Memory 是最容易被忽略的一层。

它不是“知道什么”。

而是“怎么做”。

对 Coding Agent 来说，它包括：

- skills；
- workflows；
- hooks；
- commands；
- checklists；
- subagent playbooks；
- 自动化脚本。

比如：

```text
tdd-loop.skill
ci-debug.skill
api-change.workflow
migration-checklist
release-command
security-review-agent
```

一个 Agent 如果只记住：

```text
测试前要启动 Redis
```

它只是知道一个事实。

但如果它形成了流程：

```text
运行 integration test 前：
1. 检查依赖服务
2. 检查环境变量
3. 启动 Redis
4. 再运行测试
```

它才真的长了经验。

Procedural Memory 的价值在于：

> 把重复推理压缩成稳定动作。

真正会成长的 Agent，不应该只写 project_memory.md。

它应该把高频成功流程升级成 skill。

---

## 八、四层记忆之外，还需要五个治理机制

光有四层还不够。

因为记忆不是静态文件。

它需要生命周期。

一个成熟的 Agent Memory 至少需要五个治理机制：

```text
Write
Retrieve
Consolidate
Forget
Promote
```

这五个机制，决定 memory 能不能长期可用。

---

## 九、Write：什么值得写入？

不是所有信息都值得记。

Agent 每次任务都会产生大量日志。

如果全部写入，memory 会很快污染。

所以写入时要判断：

- 是否影响未来任务？
- 是否被验证？
- 是否重复出现？
- 是否来自用户明确纠正？
- 是否是高风险事件？
- 是否代表稳定偏好？
- 是否能指导下一次行动？

Write 的关键不是存储能力，而是准入判断。

> 记忆的第一步不是写入，而是选择。

---

## 十、Retrieve：什么时候召回？

记忆不能全量注入。

全量注入会浪费 token，也会稀释注意力。

Agent 应该按线索召回。

线索包括：

- 用户目标；
- 当前文件；
- 模块路径；
- 错误栈；
- 测试名称；
- 最近 diff；
- git branch；
- 任务类型。

比如用户说：

> 修一下 payment webhook 测试。

系统应该召回：

```text
payment 模块历史坑点
webhook signature 相关经验
integration test 依赖服务
相关测试命令
```

而不是召回整个项目历史。

> 好的召回，不是最相似，而是当前最有用。

---

## 十一、Consolidate：如何巩固？

一次经历只是日志。

多次重复才是规律。

巩固的作用是：

```text
episode → fact → rule
```

比如：

```text
第一次：Redis 没启动导致测试失败
第二次：同类测试再次失败
第三次：确认该类测试依赖 Redis
巩固：写入 verified_facts
```

Consolidate 可以在任务结束时做，也可以定期做。

它对应人类记忆里的“睡眠整理”。

对 Agent 来说，就是：

- 合并重复事件；
- 删除临时猜测；
- 提炼稳定事实；
- 更新 project memory；
- 生成 known pitfalls；
- 标记可能过期的内容。

> 没有巩固，memory 只是日志堆。

---

## 十二、Forget：如何遗忘？

不会遗忘的 Agent，会被旧经验误导。

Coding Agent 特别需要遗忘，因为代码一直变。

遗忘不一定是删除。

可以是：

- 降权；
- 归档；
- 标记 stale；
- 等待重新验证；
- 被新规则覆盖；
- 限制只在历史查询时出现。

每条长期记忆都应该有：

```text
confidence
last_verified
expires
stale
archived
invalidated_by
superseded_by
```

比如：

```text
旧规则：测试必须连接 Redis
新事实：测试迁移到 mock Redis
处理：旧规则 superseded，新规则生效
```

> 长期记忆如果没有失效机制，迟早会变成长期污染。

---

## 十三、Promote：如何升级成技能？

Promote 是最重要、也最容易被忽略的机制。

因为它决定 Agent 是否真的“成长”。

记忆的晋升链应该是：

```text
Episode
  ↓
Fact
  ↓
Rule
  ↓
Skill
  ↓
Automation
```

比如：

```text
一次测试失败 → 情景记忆
多次失败共性 → known pitfall
被验证事实 → project memory
固定处理流程 → skill
可自动执行 → hook / command
```

这就是从“记住”到“会做”。

没有 Promote，Agent 只是一个会写笔记的助手。

有 Promote，Agent 才可能变成越来越熟练的工程搭档。

> Agent Memory 的终点不是长期笔记，而是可复用技能。

---

## 十四、最终公式

一个会成长的 Coding Agent，记忆系统可以写成一个公式：

```text
Agent Memory
= Working + Episodic + Semantic + Procedural
+ Write / Retrieve / Consolidate / Forget / Promote
```

但这还不完整。

如果要进入团队场景，它还需要：

```text
Scope + Evidence + Permission + Governance
```

这会在第二系列展开。

本篇先解决单 Agent 的基本模型。

---

## 结语：为什么你的 Agent 总像第一次见这个项目？

因为它可能有上下文，但没有经验。

它可能能检索，但不会巩固。

它可能能总结，但不会遗忘。

它可能能记住事实，但不会把流程变成技能。

所以，它每次看起来都像第一次见这个项目。

一个会成长的 Agent，不应该只知道过去发生过什么。

它应该知道：

- 什么值得记；
- 什么仍然有效；
- 什么应该召回；
- 什么应该遗忘；
- 什么应该升级成技能；
- 下一次应该怎么做得更好。

这就是 Agent Memory 和 RAG 的真正差别。

下一篇，我们用这套框架回头看主流 Coding Agent：

> Codex、Claude Code、OpenCode、Hermes、MiMo，谁真正开始长经验？
