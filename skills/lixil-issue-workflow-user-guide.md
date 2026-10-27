# LIXIL Issue Workflow Skill - User Guide

Tài liệu này dành cho người dùng Claude Code trong workspace LIXIL khi xử lý ticket ERP (TKT/TASK/BUG) hoặc danh sách issue Excel/CSV, cùng với `.claude-worklog`.

Skill được link vào workspace (mở Claude Code tại `D:\PROJECTS\LIXIL`):

```text
.claude/skills/lixil-issue-workflow   → junction tới ai-skill-lixil/skills/lixil-issue-workflow
.claude/skills/tinhlm-skill           → lối tắt, điền sẵn owner tinhlm
```

Lệnh gọi trong Claude Code:

```text
/tinhlm-skill            (khuyến nghị — đã có mặc định owner)
/lixil-issue-workflow    (bản gốc — phải tự ghi Repo/Branch owner)
```

## Các repo

| Repo | Stack | Base branch |
|---|---|---|
| `nretail_service_abp` | ABP .NET 8, PostgreSQL | `develop` (push `develop` = deploy api-dev) |
| `WebAPI` | .NET 6, job tích hợp AppShops/DMS/PIM | `main` |
| `IN23201` | ASP.NET MVC .NET Fx 4.8, Ext.NET | `main` |
| `BO/AR20400` | .NET Fx 4.6.1 | không có git |
| `nretail_zalo_mini_app` | Zalo Mini App React + TS | `develop` |

## Skill này dùng để làm gì

- Xử lý ticket UAT/bug từ ERP (`TKT-YYYYMMDD-NNNNN`, `TASK-YYYY-NNNNN`, `BUG-YYYY-NNNNN`) hoặc file Excel/CSV.
- Đọc `.claude-worklog/state.md`, progress và issue note để tìm issue tiếp theo.
- Tạo branch riêng cho từng ticket, trên đúng repo.
- Phân tích yêu cầu trước khi sửa behavior.
- Viết script SQL cho thay đổi DB (người dùng tự chạy).
- Commit/push branch fix đúng convention của team.
- Cập nhật worklog local sau khi xong.

Skill không thay thế việc xác nhận nghiệp vụ. Nếu issue mơ hồ, agent phải hỏi lại.

## Chuẩn bị trước khi gọi skill

Cần có tối thiểu:

- Repo, ví dụ: `nretail_service_abp`
- Nội dung ticket (agent không đăng nhập được erp.hqsoft.vn — hãy dán nội dung), hoặc file nguồn trong `D:\PROJECTS\LIXIL\ai-skill-lixil\input\`
- Branch owner suffix, ví dụ: `tinhlm`
- Worklog/progress nếu đã có:
  - `.claude-worklog/state.md`
  - `.claude-worklog/functions/<function-slug>/progress.md`
  - `.claude-worklog/issues/issue-{TicketId}.md`

Branch fix sẽ có format:

```text
fix/{ShortDesc}-{TicketId}-{Owner}
fix/{ShortDesc}-{Owner}
```

Ví dụ:

```text
fix/shopCancelOrder-TKT-20261014-00007-tinhlm
fix/appshops-dms-sync-tinhlm
```

Commit theo convention của team:

```text
Fix[TKT-20261014-00007]: block duplicate ShopCancelOrder requests
Fix(AppShopsSync): run sync jobs on Vietnam time
```

## Prompt khuyến nghị cho người mới

```text
/tinhlm-skill

Repo: <repo>
Ticket: <TKT-.../link ERP> (dán nội dung ticket bên dưới) hoặc Issue source: <csv-or-xlsx-path>
Branch owner: <owner-suffix>

Hãy xử lý ticket này theo đúng LIXIL issue workflow.

