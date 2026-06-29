# 第二系列合集：给 Agent 团队装一个共享大脑

## 系列总定位

第一系列解决的是：

> 一个 Coding Agent 如何从经历中成长。

第二系列解决的是：

> 多个 Agent 进入团队协作后，经验如何共享、隔离、验证、路由和治理。

核心判断：

> 团队记忆不是把所有经验塞进一个共享库，而是让正确经验在正确作用域内，被正确 Agent 在正确时机使用。

## 文件导航

| 用途 | 文件 |
|---|---|
| 系列策略 | `00_series_strategy.md` |
| 博客长文 | `01_blog_articles/blog_articles.md` |
| 微信公众号 | `02_wechat_public_account/wechat_article_plan.md` |
| 小红书图文 | `03_xiaohongshu_notes/xiaohongshu_card_scripts.md` |
| 专利交底 | `04_patent/patent_disclosure_team_memory.md` |
| 论文 proposal | `05_paper/paper_proposal.md` |

## 五篇内容主线

### 01 个人记忆不能直接变成团队规则

关键词：Memory Scope、Personal Memory、Project Memory、Team Memory、Organization Memory。

核心句：

> 记忆不是越共享越好，而是要在正确作用域内生效。

### 02 Agent 记住什么不够，还要知道凭什么相信

关键词：Source、Evidence、Confidence、Verified By、Last Verified。

核心句：

> Agent 记住什么不够，它还必须记住自己凭什么相信。

### 03 最高级的记忆，是在未来正确时机自动提醒

关键词：Prospective Memory、Trigger、Workflow Reminder、Event-driven Memory。

核心句：

> 最高级的团队记忆，不是查到过去，而是在未来正确时机自动提醒。

### 04 团队不是共享所有知识，而是知道谁知道什么

关键词：Transactive Memory、Role-based Memory、Multi-Agent Routing。

核心句：

> 团队记忆不是所有人记住所有知识，而是系统知道该找谁、信谁、问谁。

### 05 Agent Memory 也需要免疫系统

关键词：Governance、Quarantine、Approval、Rollback、Stale Detection、Audit Log。

核心句：

> 记忆一旦可以共享，就必须被治理。

## 工程架构总图

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

## 和第一系列的关系

第一系列：给 Coding Agent 装一个外置大脑。

```text
单 Agent
记忆生命周期
Hook Runtime
Checkpoint
Skill Promotion
Agent Style
```

第二系列：给 Agent 团队装一个共享大脑。

```text
多 Agent
记忆作用域
证据可信
前瞻触发
角色路由
团队治理
```

两者组合起来，形成完整路线：

```text
个人外置大脑 → 项目经验系统 → 团队共享大脑 → 组织级 Agent Memory Infrastructure
```

## 后续可扩展文档

建议后续继续补：

1. Team Memory Schema 详细字段设计。
2. Memory Routing 算法设计。
3. Prospective Trigger DSL。
4. Team Memory Dashboard 产品原型。
5. Agent Memory Runtime + Team Memory 的合并架构文档。
