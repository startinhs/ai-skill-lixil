# Issue Workflow

Use this reference after selecting an issue source and target issue.

## Context Checklist

Before editing behavior, establish and say explicitly:

- Ticket ID.
- Target repo(s).
- Branch name.
- Source: ticket content or CSV/XLSX row.
- Issue description and discussion.
- Existing issue note path, if any.
- Related screen/API/job/component.
- Current behavior observed from code, logs (`D:\PROJECTS\LIXIL\Logs`), or user evidence.
- Expected behavior from ticket, discussion, screenshot, worklog, or user confirmation.
- Files likely in scope, including SQL objects (functions, stored procedures, views).

## Brainstorming Gate

Use `superpowers:brainstorming` before changing behavior. If it is unavailable, ask the user to install/setup it or approve the fallback checklist.

During brainstorming:

- Restate the request in your own words.
- List 2-3 interpretations or fix approaches when ambiguity exists.
- Recommend one approach with trade-offs.
- Ask the user when the repo, screen, field, button, popup, grid, API contract, expected behavior, or duplicate scope is unclear.
- Do not guess business rules.

## Cross-Repo And Duplicate Rule

The same behavior often spans several places: Zalo Mini App screen → `nretail_service_abp` API → DMS/SQL Server via `WebAPI` jobs or stored procedures → BO screens. Also `nretail_zalo_mini_app/BE` contains a backend copy.

If similar behavior exists in multiple files, views, or repos:

1. Search for duplicate components and patterns.
2. List all candidate files and repos.
3. Explain whether they appear to share the same behavior.
4. Ask before applying synchronized edits. One branch per repo, same `{ShortDesc}-{TicketId}-{Owner}`.

Do not silently edit duplicates just because names look similar. Do not change an API contract used by the Mini App without saying so.

## Database Changes

- Never run write/DDL statements yourself.
- Put each change in a script file (e.g. `D:\PROJECTS\LIXIL\docs\sql\<object>_<yyyyMMdd>_<shortDesc>.sql`, matching the existing `eAPI_GetRPT_SalesOrd_20261014_fix_TongTien.sql` style), and a backup/rollback script when altering an existing function or procedure.
- Tell the user which database/server the script targets and wait for their result.

## Stop Conditions

Stop before editing when:

- Current and expected behavior cannot be proven from source, ticket, worklog, screenshot, logs, or user confirmation.
- More than one business interpretation is plausible.
- The required repo/screen/API/component is unknown.
- A dirty/staged file in scope was not created by you and changes the same logic.
- The repo is in merge, rebase, or cherry-pick state.
- The fix branch is owned by another worktree and safe switching is unclear.

## Correction Flow

When the user asks to return to a completed issue:

- Treat it as a correction for that issue.
- Use the same fix branch when possible.
- Read previous commits, worklog, and issue note.
- Do not select the next issue.
- Verify the correction independently.
- Final response must mention it is a correction.
