# Hướng dẫn triển khai Microservice với Spring Boot

Chào bạn, đây là hướng dẫn chi tiết từng bước để bạn có thể tự tay xây dựng một microservice bằng Java Spring Boot, dựa trên các task đã được định nghĩa trong sprint này.

## Yêu cầu chuẩn bị

Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt các công cụ sau:

1.  **Java Development Kit (JDK):** Phiên bản 17 hoặc mới hơn.
2.  **IDE (Môi trường phát triển):** IntelliJ IDEA (Community hoặc Ultimate) hoặc VS Code với Extension Pack for Java.
3.  **Docker Desktop:** Để chạy các dịch vụ hạ tầng (database, Keycloak...) đã định nghĩa trong `docker-compose.dev.yml`.
4.  **Postman (hoặc một công cụ tương tự):** Để kiểm tra các API endpoint.

---

## Bước 1: Khởi tạo dự án với Spring Initializr

Đây là công cụ chính thức từ Spring giúp tạo nhanh một bộ khung dự án.

1.  **Truy cập:** Mở trình duyệt và đi đến [start.spring.io](https://start.spring.io).

2.  **Cấu hình dự án:** Điền các thông tin sau:
    *   **Project:** `Maven`
    *   **Language:** `Java`
    *   **Spring Boot:** Chọn phiên bản ổn định mới nhất (không chọn `SNAPSHOT`). Ví dụ: `3.x.x`.
    *   **Project Metadata:**
        *   **Group:** `com.studyhub` (Ví dụ, bạn có thể thay đổi)
        *   **Artifact:** `auth-service` (Tên của microservice, ví dụ: `user-service`, `api-gateway`)
        *   **Name:** `auth-service`
        *   **Description:** `Authentication Service for StudyHub` (Mô tả về dịch vụ)
        *   **Package name:** `com.studyhub.auth` (Tên package gốc)
    *   **Packaging:** `Jar`
    *   **Java:** `17` (Hoặc phiên bản bạn đã cài)

3.  **Thêm Dependencies (Các thư viện cần thiết):**
    Ở cột bên phải, nhấn nút "ADD DEPENDENCIES" và tìm kiếm, sau đó thêm các thư viện sau (đã được liệt kê trong Task 2.1):
    *   `Spring Web`: Để xây dựng RESTful API.
    *   `Spring Security`: Để tích hợp bảo mật, đặc biệt là OAuth2.
    *   `OAuth2 Resource Server`: Để cấu hình dịch vụ này như một máy chủ tài nguyên, có khả năng xác thực JWT token từ Keycloak.
    *   `Bean Validation`: Để kiểm tra tính hợp lệ của dữ liệu đầu vào.
    *   `Spring Boot Actuator`: Cung cấp các endpoint để giám sát và quản lý ứng dụng.
    *   `Spring Data JPA`: Để làm việc với cơ sở dữ liệu quan hệ.
    *   `PostgreSQL Driver`: Driver để kết nối tới PostgreSQL.
    *   `Lombok`: Giúp giảm thiểu việc viết code lặp đi lặp lại (boilerplate code) như getter, setter.
    *   `Springdoc OpenAPI`: Tự động sinh tài liệu API (Swagger UI).

4.  **Tạo và tải về:**
    *   Nhấn nút **"GENERATE"**. Một file `.zip` sẽ được tải về.
    *   Giải nén file này vào thư mục `services` trong monorepo của bạn. Ví dụ: `D:\Code\Projects\learning-project\studyhub\services\auth-service`.

---

## Bước 2: Tìm hiểu cấu trúc dự án

Sau khi giải nén, bạn sẽ có một cấu trúc thư mục chuẩn của Spring Boot:

```
auth-service/
├── .mvn/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/studyhub/auth/
│   │   │       └── AuthServiceApplication.java  <-- File chính để chạy ứng dụng
│   │   └── resources/
│   │       ├── static/
│   │       ├── templates/
│   │       └── application.properties         <-- File cấu hình quan trọng
│   └── test/
│       └── java/
│           └── com/studyhub/auth/
│               └── AuthServiceApplicationTests.java
├── mvnw
├── mvnw.cmd
└── pom.xml                                  <-- File quản lý dependencies của Maven
```

---

## Bước 3: Cấu hình ứng dụng

File `src/main/resources/application.properties` là nơi bạn định nghĩa các cấu hình cho ứng dụng. Chúng ta nên đổi nó thành `application.yml` vì định dạng YAML dễ đọc hơn.

1.  **Đổi tên file:** Đổi `application.properties` thành `application.yml`.

2.  **Thêm cấu hình:** Mở file `application.yml` và thêm nội dung sau. Đây là cấu hình cho `user-service` làm ví dụ, bạn sẽ cần điều chỉnh cho phù hợp.

    ```yaml
    # Cấu hình cổng chạy cho server (mỗi service nên chạy ở một cổng khác nhau)
    server:
      port: 8081 # Ví dụ: auth-service 8080, user-service 8081

    # Tên ứng dụng
    spring:
      application:
        name: user-service

      # Cấu hình kết nối cơ sở dữ liệu (Data Source)
      datasource:
        url: jdbc:postgresql://postgres-user-db:5432/studyhub_user # 'postgres-user-db' là tên service trong docker-compose.dev.yml
        username: user # Thay bằng username bạn cấu hình cho DB
        password: password # Thay bằng password bạn cấu hình cho DB
        driver-class-name: org.postgresql.Driver

      # Cấu hình JPA (Hibernate)
      jpa:
        hibernate:
          ddl-auto: update # Tự động cập nhật schema DB khi entity thay đổi (chỉ dùng cho dev)
        show-sql: true # Hiển thị câu lệnh SQL trong log, hữu ích khi debug
        properties:
          hibernate:
            format_sql: true
            dialect: org.hibernate.dialect.PostgreSQLDialect

      # Cấu hình Spring Security và OAuth2 Resource Server
      security:
        oauth2:
          resourceserver:
            jwt:
              # URI của Keycloak realm, nơi cung cấp thông tin để xác thực token
              issuer-uri: http://localhost:8080/realms/studyhub # 'studyhub' là tên realm bạn tạo
              jwk-set-uri: ${spring.security.oauth2.resourceserver.jwt.issuer-uri}/protocol/openid-connect/certs

    # Cấu hình Springdoc (Swagger UI)
    springdoc:
      api-docs:
        path: /api-docs # Đường dẫn để truy cập file JSON OpenAPI
      swagger-ui:
        path: /swagger-ui.html # Đường dẫn để truy cập giao diện Swagger
    ```

---

## Bước 4: Tạo một API Endpoint đơn giản được bảo mật

Bây giờ, hãy tạo một "controller" để xử lý các yêu cầu HTTP. Controller này sẽ có một endpoint yêu cầu người dùng phải xác thực.

1.  Tạo một package mới tên là `controller` bên trong `com.studyhub.user` (ví dụ cho user-service).
2.  Tạo một class Java mới tên là `UserController.java` trong package `controller`.

    ```java
    package com.studyhub.user.controller;

    import org.springframework.http.ResponseEntity;
    import org.springframework.security.core.annotation.AuthenticationPrincipal;
    import org.springframework.security.oauth2.jwt.Jwt;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    import java.util.Map;

    @RestController
    @RequestMapping("/api/users")
    public class UserController {

        /**
         * Endpoint này trả về thông tin của người dùng đã được xác thực.
         * Spring Security sẽ tự động xác thực JWT token từ header Authorization.
         * Nếu token hợp lệ, nó sẽ trích xuất thông tin và inject vào tham số 'jwt'.
         */
        @GetMapping("/me")
        public ResponseEntity<Map<String, Object>> getCurrentUser(@AuthenticationPrincipal Jwt jwt) {
            // jwt.getClaims() chứa tất cả thông tin trong token (subject, name, email...)
            return ResponseEntity.ok(jwt.getClaims());
        }
    }
    ```

---

## Bước 5: Chạy và kiểm thử ứng dụng

1.  **Khởi động hạ tầng:** Đảm bảo bạn đã chạy lệnh `docker-compose -f ops/docker-compose.dev.yml up -d` và tất cả các container (đặc biệt là `postgres-user-db` và `keycloak`) đang hoạt động.

2.  **Chạy ứng dụng:**
    *   **Từ IDE:** Mở file `UserServiceApplication.java` và nhấn nút "Run".
    *   **Từ dòng lệnh:** Mở terminal trong thư mục gốc của service (ví dụ: `studyhub/services/user-service`) và chạy lệnh `./mvnw spring-boot:run`.

3.  **Kiểm thử (Task 2.3):**
    *   **Lấy Token:** Dùng Postman gửi yêu cầu `POST` đến Keycloak để lấy access token.
        *   URL: `http://localhost:8080/realms/studyhub/protocol/openid-connect/token`
        *   Body (x-www-form-urlencoded):
            *   `client_id`: `studyhub-backend` (tên client bạn tạo trong Keycloak)
            *   `client_secret`: (secret của client `studyhub-backend`)
            *   `grant_type`: `password`
            *   `username`: (tên người dùng thử nghiệm bạn tạo)
            *   `password`: (mật khẩu của người dùng)
        *   Bạn sẽ nhận lại một `access_token`.
    *   **Gọi API:** Dùng Postman gửi yêu cầu `GET` đến endpoint `/api/users/me` của bạn.
        *   URL: `http://localhost:8081/api/users/me` (Lưu ý: đây là gọi trực tiếp, sau này sẽ gọi qua API Gateway).
        *   Header:
            *   `Authorization`: `Bearer <your_access_token>` (dán token bạn vừa lấy được vào đây).
    *   **Kết quả:** Nếu thành công, bạn sẽ nhận lại một JSON chứa thông tin người dùng từ token. Nếu gửi yêu cầu mà không có token, bạn sẽ nhận lỗi `401 Unauthorized`.

---

## Bước tiếp theo

Chúc mừng! Bạn đã tạo và chạy thành công microservice đầu tiên. Các bước tiếp theo sẽ là:

*   **Tạo API Gateway:** Tạo một service Spring Boot khác với dependency `Spring Cloud Gateway` để điều hướng request.
*   **Hoàn thiện Service:** Xây dựng các lớp `Entity` (để ánh xạ với bảng trong DB), `Repository` (để truy vấn DB), và `Service` (để xử lý logic nghiệp vụ).
*   **Giao tiếp giữa các service:** Tìm hiểu cách các service gọi nhau hoặc giao tiếp qua message broker như Kafka.

Hướng dẫn này là điểm khởi đầu. Hãy tiếp tục bám sát các task trong sprint và đừng ngần ngại đặt câu hỏi khi gặp khó khăn.

---

## Hướng dẫn cấu hình chi tiết cho từng Service

Dưới đây là cấu hình gợi ý cho từng microservice dựa trên kiến trúc mục tiêu. Bạn hãy sử dụng phần này để điều chỉnh **Dependencies** và file `application.yml` cho mỗi service bạn tạo.

### 1. API Gateway (`api-gateway`)

-   **Mục đích:** Là cửa ngõ duy nhất cho mọi request từ client, xử lý routing, rate limiting, và bảo mật.
-   **Port:** `8888`
-   **Dependencies trên Spring Initializr:**
    -   `Gateway` (Spring Cloud Gateway)
    -   `Spring Security`
    -   `OAuth2 Resource Server`
    -   `Resilience4j` (cho Circuit Breaker)
    -   `Spring Boot Actuator`
-   **Cấu hình `application.yml` (ví dụ):**
    ```yaml
    server:
      port: 8888
    spring:
      application:
        name: api-gateway
      cloud:
        gateway:
          routes:
            - id: user-service-route
              uri: lb://user-service # lb = load balancer, cần cấu hình service discovery
              predicates:
                - Path=/api/users/**
            - id: chat-service-route
              uri: lb://chat-service
              predicates:
                - Path=/api/messages/**, /ws/chat/**
      security: # Cấu hình bảo mật tương tự các service khác
        oauth2:
          resourceserver:
            jwt:
              issuer-uri: http://localhost:8080/realms/studyhub
    # Cần thêm cấu hình cho Resilience4j (timeouts, retries)
    ```

### 2. Auth Service (`auth-service`)

-   **Mục đích:** Xác thực người dùng và cấp token (vai trò này do Keycloak đảm nhiệm). Service này chủ yếu để kiểm tra và giải mã token, cung cấp thông tin người dùng cho các service khác nếu cần.
-   **Port:** `9000`
-   **Dependencies:**
    -   `Spring Web`
    -   `Spring Security`
    -   `OAuth2 Resource Server`
    -   `Spring Boot Actuator`
    -   `Springdoc OpenAPI`
-   **Logic chính:** Thường không có logic nghiệp vụ phức tạp. Có thể cung cấp một endpoint `/me` để client kiểm tra token của mình.

### 3. User Service (`user-service`)

-   **Mục đích:** Quản lý thông tin hồ sơ người dùng (profile, preferences...).
-   **Port:** `9001`
-   **Dependencies:**
    -   `Spring Web`
    -   `Spring Security`, `OAuth2 Resource Server`
    -   `Spring Data JPA`, `PostgreSQL Driver`
    -   `Bean Validation`, `Lombok`, `Springdoc OpenAPI`, `Spring Boot Actuator`
-   **Cấu hình `application.yml`:**
    -   Cấu hình `datasource` để kết nối tới `postgres-user-db`.
    -   Cấu hình `issuer-uri` để xác thực token.
-   **Logic chính:** Tạo `User` Entity, `UserRepository` và `UserController` cho các hoạt động CRUD (Create, Read, Update, Delete) trên hồ sơ người dùng.

### 4. Chat Service (`chat-service`)

-   **Mục đích:** Xử lý tin nhắn real-time và lưu trữ lịch sử chat.
-   **Port:** `9002`
-   **Dependencies:**
    -   `Spring Web`, `WebSocket`
    -   `Spring for Apache Kafka` (để phát event `ChatMessageCreated`)
    -   `Spring Data JPA`, `PostgreSQL Driver`
    -   `Spring Data Redis` (dùng cho Pub/Sub để scale WebSocket)
    -   `Spring Security`, `OAuth2 Resource Server`, `Lombok`, `Springdoc OpenAPI`
-   **Cấu hình `application.yml`:**
    -   Cấu hình `datasource` tới DB chat.
    -   Cấu hình `spring.kafka.bootstrap-servers`.
    -   Cấu hình `spring.redis.host`.
-   **Logic chính:**
    -   `ChatController` cho REST API (lấy lịch sử).
    -   `WebSocketHandler` để xử lý kết nối và tin nhắn real-time.
    -   Sử dụng `KafkaTemplate` để gửi event sau khi lưu tin nhắn.

### 5. Realtime Service (`realtime-service`)

-   **Mục đích:** Xử lý signaling cho WebRTC (video/voice call).
-   **Port:** `9003`
-   **Dependencies:**
    -   `Spring Web`, `WebSocket`
    -   `Spring Security`, `OAuth2 Resource Server`, `Lombok`
-   **Logic chính:**
    -   Tạo một `WebSocketHandler` để quản lý các "phòng" (study rooms).
    -   Xử lý các thông điệp signaling như `join`, `leave`, `offer`, `answer`, `ice-candidate` và chuyển tiếp chúng đến các thành viên phù hợp trong phòng. Service này không xử lý media stream.

### 6. Media Service (`media-service`)

-   **Mục đích:** Xử lý upload file media và khởi tạo các tác vụ xử lý (transcoding).
-   **Port:** `9004`
-   **Dependencies:**
    -   `Spring Web`
    -   `Spring for Apache Kafka` (để phát event `MediaUploaded`)
    -   `Spring for RabbitMQ` (để gửi task cho worker)
    -   `Spring Security`, `OAuth2 Resource Server`, `Lombok`, `Springdoc OpenAPI`
    -   Thư viện cho S3-compatible storage (ví dụ: `software.amazon.awssdk:s3`).
-   **Cấu hình `application.yml`:**
    -   Cấu hình Kafka, RabbitMQ.
    -   Cấu hình thông tin kết nối tới MinIO/S3 (endpoint, access-key, secret-key).
-   **Logic chính:**
    -   `MediaController` với endpoint `POST /api/media/upload`.
    -   Sau khi upload file lên MinIO thành công, gửi event `MediaUploaded` qua Kafka và/hoặc gửi một message tới RabbitMQ để worker xử lý.

### 7. AI Service (`ai-service`)

-   **Mục đích:** Thực hiện các tác vụ AI như Speech-to-Text, tóm tắt văn bản.
-   **Port:** `9005`
-   **Dependencies:**
    -   `Spring for Apache Kafka` (để lắng nghe event `MediaTranscoded`)
    -   `Spring Data Elasticsearch`
    -   `WebClient` (từ `spring-webflux` để gọi API của LLM/Whisper)
    -   `Lombok`
-   **Logic chính:**
    -   Tạo một `@KafkaListener` để nhận event.
    -   Khi có event, dùng `WebClient` để gọi dịch vụ STT (như Whisper).
    -   Lưu kết quả transcript vào Elasticsearch.

### 8. Search Service (`search-service`)

-   **Mục đích:** Cung cấp API để tìm kiếm thông tin (tin nhắn, tài liệu...).
-   **Port:** `9006`
-   **Dependencies:**
    -   `Spring Web`
    -   `Spring Data Elasticsearch`
    -   `Spring Security`, `OAuth2 Resource Server`, `Lombok`, `Springdoc OpenAPI`
-   **Cấu hình `application.yml`:**
    -   Cấu hình `spring.elasticsearch.uris`.
-   **Logic chính:**
    -   `SearchController` với endpoint `GET /api/search?q=...`.
    -   Sử dụng `ElasticsearchRestTemplate` hoặc `ElasticsearchRepository` để thực hiện truy vấn trên Elasticsearch.
