# Agent Conventions

## Mission Boundaries

<!-- Customize these boundaries for your project -->

**Ports:**
- Define your service ports in `docs/services.yaml`
- Do NOT use ports reserved for other local dev servers

**External Services:**
- Do NOT start, stop, or modify external services (databases, etc.) unless explicitly configured in `docs/services.yaml`

Workers: return to orchestrator if you cannot
complete work within these boundaries.

## Coding Conventions

<!-- Customize these for your project's tech stack -->

- Follow existing patterns in the codebase before inventing new ones
- Keep functions small and focused
- Prefer explicit over implicit
- Handle errors at system boundaries only

## Shared State Format

Agents communicate through structured files in `docs/`:

| File | Format | Purpose |
|------|--------|---------|
| `docs/validation-contract.md` | Structured markdown with `VAL-*` IDs | Defines success criteria |
| `docs/features.json` | JSON array | Tracks features, status, and fulfillment |
| `docs/services.yaml` | YAML | Commands, services, ports, healthchecks |
| `docs/requirements.md` | Free-form markdown | User requirements and clarifications |

## File Format Conventions

### Validation Contract Assertions

Each assertion uses a unique `VAL-<CATEGORY>-<NNN>` ID and includes a verification tool and evidence type:

```markdown
### VAL-EXAMPLE-001: Example assertion
A user performs an action and sees the expected result.
Tool: terminal
Evidence: test-output, curl(GET /api/resource -> 200)
```

### Feature Specs

Features are tracked in `docs/features.json` as structured objects:

```json
{
  "id": "example-feature",
  "description": "Implement the example feature.",
  "milestone": "example-milestone",
  "expectedBehavior": [
    "Returns expected result on valid input",
    "Returns error on invalid input"
  ],
  "verificationSteps": [
    "npm test -- --grep 'example feature'"
  ],
  "fulfills": ["VAL-EXAMPLE-001"],
  "status": "pending"
}
```

### Services Configuration

Project commands and services are defined in `docs/services.yaml`:

```yaml
commands:
  install: npm install
  test: npm test
  build: npm run build

services:
  app:
    start: npm run dev
    healthcheck: curl -sf http://localhost:3000/health
    port: 3000
```
