# GenCards — Agent Conventions

## Mission Boundaries

**Port Range:** 3100-3101 only.
- Frontend: port 3100
- Backend: port 3101

**Database:**
- USE existing PostgreSQL on localhost:5432
- Do NOT start, stop, or modify the PostgreSQL server

**Off-Limits:**
- Ports 3000-3010 (user's dev servers)
- Any Docker containers not belonging to this project

Workers: return to orchestrator if you cannot
complete work within these boundaries.

## Coding Conventions

- TypeScript strict mode, no `any` types
- Prisma for all database access, no raw SQL
- JWT tokens in httpOnly cookies, bcrypt for hashing
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
### VAL-AUTH-001: Successful login
A user with valid credentials submits the login form
and is redirected to the dashboard.
Tool: agent-browser
Evidence: screenshot, network(POST /api/auth/login -> 200)
```

### Feature Specs

Features are tracked in `docs/features.json` as structured objects:

```json
{
  "id": "auth-login-endpoint",
  "description": "POST /api/auth/login - Validate credentials, issue JWT, set session cookie.",
  "milestone": "authentication",
  "expectedBehavior": [
    "Returns 200 with session cookie on valid credentials",
    "Returns 401 with error message on invalid credentials"
  ],
  "verificationSteps": [
    "npm test -- --grep 'auth login'",
    "curl POST /api/auth/login with valid creds -> 200"
  ],
  "fulfills": ["VAL-AUTH-001"],
  "status": "pending"
}
```

### Services Configuration

Project commands and services are defined in `docs/services.yaml`:

```yaml
commands:
  install: pnpm install
  test: npm run test
  build: turbo build

services:
  api:
    start: PORT=3101 npm run dev:api
    healthcheck: curl -sf http://localhost:3101/health
    port: 3101
```
