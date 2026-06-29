# 给 Codex 装一个海马体

> 用 Hook、Checkpoint 和 Verifier 做 Agent Memory Runtime。

前三篇讲完，我们已经有了一个比较清晰的判断：

```text
Agent Memory 不是 RAG。
Agent Memory 是经历到技能的生命周期系统。
主流 Coding Agent 大多还停在记录和整理阶段。
```

那问题来了：

如果我们现在就想让 Codex、Claude Code、OpenCode 这类工具更会记忆，应该怎么做？

最现实的答案不是重新训练模型。

也不是把上下文窗口继续拉长。

而是：

> 用 Hook 给 Coding Agent 外挂一套 Memory Runtime。

这就像给 Codex 装一个“海马体”。

主 Agent 负责当前任务。

Memory Runtime 负责记录、召回、验证、巩固和技能化。

---

## 一、为什么 Hook 是最佳切入口？

因为记忆不是一个文件。

记忆发生在 Agent 工作过程里。

如果只在任务结束后让 Agent 总结一句：

```text
本次任务修复了 payment webhook 测试。
```

这远远不够。

你会丢掉很多关键东西：

- 它跑过哪些命令；
- 哪些测试失败；
- 哪些方案试过但失败；
- 哪个错误栈是关键；
- 哪个文件改动导致结果变化；
- 用户中途补了什么约束；
- 它为什么认为任务完成；
- 是否真的验证通过。

这些信息都发生在生命周期中。

所以，Hook 的价值在于：

> 它能在 Agent 行为发生的现场记录证据。

这比事后写总结可靠得多。

事后总结靠模型回忆。

Hook 记录靠真实事件。

对 Coding Agent 来说，真实事件包括命令、测试、diff、错误栈、文件路径、工具结果。

这些才是记忆系统最可靠的原材料。

---

## 二、把 Agent 生命周期拆成记忆事件

一个 Coding Agent 的任务过程，大致可以拆成：

```text
SessionStart
  ↓
UserPrompt
  ↓
ToolUse
  ↓
PostToolUse
  ↓
Compact
  ↓
Stop
  ↓
Review / Consolidate
```

对应记忆系统，就是：

```text
SessionStart：加载相关记忆
UserPrompt：召回历史经验
PostToolUse：记录工具结果
PreCompact：写 checkpoint
Stop：验证是否完成
AfterStop：复盘和巩固
PeriodicJob：Dream / Distill / Curator
```

这就是最小 Memory Runtime。

它不要求模型本身有长期参数更新。

它只要求 Agent 的执行过程能被观察、记录和干预。

也就是说：

```text
模型负责推理。
Hook 负责记忆事件。
Runtime 负责治理记忆。
```

---

## 三、SessionStart：启动时加载最小必要记忆

Agent 每次开始任务时，不应该把所有记忆都塞进去。

它应该加载最小必要上下文。

比如：

```text
当前项目基本信息
当前分支状态
近期 checkpoint
已验证项目规则
高风险模块坑点
用户长期偏好
```

可以注入类似内容：

```text
Relevant Memory:
- This repo uses pnpm, not npm.
- Generated API files should not be edited directly.
- Backend API changes require OpenAPI schema updates.
- Payment integration tests require Redis unless mocked.
```

注意，这不是全量 memory。

这是经过筛选后的启动上下文。

实现上可以读取：

```text
memory/semantic/project_memory.md
memory/semantic/known_pitfalls.md
memory/working/checkpoint.md
memory/user/preferences.md
```

这里最重要的不是“加载多少”。

而是“加载什么”。

加载太少，Agent 像新人。

加载太多，Agent 被旧信息淹没。

所以 SessionStart 的目标是：

> 给 Agent 一个最小但足够的开工状态。

---

## 四、UserPrompt：根据任务召回相关经验

用户输入是最重要的检索线索。

比如用户说：

> 帮我修一下 payment webhook 测试。

Memory Runtime 应该召回：

- payment 模块历史坑点；
- webhook signature 相关经验；
- integration test 依赖服务；
- 相关测试命令；
- 过去失败过的方案；
- 最近相关文件变更。

而不是召回整个项目历史。

检索可以按这些线索做：

```text
用户 prompt
当前目录
git branch
最近修改文件
错误关键词
测试名称
模块名称
```

输出应该很短：

```text
Relevant Past Experience:
1. payment webhook signature requires raw body before JSON parsing.
2. payment integration tests previously failed when Redis was not running.
3. Test command: pnpm test payment-webhook.test.ts.
```

这一步像人类记忆里的“线索召回”。

你看到某个错误栈，会想起之前踩过类似坑。

你进入某个模块，会想起这里有特殊约束。

你准备改数据库，会想起之前 migration 出过事故。

Agent 也应该这样。

