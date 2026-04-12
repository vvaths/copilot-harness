# Copilot Agent Orchestration Template

A project template that turns GitHub Copilot into a disciplined, multi-agent development system inside VS Code. It provides custom agents, instructions, hooks, skills, and prompt files that work together to enforce contract-first, test-first development with automated validation.

**Use this template when starting any new project to get structured AI-assisted development out of the box.**

## What You Get

```
.github/
  agents/              5 custom agents (orchestrator, planner, worker, validator, thorough-validator)
  hooks/               5 lifecycle hooks (safety, TDD reminders, context injection, subagent hints, compaction recovery)
  instructions/        2 auto-applied instructions (TDD, scoped changes)
  prompts/             3 slash commands (/run-mission, /define-validation-contract, /decompose-features)
  skills/              1 skill (mission-lifecycle with templates)
  copilot-instructions.md   Always-on project conventions
.vscode/
  settings.json        Required VS Code settings for nested subagents and agent hooks
AGENTS.md              Always-on agent conventions and shared state formats
docs/
  SYSTEM.md            Full system documentation
  validation-contract.md   Placeholder — replace per project
  features.json        Placeholder — replace per project
  services.yaml        Placeholder — customize per project
```

## Quick Start

### 1. Copy the template

```bash
# Clone or copy into your new project directory
cp -r template-repo/.github your-project/.github
cp -r template-repo/.vscode your-project/.vscode
cp -r template-repo/docs your-project/docs
cp template-repo/AGENTS.md your-project/AGENTS.md
```

### 2. Customize for your project

Edit three files to match your tech stack:

**`AGENTS.md`** — Add your project's boundaries and coding conventions:
```markdown
## Mission Boundaries
- Frontend: port 3000
- Backend: port 3001
- Database: PostgreSQL on localhost:5432

## Coding Conventions
- TypeScript strict mode
- Prisma for database access
- JWT in httpOnly cookies
```

**`docs/services.yaml`** — Define your build/test/run commands:
```yaml
commands:
  install: pnpm install
  test: vitest run
  build: tsc && vite build

services:
  api:
    start: npm run dev
    healthcheck: curl -sf http://localhost:3001/health
    port: 3001
```

**`.github/copilot-instructions.md`** — Add your code style under `## Code Style`:
```markdown
- TypeScript strict mode, no `any` types
- Use Zod for runtime validation
- Prefer functional composition over class hierarchies
```

### 3. Start using it

Open your project in VS Code. You're ready to go.

---

## How to Use GitHub Copilot With This Template

This template supports two development modes: **everyday development** (zero ceremony) and **mission mode** (structured multi-feature projects).

### Everyday Development

For most tasks, use the default **Agent** mode in VS Code chat. The template automatically provides:

- **Project conventions** from `copilot-instructions.md` and `AGENTS.md` — applied to every prompt
- **TDD enforcement** on source files matching `src/**`, `lib/**`, `packages/**`
- **Scoped change discipline** on all files — one concern per change
- **Safety hooks** — dangerous commands blocked, test reminders after edits, project context at session start

No slash commands or special agents needed. Just chat normally.

#### Switch agents for specialized tasks

| Agent | When to use | How to invoke |
|-------|-------------|---------------|
| **Worker** | You want strict TDD: tests first, then implement | Select "Worker" from the agent dropdown |
| **Validator** | You want a code review with structured output | Select "Validator" from the agent dropdown |
| **Agent** (default) | General development, questions, refactoring | Default — just chat |

**Example — using Worker for TDD:**
```
Switch to: Worker agent

"Add a GET /api/users endpoint that returns paginated results"
```
The Worker will write tests first, run them to confirm failure, implement the minimum code, and run tests again.

**Example — using Validator for code review:**
```
Switch to: Validator agent

"Review the authentication module in src/auth/"
```
The Validator will check code quality, test coverage, edge cases, and report findings without modifying anything.

### Mission Mode (Large Features)

For multi-feature projects, use the mission pipeline. This is the template's flagship workflow.

#### Option A: Full automation with `/run-mission`

