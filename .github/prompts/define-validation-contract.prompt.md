---
description: "Define a validation contract — testable behavioral assertions — before any implementation begins. Use when starting a new project, feature, or mission."
agent: orchestrator
tools: ['read', 'search', 'agent']
---

${input:requirements:Describe the requirements or paste a spec}

Before writing any features or code, define the VALIDATION CONTRACT for this project.

## Steps

1. Use a subagent to research the requirements thoroughly — examine existing code, docs, and specs.
2. Ask clarifying questions if requirements are ambiguous.
3. Write behavioral assertions using **structured `VAL-<CATEGORY>-<NNN>` IDs**.
4. Each assertion MUST include: a description, a verification tool, and expected evidence.
5. Group assertions by milestone if the project has natural phases.
6. Save the contract to `docs/validation-contract.md`.

## Format

Use this exact structure for each assertion:

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

## Milestone 2: API

### VAL-API-001: List endpoint returns paginated results
GET /api/cards returns a paginated list of cards.
Tool: terminal
Evidence: curl(GET /api/cards -> 200), response-shape({data: [], total: number})
```

## Category Prefixes

- `AUTH` — authentication/authorization
- `API` — REST endpoint behavior
- `UI` — frontend rendering/interaction
- `DATA` — data integrity/persistence
- `CROSS` — cross-cutting concerns (e.g., auth gates other features)
- `PERF` — performance requirements

DO NOT define features or begin implementation until this contract is reviewed and approved.
