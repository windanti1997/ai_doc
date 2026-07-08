# Agent External Brain MVP 需求文档

版本：v0.3
项目名称：agent-external-brain
命令简称：aeb
产品形态：Agent-native 本地记忆运行时
优先支持：Codex、Claude Code
核心原则：Hook-first、Skill-first、Auto-trigger-first
MVP 定位：增强型 MVP，必须包含自动巩固、技能晋升、技能回滚、记忆老化、记忆覆盖、记忆图谱的最小可用闭环。

---

# 1. 背景

当前 Coding Agent 在真实开发任务中不是完全“不会做”，而是缺少持续积累经验的能力。

典型问题包括：

1. 每次进入项目都像第一次接触。
2. 用户纠正过的问题，下次仍然重复发生。
3. Agent 会重复尝试看似合理但历史上已经失败过的路径。
4. 长任务中断后，Agent 难以恢复目标、进度、假设和未验证事项。
5. 任务日志、diff、测试结果、用户反馈没有沉淀成可复用经验。
6. 现有记忆方案多停留在“存储和召回”，缺少巩固、老化、覆盖、技能化和图谱化。
7. 如果记忆依赖用户手动维护，用户成本过高，无法长期使用。
8. 如果记忆只追加不整理，系统会长期积累噪声、过期规则和错误经验。

因此，Agent External Brain 的目标不是给 Agent 增加一个笔记本，而是构建一个嵌入 Coding Agent 生命周期的本地经验运行时。

它需要让 Agent 在每次任务中自动经历：

```text id="t6jfw2"
捕获经历
→ 形成 episode
→ 提取候选记忆
→ 巩固成规则 / 触发器 / 技能
→ 后续自动召回或触发
→ 使用后记录效果
→ 老化、覆盖、回滚或升级
```

---

# 2. 产品定位

Agent External Brain 是面向 Codex、Claude Code 等 Coding Agent 的本地外置经验系统。

它通过 hook、skill、instructions、CLI 控制面嵌入 Agent 工作流，让 Agent 自动完成：

1. 任务开始前召回项目记忆。
2. 命令执行前检查历史坑点。
3. 文件修改后触发路径相关提醒。
4. 用户纠正后生成候选记忆。
5. 任务结束后生成 episode。
6. 任务结束后自动运行 Dream consolidation。
7. 从重复成功流程中生成 skill candidate。
8. 发现坏 skill 后自动降权或回滚。
9. 长期未验证记忆自动老化降权。
10. 新记忆与旧记忆冲突时执行 supersession。
11. 在 memory graph 中建立 episode、rule、skill、incident 的关系。
12. 通过 benchmark 验证记忆是否真的改善 Agent 行为。

一句话定位：

> Agent External Brain 让 Coding Agent 从每次任务中积累可验证、可触发、可老化、可回滚、可技能化的工程经验。

核心原则：

```text id="muj1dj"
Zero Manual Memory Ops
```

即：

> 默认情况下，用户不应该为了让 Agent 记忆生效而手动运行任何 aeb 命令。

用户只在以下场景介入：

1. 初始化项目。
2. 审核中高风险候选记忆。
3. 处理 inbox / quarantine。
4. 查看或修改记忆。
5. 调试 recall / trigger。
6. 运行 benchmark。
7. 手动回滚错误 skill 或错误 memory。

---

# 3. MVP 核心目标

v0.3 MVP 不再只是“记录和召回”，而是必须跑通一个最小成长闭环：

```text id="90f6ll"
Experience
→ Episode
→ Candidate Memory
→ Dream Consolidation
→ Active Memory / Skill Candidate / Graph Edge
→ Recall / Trigger / Skill Use
→ Usage Feedback
→ Aging / Supersession / Rollback
```

MVP 必须证明：

1. Agent 可以在 Codex / Claude Code 生命周期中自动调用 AEB。
2. AEB 可以自动召回、触发、捕获、写 episode。
3. 任务结束后 AEB 可以自动做轻量 Dream consolidation。
4. 重复成功流程可以生成 skill candidate。
5. skill 使用失败后可以自动降权或回滚。
6. 长期未验证记忆可以自动 aging。
7. 新旧记忆冲突时可以 supersede 旧记忆。
8. episode、memory、skill、incident 可以形成最小 memory graph。
9. benchmark 能验证这些机制是否让 Agent 少犯错、少重复、恢复更快、污染更少。

---

# 4. MVP 必须包含的高级能力

本版 MVP 明确包含以下能力，但每项只做最小可用版本。

## 4.1 Dream Consolidation：任务结束后自动巩固

MVP 要求：

1. 每次 task_stop 后自动运行 lightweight dream。
2. Dream 不做复杂推理平台，只做规则化巩固。
3. Dream 输入为最近 session 的 episode、events、candidate memories。
4. Dream 输出为：

   * 合并后的重复候选记忆；
   * 新 project rule candidate；
   * 新 prospective memory candidate；
   * 新 skill candidate；
   * stale memory warning；
   * conflict / supersession candidate；
   * memory graph edges。

MVP 成功标准：

```text id="s4mmt6"
任务结束后，不需要用户手动运行 dream，AEB 自动整理本次经验，并产生可审计的 consolidation report。
```

---

## 4.2 Skill Promotion：重复成功流程升级为 skill

MVP 要求：

1. Dream 检测最近 episodes 中重复出现的成功命令序列。
2. 如果同一 repo / path pattern 下至少出现 2 次成功流程，可生成 skill_candidate。
3. 默认不自动激活 skill。
4. 低风险 skill 可以进入 inbox 等待用户确认。
5. skill 必须包含：

   * trigger；
   * steps；
   * evidence episodes；
   * scope；
   * confidence；
   * rollback pointer；
   * last_verified_at。

