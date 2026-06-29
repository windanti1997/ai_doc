# 从踩坑到技能：让 Agent 不再重复犯错

> 经验如何升级成 Skill，并塑造 Agent 的工作风格。

上一篇我们讲了如何用 Hook 给 Coding Agent 外挂一套 memory runtime。

到这一步，Agent 已经能做到：

- 启动时加载项目记忆；
- 用户输入时召回相关经验；
- 工具调用后记录事实；
- 上下文压缩前写 checkpoint；
- 停止前做 verifier；
- 任务结束后生成 memory candidates。

但这还不够。

因为记录经历只是第一步。

真正的成长，是把经历变成技能。

更进一步，是让这些技能和记忆塑造 Agent 的工作风格。

也就是说：

```text
Episode → Fact → Rule → Skill → Style
```

这篇是第一系列的收束。

我们讨论一个更重要的问题：

> Agent Memory 的终点到底是什么？

答案不是长期笔记。

也不是 project_memory.md。

而是可复用技能，以及稳定的工程行为风格。

## 一、如果只会记笔记，Agent 仍然像新人

很多 memory 系统到 project_memory.md 就停了。

它会记录：

```text
本项目使用 pnpm。
payment integration tests require Redis。
generated API files 不要直接手改。
```

这些都很有用。

但它们只是事实。

一个新人也可以看项目文档。

真正有经验的工程师，不只是知道事实，而是形成了做事方法。

比如他遇到 bug，会自然按这个路径走：

```text
复现问题
定位边界
找最近变更
写失败测试
最小修改
跑验证
检查 diff
提交 PR
```

他不需要每次重新思考。

这就是程序记忆。

对应到 Agent，就是 skill、workflow、hook、command、subagent playbook。

> 只会记笔记的 Agent，是知识库。会把经验变成流程的 Agent，才是工程搭档。

## 二、经验升级链：Episode → Fact → Rule → Skill → Automation

一个成熟的 Agent Memory 系统，应该有明确的晋升链。

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

### 1. Episode：一次经历

例如：

```text
某次 payment webhook test 失败，原因是 Redis 未启动。
```

这是情景记忆。

它记录发生过什么。

但一次经历不能直接变成规则。

因为它可能只是偶然现象。

### 2. Fact：被验证事实

如果同类测试多次因为 Redis 失败，并且启动 Redis 后通过，就可以形成事实：

```text
payment integration tests require Redis。
```

这需要证据。

比如测试结果、命令输出、相关文件。

### 3. Rule：项目规则

如果这个事实长期有效，并且影响未来任务，就可以写入项目规则：

```text
运行 payment integration tests 前，必须检查 Redis 是否启动。
```

这是 semantic memory。

### 4. Skill：可复用流程

如果这个规则对应一套反复执行的流程，就应该升级成 skill：

```text
payment-test-setup.skill
```

内容可能是：

```text
1. 检查 Redis 状态
2. 检查环境变量
3. 启动依赖服务
4. 运行测试
5. 如果失败，收集错误栈
```

### 5. Automation：自动化动作

如果这个流程足够稳定，还可以变成 hook 或 command：

```text
运行 integration test 前自动检查 Redis。
```

这就是从记忆到能力。

> 记忆的最高价值，是把重复推理变成稳定动作。

## 三、什么经验值得升级成 Skill？

不是所有经验都应该升级。

否则 skill library 会变成垃圾堆。

适合升级的经验通常有几个特征：

```text
重复出现
流程稳定
结果可验证
未来复用概率高
错误成本较高
步骤可以明确描述
不依赖太多临时上下文
```

比如适合升级成 skill 的场景：

- TDD 开发流程；
- CI debug；
- API schema 变更；
- 数据库 migration；
- release checklist；
- PR review；
- 安全审查；
- 性能回归排查；
- integration test setup。

不适合升级成 skill 的内容：

- 一次性临时命令；
- 未验证猜测；
- 特定 bug 的偶然修法；
- 已废弃架构下的 workaround；
- 依赖单次上下文的判断；
- 用户临时偏好。

> Skill 不是记忆越多越好，而是流程越稳定越值得固化。

## 四、错误经验也可能被升级成坏技能

技能化有一个风险：

> 错误经验一旦变成 skill，污染会被放大。

比如 Agent 某次误以为：

```text
payment 测试失败通常都是 Redis。
```

如果它把这条经验升级成 skill，下次所有 payment 测试失败都先去处理 Redis，就会误导调试。

真正正确的做法是：

```text
payment integration tests 曾因 Redis 未启动失败。
当错误栈包含 Redis connection refused 时，优先检查 Redis。
当错误栈是 Stripe signature verification failed 时，应检查 raw body parsing。
```

这就是 scope 和 evidence 的重要性。

Skill promotion 前必须检查：

- 证据是否充分？
- 是否重复出现？
- 是否有明确适用范围？
- 是否有反例？
- 是否经过测试验证？
- 是否会过度泛化？
- 用户是否需要审查？

可以设置状态：

