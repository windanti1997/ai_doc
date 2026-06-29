# 小红书 04：给 Codex 装一个海马体

## 标题

给 Codex 装一个海马体

## 封面文案

给 Codex 装一个海马体

副标题：Hook 是 Agent Memory 的生命周期触发器

## 9 张卡片文案

### 01 封面

给 Codex 装一个海马体

用 Hook 做 Agent Memory Runtime。

### 02 核心观点

不训练新模型，也能让 Agent 更会记忆。

方法是：把 Agent 的关键生命周期事件接出来。

### 03 为什么是 Hook

记忆不是任务结束后写一句 summary。

真正的证据发生在过程里：命令、测试、错误栈、diff、用户纠正。

### 04 SessionStart

启动时加载最小必要记忆。

项目规则、近期 checkpoint、高风险模块、用户偏好。

不要一上来塞满上下文。

### 05 UserPrompt

用户输入时，按任务召回相关经验。

改 API 召回 schema checklist。

改 payment 召回安全和测试坑点。

### 06 PostToolUse

工具调用后记录真实证据。

命令是否成功、测试是否通过、错误栈是什么、哪些文件被改了。

这是最可靠的 memory 来源。

### 07 PreCompact

长任务压缩前必须写 checkpoint。

保存目标、状态、约束、改动文件、失败尝试、下一步。

防止任务断线。

### 08 Stop

Stop hook 用来防止假完成。

改了代码但没跑测试，测试失败却准备总结，都应该被 verifier 拦住。

### 09 总结

Hook 让 memory 不只是文件。

它变成 Agent 工作过程中的运行时系统。

## 正文 Caption

如果要给 Codex 做长期记忆，我不会从“存聊天记录”开始。

我会先从 Hook 开始。

因为 Coding Agent 最可靠的记忆来源，不是事后总结，而是工作过程中的真实证据：

命令输出、测试结果、错误栈、diff、用户纠正、验证状态。

一个最小 Memory Runtime 可以有这些节点：

SessionStart：加载最小必要记忆。

UserPrompt：按任务召回相关经验。

PostToolUse：记录工具结果。

PreCompact：压缩前写 checkpoint。

Stop：用 verifier 防止假完成。

AfterStop：生成候选记忆，后续再 Dream / Distill / Curator。

一句话：Hook 是 Agent Memory 的生命周期触发器。

## 话题标签

#Codex #CodingAgent #AgentMemory #AI工程化 #AI编程 #程序员 #ClaudeCode #OpenCode

## 配图提示

建议生成 9 张 3:4 小红书卡片，视觉关键词：海马体隐喻、Hook 事件流、Checkpoint、Verifier、Memory Runtime 架构。

