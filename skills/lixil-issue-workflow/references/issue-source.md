# Issue Source

Use this reference when locating and validating the user's issue source.

## Accepted Inputs

LIXIL issues usually come as an ERP/SynergyHub ticket:

```text
TKT-YYYYMMDD-NNNNN                                    (ví dụ TKT-20261014-00007)
https://erp.hqsoft.vn/app/task/TASK-YYYY-NNNNN
https://erp.hqsoft.vn/synergyhub#/bugs/BUG-YYYY-NNNNN
```

The agent cannot log in to erp.hqsoft.vn. If the user gives only a link or ID, ask them to paste the ticket content (title, description, screenshots/attachments, expected behavior).

Also accept UAT lists as files, usually in `D:\PROJECTS\LIXIL\ai-skill-lixil\input\`:

```text
input/<source>.csv
input/<source>.xlsx
```

Do not pick the "latest" file in `input/` on your own; use the file the user names.

If the user does not provide a source, read `.claude-worklog/state.md` first and infer the current function and current issue. If inference is not possible, ask for the source.

## Columns (CSV/XLSX)

Template: `assets/issue-template.csv`

```csv
Ticket ID,Priority,Repo/App,Screen/Topic,Issue Description (VN),Discussion
```

Accept extra columns and any column order. Accept the variants `Issue No` for `Ticket ID` and `Dicussion item` for `Discussion`. If `Repo/App` is missing, infer the repo from the screen/topic and confirm with the user.

## Validation Cases

Stop and ask or report clearly when:

- The source file does not exist, or only an ERP link was given without content.
- The file is `.xlsx` and no spreadsheet-reading method is available.
- Vietnamese text appears corrupted because of encoding.
- Required columns are missing.
- The target ticket is absent.
- The same ticket appears multiple times with conflicting content.
- Worklog/progress says an issue is done but the user asks to continue it; treat it as a correction, not the next issue.
- The ticket references images/attachments that are not available.
- The target repo is unclear (e.g. Mini App screen vs nRetail API vs BO screen vs WebAPI job).

## Selection Rules

- If the user names a ticket, that ticket wins over `Current Issue` in worklog.
- If the user says "next issue", use `.claude-worklog/state.md` and progress to select the next not-started issue.
- If the user says "go back to ticket X", work on X and its branch even if progress marks it done.
- If source and progress conflict, explain the conflict and use the user's latest instruction when safe.
