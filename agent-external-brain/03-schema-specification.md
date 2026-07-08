# Agent External Brain Schema Specification

版本：v0.3
项目名称：agent-external-brain
命令简称：aeb
适用范围：增强型 MVP
目标：固定 AEB 本地文件格式、核心对象 Schema、状态机、索引规则和示例数据，支持 Codex / Claude Code hook-first 集成。

---

# 1. Schema 设计原则

AEB 的 Schema 不服务于“存文本”，而服务于“让记忆未来能改变 Agent 行为”。

因此，每条核心对象都必须尽量回答以下问题：

```text id="xi22f8"
1. 它来自哪里？
2. 它适用于哪里？
3. 它什么时候应该被触发？
4. 它会如何改变 Agent 行为？
5. 它是否被验证过？
6. 它是否过期？
7. 它是否被新记忆覆盖？
8. 它是否参与 skill 晋升或回滚？
9. 它能否追溯来源链？
```

核心约束：

```text id="13ns1d"
所有长期 memory 必须有 source、scope、evidence、status、lifecycle、graph。
所有 skill 必须有 trigger、steps、scope、evidence、usage、version。
所有 dream 输出必须可审计。
所有 supersession 和 rollback 都不能删除历史，只能改变状态。
所有 graph 边必须能追溯到具体对象。
```

---

# 2. 文件格式约定

## 2.1 文件格式选择

MVP 使用本地文件系统，不引入数据库。

```text id="d5ivcw"
YAML：配置、memory、skill、episode、candidate、dream report
JSONL：events、audit、graph nodes、graph edges
JSON：runtime state、indexes
```

原因：

```text id="528e4e"
YAML 适合人类审查和编辑。
JSONL 适合 append-only 日志。
JSON 适合索引和运行时状态。
```

---

## 2.2 时间格式

所有时间统一使用 ISO 8601 UTC 字符串。

```yaml id="w4v6fr"
created_at: "2026-07-02T10:30:00.000Z"
```

---

## 2.3 ID 命名规范

ID 必须可读、可排序、可定位类型。

```text id="0o52ru"
evt_20260702_103000_ab12
sess_20260702_103000_cd34
ep_20260702_104500_ef56
cand_20260702_104501_gh78
mem_20260702_104530_ij90
skill_20260702_105000_kl12
dream_20260702_105500_mn34
graph_node_20260702_105501_op56
graph_edge_20260702_105502_qr78
audit_20260702_105503_st90
```

ID 生成规则：

```text id="s77a43"
<prefix>_<YYYYMMDD>_<HHMMSS>_<short_random>
```

---

# 3. 本地目录结构

初始化后生成：

```text id="tjvp07"
.aeb/
  config.yaml
  policy.yaml
  dream-policy.yaml
  skill-policy.yaml
  aging-policy.yaml

  runtime/
    state.json
    lock

  sessions/
    current.json
    archived/

  events/
    2026-07-02/
      session-xxx.jsonl

  episodes/
    ep-xxx.yaml

  checkpoints/
    ckpt-xxx.yaml

  candidates/
    inbox/
    quarantine/
    accepted/
    rejected/
    supersession/
    rollback/

  memories/
    project.yaml
    user.yaml
    anti-patterns.yaml
    prospective.yaml
    incidents.yaml

  skills/
    candidates.yaml
    active.yaml
    degraded.yaml
    disabled.yaml
    versions/

  dream/
    reports/

  graph/
    nodes.jsonl
    edges.jsonl

  indexes/
    keyword.json
    path.json
    command.json
    topic.json
    scope.json

  audit/
    audit.jsonl

  bench/
    runs/
```

---

# 4. Config Schema

文件：

```text id="tnnb4v"
.aeb/config.yaml
```

示例：

```yaml id="s71kdq"
schema_version: "0.3"

project:
  name: "sample-repo"
  root: "/Users/me/projects/sample-repo"
  repo_id: "sample-repo"
  default_scope: "repo"

agents:
  codex:
    enabled: true
    hook_config_path: ".codex/hooks.json"
    instructions_path: "AGENTS.md"
    skills_path: ".agents/skills"

  claude_code:
    enabled: true
    hook_config_path: ".claude/settings.json"
    instructions_path: "CLAUDE.md"
    skills_path: ".claude/skills"

runtime:
  auto_session: true
  hook_timeout_ms: 10000
  dream_on_task_stop: true
  graph_enabled: true
  audit_enabled: true

recall:
  max_items: 8
  include_stale_warnings: true
  include_skill_suggestions: true
  include_checkpoints: true

trigger:
  default_mode: "warn"
  allow_rewrite: false
  allow_block: true

storage:
  format: "local_files"
  lock_timeout_ms: 3000
```

