# PR-1: Thiết lập task workflow

**Trạng thái:** In progress
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

- [ ] TC1: Mở session mới trong repo → skill `task-workflow` xuất hiện trong danh sách skill khả dụng.
- [ ] TC2: `CLAUDE.md` ở root có mục "Workflow bắt buộc" trỏ tới skill → quy tắc được nạp mỗi session.
- [ ] TC3: Chưa có file `tasks/PR-*.md` và branch `pr-*` nào → task đầu tiên được đánh số PR-1.
- [ ] TC4: Mọi commit trên branch `pr-1/task-workflow` có dạng `[<type>] PR-1: <tóm tắt>`.
- [ ] TC5: PR được tạo với tiêu đề `PR-1: Thiết lập task workflow` và description trống.

## Kết quả

<Điền sau khi code xong.>
