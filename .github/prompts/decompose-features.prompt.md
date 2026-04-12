---
description: "Decompose a validated contract into ordered features grouped by milestone. Use after the validation contract is defined."
agent: orchestrator
tools: ['agent', 'read', 'edit', 'search', 'todo']
---

The validation contract has been defined. Now decompose the work into features.

## Steps

1. Read `docs/validation-contract.md` to understand the `VAL-*` assertions.
2. Use a Planner subagent to research the codebase and identify what needs to change.
3. Create a list of **bounded features** as a JSON array, each with:
   - `id`: kebab-case identifier (e.g., `user-list-endpoint`)
   - `description`: what the feature implements
   - `milestone`: which milestone it belongs to
   - `expectedBehavior`: array of testable behavior statements
   - `verificationSteps`: array of commands/checks to verify
   - `fulfills`: array of `VAL-*` IDs this feature satisfies
   - `status`: `"pending"`
4. Group features by milestone and order by dependency.
5. Save to `docs/features.json`.
6. Also create or update `docs/services.yaml` with project commands and service definitions.

## Example Output (`docs/features.json`)

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

DO NOT begin implementation until the feature plan is reviewed.
