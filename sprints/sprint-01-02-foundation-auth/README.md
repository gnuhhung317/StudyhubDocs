# Sprint 1-2: Foundation & Auth

**Mục tiêu:** Xây dựng nền tảng cơ bản của dự án, bao gồm hạ tầng phát triển và các dịch vụ xác thực, quản lý người dùng cốt lõi.

**Giải thích cho Junior:**

*   **Microservice là gì?** Thay vì xây dựng một ứng dụng lớn duy nhất (monolith), chúng ta chia nhỏ nó thành nhiều dịch vụ nhỏ, độc lập. Mỗi dịch vụ tập trung vào một chức năng cụ thể (ví dụ: `auth-service` chỉ lo xác thực, `user-service` chỉ lo quản lý người dùng). Điều này giúp dễ phát triển, triển khai và mở rộng hơn.
*   **Monorepo:** Mặc dù là microservice, chúng ta sẽ bắt đầu với một "monorepo" – tức là tất cả mã nguồn của các dịch vụ nằm trong cùng một kho lưu trữ Git. Điều này giúp quản lý mã nguồn dễ dàng hơn khi dự án còn nhỏ và bạn chưa quen với việc quản lý nhiều repo.
*   **Hạ tầng phát triển với Docker Compose:** Để các microservice hoạt động, chúng cần các dịch vụ hỗ trợ như cơ sở dữ liệu, hệ thống xác thực, hàng đợi tin nhắn... Docker Compose giúp bạn định nghĩa và chạy tất cả các dịch vụ này trên máy cục bộ của mình một cách dễ dàng, mô phỏng môi trường sản phẩm.
*   **API Gateway:** Tưởng tượng bạn có nhiều cửa hàng nhỏ (microservice). API Gateway là một "trung tâm mua sắm" duy nhất mà khách hàng (ứng dụng frontend) đến. Nó sẽ điều hướng khách hàng đến đúng cửa hàng, và có thể xử lý một số việc chung như xác thực trước khi gửi yêu cầu vào bên trong.
*   **Keycloak (Identity Provider):** Đây là một "người gác cổng" chuyên nghiệp. Thay vì mỗi dịch vụ tự lo việc đăng nhập/đăng ký, Keycloak sẽ làm tất cả. Khi người dùng đăng nhập thành công, Keycloak sẽ cấp cho họ một "vé" (JWT - JSON Web Token). Các dịch vụ khác chỉ cần kiểm tra "vé" này là hợp lệ hay không, mà không cần biết chi tiết về quá trình đăng nhập.
*   **OAuth2 & JWT:** OAuth2 là một giao thức ủy quyền, cho phép ứng dụng của bạn truy cập tài nguyên của người dùng mà không cần biết mật khẩu của họ. JWT là định dạng của "vé" mà Keycloak cấp. Nó chứa thông tin về người dùng và được ký điện tử để đảm bảo tính toàn vẹn.

**Kế hoạch chi tiết (có hướng dẫn):**

### Tuần 1: Hạ tầng & Thiết lập dự án

1.  **Task 1.1: Tạo cấu trúc Monorepo:**
    *   **Mục tiêu:** Tổ chức mã nguồn một cách rõ ràng.
    *   **Thực hiện:** Tạo các thư mục như `services` (chứa mã nguồn các microservice), `contracts` (chứa định nghĩa API, sự kiện chung), `platform` (cấu hình chung), `libs` (thư viện dùng chung), `ops` (cấu hình hạ tầng như Docker Compose), `docs` (tài liệu).
    *   **Lời khuyên:** Việc này giúp bạn dễ dàng tìm thấy các phần khác nhau của dự án và duy trì sự ngăn nắp.

2.  **Task 1.2: Định nghĩa hạ tầng phát triển với Docker Compose:**
    *   **Mục tiêu:** Chạy các dịch vụ hỗ trợ (database, Keycloak, Kafka...) trên máy cục bộ.
    *   **Thực hiện:**
        *   Tạo file `ops/docker-compose.dev.yml`.
        *   Thêm các dịch vụ:
            *   `keycloak`: Hệ thống xác thực.
            *   `postgres-user-db`: Cơ sở dữ liệu cho dịch vụ người dùng và xác thực.
            *   `kafka` & `zookeeper`: Hệ thống hàng đợi tin nhắn (event broker) – dùng để các dịch vụ giao tiếp với nhau một cách bất đồng bộ.
            *   `rabbitmq`: Hàng đợi tác vụ (task queue broker) – dùng cho các tác vụ nền.
            *   `redis`: Bộ nhớ đệm (cache) – giúp tăng tốc độ đọc dữ liệu.
            *   `minio`: Lưu trữ file tương thích S3 – dùng để lưu trữ ảnh, video...
            *   `zipkin`: Theo dõi phân tán (distributed tracing) – giúp bạn xem một yêu cầu đi qua những dịch vụ nào.
        *   Chạy lệnh `docker-compose -f ops/docker-compose.dev.yml up -d` để khởi động tất cả.
    *   **Lời khuyên:** Hiểu rõ vai trò của từng dịch vụ. Khi gặp lỗi, hãy kiểm tra log của từng container (`docker-compose logs <service_name>`).

