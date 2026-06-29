# 小红书图文脚本：给 Coding Agent 装一个外置大脑

## 账号定位建议

适合定位为：AI 工程化 / Coding Agent 实战 / Agent Memory 系统设计。

小红书不适合直接发长文，建议每篇拆成 7-9 张卡片。

统一视觉关键词：

```text
外置大脑 / 记忆外挂 / 长经验 / 不再重复踩坑 / Hook / Skill
```

---

## 选题 01：别再把 Agent 记忆做成向量库了

### 封面标题

别再把 Agent 记忆做成向量库了

### 卡片结构

1. 大多数 Agent 不是缺上下文，而是缺经验系统。
2. RAG 是查资料，Memory 是长经验。
3. 人类记忆不是硬盘，而是行为塑造系统。
4. 记忆会筛选：不是所有日志都值得存。
5. 记忆会加权：失败、风险、用户纠正更重要。
6. 记忆会巩固：睡眠式整理，把经历变知识。
7. 记忆会遗忘：旧经验不清理会污染未来。
8. 记忆会技能化：经验最终应该变成 workflow。
9. 总结：Agent Memory 的目标不是记住更多，而是下一次做得更好。

### 文案钩子

如果你的 Agent 每次都像第一次见这个项目，问题不是模型不够强，而是它没有真正的记忆系统。

---

## 选题 02：为什么你的 Agent 总像第一次见这个项目？

### 封面标题

为什么你的 Agent 总像第一次见这个项目？

### 卡片结构

1. 你告诉过它用 pnpm，它下次还是 npm。
2. 你纠正过它不要改 generated 文件，它下次又改。
3. 长上下文不等于长期记忆。
4. Coding Agent 需要四层记忆：Working / Episodic / Semantic / Procedural。
5. Working：当前任务别断线。
6. Episodic：记录真实执行轨迹。
7. Semantic：沉淀项目知识。
8. Procedural：把流程变成 skill。
9. 总结：Agent Memory = 四层记忆 + 五个治理机制。

---

## 选题 03：我对比了十几个 Coding Agent

### 封面标题

我对比了十几个 Coding Agent，发现大多数还不会长经验

### 卡片结构

1. 现在很多工具都说自己有 memory，但不是一回事。
2. 规则型：会读手册，不会长经验。
3. 上下文型：当前任务很强，session 后容易断。
4. 自动记忆型：会总结，但不一定可治理。
5. 生命周期型：有 Hook，才开始像 Runtime。
6. 进化型：能 Dream / Distill / Skill。
7. Codex / Claude Code / OpenCode 的价值是可改造。
8. Hermes / MiMo 的价值是开始技能化。
9. 总结：分水岭不是有没有 memory，而是有没有 Skill Promotion。

---

## 选题 04：给 Codex 装一个海马体

### 封面标题

给 Codex 装一个海马体

### 卡片结构

1. 不训练模型，也能让 Agent 更会记忆。
2. 关键是 Hook。
3. SessionStart：启动时加载项目记忆。
4. UserPrompt：按任务召回历史坑点。
5. PostToolUse：记录命令、测试、diff、错误栈。
6. PreCompact：压缩前写 checkpoint。
7. Stop：防止 Agent 假完成。
8. Dream / Distill：把日志变知识，把知识变 skill。
9. 总结：Hook 是 Agent Memory 的生命周期触发器。

---

## 选题 05：从踩坑到技能

### 封面标题

让 Agent 不再重复犯错：从踩坑到技能

### 卡片结构

1. 一次失败只是经历。
2. 多次重复才是规律。
3. 被验证才是项目知识。
4. 反复执行才值得变成 skill。
5. Episode → Fact → Rule → Skill → Automation。
6. 错误经验变成 skill 会放大污染。
7. 需要 Dream、Distill、Curator。
8. 长期记忆会塑造 Agent 工作风格。
9. 总结：你给 Agent 什么记忆，它就会变成什么样的工程师。

---

## 通用结尾模板

如果你也在用 Codex / Claude Code / OpenCode 做真实项目，建议不要只调 prompt。

真正值得做的是：给 Coding Agent 加一套外置大脑。

它不只是记住过去，而是让 Agent 从经历中长经验。