MVP 成功标准：

```text id="73wb4n"
如果 Agent 多次执行“修改 migration → generate schema → run integration test”并成功，AEB 应生成 update-schema-safely skill candidate。
```

---

## 4.3 Skill Rollback：坏 skill 自动降权或回滚

MVP 要求：

1. 每次 skill 被触发或建议后，记录 usage。
2. 如果 skill 关联命令失败，记录 failure。
3. 如果连续失败达到阈值，skill 自动降权。
4. 如果 failure 影响严重，skill 进入 disabled 状态。
5. 用户可以手动 rollback 到上一版本。
6. Dream 可以生成 rollback recommendation。

MVP 成功标准：

```text id="jefzil"
当一个已激活 skill 的命令在新项目状态下失败，AEB 不应继续高置信度推荐它，而应降低 confidence 或进入 rollback candidate。
```

---

## 4.4 Memory Aging：长期未验证记忆自动降权

MVP 要求：

1. 每条 active memory 必须有 `last_verified_at`。
2. 每条 memory 可以有 `half_life_days`。
3. task_start / dream 时计算 aging score。
4. 长期未验证记忆不立即删除，而是降权。
5. recall 时对 stale memory 输出 warning。
6. 如果 stale memory 被重新验证成功，刷新 `last_verified_at`。
7. 如果 stale memory 被证明错误，触发 supersession 或 invalidation。

MVP 成功标准：

```text id="c6qp0t"
如果一条“项目使用 Jest”的记忆长期未验证，在后续测试相关任务中应被标记为 stale，并提示 Agent 先检查 package.json。
```

---

## 4.5 Supersession：新记忆覆盖旧记忆

MVP 要求：

1. Dream 或 candidate accept 时检测冲突。
2. 如果新记忆和旧记忆在同一 scope 下冲突，生成 supersession candidate。
3. 用户确认或 policy 允许后，旧 memory 状态变为 superseded。
4. 新 memory 记录 `supersedes`。
5. 旧 memory 记录 `superseded_by`。
6. recall 默认不召回 superseded memory。
7. audit 记录覆盖原因。

MVP 成功标准：

```text id="pg1s43"
如果项目从 Jest 迁移到 Vitest，AEB 应把“项目使用 Jest”标记为 superseded，而不是让两条冲突记忆同时 active。
```

---

## 4.6 Memory Graph：episode、rule、skill、incident 建图

MVP 要求：

1. 建立最小 memory graph，不做复杂图数据库。
2. 使用本地 JSON / YAML 存储 graph nodes 和 edges。
3. 节点类型至少包括：

   * episode；
   * memory；
   * candidate；
   * skill；
   * incident；
   * checkpoint。
4. 边类型至少包括：

   * derived_from；
   * evidence_for；
   * triggers；
   * supersedes；
   * conflicts_with；
   * promoted_to_skill；
   * failed_by；
   * verified_by。
5. Dream 自动写入 graph edges。
6. `aeb graph show <id>` 可以查看某条 memory 的来源链。

MVP 成功标准：

```text id="x7h2rf"
用户可以追溯一条 project rule 是从哪些 episode、哪些测试结果、哪些用户纠正中来的。
```

---

# 5. 非目标

即使 v0.3 MVP 纳入高级机制，也明确不做：

1. 不做复杂 Web UI。
2. 不做云端同步。
3. 不做团队权限系统。
4. 不做完整 Team Memory 审批平台。
5. 不做远程 Memory Server。
6. 不把 MCP 作为 MVP 必需项。
7. 不做复杂向量数据库依赖。
8. 不做完整图数据库。
9. 不做自主改代码 Agent。
10. 不做不可审计的自动长期记忆写入。
11. 不做无限制自动 skill 激活。
12. 不自动执行高风险命令。
13. 不让用户手动维护主流程。

MVP 中的 Dream、Skill、Aging、Supersession、Graph 都是本地、轻量、可审计版本。

---

# 6. 技术路线原则

## 6.1 CLI Runtime 是 MVP 主执行入口

MVP 中，AEB 的核心能力通过 CLI 暴露。

Codex / Claude Code hooks 调用 CLI。

```text id="zhepi9"
Codex Hook / Claude Hook
→ aeb hook ...
→ AEB Runtime
→ Memory Store / Dream / Skill / Graph
→ Hook Response
```

MCP 不是 MVP 必需项。

后续可以在 CLI Runtime 之上增加 MCP Server，但 MVP 不依赖 MCP。

---

## 6.2 Codex / Claude Code 原生机制优先

MVP 优先利用：

```text id="poe387"
Codex：
- hooks
- AGENTS.md
- skills
- non-interactive / benchmark mode

Claude Code：
- hooks
- CLAUDE.md
- .claude/rules/
- skills
- compaction lifecycle
```

AEB 不应要求用户绕开这些原生机制。

---

## 6.3 Hook 负责确定性动作

Hook 负责：

1. task_start 自动 recall。
2. command_before 自动 trigger。
3. file_edit_after 自动 trigger。
4. command_after 自动 capture。
5. user_prompt_submit 自动 correction capture。
6. task_stop 自动 episode + dream。
7. post_compact 自动 recall critical memory。

---

## 6.4 Skill 负责 Agent 行为习惯

Skill 负责：

