---
name: lixil-issue-workflow
description: "Workflow for processing LIXIL project issues (ERP tickets TKT-/TASK-/BUG-, UAT Excel/CSV lists) into isolated fix branches with mandatory requirement analysis, git safety, verification, trace comments, commit/push rules, user-run DB scripts, and local worklog updates. Use when fixing bugs or handling UAT issues in the HQSOFT LIXIL repos (nretail_service_abp, WebAPI, IN23201, BO/AR20400, nretail_zalo_mini_app), especially requests that mention a ticket ID, erp.hqsoft.vn link, Excel issue file, or .claude-worklog."
---

# LIXIL Issue Workflow

## Overview

Use this skill to process one LIXIL issue at a time from a ticket (ERP/SynergyHub) or a user-provided Excel/CSV list and the workspace worklog. The workflow is deliberately conservative: read context first, use one branch per issue, never push straight to a deploy branch, verify before claiming done, push only the issue branch, and leave user/unrelated changes untouched.

Workspace root: `D:\PROJECTS\LIXIL`.

## Required Companion Skill

Before any bug fix or behavior change, use **`superpowers:brainstorming`** to restate the issue, identify ambiguities, list options, and confirm scope before any edit.

If it is unavailable, use the fallback checklist: read ticket/CSV/worklog/issue notes, restate the issue, identify repo/screen/API/component, list current vs expected behavior, ask when ambiguous.

## Reference Map

Read these references as needed:

- `references/issue-source.md`: ticket IDs, Excel/CSV inputs, columns, encoding, mismatch cases.
- `references/issue-workflow.md`: end-to-end issue flow and stop conditions.
- `references/git-branching.md`: per-repo base branch, branch/worktree/push rules.
- `references/verification.md`: RED/GREEN, static checks, build hand-off, trace comments.
- `references/worklog.md`: `.claude-worklog` update rules and final reporting.
- `references/commit-prompt.md`: commit message format.
- `assets/issue-template.csv`: header-only issue CSV template.

## LIXIL Repos

| Repo (under workspace root) | Stack | Base branch | Notes |
|---|---|---|---|
| `nretail_service_abp` | ABP, .NET 8, PostgreSQL, Blazor host (`nRetailApp.sln`) | `develop` | API for Zalo Mini App / retailer web. CI builds + deploys `develop` to api-dev. Remote `source` = standard product repo — never push to it. Status notes in `crew/`, `.crew/`, `*-STATUS.md`. |
| `WebAPI` | .NET 6 (`WebAPI.sln`) | `main` | Integration jobs (AppShops ↔ DMS, PIM, order-status mail). |
| `IN23201` | ASP.NET MVC, .NET Framework 4.8, Ext.NET, EDMX | `main` | Back-office screen (product/media mapping). |
| `BO/AR20400` | .NET Framework 4.6.1 (`AR20400.sln`) | — (no git) | Back-office screen. No branch/commit flow; report changed files instead. |
| `nretail_zalo_mini_app` | Zalo Mini App, React + Vite + TypeScript | `develop` | Release versions noted in commit messages (`rls 1.29`). `BE/` holds a backend copy — ask before editing it. |

Other workspace folders: `docs/` (perf guides, plans, `docs/sql/` scripts), `RELEASE_BUILD/` (release packages + `release-note.txt`), `Logs/`. Read relevant `docs/` files before touching performance, promotion, or report logic.

If the base branch above does not match what the user or the repo shows (`git remote show origin`, recent history), ask before branching.

## Non-Negotiable Rules

- Work on exactly one ticket per branch unless the user explicitly requests otherwise.
- Branch format: `fix/{ShortDesc}-{TicketId}-{Owner}` (with ticket) or `fix/{ShortDesc}-{Owner}` (without). Examples: `fix/shopCancelOrder-TKT-20261014-00007-tinhlm`, `fix/appshops-dms-sync-tinhlm`. Default `{Owner}` = `tinhlm`.
- `{ShortDesc}` must be short, ASCII, no spaces: use camelCase or lowercase words separated by `-`.
- Never push directly to `develop` or `main`; pushing `develop` in `nretail_service_abp` deploys api-dev. Push only the fix branch; merging is done by the user/MR.
- Never push to the `source` remote of `nretail_service_abp`.
- Never execute database writes (INSERT/UPDATE/DELETE/DDL, function/procedure changes). Write the SQL script to a file and hand it to the user to run. Read-only queries only with the user's agreement.
- Do not run `dotnet build` / `msbuild` (AGENTS.md Rule 13). Give the user the build command and wait for their result before commit/push.
- Do not stage or commit `.claude-worklog/`, Excel/CSV issue files, `appsettings*.json` / `Web.config` credentials, or `RELEASE_BUILD/` unless the user explicitly asks.
- Stage intended files explicitly. Never run `git add -A`.
- Preserve unrelated dirty/untracked files. Do not revert user changes.
- If a branch is held by another worktree, report the path and handle it safely.
- If requirements are unclear, ask before changing behavior.
- If an issue may affect duplicate views/APIs/files (e.g. Mini App screen + API + BO screen, or `nretail_zalo_mini_app/BE` copy), identify them and ask before applying synchronized edits.

## Default Execution Flow

1. Read `.claude-worklog/state.md`, function progress, and the issue note if present.
2. Read the ticket content or the CSV/XLSX row for the target issue.
3. Identify the target repo(s). Check `git status`, current branch, staged files, and worktrees.
4. Resolve `{Owner}`, sync the repo's base branch, then create/switch the fix branch.
5. Use `superpowers:brainstorming` to analyze the requirement before editing.
6. Establish ticket, repo, branch, screen/API/component, current behavior, expected behavior, and file scope.
7. Run a RED check or a documented baseline static/regression check before the fix.
8. Implement the smallest scoped change. DB changes go to a SQL script file for the user.
9. Add trace comments for bugfixes in comment-capable source files.
10. Run static/GREEN checks; hand the build command to the user and wait for the result.
11. Inspect diff for scope, stage files explicitly, commit using `references/commit-prompt.md`.
12. If trace comments require the fix commit hash, add/update those comments after the fix commit and commit the trace update.
13. Push only the fix branch to origin.
14. Update local worklog/progress/issue note, but do not commit those files unless requested.
15. Final response must state ticket, repo, branch, commits, push status, verification, SQL scripts handed over, and untouched dirty files.

## Trace Comment Format

For every bugfix in a comment-capable source file, add a short trace comment near the corrected logic:

```csharp
// TKT-20261014-00007 | fix/shopCancelOrder-TKT-20261014-00007-tinhlm | d6634849
// Block duplicate ShopCancelOrder requests so one order gets only one return order.
```

Use the real ticket ID, branch owner, and fix commit hash. Because a commit hash cannot be embedded inside the same commit that creates it, use a second trace/comment commit when the user requires the hash in source.

If the changed file format does not support comments, do not break syntax. Record the trace in tests, worklog, or the nearest comment-capable source file that owns the behavior. SQL scripts use `-- ` comments with the same format.
