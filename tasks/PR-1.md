# PR-1: Thiết lập task workflow

**Trạng thái:** Done
**Phạm vi:** umbrella

## Mô tả chi tiết

Chuẩn hóa quy trình làm task trong repo Petopia để mọi thay đổi đều có tài liệu đi kèm và dễ theo dõi:

- Thêm thư mục `tasks/`. Mỗi yêu cầu (tính năng, sửa lỗi, chỉnh sửa...) có một file `tasks/PR-X.md`, X tăng dần từ 1, gồm tên task, mô tả chi tiết, testcase và các case đã hoàn thành.
- Tạo file task trước khi code, cập nhật file sau khi code xong.
- Commit message: `[<type>] PR-X: <tóm tắt thay đổi>`.
- PR tên `PR-X: <tên task>`, không có description.

Hướng tiếp cận:

- Skill `.claude/skills/task-workflow/` (SKILL.md + template.md) mô tả chi tiết các bước, gồm cả cách xử lý khi code nằm trong submodule.
- `CLAUDE.md` ở root tóm tắt quy tắc và trỏ tới skill, để workflow được nạp vào mọi session.

## Testcase

- [x] TC1: Mở session mới trong repo → skill `task-workflow` xuất hiện trong danh sách skill khả dụng.
  - Kiểm chứng: skill được Claude Code nhận ngay sau khi tạo file, hiển thị trong danh sách skill.
- [x] TC2: `CLAUDE.md` ở root có mục "Workflow bắt buộc" trỏ tới skill → quy tắc được nạp mỗi session.
  - Kiểm chứng: đọc lại nội dung `CLAUDE.md`.
- [x] TC3: Chưa có file `tasks/PR-*.md` và branch `pr-*` nào → task đầu tiên được đánh số PR-1.
  - Kiểm chứng: `git branch -a --list '*pr-*'` không trả về branch nào, `tasks/` chưa có file task.
- [x] TC4: Mọi commit trên branch `pr-1/task-workflow` có dạng `[<type>] PR-1: <tóm tắt>`.
  - Kiểm chứng: `git log --oneline main..pr-1/task-workflow`.
- [x] TC5: PR được tạo với tiêu đề `PR-1: Thiết lập task workflow` và description trống.
  - Kiểm chứng: `gh pr view 1 --json title,body` trả về `body: ""`.

## Kết quả

- Thêm skill [task-workflow](../.claude/skills/task-workflow/SKILL.md) và [template](../.claude/skills/task-workflow/template.md) cho file task.
- Thêm [CLAUDE.md](../CLAUDE.md) tóm tắt workflow bắt buộc.
- Thêm thư mục `tasks/`, bắt đầu với file này.
- PR: https://github.com/Aaron-24DucAnh24/Petopia/pull/1
