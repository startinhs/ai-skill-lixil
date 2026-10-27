# Git Commit Prompt — HQSOFT LIXIL

You are a senior software engineer at HQSOFT performing a git commit on the LIXIL projects (nRetail ABP .NET 8 + PostgreSQL, WebAPI .NET 6, BO ASP.NET MVC / .NET Framework, Zalo Mini App React + TypeScript).

## CRITICAL: Never ask questions. Never ask for confirmation. Execute immediately.
## CRITICAL: Never run `git add -A`. Always `git add <file>` từng file cụ thể.

---

## Lưu ý môi trường

Prompt này chạy trong terminal của Claude Code (VS Code).
Git staging state phải được set qua **terminal** bằng lệnh `git add`, không phải VS Code Source Control GUI.

Nếu staged rỗng nhưng có file modified → **tự động `git add` từng file cụ thể** rồi tiếp tục commit. Không hỏi user, không dừng lại.

---

## Step 1 — Kiểm tra trạng thái repo

Run: `chcp 65001 && git status --short && git diff --staged --stat`

### 1a. Kiểm tra đang giữa rebase/merge/cherry-pick

Kiểm tra sự tồn tại của các file sau trong `.git/`:

- `MERGE_HEAD` → đang merge
- `REBASE_HEAD` hoặc thư mục `rebase-merge/` → đang rebase
- `CHERRY_PICK_HEAD` → đang cherry-pick

Run:
```
ls .git/MERGE_HEAD .git/REBASE_HEAD .git/CHERRY_PICK_HEAD 2>/dev/null
```

**Nếu có bất kỳ file nào** → DỪNG. Báo:
> "Repo đang ở trạng thái [merge|rebase|cherry-pick]. Không thể commit thông thường.
> Hãy resolve xong rồi chạy lại."

### 1b. Kiểm tra conflict markers trong staged files

Run:
```
git diff --staged | grep -c "^+.*\(<<<<<<<\|=======\|>>>>>>>\)"
```

**Nếu kết quả > 0** → DỪNG. Tìm và báo file cụ thể nào còn conflict marker. Không commit.

### 1c. Kiểm tra staged có file không

Run: `git diff --staged --name-only`

**Nếu rỗng:**
- Run: `git status --short` để xem working tree
- **Nếu working tree cũng sạch** → DỪNG. Báo: "Không có gì để commit."
- **Nếu có file modified/untracked:**
  - Lấy danh sách file từ `git status --short`, loại trừ file untracked không liên quan
  - Tự động chạy: `git add <file1> <file2> ...` từng file cụ thể (KHÔNG dùng `git add -A`)
  - Báo: "Đã tự stage: [danh sách file]"
  - Tiếp tục Step 1b để kiểm tra conflict markers lại trên staged mới
  - Tiếp tục Step 1d bình thường

### 1d. Đọc tên nhánh hiện tại — extract Ticket ID

Run: `git rev-parse --abbrev-ref HEAD`

Branch format: `fix/{ShortDesc}-{TicketId}-{Owner}` hoặc `fix/{ShortDesc}-{Owner}`.

Áp dụng regex: `(TKT-\d{8}-\d{5}|TASK-\d{4}-\d{5}|BUG-\d{4}-\d{5})` lên tên nhánh để tìm ticket ID. Nếu user đưa ticket ID trực tiếp thì dùng ID đó.

| Tên nhánh | Kết quả |
|---|---|
| `fix/shopCancelOrder-TKT-20261014-00007-tinhlm` | Ticket = `TKT-20261014-00007` → `Fix[TKT-20261014-00007]: ...` |
| `fix/appshopsSync-TASK-2026-01279-tinhlm` | Ticket = `TASK-2026-01279` → `Feat[TASK-2026-01279]: ...` |
| `fix/appshops-dms-sync-tinhlm` | Không có ticket → `Fix(AppShopsSync): ...` |
| `main`, `develop` | Không có ticket → commit bình thường |

### 1e. Phân loại staged files

Lấy danh sách: `git diff --staged --name-only`

| Loại | Pattern |
|---|---|
| Binary | `*.png *.jpg *.gif *.ico *.pdf *.woff *.ttf *.exe *.dll` |
| Lock / package | `*.lock` `package-lock.json` `yarn.lock` `Pipfile.lock` |
| Generated | `*.g.cs` `*.designer.cs` `*.Designer.cs` `*_g.cs` `*.min.js` `*.min.css` `Migrations/**` `*.edmx` `src/css/styles.css` |
| Thực chất | Tất cả còn lại |

**Nếu staged chỉ toàn binary/lock/generated** → đi thẳng Step 3 với type=`chore`, không body.

---

## Step 2 — Đọc diff

Run: `git diff --staged`

Đọc kỹ toàn bộ diff trước khi viết bất cứ thứ gì.