> 好的召回不是找最相似，而是找当前最有用。

---

## 五、PostToolUse：把工具结果写成情景记忆

工具调用是 Coding Agent 最重要的事实来源。

因为它不像模型推理那样主观。

它是真实发生过的。

比如：

- shell 命令是否成功；
- 测试是否通过；
- 错误栈是什么；
- apply_patch 改了哪些文件；
- git diff 发生了什么；
- lint 是否失败；
- typecheck 是否通过。

PostToolUse 应该记录这些内容。

示例事件：

```json
{
  "event": "PostToolUse",
  "tool": "Bash",
  "command": "pnpm test payment-webhook.test.ts",
  "exit_code": 1,
  "summary": "Stripe signature verification failed.",
  "related_files": [
    "src/payments/webhook.ts",
    "tests/payment-webhook.test.ts"
  ],
  "memory_candidate": true
}
```

这类事件先进入 episodic memory。

不要立刻写入 project memory。

因为一次失败可能只是偶然现象。

它需要后续验证、重复出现、巩固之后，才能变成稳定知识。

这也是为什么我们要区分：

```text
事件记录 ≠ 长期记忆
```

PostToolUse 只负责把经历记录下来。

Memory Gate 再判断它是否值得晋升。

> Tool result 是 Agent Memory 最可靠的证据来源。

---

## 六、PreCompact：长任务压缩前必须写 checkpoint

长任务最怕上下文压缩。

因为压缩时最容易丢掉：

- 用户真实目标；
- 当前进展；
- 已经失败过的方案；
- 不能改的文件；
- 测试命令；
- 当前 bug 假设；
- 下一步计划。

所以在 compaction 前，必须写 checkpoint。

checkpoint 不要写成流水账。

建议固定格式：

```markdown
# Checkpoint

## User Goal
用户真正要完成什么。

## Current Status
已经做了什么，还差什么。

## Active Constraints
不能违反哪些约束。

## Files Changed
改了哪些文件，为什么。

## Tests Run
跑过哪些测试，结果如何。

## Failed Attempts
哪些方案试过但失败。

## Verified Facts
哪些事实已被验证。

## Next Action
下一步应该做什么。
```

这就是 Working Memory 的持久化版本。

它的作用不是写一份漂亮总结。

而是让下一轮继续不漂。

如果 Agent 压缩后还能清楚知道：

```text
目标是什么
改了哪里
什么不能改
试过什么失败
下一步应该做什么
```

那它就不会像中途失忆。

> Checkpoint 的作用不是总结过去，而是让下一轮继续不漂。

---

## 七、Stop：防止 Agent 假完成

Coding Agent 经常有一个问题：

> 还没真的完成，就开始总结。

比如：

- 改了代码但没跑测试；
- 跑了测试但失败；
- 修了后端但没更新文档；
- 改了 API 但没更新类型；
- 留下 TODO 和临时代码；
- 没检查 git diff。

Stop hook 可以做完成定义检查。

比如：

```text
如果改了代码但没有跑相关测试 → 不允许停止
如果测试失败 → 不允许停止
如果修改 API 但没有更新 schema → 不允许停止
如果出现 TODO / console.log → 要求继续清理
```

Stop verifier 可以返回：

```text
你修改了 backend 文件，但没有运行 API tests。
请先运行相关测试，修复失败后再总结。
```

这不是简单的安全检查。

它是记忆系统的一部分。

因为验证结果会决定这次经历能不能被写入长期记忆。

如果没有验证，Agent 生成的“经验”可能只是幻觉。

如果验证失败，它就应该记录为 failed attempt。

如果验证成功，它才有资格进入 verified facts。

> 没有验证的记忆，只是模型的自我感觉。

---

## 八、AfterStop：任务结束后做复盘

任务完成后，不应该只输出总结给用户。

还应该生成 memory candidates。

也就是：

```text
本次哪些内容值得长期保存？
哪些只是临时信息？
哪些是已验证事实？
哪些是失败方案？
哪些是可复用流程？
哪些可能升级成 skill？
```

示例：

```markdown
# Memory Candidates

## Verified Facts
- payment webhook signature must be verified before JSON parsing.

## Known Pitfalls
- integration tests may fail if Redis is not running.

## Failed Attempts
- parsing JSON body before signature verification breaks Stripe webhook tests.

## Skill Candidate
- payment-webhook-debug workflow.
```

注意，这一步只是候选。

真正写入长期 memory 前，还要经过筛选和巩固。

这一步对应人类的“复盘”。

复盘不是写日报。

复盘是为了让下一次不再从零开始。

> 任务总结给用户看，复盘结果给未来的 Agent 用。

---

## 九、Dream：定期整理记忆

如果 memory 只增不减，很快会乱。

所以需要定期 Dream。

Dream 做的不是继续写代码，而是整理记忆：

