# Agent External Brain 技术架构文档

版本：v0.3
项目名称：agent-external-brain
命令简称：aeb
目标形态：面向 Codex / Claude Code 的 Agent-native 本地记忆运行时
核心原则：Hook-first、Skill-first、Auto-trigger-first
MVP 范围：增强型 MVP，包含自动召回、自动触发、自动 episode、Dream consolidation、Skill promotion / rollback、Memory aging、Supersession、Memory graph、Benchmark。

---

# 1. 架构目标

Agent External Brain，简称 AEB，是一个嵌入 Coding Agent 生命周期的本地记忆运行时。

它不是一个让用户手动维护的记事本，也不是单纯向量库，而是一套帮助 Coding Agent 从真实任务中持续形成工程经验的运行时系统。

AEB 需要完成以下闭环：

```text
Agent lifecycle event
→ normalize event
→ record evidence
→ recall / trigger / capture
→ write episode
→ run dream consolidation
→ create / update memory
→ promote / degrade skill
→ age / supersede memory
→ update memory graph
→ affect future Agent behavior
```

MVP 的核心目标是证明：

```text
Agent 可以在不增加用户手动操作成本的情况下，
从任务经历中形成可验证、可触发、可老化、可回滚、可追溯的工程经验。
```

---

# 2. 总体架构

AEB 分为 10 层：

```text
1. Agent Integration Layer
2. Lifecycle Adapter Layer
3. Runtime Orchestrator
4. Event / Session Layer
5. Recall / Trigger Layer
6. Episode / Dream Layer
7. Memory / Candidate Layer
8. Skill Layer
9. Lifecycle Governance Layer
10. Storage / Graph / Audit Layer
```

整体数据流：

```text
Codex / Claude Code Hook
        ↓
Agent Adapter
        ↓
AebLifecycleEvent
        ↓
Runtime Orchestrator
        ↓
Recall / Trigger / Capture / Episode / Dream
        ↓
Memory Store + Skill Store + Graph Store
        ↓
Hook response / Agent context / Warning / Skill suggestion
```

MVP 技术路线：

```text
Codex / Claude Code hooks
→ call aeb CLI
→ AEB Runtime Core
→ local file storage
→ structured hook response
```

MCP 不作为 MVP 必需项。后续可以在 Runtime Core 上增加 MCP Server，但 MVP 优先保证 CLI + hook 路径稳定。

---

# 3. 项目目录结构

推荐 TypeScript / Node.js 实现。

```text
agent-external-brain/
  package.json
  pnpm-lock.yaml
  tsconfig.json
  vitest.config.ts

  src/
    cli/
      index.ts
      commands/
        init.ts
        hook.ts
        inbox.ts
        quarantine.ts
        memory.ts
        skill.ts
        dream.ts
        graph.ts
        bench.ts
        audit.ts
        status.ts

    integrations/
      codex/
        codex-adapter.ts
        codex-hook-renderer.ts
        codex-instructions.ts
        codex-skills.ts
      claude-code/
        claude-adapter.ts
        claude-hook-renderer.ts
        claude-instructions.ts
        claude-skills.ts
      generic/
        generic-adapter.ts

    lifecycle/
      event-schema.ts
      event-normalizer.ts
      session-manager.ts
      lifecycle-router.ts

    runtime/
      runtime.ts
      runtime-result.ts
      runtime-context.ts

    storage/
      file-store.ts
      workspace.ts
      lock.ts
      jsonl.ts
      yaml.ts

    memory/
      memory-schema.ts
      memory-store.ts
      candidate-store.ts
      memory-index.ts
      memory-lifecycle.ts

    recall/
      recall-engine.ts
      retrieval-pack.ts
      scoring.ts

    trigger/
      trigger-engine.ts
      command-pattern.ts
      path-pattern.ts
      trigger-result.ts

    episode/
      episode-writer.ts
      checkpoint-writer.ts
      episode-extractor.ts

    dream/
      dream-runner.ts
      dream-report.ts
      candidate-merger.ts
      workflow-miner.ts
      stale-detector.ts
      conflict-detector.ts

    candidate/
      candidate-extractor.ts
      correction-parser.ts
      observation-extractor.ts

    policy/
      policy-engine.ts
      risk-classifier.ts
      source-classifier.ts
      aging-policy.ts
      skill-policy.ts
      dream-policy.ts

    skill/
      skill-schema.ts
      skill-store.ts
      skill-promoter.ts
      skill-trigger.ts
      skill-usage.ts
      skill-rollback.ts

    supersession/
      topic-detector.ts
      conflict-detector.ts
      supersession-engine.ts

    graph/
      graph-schema.ts
      graph-store.ts
      graph-writer.ts
      graph-query.ts

    audit/
      audit-log.ts

    bench/
      lifecycle-simulator.ts
      benchmark-runner.ts
      assertions.ts
      scorecard.ts

    utils/
      ids.ts
      time.ts
      repo.ts
      git.ts
      paths.ts

  tests/
    unit/
    integration/
    fixtures/

  benchmarks/
    pnpm-vs-npm/
    interrupted-task/
    migration-trigger/
    dream-skill-promotion/
    skill-rollback/
    memory-aging/
    supersession/
    memory-graph/
```

