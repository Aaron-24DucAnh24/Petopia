# Petopia

Umbrella repo; code nằm trong các submodule `backend/`, `frontend/`, `chat/` (xem [README.md](README.md)).

## Workflow bắt buộc

Mọi yêu cầu làm tính năng / sửa lỗi / chỉnh sửa đều theo skill [task-workflow](.claude/skills/task-workflow/SKILL.md):

1. Tạo `tasks/PR-X.md` (X tăng dần từ 1): tên task, mô tả chi tiết, testcase.
2. Code, rồi cập nhật file md (testcase đã hoàn thành, kết quả).
3. Commit: `[<type>] PR-X: <tóm tắt>` (type: feat, fix, chore, refactor, docs, test, ci...).
4. Tạo PR tên `PR-X: <tên task>`, không có description.