```text
candidate
verified
active
stale
archived
superseded
```

> 让错误经验变成技能，比忘记经验更危险。

## 五、Dream / Distill / Curator 的分工

要让经验持续升级，需要三个后台角色。

### Dream：整理记忆

Dream 负责：

- 合并重复事件；
- 去掉临时猜测；
- 压缩日志；
- 找出共性；
- 更新 project memory；
- 标记冲突和过期内容。

它回答：

> 哪些经历可以变成知识？

### Distill：提炼技能

Distill 负责：

- 找出重复成功流程；
- 提炼步骤；
- 生成 skill；
- 生成 workflow；
- 生成 checklist；
- 生成 command 或 hook 候选。

它回答：

> 哪些知识可以变成能力？

### Curator：管理遗忘

Curator 负责：

- 发现长期不用的 skill；
- 归档过期流程；
- 降权未验证记忆；
- 删除错误泛化；
- 管理冲突规则；
- 提醒用户审查高风险 skill。

它回答：

> 哪些记忆和技能应该被弱化或淘汰？

这三个角色合起来，才是完整的成长闭环。

```text
Dream：经历 → 知识
Distill：知识 → 技能
Curator：清理坏记忆和旧技能
```

> 没有 Curator 的 Skill Library，迟早会变成新的垃圾堆。

## 六、Agent 的工作风格，是被长期记忆塑造出来的

记忆不仅影响 Agent 知道什么。

还会影响 Agent 怎么做事。

如果一个 Agent 的 memory 总是强化：

```text
不要冒险
不要改大文件
不确定就先问
测试失败很危险
```

它会逐渐变成保守型 Agent。

如果 memory 总是强化：

```text
先快速实现
失败后再修
小步迭代
优先探索
```

它会变成实验型 Agent。

如果 memory 总是强化：

```text
先读架构
先写测试
先查历史坑点
必须 verifier 通过
必须更新文档
```

它会变成严谨型 Agent。

如果 memory 总是强化：

```text
先看攻击面
先检查权限
先验证输入边界
优先保护数据
```

它会变成安全型 Agent。

所以，Agent 的“性格”不一定来自模型本身。

它也可能来自长期规则、长期记忆、失败复盘、用户纠错、奖励信号、verifier、skill library、团队规范。

> 你给 Agent 什么记忆，它就会变成什么样的工程师。

## 七、四种 Agent 工作风格设计

既然 memory 会塑造行为风格，就可以主动设计。

### 1. 严谨型 Agent

适合金融、医疗、生产系统、基础设施、大型企业项目。

记忆强化：

```text
先读规范
先写测试
必须验证
必须审查 diff
禁止未验证总结
```

默认流程：

```text
Read → Plan → Test → Implement → Verify → Review
```

### 2. 探索型 Agent

适合原型、Hackathon、创业项目、早期产品验证。

记忆强化：

```text
快速尝试
小步迭代
允许失败
优先跑通
失败后复盘
```

默认流程：

```text
Prototype → Run → Learn → Iterate
```

### 3. 安全型 Agent

适合权限系统、支付系统、用户数据、安全敏感业务。

记忆强化：

```text
默认不信任输入
优先检查权限
记录风险点
敏感操作需确认
```

默认流程：

```text
Threat Model → Change → Test → Audit
```

### 4. 架构型 Agent

适合大重构、多模块系统、平台工程、长期项目。

记忆强化：

```text
先看边界
先看依赖
先看历史决策
避免局部最优
更新架构文档
```

默认流程：

```text
Context → Boundary → Design → Implement → Document
```

这说明 Agent Memory 不只是工具层设计。

它也是 Agent 行为设计。

## 八、从个人 Agent 走向团队 Agent

到这里，第一系列的主线已经完成：

```text
人类记忆不是硬盘
→ Agent Memory 不是 RAG
→ Coding Agent 需要记忆生命周期
→ Hook 是工程落地入口
→ 经验最终升级为 Skill 和 Style
```

但真实工程里，Agent 不会只服务一个人。

它会进入团队、项目和组织。

这时问题会升级：

- 个人经验能不能升级成团队规则？
- 团队记忆谁能修改？
- 错误记忆如何回滚？
- 不同 Agent 是否应该共享同一份 memory？
- 一条记忆适用于个人、项目、团队，还是组织？

这就进入第二系列：

> 给 Agent 团队装一个共享大脑。

## 结语

这个系列真正想说的不是：Agent 要不要有 memory。

而是：Agent 需要什么样的 memory。

如果 memory 只是存储，它就是资料库。

如果 memory 只是摘要，它就是笔记本。

如果 memory 只是向量库，它就是搜索系统。

但如果 memory 能筛选、加权、记录、召回、巩固、遗忘、升级、技能化、塑造行为风格，它才是经验系统。

而经验系统，才是 Coding Agent 从“会执行任务”走向“会成长”的关键。

最终一句话：

> 真正强的 Agent，不是会写更多代码，而是会把经验变成稳定的工作方式。