---

# 4. 本地工作区结构

用户在 repo 根目录运行：

```bash
aeb init --agent codex
```

或：

```bash
aeb init --agent claude-code
```

生成：

```text
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

  generated/
    codex/
    claude-code/
```

Agent 侧生成：

## Codex

```text
.codex/
  hooks.json

.agents/
  skills/
    aeb-memory-recall/
      SKILL.md
    aeb-trigger-check/
      SKILL.md
    aeb-capture-correction/
      SKILL.md
    aeb-checkpoint/
      SKILL.md
    aeb-task-review/
      SKILL.md
    aeb-skill-use/
      SKILL.md

AGENTS.md
```

## Claude Code

```text
.claude/
  settings.json
  hooks/
    aeb-hook.js
  skills/
    aeb-memory-recall/
      SKILL.md
    aeb-trigger-check/
      SKILL.md
    aeb-capture-correction/
      SKILL.md
    aeb-checkpoint/
      SKILL.md
    aeb-task-review/
      SKILL.md
    aeb-skill-use/
      SKILL.md
  rules/
    aeb-memory.md

CLAUDE.md
```

---

# 5. 核心运行时入口

MVP 中，所有 hook 都调用 CLI。

```bash
aeb hook codex session-start
aeb hook codex user-prompt-submit
aeb hook codex pre-tool-use
aeb hook codex post-tool-use
aeb hook codex post-compact
aeb hook codex stop

aeb hook claude session-start
aeb hook claude user-prompt-submit
aeb hook claude pre-tool-use
aeb hook claude post-tool-use
aeb hook claude post-compact
aeb hook claude stop
```

Hook 输入从 stdin 读取。

内部统一走：

```ts
class AebRuntime {
  async handle(event: AebLifecycleEvent): Promise<RuntimeResult> {
    // 1. record event
    // 2. route by event type
    // 3. run recall / trigger / capture / episode / dream
    // 4. write audit / graph
    // 5. return agent-specific response
  }
}
```

---

# 6. 统一生命周期事件模型

Codex 和 Claude Code 的 hook 输入不同，AEB 内部必须统一。

```ts
type AgentName = "codex" | "claude-code" | "generic";

type AebLifecycleEventType =
  | "session_start"
  | "user_prompt_submit"
  | "pre_tool_use"
  | "post_tool_use"
  | "post_tool_use_failure"
  | "file_changed"
  | "test_after"
  | "pre_compact"
  | "post_compact"
  | "stop"
  | "session_end";

interface AebLifecycleEvent {
  id: string;
  sourceAgent: AgentName;
  sourceEventName: string;
  normalizedType: AebLifecycleEventType;

  sessionId?: string;
  turnId?: string;

  cwd: string;
  repoRoot?: string;
  branch?: string;

  timestamp: string;
  model?: string;
  permissionMode?: string;

  userPrompt?: string;

  tool?: {
    name: string;
    input: unknown;
    output?: unknown;
    exitCode?: number;
    stdout?: string;
    stderr?: string;
  };

  command?: {
    raw: string;
    normalized?: string;
    exitCode?: number;
    stdout?: string;
    stderr?: string;
  };

  files?: Array<{
    path: string;
    operation?: "read" | "write" | "edit" | "delete";
  }>;

  raw: unknown;
}
```

---

# 7. Runtime Result

所有 runtime 处理返回统一结构，再由 Codex / Claude adapter 转换为各自 hook 输出。

```ts
interface RuntimeResult {
  action:
    | "no_op"
    | "record"
    | "add_context"
    | "warn"
    | "suggest_skill"
    | "rewrite"
    | "block"
    | "create_candidate"
    | "write_episode"
    | "run_dream"
    | "checkpoint";

  severity?: "info" | "low" | "medium" | "high" | "critical";

  message?: string;
  additionalContext?: string;

  suggestedCommands?: string[];
  suggestedSkillIds?: string[];

  updatedInput?: unknown;
  blockReason?: string;

  created?: {
    eventIds?: string[];
    candidateIds?: string[];
    memoryIds?: string[];
    episodeId?: string;
    checkpointId?: string;
    dreamReportId?: string;
    graphNodeIds?: string[];
    graphEdgeIds?: string[];
    auditIds?: string[];
  };

  agentVisibleSummary?: string;
  raw?: unknown;
}
```

---

# 8. Codex 集成设计

## 8.1 Codex 使用机制

MVP 优先使用：

```text
1. hooks
2. AGENTS.md
3. skills
4. CLI Runtime
```

不依赖 MCP。

## 8.2 Codex hook 映射

| Codex lifecycle                    | AEB event          | AEB action                                    |
| ---------------------------------- | ------------------ | --------------------------------------------- |
| SessionStart                       | session_start      | 创建 session、aging check、recall、注入 context      |
| UserPromptSubmit                   | user_prompt_submit | 记录用户消息、识别纠正、生成候选                              |
| PreToolUse Bash                    | pre_tool_use       | command trigger、anti-pattern、skill suggestion |
| PreToolUse Edit/Write/apply_patch  | pre_tool_use       | path trigger、generated/migration/security 规则  |
| PostToolUse Bash                   | post_tool_use      | 记录命令结果、测试结果、skill usage                       |
| PostToolUse Edit/Write/apply_patch | post_tool_use      | 记录文件修改                                        |
| PostCompact                        | post_compact       | checkpoint + critical recall                  |
| Stop                               | stop               | 写 episode、运行 dream、写 graph、输出摘要               |

