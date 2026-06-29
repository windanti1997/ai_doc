# 系列一：给 Coding Agent 装一个外置大脑

## 系列定位

**给 Coding Agent 装一个外置大脑：从人类记忆到 Agent Memory Runtime**

这个系列面向单个 Coding Agent，讨论如何让 Codex、Claude Code、OpenCode、Qwen Code 等工具从“会执行任务”进化为“会积累经验”。

核心观点：

> Agent Memory 不是 RAG，也不是更长上下文，而是一套把经历转化为知识、技能和工作风格的经验系统。

## 目标读者

- 正在使用 Codex / Claude Code / OpenCode / Cursor / Cline 的开发者
- 想给 Coding Agent 加长期记忆、Hook、Checkpoint、Skill 的工程师
- 想做 Agent Memory Runtime 开源项目的人
- 想把 Agent Memory 做成论文、专利或产品方向的人

## 五篇主线

| 篇数 | 标题 | 核心问题 |
|---|---|---|
| 01 | 别再把 Agent 记忆做成向量库了 | 人类记忆为什么不是存储？ |
| 02 | 为什么你的 Agent 总像第一次见这个项目？ | Coding Agent Memory 应该怎么设计？ |
| 03 | 我对比了十几个 Coding Agent，发现大多数还不会长经验 | 主流工具做到了哪一步？ |
| 04 | 给 Codex 装一个海马体 | Hook Memory Runtime 怎么落地？ |
| 05 | 从踩坑到技能：让 Agent 不再重复犯错 | 经验如何升级成 Skill 和工作风格？ |

## 统一叙事链路

```text
人类记忆不是硬盘
→ Agent Memory 不是 RAG
→ Coding Agent 需要四层记忆
→ Hook 是运行时切入口
→ 经验最终要升级成 Skill 和 Style
```

## 系列钉子句

- 记忆的价值不是保存过去，而是改变下一次行动。
- RAG 是查资料，Memory 是长经验。
- 没有证据的记忆，只是模型的自我感觉。
- Hook 是给 Coding Agent 外挂记忆系统的最佳切入口。
- Agent Memory 的终点不是长期笔记，而是可复用技能。

## 内容边界

本系列重点讲**单个 Coding Agent 如何长经验**。

不展开团队级权限、组织知识治理、多 Agent 记忆路由等内容；这些放到第二系列「给 Agent 团队装一个共享大脑」。

## 推荐发布顺序

1. 博客长文：先发 01-03 建立认知，再发 04-05 承接开源项目。
2. 微信公众号：每篇压缩成观点更强的技术评论。
3. 小红书：每篇拆成 6-8 张卡片，主打反常识观点。
4. 专利：从 Hook Memory Runtime、Evidence-grounded Memory、Skill Promotion 中抽技术方案。
5. 论文：以“面向长周期 Coding Agent 的证据约束记忆运行时”为主线。
