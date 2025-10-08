# Sprint 9-10: AI & Search

**Mục tiêu:** Tận dụng media đã xử lý để cung cấp các tính năng hỗ trợ AI (Speech-to-Text) và làm cho nội dung có thể tìm kiếm được.

**Giải thích cho Junior:**

*   **Elasticsearch:** Một công cụ tìm kiếm và phân tích dữ liệu mạnh mẽ. Rất tốt để lập chỉ mục (indexing) và tìm kiếm toàn văn (full-text search) trên một lượng lớn dữ liệu.
*   **Speech-to-Text (STT):** Công nghệ chuyển đổi giọng nói thành văn bản.
*   **Data Enrichment (Làm giàu dữ liệu):** Lấy dữ liệu hiện có (ví dụ: file âm thanh) và thêm thông tin mới vào đó (ví dụ: bản ghi văn bản từ STT).
*   **Dedicated Search Service:** Có một dịch vụ tìm kiếm riêng biệt giúp tập trung logic tìm kiếm, dễ dàng mở rộng và quản lý các chỉ mục Elasticsearch.

**Kế hoạch chi tiết (có hướng dẫn):**

1.  **Task 9.1: Thiết lập Elasticsearch:**
    *   **Mục tiêu:** Có một Elasticsearch server hoạt động.
    *   **Thực hiện:**
        *   Thêm một dịch vụ Elasticsearch vào `docker-compose.dev.yml` của bạn.
    *   **Lời khuyên:** Elasticsearch cần khá nhiều RAM. Đảm bảo máy của bạn có đủ tài nguyên.

2.  **Task 9.2: Tạo `ai-service`:**
    *   **Mục tiêu:** Tạo microservice chuyên xử lý các tác vụ AI.
    *   **Thực hiện:**
        *   Tạo một dự án Spring Boot mới.
        *   **Dependencies:** `Web`, `Security`, `Actuator`, `spring-kafka`.
    *   **Lời khuyên:** Dịch vụ này sẽ là nơi bạn tích hợp với các API AI bên ngoài.

3.  **Task 9.3: Triển khai Consumer Speech-to-Text (STT):**
    *   **Mục tiêu:** Tự động chuyển đổi âm thanh thành văn bản khi có file media mới được xử lý.
    *   **Thực hiện:**
        *   Tạo một Kafka consumer trong `ai-service` đăng ký topic `media.transcodes`.
        *   Khi nó nhận được một sự kiện cho một file âm thanh đã được xử lý, nó sẽ:
            1.  Tải file âm thanh từ MinIO.
            2.  Gọi một dịch vụ STT bên ngoài (ví dụ: OpenAI's Whisper API, Google Speech-to-Text) với dữ liệu âm thanh.
            3.  Nhận văn bản đã được ghi lại.
    *   **Lời khuyên:** Việc tích hợp với các API bên ngoài cần xử lý khóa API, giới hạn tốc độ và lỗi mạng.

4.  **Task 9.4: Lập chỉ mục bản ghi:**
    *   **Mục tiêu:** Lưu trữ bản ghi văn bản vào Elasticsearch để có thể tìm kiếm.
    *   **Thực hiện:**
        *   Sau khi có bản ghi, `ai-service` kết nối với Elasticsearch.
        *   Nó tạo một tài liệu mới trong một chỉ mục (ví dụ: `transcripts`) chứa văn bản, tham chiếu đến media và các metadata liên quan khác (ví dụ: `roomId`, `userId`).
    *   **Lời khuyên:** Thiết kế schema (mapping) cho chỉ mục Elasticsearch của bạn một cách cẩn thận để tối ưu hóa việc tìm kiếm.

5.  **Task 9.5: Tạo `search-service`:**
    *   **Mục tiêu:** Tạo microservice chuyên xử lý các yêu cầu tìm kiếm.
    *   **Thực hiện:**
        *   Tạo một dự án Spring Boot mới.
        *   **Dependencies:** `Web`, `Security`, `Actuator`, `Data Elasticsearch`.
    *   **Lời khuyên:** Dịch vụ này sẽ là cổng vào duy nhất cho tất cả các truy vấn tìm kiếm.

6.  **Task 9.6: Triển khai API tìm kiếm:**
    *   **Mục tiêu:** Cung cấp một endpoint để người dùng thực hiện tìm kiếm.
    *   **Thực hiện:**
        *   Tạo một endpoint: `GET /api/search`.
        *   Endpoint này chấp nhận một tham số truy vấn (ví dụ: `?q=...`).
        *   Dịch vụ sử dụng `ElasticsearchRestTemplate` hoặc một `Repository` để truy vấn một hoặc nhiều chỉ mục (ví dụ: `transcripts`, và có thể `chat_messages` nếu bạn cũng lập chỉ mục chúng).
        *   Nó tổng hợp và xếp hạng kết quả trước khi trả về cho người dùng.
    *   **Lời khuyên:** Tìm hiểu về các loại truy vấn Elasticsearch (match, term, fuzzy...) để xây dựng chức năng tìm kiếm mạnh mẽ.

7.  **Task 9.7: Set up Basic Monitoring:**
    *   **Mục tiêu:** Theo dõi hiệu suất và sức khỏe của `ai-service`.
    *   **Thực hiện:**
        *   Tạo một dashboard Grafana cơ bản để giám sát `ai-service`.
        *   **Các chỉ số chính:** Độ trễ của Kafka consumer (để xem nó có theo kịp các sự kiện không), số lượng công việc STT thành công/thất bại, và thời gian xử lý trung bình.
        *   Thiết lập một cảnh báo cơ bản nếu độ trễ của consumer vượt quá một ngưỡng nhất định.
    *   **Lời khuyên:** Giám sát là chìa khóa để vận hành microservice. Grafana và Prometheus là các công cụ phổ biến.