## 8.3 Codex hook 配置示例

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex session-start",
            "timeout": 10,
            "statusMessage": "Loading AEB memories"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex user-prompt-submit",
            "timeout": 10,
            "statusMessage": "Capturing memory-worthy user input"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write|apply_patch",
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex pre-tool-use",
            "timeout": 10,
            "statusMessage": "Checking AEB memory"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash|Edit|Write|apply_patch",
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex post-tool-use",
            "timeout": 10,
            "statusMessage": "Recording AEB evidence"
          }
        ]
      }
    ],
    "PostCompact": [
      {
        "matcher": "manual|auto",
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex post-compact",
            "timeout": 10,
            "statusMessage": "Restoring critical AEB memory"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "aeb hook codex stop",
            "timeout": 30,
            "statusMessage": "Writing AEB episode and dream report"
          }
        ]
      }
    ]
  }
}
```

## 8.4 Codex AGENTS.md 片段

AGENTS.md 只写稳定行为，不写动态记忆。

```markdown
## Agent External Brain

This repository uses Agent External Brain (AEB), a local memory runtime for coding agents.

Rules:
- At task start, use AEB memory context injected by hooks.
- Before running commands or editing files, respect AEB trigger warnings.
- If AEB suggests an active skill, consider using it when its scope matches the current task.
- If the user corrects you, allow AEB to capture it as a memory candidate.
- At task end, allow AEB to write an episode and run lightweight dream consolidation.
- Do not ask the user to manually run AEB commands unless debugging or processing inbox/quarantine.
- AEB memories have source, scope, confidence, lifecycle, and risk. Do not generalize repo-scoped memories to other repositories.
- Superseded and quarantined memories must not be used.
- Stale memories require verification before action.
```

---

# 9. Claude Code 集成设计

## 9.1 Claude Code 使用机制

MVP 优先使用：

```text
1. hooks
2. CLAUDE.md
3. .claude/rules/
4. skills
5. CLI Runtime
```

不依赖 MCP。

## 9.2 Claude hook 映射

| Claude Code lifecycle    | AEB event          | AEB action                                    |
| ------------------------ | ------------------ | --------------------------------------------- |
| SessionStart             | session_start      | 创建 session、aging check、recall                 |
| UserPromptSubmit         | user_prompt_submit | 捕获纠正、偏好、未来规则                                  |
| PreToolUse Bash          | pre_tool_use       | command trigger、anti-pattern、skill suggestion |
| PreToolUse Edit/Write    | pre_tool_use       | path trigger、generated/migration/security 规则  |
| PostToolUse Bash         | post_tool_use      | 记录命令结果、测试结果、skill usage                       |
| PostToolUse Edit/Write   | post_tool_use      | 记录文件修改                                        |
| PreCompact / PostCompact | post_compact       | checkpoint + critical recall                  |
| Stop                     | stop               | episode writer + dream                        |
| SessionEnd               | session_end        | session archive                               |

## 9.3 Claude settings 示例

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js session-start"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js user-prompt-submit"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash|Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js pre-tool-use"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash|Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js post-tool-use"
          }
        ]
      }
    ],
    "PostCompact": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js post-compact"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/aeb-hook.js stop"
          }
        ]
      }
    ]
  }
}
```

## 9.4 Claude CLAUDE.md 片段

```markdown
## Agent External Brain

This repository uses Agent External Brain (AEB).

Behavior:
- Use AEB memory context injected at session start.
- Respect AEB trigger warnings before Bash/Edit/Write actions.
- Use AEB active skills only when their scope and trigger match the current task.
- When the user corrects you, allow AEB to create a memory candidate.
- During long tasks or before compaction, create an AEB checkpoint.
- Before ending a task, allow AEB to write an episode and run lightweight dream consolidation.
- Do not ask the user to manually run AEB commands unless debugging, auditing, or processing inbox/quarantine.

Memory interpretation:
- AEB memories include source, scope, confidence, risk, lifecycle, and graph provenance.
- Repo-scoped memories apply only to this repository.
- Stale memories require verification before use.
- Superseded and quarantined memories must not be used.
- Degraded or disabled skills must not be recommended as default actions.
```

---

# 10. 核心状态机

## 10.1 Memory 状态机

```text
candidate
  ├─ policy: auto_accept → active
  ├─ policy: inbox → inbox
  ├─ policy: quarantine → quarantined
  └─ policy: reject → rejected

inbox
  ├─ user accept → active
  ├─ user edit → candidate
  ├─ user reject → rejected
  └─ user quarantine → quarantined

active
  ├─ aging → stale active
  ├─ conflict → supersession_candidate
  ├─ superseded → superseded
  ├─ user archive → archived
  └─ high-risk failure → quarantined

superseded
  └─ historical only, not default recall

quarantined
  ├─ approve → active
  └─ reject → rejected
```

MVP 要求：

