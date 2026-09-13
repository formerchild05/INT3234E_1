# Hướng dẫn làm việc trong repo

- Trước khi xử lý mỗi prompt, đọc `.agents/KNOWLEDGE.md` để lấy context. Sau khi pull/checkout, mở phiên mới hoặc context bị rút gọn, đọc lại bản trên đĩa. Nếu thiếu file, tạo bản ngắn từ dữ kiện đã xác minh; không suy đoán lịch sử phiên trước.
- Với yêu cầu đọc hiểu, giải thích, phản biện hoặc triển khai một bài báo khoa học, dùng skill `paper-study` tại `.agents/skills/paper-study/SKILL.md`.
- Chỉ đọc thêm phần tài liệu/code liên quan đến câu hỏi; không tự động nạp toàn bộ PDF, repo hay ghi chú cũ. Context là bản tóm tắt có thể lỗi thời, không thay thế bằng chứng gốc hoặc chỉ dẫn mới của người dùng.
- Trước khi kết thúc lượt có kiến thức, quyết định, kết quả kiểm tra hoặc tiến độ mới, cập nhật `.agents/KNOWLEDGE.md`. Không ghi lại hội thoại hay sửa file nếu không có thay đổi có ý nghĩa.
- Giữ `.agents/KNOWLEDGE.md` khoảng 800–1.200 từ, ưu tiên mục tiêu hiện tại, kết luận có nguồn, điểm chưa rõ và bước tiếp theo. Khi dài hơn, chuyển chi tiết cần giữ sang `notes/<paper-id>.md` và để lại liên kết; chỉ đọc ghi chú khi cần.
- Dùng đường dẫn tương đối trong context để chạy được sau clone/pull trên máy khác. Phân biệt kết quả bài báo, suy luận của trợ lý và kết quả chạy code thực tế. Không ghi bí mật hoặc dữ liệu cá nhân không cần thiết.
- Trả lời bằng tiếng Việt, giữ thuật ngữ chuyên ngành tiếng Anh khi hữu ích. Không tự commit/push chỉ để lưu context; các file cần được commit/push cùng repo để đồng bộ sang máy khác.
