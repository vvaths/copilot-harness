# Validation Contract

<!-- Replace this with your project's validation assertions using /define-validation-contract -->

## Milestone 1: Example

### VAL-EXAMPLE-001: Example assertion
Describe the expected behavior in plain language.
Tool: terminal
Evidence: test-output
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