1. `active` 才参与 recall / trigger。
2. `stale active` 可以 recall，但必须附带 warning。
3. `superseded` 不参与默认 recall。
4. `quarantined` 不参与 recall / trigger。
5. 所有状态变化写 audit 和 graph edge。

---

## 10.2 Skill 状态机

```text
skill_candidate
  ├─ user accept → active
  ├─ user reject → rejected
  └─ policy quarantine → quarantined

active
  ├─ usage success → refresh last_verified_at
  ├─ usage failure x1 → lower confidence
  ├─ usage failure threshold → degraded
  ├─ severe failure → disabled
  └─ superseded by new version → superseded

degraded
  ├─ success → active
  ├─ failure → disabled
  └─ user rollback → rolled_back

disabled
  ├─ user rollback → rolled_back
  ├─ user re-enable → active
  └─ new version → superseded

rolled_back
  └─ historical, previous version active if exists
```

MVP 要求：

1. `active` skill 可被 recall / trigger 推荐。
2. `degraded` skill 低优先级显示，默认不推荐。
3. `disabled` skill 不推荐。
4. skill 失败必须记录 failure_count。
5. rollback 必须写 audit 和 graph edge。

---

## 10.3 Supersession 状态机

```text
conflict detected
  ↓
supersession_candidate
  ├─ apply → old memory superseded + new memory active
  ├─ reject → both remain unchanged
  └─ edit → regenerate candidate
```

MVP 要求：

1. supersession candidate 默认进入 inbox。
2. 只有强证据低风险 fact 可以 auto apply。
3. 应用后旧 memory 不删除。
4. recall 默认隐藏 superseded memory。
5. graph 写 `new -> supersedes -> old`。

---

## 10.4 Aging 状态机

```text
active memory
  ↓
aging check
  ├─ fresh → no change
  ├─ stale → stale=true + recall downweight
  ├─ reverified success → stale=false + refresh last_verified_at
  └─ verification failed → supersession / invalidation candidate
```

MVP 要求：

1. aging 不删除 memory。
2. aging 只影响 recall ranking 和 warning。
3. stale memory 命中时提醒 Agent 先验证。
4. 成功验证后刷新 `last_verified_at`。

---

# 11. 核心 Schema

## 11.1 MemoryObject

```ts
type MemoryType =
  | "project_fact"
  | "user_preference"
  | "anti_pattern"
  | "prospective"
  | "incident";

type MemoryStatus =
  | "candidate"
  | "inbox"
  | "active"
  | "rejected"
  | "quarantined"
  | "archived"
  | "superseded";

interface MemoryObject {
  id: string;
  type: MemoryType;

  summary: string;
  content: Record<string, unknown>;

  source: {
    kind:
      | "user_correction"
      | "user_message"
      | "tool_observation"
      | "command_result"
      | "test_result"
      | "file_observation"
      | "model_inference"
      | "manual";
    actor?: string;
    artifact?: string;
    timestamp: string;
  };

  evidence: Array<{
    type: "episode" | "event" | "file" | "command" | "test" | "user_message";
    ref: string;
    confidence: "low" | "medium" | "high";
  }>;

  scope: {
    level: "repo" | "path" | "user" | "team" | "global";
    repo?: string;
    paths?: string[];
    agent?: "codex" | "claude-code" | "generic";
  };

  trigger?: {
    events?: string[];
    commandPatterns?: string[];
    pathPatterns?: string[];
    intentKeywords?: string[];
  };

  actionEffect?: {
    type: "context" | "warn" | "suggest_command" | "rewrite" | "block" | "ask";
    message?: string;
    suggestedCommands?: string[];
    rewriteRule?: {
      from: string;
      to: string;
    };
  };

  confidence: "low" | "medium" | "high";
  risk: "low" | "medium" | "high" | "critical";
  status: MemoryStatus;

  lifecycle: {
    createdAt: string;
    updatedAt: string;
    lastVerifiedAt?: string;
    lastRetrievedAt?: string;
    lastTriggeredAt?: string;
    halfLifeDays: number;
    stale: boolean;
    agingScore: number;
  };

  version: {
    current: number;
    supersedes?: string[];
    supersededBy?: string;
  };

  usage: {
    retrievedCount: number;
    triggeredCount: number;
    actedOnCount: number;
    successCount: number;
    failureCount: number;
  };

  graph: {
    nodeId: string;
  };
}
```

---

## 11.2 SkillObject

```ts
type SkillStatus =
  | "candidate"
  | "active"
  | "degraded"
  | "disabled"
  | "rolled_back"
  | "rejected"
  | "superseded";

interface SkillObject {
  id: string;
  name: string;
  summary: string;

  trigger: {
    events?: string[];
    pathPatterns?: string[];
    commandPatterns?: string[];
    intentKeywords?: string[];
  };

  steps: Array<{
    type: "command" | "check" | "read" | "manual";
    command?: string;
    description?: string;
    expectedResult?: string;
    fallback?: string;
  }>;

  scope: {
    repo?: string;
    paths?: string[];
  };

  evidence: {
    episodes: string[];
    memories: string[];
    commands?: string[];
  };

  confidence: "low" | "medium" | "high";
  risk: "low" | "medium" | "high" | "critical";
  status: SkillStatus;

  version: {
    number: number;
    previous?: string;
    rollbackTo?: string;
    supersededBy?: string;
  };

  usage: {
    suggestedCount: number;
    executedCount: number;
    successCount: number;
    failureCount: number;
  };

  lifecycle: {
    createdAt: string;
    updatedAt: string;
    lastVerifiedAt?: string;
    halfLifeDays: number;
    stale: boolean;
  };

  graph: {
    nodeId: string;
  };
}
```