必填字段：

```text id="t4115s"
schema_version
project.root
runtime.auto_session
runtime.dream_on_task_stop
runtime.graph_enabled
```

---

# 5. Policy Schema

文件：

```text id="kaypjz"
.aeb/policy.yaml
```

示例：

```yaml id="q8fnz2"
schema_version: "0.3"

auto_accept:
  project_fact:
    enabled: true
    max_risk: "low"
    require_scope: true
    allowed_sources:
      - "file_observation"
      - "command_result"
      - "test_result"
      - "tool_observation"

  prospective:
    enabled: true
    max_risk: "low"
    require_scope: true

  user_preference:
    enabled: false

  anti_pattern:
    enabled: false

  skill_candidate:
    enabled: false

inbox:
  include_types:
    - "user_preference"
    - "anti_pattern"
    - "skill_candidate"
    - "supersession_candidate"
    - "rollback_candidate"

quarantine:
  conditions:
    - "skip_tests"
    - "bypass_security"
    - "delete_failing_tests"
    - "direct_push_main"
    - "untrusted_instruction_source"
    - "cross_repo_globalization"
    - "high_impact_model_inference"

risk_keywords:
  critical:
    - "skip all tests"
    - "ignore security"
    - "push directly to main"
    - "delete failing tests"
  high:
    - "bypass"
    - "disable validation"
    - "turn off checks"
```

---

# 6. Dream Policy Schema

文件：

```text id="89888q"
.aeb/dream-policy.yaml
```

示例：

```yaml id="4v95la"
schema_version: "0.3"

enabled: true

run:
  on_task_stop: true
  on_idle: false
  manual_allowed: true

window:
  recent_episode_limit: 20
  recent_days: 14

candidate_merge:
  enabled: true
  similarity:
    same_type_required: true
    same_scope_required: true
    same_topic_required: true

workflow_mining:
  enabled: true
  min_successful_episodes: 2
  max_command_sequence_length: 6
  require_same_path_pattern: true
  require_successful_verification: true

stale_detection:
  enabled: true

conflict_detection:
  enabled: true
  topics:
    - "package_manager"
    - "test_framework"
    - "test_command"
    - "build_command"
    - "generated_file_policy"
    - "migration_policy"
    - "schema_generation_policy"

graph:
  write_edges: true
  write_dream_report_node: true
```

---

# 7. Skill Policy Schema

文件：

```text id="t6bn30"
.aeb/skill-policy.yaml
```

示例：

```yaml id="cdg4g6"
schema_version: "0.3"

promotion:
  enabled: true
  min_evidence_episodes: 2
  auto_activate: false
  require_same_repo: true
  require_same_path_pattern: true
  require_successful_verification: true

activation:
  require_user_acceptance: true
  allow_low_risk_auto_activation: false

usage:
  track_suggested: true
  track_executed: true
  track_success: true
  track_failure: true

failure_policy:
  lower_confidence_on_first_failure: true
  consecutive_failures_to_degrade: 2
  severe_failure_to_disable: true

rollback:
  enabled: true
  keep_versions: true
  auto_rollback: false
```

---

# 8. Aging Policy Schema

文件：

```text id="h7go2p"
.aeb/aging-policy.yaml
```

示例：

```yaml id="r2xy7a"
schema_version: "0.3"

enabled: true

half_life_days:
  project_fact: 90
  test_command: 30
  dependency_rule: 30
  user_preference: 180
  anti_pattern: 120
  prospective: 90
  skill: 30
  incident: 365

recall_behavior:
  stale_memory_downrank: true
  include_stale_warning: true
  require_reverification_for_stale: true

refresh:
  refresh_on_successful_trigger: true
  refresh_on_successful_verification: true

failure:
  create_supersession_candidate_on_failed_verification: true
```

---

# 9. Event Schema

文件：

```text id="6ogok3"
.aeb/events/<date>/session-xxx.jsonl
```

每行一个 Event。

