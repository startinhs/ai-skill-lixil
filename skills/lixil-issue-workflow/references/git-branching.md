# Git And Branching

Use this reference before creating or switching branches.

## Baseline

Run these checks first, inside the target repo:

```powershell
git status --short --branch
git diff --staged --name-only
git worktree list
git remote -v
```

Also check whether the repo is in merge/rebase/cherry-pick state.

`BO/AR20400` has no git repo: skip branching, list changed files in the final response instead.

## Base Branch Per Repo

| Repo | Base branch |
|---|---|
| `nretail_service_abp` | `develop` |
| `nretail_zalo_mini_app` | `develop` |
| `WebAPI` | `main` |
| `IN23201` | `main` |

If the user names another base, use it. If the repo history shows the team now works on a different branch, ask before branching.

Before creating a new fix branch:

```powershell
git fetch origin {Base}
git checkout {Base}
git pull origin {Base}
```

If the current worktree has unrelated dirty files, use a separate worktree or ask before switching. Never discard those changes.

### nretail_service_abp remotes

- `origin` = `customized-projects/lixil/nretail/nretail_service_abp` — the only push target.
- `source` = `standard-products/.../nretail_service_abp` — the standard product. Never push to it. Do not merge `source/*` or `custom/lixil` into a fix branch unless the user asks.

## Branch Format

```text
fix/{ShortDesc}-{TicketId}-{Owner}   ← khi có mã ticket
fix/{ShortDesc}-{Owner}              ← khi không có mã ticket
```

**Rules:**
- `{ShortDesc}` — tên ngắn gọn mô tả nội dung fix, ASCII, không dấu cách; dùng camelCase hoặc lowercase words nối bằng `-`. Ví dụ: `shopCancelOrder`, `appshops-dms-sync`, `dashboardTotal`.
- `{TicketId}` — mã ticket đầy đủ: `TKT-YYYYMMDD-NNNNN`, `TASK-YYYY-NNNNN`, hoặc `BUG-YYYY-NNNNN`. Bỏ qua nếu không có mã.
- `{Owner}` — username của người fix. Mặc định: `tinhlm`.

**Examples:**
```text
fix/shopCancelOrder-TKT-20261014-00007-tinhlm
fix/appshopsSync-TASK-2026-01279-tinhlm
fix/appshops-dms-sync-tinhlm
```

Resolve `{Owner}` before creating or switching branches:

- Use the owner explicitly provided by the user.
- Otherwise infer it from `.claude-worklog`, the current branch, an existing local branch, or an existing remote branch for the same fix.
- If no owner can be inferred, default to `tinhlm` or ask once.
- After `{Owner}` is resolved, reuse it consistently in branch name, push command, trace comments, and worklog.

If the branch exists locally, switch to it after verifying it is not held by another worktree.

If the branch exists on origin but not locally:

```powershell
git fetch origin fix/{ShortDesc}-{TicketId}-{Owner}
git checkout -b fix/{ShortDesc}-{TicketId}-{Owner} origin/fix/{ShortDesc}-{TicketId}-{Owner}
```

If the branch does not exist:

```powershell
git checkout -b fix/{ShortDesc}-{TicketId}-{Owner}
```

If there is no ticket ID, omit `-{TicketId}` in the commands.

## Forbidden

- Do not push to `develop` or `main` (in `nretail_service_abp`, a push to `develop` triggers the api-dev deploy pipeline).
- Do not merge the fix branch into the base branch; the user merges or opens the MR.
- Do not push to the `source` remote.
- Do not run destructive commands such as `git reset --hard` or `git checkout -- <file>` unless the user explicitly asks.
- Do not use `git add -A`.

## Push

Push only the fix branch:

```powershell
git push -u origin fix/{ShortDesc}-{TicketId}-{Owner}
```

If push is rejected because remote has new commits, stop and inspect. Do not force push unless the user explicitly instructs it and the branch ownership is clear.