1. 让 Agent 理解 recall pack。
2. 让 Agent 根据 trigger warning 修改计划。
3. 让 Agent 在长任务中主动 checkpoint。
4. 让 Agent 在任务结束前进行 memory-aware review。
5. 让 Agent 使用已激活 skill。
6. 让 Agent 在 skill 失败时报告 failure evidence。

---

# 7. 核心用户路径

## 7.1 初始化

用户只运行一次：

```bash id="vyc2h6"
aeb init --agent codex
```

或：

```bash id="lekunt"
aeb init --agent claude-code
```

初始化完成：

1. 创建 `.aeb/`。
2. 生成 config / policy。
3. 生成 hooks 配置。
4. 生成 skills。
5. 生成 AGENTS.md 或 CLAUDE.md 片段。
6. 生成默认 dream policy。
7. 生成默认 skill promotion policy。
8. 生成默认 aging policy。
9. 生成本地 memory graph 文件。
10. 输出安全提示。

---

## 7.2 用户正常使用 Coding Agent

用户说：

```text id="tjir59"
帮我加一个依赖。
```

AEB 自动：

1. task_start recall。
2. 读取 project memory。
3. 发现 repo 使用 pnpm。
4. Agent 计划中使用 pnpm。
5. command_before 检查是否误用 npm。
6. command_after 记录命令结果。
7. task_stop 生成 episode。
8. dream 运行轻量巩固。
9. graph 写入 episode → memory evidence 边。

用户不需要手动运行任何记忆命令。

---

## 7.3 用户纠正 Agent

用户说：

```text id="9zk6wp"
这个项目别用 npm，以后都用 pnpm。
```

AEB 自动：

1. 捕获 user_prompt_submit。
2. 识别 user correction。
3. 生成 project_fact candidate。
4. 生成 anti_pattern candidate。
5. 生成 prospective candidate。
6. 根据 policy：

   * project_fact 可自动接受；
   * anti_pattern 进入 inbox；
   * prospective 进入 inbox 或自动接受；
7. graph 写入 user_correction → candidate 边。
8. task_stop 后 dream 合并重复候选。

---

## 7.4 重复成功流程升级 skill

连续两次任务中，Agent 都执行：

```text id="flv9pg"
修改 migrations/**
→ pnpm db:generate
→ pnpm test:integration -- schema
```

Dream 检测到重复成功流程，生成：

```text id="96mhm6"
skill_candidate: update-schema-safely
```

用户在 inbox 中接受后，skill 状态变为 active。

后续 Agent 修改 migrations 时，AEB 触发：

```text id="34treq"
建议使用 active skill: update-schema-safely
```

---

## 7.5 坏 skill 降权 / 回滚

项目升级后，旧 skill 中的命令失效：

```bash id="5ngpa5"
pnpm test:integration -- schema
```

失败。

AEB 自动：

1. 记录 skill usage failure。
2. skill failure_count +1。
3. confidence 降低。
4. 如果连续失败达到阈值，状态变为 degraded。
5. Dream 生成 rollback recommendation。
6. 如果存在旧版本，可 rollback。
7. graph 写入 skill → failed_by → episode。

---

## 7.6 旧记忆被新记忆覆盖

旧记忆：

```text id="m3y44l"
项目使用 Jest。
```

新 episode 观察到：

```text id="pdxvei"
package.json 使用 Vitest，vitest.config.ts 存在。
```

Dream 发现冲突，生成 supersession candidate：

```text id="o8wt99"
old: project uses Jest
new: project uses Vitest
decision: supersede old memory
```

用户确认或 policy 自动处理后：

1. old memory 状态变为 superseded。
2. new memory 状态变为 active。
3. graph 写入 new → supersedes → old。
4. recall 默认只返回 Vitest。

---

# 8. 功能需求

---

## 8.1 Agent 初始化与配置生成

命令：

```bash id="x3urg7"
aeb init --agent codex
aeb init --agent claude-code
aeb init --agent both
aeb init --agent generic
```

需求：

1. 创建 `.aeb/` 工作区。
2. 创建 `config.yaml`。
3. 创建 `policy.yaml`。
4. 创建 `dream-policy.yaml`。
5. 创建 `skill-policy.yaml`。
6. 创建 `aging-policy.yaml`。
7. 创建 `graph.yaml` 或 `graph.json`。
8. 生成 Codex / Claude Code hooks。
9. 生成 Codex / Claude Code skills。
10. 生成 AGENTS.md / CLAUDE.md 片段。
11. 生成 benchmark 目录。
12. 不覆盖用户已有配置，采用 merge 或提示确认。

验收标准：

1. 初始化后 Agent 可以自动调用 AEB。
2. 初始化后 graph store 存在。
3. 初始化后 dream / skill / aging policy 存在。
4. 用户无需手动执行 start / recall / stop 主流程命令。

---

## 8.2 自动 Session 管理

触发：

```text id="0oxnmq"
task_start / session_start
```

需求：

1. 自动创建 session。
2. 自动记录 repo、branch、agent、model、cwd。
3. 如果已有 dangling session，自动恢复或归档。
4. task_stop 时自动结束 session。
5. session 和所有 events 关联。

验收标准：

1. 用户无需手动 start / stop。
2. task_start 自动产生 session。
3. task_stop 自动关闭 session。
4. session 能生成 episode。

---

## 8.3 自动 Recall

触发：

1. task_start；
2. resume；
3. post_compact；
4. Agent 主动调用 skill；
5. benchmark lifecycle simulation。

需求：