```yaml id="x7t2sd"
id: "evt_20260702_103000_ab12"
schema_version: "0.3"
session_id: "sess_20260702_102900_cd34"

source_agent: "codex"
source_event_name: "PreToolUse"
normalized_type: "pre_tool_use"

timestamp: "2026-07-02T10:30:00.000Z"

cwd: "/Users/me/projects/sample-repo"
repo_root: "/Users/me/projects/sample-repo"
branch: "main"

user_prompt: null

tool:
  name: "Bash"
  input:
    command: "npm install lodash"
  output: null
  exit_code: null
  stdout: null
  stderr: null

command:
  raw: "npm install lodash"
  normalized: "npm install lodash"
  exit_code: null
  stdout: null
  stderr: null

files: []

raw_ref: "raw_event_20260702_103000_ab12"
```

`normalized_type` 可选值：

```text id="djiz1u"
session_start
user_prompt_submit
pre_tool_use
post_tool_use
post_tool_use_failure
file_changed
test_after
pre_compact
post_compact
stop
session_end
```

---

# 10. Session Schema

文件：

```text id="t1ssnx"
.aeb/sessions/current.json
.aeb/sessions/archived/sess-xxx.json
```

示例：

```json id="4r8x85"
{
  "id": "sess_20260702_102900_cd34",
  "schema_version": "0.3",
  "source_agent": "codex",
  "status": "active",
  "started_at": "2026-07-02T10:29:00.000Z",
  "ended_at": null,
  "repo_root": "/Users/me/projects/sample-repo",
  "branch": "main",
  "goal": "Add lodash dependency",
  "event_file": ".aeb/events/2026-07-02/session-sess_20260702_102900_cd34.jsonl",
  "episode_id": null,
  "dream_report_id": null
}
```

`status` 可选值：

```text id="6ygs00"
active
ended
archived
dangling
```

---

# 11. Checkpoint Schema

文件：

```text id="h5hdx1"
.aeb/checkpoints/ckpt-xxx.yaml
```

示例：

```yaml id="rz06h3"
id: "ckpt_20260702_104000_ab12"
schema_version: "0.3"
session_id: "sess_20260702_102900_cd34"

repo: "/Users/me/projects/sample-repo"
branch: "main"

goal: "实现登录页表单校验"

completed:
  - "已完成 email validation"

pending:
  - "补充 password validation"
  - "运行 LoginForm tests"

verified:
  - "email validation 单元测试通过"

unverified:
  - "password rule 尚未测试"

assumptions:
  - "LoginForm.vue 是唯一需要修改的入口"

risks:
  - "未运行完整表单测试"

next_step:
  - "继续修改 password validation"

created_at: "2026-07-02T10:40:00.000Z"

graph:
  node_id: "graph_node_20260702_104000_ckpt"
```

---

# 12. Episode Schema

文件：

```text id="ixcdaw"
.aeb/episodes/ep-xxx.yaml
```

示例：

```yaml id="68q1id"
id: "ep_20260702_104500_ab12"
schema_version: "0.3"
session_id: "sess_20260702_102900_cd34"

agent: "codex"
repo: "/Users/me/projects/sample-repo"
branch: "main"

task:
  goal: "Add lodash dependency"
  user_intent: "给项目添加 lodash 依赖"
  constraints:
    - "不要污染 lockfile"

context:
  files_read:
    - "package.json"
  files_changed:
    - "package.json"
    - "pnpm-lock.yaml"
  commands_run:
    - "npm install lodash"
    - "rm package-lock.json"
    - "pnpm add lodash"
  tools_used:
    - "Bash"
    - "Edit"

failures:
  - symptom: "package-lock.json was created"
    failed_action: "npm install lodash"
    evidence:
      - "package-lock.json appeared after npm install"
    resolution: "Removed package-lock.json and used pnpm add"

verification:
  status: "passed"
  commands:
    - "pnpm install --frozen-lockfile"
  evidence:
    - "pnpm install completed successfully"

user_feedback:
  - "这个项目别用 npm，以后都用 pnpm。"

candidates:
  - "cand_20260702_104501_pnpm_fact"
  - "cand_20260702_104502_anti_npm"

skills_used: []

outcome:
  status: "success"
  summary: "Dependency was added using pnpm after correcting package manager mistake."

created_at: "2026-07-02T10:45:00.000Z"

graph:
  node_id: "graph_node_20260702_104500_episode"
```

---

# 13. Candidate Memory Schema

Candidate 可以存放在：