Yêu cầu:
- Trước khi sửa code, hãy nói rõ ticket, repo, branch sẽ dùng, màn hình/API liên quan, current behavior, expected behavior, file scope.
- Nếu thiếu thông tin hoặc có nhiều cách hiểu, hãy hỏi tôi trước. Không đoán nghiệp vụ.
- Dùng superpowers:brainstorming trước khi sửa behavior.
- Rà các chỗ trùng (Mini App ↔ API ↔ BO ↔ WebAPI job) và hỏi trước khi sửa đồng bộ.
- Chạy RED/GREEN hoặc static regression check.
- Không tự build; đưa tôi lệnh build và chờ kết quả trước commit/push.
- Thay đổi DB: viết script SQL, tôi sẽ tự chạy.
- Thêm trace comment cho bugfix theo format TicketId | branch | commit hash.
- Commit/push branch fix, không push develop/main.
- Update worklog local, không commit worklog/Excel nếu tôi không yêu cầu.

<nội dung ticket>
```

## Prompt ngắn cho người đã quen

```text
/tinhlm-skill

Repo: <repo>
Ticket: <TicketId> — <nội dung>
Branch owner: <owner-suffix>

Tuân thủ LIXIL workflow: đúng base branch, one ticket/branch, brainstorming trước khi sửa, hỏi nếu thiếu expected behavior, rà duplicate, RED/GREEN hoặc static check, tôi build, SQL tôi chạy, trace comment, push fix branch, update worklog local.
```

## Prompt cực ngắn cho session đã có context

```text
/tinhlm-skill Làm issue tiếp theo theo state/progress hiện tại.
```

Không nên dùng ở session mới vì agent có thể thiếu repo, nguồn issue hoặc `{Owner}`.

## Quay lại correction ticket đã done

```text
/tinhlm-skill

Repo: <repo>
Ticket: <TicketId>
Branch: fix/<short-desc>-<TicketId>-<owner-suffix>

Đây là correction cho ticket đã done.
Đọc lại issue note, commits cũ, worklog nếu cần.
Phân tích bằng superpowers:brainstorming trước khi sửa.
Không chọn issue tiếp theo.
```

## Việc agent sẽ làm

1. Đọc state/progress và issue note nếu có.
2. Đọc nội dung ticket hoặc row trong CSV/XLSX.
3. Xác định repo; kiểm tra git status, staged files và worktree.
4. Sync base branch của repo (`develop` hoặc `main`).
5. Tạo/switch branch `fix/{ShortDesc}-{TicketId}-{Owner}` hoặc `fix/{ShortDesc}-{Owner}`.
6. Dùng `superpowers:brainstorming` để phân tích yêu cầu.
7. Hỏi lại nếu thiếu repo, màn hình, field, API contract hoặc expected behavior.
8. Chạy RED/GREEN hoặc static regression check.
9. Sửa code trong scope nhỏ nhất; thay đổi DB thì viết script SQL.
10. Thêm trace comment cho bugfix nếu file hỗ trợ comment.
11. Đưa lệnh build cho người dùng và chờ kết quả.
12. Commit theo `references/commit-prompt.md`.
13. Push branch fix lên origin (không push develop/main, không push remote `source`).
14. Update worklog local và không commit worklog/Excel nếu user không yêu cầu.

## Trace comment

```csharp
// TKT-20261014-00007 | fix/shopCancelOrder-TKT-20261014-00007-tinhlm | d6634849
// Block duplicate ShopCancelOrder requests so one order gets only one return order.
```

Vì commit hash không thể nằm trong chính commit tạo ra nó, agent có thể tạo thêm một commit trace/comment sau khi đã có hash fix commit.

## Prompt không nên dùng

```text
làm tiếp đi
fix bug này
```

Nếu cần dùng prompt ngắn, hãy đảm bảo session đã có sẵn repo, ticket, owner và issue đang bàn.

## Troubleshooting

- Agent hỏi `Branch owner suffix` → trả lời tên owner, ví dụ `tinhlm`.
- CSV không đọc được tiếng Việt → kiểm tra encoding hoặc export lại CSV UTF-8.
- Chỉ có link ERP → dán nội dung ticket và ảnh chụp; agent không truy cập được ERP.
- Issue có nhiều view/API/repo giống nhau → agent liệt kê candidate và hỏi trước khi sửa đồng bộ.
- Không có `superpowers:brainstorming` → agent hỏi user muốn cài skill hay dùng fallback checklist cho session đó.