1. 按 repo scope 过滤。
2. 按 path / command / intent 匹配。
3. 返回 retrieval pack。
4. 包含 aging warning。
5. 包含 active skill suggestion。
6. 默认不返回 superseded / quarantined memory。
7. 如果 stale memory 命中，提醒先验证。

输出：

```text id="vywdf4"
Relevant AEB memories:
- This repo uses pnpm.
- Avoid npm install.
- Migration changes should run schema generation.
- Skill available: update-schema-safely.
- Warning: test command memory is stale; verify package.json before use.
```

验收标准：

1. task_start 自动 recall。
2. post_compact 自动重新注入关键记忆。
3. aging / supersession 状态影响 recall。
4. skill suggestion 可以进入 Agent plan。

---

## 8.4 自动 Trigger Check

触发：

1. command_before；
2. file_edit_after；
3. user_prompt_submit；
4. test_after。

需求：

1. command pattern 匹配 anti-pattern。
2. path pattern 匹配 prospective memory。
3. active skill trigger 匹配后推荐 skill。
4. degraded skill 不默认推荐。
5. stale memory 只提示验证，不强制行动。
6. high-risk policy 可 block。
7. 所有 trigger 写 audit 和 graph usage edge。

验收标准：

1. `npm install` 可触发 pnpm memory。
2. `migrations/**` 修改可触发 schema skill / prospective memory。
3. degraded skill 不再高优先级触发。
4. superseded memory 不触发。

---

## 8.5 用户纠正自动转候选记忆

触发：

```text id="42g6kj"
user_prompt_submit
```

需求：

1. 自动识别纠正、偏好、规则、未来条件。
2. 生成 candidate memory。
3. 标注 source=user_correction。
4. 标注 scope=current_repo。
5. 进入 policy engine。
6. 写 graph edge：user_message → candidate。
7. task_stop dream 时可进一步合并。

验收标准：

1. “以后别用 npm”生成 anti_pattern candidate。
2. “我希望文档完整展开”生成 user_preference candidate。
3. “修改 migration 后要跑测试”生成 prospective candidate。
4. 高风险内容进入 quarantine。

---

## 8.6 Task Stop 自动 Episode Writer

触发：

```text id="5xykki"
stop / session_end
```

需求：

1. 汇总 session events。
2. 提取 task goal。
3. 提取 commands run。
4. 提取 files changed。
5. 提取 failures。
6. 提取 verification。
7. 提取 user feedback。
8. 生成 episode。
9. 写 graph node：episode。
10. 写 edges：event → episode。
11. 触发 lightweight dream。

验收标准：

1. task_stop 自动写 episode。
2. episode 包含验证状态。
3. episode 进入 graph。
4. episode 能被 dream 使用。

---

## 8.7 Dream Consolidation MVP

触发：

1. task_stop 后自动运行；
2. idle_post_task；
3. 手动 fallback：`aeb dream run`；
4. benchmark 中模拟运行。

需求：

Dream MVP 做 7 件事：

1. 合并重复 candidate；
2. 从 episode 中提取 project_fact candidate；
3. 从失败路径中提取 anti_pattern candidate；
4. 从路径 + 验证结果中提取 prospective candidate；
5. 从重复成功流程中生成 skill_candidate；
6. 检测 stale memory；
7. 检测冲突并生成 supersession candidate；
8. 写 memory graph edges。

Dream 不做：

1. 不自动激活高风险 skill；
2. 不删除 memory；
3. 不执行代码；
4. 不做大型离线推理。

Dream report 示例：

```yaml id="6ki8hd"
dream_report:
  session_id: sess_001
  episode_id: ep_001

  created_candidates:
    - cand_project_pnpm
    - cand_anti_npm
    - cand_pro_migration_check

  skill_candidates:
    - skill_update_schema_safely

  stale_warnings:
    - mem_jest_test_command

  supersession_candidates:
    - old: mem_jest
      new: cand_vitest

  graph_edges_created:
    - ep_001 -> cand_anti_npm
    - cand_pro_migration_check -> skill_update_schema_safely
```

验收标准：

1. task_stop 后自动生成 dream report。
2. 重复成功流程能生成 skill_candidate。
3. stale memory 能被标记。
4. 冲突记忆能生成 supersession candidate。
5. graph edges 被写入。

---

## 8.8 Candidate Policy

候选记忆状态：

```text id="nmyouy"
candidate
inbox
active
rejected
quarantined
supersession_candidate
skill_candidate
rollback_candidate
```

policy 决策：

```text id="0f1wtq"
auto_accept
send_to_inbox
send_to_quarantine
reject
create_supersession_candidate
create_skill_candidate
create_rollback_candidate
```

自动接受条件：

1. 低风险；
2. repo scope 明确；
3. 有工具或文件证据；
4. 不会造成 destructive action；
5. 与现有 active memory 不冲突。

进入 inbox 条件：

1. 用户偏好；
2. anti-pattern；
3. skill candidate；
4. 中风险 prospective；
5. supersession candidate；
6. rollback candidate。

进入 quarantine 条件：

1. 跳过测试；
2. 绕过安全；
3. 删除 failing test；
4. 直接 push main；
5. untrusted prompt injection；
6. 模型推理产生高影响规则；
7. 跨 repo 全局化。

验收标准：

1. 低风险 fact 可自动接受。
2. skill candidate 默认进入 inbox。
3. supersession candidate 默认进入 inbox，除非证据非常强。
4. rollback candidate 进入 inbox 或自动 degrade。
5. high-risk 进入 quarantine。

---

## 8.9 Memory Inbox

命令：