---

## 11.3 Episode

```ts
interface Episode {
  id: string;
  sessionId: string;
  agent: "codex" | "claude-code" | "generic";
  repo: string;
  branch?: string;

  task: {
    goal?: string;
    userIntent?: string;
    constraints: string[];
  };

  context: {
    filesRead: string[];
    filesChanged: string[];
    commandsRun: string[];
    toolsUsed: string[];
  };

  failures: Array<{
    symptom: string;
    failedAction?: string;
    evidence: string[];
    resolution?: string;
  }>;

  verification: {
    status: "passed" | "failed" | "not_run" | "unknown";
    commands: string[];
    evidence: string[];
  };

  userFeedback: string[];

  candidates: string[];
  skillsUsed: string[];

  outcome: {
    status: "success" | "partial" | "failed" | "unknown";
    summary: string;
  };

  graph: {
    nodeId: string;
  };

  createdAt: string;
}
```

---

## 11.4 DreamReport

```ts
interface DreamReport {
  id: string;
  sessionId: string;
  episodeId: string;
  createdAt: string;

  inputs: {
    episodes: string[];
    candidates: string[];
    memories: string[];
    skills: string[];
  };

  outputs: {
    createdCandidates: string[];
    mergedCandidates: string[];
    skillCandidates: string[];
    staleWarnings: string[];
    supersessionCandidates: string[];
    rollbackCandidates: string[];
    graphEdges: string[];
  };

  summary: string;
}
```

---

## 11.5 Graph

```ts
type GraphNodeType =
  | "event"
  | "episode"
  | "candidate"
  | "memory"
  | "skill"
  | "skill_version"
  | "incident"
  | "checkpoint"
  | "dream_report";

interface GraphNode {
  id: string;
  type: GraphNodeType;
  ref: string;
  summary: string;
  createdAt: string;
  metadata?: Record<string, unknown>;
}

type GraphEdgeType =
  | "derived_from"
  | "evidence_for"
  | "triggered_by"
  | "used_in"
  | "verified_by"
  | "failed_by"
  | "promoted_to_skill"
  | "supersedes"
  | "conflicts_with"
  | "rolled_back_to"
  | "aged_from";

interface GraphEdge {
  id: string;
  type: GraphEdgeType;
  from: string;
  to: string;
  createdAt: string;
  metadata?: Record<string, unknown>;
}
```

---

# 12. Recall Engine

## 12.1 输入

```ts
interface RecallQuery {
  repo: string;
  cwd: string;
  branch?: string;
  intent?: string;
  paths?: string[];
  command?: string;
  agent: "codex" | "claude-code" | "generic";
  mode: "task_start" | "resume" | "pre_tool" | "post_compact" | "manual";
}
```

## 12.2 输出

```ts
interface RetrievalPack {
  projectFacts: MemoryObject[];
  userPreferences: MemoryObject[];
  antiPatterns: MemoryObject[];
  prospective: MemoryObject[];
  staleWarnings: MemoryObject[];
  skillSuggestions: SkillObject[];
  checkpoints: Checkpoint[];

  agentContext: string;
}
```

## 12.3 评分规则

MVP 不使用向量库，先用规则评分。

```text
score =
  repo_match * 40
+ path_match * 25
+ command_match * 25
+ intent_keyword_match * 15
+ type_priority * 10
+ confidence_bonus
+ skill_active_bonus
- stale_penalty
- degraded_skill_penalty
- risk_penalty
```

过滤规则：

```text
- rejected 不召回
- quarantined 不召回
- superseded 默认不召回
- disabled skill 不召回
- degraded skill 只作为低优先级 warning
- stale memory 可召回，但必须附带 verification warning
```

---

# 13. Trigger Engine

## 13.1 输入

```ts
interface TriggerInput {
  eventType: "pre_tool_use" | "file_changed" | "user_prompt_submit" | "test_after";
  repo: string;
  command?: string;
  paths?: string[];
  prompt?: string;
  toolName?: string;
  agent: "codex" | "claude-code" | "generic";
}
```

## 13.2 输出

```ts
interface TriggerResult {
  matchedMemories: Array<{
    memoryId: string;
    summary: string;
    severity: "info" | "low" | "medium" | "high" | "critical";
    actionEffect?: MemoryObject["actionEffect"];
  }>;

  matchedSkills: Array<{
    skillId: string;
    name: string;
    confidence: "low" | "medium" | "high";
    status: SkillStatus;
    steps: SkillObject["steps"];
  }>;

  decision: "none" | "context" | "warn" | "suggest_skill" | "rewrite" | "block";

  message?: string;
  suggestedCommands?: string[];
  updatedInput?: unknown;
  blockReason?: string;
}
```

