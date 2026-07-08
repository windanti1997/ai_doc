# 给 AI 装一个外置大脑：公众号系列写作上下文包

这个目录用于让其他 Agent 接着写「给 AI / Coding Agent 装一个外置大脑」系列文章而不跑偏。

目录结构：

```text
wechat-ai-memory-series/
├── context_pack/          # 给其他 Agent 的系列上下文包
├── archive_published/    # 已发布/已定稿文章归档，每篇一个文件
├── skills/               # 写作 skill、workflow skill、封面 skill
└── contracts/            # 每篇 Article Contract，每篇一个文件
```

使用顺序：

1. 先读 `context_pack/01_SERIES_BIBLE.md`，理解系列主线。
2. 再读 `context_pack/02_WRITING_SKILL.md` 与 `skills/`，理解写作风格、评审和工作流。
3. 写某一篇之前，必须读对应的 `contracts/ARTICLE_CONTRACT_XX.md`。
4. 写完后，把最终文章归档到 `archive_published/`，并把新的契约也追加到 `contracts/`。

> 重要：这个系列不能写成泛 AI 科普、提示词教程、产品架构文档或作者独白。每篇只推进一个记忆问题，从普通用户真实痛点进入，再轻轻点出背后的 Agent Memory 机制。