```text id="z6ybj2"
.aeb/candidates/inbox/
.aeb/candidates/quarantine/
.aeb/candidates/accepted/
.aeb/candidates/rejected/
.aeb/candidates/supersession/
.aeb/candidates/rollback/
```

示例：

```yaml id="okdoiw"
id: "cand_20260702_104502_anti_npm"
schema_version: "0.3"

type: "anti_pattern"
status: "inbox"

summary: "不要在该 repo 中使用 npm install。"

content:
  bad_action: "npm install"
  reason: "该 repo 使用 pnpm，npm install 会生成 package-lock.json 并污染 lockfile。"
  preferred_action: "使用 pnpm add / pnpm install"

source:
  kind: "user_correction"
  actor: "user"
  artifact: "evt_20260702_104400_user_correction"
  timestamp: "2026-07-02T10:44:00.000Z"

evidence:
  - type: "episode"
    ref: "ep_20260702_104500_ab12"
    confidence: "high"
  - type: "event"
    ref: "evt_20260702_103000_ab12"
    confidence: "high"

scope:
  level: "repo"
  repo: "/Users/me/projects/sample-repo"
  paths: []

trigger:
  events:
    - "pre_tool_use"
  command_patterns:
    - "npm install"
  path_patterns: []
  intent_keywords:
    - "add dependency"
    - "安装依赖"

action_effect:
  type: "warn"
  message: "该 repo 使用 pnpm，请改用 pnpm add / pnpm install。"
  suggested_commands:
    - "pnpm add <package>"

confidence: "high"
risk: "medium"

lifecycle:
  created_at: "2026-07-02T10:45:02.000Z"
  updated_at: "2026-07-02T10:45:02.000Z"
  last_verified_at: "2026-07-02T10:45:00.000Z"
  last_retrieved_at: null
  last_triggered_at: null
  half_life_days: 120
  stale: false
  aging_score: 1.0

version:
  current: 1
  supersedes: []
  superseded_by: null

usage:
  retrieved_count: 0
  triggered_count: 0
  acted_on_count: 0
  success_count: 0
  failure_count: 0

policy_decision:
  action: "send_to_inbox"
  reason:
    - "anti_pattern requires user confirmation"
    - "medium risk"

graph:
  node_id: "graph_node_20260702_104502_candidate"
```

---

# 14. Active Memory Schema

Active memory 与 candidate 使用同一核心结构，但 `status = active`，存放在：

```text id="jko2kk"
.aeb/memories/project.yaml
.aeb/memories/user.yaml
.aeb/memories/anti-patterns.yaml
.aeb/memories/prospective.yaml
.aeb/memories/incidents.yaml
```

`anti-patterns.yaml` 示例：

```yaml id="61ko6c"
schema_version: "0.3"

memories:
  - id: "mem_20260702_105000_anti_npm"
    type: "anti_pattern"
    status: "active"

    summary: "不要在该 repo 中使用 npm install。"

    content:
      bad_action: "npm install"
      reason: "该 repo 使用 pnpm，npm install 会生成 package-lock.json。"
      preferred_action: "使用 pnpm add / pnpm install"

    source:
      kind: "user_correction"
      actor: "user"
      artifact: "cand_20260702_104502_anti_npm"
      timestamp: "2026-07-02T10:44:00.000Z"

    evidence:
      - type: "episode"
        ref: "ep_20260702_104500_ab12"
        confidence: "high"

    scope:
      level: "repo"
      repo: "/Users/me/projects/sample-repo"
      paths: []

    trigger:
      events:
        - "pre_tool_use"
      command_patterns:
        - "npm install"
      path_patterns: []
      intent_keywords:
        - "add dependency"
        - "安装依赖"

    action_effect:
      type: "warn"
      message: "该 repo 使用 pnpm，请改用 pnpm。"
      suggested_commands:
        - "pnpm add <package>"

    confidence: "high"
    risk: "medium"

    lifecycle:
      created_at: "2026-07-02T10:50:00.000Z"
      updated_at: "2026-07-02T10:50:00.000Z"
      last_verified_at: "2026-07-02T10:45:00.000Z"
      last_retrieved_at: null
      last_triggered_at: null
      half_life_days: 120
      stale: false
      aging_score: 1.0

    version:
      current: 1
      supersedes: []
      superseded_by: null

    usage:
      retrieved_count: 0
      triggered_count: 0
      acted_on_count: 0
      success_count: 0
      failure_count: 0

    graph:
      node_id: "graph_node_20260702_105000_memory"
```