## 13.3 触发策略

```text
low-risk active memory → context / warn
medium-risk memory → warn
high-risk policy match → block
active skill match → suggest_skill
degraded skill match → warn only
stale memory match → warn + verify first
superseded memory → ignore
quarantined memory → ignore
```

---

# 14. Candidate Extractor

候选来源：

```text
1. user correction
2. command observation
3. test result
4. file observation
5. repeated workflow
6. stale/conflict detection
7. skill failure
```

## 14.1 Correction Parser

识别关键词：

```text
中文：
不要、别、不能、以后、下次、必须、记住、别再、不是这样、应该

英文：
don't, do not, never, always, next time, remember, should, must, prefer
```

类型判断：

```text
不要 / never → anti_pattern
以后 / next time / when → prospective
我喜欢 / prefer → user_preference
这个项目 / this repo → project_fact
```

## 14.2 Observation Extractor

例子：

```text
存在 pnpm-lock.yaml → project_fact: repo uses pnpm
npm install 生成 package-lock.json → anti_pattern candidate
测试命令连续成功 → project_fact: verified test command
migrations/** 修改后固定命令成功 → prospective / skill candidate
```

---

# 15. Dream Consolidation

## 15.1 触发

```text
1. task_stop 后自动运行
2. idle_post_task
3. aeb dream run
4. benchmark lifecycle
```

## 15.2 输入

```ts
interface DreamInput {
  sessionId: string;
  episodeId: string;
  recentEpisodes: Episode[];
  candidates: MemoryObject[];
  activeMemories: MemoryObject[];
  activeSkills: SkillObject[];
}
```

## 15.3 Dream Runner 流程

```text
1. Load episode
2. Load session events
3. Merge duplicate candidates
4. Extract project facts
5. Extract anti-patterns
6. Extract prospective memories
7. Mine repeated successful workflows
8. Detect stale memories
9. Detect conflicts / supersession
10. Detect skill rollback signals
11. Write dream report
12. Write graph nodes / edges
13. Send candidates to policy engine
```

## 15.4 Dream 不做什么

MVP Dream 不做：

```text
1. 不调用大型复杂推理任务
2. 不自动激活高风险 skill
3. 不自动删除 memory
4. 不自动执行代码
5. 不做团队审批
```

---

# 16. Skill Promotion

## 16.1 Workflow Mining

从 episode 中提取命令序列：

```text
files_changed pattern
→ commands_run
→ verification status
```

如果满足：

```text
same repo
same path pattern
same or similar command sequence
success count >= skill-policy.minEvidence
```

生成 skill candidate。

默认：

```yaml
minEvidence: 2
autoActivate: false
```

## 16.2 Skill Candidate 示例

```yaml
id: skill_cand_update_schema_safely
name: update-schema-safely
summary: Safely update schema after migration changes.
trigger:
  pathPatterns:
    - "migrations/**"
    - "schema/**"
steps:
  - type: command
    command: "pnpm db:generate"
    expectedResult: "Generated schema updated"
  - type: command
    command: "pnpm test:integration -- schema"
    expectedResult: "Schema integration tests pass"
scope:
  repo: current_repo
evidence:
  episodes:
    - ep_001
    - ep_007
confidence: medium
risk: medium
status: candidate
```

## 16.3 Activation

用户执行：

```bash
aeb skill accept skill_cand_update_schema_safely
```

系统：

```text
1. 写入 active skill store
2. 创建 skill node
3. 创建 promoted_to_skill edges
4. 写 audit
5. 后续 recall / trigger 可推荐该 skill
```

---

# 17. Skill Rollback

## 17.1 Failure Capture

在 `post_tool_use` 中，如果命令属于某个 active skill 的 step：

```text
exit_code != 0 → skill usage failure
test failed → skill usage failure
user correction says skill invalid → severe failure
```

## 17.2 降权规则

```yaml
failurePolicy:
  firstFailure:
    confidenceDown: true
  consecutiveFailuresToDegrade: 2
  severeFailureToDisable: true
```

状态变化：

```text
active + failure x1 → active with lower confidence
active + failure x2 → degraded
degraded + failure → disabled
user explicit rejection → disabled
```

## 17.3 Rollback

```bash
aeb skill rollback <skill-id>
```

行为：

```text
1. 当前版本 disabled / rolled_back
2. 如果 previous version 存在，恢复 previous
3. 写 rolled_back_to graph edge
4. 写 audit
```

---

# 18. Memory Aging

## 18.1 Aging Policy

```yaml
halfLifeDays:
  project_fact: 90
  test_command: 30
  dependency_rule: 30
  user_preference: 180
  anti_pattern: 120
  prospective: 90
  skill: 30
  incident: 365
```

## 18.2 Aging Check

触发：

```text
1. task_start
2. dream
3. aeb memory age
```

逻辑：

```text
if now - lastVerifiedAt > halfLifeDays:
  stale = true
  agingScore down
else:
  stale = false
```

## 18.3 Recall 行为

```text
stale memory 可以返回，但必须附带：
- stale warning
- verification suggestion
- lower ranking
```

示例：

```text
Warning: This test command memory is stale. Check package.json before running it.
```

---

