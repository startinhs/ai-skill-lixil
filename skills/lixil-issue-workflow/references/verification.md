# Verification

Use this reference while implementing and before reporting completion.

## TDD Or Regression Baseline

Use `superpowers:test-driven-development` when adding a testable bugfix. At minimum, run a RED or baseline check before implementation:

- Existing failing test, if available (`nretail_service_abp/test/*` has per-feature test projects).
- New focused regression test.
- Static scan that demonstrates the missing/wrong behavior.
- Log evidence or a manual reproduction note when automation is not practical.

If a RED test does not fail, the test has not proven the bug. Fix the test or choose another baseline.

`dotnet test` builds the project, so ask the user before running it (AGENTS.md Rule 13).

## Green Checks

After implementation:

- Run the focused static check that covers the fix; run focused tests when the user agrees.
- Inspect `git diff` for scope.
- Hand the build command to the user and wait for the result before commit/push. Do not run it yourself.

Build commands per repo:

| Repo | Command (run by user) |
|---|---|
| `nretail_service_abp` | `dotnet build nRetailApp.sln -c Debug` |
| `WebAPI` | `dotnet build WebAPI.sln -c Debug` |
| `IN23201` | Build `IN23201.sln` in Visual Studio (.NET Framework 4.8) |
| `BO/AR20400` | Build `AR20400.sln` in Visual Studio (.NET Framework 4.6.1) |
| `nretail_zalo_mini_app` | `zmp start` and check the screen (no build script; `npm run build:css` if Tailwind classes changed) |

If the user reports a build failure, fix it and ask for a rebuild. If the user explicitly waives the build, record that in the final response.

## Trace Comments

For every bugfix in comment-capable source, add a trace comment near the fixed logic:

```csharp
// TKT-20261014-00007 | fix/shopCancelOrder-TKT-20261014-00007-tinhlm | d6634849
// Block duplicate ShopCancelOrder requests so one order gets only one return order.
```

Rules:

- Use the real ticket ID, branch owner, and fix commit hash.
- Keep the second line focused on why the fix exists.
- Do not describe obvious code mechanics.
- Use the comment syntax valid for the file type (`//`, `--`, `{/* */}` in JSX, `@* *@` in Razor).
- If comments are unsupported, do not break syntax; record the trace in tests or worklog.

Because a commit hash cannot be embedded in its own commit, default bugfix flow with hash comments is:

1. Implement fix and provisional trace comment without hash if needed.
2. Run RED/GREEN checks; user builds.
3. Commit the fix.
4. Capture the fix commit hash.
5. Update trace comments with the fix hash.
6. Commit the trace/comment update.
7. Push the fix branch.

## Completion Gate

Do not say the issue is fixed until:

- Diff is scoped.
- Required checks have passed or skipped checks are explicitly reported.
- The user has confirmed the build, or explicitly waived it.
- SQL scripts (if any) are handed over and their run status is reported.
- Commit(s) are created.
- Branch is pushed when the workflow requires push.
- Worklog is updated locally.
