---
name: mission-lifecycle
description: 'Manage the full mission lifecycle: define validation contracts, decompose features, execute milestones, and run validation loops. Use for orchestrating multi-step autonomous work.'
user-invocable: false
---

# Mission Lifecycle

## Overview

A mission is a structured workflow for completing large projects through focused, isolated agent work with explicit validation gates.

## Lifecycle Phases

### 1. Requirements Gathering
- Investigate the user's goal
- Ask clarifying questions until requirements are unambiguous
- Document requirements in `docs/requirements.md`

### 2. Validation Contract
- Define testable behavioral assertions BEFORE any features
- Each assertion uses a `VAL-<CATEGORY>-<NNN>` ID with tool and evidence fields
- Group assertions by milestone
- Save to `docs/validation-contract.md`

### 3. Feature Decomposition
- Break work into bounded features, each claiming which `VAL-*` IDs it fulfills
- Write features as a JSON array to `docs/features.json`
- Also create `docs/services.yaml` with project commands and service definitions
- Order by dependency within each milestone

### 4. Execution Loop (per milestone)
1. Hand each feature to a Worker subagent (one at a time)
2. Worker writes tests first, then implements
3. After all milestone features complete, invoke ThoroughValidator
4. If validation surfaces issues, create fix features and re-run
5. Repeat until milestone passes (max 3 validation rounds)

### 5. Completion
- All milestones pass validation
- Final validation contract status: all assertions checked
- Report generated

## Key Principles

| Principle | Implementation |
|-----------|---------------|
| Context isolation | Fresh subagent per feature — no accumulated bias |
| Separation of concerns | Orchestrator plans, Workers implement, Validators evaluate |
| Test-first at two levels | Workers write tests before code; Orchestrator defines contract before features |
| Externalized state | Shared files (`docs/`) instead of in-context accumulation |
| Model specialization | Orchestrator uses strong reasoning model; Workers use cost-efficient models (Claude Sonnet 4.6) |

## Shared State Files

| File | Format | Purpose |
|------|--------|--------|
| `docs/requirements.md` | Markdown | User requirements and clarifications |
| `docs/validation-contract.md` | Structured markdown | Testable assertions with `VAL-<CATEGORY>-<NNN>` IDs, tool, and evidence |
| `docs/features.json` | JSON array | Feature specs with status, fulfillment, and verification steps |
| `docs/services.yaml` | YAML | Commands, services, ports, and healthchecks |
| `AGENTS.md` | Markdown | Mission boundaries, coding conventions, format specs (always-on) |

## Templates

Use these templates as starting points for shared state files:

- [Validation contract template](./validation-contract-template.md) — starter for `docs/validation-contract.md`
- [Features template](./features-template.json) — starter for `docs/features.json`

## Troubleshooting

- **Worker stuck**: Check if the feature spec is specific enough. Vague specs produce vague work.
- **Validator finds too many issues**: The feature was under-specified or the worker skipped tests.
- **Validation loops > 3**: The milestone is too large. Split it into smaller milestones.