Type `/run-mission` in chat. You'll be prompted for your goal:

```
/run-mission

Goal: Build a REST API with user authentication, CRUD endpoints for posts,
and role-based access control.
```

**What happens automatically:**

```
1. Contract     → Orchestrator defines VAL-* assertions in docs/validation-contract.md
                  (pauses for your review)
2. Decompose    → Features broken into milestones in docs/features.json
                  (pauses for your review)
3. Per milestone:
   a. Worker    → Implements each feature using TDD
   b. Validator → Quick check after each feature
   c. Thorough  → 3 parallel review perspectives after all features complete
   d. Fix loop  → Issues become new features, max 3 rounds
4. Report       → Final status of all assertions
```

You stay in control — the Orchestrator pauses for review at key decision points.

#### Option B: Step-by-step control

Run each phase manually:

```
/define-validation-contract
Requirements: User auth with email/password, posts CRUD, admin role
```

Review the contract. Then:

```
/decompose-features
```

Review the feature plan. Then use handoff buttons or `/run-mission` to execute.

#### The Validation Contract

The validation contract is the core concept. It defines "done" as testable behavioral assertions **before** any code is written:

```markdown
### VAL-AUTH-001: Successful login
A user with valid credentials receives a session token.
Tool: terminal
Evidence: curl(POST /api/auth/login -> 200), response has "token" field

### VAL-AUTH-002: Invalid credentials rejected
A user with wrong password receives 401.
Tool: terminal
Evidence: curl(POST /api/auth/login -> 401)
```

Every feature in `docs/features.json` must link back to specific `VAL-*` IDs. Validators check these assertions, not vague "does it work" criteria.

---

## Architecture

### Agent Hierarchy

```
                    ┌──────────────────────┐
                    │     Orchestrator      │
                    │  (plans, delegates,   │  ← You interact with this
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
                                      ▼          ▼          ▼
                                  Contract   Scrutiny   Security
                                  Checker    Reviewer   Reviewer
```

### Agents

| Agent | Role | Tools | User-Invocable | Model |
|-------|------|-------|---------------|-------|
| **Orchestrator** | Plans, decomposes, delegates, tracks progress | agent, read, edit, search, todo, web | Yes | Default |
| **Planner** | Researches codebase, creates implementation plans | read, search, web | No (subagent only) | Default |
| **Worker** | Implements features with strict TDD | read, edit, search, terminal | Yes | Claude Sonnet 4.6 / GPT-5.4 |
| **Validator** | Reviews code, surfaces issues (never fixes) | read, search, terminal | Yes | Default |
| **ThoroughValidator** | 3 parallel review perspectives for milestones | agent, read, search, terminal | No (subagent only) | Default |

Workers use cost-efficient models (Claude Sonnet 4.6, GPT-5.4 fallback). The Orchestrator and validators use the default (stronger reasoning) model.

### Hooks

All hooks run automatically — no manual action needed.

| Hook | Event | What it does |
|------|-------|-------------|
| **Block dangerous commands** | `PreToolUse` | Denies `rm -rf /`, `DROP TABLE`, `git push --force`, etc. |
| **Inject project context** | `SessionStart` | Adds project name, git branch, and CWD to every session |
| **Remind tests after edits** | `PostToolUse` | Nudges the agent to run tests after editing `.ts/.js/.tsx/.jsx` files |
| **SubagentStart hints** | `SubagentStart` | Injects role-specific reminders when Worker/Planner/Validator subagents start |
| **Save context on compaction** | `PreCompact` | Reminds the agent to re-read shared state files after context compaction |
| **Require contract** *(Orchestrator only)* | `Stop` | Blocks the Orchestrator from finishing without a validation contract |
| **Contract-first reminder** *(Orchestrator only)* | `UserPromptSubmit` | Reminds about contract-first when implementation is requested in Orchestrator |

The last two are **agent-scoped hooks** — they only fire when the Orchestrator is active, not during everyday development.

### Instructions (Auto-Applied)

