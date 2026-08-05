---
name: gsx-vrf-autorun
description: "Auto-run every `<how-to-verify>` step of a GSD `checkpoint:human-verify` task (PLAN.md): background tasks for cold-start, Playwright MCP for web E2E (≤10 retries/step), curl for APIs, self-seed+test; skip un-judgeable steps; summary in conversation. Verify-only — never fix/release/write files. Arg = task # (4.4.1) or Plan # (4.4 / Plan 4); inferred from context/STATE.md if blank. Invoke: 'auto-verify human-verify', 'autorun', 'gsx-vrf-autorun'. Sibling: gsx-vrf-approved releases the gate; gsx-vrf-review summarizes changes."
argument-hint: "[task # e.g. 4.4.1, or Plan # e.g. 4.4 / Plan 4; inferred from context/STATE.md if blank]"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
  - mcp__playwright__browser_navigate
  - mcp__playwright__browser_navigate_back
  - mcp__playwright__browser_snapshot
  - mcp__playwright__browser_click
  - mcp__playwright__browser_type
  - mcp__playwright__browser_fill_form
  - mcp__playwright__browser_select_option
  - mcp__playwright__browser_press_key
  - mcp__playwright__browser_hover
  - mcp__playwright__browser_wait_for
  - mcp__playwright__browser_take_screenshot
  - mcp__playwright__browser_console_messages
  - mcp__playwright__browser_network_requests
  - mcp__playwright__browser_evaluate
  - mcp__playwright__browser_handle_dialog
  - mcp__playwright__browser_tabs
  - mcp__playwright__browser_close
---

# gsx-vrf-autorun

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Machine-pre-run the `<how-to-verify>` checklist of a GSD `checkpoint:human-verify` gate (`gate="blocking"`). Pre-step for `gsx-vrf-approved` (which releases the gate).

**Boundary (hard rules):**
- Verify only, never fix. Bug → report Gap section; don't change business code (fixes → `/gsd:quick` / `/gsd:debug`).
- Never release. Never reply `<resume-signal>`; on all-pass, prompt user to run `gsx-vrf-approved`.
- No writes / no edits to `PLAN.md` / `STATE.md` / `SUMMARY.md`. Report is conversation-only. No commits, no branch switches.
- Test data not force-cleaned (left for re-verification); report lists what was created.
- Reply in Chinese.

## 0 — Parse arguments, locate the target gate

Numbering: phase dirs `.planning/phases/{NN}-{slug}/` (2-digit zero-pad); `X.Y` = Phase X / Plan Y; `X.Y.Z` = Phase X / Plan Y / Zth `<task>`.

**`$ARGUMENTS` has a number** — normalize (case / `Phase`·`Plan`·`Task` prefix / whitespace all optional):

| User writes | Parsed as | Scope |
|-------------|-----------|-------|
| `4.4`, `Phase 4.4`, `Plan 4.4`, `04-04` | Phase 04 / Plan 04 | **All** human-verify tasks in Plan |
| `Plan 4`, `Plan 04` | Plan 04 in current Phase (context/STATE.md) | Same |
| `4.4.1`, `Task 4.4.1` | Phase 04 / Plan 04 / 1st task | That task only (confirm human-verify) |
| `Task 3` | 3rd task in active Plan | That task only |

Locate via glob (zero-pad 2 digits; Plan may have letter suffix, e.g. `04-03a-PLAN.md`):

```bash
ls .planning/phases/04-*/04-04*-PLAN.md
```