---

# 15. Skill Candidate Schema

文件：

```text id="p0b8nt"
.aeb/skills/candidates.yaml
```

示例：

```yaml id="h1esk3"
schema_version: "0.3"

skills:
  - id: "skill_cand_20260702_update_schema"
    name: "update-schema-safely"
    status: "candidate"

    summary: "修改 migration 或 schema 后，生成 schema 并运行 schema integration tests。"

    trigger:
      events:
        - "file_changed"
        - "pre_tool_use"
      path_patterns:
        - "migrations/**"
        - "schema/**"
      command_patterns: []
      intent_keywords:
        - "modify schema"
        - "修改 migration"

    steps:
      - type: "command"
        command: "pnpm db:generate"
        expected_result: "Generated schema files are updated"
        fallback: "Check package.json for the correct schema generation command"
      - type: "command"
        command: "pnpm test:integration -- schema"
        expected_result: "Schema integration tests pass"
        fallback: "Check package.json test scripts"

    scope:
      repo: "/Users/me/projects/sample-repo"
      paths:
        - "migrations/**"
        - "schema/**"

    evidence:
      episodes:
        - "ep_20260702_104500_schema_1"
        - "ep_20260702_110000_schema_2"
      memories:
        - "mem_20260702_105000_schema_prospective"
      commands:
        - "pnpm db:generate"
        - "pnpm test:integration -- schema"

    confidence: "medium"
    risk: "medium"

    version:
      number: 1
      previous: null
      rollback_to: null
      superseded_by: null

    usage:
      suggested_count: 0
      executed_count: 0
      success_count: 0
      failure_count: 0

    lifecycle:
      created_at: "2026-07-02T11:10:00.000Z"
      updated_at: "2026-07-02T11:10:00.000Z"
      last_verified_at: "2026-07-02T11:00:00.000Z"
      half_life_days: 30
      stale: false

    graph:
      node_id: "graph_node_20260702_skill_candidate_schema"
```

---

# 16. Active Skill Schema

文件：

```text id="1hrgm0"
.aeb/skills/active.yaml
```

结构与 skill candidate 一致，但：

```yaml id="gbihd2"
status: "active"
```

激活时必须写入：

```text id="ein21r"
graph edge: episode -> promoted_to_skill -> skill
audit action: skill_accepted
```

---

# 17. Skill Usage Schema

Skill usage 可以写入 audit，也可以附加在 skill usage 中。

建议 audit 事件：

```yaml id="e0iuhr"
id: "audit_20260702_113000_skill_usage"
timestamp: "2026-07-02T11:30:00.000Z"
action: "skill_used"
agent: "claude-code"
repo: "/Users/me/projects/sample-repo"
subject_id: "skill_20260702_update_schema"
session_id: "sess_20260702_112000_ab12"

metadata:
  trigger:
    path: "migrations/20260702_add_profile.sql"
  steps:
    - command: "pnpm db:generate"
      status: "success"
    - command: "pnpm test:integration -- schema"
      status: "failed"
  result: "failure"
```

Skill failure 后更新：

```yaml id="fqxkq3"
usage:
  suggested_count: 3
  executed_count: 2
  success_count: 1
  failure_count: 1
```

如果达到降权阈值：

```yaml id="s2f15o"
status: "degraded"
confidence: "low"
```

---

# 18. Dream Report Schema

文件：

```text id="suw4k0"
.aeb/dream/reports/dream-xxx.yaml
```

示例：

```yaml id="axzbye"
id: "dream_20260702_111500_ab12"
schema_version: "0.3"

session_id: "sess_20260702_102900_cd34"
episode_id: "ep_20260702_104500_ab12"
created_at: "2026-07-02T11:15:00.000Z"

inputs:
  episodes:
    - "ep_20260702_104500_ab12"
    - "ep_20260702_110000_cd34"
  candidates:
    - "cand_20260702_104502_anti_npm"
  memories:
    - "mem_20260702_105000_project_pnpm"
  skills:
    - "skill_20260702_update_schema"

outputs:
  created_candidates:
    - "cand_20260702_111501_prospective_migration"
  merged_candidates:
    - "cand_20260702_104502_anti_npm"
  skill_candidates:
    - "skill_cand_20260702_update_schema"
  stale_warnings:
    - "mem_20260601_jest_test_command"
  supersession_candidates:
    - "sup_20260702_jest_to_vitest"
  rollback_candidates: []
  graph_edges:
    - "graph_edge_20260702_ep_to_cand"
    - "graph_edge_20260702_ep_to_skill"

summary: "Dream consolidated npm correction, detected stale Jest memory, and generated one schema update skill candidate."

graph:
  node_id: "graph_node_20260702_dream_report"
```

