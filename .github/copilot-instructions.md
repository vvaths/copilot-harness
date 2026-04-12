# Project-Wide AI Instructions

## Project Structure

This project uses a mission-lifecycle orchestration framework. See [docs/SYSTEM.md](docs/SYSTEM.md) for full documentation and [AGENTS.md](AGENTS.md) for mission boundaries and conventions.

## Key Conventions

- **Contract-first development**: Always define `docs/validation-contract.md` (with `VAL-*` IDs) before implementing features
- **Test-first development**: Write tests before implementation code in `src/`, `lib/`, `packages/`
- **Scoped changes**: Each change addresses ONE concern — never mix features with refactoring
- **Externalized state**: Use structured files in `docs/` for inter-agent communication:
  - `docs/validation-contract.md` — assertions with `VAL-<CATEGORY>-<NNN>` IDs
  - `docs/features.json` — feature specs, status tracking, contract fulfillment
  - `docs/services.yaml` — commands, services, ports, healthchecks

## Agent Workflow

- The **Orchestrator** plans and delegates — it never writes code
- **Workers** read their feature from `docs/features.json` and implement using TDD
- **Validators** review against `VAL-*` assertions — they never fix issues
- Use `/run-mission` for end-to-end workflows
- Use `/define-validation-contract` and `/decompose-features` for step-by-step control

## Code Style

<!-- Customize these for your project's tech stack -->

- Follow existing patterns in the codebase before inventing new ones
- Keep functions small and focused
- Prefer explicit over implicit
- Handle errors at system boundaries only
