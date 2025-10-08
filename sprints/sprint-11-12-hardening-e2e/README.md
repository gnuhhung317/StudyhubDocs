# Sprint 11-12: Hardening & E2E Testing

**Mục tiêu:** Cải thiện khả năng phục hồi, khả năng quan sát và độ tin cậy tổng thể của hệ thống. Xác minh các luồng người dùng chính bằng kiểm thử end-to-end.

**Giải thích cho Junior:**

*   **Resilience (Khả năng phục hồi):** Khả năng của hệ thống để tiếp tục hoạt động ngay cả khi một số thành phần bị lỗi. Trong microservice, một dịch vụ có thể gọi nhiều dịch vụ khác, và nếu một dịch vụ con bị lỗi, nó không nên làm sập toàn bộ hệ thống.
*   **Circuit Breaker (Cầu dao ngắt mạch):** Một mẫu thiết kế để ngăn chặn một dịch vụ liên tục gọi một dịch vụ khác đang bị lỗi. Nếu một dịch vụ con bị lỗi nhiều lần, circuit breaker sẽ "ngắt mạch" và ngăn các cuộc gọi tiếp theo trong một thời gian, cho phép dịch vụ con có thời gian phục hồi.
*   **Retry (Thử lại):** Khi một yêu cầu đến một dịch vụ khác bị lỗi tạm thời (ví dụ: lỗi mạng), hệ thống có thể thử lại yêu cầu đó một vài lần.
*   **Rate Limiter (Giới hạn tốc độ):** Ngăn chặn một client hoặc người dùng gửi quá nhiều yêu cầu trong một khoảng thời gian nhất định, bảo vệ dịch vụ khỏi bị quá tải hoặc tấn công.
*   **Distributed Tracing (Theo dõi phân tán):** Như đã đề cập, nó giúp bạn theo dõi một yêu cầu đi qua nhiều microservice.
*   **End-to-End (E2E) Testing:** Kiểm thử toàn bộ luồng người dùng từ đầu đến cuối, bao gồm tất cả các dịch vụ và giao diện người dùng. Nó mô phỏng cách người dùng thực sự tương tác với ứng dụng.
*   **Helm Charts:** Một công cụ quản lý gói cho Kubernetes. Nó giúp bạn định nghĩa, cài đặt và nâng cấp các ứng dụng chạy trên Kubernetes một cách dễ dàng.
*   **Runbooks:** Tài liệu hướng dẫn từng bước để giải quyết các vấn đề vận hành phổ biến.

**Kế hoạch chi tiết (có hướng dẫn):**

1.  **Task 11.1: Triển khai các mẫu phục hồi với Resilience4j:**
    *   **Mục tiêu:** Làm cho các microservice của bạn mạnh mẽ hơn khi gặp lỗi.
    *   **Thực hiện:**
        *   **Circuit Breaker:** Áp dụng cho các giao tiếp giữa các dịch vụ (ví dụ: sử dụng Feign client hoặc `RestTemplate`). Ví dụ, nếu `chat-service` không thể kết nối đến `user-service`, nó nên "ngắt mạch" và thất bại nhanh chóng thay vì chờ timeout.
        *   **Retry:** Áp dụng cho các RabbitMQ consumer trong `media-worker`. Nếu quá trình xử lý thất bại do lỗi tạm thời (ví dụ: lỗi mạng), nó nên thử lại một vài lần trước khi gửi tin nhắn đến DLQ.
        *   **Rate Limiter:** Áp dụng ở cấp độ `api-gateway` để ngăn chặn lạm dụng và bảo vệ các dịch vụ hạ nguồn khỏi lưu lượng truy cập đột biến.
    *   **Lời khuyên:** Resilience4j là một thư viện tuyệt vời cho các mẫu phục hồi. Hãy đọc tài liệu của nó và bắt đầu với các ví dụ đơn giản.

2.  **Task 11.2: Nâng cao khả năng quan sát:**
    *   **Mục tiêu:** Dễ dàng theo dõi và gỡ lỗi hệ thống.
    *   **Thực hiện:**
        *   **Distributed Tracing:** Xác minh rằng các ID theo dõi (`traceId`) và ID span (`spanId`) đang được truyền qua tất cả các dịch vụ, từ `api-gateway` qua các tin nhắn Kafka/RabbitMQ đến consumer cuối cùng. Sử dụng Zipkin hoặc Jaeger để hình dung toàn bộ luồng yêu cầu của người dùng.
        *   **Metrics:** Đảm bảo tất cả các dịch vụ hiển thị các chỉ số chính thông qua Spring Boot Actuator / Micrometer. Kiểm tra xem Prometheus có đang thu thập các chỉ số này không.
    *   **Lời khuyên:** Sử dụng các công cụ như Zipkin/Jaeger để trực quan hóa các trace. Điều này giúp bạn hiểu rõ hơn về cách các dịch vụ tương tác và nơi xảy ra tắc nghẽn hoặc lỗi.

3.  **Task 11.3: Viết kiểm thử End-to-End (E2E):**
    *   **Mục tiêu:** Xác minh các luồng người dùng quan trọng hoạt động đúng trên toàn bộ hệ thống.
    *   **Thực hiện:**
        *   Sử dụng một công cụ như **Postman (Newman)** hoặc một script đơn giản.
        *   Định nghĩa các bộ kiểm thử cho các luồng người dùng quan trọng:
            1.  **Luồng Chat:** Người dùng A đăng nhập -> lấy token -> gửi tin nhắn -> Người dùng B đăng nhập -> lấy token -> nhận tin nhắn.
            2.  **Luồng Media:** Người dùng A đăng nhập -> tải lên một file media -> thăm dò trạng thái xử lý -> cuối cùng xác nhận sự kiện `MediaTranscoded` đã được tạo ra.
    *   **Lời khuyên:** Kiểm thử E2E có thể tốn thời gian và phức tạp, nhưng chúng rất quan trọng để đảm bảo toàn bộ hệ thống hoạt động như mong đợi.

4.  **Task 11.4: Tạo Helm Charts cơ bản:**
    *   **Mục tiêu:** Chuẩn bị cho việc triển khai lên Kubernetes.
    *   **Thực hiện:**
        *   Đối với ít nhất hai dịch vụ (ví dụ: `api-gateway`, `chat-service`), tạo một Helm chart cơ bản.
        *   Chart nên tạo các đối tượng Kubernetes `Deployment` và `Service`.
        *   Nó nên cho phép cấu hình tag image, số lượng replica và các biến môi trường chính từ file `values.yaml`.
    *   **Lời khuyên:** Helm là một công cụ mạnh mẽ cho Kubernetes. Bắt đầu với một chart đơn giản và dần dần thêm các cấu hình phức tạp hơn.

5.  **Task 11.5: Viết Runbooks ban đầu:**
    *   **Mục tiêu:** Tạo tài liệu hướng dẫn vận hành cơ bản.
    *   **Thực hiện:**
        *   Trong thư mục `docs/runbook` của kho mã nguồn, tạo các file markdown đơn giản cho các tác vụ vận hành phổ biến:
            *   `how-to-restart-a-service.md`
            *   `what-to-do-when-kafka-consumer-lag-is-high.md`
    *   **Lời khuyên:** Runbooks là tài liệu sống. Chúng nên được cập nhật thường xuyên khi bạn học được cách vận hành hệ thống tốt hơn.