# 19. Supersession

## 19.1 Topic Detector

MVP 支持固定 topic：

```text
package_manager
test_framework
test_command
build_command
generated_file_policy
migration_policy
schema_generation_policy
```

## 19.2 Conflict Detector

同一 repo + 同一 topic + 内容互斥：

```text
npm vs pnpm
Jest vs Vitest
npm test vs pnpm test:schema
manual generated editing allowed vs forbidden
```

## 19.3 Supersession Candidate

```ts
interface SupersessionCandidate {
  id: string;
  oldMemoryId: string;
  newCandidateId: string;
  topic: string;
  reason: string[];
  evidence: string[];
  status: "inbox" | "applied" | "rejected";
}
```

## 19.4 Apply

```bash
aeb inbox apply-supersession <id>
```

行为：

```text
1. old.status = superseded
2. new.status = active
3. old.version.supersededBy = new.id
4. new.version.supersedes = [old.id]
5. graph edge: new -> supersedes -> old
6. audit entry
```

---

# 20. Memory Graph

## 20.1 存储

```text
.aeb/graph/nodes.jsonl
.aeb/graph/edges.jsonl
```

## 20.2 写入时机

| 事件                | Graph 写入                                          |
| ----------------- | ------------------------------------------------- |
| event recorded    | event node                                        |
| episode written   | episode node + event → episode                    |
| candidate created | candidate node + episode → candidate              |
| memory accepted   | memory node + candidate → memory                  |
| dream report      | dream node + episode → dream                      |
| skill promoted    | skill node + episode/memory → skill               |
| skill failed      | skill → failed_by → episode                       |
| rollback          | skill version → rolled_back_to → previous version |
| supersession      | new memory → supersedes → old memory              |
| aging             | memory → aged_from → aging report                 |

## 20.3 CLI 查询

```bash
aeb graph trace <id>
aeb graph show <id>
aeb graph related <id>
```

MVP 输出文本树，不做 UI。

---

# 21. Policy Engine

## 21.1 PolicyDecision

```ts
type PolicyAction =
  | "auto_accept"
  | "send_to_inbox"
  | "send_to_quarantine"
  | "reject"
  | "create_skill_candidate"
  | "create_supersession_candidate"
  | "create_rollback_candidate";

interface PolicyDecision {
  action: PolicyAction;
  reason: string[];
  risk: "low" | "medium" | "high" | "critical";
}
```

## 21.2 默认策略

自动接受：

```text
低风险
repo scope 明确
source 是 file_observation / command_result / test_result
evidence 明确
无冲突
不会造成危险行为
```

进入 inbox：

```text
user_preference
anti_pattern
skill_candidate
supersession_candidate
rollback_candidate
medium-risk prospective
```

进入 quarantine：

```text
跳过测试
绕过安全
删除 failing tests
直接 push main
prompt injection
低可信来源指令
模型推理产生高影响规则
跨 repo 全局化
```

---

# 22. CLI 控制面

主路径自动运行，CLI 用于初始化、治理、调试、benchmark。

```bash
aeb init --agent codex
aeb init --agent claude-code
aeb init --agent both

aeb status

aeb inbox
aeb inbox accept <id>
aeb inbox reject <id>
aeb inbox edit <id>
aeb inbox apply-supersession <id>
aeb inbox rollback-skill <id>

aeb quarantine
aeb quarantine approve <id>
aeb quarantine reject <id>

aeb memory list
aeb memory show <id>
aeb memory age
aeb memory supersede <old> <new>

aeb skill list
aeb skill show <id>
aeb skill accept <id>
aeb skill disable <id>
aeb skill rollback <id>
aeb skill usage <id>

aeb dream run
aeb dream show <id>

aeb graph show <id>
aeb graph trace <id>
aeb graph related <id>

aeb audit
aeb bench run
```

---

# 23. Benchmark 架构

Benchmark 必须模拟 Agent lifecycle，而不是模拟用户手动 CLI。

## 23.1 Benchmark Case 结构

```text
benchmarks/
  pnpm-vs-npm/
    case.yaml
    lifecycle.phase1.yaml
    lifecycle.phase2.yaml
    assertions.yaml
    repo/
```

## 23.2 Lifecycle 示例

```yaml
phase1:
  - event: session_start
    prompt: "Add lodash dependency"
  - event: pre_tool_use
    tool: Bash
    command: "npm install lodash"
  - event: post_tool_use
    tool: Bash
    command: "npm install lodash"
    exitCode: 0
  - event: user_prompt_submit
    prompt: "这个项目别用 npm，以后都用 pnpm。"
  - event: stop

phase2:
  - event: session_start
    prompt: "Add dayjs dependency"
  - event: pre_tool_use
    tool: Bash
    command: "npm install dayjs"
```

## 23.3 必备 Benchmarks

```text
1. pnpm-vs-npm
2. interrupted-task
3. migration-trigger
4. dream-skill-promotion
5. skill-rollback
6. memory-aging
7. supersession
8. memory-graph
```

## 23.4 Scorecard

```yaml
case: pnpm-vs-npm
scores:
  recall_happened: true
  correction_captured: true
  memory_created: true
  trigger_matched_phase2: true
  repeated_mistake_avoided: true
  graph_trace_available: true
total: 6/6
```