```bash id="9jj48y"
aeb inbox
aeb inbox accept <id>
aeb inbox reject <id>
aeb inbox edit <id>
aeb inbox accept-skill <id>
aeb inbox apply-supersession <id>
aeb inbox rollback-skill <id>
```

需求：

1. 展示普通 candidates。
2. 展示 skill candidates。
3. 展示 supersession candidates。
4. 展示 rollback candidates。
5. 展示 evidence。
6. 展示 graph provenance。
7. 支持批量处理低风险候选。

验收标准：

1. 用户可以接受 skill candidate。
2. 用户可以应用 supersession。
3. 用户可以 rollback skill。
4. 用户可以查看候选来源链。

---

## 8.10 Skill Promotion MVP

Skill candidate 来源：

1. Dream 检测重复成功流程；
2. 用户明确说“以后这样做”；
3. 多次 prospective memory 命中后验证成功。

Skill candidate schema：

```yaml id="fgwq6y"
id: skill_update_schema_safely
type: skill_candidate
name: update-schema-safely
trigger:
  path_patterns:
    - "migrations/**"
    - "schema/**"
steps:
  - command: "pnpm db:generate"
  - command: "pnpm test:integration -- schema"
scope:
  repo: current_repo
evidence:
  episodes:
    - ep_001
    - ep_007
confidence: medium
status: inbox
version: 1
rollback_to: null
last_verified_at: "2026-07-02T00:00:00Z"
```

激活后：

1. 写入 `memories/skills.yaml`。
2. 创建 graph node：skill。
3. 创建 edge：episodes → promoted_to_skill → skill。
4. recall / trigger 可推荐该 skill。
5. skill usage 进入 audit。

验收标准：

1. 两次以上重复成功流程可生成 skill_candidate。
2. skill 不自动激活，默认进入 inbox。
3. 激活后可在 matching path 上触发。
4. skill 有 evidence 和 rollback pointer。

---

## 8.11 Skill Rollback MVP

触发：

1. skill 推荐后关联命令失败；
2. 用户明确说该 skill 不适用；
3. Dream 发现 skill 与新 memory 冲突；
4. skill 长期未验证。

Skill 状态：

```text id="6vyqya"
active
degraded
disabled
rolled_back
superseded
```

降权规则：

```text id="h7fpr2"
一次失败：confidence 降一级，记录 failure。
连续两次失败：状态变 degraded。
严重失败或用户否定：状态变 disabled，并生成 rollback candidate。
```

Rollback 行为：

1. 禁用当前版本；
2. 恢复上一版本，若存在；
3. 若无上一版本，skill 保留 disabled；
4. 写 audit；
5. 写 graph edge：skill_version_n → rolled_back_to → skill_version_n-1。

验收标准：

1. skill 失败后不再高优先级推荐。
2. 用户可以手动 rollback。
3. Dream 可以生成 rollback recommendation。
4. rollback 过程可审计。

---

## 8.12 Memory Aging MVP

每条 memory 增加字段：

```yaml id="uslcv6"
lifecycle:
  created_at:
  last_retrieved_at:
  last_triggered_at:
  last_verified_at:
  half_life_days:
  aging_score:
  stale: false
```

Aging 计算触发：

1. task_start；
2. dream；
3. manual：`aeb memory age`;
4. benchmark。

最小规则：

```text id="8b66es"
如果 now - last_verified_at > half_life_days，则 stale=true。
stale memory recall 降权。
stale memory 命中时提示 re-verify。
重新验证成功则刷新 last_verified_at。
验证失败则生成 supersession / invalidation candidate。
```

默认 half-life：

```yaml id="7z3w92"
project_fact: 90
test_command: 30
dependency_rule: 30
user_preference: 180
anti_pattern: 120
prospective: 90
skill: 30
incident: 365
```

验收标准：

1. 旧测试命令记忆会 stale。
2. stale memory 不直接删除。
3. stale memory recall 时提示查证。
4. 成功验证后刷新 last_verified_at。
5. 失败后触发 supersession candidate。

---

## 8.13 Supersession MVP

冲突检测范围：

1. 同一 repo；
2. 同一 memory type；
3. 同一 topic；
4. 内容互斥。

MVP topic 识别可用规则：

```text id="jct47r"
package_manager
test_framework
test_command
build_command
generated_file_policy
migration_policy
```

示例：

```text id="5rx8tn"
old: package_manager = npm
new: package_manager = pnpm

old: test_framework = Jest
new: test_framework = Vitest
```

Supersession candidate schema：

```yaml id="4igbjh"
id: sup_001
type: supersession_candidate
old_memory: mem_jest
new_memory: cand_vitest
reason:
  - "Same repo and topic: test_framework"
  - "New evidence found in package.json"
decision: pending
```

应用后：

1. old memory status = superseded；
2. new memory status = active；
3. old.superseded_by = new.id；
4. new.supersedes = old.id；
5. graph edge：new → supersedes → old；
6. audit 记录。

验收标准：

1. Jest → Vitest 可生成 supersession candidate。
2. 应用后 recall 不返回 old memory。
3. 旧 memory 不删除，可追溯。
4. graph 可以显示覆盖关系。

---

## 8.14 Memory Graph MVP

Graph 存储：

```text id="oef9np"
.aeb/graph/nodes.jsonl
.aeb/graph/edges.jsonl
```

节点类型：

```text id="vqfmd4"
event
episode
candidate
memory
skill
skill_version
checkpoint
incident
dream_report
```

边类型：

```text id="n3m93k"
derived_from
evidence_for
triggered_by
used_in
verified_by
failed_by
promoted_to_skill
supersedes
conflicts_with
rolled_back_to
aged_from
```

