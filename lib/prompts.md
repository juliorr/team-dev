# Shared Role Definitions & Prompts

## Team Communication Protocol

All agents communicate through the `.team/` directory in the project root:

- **State**: Read/write `.team/state.json` to track phase and progress
- **Backlog**: Read `.team/backlog.json` for task assignments
- **Comms**: Write messages to `.team/comms/` as JSON files
- **Reports**: Write reports to `.team/reports/` as JSON files

### Message Format (comms/)

```json
{
  "id": "MSG-001",
  "from": "tester",
  "to": "developer",
  "type": "test-failure",
  "taskId": "TASK-003",
  "timestamp": "2025-01-15T10:30:00Z",
  "content": {
    "summary": "Unit test auth.test.ts failing",
    "details": "Expected 401, got 500 on invalid token",
    "files": ["src/auth/middleware.ts"],
    "testOutput": "..."
  }
}
```

### Report Format (reports/)

```json
{
  "type": "test-results",
  "phase": "develop",
  "timestamp": "2025-01-15T10:30:00Z",
  "summary": { "passed": 12, "failed": 2, "skipped": 1 },
  "failures": [
    {
      "test": "auth.test.ts > should reject invalid tokens",
      "error": "Expected 401, received 500",
      "file": "src/auth/middleware.ts",
      "assignedTo": "TASK-003"
    }
  ]
}
```

## Escalation Chain

When a problem cannot be resolved at the current level:

```
Developer (auto-fix, up to maxAutoFixRetries)
  → Architect (redesign/review, 1 attempt)
    → User (final decision)
```

Each escalation must include:
1. What was attempted
2. Why it failed
3. Suggested alternatives

## Task Status Values

| Status | Meaning |
|--------|---------|
| `pending` | Not started |
| `in-progress` | Agent actively working |
| `testing` | Implementation done, tests running |
| `auto-fixing` | Test failed, developer retrying (track retry count) |
| `escalated` | Exceeded retry limit, needs higher-level review |
| `blocked` | Waiting on dependency or external input |
| `review` | Code review by architect |
| `done` | Completed and verified |

## Stack Detection Patterns

When the project already has code, detect the stack by checking for:

| File | Stack |
|------|-------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `requirements.txt`, `pyproject.toml`, `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml`, `build.gradle` | Java/Kotlin |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | .NET/C# |
| `pubspec.yaml` | Dart/Flutter |
| `docker-compose.yml` | Docker (infra) |
| `Dockerfile` | Docker (container) |
| `.github/workflows/` | GitHub Actions CI |
| `openapi.yaml`, `openapi.json` | API spec exists |

## Worktree Naming Convention

When dispatching developers to isolated worktrees:

```
Format: team-dev-{role}-{task-scope}
Example: team-dev-backend-api-auth
         team-dev-frontend-dashboard
         team-dev-mobile-map-screen
```
