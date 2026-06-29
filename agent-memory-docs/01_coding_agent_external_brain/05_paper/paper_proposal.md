# 论文 Proposal：Agent Memory Runtime for Long-Horizon Coding Agents

## Title

**Agent Memory Runtime: A Hook-based Evidence-driven Memory System for Long-Horizon Coding Agents**

## 1. Abstract

Current coding agents rely on context windows, retrieval-augmented generation, or rule-based prompts, which are insufficient for long-horizon software engineering tasks. We propose a memory runtime architecture that integrates lifecycle hooks, evidence-grounded episodic logging, checkpoint reconstruction, and skill promotion mechanisms to enable agents to accumulate actionable experience over time.

## 2. Problem Statement

Existing systems suffer from:

- Lack of persistent structured experience across sessions
- No explicit linkage between tool execution and memory formation
- Weak handling of task continuity under context compression
- Absence of verifiable completion checks
- No mechanism for experience-to-skill conversion

## 3. Key Idea

We introduce a runtime memory system composed of:

- Hook-based event capture (SessionStart, ToolUse, Stop, Compact)
- Evidence-grounded episodic memory
- Structured checkpoint reconstruction
- Stop-time verification mechanism
- Skill promotion pipeline

## 4. System Overview

```text
Agent Execution
  ↓
Lifecycle Hooks
  ↓
Episodic Memory Store
  ↓
Memory Gate (filter + score)
  ↓
Checkpoint Builder
  ↓
Retriever + Verifier
  ↓
Skill Distillation Layer
```

## 5. Memory Model

Each memory item contains:

- content
- scope (personal/project/team)
- evidence (tool logs, tests, diffs)
- confidence score
- timestamp
- invalidation links

## 6. Evaluation Metrics

- task success rate
- repeated mistake rate
- skill reuse rate
- checkpoint recovery accuracy
- memory retrieval precision/recall

## 7. Hypothesis

A hook-based, evidence-driven memory runtime will significantly improve long-horizon task completion stability and reduce repeated failure patterns in coding agents.

## 8. Expected Contributions

- A unified memory runtime architecture for coding agents
- A lifecycle hook design for memory capture
- A skill promotion mechanism from episodic traces
- A verifiable checkpoint reconstruction method