命令：

```bash id="1dj7oc"
aeb graph show <id>
aeb graph trace <memory-id>
aeb graph related <episode-id>
```

MVP 输出文本树即可：

```text id="zfhg44"
mem_project_pnpm
├─ derived_from: cand_project_pnpm
│  └─ derived_from: ep_001
│     └─ evidence: package.json, pnpm-lock.yaml
└─ triggered_in: ep_007
```

验收标准：

1. episode 写入 graph node。
2. candidate 写入 graph node。
3. memory accept 写入 graph node 和 evidence edge。
4. skill promotion 写入 promoted_to_skill edge。
5. supersession 写入 supersedes edge。
6. rollback 写入 rolled_back_to edge。
7. 用户能 trace 一条 memory 的来源。

---

# 9. 核心数据结构

## 9.1 Memory Object

```yaml id="x386uh"
id:
type:
  - project_fact
  - user_preference
  - anti_pattern
  - prospective
  - skill
  - incident
summary:
content:
source:
  kind:
  actor:
  artifact:
  timestamp:
evidence:
  - type:
    ref:
scope:
  level:
  repo:
  paths:
trigger:
  events:
  command_patterns:
  path_patterns:
  intent_keywords:
action_effect:
  type:
  message:
  suggested_commands:
confidence:
risk:
status:
  - active
  - inbox
  - rejected
  - quarantined
  - archived
  - superseded
  - degraded
  - disabled
lifecycle:
  created_at:
  updated_at:
  last_verified_at:
  last_retrieved_at:
  last_triggered_at:
  half_life_days:
  stale:
  aging_score:
version:
  current:
  supersedes:
  superseded_by:
usage:
  retrieved_count:
  triggered_count:
  acted_on_count:
  success_count:
  failure_count:
graph:
  node_id:
```

---

## 9.2 Skill Object

```yaml id="e4rghm"
id:
name:
summary:
trigger:
  events:
  path_patterns:
  command_patterns:
  intent_keywords:
steps:
  - type: command
    command:
    expected_result:
    fallback:
scope:
  repo:
  paths:
evidence:
  episodes:
  memories:
confidence:
risk:
status:
  - candidate
  - active
  - degraded
  - disabled
  - rolled_back
version:
  number:
  previous:
  rollback_to:
usage:
  suggested_count:
  executed_count:
  success_count:
  failure_count:
lifecycle:
  created_at:
  last_verified_at:
  half_life_days:
graph:
  node_id:
```

---

## 9.3 Dream Report

```yaml id="2nz1df"
id:
session_id:
episode_id:
created_at:
inputs:
  episodes:
  candidates:
  memories:
outputs:
  created_candidates:
  merged_candidates:
  skill_candidates:
  stale_warnings:
  supersession_candidates:
  rollback_candidates:
  graph_edges:
summary:
```

---

## 9.4 Graph Node

```yaml id="1x17ea"
id:
type:
  - event
  - episode
  - candidate
  - memory
  - skill
  - skill_version
  - incident
  - checkpoint
  - dream_report
ref:
summary:
created_at:
metadata:
```

---

## 9.5 Graph Edge

```yaml id="f6q8w3"
id:
type:
  - derived_from
  - evidence_for
  - triggered_by
  - used_in
  - verified_by
  - failed_by
  - promoted_to_skill
  - supersedes
  - conflicts_with
  - rolled_back_to
  - aged_from
from:
to:
created_at:
metadata:
```

---

# 10. CLI 控制面

主流程自动运行，但 CLI 需要覆盖治理和调试。

## 10.1 初始化

```bash id="x82a86"
aeb init --agent codex
aeb init --agent claude-code
aeb init --agent both
```

## 10.2 Hook 入口

```bash id="hljhk5"
aeb hook codex session-start
aeb hook codex user-prompt-submit
aeb hook codex pre-tool-use
aeb hook codex post-tool-use
aeb hook codex stop

aeb hook claude session-start
aeb hook claude user-prompt-submit
aeb hook claude pre-tool-use
aeb hook claude post-tool-use
aeb hook claude stop
```

## 10.3 Memory 治理

```bash id="u81cbm"
aeb inbox
aeb inbox accept <id>
aeb inbox reject <id>
aeb quarantine
aeb memory list
aeb memory show <id>
aeb memory supersede <old> <new>
aeb memory age
```

## 10.4 Dream

```bash id="7c2rnz"
aeb dream run
aeb dream show <id>
```

虽然 task_stop 会自动运行 dream，但保留手动命令用于调试。

## 10.5 Skill

```bash id="e0nvdj"
aeb skill list
aeb skill show <id>
aeb skill accept <candidate-id>
aeb skill disable <id>
aeb skill rollback <id>
aeb skill usage <id>
```

## 10.6 Graph

```bash id="xm9mfn"
aeb graph show <id>
aeb graph trace <id>
aeb graph related <id>
```

## 10.7 Benchmark

```bash id="qenbpu"
aeb bench run
aeb bench run pnpm-vs-npm
aeb bench run dream-skill-promotion
aeb bench run skill-rollback
aeb bench run memory-aging
aeb bench run supersession
aeb bench run memory-graph
```

---

# 11. Codex / Claude Code 集成需求

## 11.1 Codex

`aeb init --agent codex` 应生成：

1. Codex hook 配置。
2. Codex skills。
3. AGENTS.md 片段。
4. AEB hook 命令。
5. AEB memory instructions。

Codex 生命周期映射：

