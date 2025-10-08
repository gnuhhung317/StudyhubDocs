# API Mapping: Frontend Features to Backend Services

Đây là tài liệu mapping các chức năng của frontend StudyHub tới các microservice và API endpoint tương ứng ở backend. Mục đích là để xác định các API cần được triển khai.

## 1. Tổng quan

| Frontend Page/Component | Feature | Required Backend API | Service | Contract Status |
| :--- | :--- | :--- | :--- | :--- |
| `auth-modal.tsx` | Đăng ký người dùng | `POST /auth/register` | `auth-service` | ✅ Đã định nghĩa |
| `auth-modal.tsx` | Đăng nhập người dùng | `POST /auth/login` | `auth-service` | ✅ Đã định nghĩa |
| `auth-modal.tsx` | Đăng nhập qua mạng xã hội | (Luồng OAuth2) | `auth-service` (Keycloak) | ✅ Đã định nghĩa (khái niệm) |
| `user-profile-page.tsx` | Xem thông tin cá nhân | `GET /users/{userId}` | `user-service` | ✅ Đã định nghĩa |
| `user-profile-page.tsx` | Cập nhật thông tin cá nhân | `PUT /users/{userId}` | `user-service` | ✅ Đã định nghĩa |
| `settings-page.tsx` | Cập nhật cài đặt | `PUT /users/{userId}` | `user-service` | ✅ Đã định nghĩa |
| `dashboard-page.tsx` | Lấy danh sách phòng học | `GET /rooms` | `chat-service` | ❌ **GAP** |
| `dashboard-page.tsx` | Tạo phòng học mới | `POST /rooms` | `chat-service` | ❌ **GAP** |
| `dashboard-page.tsx` | Tham gia phòng học | `POST /rooms/{roomId}/join` | `chat-service` | ❌ **GAP** |
| `workspace-page.tsx` | Lấy lịch sử chat | `GET /rooms/{roomId}/messages` | `chat-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | Gửi tin nhắn | `POST /rooms/{roomId}/messages` | `chat-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | Chat thời gian thực | WebSocket connection | `realtime-service` | ✅ Đã định nghĩa (`/ws`) |
| `workspace-page.tsx` | Gọi Video/Voice | WebRTC Signaling | `realtime-service` | ✅ Đã định nghĩa (khái niệm) |
| `workspace-page.tsx` | Tải file lên | `POST /upload` | `media-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | Lấy trạng thái file | `GET /files/{fileId}` | `media-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | Lấy danh sách file trong phòng | `GET /rooms/{roomId}/files` | `media-service` | ❌ **GAP** |
| `workspace-page.tsx` | AI: Tóm tắt video | `POST /summarize/video/{videoId}` | `ai-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | AI: Hỏi đáp tài liệu | `POST /qna/document/{documentId}` | `ai-service` | ✅ Đã định nghĩa |
| `workspace-page.tsx` | AI: Tạo câu đố | `POST /quiz/document/{documentId}` | `ai-service` | ✅ Đã định nghĩa |
| `discover-page.tsx` | Tìm kiếm chung | `GET /search` | `search-service` | ✅ Đã định nghĩa |
| `admin-page.tsx` | Quản lý người dùng | `GET /users`, `DELETE /users/{userId}` | `user-service` | ❌ **GAP** |
| `admin-page.tsx` | Tình trạng hệ thống | Actuator endpoints | All services | ✅ Đã định nghĩa (khái niệm) |

---

## 2. Chi tiết các API còn thiếu (GAP)

Dưới đây là danh sách các API quan trọng cần được định nghĩa trong các file contract OpenAPI (`.yml`) để frontend có thể hoạt động đầy đủ.

### 🔹 Chat Service (`chat-service`)

-   **`GET /api/v1/rooms`**
    -   **Mô tả:** Lấy danh sách các phòng học mà người dùng hiện tại đã tham gia.
    -   **Sử dụng:** `dashboard-page.tsx` để hiển thị danh sách phòng học.
    -   **Response Body (Gợi ý):**
        ```json
        [
          {
            "roomId": "uuid",
            "name": "Machine Learning Fundamentals",
            "onlineMembers": 5,
            "lastActivity": "2025-10-04T10:30:00Z"
          }
        ]
        ```

-   **`POST /api/v1/rooms`**
    -   **Mô tả:** Tạo một phòng học mới.
    -   **Sử dụng:** `dashboard-page.tsx` khi người dùng nhấn nút "Tạo phòng học mới".
    -   **Request Body (Gợi ý):**
        ```json
        {
          "name": "New Study Room",
          "description": "A room for discussing advanced topics.",
          "memberIdsToInvite": ["uuid1", "uuid2"]
        }
        ```

-   **`GET /api/v1/rooms/{roomId}`**
    -   **Mô tả:** Lấy thông tin chi tiết của một phòng học, bao gồm danh sách thành viên.
    -   **Sử dụng:** `workspace-page.tsx` khi vào một phòng cụ thể.

-   **`POST /api/v1/rooms/{roomId}/join`**
    -   **Mô tả:** Cho phép người dùng tham gia một phòng học (ví dụ: qua mã mời).
    -   **Sử dụng:** `dashboard-page.tsx`.

-   **`GET /api/v1/rooms/{roomId}/members`**
    -   **Mô tả:** Lấy danh sách thành viên trong một phòng.
    -   **Sử dụng:** `workspace-page.tsx` để hiển thị danh sách thành viên online.

### 🔹 Media Service (`media-service`)

-   **`GET /api/v1/rooms/{roomId}/files`**
    -   **Mô tả:** Lấy danh sách tất cả các file (tài liệu, video) đã được tải lên trong một phòng học.
    -   **Sử dụng:** `workspace-page.tsx` trong panel "Tài liệu và Media".
    -   **Response Body (Gợi ý):**
        ```json
        [
          {
            "fileId": "uuid",
            "fileName": "neural_networks.pdf",
            "fileType": "document",
            "uploadedBy": "Alice Chen",
            "uploadedAt": "2025-10-03T14:00:00Z",
            "status": "ready",
            "url": "..."
          }
        ]
        ```

### 🔹 User Service (`user-service`)

-   **`GET /api/v1/admin/users`**
    -   **Mô tả:** Lấy danh sách tất cả người dùng với các tùy chọn lọc và phân trang (dành cho admin).
    -   **Sử dụng:** `admin-page.tsx`.

-   **`DELETE /api/v1/admin/users/{userId}`**
    -   **Mô tả:** Xóa hoặc vô hiệu hóa một người dùng (dành cho admin).
    -   **Sử dụng:** `admin-page.tsx`.

-   **`PUT /api/v1/admin/users/{userId}`**
    -   **Mô tả:** Cập nhật thông tin của một người dùng (dành cho admin).
    -   **Sử dụng:** `admin-page.tsx`.
