---
name: griffin-cli
description: Teaches LLM agents how to use the Griffin CLI to run, validate, and deploy API monitors; manage environments and variables; and interact with the hub. Use when the user wants to run monitors locally, preview or apply monitor changes, manage secrets or variables, check run history, or perform any griffin-cli workflow supporting Griffin monitors.
---

# Griffin CLI

This skill teaches you how to use **griffin-cli** to support Griffin API monitors: local runs, validation, deploying to the hub, and managing configuration. The CLI is the main way to execute monitors locally, preview and apply changes, and operate on environments and secrets.

---

## 1. When to use this skill

- User asks to run monitors locally, validate monitor files, or run Griffin tests.
- User wants to preview or apply monitor changes to the hub, or destroy monitors.
- User needs to set up a project (init), manage environments, or manage variables/secrets.
- User wants to check status, view runs, trigger a run, or view metrics.
- User mentions griffin-cli, `griffin` commands, or workflows that support Griffin monitors.

---

## 2. Command overview

Commands are either **top-level** or under a **group**. The default environment is `default` unless overridden with `--env <name>` where supported.

| Area | Commands | Purpose |
|------|----------|--------|
| **Project** | `init`, `validate`, `status` | Bootstrap, validate monitors, show status |
| **Local run** | `test` | Run monitors locally |
| **Hub sync** | `plan`, `apply`, `destroy` | Preview changes, push to hub, remove from hub |
| **Hub runs** | `runs`, `run`, `metrics` | List runs, trigger run, view metrics |
| **Auth** | `auth login`, `auth logout`, `auth connect`, `auth generate-key` | Cloud or self‑hosted auth |
| **Environments** | `env list`, `env add`, `env remove` | Manage environments |
| **Variables** | `variables list`, `variables add`, `variables remove` | Per-environment variables (in state) |
| **Secrets** | `secrets list`, `secrets set`, `secrets get`, `secrets delete` | Per-environment secrets (stored on hub) |
| **Integrations** | `integrations list`, `integrations show`, `integrations connect`, `integrations update`, `integrations remove` | Slack, email, webhooks, etc. |
| **Notifications** | `notifications list`, `notifications test` | Notification rules and test sends |

---

## 3. Core workflows

### 3.1 Project setup

1. **Initialize** (once per project):
   ```bash
   griffin init
   griffin init --project my-service   # override project ID
   ```
   Creates `.griffin/state.json` with project ID, default environment, and hub config. Add `.griffin/` to `.gitignore`.

2. **Optional: add environments and variables**
   ```bash
   griffin env add staging
   griffin env add production
   griffin variables add API_BASE=https://staging.example.com --env staging
   griffin variables add API_BASE=https://api.example.com --env production
   ```

3. **Connect to hub** (for plan/apply/runs/run/metrics):
   - **Griffin Cloud**: `griffin auth login` (device flow; token stored in `~/.griffin/credentials.json`).
   - **Self‑hosted**: `griffin auth connect --url https://hub.example.com --token <api-key>`.

### 3.2 Local development and validation

- **Validate** monitor files (no run, no hub):
  ```bash
  griffin validate
  ```

- **Run monitors locally** against an environment (uses variables from state for that env):
  ```bash
  griffin test
  griffin test --env staging
  ```

- **Check status** (project, hub connection):
  ```bash
  griffin status
  ```

### 3.3 Preview and deploy to hub

- **Preview** what would be created/updated/deleted (exit code 2 if there are changes):
  ```bash
  griffin plan
  griffin plan --env production --json
  ```

- **Apply** changes to the hub (creates/updates monitors; optionally prune):
  ```bash
  griffin apply --env production
  griffin apply --env production --auto-approve
  griffin apply --env production --dry-run
  griffin apply --env production --prune    # delete hub monitors not present locally
  ```

- **Destroy** monitors on the hub:
  ```bash
  griffin destroy --env production
  griffin destroy --env production --monitor health-check --dry-run
  griffin destroy --env production --auto-approve
  ```

