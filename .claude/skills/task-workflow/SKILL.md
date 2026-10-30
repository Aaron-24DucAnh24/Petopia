---
name: task-workflow
description: Quy trình bắt buộc cho MỌI yêu cầu làm tính năng, sửa lỗi, chỉnh sửa, refactor, cấu hình... trong repo Petopia. Tạo file tasks/PR-X.md (tên task, mô tả, testcase), code, cập nhật file md, commit theo format "[type] PR-X: tóm tắt", rồi tạo PR tên "PR-X: <tên task>" không có description. Dùng trước khi bắt đầu sửa bất kỳ file nào.
---

# Task workflow (PR-X)

Áp dụng cho mọi yêu cầu có thay đổi code/config/docs: tính năng mới, fix bug, chỉnh sửa, refactor, CI... Không áp dụng cho câu hỏi thuần túy (giải thích code, tra cứu) không sửa file.

## 1. Xác định số X

- X = số lớn nhất trong các file `tasks/PR-*.md` hiện có + 1. Chưa có file nào thì X = 1.
- Kiểm tra cả branch remote `pr-*` để tránh trùng số khi có task đang dở:
  ```bash
  ls tasks/PR-*.md 2>/dev/null
  git branch -a --list '*pr-*'
  ```
- Một yêu cầu = một số X. Nếu người dùng yêu cầu tiếp chỉnh sửa cho task đang làm dở (chưa merge), cập nhật file PR-X.md hiện tại thay vì tạo số mới.

## 2. Tạo file task TRƯỚC khi code

1. Từ `main` mới nhất của umbrella repo, tạo branch `pr-X/<slug-ngan>` (ví dụ `pr-3/adoption-filter`).
2. Tạo `tasks/PR-X.md` theo [template.md](template.md), điền:
   - **Tên task**: ngắn gọn, dùng làm tên PR.
   - **Mô tả chi tiết**: bối cảnh, yêu cầu, phạm vi (service nào: backend/frontend/chat), hướng tiếp cận dự kiến.
   - **Testcase**: liệt kê dạng checklist `- [ ]`, mỗi case ghi rõ input/thao tác và kết quả mong đợi. Gồm case chính, case biên, case lỗi.
   - **Trạng thái**: `In progress`.
3. Commit: `[docs] PR-X: add task description`.

## 3. Code

- Code nằm trong submodule (`backend/`, `frontend/`, `chat/`) thì làm việc trong repo của submodule đó:
  - `git -C <submodule> checkout main && git -C <submodule> pull`
  - Tạo branch cùng tên `pr-X/<slug>` trong submodule.
- Commit nhỏ, theo từng bước hợp lý. Mỗi commit (ở cả umbrella lẫn submodule) có message:
  ```
  [<type>] PR-X: <tóm tắt thay đổi>
  ```
  `<type>` ∈ `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `ci`, `perf`, `style`, `build`. Ví dụ: `[feat] PR-3: add species filter to adoption list`.
- Chạy test/build/lint liên quan để kiểm chứng từng testcase.

## 4. Cập nhật file task SAU khi code

Cập nhật `tasks/PR-X.md`:
- Tick `- [x]` các testcase đã pass, kèm cách kiểm chứng (lệnh test, thao tác thủ công...). Case chưa pass/không kiểm được thì để `- [ ]` và ghi lý do.
- Điền mục **Kết quả**: tóm tắt thay đổi, các file/module chính bị ảnh hưởng, link PR ở từng repo.
- Đổi **Trạng thái** thành `Done` (hoặc `Blocked` kèm lý do).
- Commit: `[docs] PR-X: update task result`.

## 5. Tạo PR

- Push branch và tạo PR ở **mỗi repo có thay đổi** (submodule trước, umbrella sau), tiêu đề giống nhau, **không có description**:
  ```bash
  git push -u origin pr-X/<slug>
  gh pr create --base main --title "PR-X: <Tên task trong file md>" --body ""
  ```
  Với submodule: chạy trong thư mục submodule (hoặc thêm `--repo Aaron-24DucAnh24/Petopia-<Service>`).
- PR của umbrella repo chứa file `tasks/PR-X.md`. Không bump con trỏ submodule tới commit trên branch chưa merge; sau khi PR submodule được merge vào `main`, bump bằng commit `[chore] PR-X: bump <submodule>`.
- Báo lại cho người dùng link các PR đã tạo và các testcase chưa hoàn thành (nếu có).
