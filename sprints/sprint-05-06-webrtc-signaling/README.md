# Sprint 5-6: Real-time/WebRTC Signaling

**Mục tiêu:** Triển khai máy chủ báo hiệu cần thiết để các client thiết lập kết nối WebRTC ngang hàng (peer-to-peer) cho cuộc gọi thoại/video.

**Giải thích cho Junior:**

*   **WebRTC (Web Real-Time Communication):** Một công nghệ cho phép giao tiếp thời gian thực trực tiếp giữa các trình duyệt hoặc thiết bị, bao gồm thoại, video và chia sẻ dữ liệu.
*   **Signaling Server (Máy chủ báo hiệu):** WebRTC là peer-to-peer, nhưng để các "peer" (người tham gia) tìm thấy và kết nối với nhau, họ cần một "người mai mối" – đó là signaling server. Signaling server giúp trao đổi thông tin cần thiết để thiết lập kết nối (như địa chỉ IP, thông tin về khả năng nghe/nói của thiết bị). **Quan trọng:** Signaling server không xử lý luồng dữ liệu thoại/video thực tế, nó chỉ giúp thiết lập kết nối ban đầu.
*   **LiveKit:** Một nền tảng WebRTC mã nguồn mở, giúp đơn giản hóa việc xây dựng các ứng dụng thời gian thực. Sử dụng một giải pháp có sẵn như LiveKit giúp bạn tiết kiệm rất nhiều công sức so với việc tự xây dựng signaling server từ đầu.
*   **Backend-for-Frontend (BFF):** `realtime-service` ở đây đóng vai trò là BFF. Thay vì client gọi trực tiếp LiveKit, nó gọi `realtime-service`. BFF có thể thêm logic nghiệp vụ, bảo mật, hoặc chuyển đổi dữ liệu trước khi gọi dịch vụ bên thứ ba.

**Kế hoạch chi tiết (có hướng dẫn):**

1.  **Task 5.1: Chọn và thiết lập giải pháp báo hiệu:**
    *   **Mục tiêu:** Có một signaling server hoạt động.
    *   **Thực hiện:**
        *   **Khuyến nghị cho MVP:** Tích hợp một giải pháp như **LiveKit**.
        *   Thêm LiveKit vào `docker-compose.dev.yml` nếu bạn tự host.
    *   **Lời khuyên:** Đọc tài liệu của LiveKit để hiểu cách nó hoạt động và cách cấu hình.

2.  **Task 5.2: Tạo `realtime-service`:**
    *   **Mục tiêu:** Tạo microservice đóng vai trò là BFF cho LiveKit.
    *   **Thực hiện:**
        *   Tạo một dự án Spring Boot mới.
        *   **Dependencies:** `Web`, `Security`, `Validation`, `Actuator`.
    *   **Lời khuyên:** Dịch vụ này sẽ là cầu nối giữa client và LiveKit.

3.  **Task 5.3: Triển khai quản lý phòng và tạo token:**
    *   **Mục tiêu:** Cho phép người dùng tham gia phòng và nhận token để kết nối với LiveKit.
    *   **Thực hiện:**
        *   Tạo một endpoint (ví dụ: `POST /api/realtime/rooms/{roomId}/join`) được bảo vệ bằng xác thực JWT của Keycloak.
        *   Trong endpoint này, `realtime-service` sẽ giao tiếp với LiveKit server để:
            1.  Xác minh người dùng có quyền tham gia phòng.
            2.  Tạo một access token ngắn hạn dành riêng cho LiveKit.
        *   Dịch vụ sẽ trả về token LiveKit này cho client.
    *   **Lời khuyên:** Việc tạo token ở backend giúp bảo mật hơn, vì client không cần biết khóa API của LiveKit.

4.  **Task 5.4: Tích hợp phía Client (Khái niệm):**
    *   **Mục tiêu:** Hiểu cách client sẽ sử dụng token LiveKit.
    *   **Thực hiện:** (Đây là phần client-side, bạn chỉ cần hiểu khái niệm)
        *   Ứng dụng client (ví dụ: trình duyệt) nhận token LiveKit.
        *   Sau đó, client sử dụng SDK của LiveKit để kết nối trực tiếp với LiveKit server bằng token này.
        *   Từ thời điểm này, LiveKit server sẽ xử lý việc báo hiệu WebRTC (trao đổi `offer`, `answer`, và `ICE candidate` giữa các client trong phòng).
    *   **Lời khuyên:** Mặc dù bạn tập trung vào backend, việc hiểu luồng hoạt động của client giúp bạn thiết kế API backend tốt hơn.

5.  **Task 5.5: Tập trung vào khả năng quan sát (Observability):**
    *   **Mục tiêu:** Dễ dàng theo dõi và gỡ lỗi `realtime-service`.
    *   **Thực hiện:**
        *   Triển khai ghi log có cấu trúc (structured logging) cho tất cả các bước quan trọng trong `realtime-service` (ví dụ: yêu cầu tạo token, lỗi từ LiveKit).
        *   Đảm bảo theo dõi phân tán (distributed tracing) (qua OpenTelemetry/Zipkin) hoạt động để theo dõi các yêu cầu từ API Gateway đến `realtime-service`.
    *   **Lời khuyên:** Observability là cực kỳ quan trọng trong microservice. Khi có lỗi, bạn cần biết yêu cầu đã đi qua những dịch vụ nào và lỗi xảy ra ở đâu.
