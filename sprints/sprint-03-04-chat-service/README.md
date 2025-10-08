# Sprint 3-4: Chat Service

**Mục tiêu:** Xây dựng chức năng chat cốt lõi, bao gồm lưu trữ tin nhắn, API REST cho lịch sử, WebSocket cho thời gian thực và phát hành các sự kiện domain.

**Giải thích cho Junior:**

*   **Event-Driven Architecture (Kiến trúc hướng sự kiện):** Các microservice thường giao tiếp với nhau bằng cách phát ra và lắng nghe "sự kiện". Ví dụ, khi một tin nhắn chat được tạo, `chat-service` sẽ phát ra sự kiện `ChatMessageCreated`. Các dịch vụ khác quan tâm có thể lắng nghe sự kiện này để thực hiện các hành động của riêng chúng (ví dụ: `search-service` có thể lập chỉ mục tin nhắn để tìm kiếm). Kafka là một "event broker" phổ biến cho việc này.
*   **WebSocket:** Đây là một giao thức cho phép giao tiếp hai chiều, liên tục giữa client (trình duyệt) và server. Rất lý tưởng cho các ứng dụng thời gian thực như chat, nơi tin nhắn cần được gửi và nhận ngay lập tức mà không cần client phải liên tục hỏi server.
*   **Flyway:** Một công cụ quản lý phiên bản cơ sở dữ liệu. Nó giúp bạn theo dõi và áp dụng các thay đổi cấu trúc cơ sở dữ liệu (schema) một cách có kiểm soát.
*   **Testcontainers:** Một thư viện tuyệt vời cho phép bạn chạy các dịch vụ cơ sở dữ liệu, Kafka... trong các container Docker tạm thời trong quá trình chạy kiểm thử tích hợp. Điều này đảm bảo môi trường test của bạn luôn sạch sẽ và giống với môi trường thật.

**Kế hoạch chi tiết (có hướng dẫn):**

### Tuần 3: Lưu trữ và API

1.  **Task 3.1: Tạo dự án `chat-service`:**
    *   **Mục tiêu:** Khởi tạo microservice chat.
    *   **Thực hiện:**
        *   Sử dụng `start.spring.io`.
        *   **Dependencies:** `Web`, `Security`, `Validation`, `Actuator`, `Data JPA`, `PostgreSQL Driver`, `Flyway`, `Lombok`, `spring-kafka`, `WebSocket`.
    *   **Lời khuyên:** Luôn đảm bảo các dependencies cần thiết được thêm vào từ đầu để tránh phải thêm lại sau này.

2.  **Task 3.2: Thiết kế và di chuyển cơ sở dữ liệu:**
    *   **Mục tiêu:** Tạo bảng để lưu trữ tin nhắn chat.
    *   **Thực hiện:**
        *   Thêm một dịch vụ PostgreSQL mới (`postgres-chat-db`) vào `docker-compose.dev.yml`.
        *   Trong `chat-service`, tạo một script Flyway migration (`src/main/resources/db/migration/V1__create_message_table.sql`) để định nghĩa bảng `messages` (ví dụ: `id`, `room_id`, `sender_id`, `content`, `created_at`).
        *   Cấu hình `application.yml` để kết nối đến `postgres-chat-db`.
    *   **Lời khuyên:** Flyway sẽ tự động chạy các script migration khi ứng dụng khởi động. Hãy kiểm tra log để đảm bảo migration thành công.

3.  **Task 3.3: Xây dựng API REST:**
    *   **Mục tiêu:** Cung cấp các endpoint để gửi và lấy tin nhắn.
    *   **Thực hiện:**
        *   Implement `POST /api/messages` để tạo và lưu tin nhắn mới.
        *   Implement `GET /api/rooms/{roomId}/messages` để lấy lịch sử tin nhắn cho một phòng chat.
    *   **Lời khuyên:** Sử dụng các nguyên tắc RESTful API. Đảm bảo các endpoint được bảo mật bằng Spring Security.

4.  **Task 3.4: Phát hành sự kiện `ChatMessageCreated`:**
    *   **Mục tiêu:** Thông báo cho các dịch vụ khác biết khi có tin nhắn mới.
    *   **Thực hiện:**
        *   Sau khi lưu tin nhắn thành công trong endpoint `POST`, sử dụng `KafkaTemplate` để phát hành một sự kiện `ChatMessageCreated` đến một topic Kafka (ví dụ: `chat.messages.created`).
        *   **Định nghĩa đối tượng sự kiện:** Tạo một Java Record hoặc POJO cho `ChatMessageCreated` (ví dụ: chứa `messageId`, `roomId`, `senderId`, `content`).
    *   **Lời khuyên:** Việc phát hành sự kiện giúp các dịch vụ khác có thể phản ứng mà không cần `chat-service` phải biết về chúng, tăng tính độc lập.

### Tuần 4: Thời gian thực và Kiểm thử

1.  **Task 4.1: Implement WebSocket Handler:**
    *   **Mục tiêu:** Gửi tin nhắn mới đến các client đang kết nối theo thời gian thực.
    *   **Thực hiện:**
        *   Tạo một `WebSocketHandler` để quản lý các kết nối client.
        *   Tạo một Kafka Consumer (`@KafkaListener`) lắng nghe topic `chat.messages.created`.
        *   Khi consumer nhận được một tin nhắn từ Kafka, nó sẽ tìm các phiên WebSocket phù hợp cho `roomId` đó và đẩy nội dung tin nhắn đến các client đang kết nối.
    *   **Lời khuyên:** WebSocket có thể phức tạp. Bắt đầu với một ví dụ đơn giản để hiểu cách client và server giao tiếp.

2.  **Task 4.2: Viết kiểm thử tích hợp với Testcontainers:**
    *   **Mục tiêu:** Đảm bảo `chat-service` hoạt động đúng với các dịch vụ bên ngoài (database, Kafka).
    *   **Thực hiện:**
        *   Tạo một kiểm thử tích hợp cho `chat-service`.
        *   Sử dụng `@Testcontainers` để khởi động các container PostgreSQL và Kafka tạm thời cho môi trường test.
        *   Viết một test case:
            1.  Gọi `POST /api/messages`.
            2.  Xác minh tin nhắn được lưu đúng trong database test.
            3.  Xác minh sự kiện `ChatMessageCreated` được phát hành đến topic Kafka test.
    *   **Lời khuyên:** Kiểm thử tích hợp rất quan trọng trong microservice. Testcontainers giúp bạn có một môi trường test đáng tin cậy, không phụ thuộc vào các dịch vụ đang chạy trên máy cục bộ.
