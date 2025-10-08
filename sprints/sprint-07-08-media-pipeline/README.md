# Sprint 7-8: Media Pipeline

**Mục tiêu:** Xây dựng một pipeline bất đồng bộ, hướng sự kiện để xử lý tải lên và chuyển đổi (transcoding) media.

**Giải thích cho Junior:**

*   **Asynchronous Processing (Xử lý bất đồng bộ):** Khi người dùng tải lên một file video lớn, việc xử lý (chuyển đổi định dạng, nén) có thể mất rất nhiều thời gian. Nếu xử lý đồng bộ, người dùng sẽ phải chờ đợi. Xử lý bất đồng bộ có nghĩa là server chấp nhận file, trả lời ngay lập tức cho người dùng, và sau đó xử lý file ở chế độ nền.
*   **Kafka (Event Broker):** Lại một lần nữa, Kafka được dùng để các dịch vụ giao tiếp. Khi một file media được tải lên, một sự kiện `MediaUploaded` được gửi đến Kafka.
*   **RabbitMQ (Task Queue):** RabbitMQ thường được dùng cho các "tác vụ" cụ thể cần được xử lý bởi một worker. Trong trường hợp này, khi có sự kiện `MediaUploaded`, một "lệnh" (command) để chuyển đổi file sẽ được gửi đến RabbitMQ.
*   **MinIO (S3-compatible Object Storage):** Một giải pháp lưu trữ đối tượng tương thích với Amazon S3. Rất tốt để lưu trữ các file lớn như video, hình ảnh.
*   **Worker:** Một ứng dụng hoặc một phần của dịch vụ chuyên làm các công việc nặng nhọc ở chế độ nền, như chuyển đổi video.

**Kế hoạch chi tiết (có hướng dẫn):**

1.  **Task 7.1: Thiết lập `media-service` và lưu trữ file:**
    *   **Mục tiêu:** Khởi tạo microservice media và đảm bảo có nơi lưu trữ file.
    *   **Thực hiện:**
        *   Tạo một dự án Spring Boot mới: `media-service`.
        *   **Dependencies:** `Web`, `Security`, `Validation`, `Actuator`, `spring-kafka`.
        *   Đảm bảo MinIO đang chạy từ `docker-compose.dev.yml` để hoạt động như một kho lưu trữ đối tượng tương thích S3.
    *   **Lời khuyên:** MinIO cung cấp giao diện web để bạn có thể xem các file đã tải lên.

2.  **Task 7.2: Triển khai API tải lên:**
    *   **Mục tiêu:** Cho phép người dùng tải file lên.
    *   **Thực hiện:**
        *   Tạo một endpoint: `POST /api/media/upload`.
        *   Endpoint này chấp nhận một file (ví dụ: `multipart/form-data`).
        *   **Hành động:** Ngay lập tức tải file thô lên một bucket "raw-media" trong MinIO.
        *   Sau khi tải lên thành công, phát hành một sự kiện `MediaUploaded` đến một topic Kafka (ví dụ: `media.uploads`). Sự kiện này nên chứa metadata như tên file, vị trí trong MinIO và người dùng đã tải lên.
        *   API sau đó trả về phản hồi `202 Accepted` cho client với một ID công việc hoặc ID nội dung.
    *   **Lời khuyên:** Trả về `202 Accepted` là một mẫu thiết kế tốt cho các tác vụ bất đồng bộ, cho biết yêu cầu đã được chấp nhận và đang được xử lý.

3.  **Task 7.3: Tạo Media Worker (RabbitMQ Consumer):**
    *   **Mục tiêu:** Có một thành phần sẵn sàng xử lý các tác vụ chuyển đổi media.
    *   **Thực hiện:**
        *   Đây có thể là một module/dự án mới hoặc một thành phần trong `media-service` được cấu hình để chạy riêng.
        *   **Dependencies:** `spring-rabbit`, các thư viện cho xử lý media (ví dụ: một wrapper Java cho `ffmpeg`).
        *   Tạo một RabbitMQ consumer lắng nghe một hàng đợi tác vụ cụ thể (ví dụ: `media.transcoding.tasks`).
    *   **Lời khuyên:** `ffmpeg` là một công cụ mạnh mẽ để xử lý video. Bạn có thể gọi nó từ Java bằng cách chạy các lệnh shell.

4.  **Task 7.4: Điều phối luồng:**
    *   **Mục tiêu:** Kết nối sự kiện tải lên với tác vụ chuyển đổi.
    *   **Thực hiện:**
        *   Tạo một Kafka consumer (có thể nằm trong một `orchestration-service` mới hoặc trong `media-service`) lắng nghe topic `media.uploads`.
        *   Khi nó nhận được sự kiện `MediaUploaded`, nó sẽ gửi một **lệnh** (một tác vụ cụ thể) đến hàng đợi RabbitMQ `media.transcoding.tasks`. Tin nhắn lệnh chứa các chi tiết cần thiết để thực hiện việc chuyển đổi.
    *   **Lời khuyên:** Đây là nơi bạn kết nối kiến trúc hướng sự kiện với kiến trúc hàng đợi tác vụ.

5.  **Task 7.5: Triển khai logic chuyển đổi:**
    *   **Mục tiêu:** Thực hiện việc chuyển đổi file media.
    *   **Thực hiện:**
        *   `media-worker` nhận tác vụ chuyển đổi từ RabbitMQ.
        *   Nó tải file thô từ MinIO.
        *   Nó thực thi `ffmpeg` (hoặc công cụ khác) để xử lý file (ví dụ: thay đổi định dạng, giảm độ phân giải).
        *   Nó tải file đã xử lý lên một bucket "processed-media" trong MinIO.
        *   Cuối cùng, worker phát hành một sự kiện `MediaTranscoded` đến một topic Kafka mới (ví dụ: `media.transcodes`) để thông báo cho các hệ thống khác rằng quá trình xử lý đã hoàn tất.
    *   **Lời khuyên:** Xử lý lỗi là rất quan trọng ở đây. Nếu `ffmpeg` thất bại, bạn cần ghi log lỗi và có thể gửi lại tác vụ hoặc thông báo cho người dùng.

6.  **Task 7.6: Triển khai các mẫu độ tin cậy:**
    *   **Mục tiêu:** Đảm bảo hệ thống xử lý lỗi một cách mạnh mẽ.
    *   **Thực hiện:**
        *   Cấu hình Dead-Letter Queue (DLQ) cho RabbitMQ consumer để xử lý các tác vụ bị lỗi lặp đi lặp lại.
        *   Đảm bảo logic consumer là idempotent (xử lý lại cùng một tác vụ không gây ra tác dụng phụ không mong muốn).
    *   **Lời khuyên:** DLQ là nơi các tin nhắn không thể xử lý được gửi đến, giúp bạn kiểm tra và sửa lỗi sau này. Idempotency là khả năng xử lý một yêu cầu nhiều lần mà không gây ra tác dụng phụ không mong muốn.
