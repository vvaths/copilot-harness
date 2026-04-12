# Validation Contract — System Documentation & Improvements

## Milestone 1: Comprehensive Documentation

### VAL-DOC-001: System documentation exists
`docs/SYSTEM.md` exists and contains sections for all 5 agents, 2 instructions,
3 prompts, 4 hooks, and 1 skill.
Tool: file-read
Evidence: file-exists(docs/SYSTEM.md), section-count >= 12

### VAL-DOC-002: Architecture diagrams
`docs/SYSTEM.md` includes an architecture diagram showing the agent hierarchy
and a data flow diagram showing shared state communication.
Tool: file-read
Evidence: content-match(```...Orchestrator...Worker...Planner```)

### VAL-DOC-003: Usage guide
`docs/SYSTEM.md` includes a "How to Use" section with quick start,
full mission workflow, and individual step instructions.
Tool: file-read
Evidence: section-exists(Quick Start), section-exists(Running a Full Mission)

### VAL-DOC-004: Troubleshooting and customization
`docs/SYSTEM.md` includes a troubleshooting table and a customization guide
for adding new agents, hooks, instructions, prompts, and skills.
Tool: file-read
Evidence: section-exists(Troubleshooting), section-exists(Customization)

### VAL-DOC-005: Glossary
`docs/SYSTEM.md` includes a glossary defining key terms: mission, milestone,
validation contract, assertion, feature, subagent, handoff.
Tool: file-read
Evidence: section-exists(Glossary), term-count >= 10

## Milestone 2: Always-On Project Instructions

### VAL-INST-001: Project-wide instructions
`.github/copilot-instructions.md` exists with project-wide conventions
covering contract-first, TDD, and scoped changes.
Tool: file-read
Evidence: file-exists(.github/copilot-instructions.md)

### VAL-INST-002: AGENTS.md boundaries
`AGENTS.md` exists at workspace root with mission boundaries, coding conventions,
and shared state format documentation with examples.
Tool: file-read
Evidence: file-exists(AGENTS.md), content-match(Mission Boundaries)

## Milestone 3: Improved Hooks

### VAL-HOOK-001: Session context injection
A `SessionStart` hook exists that injects project name, branch, and CWD
into the agent's context at the start of every session.
Tool: file-read
Evidence: file-exists(.github/hooks/inject-project-context.json), json-key(hooks.SessionStart)

### VAL-HOOK-002: Dangerous command blocking
A `PreToolUse` hook exists that blocks dangerous terminal commands
(rm -rf, DROP TABLE, git push --force, git reset --hard).
Tool: file-read
Evidence: file-exists(.github/hooks/block-dangerous-commands.json), json-key(hooks.PreToolUse)

## Milestone 4: Agent Improvements

### VAL-AGENT-001: Internal agent isolation
Internal agents (Planner, Worker, Validator) have `disable-model-invocation: true`
to prevent accidental auto-invocation outside the orchestrator workflow.
Tool: file-read
Evidence: frontmatter-match(planner.agent.md, disable-model-invocation: true)

### VAL-AGENT-002: Orchestrator argument hint
Orchestrator has `argument-hint` for better UX guidance in the chat input field.
Tool: file-read
Evidence: frontmatter-match(orchestrator.agent.md, argument-hint)
