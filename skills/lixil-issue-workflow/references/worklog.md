# Worklog

Use this reference when reading or updating `.claude-worklog`.

The worklog lives at the workspace root (`D:\PROJECTS\LIXIL\.claude-worklog`) because one ticket can span several repos.

## Read First

Read these before choosing or editing an issue:

```text
.claude-worklog/state.md
.claude-worklog/functions/<function-slug>/progress.md
.claude-worklog/issues/issue-{TicketId}.md
```

If paths are missing, infer what you can from the repos and ask only when selection is unsafe. Create the folder on first use.

## Update After Done

After verification, commit, and push, update locally:

```text
.claude-worklog/state.md
.claude-worklog/functions/<function-slug>/progress.md
.claude-worklog/issues/issue-{TicketId}.md
```

Include:

- Ticket ID, repo(s), and branch(es).
- Status.
- Commit hash(es).
- Push destination.
- Summary of behavior changed.
- SQL scripts handed over and whether the user has run them.
- Tests/static checks/build results (build reported by user).
- Any user confirmation or ambiguity resolved.

## Commit Rule

Do not stage or commit `.claude-worklog/` unless the user explicitly asks.

If the user asks to commit worklog, stage only the specific worklog files and make that decision visible in the final response.

## Final Response Checklist

Final response should include:

- Ticket ID.
- Repo(s) and branch(es).
- Commit hash(es).
- Push status.
- Files changed at a high level.
- SQL scripts and target database, if any.
- Verification commands/results.
- Worklog updated locally and not committed, if applicable.
- Unrelated dirty files left untouched.