| Instruction | Applies to | Enforces |
|-------------|-----------|----------|
| **Scoped Changes** | All files (`**`) | One concern per change, minimal diffs, no mixed refactoring |
| **TDD** | Source files (`{src,lib,packages}/**`) | Write tests first → fail → implement → pass |

### Slash Commands

| Command | Description |
|---------|-------------|
| `/run-mission` | Full pipeline: contract → features → implement → validate |
| `/define-validation-contract` | Define testable assertions before implementation |
| `/decompose-features` | Break the contract into ordered features by milestone |

### Shared State Files

Agents communicate through files in `docs/`, not in-context memory:

| File | Written by | Read by | Purpose |
|------|-----------|---------|---------|
| `docs/validation-contract.md` | Orchestrator | All agents | Testable `VAL-*` assertions |
| `docs/features.json` | Orchestrator | Worker, Validator | Feature specs with status tracking |
| `docs/services.yaml` | You / Orchestrator | Worker, Validator | Build, test, run commands |
| `AGENTS.md` | You | All agents | Coding conventions, boundaries |

---

## Customization

### Add a new agent

Create `.github/agents/my-agent.agent.md`:

```markdown
---
name: MyAgent
description: "What this agent does and when to use it."
tools: ['read', 'search', 'terminal']
---

Instructions for the agent...
```

See the [custom agents docs](https://code.visualstudio.com/docs/copilot/customization/custom-agents) for all frontmatter options (`model`, `handoffs`, `agents`, `hooks`, etc.).

### Add a new instruction

Create `.github/instructions/my-rule.instructions.md`:

```markdown
---
description: "When this rule applies"
applyTo: '**/*.py'
---

# My Rule

- Always do X
- Never do Y
```

### Add a new slash command

Create `.github/prompts/my-task.prompt.md`:

```markdown
---
description: "What this command does"
tools: ['read', 'edit', 'terminal']
---

${input:description:Describe what you want}

Instructions for the task...
```

### Add a new hook

Create `.github/hooks/my-hook.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "node -e \"/* your logic here */\"",
        "timeout": 10
      }
    ]
  }
}
```

Hook events: `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PreCompact`, `SubagentStart`, `SubagentStop`, `Stop`

### Add a new skill

Create `.github/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: "What this skill teaches Copilot and when to use it."
---

# My Skill

Instructions, procedures, and references to scripts in this directory...
```

---

## Requirements

- **VS Code** with GitHub Copilot agent mode
- **Node.js** (for hook scripts — they use inline `node -e` commands)
- **Git** (for the session-start hook to detect the current branch)

### VS Code Settings

The template includes `.vscode/settings.json` with required settings:

```json
{
  "chat.subagents.allowInvocationsFromSubagents": true,
  "chat.useAgentsMdFile": true,
  "chat.useCustomAgentHooks": true,
  "chat.includeApplyingInstructions": true,
  "chat.includeReferencedInstructions": true
}
```

These enable nested subagents (required for ThoroughValidator's parallel reviews), `AGENTS.md` discovery, agent-scoped hooks, and auto-applied instructions.

---

## Tips for Best Results

- **Be specific** in your prompts — include edge cases, constraints, and expected behavior
- **Review the validation contract carefully** — this is your primary leverage point over the entire mission
- **Keep milestones small** — 3–5 features per milestone works best
- **Use Worker directly** for small TDD tasks instead of running a full mission
- **Use Validator directly** for quick code reviews without the full pipeline
- **Check AGENTS.md** — add your project's specific conventions before starting
- **If validation loops > 3**, the milestone is too large — split it

## Further Reading

- [docs/SYSTEM.md](docs/SYSTEM.md) — Complete system documentation with architecture diagrams and troubleshooting
- [VS Code Copilot Customization](https://code.visualstudio.com/docs/copilot/customization/overview)
- [Custom Agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Hooks](https://code.visualstudio.com/docs/copilot/customization/hooks)
- [Subagents](https://code.visualstudio.com/docs/copilot/agents/subagents)

## License

This template is provided as-is. Use it as a starting point for your projects.