---

# 24. Milestones

## Milestone 1：Core Store + Schema

交付：

```text
Workspace
FileStore
EventStore
MemoryStore
CandidateStore
SkillStore
GraphStore
AuditLog
Core schemas
```

验收：

```text
.aeb 能保存 events、episodes、memories、skills、graph nodes/edges。
```

---

## Milestone 2：Codex / Claude Adapter

交付：

```text
aeb init --agent codex
aeb init --agent claude-code
hook config generator
skills generator
AGENTS.md / CLAUDE.md fragment
hook command handlers
```

验收：

```text
Codex / Claude hooks 可以自动触发 session_start、pre_tool_use、post_tool_use、stop。
```

---

## Milestone 3：Recall / Trigger / Candidate

交付：

```text
RecallEngine
TriggerEngine
CorrectionParser
CandidateExtractor
PolicyEngine
```

验收：

```text
用户纠正 pnpm 后，后续 npm install 命令触发 warning。
```

---

## Milestone 4：Episode + Dream

交付：

```text
EpisodeWriter
DreamRunner
DreamReport
CandidateMerger
WorkflowMiner
StaleDetector
ConflictDetector
```

验收：

```text
task_stop 自动生成 episode 和 dream report。
```

---

## Milestone 5：Skill Promotion / Rollback

交付：

```text
SkillPromoter
SkillStore
SkillTrigger
SkillUsageTracker
SkillRollback
```

验收：

```text
重复成功流程生成 skill candidate；skill 失败后降权或 rollback。
```

---

## Milestone 6：Aging / Supersession

交付：

```text
AgingCalculator
StaleWarning
TopicDetector
SupersessionEngine
```

验收：

```text
Jest → Vitest 生成 supersession candidate，应用后旧 memory 不再 recall。
```

---

## Milestone 7：Memory Graph

交付：

```text
GraphWriter
GraphQuery
graph trace
graph related
```

验收：

```text
一条 active memory 可以追溯到 candidate、episode、event/evidence。
```

---

## Milestone 8：Benchmark Suite

交付：

```text
Lifecycle simulator
assertions
scorecard
8 个 benchmark cases
```

验收：

```text
aeb bench run 能验证增强型 MVP 核心闭环。
```

---

# 25. 第一版成功 Demo

## Demo 1：Codex 自动避免重复错误

1. 用户初始化：

```bash
aeb init --agent codex
```

2. 用户纠正：

```text
这个项目别用 npm，以后都用 pnpm。
```

3. AEB 自动生成 memory。

4. 下一次 Agent 准备执行：

```bash
npm install dayjs
```

5. PreToolUse hook 返回 warning：

```text
该 repo 使用 pnpm，请改用 pnpm add dayjs。
```

6. Agent 自动改用：

```bash
pnpm add dayjs
```

成功标准：

```text
用户没有手动运行任何 AEB 记忆命令。
```

---

## Demo 2：Claude Code 自动生成 skill

1. 用户修改 migration。
2. Agent 连续两次成功执行：

```bash
pnpm db:generate
pnpm test:integration -- schema
```

3. Stop hook 触发 Dream。
4. Dream 生成：

```text
skill_candidate: update-schema-safely
```

5. 用户接受后，后续修改 migration 自动推荐该 skill。

---

## Demo 3：Skill 失败后降权

1. active skill 推荐旧测试命令。
2. 命令失败。
3. AEB 记录 skill failure。
4. 连续失败后 skill 状态变 degraded。
5. 不再默认推荐。
6. 用户可 rollback。

---

## Demo 4：Memory Aging + Supersession

1. 旧 memory：项目使用 Jest。
2. 新 evidence：package.json 使用 Vitest。
3. Dream 检测冲突。
4. 生成 supersession candidate。
5. 应用后：

   * Jest memory → superseded；
   * Vitest memory → active；
   * graph 记录覆盖关系。
6. recall 只返回 Vitest。

---

# 26. 安全与治理

MVP 必须遵守：

```text
1. 不自动执行危险命令。
2. 不自动删除文件。
3. 不自动 commit / push。
4. 不自动接受高风险记忆。
5. 不使用 quarantined memory。
6. 不默认推荐 degraded / disabled skill。
7. 不删除 superseded memory，只隐藏默认召回。
8. 所有关键动作写 audit。
9. 所有重要 memory 可 graph trace。
```

---

# 27. 结论

AEB v0.3 的技术核心是：

```text
用 Codex / Claude Code hooks 捕获 Agent 生命周期，
用 CLI Runtime 执行本地记忆逻辑，
用 Dream 把 episode 巩固成 memory / skill / graph，
用 Aging / Supersession / Rollback 维持长期记忆健康。
```

第一版不是要做完整平台，而是要证明一个最小但完整的成长闭环：

```text
任务经历
→ episode
→ dream
→ memory / skill
→ recall / trigger
→ 使用反馈
→ aging / supersession / rollback
→ graph provenance
```

只要这个闭环在 Codex 和 Claude Code 中跑通，Agent External Brain 就不再是一个“记忆库”，而是一个真正的 Coding Agent 外置经验运行时。