---

# 19. Supersession Candidate Schema

文件：

```text id="72a1wq"
.aeb/candidates/supersession/sup-xxx.yaml
```

示例：

```yaml id="6vdr9b"
id: "sup_20260702_jest_to_vitest"
schema_version: "0.3"

type: "supersession_candidate"
status: "inbox"

topic: "test_framework"

old_memory:
  id: "mem_20260601_project_jest"
  summary: "该项目使用 Jest。"

new_candidate:
  id: "cand_20260702_project_vitest"
  summary: "该项目使用 Vitest。"

scope:
  level: "repo"
  repo: "/Users/me/projects/sample-repo"

reason:
  - "Same repo and same topic: test_framework"
  - "New evidence found in package.json"
  - "vitest.config.ts exists"
  - "jest.config.js no longer exists"

evidence:
  - type: "file"
    ref: "package.json"
    confidence: "high"
  - type: "file"
    ref: "vitest.config.ts"
    confidence: "high"

decision:
  action: "pending"
  applied_at: null
  applied_by: null

graph:
  node_id: "graph_node_20260702_supersession"
```

应用后：

```yaml id="m9gb3k"
decision:
  action: "applied"
  applied_at: "2026-07-02T11:30:00.000Z"
  applied_by: "user"
```

旧 memory 更新：

```yaml id="d2zxkr"
status: "superseded"
version:
  superseded_by: "mem_20260702_project_vitest"
```

新 memory 更新：

```yaml id="d0n0xi"
version:
  supersedes:
    - "mem_20260601_project_jest"
```

---

# 20. Rollback Candidate Schema

文件：

```text id="a8locd"
.aeb/candidates/rollback/rollback-xxx.yaml
```

示例：

```yaml id="j6alza"
id: "rollback_20260702_schema_skill"
schema_version: "0.3"

type: "rollback_candidate"
status: "inbox"

skill:
  id: "skill_20260702_update_schema"
  name: "update-schema-safely"
  current_version: 2
  rollback_to_version: 1

reason:
  - "Skill command failed twice consecutively"
  - "Command pnpm test:integration -- schema no longer exists"

evidence:
  - type: "episode"
    ref: "ep_20260702_schema_failure_1"
    confidence: "high"
  - type: "episode"
    ref: "ep_20260702_schema_failure_2"
    confidence: "high"

suggested_action: "rollback"

graph:
  node_id: "graph_node_20260702_rollback_candidate"
```

---

# 21. Graph Node Schema

文件：

```text id="lcsf3f"
.aeb/graph/nodes.jsonl
```

每行一个 GraphNode。

```yaml id="8rn6z0"
id: "graph_node_20260702_105000_memory"
schema_version: "0.3"
type: "memory"
ref: "mem_20260702_105000_anti_npm"
summary: "不要在该 repo 中使用 npm install。"
created_at: "2026-07-02T10:50:00.000Z"
metadata:
  memory_type: "anti_pattern"
  status: "active"
  repo: "/Users/me/projects/sample-repo"
```

节点类型：

```text id="v5qjsc"
event
episode
candidate
memory
skill
skill_version
incident
checkpoint
dream_report
supersession_candidate
rollback_candidate
```

---

# 22. Graph Edge Schema

文件：

```text id="t2fz11"
.aeb/graph/edges.jsonl
```

每行一个 GraphEdge。

```yaml id="v70xac"
id: "graph_edge_20260702_cand_to_mem"
schema_version: "0.3"
type: "derived_from"
from: "graph_node_20260702_105000_memory"
to: "graph_node_20260702_104502_candidate"
created_at: "2026-07-02T10:50:01.000Z"
metadata:
  reason: "memory accepted from candidate"
```

边类型：

