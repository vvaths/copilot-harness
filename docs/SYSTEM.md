# Copilot Agent System Documentation

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Agents](#agents)
- [Instructions](#instructions)
- [Prompts (Slash Commands)](#prompts-slash-commands)
- [Hooks](#hooks)
- [Skills](#skills)
- [Shared State Files](#shared-state-files)
- [How to Use This System](#how-to-use-this-system)
- [Troubleshooting](#troubleshooting)
- [Customization](#customization)
- [Glossary](#glossary)

---

## Overview

This system is a **mission-lifecycle orchestration framework** for GitHub Copilot's agent mode in VS Code. It structures AI-driven development into repeatable, validated workflows where:

- Requirements are captured before code is written
- **Validation contracts** define success criteria as testable behavioral assertions before any implementation begins
- Work is **decomposed** into bounded features grouped by milestones
- Dedicated **subagents** handle planning, implementation, and validation with strict separation of concerns
- **Test-first development** is enforced at every level — from individual workers writing tests before code, to the orchestrator defining contracts before features
- Progress is tracked through **shared state files** in `docs/`, not in-context memory

The goal: eliminate unstructured, open-ended AI coding sessions and replace them with disciplined, auditable, multi-phase workflows that converge on correct implementations.

---

## Architecture

### Agent Hierarchy

```
                    ┌──────────────────────┐
                    │     Orchestrator      │
                    │  (plans, delegates,   │
                    │   never writes code)  │
                    └──────────┬───────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                   │
            ▼                  ▼                   ▼
   ┌─────────────┐   ┌─────────────┐   ┌───────────────────┐
   │   Planner   │   │   Worker    │   │ ThoroughValidator  │
   │ (research,  │   │  (TDD per   │   │ (3 parallel review │
   │  read-only) │   │   feature)  │   │   perspectives)    │
   └─────────────┘   └─────────────┘   └────────┬──────────┘
                                                 │
                                      ┌──────────┼──────────┐
                                      │          │          │
                                      ▼          ▼          ▼
                                  Contract   Scrutiny   Security
                                  Checker    Reviewer   Reviewer
```

### Data Flow

```
 User Request
      │
      ▼
 ┌────────────┐   research    ┌──────────┐
 │Orchestrator│──────────────▶│ Planner  │
 │            │◀──────────────│          │
 └─────┬──────┘   plan        └──────────┘
       │
       │  writes
       ▼
 ┌──────────────────────────────────────────────────────┐
 │                   docs/ (shared state)               │
 │  requirements.md → validation-contract.md            │
 │                         → features.json               │
 │                              → services.yaml           │
 └───────────┬─────────────────────────────┬────────────┘
             │ reads                       │ reads
             ▼                             ▼
       ┌──────────┐                ┌────────────────┐
       │  Worker  │──implements──▶ │ Source Code +   │
       │          │                │ Tests           │
       └──────────┘                └───────┬────────┘
                                           │ validates
                                           ▼
                                  ┌──────────────────┐
                                  │ThoroughValidator  │
                                  └──────────────────┘
```

---

## Agents

### Summary Table

| Agent | Role | Tools | Model | User-Invocable | Key Constraint |
|-------|------|-------|-------|----------------|----------------|
| **Orchestrator** | Plans, decomposes, delegates, tracks | `agent`, `read`, `edit`, `search`, `todo`, `web` | Default (strong reasoning) | Yes | Never writes code |
| **Planner** | Researches codebase, creates plans | `read`, `search`, `web` | Default | No | Read-only access |
| **Worker** | Implements features via TDD | `read`, `edit`, `search`, `terminal` | Claude Sonnet 4.6 → GPT-5.4 (fallback) | Yes | TDD mandatory |
| **Validator** | Reviews work, surfaces issues | `read`, `search`, `terminal` | Default | Yes | Never fixes — only reports |
| **ThoroughValidator** | Milestone-level multi-perspective validation | `agent`, `read`, `search`, `terminal` | Default | No | 3 parallel isolated perspectives |

---

### Orchestrator

**File:** `.github/agents/orchestrator.agent.md`

**Role:** Project orchestrator — the brain of the system. Plans work, decomposes features into milestones, delegates to subagents, and tracks progress via todo lists. Maintains all shared state files in `docs/`.

**When invoked:** Directly by the user, or implicitly when using `/run-mission`, `/define-validation-contract`, or `/decompose-features`.

**Subagents:** Planner, Worker, Validator, ThoroughValidator

**Handoffs:**

| Label | Target Agent | Purpose |
|-------|-------------|--------|
| Start Implementation | Worker | Implement the next feature from the plan |
| Validate Feature | Validator | Review the last completed feature |
| Validate Milestone | ThoroughValidator | Validate current milestone against the contract |

**Process:**
1. Understand requirements via Planner
2. Define validation contract → `docs/validation-contract.md`
3. Decompose into features → `docs/features.json`
4. Create `docs/services.yaml` for commands/ports
5. Execute features by delegating to Workers (one at a time)
6. Validate milestones via ThoroughValidator
7. Handle failures with targeted fix features
8. Halt if blocked — hands control to user

**Rules:**
- Never implements code directly
- Never accumulates granular implementation context
- Never drives validation directly
- Always defines validation contract before features
- Warns user if a milestone requires more than 3 validation rounds

---

### Planner

**File:** `.github/agents/planner.agent.md`

**Role:** Research and planning specialist with read-only access. Examines existing code, patterns, and architecture to produce concrete implementation plans.

**When invoked:** By the Orchestrator when it needs to understand the codebase or break down a feature request.

**Output format:**
```
## Feature: <name>
### Tasks
1. <task> — files: [...], depends on: [...]
### Risks
- <potential issue>
### Open Questions
- <anything that needs clarification>
```

**Rules:**
- Cannot modify any files
- Must examine existing code before suggesting approaches
- Must be specific about files, functions, and patterns — no vague tasks

---

### Worker

**File:** `.github/agents/worker.agent.md`

**Role:** Implements features using strict test-first development. Can be used directly for ad-hoc TDD or as a subagent in the mission pipeline.

**When invoked:** Directly by the user for TDD work, or by the Orchestrator during mission execution.

**Model selection:** Uses cheaper/faster models (Claude Sonnet 4.6 as primary, GPT-5.4 as fallback) for cost efficiency.

**TDD Process:**
1. Read feature spec and `AGENTS.md`
2. Write tests first (encoding success criteria)
3. Run tests — confirm they fail (red phase)
4. Implement minimum code to pass tests
5. Run tests — confirm they pass (green phase)
6. Report summary of implementation and any blockers

**Rules:**
- Cannot modify code outside feature scope
- Cannot skip writing tests first
- Cannot evaluate own work for overall correctness (Validator's job)
- Cannot refactor unrelated code
- Must follow existing code patterns

---

### Validator

**File:** `.github/agents/validator.agent.md`

**Role:** Independent code reviewer. Evaluates work with fresh eyes against success criteria or general best practices. Never fixes issues — only surfaces them.

**When invoked:** Directly by the user for code review, or by the Orchestrator for per-feature validation.

**Output format:**
```
## Validation Report
### Assertions Checked
- [ ] Assertion 1: PASS/FAIL — <evidence>
### Issues Found
1. [CRITICAL/HIGH/MEDIUM/LOW] <description> — <file:line>
### Summary
- Assertions passed: X/Y
- Recommendation: PASS / NEEDS FIXES
```

**Rules:**
- Never implements fixes
- Cannot access prior Worker reasoning or trajectories
- Tests behavior as a black box
- Must cite specific files, lines, and evidence

---

### ThoroughValidator

**File:** `.github/agents/thorough-validator.agent.md`

**Role:** Milestone-level validation through 3 independent, parallel review perspectives. Synthesizes findings into a prioritized report with actionable fix features.

**When invoked:** By the Orchestrator after all features in a milestone are complete.

**Parallel Perspectives:**

| Perspective | Focus |
|-------------|-------|
| **Contract Checker** | Verifies each assertion in `docs/validation-contract.md` |
| **Scrutiny Reviewer** | Code quality, correctness, edge cases, naming, duplication |
| **Security Reviewer** | Input validation, injection risks, data exposure, auth gaps |

**Output format:**
```
## Milestone Validation: <name>
### Contract Status
- Assertions passed: X/Y
### Critical Issues (must fix)
1. <issue> — <source perspective> — <evidence>
### Recommended Fixes
1. <fix feature description>
### Passed
- <what's working well>
```

---

## Instructions

Instructions are auto-injected guidelines that apply to matching files.

| Instruction | Applies To | Enforces |
|-------------|-----------|----------|
| **Scoped Changes** | All changes | One concern per change. Don't mix features with refactoring. Keep diffs minimal. |
| **TDD** | `src/**`, `lib/**`, `packages/**` | Test-first: write tests → fail → implement → pass. |

### Scoped Changes

**File:** `.github/instructions/scoped-changes.instructions.md`

- Each change addresses ONE concern
- Don't mix feature work with refactoring
- Only modify files within the current task's scope
- If you discover a refactor need, note it as a separate task
- Keep diffs minimal and reviewable

### TDD

**File:** `.github/instructions/tdd.instructions.md`

Auto-applies to files matching `src/**`, `lib/**`, `packages/**`:
1. Write or update tests FIRST describing intended behavior
2. Run tests to confirm they fail for the right reason
3. Implement the minimum code to make tests pass
4. Run tests again to confirm they pass
5. Do not refactor unrelated code in the same change

---

## Prompts (Slash Commands)

### `/define-validation-contract`

**File:** `.github/prompts/define-validation-contract.prompt.md`
**Agent:** Orchestrator | **Tools:** `read`, `search`, `agent`

Defines testable behavioral assertions before any implementation begins. Uses Planner to research, asks clarifying questions, writes assertions grouped by milestone to `docs/validation-contract.md`.

**Output format:**
```markdown
# Validation Contract

## Milestone 1: Authentication

### VAL-AUTH-001: Successful login
A user with valid credentials submits the login form
and is redirected to the dashboard.
Tool: agent-browser
Evidence: screenshot, network(POST /api/auth/login -> 200)

### VAL-AUTH-002: Invalid credentials rejected
A user with wrong password sees an error message
and is not redirected.
Tool: agent-browser
Evidence: screenshot, network(POST /api/auth/login -> 401)
```

> Stops after contract definition for user review.

### `/decompose-features`

**File:** `.github/prompts/decompose-features.prompt.md`
**Agent:** Orchestrator | **Tools:** `read`, `search`, `agent`

**Prerequisite:** `docs/validation-contract.md` must exist.

Decomposes the contract into ordered features grouped by milestone. Each feature has a description, success criteria, and scope. Output: `docs/features.json`.

> Stops after plan creation for user review.

### `/run-mission`

**File:** `.github/prompts/run-mission.prompt.md`
**Agent:** Orchestrator | **Tools:** `agent`, `read`, `search`, `todo`, `edit`

Executes the full end-to-end workflow:
1. Define validation contract
2. Decompose into features
3. Per milestone: implement features → validate → fix → re-validate (max 3 rounds)
4. Report final status

---

## Hooks

### PostToolUse: Remind Tests

**File:** `.github/hooks/post-edit-remind-tests.json`

Fires when `create_file`, `replace_string_in_file`, or `multi_replace_string_in_file` is used on `.ts/.js/.tsx/.jsx` files. Injects a reminder to run tests.

### SessionStart: Inject Project Context

**File:** `.github/hooks/inject-project-context.json`

Injects project name, active git branch, and working directory into the agent's context at the start of every session.

### PreToolUse: Block Dangerous Commands

**File:** `.github/hooks/block-dangerous-commands.json`

Blocks execution of dangerous terminal commands (`rm -rf /`, `DROP TABLE`, `git push --force`, etc.) before they run.

### SubagentStart: Inject Role Hints

**File:** `.github/hooks/subagent-lifecycle.json`

Injects role-specific reminders when Worker, Planner, Validator, or ThoroughValidator subagents start.

### PreCompact: Save Context

**File:** `.github/hooks/pre-compact-save-context.json`

Reminds the agent to re-read shared state files after context compaction.

### Orchestrator-Scoped Hooks

The Orchestrator agent has two agent-scoped hooks that only fire during mission workflows:

- **Stop**: Blocks the Orchestrator from finishing if `docs/validation-contract.md` doesn't exist
- **UserPromptSubmit**: Reminds about contract-first development when implementation is requested

---

## Skills

### Mission Lifecycle

**File:** `.github/skills/mission-lifecycle/SKILL.md`

Documents the complete mission lifecycle in 5 phases:

| Phase | Activity | Output |
|-------|----------|--------|
| 1. Requirements | Investigate user's goal | `docs/requirements.md` |
| 2. Contract | Define testable assertions | `docs/validation-contract.md` |
| 3. Decomposition | Break into features by milestone | `docs/features.json` |
| 4. Execution | Worker implements, Validator checks, fix loop (max 3) | Source code + tests |
| 5. Completion | All milestones pass | Final report |

**Key Principles:** Context isolation, separation of concerns, test-first at two levels, externalized state, model specialization.

---

## Shared State Files

| File | Format | Purpose | Written By | Read By |
|------|--------|---------|-----------|---------|
| `docs/validation-contract.md` | Structured markdown | Testable assertions with `VAL-*` IDs, tool, evidence | Orchestrator | All agents |
| `docs/features.json` | JSON array | Feature specs with status, fulfillment, verification | Orchestrator | Worker, Orchestrator |
| `docs/services.yaml` | YAML | Commands, services, ports, healthchecks | Orchestrator | Worker, Validator |
| `docs/requirements.md` | Free-form markdown | User requirements and clarifications | Orchestrator | Planner, Orchestrator |
| `AGENTS.md` | Markdown | Mission boundaries, coding conventions, format specs | User / Orchestrator | All agents (always-on) |

### Validation Contract Format (`docs/validation-contract.md`)

Each assertion uses a unique `VAL-<CATEGORY>-<NNN>` ID and specifies the verification tool and evidence type:

```markdown
### VAL-API-001: List endpoint returns results
GET /api/items returns a paginated list of items.
Tool: terminal
Evidence: curl(GET /api/items -> 200), response-shape({data: [], total: number})

### VAL-API-002: Create endpoint validates input
POST /api/items with invalid body returns 400.
Tool: terminal
Evidence: curl(POST /api/items {} -> 400)
```

**Naming convention:** `VAL-<CATEGORY>-<NNN>`
- `AUTH` — authentication/authorization
- `API` — REST endpoint behavior
- `UI` — frontend rendering/interaction
- `DATA` — data integrity/persistence
- `CROSS` — cross-cutting concerns
- `PERF` — performance requirements

### Feature Plan Format (`docs/features.json`)

Features are tracked as a JSON array. Each entry links back to contract assertions:

```json
[
  {
    "id": "user-list-endpoint",
    "description": "GET /api/users - Return a paginated list of users.",
    "milestone": "core-api",
    "expectedBehavior": [
      "Returns 200 with paginated results",
      "Returns 400 on invalid query parameters"
    ],
    "verificationSteps": [
      "npm test -- --grep 'user list'",
      "curl GET /api/users -> 200"
    ],
    "fulfills": ["VAL-API-001"],
    "status": "pending"
  }
]
```

**Status values:** `pending` → `in-progress` → `complete` → `validated`

### Services Configuration (`docs/services.yaml`)

Defines how to build, test, and run the project. Agents read this to discover commands and ports:

```yaml
commands:
  install: npm install
  test: npm test
  build: npm run build

# services:
#   app:
#     start: npm run dev
#     healthcheck: curl -sf http://localhost:3000/health
#     port: 3000
```

---

## How to Use This System

### Quick Start

1. Copy this template to your new project
2. Customize `AGENTS.md` with your project's boundaries and coding conventions
3. Customize `docs/services.yaml` with your project's commands and services
4. Customize `.github/copilot-instructions.md` with your code style
5. Use the default **Agent** mode for everyday development
6. Use **Worker** for TDD workflows or **Validator** for code review
7. Use `/run-mission` for large multi-feature projects

### Everyday Development

For most tasks, use the default Agent mode. The instructions and hooks provide:
- Project conventions (from `copilot-instructions.md` and `AGENTS.md`)
- TDD enforcement on source files (from `tdd.instructions.md`)
- Scoped change discipline (from `scoped-changes.instructions.md`)
- Safety guardrails (from hooks)

Switch to **Worker** when you want strict TDD, or **Validator** for code review.

### Running a Full Mission

```
/run-mission Build a REST API with CRUD endpoints, authentication, and tests.
```

**What happens:**
1. **Contract phase** — Planner researches, Orchestrator writes `docs/validation-contract.md`
2. **Decomposition** — Features grouped by milestone in `docs/features.json`
3. **Execution loop** — Worker implements each feature (TDD) → ThoroughValidator reviews → fix loop
4. **Completion** — Final report with all assertions checked

### Running Individual Steps

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/define-validation-contract` | Define success criteria | Starting a new project |
| `/decompose-features` | Plan the implementation | After contract is approved |
| `/run-mission` | Full automated workflow | End-to-end automation |

### Tips for Best Results

- **Be specific** in your initial request — include edge cases and constraints
- **Review the validation contract carefully** — this is your primary leverage point
- **Keep milestones small** — 3–5 features per milestone is ideal
- **Check `AGENTS.md`** — add your project's coding conventions and boundaries
- **If validation loops exceed 3**, split the milestone into smaller ones

---

## Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| Worker produces vague code | Feature spec too vague | Add specific files and success criteria to `docs/features.json` |
| Too many validation rounds | Milestone too large | Split into smaller milestones |
| Tests not reminded after edits | File extension not matching | Hook only fires for `.ts/.js/.tsx/.jsx` |
| Worker skips tests | TDD instruction not matching | Check `applyTo` pattern in `tdd.instructions.md` |

---

## Customization

### Adding a New Agent

Create `.github/agents/<name>.agent.md`:
```yaml
---
description: "What this agent does"
tools: ['read', 'search']
user-invocable: false
disable-model-invocation: true
---
```

### Adding a New Hook

Create `.github/hooks/<name>.json`:
```json
{
  "hooks": {
    "<EventName>": [{
      "type": "command",
      "command": "<shell command>",
      "timeout": 10
    }]
  }
}
```

Supported events: `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PreCompact`, `SubagentStart`, `SubagentStop`, `Stop`

### Adding a New Instruction

Create `.github/instructions/<name>.instructions.md`:
```yaml
---
description: "When to apply"
applyTo: ["src/**"]
---
```

### Adding a New Prompt

Create `.github/prompts/<name>.prompt.md`:
```yaml
---
description: "What it does"
agent: orchestrator
tools: ['read', 'search']
---
```

### Adding a New Skill

Create `.github/skills/<name>/SKILL.md`:
```yaml
---
name: skill-name
description: "When to activate"
---
```

---

## Glossary

| Term | Definition |
|------|-----------|
| **Mission** | Complete workflow: requirements → contract → features → implement → validate |
| **Milestone** | Logical grouping of features; validation occurs at this level |
| **Validation Contract** | Testable behavioral assertions defining "done" (`docs/validation-contract.md`) |
| **VAL-ID** | Unique assertion identifier: `VAL-<CATEGORY>-<NNN>` (e.g., `VAL-AUTH-001`) |
| **Assertion** | Single testable statement with tool and evidence requirements |
| **Feature** | Bounded implementation unit in `docs/features.json` claiming specific VAL-IDs |
| **Subagent** | Agent invoked by another agent with isolated context |
| **Handoff** | UI-driven transfer of control between agents |
| **Shared State** | `docs/` files and `AGENTS.md` serving as inter-agent communication |
| **services.yaml** | YAML config declaring commands, services, ports, and healthchecks |
| **features.json** | JSON array tracking feature specs, status, and contract fulfillment |
| **AGENTS.md** | Always-on instructions: mission boundaries, conventions, format specs |
| **TDD** | Test-Driven Development: tests first → fail → implement → pass |
| **Execution Loop** | Per-milestone: implement → validate → fix → re-validate (max 3 rounds) |
| **Context Isolation** | Fresh subagent per feature to prevent accumulated bias |