- 合并重复事件；
- 删除临时猜测；
- 把已验证事实写入 semantic memory；
- 标记过期记忆；
- 更新 known pitfalls；
- 发现重复流程；
- 生成 skill candidates。

可以每天、每周或按事件触发。

触发条件可以是：

```text
events.jsonl 增长超过一定数量
同类失败出现 2 次以上
某条 memory 频繁被召回
测试规则发生变化
用户明确纠正旧规则
```

Dream 的目标不是让 memory 变多。

而是让 memory 变干净。

它会把一堆散乱事件整理成结构化知识。

> Dream 是让 Agent 从日志堆里长出知识的过程。

---

## 十、Distill：把重复流程提炼成 Skill

如果某类流程反复出现，就不应该永远保存在 notes 里。

它应该变成 skill。

例如，Agent 多次修 CI，流程总是类似：

```text
读取失败日志
定位失败 job
复现本地命令
找最近变更
修复最小代码
重新跑测试
总结原因
```

那就可以生成：

```text
ci-debug.skill
```

如果 API 变更总是需要：

```text
改 schema
改 backend
更新生成类型
更新前端调用
跑 contract test
```

那就可以生成：

```text
api-change.workflow
```

Skill 的核心不是多一段 prompt。

而是把高频流程固化成可复用操作。

这就是程序记忆。

> 经验只有升级成 Skill，Agent 才真的不再从零开始。

---

## 十一、Curator：清理过期记忆和坏 Skill

Skill 也会过期。

记忆也会出错。

所以必须有 Curator。

它负责：

- 找出长期不用的 memory；
- 标记 stale；
- 归档旧 skill；
- 发现冲突规则；
- 降权未验证经验；
- 删除错误泛化；
- 提醒用户审查高风险 memory。

比如：

```text
旧记忆：测试必须 Redis
新事实：测试已 mock Redis
处理：旧记忆 superseded，新规则生效
```

或者：

```text
ci-debug.skill 90 天未使用
处理：标记 stale，不再默认召回
```

Curator 是 memory 的垃圾回收器。

没有它，记忆系统会越来越脏。

> 不会清理的记忆系统，最后都会变成污染系统。

---

## 十二、Trellis 和 Hook 的区别：一个管流程，一个管记忆

这里容易和 Trellis 混淆。

Trellis 更像工程流程脚手架。

它负责告诉 Agent：

```text
任务怎么拆
PRD 放哪里
验收标准是什么
计划怎么写
实现怎么推进
检查怎么做
```

Hook Memory Runtime 解决的是另一个问题：

```text
Agent 执行过程中实际发生了什么？
它有没有跑测试？
有没有失败？
有没有重复踩坑？
有没有遗漏验收标准？
这次经验是否应该写入长期记忆？
是否应该升级成 skill？
```

所以两者不是替代关系。

```text
Trellis 规定 Agent 应该怎么走。
Hook 记录 Agent 实际怎么走，并在关键节点纠偏。
```

最好的做法是：

```text
用 Trellis 约束任务结构。
用 Hook 捕获执行事实。
用 Dream / Distill 把执行经验沉淀回 memory、skills 和 workflow。
```

这样才形成闭环。

---

## 十三、最小可行目录设计

不需要一开始做很复杂。

一个 MVP 可以这样：

```text
memory/
  working/
    checkpoint.md
    current_goal.md

  episodic/
    events.jsonl
    tool_calls.jsonl
    checkpoints/

  semantic/
    project_memory.md
    verified_facts.jsonl
    known_pitfalls.jsonl

  procedural/
    skills/
    workflows/

  user/
    preferences.md

  indexes/
    fts.sqlite
    file_memory_map.json
```

先实现 5 个能力：

```text
1. SessionStart 加载 project memory
2. UserPrompt 检索相关经验
3. PostToolUse 记录工具结果
4. PreCompact 写 checkpoint
5. Stop verifier 防止假完成
```

后续再加：

```text
Dream
Distill
Curator
Team Memory
Dashboard
```

> 先让 Agent 记对，再让 Agent 记多。

---

## 结语：不是让 Agent 自己记，而是给它一套记忆 Runtime

这一篇真正想说的不是：

> Codex 有没有 memory？

而是：

> 我们能不能用 Codex / Claude Code / OpenCode 的生命周期接口，自己构建一套 Memory Runtime？

答案是可以。

因为记忆不是一个单点功能。

它是一个过程系统：

```text
写入
召回
验证
压缩
巩固
遗忘
技能化
```

Hook 的价值就在于，它能把这些过程接进 Agent 的工作流。

下一篇，我们继续往终局走：

> 当 Agent 已经能记录经历、写 checkpoint、做 verifier 之后，如何进一步把经验升级成 skill，并塑造它的工作风格？
