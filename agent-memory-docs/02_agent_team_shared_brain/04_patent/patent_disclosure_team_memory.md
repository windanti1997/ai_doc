# 专利交底草案：面向多 Agent 协作的团队记忆管理系统

> 说明：本文是技术交底草案，不构成法律意见。正式申请前应由专利代理人检索、撰写和定稿。

## 一、拟定名称

一种面向多智能体协作的软件工程团队记忆管理系统及方法。

## 二、技术领域

本方案涉及人工智能智能体、多智能体协作、软件工程自动化、长期记忆管理、知识权限治理、记忆可信度评估、上下文路由和流程自动触发等领域。

## 三、背景问题

现有 Agent Memory 多面向个人助手或单个 Agent，通常以聊天摘要、向量检索、项目规则文件或本地 memory 文件形式存在。当多个 Agent 进入团队工程协作后，会出现以下问题：

1. 个人偏好、项目事实、团队规则和组织规范混在同一记忆空间。
2. 单个 Agent 生成的经验缺少审查，可能被错误放大为团队规则。
3. 不同角色 Agent 获取相同记忆，导致噪声增加和职责边界混乱。
4. 记忆缺少来源、证据、置信度和责任归属。
5. 团队规则只能被动检索，无法在未来关键事件发生时主动触发。
6. 过期、冲突或不合规的记忆缺少降权、隔离、回滚机制。

## 四、核心发明点

### 发明点 1：多层级记忆作用域模型

将长期记忆划分为多个作用域：

```text
Personal Memory
Project Memory
Team Memory
Organization Memory
Reusable Skill
```

每条记忆包含 scope、visibility、owner、promotion_state 等字段，用于限定其生效范围、可见范围和升级状态。

### 发明点 2：个人记忆到团队记忆的晋升治理方法

个人或项目经验不能直接进入团队记忆，必须经过：

```text
候选生成 → 去个人化 → 证据绑定 → 作用域收敛 → 审查确认 → 发布生效
```

该方法避免个人偏好被误升级成团队规范。

### 发明点 3：基于来源和证据的记忆可信度评估

每条团队记忆记录：

```text
source_type
evidence_ref
confidence
verified_by
last_verified
invalidated_by
owner
audit_log
```

系统根据来源类型、验证记录、历史成功率、冲突情况动态计算可信度。

### 发明点 4：前瞻式团队记忆触发机制

将部分团队记忆建模为条件触发器，当检测到特定事件时主动提醒或启动对应 Agent。

示例：

```text
文件路径匹配 backend/api/** → 提醒更新 schema 和生成类型
修改 payment 模块 → 触发支付检查流程
修改 auth 模块 → 触发权限 review
创建 release 分支 → 召回发布 checklist
```

### 发明点 5：多 Agent 角色化记忆路由方法

根据 Agent 角色、任务类型、文件路径、风险级别、项目作用域选择性召回记忆。

不同 Agent 使用不同记忆视图：

```text
Planner Agent：需求、优先级、路线图
Coder Agent：代码约束、测试命令、模块坑点
Reviewer Agent：review 标准、历史拒绝点
Security Agent：权限边界、敏感模块规则
Memory Agent：写入准入、证据规则、冲突检测
```

### 发明点 6：团队记忆治理与回滚机制

系统对候选记忆进行隔离、审查、脱敏、权限控制、降权、归档、冲突检测和回滚。

包括：

```text
quarantine_state
approval_required
redaction_policy
permission_policy
stale_detection
conflict_detection
rollback_version
```

## 五、系统结构

```text
Memory Candidate Collector
  ↓
Scope Classifier
  ↓
Evidence Binder
  ↓
Trust Scorer
  ↓
Approval Workflow
  ↓
Team Memory Store
  ↓
Role-based Memory Router
  ↓
Prospective Trigger Engine
  ↓
Governance / Rollback Layer
```

## 六、实施例

### 实施例：API 变更规则升级为团队记忆

1. 多次 CI 失败显示后端 API 变更后未同步前端 generated client。
2. 系统从多个项目事件中提取候选记忆。
3. Evidence Binder 绑定 CI 日志、PR review、失败测试。
4. Scope Classifier 判断该经验适用于 platform 团队。
5. 审查通过后，写入 Team Memory。
6. 生成前瞻触发器：当 backend/api/** 文件变更时，提醒更新 schema 和 generated client。
7. 未来 Coder Agent 修改 API 文件时，自动召回该规则。

## 七、可保护的权利要求方向

1. 一种多作用域 Agent 记忆管理方法。
2. 一种个人记忆晋升为团队记忆的审查方法。
3. 一种基于来源和证据计算记忆可信度的方法。
4. 一种前瞻式团队记忆触发方法。
5. 一种面向多 Agent 角色的记忆路由方法。
6. 一种团队记忆的隔离、失效和回滚机制。

## 八、区别于现有方案的重点

本方案不是单 Agent 的本地长期记忆，也不是简单共享知识库，而是将记忆作用域、证据可信、角色化路由、前瞻触发和治理回滚组合为团队级 Agent Memory Infrastructure。