3.  **Task 1.3: Cấu hình Keycloak:**
    *   **Mục tiêu:** Thiết lập Keycloak để quản lý người dùng và cấp phát token.
    *   **Thực hiện:**
        *   Truy cập giao diện người dùng của Keycloak (thường là `http://localhost:8080`).
        *   Tạo một "realm" mới, ví dụ: `studyhub`. Realm là một không gian riêng biệt để quản lý người dùng và ứng dụng.
        *   Tạo một "client" mới: `studyhub-backend` với `Access Type: confidential`. Client này đại diện cho các dịch vụ backend của bạn khi giao tiếp với Keycloak.
        *   Tạo một người dùng thử nghiệm.
    *   **Lời khuyên:** Keycloak có nhiều khái niệm mới, hãy dành thời gian đọc tài liệu cơ bản về Realm, Client, User để hiểu rõ hơn.

### Tuần 2: Các dịch vụ đầu tiên & API Gateway

1.  **Task 2.1: Tạo `auth-service` và `user-service`:**
    *   **Mục tiêu:** Xây dựng hai microservice đầu tiên.
    *   **Thực hiện:**
        *   Sử dụng `start.spring.io` để tạo hai dự án Spring Boot mới trong thư mục `services`.
        *   **Dependencies (phụ thuộc):** Chọn `Web`, `Security`, `Validation`, `Actuator`, `Data JPA`, `PostgreSQL Driver`, `Lombok`, `Springdoc OpenAPI`.
            *   `Web`: Để xây dựng API REST.
            *   `Security`: Để bảo mật ứng dụng.
            *   `Validation`: Để kiểm tra dữ liệu đầu vào.
            *   `Actuator`: Cung cấp các endpoint để giám sát ứng dụng.
            *   `Data JPA` & `PostgreSQL Driver`: Để tương tác với cơ sở dữ liệu PostgreSQL.
            *   `Lombok`: Giúp giảm boilerplate code (mã lặp lại).
            *   `Springdoc OpenAPI`: Tự động tạo tài liệu API.
        *   **Cấu hình (`application.yml`):**
            *   Cấu hình `user-service` kết nối đến `postgres-user-db` (đã chạy trong Docker Compose).
            *   Cấu hình cả hai dịch vụ hoạt động như `OAuth2 Resource Servers`. Điều này có nghĩa là chúng sẽ kiểm tra các JWT token được gửi đến từ Keycloak. Bạn cần cung cấp `issuer-uri` của Keycloak.
    *   **Lời khuyên:** Khi cấu hình `application.yml`, hãy chú ý đến các biến môi trường hoặc tên service trong Docker Compose để đảm bảo kết nối đúng.

2.  **Task 2.2: Tạo `api-gateway`:**
    *   **Mục tiêu:** Tạo một điểm truy cập duy nhất cho các client và điều hướng yêu cầu.
    *   **Thực hiện:**
        *   Tạo một dự án Spring Boot mới với `Spring Cloud Gateway` và `Actuator`.
        *   Cấu hình các route trong `application.yml` để chuyển tiếp yêu cầu:
            *   Ví dụ: `api/auth/**` sẽ được chuyển đến `auth-service`.
            *   `api/users/**` sẽ được chuyển đến `user-service`.
    *   **Lời khuyên:** API Gateway là một thành phần quan trọng. Nó không chỉ điều hướng mà còn có thể xử lý các tác vụ chung như xác thực, ghi log, giới hạn tốc độ (rate limiting).

3.  **Task 2.3: Xác minh luồng xác thực:**
    *   **Mục tiêu:** Đảm bảo rằng quá trình đăng nhập và truy cập tài nguyên được bảo mật hoạt động đúng.
    *   **Thực hiện:**
        *   Sử dụng Postman (hoặc công cụ tương tự):
            *   **Bước 1:** Gửi yêu cầu đến Keycloak để lấy access token cho người dùng thử nghiệm.
            *   **Bước 2:** Tạo một endpoint đơn giản, được bảo mật trong `user-service` (ví dụ: `GET /api/users/me`).
            *   **Bước 3:** Gọi endpoint này thông qua `api-gateway`, cung cấp access token dưới dạng Bearer token trong header `Authorization`.
        *   **Xác minh:** Yêu cầu phải thành công và trả về dữ liệu người dùng. Một yêu cầu không có token hợp lệ phải trả về lỗi `401 Unauthorized`.
    *   **Lời khuyên:** Đây là bước kiểm tra quan trọng nhất của sprint này. Nếu có lỗi, hãy kiểm tra log của API Gateway, `auth-service`, `user-service` và Keycloak.