### 3.4 Runs and metrics

- **List recent runs**:
  ```bash
  griffin runs
  griffin runs --env production --monitor health-check --limit 20
  ```

- **Trigger a run**:
  ```bash
  griffin run --env production --monitor health-check
  griffin run --env production --monitor health-check --wait
  griffin run --env production --monitor health-check --force   # even if local differs from hub
  ```

- **Metrics summary**:
  ```bash
  griffin metrics --env production
  griffin metrics --env production --period 7d --json
  ```

### 3.5 Variables and secrets

Variables are stored in `.griffin/state.json` per environment and used when running monitors (e.g. for `variable("api-service")` in monitor DSL). Secrets are stored on the hub per environment and referenced in monitors via `secret("REF")`.

- **Variables** (in state; not sensitive):
  ```bash
  griffin variables list --env default
  griffin variables add API_BASE=https://localhost:3000 --env default
  griffin variables remove API_BASE --env default
  ```

- **Secrets** (on hub; use for tokens, API keys):
  ```bash
  griffin secrets list --env production
  griffin secrets set API_TOKEN --env production
  griffin secrets set API_TOKEN --env production --value "sk-..."
  griffin secrets get API_TOKEN --env production
  griffin secrets delete API_TOKEN --env production --force
  ```

---

## 4. File and config locations

- **State**: `.griffin/state.json` (project root). Holds `projectId`, `environments` (with optional `variables` per env), `hub`, `cloud`, and optional `discovery` (pattern/ignore). Do not commit if it contains local-only overrides; add `.griffin/` to `.gitignore` if desired.
- **Credentials**: `~/.griffin/credentials.json` (user-level). Used by `auth login` and `auth connect --token`. Do not commit.
- **Monitors**: Discovered from `__griffin__` directories; pattern is configurable in state under `discovery.pattern` (default `**/__griffin__/*.{ts,js}`), with `discovery.ignore` (e.g. `["node_modules/**", "dist/**"]`).

---

## 5. Environment and defaults

- Most hub-related and run commands accept `--env <name>`. Default is `default`.
- Set default env for the shell: `export GRIFFIN_ENV=production` (if the CLI respects it; otherwise pass `--env` explicitly).
- Variables and secrets are scoped per environment; ensure the right `--env` when adding or listing.

---

## 6. Checklist for common tasks

**First-time setup**
- [ ] Run `griffin init` (and optionally `griffin env add` for extra environments).
- [ ] Add variables with `griffin variables add KEY=value --env <env>` as needed for monitor `variable("...")` refs.
- [ ] Use `griffin auth login` or `griffin auth connect` if you will use plan/apply/runs/run/metrics.

**Before deploying**
- [ ] Run `griffin validate` to ensure monitor files are valid.
- [ ] Run `griffin test --env <env>` to confirm monitors pass locally.
- [ ] Run `griffin plan --env <env>` to preview hub changes; then `griffin apply --env <env>` (use `--dry-run` or `--auto-approve` as appropriate).

**After changing monitors**
- [ ] `griffin validate` then `griffin test`; then `griffin plan` and `griffin apply` for the target environment.

**Secrets used in monitors**
- [ ] Create/update with `griffin secrets set REF --env <env>`; ensure secret ref names match monitor DSL (e.g. `API_TOKEN`, not `api-token`).

---

## Summary

1. **Setup**: `griffin init`; optionally `env add`, `variables add`, and `auth login` or `auth connect`.
2. **Local**: `griffin validate` and `griffin test --env <env>`.
3. **Hub**: `griffin plan` to preview; `griffin apply` to sync; `griffin runs` / `griffin run` / `griffin metrics` to observe.
4. **Config**: Variables in state via `griffin variables`; secrets on hub via `griffin secrets`; both are per-environment.
5. Use `--env` consistently when targeting a non-default environment; use `griffin status` to verify project and connection.
