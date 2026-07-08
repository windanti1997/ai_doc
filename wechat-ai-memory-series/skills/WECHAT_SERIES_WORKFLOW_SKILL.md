# Skill：公众号系列文章 Workflow

## 目标

保证任何 Agent 接手本系列时，不仅能写出像风逆风格的文章，也能保证文章不偏离系列主线。

## 必读上下文顺序

1. `context_pack/01_SERIES_BIBLE.md`：理解系列主线；
2. `context_pack/04_PREVIOUS_ARTICLES_SUMMARY.md`：理解前文边界；
3. `context_pack/05_FAILURE_EXAMPLES.md`：理解已踩坑；
4. `contracts/ARTICLE_CONTRACT_XX.md`：理解当前这篇只写什么；
5. `skills/WECHAT_SERIES_WRITING_SKILL.md`：执行写作；
6. `skills/WECHAT_COVER_IMAGE_SKILL.md`：生成封面提示词。

## 执行流程

### Step 1：Series Alignment Card

写正文前必须先输出 Series Alignment Card。它负责防止跑偏。

必须明确：本篇编号、前一篇讲什么、本篇只讲什么、下一篇讲什么、本篇不能碰什么、本篇读者心声、本篇主场景、跑偏风险。

### Step 2：写前卡片

写前卡片负责文章好不好看。必须写：标题、读者、独有情绪、读者内心台词、认知推进、叙事形状、主场景、主比喻、情绪递进、禁用内容、失败判定。

### Step 3：正文初稿

输出 doocs / 公众号可复制正文。不要边写边解释规则。

### Step 4：严格评审

用 15 个评审角色逐项审。所有评审先找失败点，再判断是否通过。任一硬门槛必须改，则立即返工。

### Step 5：复审

修改后必须复审同一批硬门槛，不允许只改局部就交付。

### Step 6：发布配套

输出主标题、备选标题、摘要、封面大字、封面副标题、封面提示词。

### Step 7：归档

文章最终通过后，必须归档：

- 最终正文写入 `archive_published/ARTICLE_XX_标题.md`；
- 当前契约写入 `contracts/ARTICLE_CONTRACT_XX.md`；
- 若出现新失败样本，追加到 `context_pack/05_FAILURE_EXAMPLES.md`；
- 若出现新封面规则，追加到 `skills/WECHAT_COVER_IMAGE_SKILL.md`。

## 不允许的 shortcuts

- 不允许跳过 Series Alignment Card；
- 不允许只给正文不评审；
- 不允许用“整体还行”通过；
- 不允许一篇文章写完后不归档；
- 不允许把前一篇内容重新写一遍；
- 不允许提前展开下一篇主题。