```text id="llrsc6"
SessionStart → recall + aging check
UserPromptSubmit → correction capture
PreToolUse → trigger check + skill suggestion
PostToolUse → event capture + skill usage result
PostCompact → recall critical memory
Stop → episode writer + dream consolidation
```

## 11.2 Claude Code

`aeb init --agent claude-code` 应生成：

1. Claude Code hook 配置。
2. Claude skills。
3. CLAUDE.md 片段。
4. .claude/rules/ 片段。
5. AEB hook wrapper。

Claude Code 生命周期映射：

```text id="8uvhi2"
SessionStart → recall + aging check
UserPromptSubmit → correction capture
PreToolUse → trigger check + skill suggestion
PostToolUse → event capture + skill usage result
PostCompact → checkpoint + recall critical memory
Stop → episode writer + dream consolidation
SessionEnd → archive session
```

---

# 12. Benchmark 需求

v0.3 MVP 必须包含以下 benchmark。

## 12.1 pnpm-vs-npm

验证：

1. 用户纠正自动生成记忆。
2. 第二次任务自动避免 npm。
3. command_before trigger 生效。
4. Dream 能合并 pnpm 相关候选。
5. graph 能追踪来源。

## 12.2 interrupted-task

验证：

1. checkpoint 自动写入。
2. resume recall 生效。
3. episode 记录中断状态。

## 12.3 migration-trigger

验证：

1. path trigger 生效。
2. prospective memory 提醒验证。
3. 重复成功后生成 skill candidate。

## 12.4 dream-skill-promotion

验证：

1. 两次以上重复成功流程生成 skill_candidate。
2. skill candidate 带 evidence。
3. graph 建立 promoted_to_skill 边。

## 12.5 skill-rollback

验证：

1. active skill 失败后 failure_count 增加。
2. confidence 降低。
3. 连续失败后状态 degraded / disabled。
4. rollback candidate 生成。

## 12.6 memory-aging

验证：

1. 旧 memory 被标记 stale。
2. recall 降权。
3. stale warning 输出。
4. 验证成功刷新 last_verified_at。

## 12.7 supersession

验证：

1. Jest → Vitest 生成 supersession candidate。
2. 应用后旧 memory 变 superseded。
3. recall 不再返回旧 memory。
4. graph 写 supersedes edge。

## 12.8 memory-graph

验证：

1. episode、candidate、memory、skill 节点写入。
2. derived_from、evidence_for、promoted_to_skill、supersedes 边写入。
3. `aeb graph trace <id>` 可追溯来源。

---

# 13. 验收标准总表

MVP 完成后必须满足：

1. 支持 `aeb init --agent codex`。
2. 支持 `aeb init --agent claude-code`。
3. Codex / Claude Code 可通过 hooks 自动调用 AEB。
4. 用户无需手动 start / stop / recall / trigger。
5. task_start 自动 recall。
6. command_before 自动 trigger。
7. user_prompt_submit 自动捕获纠正。
8. task_stop 自动写 episode。
9. task_stop 自动运行 Dream consolidation。
10. Dream 能生成 candidate memory。
11. Dream 能生成 skill_candidate。
12. active skill 可被 recall / trigger 推荐。
13. skill 使用失败后能降权。
14. skill 可 rollback。
15. memory aging 能标记 stale。
16. stale memory recall 时降权并提示验证。
17. supersession candidate 能生成。
18. 应用 supersession 后旧 memory 不再 active recall。
19. memory graph 能写 episode / memory / skill 节点。
20. memory graph 能写 derived_from / promoted_to_skill / supersedes 边。
21. inbox 支持处理 skill candidate、supersession candidate、rollback candidate。
22. quarantine 防止高风险记忆污染。
23. audit 记录所有关键动作。
24. benchmark 覆盖 repeated mistake、dream、skill promotion、rollback、aging、supersession、graph。
25. README 展示 Codex 和 Claude Code 两条自动化 demo。

---

# 14. Milestones

## Milestone 1：Core Runtime + Storage

交付：

1. Workspace 初始化。
2. File store。
3. Event store。
4. Memory store。
5. Candidate store。
6. Audit store。
7. Graph store。
8. 基础 schema。

验收：

```text id="9z9yw7"
本地 .aeb 可以保存 events、episodes、memories、graph nodes/edges。
```

---

## Milestone 2：Codex / Claude Code Hook 集成

交付：

1. Codex adapter。
2. Claude Code adapter。
3. Hook config generator。
4. Skill generator。
5. AGENTS.md / CLAUDE.md fragment。

验收：

```text id="nvkz18"
Codex 和 Claude Code 都能通过 hook 自动触发 recall / trigger / capture / stop。
```

---

## Milestone 3：Recall / Trigger / Candidate

交付：

1. Recall engine。
2. Trigger engine。
3. Correction parser。
4. Candidate extractor。
5. Policy engine。

验收：

```text id="y54kte"
用户纠正 pnpm 后，第二次任务能自动避免 npm。
```

---

## Milestone 4：Episode + Dream MVP

交付：

1. Episode writer。
2. Dream runner。
3. Dream report。
4. Candidate merge。
5. Conflict detection。
6. Stale detection。
7. Graph edge writer。

验收：

```text id="5775u7"
task_stop 后自动生成 episode 和 dream report。
```

---

## Milestone 5：Skill Promotion / Rollback

交付：

1. Skill candidate extractor。
2. Skill store。
3. Skill activation。
4. Skill usage tracking。
5. Skill degradation。
6. Skill rollback。

验收：

