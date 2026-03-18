# dilu-ctl-skill

[Claude Code](https://claude.com/claude-code) plugin for [Dilu](https://github.com/baowk/dilu) — a Go web framework based on Gin + GORM.

Provides 4 skills covering the full development lifecycle: **create → generate → run → test**.

## Installation

```bash
# Local development
claude --plugin-dir ./dilu-ctl-skill
```

## Prerequisites

- [Go](https://go.dev/) 1.21+
- [dilu-ctl](https://github.com/baowk/dilu-ctl) CLI tool:
  ```bash
  go install github.com/baowk/dilu-ctl@latest
  ```

## Skills

### `/dilu-ctl:create` — Create Project

Scaffold a new Dilu project from the official template.

```
/dilu-ctl:create my-app --https
/dilu-ctl:create my-app -o /path/to/workspace --all
```

What it does:
1. Runs `dilu-ctl create` to clone and initialize the project
2. Configures `go.mod` (including local `replace` if applicable)
3. Runs `go mod tidy` and verifies compilation

### `/dilu-ctl:gen` — Generate Module

Generate a complete CRUD module from a database table.

```
/dilu-ctl:gen -t articles -d "./data.db" --driver sqlite --prefix /api
/dilu-ctl:gen -t sys_user -d "root:pass@tcp(localhost:3306)/mydb" -p sys
```

What it generates:
- GORM Model & type-safe Query (via gorm-gen)
- DTO, Service, API handler (with Swagger annotations)
- Router registration
- `cmd/start/{pkg}.go` entry point

### `/dilu-ctl:run` — Run Application

Build and start the Dilu server locally.

```
/dilu-ctl:run
/dilu-ctl:run -c resources/config.sqlite.yaml
```

What it does:
1. Compiles the project
2. Selects appropriate config file
3. Starts the server in background
4. Verifies startup and reports registered routes

### `/dilu-ctl:test` — E2E Test

Run full CRUD tests against a running Dilu service.

```
/dilu-ctl:test
/dilu-ctl:test http://localhost:7888 --resource /api/articles
```

What it does:
1. Discovers registered routes from server logs
2. Executes Create → Page → Get → Update → Delete → Verify
3. Outputs a test report table
4. Checks error logs

## Example: Full Workflow

```
> /dilu-ctl:create my-blog --https
> /dilu-ctl:gen -t articles -d "./test.db" --driver sqlite --prefix /api
> /dilu-ctl:run -c resources/config.sqlite.yaml
> /dilu-ctl:test
```

## Route Convention

| Scenario | Route Pattern | Example |
|----------|--------------|---------|
| Package == Table name | `/{prefix}/{pkg}/...` | `/api/articles/page` |
| Package != Table name | `/{prefix}/{pkg}/{module}/...` | `/api/blog/articles/page` |

## License

MIT
