---
description: "Execute the full mission workflow: contract → features → implement → validate. Use for large multi-step projects."
agent: orchestrator
tools: ['agent', 'read', 'edit', 'search', 'todo', 'web']
---

${input:goal:Describe your project goal or feature request}

Run the full mission workflow for the above request:

1. `/define-validation-contract` — Define behavioral assertions using `VAL-*` IDs before anything else.
2. `/decompose-features` — Break work into features in `docs/features.json` grouped by milestones. Also create `docs/services.yaml`.
3. **For each milestone**:
   a. Hand each feature to a Worker subagent (one at a time, in order). Run the Validator for a quick per-feature check. Update feature `status` in `docs/features.json`.
   b. After all features in the milestone complete, run the ThoroughValidator subagent.
   c. If validation fails, create fix features and re-run workers.
   d. Repeat until milestone passes (max 3 rounds).
4. Report final status.

Track all progress with #tool:todo.