```text id="aq9f35"
重复成功流程生成 skill candidate；skill 失败后自动降权并可回滚。
```

---

## Milestone 6：Memory Aging / Supersession

交付：

1. Aging calculator。
2. Stale warning。
3. Verification refresh。
4. Conflict detector。
5. Supersession candidate。
6. Supersession apply。

验收：

```text id="kqp6vb"
Jest → Vitest 场景中，旧 memory 被 superseded，新 memory active。
```

---

## Milestone 7：Memory Graph

交付：

1. Graph node writer。
2. Graph edge writer。
3. Graph trace command。
4. Graph related command。
5. Graph integration in dream / skill / supersession。

验收：

```text id="f3o4f1"
任意 active memory 可以追溯到 episode 和 evidence。
```

---

## Milestone 8：Benchmark Suite

交付：

1. Lifecycle simulator。
2. pnpm-vs-npm benchmark。
3. dream-skill-promotion benchmark。
4. skill-rollback benchmark。
5. memory-aging benchmark。
6. supersession benchmark。
7. memory-graph benchmark。
8. Scorecard。

验收：

```text id="v4uj5p"
aeb bench run 能覆盖完整增强型 MVP 闭环。
```

---

# 15. 第一版成功 Demo

## Demo 1：自动避免重复错误

用户第一次纠正：

```text id="db1c46"
这个项目别用 npm，以后都用 pnpm。
```

第二次用户只说：

```text id="46jaos"
帮我加 dayjs。
```

Agent 自动使用：

```bash id="h2589w"
pnpm add dayjs
```

如果 Agent 准备使用 npm，command_before 自动提醒。

成功标准：

```text id="1d29u2"
用户没有手动运行任何 AEB 命令，Agent 自动避免重复错误。
```

---

## Demo 2：Dream 生成 Skill

Agent 两次完成：

```text id="2rdw5q"
修改 migration
→ pnpm db:generate
→ pnpm test:integration -- schema
```

task_stop 后 Dream 自动生成：

```text id="ggay6a"
skill_candidate: update-schema-safely
```

用户接受后，后续修改 migration 自动推荐该 skill。

---

## Demo 3：Skill 失败后回滚

项目命令变化导致旧 skill 失败。

AEB 自动：

1. 记录 skill failure。
2. 降低 confidence。
3. 生成 rollback candidate。
4. 用户 rollback 后旧 skill 不再推荐。

---

## Demo 4：Memory Aging + Supersession

旧 memory：

```text id="x2xm84"
项目使用 Jest。
```

新证据：

```text id="uogqct"
项目使用 Vitest。
```

AEB 自动：

1. 标记 Jest memory stale。
2. 发现冲突。
3. 生成 supersession candidate。
4. 应用后 Vitest active，Jest superseded。
5. graph trace 可看到覆盖关系。

---

## Demo 5：Memory Graph 可追溯

用户运行：

```bash id="2oc0cz"
aeb graph trace mem_project_pnpm
```

输出：

```text id="xgjxak"
mem_project_pnpm
├─ derived_from: cand_project_pnpm
│  └─ derived_from: ep_001
│     ├─ evidence: user correction
│     └─ evidence: pnpm-lock.yaml
└─ triggered_in: ep_007
```

成功标准：

```text id="l7a4wf"
每条重要 memory 都能解释它从哪里来、被何时使用、是否仍然有效。
```

---

# 16. 风险与应对

## 风险 1：MVP 范围过大

应对：

1. Dream 只做 lightweight rule-based consolidation。
2. Skill promotion 只从重复命令序列生成 candidate。
3. Skill rollback 只做 confidence 降权和 disabled。
4. Aging 只做 half-life + stale flag。
5. Supersession 只做固定 topic 冲突检测。
6. Memory graph 用 JSONL，不引入图数据库。

---

## 风险 2：自动机制打扰 Agent

应对：

1. trigger 有 severity。
2. skill suggestion 有 confidence。
3. stale memory 只提醒查证。
4. degraded skill 不主动推荐。
5. 用户可关闭某类 trigger。

---

## 风险 3：错误经验被 skill 化

应对：

1. skill candidate 默认不自动激活。
2. 必须有 evidence episodes。
3. 必须有 scope。
4. skill 有 rollback。
5. skill usage failure 会降权。

---

## 风险 4：旧记忆和新记忆冲突

应对：

1. Supersession candidate 不直接删除旧记忆。
2. 旧记忆保留 historical trace。
3. recall 默认隐藏 superseded memory。
4. graph 保留覆盖关系。

---

## 风险 5：Memory graph 复杂度失控

应对：

1. MVP 只做本地 JSONL graph。
2. 不做复杂图查询。
3. 只支持 trace / related。
4. 只写关键节点和边。

---

# 17. 结论

v0.3 MVP 的核心不是做一个“记忆列表”，而是做一个最小可成长记忆系统。

它必须覆盖：

```text id="rfprqf"
自动捕获
自动召回
自动触发
自动 episode
自动 dream
技能晋升
技能回滚
记忆老化
记忆覆盖
记忆图谱
benchmark 验证
```

最终要证明的是：

```text id="ddysda"
Coding Agent 不只是记住历史，
而是能从历史中形成规则、触发器、技能和可回滚经验，
并在未来任务中自动改变行动。
```

第一版成功标准：

```text id="bpr46x"
用户初始化一次 AEB 后，
正常使用 Codex / Claude Code，
Agent 能自动学习项目经验，
自动避免重复错误，
自动形成技能候选，
自动处理过期和冲突记忆，
并能追溯每条记忆从哪里来。
```