0 or multiple matches → list candidates, ask user, stop (don't guess).

**`$ARGUMENTS` empty** — infer:
1. Conversation: executor just returned a `checkpoint:human-verify` gate message? Its Plan is the target.
2. Else read `.planning/STATE.md` `executing` active phase; glob its `*-PLAN.md`; keep ones that **have a `checkpoint:human-verify` task and no `*-SUMMARY.md` in the same dir**.
3. Exactly 1 → use it. 0 or multiple → "Could not uniquely locate a pending checkpoint:human-verify gate — please specify a task / Plan number." and stop.

No `.planning/` → not a GSD project; tell user and stop.

## 1 — Parse `<how-to-verify>` steps

Read all `<task>` of the Plan in document order; check each `type`.

**Plan number** → filter all `type="checkpoint:human-verify"` tasks, verify each. None → "Plan 0X-0Y has no human-verify task" and stop.

**Task number** → take the Zth task. human-verify → verify it. `auto`/other → "Task X.Y.Z is not a human-verify gate", point out the real human-verify task index, stop (don't silently switch).

For each gate task extract: `<what-built>` (feature scope accepted); `<how-to-verify>` (**core**: break into atomic steps, record each "expected observable result"); `<resume-signal>` (report footer hint only; never sent).

Also read same-dir `{NN}-CONTEXT.md` / `{NN}-*-SUMMARY.md` / `*-UI-SPEC.md` for: APIs/pages under test, credentials, initial data, env prerequisites. Root `CLAUDE.md` "Local Run" = authoritative cold-start guide.

## 2 — Classify each step

| Category | Signals | Execution |
|----------|---------|-----------|
| **Prep / cold-start** | start backend/frontend, docker-compose, Flyway migration, service up | **background tasks** (step 3), confirm readiness |
| **Web E2E** | UI behavior: page/console/list/drawer/button/click/toast/form validation/badge | **Playwright MCP** |
| **Pure API** | HTTP contract: error codes, response fields, headers, `/openapi/*`, `/admin/*`, DB persistence | **curl** (+ `mysql`/`redis-cli`) |
| **Skip** | see below | `skip` |

**Skip when (can't machine-judge):** subjective with no objective anchor (color/tint, "matches prototype", polish, layout, animation); prerequisite not self-constructible (real DID SDK, real upstream API, prod deploy/monitoring/backup, external systems); too vague (no success criterion); mixed step where automation covers only part → run coverable part, mark rest "partially skipped". When in doubt, **prefer skip**.

After classifying, show a brief table (gate task / step # / summary / category), then proceed.

## 3 — Prepare the environment

Probe first:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/ ; curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/
```

**Steps require cold-start** → stop first: `docker-compose down` (no `-v` — keep volumes), kill running dap-admin / frontend bg processes. **Otherwise** → start per below if down; reuse if online.

Follow `CLAUDE.md` "Local Run" via background tasks (`run_in_background`):
1. `docker-compose up -d` (MySQL 5.7 + Redis 7).
2. `cd dap-server && mvn -pl dap-common install -DskipTests` — install latest dap-common to `~/.m2` (foreground, wait).
3. `mvn -pl dap-admin spring-boot:run` — background (:8080). **`spring-boot:run` cannot use `-am`.**
4. `cd dap-frontend && pnpm install && pnpm dev` — background (:3000).
5. Poll: `curl` :8080 / :3000 until HTTP responds; check dap-admin logs for Flyway migration done, no real `ERROR`.

> ⚠️ **Destructive ops need consent.** Step needing "reset DB / delete `docker/mysql/data` or `docker/redis/data`" → `AskUserQuestion` first; declined → `skip`, note "DB reset required — not authorized". Local JDK must be 21; on Java version error switch `JAVA_HOME` per `CLAUDE.md`.

Services fail / migration errors → mark prep `fail`, record log excerpt, mark dependent steps `blocked`, stop trying.

## 4 — Execute steps

Run in `<how-to-verify>` order (data chains across steps — respect order).

**Data seeding:** self-seed, self-test. Login if needed (credentials from SUMMARY/CONTEXT; seed admin `admin`); "change password on first login" → walk it, note new password. Create prerequisite data (institutions/apps/scenes/templates) via API or UI.

**4a — Prep / cold-start:** start per step 3; confirm ready (migration version, ports, key pages load, no real ERROR). Record observed values.

**4b — Web E2E (Playwright MCP):** `browser_navigate` to target (`:3000`, `/console`, `/login`) → `browser_snapshot` (a11y tree) → `browser_click`/`browser_type`/`browser_fill_form`/`browser_select_option` → `browser_wait_for` on async (no blind sleep) → `browser_snapshot` to verify text/list/toasts, `browser_console_messages` for JS errors, `browser_network_requests` for API responses → `browser_take_screenshot` only to diagnose failures, `browser_close` at end.

**Retry cap: ≤10 ops/step.** Past 10 → `blocked`, record where it stuck, move on.
Playwright MCP unavailable → mark all web steps `blocked`, note "Playwright MCP not ready — web steps need manual verification." Don't downgrade to reading code and guessing.

**4c — Pure API (curl):** assert HTTP status + `R` wrapper `code`/`message`/`data`. Headers: Open API → `x-dap-appkey`+`x-dap-appsecret`; Data API → `x-dap-appkey`+`x-dap-token`; Admin API → session. DB/Redis via `mysql`/`redis-cli` (`t_audit_log`, `dap:token:*`, `dap:vc:revoked:*`). Record request sent / received / matched.

**Judgment:** all expected items match → `pass`; any mismatch → `fail` (expected vs actual); env/selector stall + retries exhausted → `blocked`; can't objectively judge after running → `skip` (don't guess).

## 5 — Output summary report (conversation only)

Multiple gate tasks → one block each.

```markdown
## Auto-Verification Report — Plan 0X-0Y

### Gate Task N · <task name>
Summary: total: M · pass: a · fail: b · skip: c · blocked: d

| # | Step | Type | Result | Notes |
|---|------|------|--------|-------|
| 1 | ... | Prep/Web/API | pass/fail/skip/blocked | actual result |

(repeat per human-verify task)

## Skipped Steps (manual follow-up)
- Task N Step K "<summary>" — reason

## Gaps (issues found)
- Task N Step M "<summary>" — fail: expected X, actual Y. Fix via /gsd:quick or /gsd:debug.
- (also blocked env + automation blind spots)

## Test Data Created
- institutions / apps / scenes / credentials / tokens created this run.
```

Report **must** call out: (1) skipped steps (which, why); (2) Gaps (fail/blocked + blind spots).

**Closing prompt:**
- All pass, no skips/Gaps → "Passed all steps — confirm, then run `gsx-vrf-approved` to release."
- Skips, no fail/blocked → "No issues, but N step(s) need manual follow-up — release after verifying those."
- Fail/blocked → "Gaps found — fix (via /gsd:quick or /gsd:debug) and re-verify before releasing."

Always remind: auto-verification ≠ human judgment — review skips first.