```text id="enk0yw"
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

方向约定：

```text id="f5b9xq"
memory -> derived_from -> candidate
candidate -> derived_from -> episode
episode -> evidence_for -> memory
memory -> supersedes -> old_memory
episode -> promoted_to_skill -> skill
skill -> failed_by -> episode
skill_version_new -> rolled_back_to -> skill_version_old
memory -> aged_from -> dream_report
```

---

# 23. Audit Schema

文件：

```text id="jg9m74"
.aeb/audit/audit.jsonl
```

示例：

```yaml id="zmtwqj"
id: "audit_20260702_105000_accept_memory"
schema_version: "0.3"
timestamp: "2026-07-02T10:50:00.000Z"

agent: "codex"
repo: "/Users/me/projects/sample-repo"
session_id: "sess_20260702_102900_cd34"

action: "memory_accepted"
subject_id: "mem_20260702_105000_anti_npm"

reason:
  - "accepted from inbox"
  - "source=user_correction"
  - "scope=repo"

metadata:
  candidate_id: "cand_20260702_104502_anti_npm"
  memory_type: "anti_pattern"
```

`action` 可选值：

```text id="kc18wv"
event_recorded
session_started
session_ended
recall_performed
trigger_matched
trigger_warned
trigger_rewritten
trigger_blocked
candidate_created
candidate_auto_accepted
candidate_sent_to_inbox
candidate_quarantined
candidate_rejected
memory_accepted
memory_rejected
memory_retrieved
memory_triggered
memory_staled
memory_reverified
memory_superseded
episode_written
dream_started
dream_completed
skill_candidate_created
skill_accepted
skill_suggested
skill_used
skill_failed
skill_degraded
skill_disabled
skill_rolled_back
graph_node_created
graph_edge_created
```

---

# 24. Index Schema

索引用于快速 recall / trigger。MVP 可在每次 memory 更新后重建。

## 24.1 Command Index

文件：

```text id="g5j0wm"
.aeb/indexes/command.json
```

示例：

```json id="ot3y18"
{
  "schema_version": "0.3",
  "patterns": [
    {
      "pattern": "npm install",
      "memory_ids": ["mem_20260702_105000_anti_npm"],
      "skill_ids": []
    },
    {
      "pattern": "pnpm db:generate",
      "memory_ids": [],
      "skill_ids": ["skill_20260702_update_schema"]
    }
  ]
}
```

## 24.2 Path Index

文件：

```text id="g4bp51"
.aeb/indexes/path.json
```

示例：

```json id="6uq71c"
{
  "schema_version": "0.3",
  "patterns": [
    {
      "pattern": "migrations/**",
      "memory_ids": ["mem_20260702_migration_prospective"],
      "skill_ids": ["skill_20260702_update_schema"]
    },
    {
      "pattern": "generated/**",
      "memory_ids": ["mem_20260702_no_generated_edit"],
      "skill_ids": []
    }
  ]
}
```

## 24.3 Topic Index

文件：

```text id="nbdf29"
.aeb/indexes/topic.json
```

示例：

```json id="u4rlk1"
{
  "schema_version": "0.3",
  "topics": {
    "package_manager": [
      "mem_20260702_project_pnpm"
    ],
    "test_framework": [
      "mem_20260601_project_jest",
      "mem_20260702_project_vitest"
    ]
  }
}
```

---

# 25. Validation Rules

实现时必须提供 schema validation。

## 25.1 Memory validation

Memory 必须满足：

```text id="95ydal"
id 必填
type 必填
summary 必填
source.kind 必填
scope.level 必填
confidence 必填
risk 必填
status 必填
lifecycle.created_at 必填
lifecycle.half_life_days 必填
usage 必填
graph.node_id 必填
```

active memory 额外要求：

```text id="7qw6oh"
evidence 至少 1 条
scope.repo 在 repo scope 下必填
trigger 或 action_effect 至少一个存在
```

## 25.2 Skill validation

Skill 必须满足：

```text id="pbqbcf"
id 必填
name 必填
trigger 必填
steps 至少 1 条
scope 必填
evidence.episodes 至少 1 条
status 必填
version.number 必填
usage 必填
lifecycle 必填
graph.node_id 必填
```

active skill 额外要求：

```text id="03gzki"
evidence.episodes 至少 2 条，除非用户手动强制接受
steps 中 command 不得为空
```

## 25.3 Supersession validation

Supersession candidate 必须满足：

```text id="sm0bje"
old_memory.id 必填
new_candidate.id 必填
topic 必填
scope.repo 必填
reason 至少 1 条
evidence 至少 1 条
```

---

# 26. Migration Rules

所有文件带 `schema_version`。

如果未来升级：

```text id="d8ig28"
0.3 → 0.4
```

必须提供：

```bash id="1gqwwp"
aeb migrate
```

MVP 只需实现：

```text id="eftd95"
检测 schema_version
如果不匹配，提示用户当前版本不支持
```

不要求 MVP 做复杂自动迁移。

---

# 27. 最小示例：pnpm-vs-npm 完整链路

## 27.1 用户纠正事件

```yaml id="6b994u"
normalized_type: "user_prompt_submit"
user_prompt: "这个项目别用 npm，以后都用 pnpm。"
```

生成 candidate：

```yaml id="a6jzxf"
type: "anti_pattern"
summary: "不要在该 repo 中使用 npm install。"
status: "inbox"
```

用户接受后生成 memory：

```yaml id="dth3xn"
type: "anti_pattern"
status: "active"
trigger:
  command_patterns:
    - "npm install"