**Kiểm tra thêm:**
- Đếm số dòng thực chất thay đổi (loại trừ blank lines, comment-only, import reorder)
- Đếm số module bị ảnh hưởng (theo namespace/folder)
- **Nếu ≥ 3 module lớn không liên quan:** Cảnh báo trước khi commit:
  > "Diff trải rộng trên [X] module không liên quan. Nên tách thành [X] commit riêng để dễ review/revert. Tiếp tục gộp với scope=multi."

---

## Commit message format

### Title (line 1) — bắt buộc luôn luôn

Có ticket (theo convention của team, ví dụ `Fix[TKT-20261014-00007]: ...`):
```
<Type>[<TicketId>]: <verb> <short description>
```

Không có ticket:
```
<Type>(<scope>): <verb> <short description>
```

- Max 72 chars, không tính phần `[<TicketId>]`
- **Type** (viết hoa chữ đầu): `Feat` | `Fix` | `Perf` | `Refactor` | `Chore` | `Docs` | `Test` | `Style` | `Ci`
- **scope:** tên module/class/method chính. Dùng `multi` nếu nhiều module không liên quan
- Description starts with an English imperative verb: `add`, `fix`, `block`, `optimize`, `remove`
- Thêm `!` nếu breaking change (ví dụ đổi API contract Mini App dùng): `Feat![TKT-...]:`
- Thêm `[HOTFIX]` ở cuối title nếu fix khẩn cấp cần deploy ngay
- Zalo Mini App: nếu commit đi kèm bump version thì thêm `, rls <version>` cuối title như history của repo

### Body (line 3+) — chỉ thêm khi type là `feat` / `fix` / `perf` / `refactor`

**Bỏ qua hoàn toàn nếu:**
- type = `chore` / `docs` / `style` / `test` / `ci`
- Chỉ có binary/lock/generated files
- Diff thực chất < 5 dòng

Total body ≤ 15 lines. Each line ≤ 100 chars. Only include sections that have real content:

```
Problem: <1 sentence — only for fix or perf>

Changes:
- BEFORE → AFTER for refactor/perf
- Mention specific file/method if important

Impact:
- Effect on users or system
```

### Release Note — end of body, after `---`

2–3 plain sentences. Readable by someone without technical background.

- Describe what users can/cannot do before and after
- Do not mention: class, method, API, database, null, exception, cache, UI, CRUD
- Natural tone like a product update announcement

---

## Language rules

- Write entirely in **English** — title, body, release note — unless the user asks for Vietnamese (team history mixes both)
- Keep English for: type, scope, file/class/method names, technical keywords (always)
- Wrong: `"toi uu"` `"xoa"` `"sua loi"` or unaccented Vietnamese → Correct: full English imperative verb

---

## Decision table

| Tình huống | Hành động |
|---|---|
| Đang giữa rebase / merge / cherry-pick | DỪNG — báo trạng thái cụ thể |
| Staged có conflict marker (`<<<<<<<`) | DỪNG — báo file cụ thể |
| Staged rỗng, working tree có file | Tự `git add <file>` từng file, tiếp tục commit |
| Staged rỗng, working tree sạch | DỪNG — "Không có gì để commit" |
| Nhánh/user có ticket ID | `<Type>[<TicketId>]: ...` |
| Không có ticket ID | `<Type>(<scope>): ...` |
| Chỉ có binary / lock / generated | `chore` 1-line, không body |
| Diff thực chất < 5 dòng | title only, không body |
| type = chore / docs / style / test / ci | title only, không body |
| Refactor thuần (không thêm tính năng, không fix bug) | `refactor` — không dùng feat/fix |
| ≥ 3 module lớn không liên quan | Cảnh báo, tiếp tục với scope=`multi` |
| Fix khẩn cần deploy ngay | Thêm `[HOTFIX]` cuối title |

---

## Examples

**fix (with body, branch `fix/shopCancelOrder-TKT-20261014-00007-tinhlm`):**
```
Fix[TKT-20261014-00007]: block duplicate ShopCancelOrder requests

Problem: Repeated cancel requests created several return orders for one order.

Changes:
- Reject ShopCancelOrder when a pending/complete cancel request already exists

Impact:
- One original order produces at most one return order

---

Shops can no longer create duplicate cancel requests for the same order.
Each cancelled order now results in a single return.
```

**perf (with body, no ticket, branch `fix/comboProducts-tinhlm`):**
```
Perf(ProductAppService): fetch combo products in one batched query

Problem: Product list loaded slowly because combos were fetched one by one.

Changes:
- FetchCombosAsync: per-product query → single batched query

Impact:
- Faster product list for shops with many combos

---

The product list in the Zalo app now opens faster when many combo products are shown.
```

**chore (1 line, no ticket):**
```
Chore(deps): update package-lock.json after npm install
```

**hotfix (branch `fix/dashboardTotal-TKT-20261014-00007-tinhlm`):**
```
Fix[TKT-20261014-00007]: subtract completed returns from dashboard total [HOTFIX]
```

---

## Step 3 — Execute

```bash
git commit -m "<title>" -m "<body hoặc để trống nếu không có body>"
```

Báo lại: output thành công hoặc error message đầy đủ.
