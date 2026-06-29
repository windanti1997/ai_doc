# 专利交底草案：面向 Coding Agent 的外置记忆运行时系统

> 说明：本文是技术交底草案，不构成法律意见。正式申请前应由专利代理人检索、撰写和定稿。

## 一、拟定名称

一种面向长周期编程智能体的外置记忆运行时系统及方法。

## 二、技术领域

本方案涉及人工智能编程助手、智能体运行时、软件工程自动化、长期记忆管理、工具调用日志分析、任务状态恢复与技能生成等领域。

## 三、背景问题

现有 Coding Agent 通常依赖当前上下文窗口、项目规则文件、会话摘要或向量检索来维持任务连续性。但在长周期软件开发任务中，仍存在以下问题：

1. 上下文压缩后任务目标、用户约束和失败路径丢失。
2. 工具调用结果、测试失败和代码 diff 未被结构化沉淀。
3. 旧经验缺少证据、作用域、置信度和失效机制。
4. Agent 容易在未验证情况下宣布任务完成。
5. 重复成功的流程无法自动晋升为 skill、workflow 或 hook。
6. 现有 memory 多为被动检索，难以在关键事件发生时主动触发。

## 四、核心发明点

### 发明点 1：基于 Agent 生命周期 Hook 的记忆写入与召回机制

在 Coding Agent 的关键生命周期事件中触发记忆操作，包括但不限于：

- SessionStart：加载相关项目记忆；
- UserPrompt：根据用户任务召回历史经验；
- PostToolUse：记录命令、测试、diff、错误栈；
- PreCompact：在上下文压缩前写 checkpoint；
- Stop：验证任务是否完成；
- AfterStop：生成 memory candidates。

### 发明点 2：证据约束的情景记忆记录方法

将工具调用结果、测试输出、代码 diff、用户纠正、PR review、CI 日志等作为记忆证据，并为每条记忆建立：

```text
source_type
evidence
scope
confidence
last_verified
invalidated_by
promotion_state
```

### 发明点 3：上下文压缩前的结构化 Checkpoint 重建方法

在压缩前生成固定格式 checkpoint，包括：

- 用户目标；
- 当前状态；
- 活跃约束；
- 修改文件；
- 已运行测试；
- 失败尝试；
- 已验证事实；
- 下一步动作。

该 checkpoint 用于下一轮上下文重建，减少长任务漂移。

### 发明点 4：Stop Verifier 驱动的任务完成校验机制

当 Agent 准备停止时，由 verifier 检查完成定义。若不满足条件，则阻止停止并生成继续执行提示。检查项包括：

- 是否运行相关测试；
- 是否存在失败测试；
- 是否存在临时代码；
- 是否满足验收标准；
- 是否更新文档或生成类型。

### 发明点 5：经验到技能的自动晋升链

通过离线 Dream / Distill 过程，将 episodic events 晋升为：

```text
Episode → Fact → Rule → Skill → Automation
```

其中 Skill 可表现为 Markdown skill、workflow、command、hook、subagent playbook。

### 发明点 6：过期记忆与错误 Skill 的 Curator 机制

通过使用频率、验证时间、代码变更、冲突记忆、用户纠正等信号，对记忆和 skill 进行降权、归档、失效或回滚。

## 五、系统结构

```text
Coding Agent
  ↓
Hook Event Collector
  ↓
Episodic Recorder
  ↓
Memory Gate / Importance Scorer
  ↓
Working Memory / Semantic Memory / Procedural Memory
  ↓
Retriever / Checkpoint Rebuilder / Stop Verifier
  ↓
Dream Consolidator / Skill Distiller / Curator
```

## 六、实施例

### 实施例：Payment Webhook 测试失败

1. Agent 执行 `pnpm test payment-webhook.test.ts`。
2. PostToolUse 捕获失败日志。
3. 系统记录：错误栈、命令、exit code、相关文件。
4. 启动 Redis 后测试通过。
5. Memory Gate 生成候选记忆：payment integration tests require Redis。
6. Dream 将多次类似事件合并为 verified fact。
7. Distill 生成 `payment-test-setup.skill`。
8. 后续修改 payment 模块时，Retriever 自动召回该 skill。

## 七、可保护的权利要求方向

1. 一种基于 Hook 事件的 Coding Agent 记忆管理方法。
2. 一种在上下文压缩前生成结构化 checkpoint 的方法。
3. 一种基于工具结果证据生成长期记忆的方法。
4. 一种任务停止前基于完成定义的 verifier 机制。
5. 一种将情景记忆自动晋升为程序技能的方法。
6. 一种基于验证时间、冲突和代码变更的记忆失效方法。

## 八、区别于现有方案的重点

本方案不是简单的会话摘要、向量检索或规则文件，而是在 Coding Agent 运行时生命周期中采集证据、维护任务状态、验证完成条件，并将重复经验晋升为可复用技能。
