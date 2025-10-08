# Mô tả các trang Frontend cho StudyHub

Dựa trên kiến trúc và các bản spec, đây là mô tả chi tiết về các trang và chức năng mà frontend của ứng dụng StudyHub sẽ có.

## Tổng quan
Frontend của StudyHub sẽ là một **Progressive Web App (PWA)**, cho phép người dùng có trải nghiệm giống như ứng dụng gốc và có khả năng hoạt động offline. Giao diện sẽ được thiết kế để hỗ trợ cộng tác thời gian thực một cách liền mạch.

---

### 1. Trang Đăng nhập / Đăng ký (Login / Registration Page)

Đây là trang đầu tiên người dùng nhìn thấy. Nó đóng vai trò là cổng vào của hệ thống.

**Chức năng:**
*   **Đăng nhập bằng tài khoản xã hội:** Cho phép người dùng đăng nhập nhanh chóng bằng tài khoản Google hoặc LinkedIn (theo `spec.md` và `target-architecture.md`). Đây là luồng chính được ưu tiên.
*   **Đăng nhập bằng Email/Mật khẩu:** Form đăng nhập truyền thống.
*   **Đăng ký tài khoản mới:** Form để người dùng mới tạo tài khoản.
*   **Bảo mật nâng cao:** Hỗ trợ đăng nhập bằng Passkeys/WebAuthn để tăng cường bảo mật.
*   **Quên mật khẩu:** Chức năng cho phép người dùng lấy lại mật khẩu.

---

### 2. Trang chính / Bảng điều khiển (Dashboard / Main Page)

Sau khi đăng nhập, người dùng sẽ được chuyển đến trang này. Đây là trung tâm điều hướng chính.

**Chức năng:**
*   **Danh sách các Phòng học (Study Rooms):** Hiển thị các phòng học mà người dùng đã tham gia. Mỗi phòng có thể hiển thị thông tin như tên phòng, số lượng thành viên online.
*   **Tạo phòng học mới:** Nút hoặc form cho phép người dùng tạo một phòng học mới, đặt tên và mời thành viên.
*   **Tham gia phòng học:** Chức năng cho phép người dùng tham gia một phòng học đã có thông qua mã mời hoặc lời mời.
*   **Thanh tìm kiếm toàn cục:** Một thanh tìm kiếm nổi bật cho phép người dùng tìm kiếm nội dung trên toàn bộ nền tảng (theo `spec.md` và `search-service`).
*   **Thông báo:** Hiển thị các thông báo gần đây (ví dụ: có tin nhắn mới, có người mời vào phòng, một file đã xử lý xong).
*   **Điều hướng đến Hồ sơ cá nhân:** Link hoặc avatar người dùng để truy cập trang cá nhân.

---

### 3. Trang Phòng học (Study Room Page)

Đây là không gian cộng tác chính và là trang phức tạp nhất của ứng dụng. Giao diện có thể được chia thành nhiều panel (khung) có thể tùy chỉnh.

**Chức năng chính:**

*   **Panel Chat thời gian thực:**
    *   Hiển thị lịch sử tin nhắn của phòng (`chat-service`).
    *   Ô nhập liệu để gửi tin nhắn mới. Tin nhắn mới sẽ xuất hiện ngay lập tức cho tất cả thành viên nhờ **WebSocket**.
    *   Hiển thị trạng thái của thành viên (online, offline).

*   **Panel Gọi Video/Thoại:**
    *   Hiển thị video của các thành viên đang tham gia cuộc gọi.
    *   Các nút điều khiển: bật/tắt micro, bật/tắt camera, chia sẻ màn hình.
    *   Chức năng **Whiteboard** (bảng trắng) để vẽ và ghi chú chung trong thời gian thực.

*   **Panel Tài liệu và Media:**
    *   Hiển thị danh sách các tài liệu (PDF, PowerPoint) và video đã được tải lên trong phòng.
    *   Chức năng **tải lên file** (`media-service`). Khi tải lên, file sẽ được xử lý ở backend (ví dụ: video được chuyển mã).
    *   Cho phép xem video đã xử lý (streaming HLS/DASH).
    *   Cho phép mở và xem tài liệu.
    *   Hỗ trợ **soạn thảo tài liệu chung** theo thời gian thực (sử dụng công nghệ CRDT/Y.js).

*   **Panel Trợ lý AI (AI Assistant):**
    *   Giao diện chat với AI.
    *   Người dùng có thể chọn một tài liệu và **đặt câu hỏi** về nội dung của nó.
    *   Yêu cầu AI **tóm tắt nội dung video** bài giảng (sau khi backend đã xử lý Speech-to-Text).
    *   Yêu cầu AI **tạo câu đố (quiz)** từ một tài liệu hoặc video.
    *   Chức năng **Text-to-Speech** để đọc các ghi chú hoặc tài liệu.

---

### 4. Trang Kết quả Tìm kiếm (Search Results Page)

Trang này hiển thị kết quả từ thanh tìm kiếm toàn cục.

**Chức năng:**
*   Hiển thị danh sách kết quả tìm kiếm được tổng hợp từ nhiều nguồn:
    *   Lịch sử chat.
    *   Nội dung tài liệu đã được lập chỉ mục.
    *   Bản ghi văn bản (transcript) của video.
*   Mỗi kết quả sẽ có tiêu đề, một đoạn trích (snippet) và liên kết trực tiếp đến nội dung gốc (ví dụ: link đến đúng tin nhắn trong phòng chat, hoặc đến đúng đoạn video).
*   Bộ lọc để thu hẹp kết quả theo loại (tin nhắn, tài liệu, video) hoặc theo ngày tháng.

---

### 5. Trang Hồ sơ Người dùng (User Profile Page)

Trang này cho phép người dùng quản lý thông tin cá nhân và cài đặt.

**Chức năng:**
*   Hiển thị thông tin cá nhân: tên, email, ảnh đại diện.
*   Cho phép **chỉnh sửa thông tin cá nhân** (`user-service`).
*   Quản lý cài đặt tài khoản:
    *   Thay đổi mật khẩu.
    *   Quản lý Passkeys.
    *   Cài đặt thông báo (Push Notifications).
    *   Chọn giao diện (Dark mode/Light mode).
