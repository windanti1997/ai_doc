# 博客文章合集：给 Agent 团队装一个共享大脑

## 系列定位

第一系列讲的是：单个 Coding Agent 如何从经历中成长。

第二系列要回答的是：当 Agent 进入团队和组织，记忆如何共享、隔离、验证、路由和治理。

核心判断：

> 团队记忆不是把所有人的经验塞进一个共享知识库，而是让正确的经验在正确的作用域内，被正确的 Agent 在正确的时机使用。

---

## 01 个人记忆不能直接变成团队规则

### 副标题

Personal、Project、Team、Organization，不同记忆不能混用。

### 核心观点

一条记忆必须有作用域。个人偏好、项目事实、团队规则、组织规范、通用技能，不能混在一个全局 memory 里。

### 正文摘要

很多 Agent Memory 系统默认把“记住”当成好事。但在团队场景里，记忆越共享，风险越大。

一个人的偏好，可能只是个人协作方式；一个项目里的经验，可能只适用于当前 repo；一个团队规则，也未必适合整个组织。如果没有作用域，个人经验会被误放大成团队规则，旧项目经验会被误召回到新项目，团队规范也会覆盖掉个人工作方式。

因此，团队级 Agent Memory 的第一原则不是共享，而是分域。

建议分成五层：

```text
Personal Memory：个人偏好和个人工作习惯
Project Memory：单个 repo 的事实和坑点
Team Memory：团队协作规则和 review 习惯
Organization Memory：组织级安全、合规和发布规范
Reusable Skill：跨项目可复用的 workflow / skill / command
```

每条长期记忆都应该回答：

```text
它属于谁？
适用于哪里？
谁能看到？
谁能修改？
能不能升级？
能不能撤回？
```

个人记忆可以成为团队记忆的候选，但不能自动升级。升级必须经过去个人化、证据验证、作用域收敛和团队审查。

钉子句：

> 记忆不是越共享越好，而是要在正确作用域内生效。

---

## 02 Agent 记住什么不够，还要知道凭什么相信

### 副标题

Source、Evidence、Confidence，是团队记忆的可信底座。

### 核心观点

团队记忆必须记录来源和证据。模型猜测、用户口头说明、测试验证、CI 日志、PR review、团队审查，可信度完全不同。

### 正文摘要

单个 Agent 记错一条经验，最多影响一次任务；团队 memory 记错一条规则，可能污染所有 Agent 的行为。

比如一条记忆：

```text
payment integration tests require Redis
```

它到底来自哪里？

是模型猜的？用户说的？本地测试验证的？CI 日志证明的？还是团队 review 确认的？

这些来源不能等价。

团队级记忆必须有 source model：

```text
source_type：来源类型
evidence：证据引用
confidence：置信度
verified_by：验证者
last_verified：最后验证时间
invalidated_by：被什么推翻
owner：责任人或团队
```

记忆也要区分状态：

```text
observed：实际观察到
inferred：推理得到
user_stated：用户说明
tool_verified：工具验证
team_approved：团队确认
stale：可能过期
superseded：被新规则替代
```

没有来源的记忆，在个人场景只是“不可靠”；在团队场景就是“风险源”。

钉子句：

> Agent 记住什么不够，它还必须记住自己凭什么相信。

---

## 03 最高级的记忆，是在未来正确时机自动提醒

### 副标题

Prospective Memory：让 Agent 记住“将来遇到 X 时要做 Y”。

### 核心观点

团队记忆不应该只是被动检索历史，还应该能在关键事件发生时主动触发。

### 正文摘要

人类有一种很重要的记忆能力：记住未来要做的事。

比如：路过超市买牛奶；开会前发材料；上线前检查回滚方案。

这类记忆不是回忆过去，而是在未来条件满足时触发行动。

Coding Agent 也需要这种前瞻记忆。

例如：

```text
当修改 backend API → 提醒更新 OpenAPI schema 和 generated client
当修改 payment 模块 → 触发支付安全 checklist
当修改 migration → 要求检查 rollback path
当修改 auth / permission → 触发 security reviewer agent
当创建 release branch → 自动召回发布 checklist
```

这类 memory 不只是 knowledge，而是 trigger。

可以建模为：

```json
{
  "type": "prospective_memory",
  "trigger": "file_changed:backend/api/**",
  "intention": "update schema and generated client",
  "evidence": ["past_ci_failure:contract-mismatch"],
  "scope": "team:platform",
  "priority": "high"
}
```

团队场景尤其需要前瞻记忆，因为很多错误不是不知道规则，而是关键时刻没人想起来。

钉子句：

> 最高级的团队记忆，不是查到过去，而是在未来正确时机自动提醒。

---

## 04 团队不是共享所有知识，而是知道谁知道什么

### 副标题

从团队分工到 Multi-Agent Memory Routing。

### 核心观点

团队记忆不是让所有 Agent 共享所有内容，而是建立“谁知道什么、谁负责什么、谁验证过什么”的路由机制。

### 正文摘要

现实团队不会让每个人记住所有知识。

一个高效团队更像这样：

```text
谁懂支付？
谁懂发布？
谁知道历史架构决策？
谁可以 review 安全？
谁维护某个模块？
```

团队的记忆不是全量共享，而是知道该找谁。

Agent Team 也一样。

不同 Agent 需要不同 memory：

```text
Planner Agent：需求、路线图、优先级、约束
Coder Agent：代码结构、测试命令、模块坑点
Reviewer Agent：review 标准、历史拒绝点
Security Agent：安全红线、权限边界、敏感模块
Test Agent：测试策略、fixture、环境依赖
Memory Writer Agent：写入准入、证据规则、去重策略
```

如果所有 Agent 都拿同一份 memory，会带来两个问题：

1. 噪声太大，召回质量下降。
2. 角色边界被污染，Reviewer 变得像 Coder，Security Agent 忘了安全优先。

所以需要 Memory Routing：

```text
根据任务类型、文件路径、Agent 角色、风险级别、记忆作用域，选择性召回。
```

钉子句：

> 团队记忆不是所有人记住所有知识，而是系统知道该找谁、信谁、问谁。

---

## 05 Agent Memory 也需要免疫系统

### 副标题

如何防止错误经验、过期规则和坏 Skill 进入团队记忆。

### 核心观点

记忆一旦可以共享，就必须被治理。团队 memory 需要隔离、审查、回滚、降权、失效和审计。

### 正文摘要

团队记忆的价值很高，风险也很高。

如果一条错误经验被写入个人 memory，影响的是一个 Agent；如果它被升级为团队规则，所有 Agent 都可能照做；如果它再被升级成 skill 或 hook，错误会被自动化放大。

所以团队级 memory 必须有治理机制。

至少包括：

```text
quarantine：候选记忆先隔离
approval：高影响记忆需要审查
redaction：敏感信息脱敏
permission：控制谁能看、谁能改
rollback：错误记忆可以回滚
stale detection：发现过期规则
conflict detection：发现互相冲突的记忆
audit log：记录记忆变化历史
```

例如，Agent 生成一条候选记忆：

```text
为了加快开发，可以跳过部分测试。
```

团队 memory 系统不应该直接写入，而应该判断：

```text
是否违反团队质量规范？
是否缺少证据？
是否可能影响发布安全？
是否需要人工审批？
```

团队 memory 的核心不是“让 Agent 记得更多”，而是“让 Agent 只共享可靠、合规、可追溯的经验”。

钉子句：

> Agent Memory 不只需要大脑，也需要免疫系统。
