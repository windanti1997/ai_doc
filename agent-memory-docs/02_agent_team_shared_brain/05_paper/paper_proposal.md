# 论文 Proposal：Team Memory Infrastructure for Multi-Agent Coding Systems

## Title

**Team Memory Infrastructure: Scope-aware, Evidence-grounded, and Role-routed Memory for Multi-Agent Coding Systems**

## 1. Abstract

As coding agents become integrated into software teams, memory systems must evolve from personal long-term memory into governed team-level memory infrastructure. Existing approaches often store summaries, rules, or retrieved histories without explicit scope, evidence, authority, or role-based routing. This proposal introduces a team memory infrastructure for multi-agent coding systems, combining memory scope modeling, evidence-grounded trust scoring, prospective triggers, role-based memory routing, and governance mechanisms.

## 2. Motivation

Single-agent memory improves continuity, but team settings introduce new challenges:

- Personal preferences may be incorrectly generalized into team rules.
- Project-specific facts may be recalled in unrelated repositories.
- Multiple agents may receive the same memory despite different roles.
- Shared memory can propagate incorrect or stale rules.
- Team rules are often recalled passively instead of triggered at the right time.

## 3. Research Questions

1. How should agent memories be scoped across personal, project, team, and organization levels?
2. How can memory trust be computed from sources, evidence, and verification history?
3. How can future-facing memories trigger actions when relevant engineering events occur?
4. How should memory be routed across planner, coder, reviewer, test, and security agents?
5. How can team memory systems reduce incorrect sharing while preserving useful knowledge reuse?

## 4. Proposed System

The proposed system includes five components:

### 4.1 Scope Classifier

Classifies memory items into:

```text
Personal / Project / Team / Organization / Reusable Skill
```

### 4.2 Evidence Binder

Links memory items to evidence such as:

```text
Tool outputs, test results, CI logs, PR reviews, code diffs, human approvals
```

### 4.3 Trust Scorer

Computes confidence using source type, verification recency, evidence strength, conflict count, and usage history.

### 4.4 Prospective Trigger Engine

Transforms selected memories into trigger rules, such as:

```text
When backend API changes, remind agents to update schema and generated clients.
```

### 4.5 Role-based Memory Router

Routes different memory subsets to agents based on:

```text
agent_role, task_type, file_path, risk_level, memory_scope
```

## 5. Methodology

### Baselines

- No shared memory
- Shared flat memory store
- RAG over team documents
- Project rule files only
- Proposed scope-aware team memory system

### Tasks

- Multi-agent feature development
- API contract change across backend and frontend
- CI failure debugging with historical causes
- Security-sensitive code review
- Release checklist execution

### Metrics

- task completion rate
- repeated mistake rate
- stale memory usage rate
- cross-scope misuse rate
- memory retrieval precision
- role-routing accuracy
- human correction count
- team rule adoption rate

## 6. Expected Contributions

1. A scope-aware memory model for multi-agent coding systems.
2. An evidence-grounded trust scoring mechanism.
3. A prospective memory trigger framework for engineering workflows.
4. A role-based memory routing approach.
5. A governance design for shared agent memory.

## 7. Hypothesis

Compared with flat shared memory or RAG-based retrieval, a scope-aware and evidence-grounded team memory infrastructure will reduce cross-context misuse and improve multi-agent coding task reliability.

## 8. Future Extensions

- Team memory dashboard
- Memory version control
- Human approval workflow
- Skill ownership and maintenance
- Organization-level memory policies