action_effect:
  type: "warn"
  message: "该 repo 使用 pnpm，请改用 pnpm。"
```

写 graph：

```text id="j0qg3n"
event → episode
episode → candidate
candidate → memory
```

第二次命令触发：

```yaml id="3qkjv8"
command: "npm install dayjs"
```

返回 trigger result：

```yaml id="ycj8ow"
decision: "warn"
message: "该 repo 使用 pnpm，请改用 pnpm add dayjs。"
suggested_commands:
  - "pnpm add dayjs"
```

usage 更新：

```yaml id="vi064d"
triggered_count: 1
acted_on_count: 1
success_count: 1
```

---

# 28. 最小示例：Jest → Vitest Supersession

旧 memory：

```yaml id="dzj965"
id: "mem_old_jest"
type: "project_fact"
summary: "该项目使用 Jest。"
status: "active"
content:
  topic: "test_framework"
  value: "jest"
```

新 candidate：

```yaml id="h2fl46"
id: "cand_new_vitest"
type: "project_fact"
summary: "该项目使用 Vitest。"
content:
  topic: "test_framework"
  value: "vitest"
```

supersession candidate：

```yaml id="yg9uv1"
topic: "test_framework"
old_memory:
  id: "mem_old_jest"
new_candidate:
  id: "cand_new_vitest"
reason:
  - "Same topic and same repo"
  - "New package.json evidence"
```

应用后：

```yaml id="qhsuwx"
mem_old_jest.status: "superseded"
mem_new_vitest.status: "active"
```

graph：

```text id="d3vuez"
mem_new_vitest -> supersedes -> mem_old_jest
```

---

# 29. 最小示例：Skill Promotion + Rollback

两次 episode 都包含：

```text id="59ajg0"
migrations/** changed
pnpm db:generate success
pnpm test:integration -- schema success
```

Dream 生成 skill candidate：

```yaml id="l47o3c"
name: "update-schema-safely"
status: "candidate"
evidence:
  episodes:
    - "ep_001"
    - "ep_002"
```

用户接受后：

```yaml id="jwlzxx"
status: "active"
```

后续执行失败两次：

```yaml id="8ct8tq"
usage:
  failure_count: 2
status: "degraded"
confidence: "low"
```

rollback candidate：

```yaml id="edl4vb"
type: "rollback_candidate"
skill:
  id: "skill_update_schema"
reason:
  - "Skill failed twice consecutively"
```

用户 rollback：

```text id="cp4o3z"
current version → rolled_back
previous version → active
graph edge: current -> rolled_back_to -> previous
```

---

# 30. 结论

这份 Schema Specification 固定了 AEB v0.3 增强型 MVP 的本地数据契约。

后续实现时必须保证：

```text id="vc8uuc"
1. 所有对象可本地持久化。
2. 所有状态变化可审计。
3. 所有重要 memory 可 graph trace。
4. active memory 可 recall / trigger。
5. stale memory 降权不删除。
6. superseded memory 默认不召回但可追溯。
7. skill 可从 episode 晋升，也可因失败降权或回滚。
8. Dream report 是 episode 到 memory / skill / graph 的巩固记录。
```

最重要的是：

```text id="k9vbb6"
AEB 的 Schema 不是为了记录更多信息，
而是为了让 Agent 的经验能够被验证、触发、老化、覆盖、回滚和追溯。
```
