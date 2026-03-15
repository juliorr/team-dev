# Team Dev — Multi-Agent Development Orchestrator

A Claude Code skill that orchestrates a team of specialized AI agents to develop web/mobile projects in parallel. Think of it as having a virtual development team — architect, developers, testers, DevOps, and stakeholder — all working together autonomously.

## What It Does

`team-dev` reads your project plan or spec, breaks it into parallelizable tasks, and dispatches specialized agents that:

- **Architect** — Defines the tech stack (greenfield) or validates existing architecture, creates API/DB contracts, performs code reviews, resolves conflicts between developers
- **Developers** — Implement features in isolated git worktrees, write tests, and auto-fix failures (up to N retries)
- **Tester** — Runs unit/integration/e2e tests, reports failures with actionable details, validates regressions
- **DevOps** — Configures Docker, CI/CD pipelines, deployment scripts, and environment setup
- **Stakeholder** — Validates that implementations meet business requirements and UX expectations

## How It Works

### 5-Phase Workflow

```
INIT → DESIGN → DEVELOP → INTEGRATE → DEPLOY
```

1. **INIT** — Reads the plan, detects if the project is greenfield or existing, creates shared state (`.team/` directory)
2. **DESIGN** — Architect defines contracts and conventions; coordinator decomposes the plan into tasks
3. **DEVELOP** — Multiple developers work in parallel via isolated git worktrees; tester validates continuously
4. **INTEGRATE** — Worktrees are merged, integration tests run, architect reviews, stakeholder validates
5. **DEPLOY** — DevOps sets up CI/CD and deployment; smoke tests run before production (always requires user approval)

### Auto-Fix & Escalation

When a test fails, the system handles it automatically:

```
Test fails → Developer retries fix (up to 3 times)
  → Architect intervenes (1 attempt)
    → Escalates to you (the user)
```

### Shared State

All agents communicate through a `.team/` directory in your project:

```
.team/
├── config.json     # Autonomy level, retry limits, roles
├── state.json      # Current phase, agent statuses, progress
├── backlog.json    # Tasks with assignments and dependencies
├── comms/          # Inter-agent messages (JSON)
└── reports/        # Test results, reviews, deploy status
```

## Usage

### In Claude Code

Simply tell Claude to use the skill:

```
Use team-dev to build this project based on the plan at docs/plan.md
```

Or with specific options:

```
/team-dev --plan docs/plan.md --autonomy high-autonomy --max-retries 5
```

### Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--plan` | Auto-detect in `docs/` | Path to your project plan or spec |
| `--autonomy` | `semi-autonomous` | `semi-autonomous` or `high-autonomy` |
| `--max-retries` | `3` | Max auto-fix retries before escalating |
| `--roles` | All enabled | Comma-separated list of roles to activate |

### Autonomy Levels

**Semi-Autonomous** (default) — Asks for your approval at key checkpoints:
- Design approval
- Phase completion
- Integration approval
- Deploy approval
- Conflict resolution
- Auto-fix limit exceeded

**High-Autonomy** — Agents resolve most issues themselves. Only asks you for:
- Deploy to production approval
- Unresolvable blockers

## Installation

Copy the skill to your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/juliorr/team-dev.git ~/.agents/skills/team-dev
```

## Project Structure

```
team-dev/
├── SKILL.md                        # Coordinator (entry point)
├── skills/
│   ├── architect/SKILL.md          # System Architect agent
│   ├── developer/SKILL.md          # Developer agent
│   ├── tester/SKILL.md             # QA Tester agent
│   ├── devops/SKILL.md             # DevOps Engineer agent
│   └── stakeholder/SKILL.md        # Business Validator agent
├── templates/
│   ├── config.json                 # Project configuration template
│   ├── backlog.json                # Task backlog template
│   └── state.json                  # Team state template
└── lib/
    └── prompts.md                  # Shared protocols and role definitions
```

## Supported Project Types

The skill auto-detects your tech stack by scanning for common files:

| File | Detected Stack |
|------|---------------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `requirements.txt`, `pyproject.toml` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `docker-compose.yml` | Docker |
| `openapi.yaml` | API spec available |

For **greenfield projects** (no code yet), the Architect agent defines the stack based on your plan.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or IDE extension
- Git (for worktree-based parallel development)
- GitHub CLI (`gh`) recommended for PR workflows

## License

MIT
