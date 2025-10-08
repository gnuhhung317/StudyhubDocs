# Sprint 1: Setup Keycloak & Authentication - Hướng dẫn từng bước

## 📋 Tổng quan
Sprint 1 tập trung vào việc thiết lập hệ thống xác thực với Keycloak, tạo trang đăng nhập và quản lý profile người dùng. Hướng dẫn này giảm thiểu code, tập trung vào các bước thực hiện cụ thể.

---

## Phần 1: Setup Keycloak (30-45 phút)

### Bước 1.1: Khởi động Keycloak với Docker

```bash
cd studyhub/ops
docker-compose -f docker-compose.dev.yml up -d keycloak postgres-keycloak
```

**Kiểm tra:** Truy cập `http://localhost:8080` - bạn sẽ thấy trang Keycloak Admin Console.

### Bước 1.2: Đăng nhập Admin Console

1. Mở browser: `http://localhost:8080`
2. Click **"Administration Console"**
3. Đăng nhập:
   - Username: `admin`
   - Password: `admin` (hoặc theo config trong `docker-compose.dev.yml`)

### Bước 1.3: Tạo Realm mới cho StudyHub

**Realm** là một không gian riêng biệt để quản lý users, roles, clients.

1. Ở góc trên bên trái, click dropdown **"Master"**
2. Click **"Create Realm"**
3. Điền thông tin:
   - **Realm name:** `studyhub`
   - **Enabled:** ON
4. Click **"Create"**

✅ **Checkpoint:** Bạn đã có realm `studyhub` và đang ở trong realm này.

### Bước 1.4: Tạo Client cho Backend Services

**Client** đại diện cho ứng dụng backend cần xác thực token.

1. Menu bên trái → **"Clients"** → Click **"Create client"**
2. **General Settings:**
   - Client type: `OpenID Connect`
   - Client ID: `studyhub-backend`
   - Name: `StudyHub Backend Services`
   - Click **"Next"**
3. **Capability config:**
   - Client authentication: **ON** (để có client secret)
   - Authorization: **OFF**
   - Authentication flow: Chỉ check **"Service accounts roles"** và **"Direct access grants"**
   - Click **"Next"**
4. **Login settings:**
   - Leave defaults (các field có thể để trống)
   - Click **"Save"**

✅ **Lưu lại Client Secret:**
1. Vào tab **"Credentials"**
2. Copy **Client secret** - sẽ dùng trong config Spring Boot

### Bước 1.5: Tạo Client cho Frontend

1. **"Clients"** → **"Create client"**
2. **General Settings:**
   - Client ID: `studyhub-frontend`
   - Name: `StudyHub Web Application`
   - Click **"Next"**
3. **Capability config:**
   - Client authentication: **OFF** (public client cho SPA)
   - Authorization: **OFF**
   - Standard flow: **ON** (Authorization Code Flow)
   - Direct access grants: **ON** (cho testing)
   - Click **"Next"**
4. **Login settings:**
   - Valid redirect URIs: `http://localhost:3000/*`
   - Valid post logout redirect URIs: `http://localhost:3000`
   - Web origins: `http://localhost:3000`
   - Click **"Save"**

### Bước 1.6: Tạo Roles

**Roles** định nghĩa quyền hạn trong hệ thống.

1. Menu → **"Realm roles"** → **"Create role"**
2. Tạo 3 roles sau:

**Role 1: STUDENT**
- Role name: `STUDENT`
- Description: `Standard student user`
- Click **"Save"**

**Role 2: INSTRUCTOR**
- Role name: `INSTRUCTOR`
- Description: `Teacher or instructor`
- Click **"Save"**

**Role 3: ADMIN**
- Role name: `ADMIN`
- Description: `System administrator`
- Click **"Save"**

### Bước 1.7: Tạo Test Users

1. Menu → **"Users"** → **"Add user"**

**User 1: Student Test**
- Username: `student1`
- Email: `student1@studyhub.local`
- First name: `Test`
- Last name: `Student`
- Email verified: **ON**
- Click **"Create"**

2. **Set Password:**
   - Tab **"Credentials"** → **"Set password"**
   - Password: `student123`
   - Password confirmation: `student123`
   - Temporary: **OFF**
   - Click **"Save"**

3. **Assign Role:**
   - Tab **"Role mapping"** → **"Assign role"**
   - Filter by realm roles
   - Check **"STUDENT"**
   - Click **"Assign"**

**Lặp lại cho User 2: Instructor Test**
- Username: `instructor1`
- Email: `instructor1@studyhub.local`
- Password: `instructor123`
- Role: `INSTRUCTOR`

**Lặp lại cho User 3: Admin Test**
- Username: `admin1`
- Email: `admin1@studyhub.local`
- Password: `admin123`
- Role: `ADMIN`

✅ **Checkpoint:** Bạn có 3 users với 3 roles khác nhau.

### Bước 1.8: Cấu hình Token Settings (Tuỳ chọn nhưng khuyến nghị)

1. Menu → **"Realm settings"** → Tab **"Tokens"**
2. Điều chỉnh:
   - **Access Token Lifespan:** `5 minutes` (dev: có thể để `30 minutes`)
   - **Refresh Token Lifespan:** `30 minutes`
   - **SSO Session Idle:** `30 minutes`
   - **SSO Session Max:** `10 hours`
3. Click **"Save"**

---

## Phần 2: Test Keycloak với Postman (15 phút)

### Bước 2.1: Lấy Access Token

1. Mở Postman
2. Tạo request mới:
   - Method: **POST**
   - URL: `http://localhost:8080/realms/studyhub/protocol/openid-connect/token`
3. Tab **"Body"** → chọn **"x-www-form-urlencoded"**
4. Thêm parameters:
   ```
   client_id: studyhub-backend
   client_secret: <paste client secret từ bước 1.4>
   grant_type: password
   username: student1
   password: student123
   ```
5. Click **"Send"**

✅ **Expected Response:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI...",
  "token_type": "Bearer"
}
```

### Bước 2.2: Decode và kiểm tra Token

1. Copy `access_token`
2. Vào https://jwt.io
3. Paste token vào phần **Encoded**
4. Kiểm tra **Payload** có:
   - `sub`: user ID
   - `preferred_username`: "student1"
   - `email`: "student1@studyhub.local"
   - `realm_access.roles`: ["STUDENT"]

---

## Phần 3: Setup Auth Service (45 phút)

### Bước 3.1: Tạo Auth Service từ Spring Initializr

**Không cần code, chỉ config!** Auth Service chủ yếu là config để xác thực token.

1. Vào https://start.spring.io
2. Config:
   - Project: **Maven**
   - Language: **Java**
   - Spring Boot: **3.2.x** (stable latest)
   - Group: `com.studyhub`
   - Artifact: `auth-service`
   - Java: **21**
3. Dependencies:
   - Spring Web
   - OAuth2 Resource Server
   - Spring Security
   - Spring Boot Actuator
   - Lombok
   - Validation
4. Generate & download
5. Giải nén vào `studyhub/services/auth-service`

### Bước 3.2: Cập nhật Parent POM

**File:** `studyhub/services/auth-service/pom.xml`

Sửa phần `<parent>`:
```xml
<parent>
    <groupId>com.studyhub</groupId>
    <artifactId>studyhub-services-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
```

Xoá phần `<dependencies>` đã có từ Initializr, thêm:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

### Bước 3.3: Tạo application.yml

**File:** `src/main/resources/application.yml`

```yaml
server:
  port: 9000

spring:
  application:
    name: auth-service
  profiles:
    active: dev

---
# Development Profile
spring:
  config:
    activate:
      on-profile: dev
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/studyhub
          jwk-set-uri: http://localhost:8080/realms/studyhub/protocol/openid-connect/certs

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

### Bước 3.4: Tạo Security Config (Minimal)

**File:** `src/main/java/com/studyhub/auth/config/SecurityConfig.java`

```java
package com.studyhub.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    @Profile("dev")
    public SecurityFilterChain devFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt());
        return http.build();
    }
}
```

### Bước 3.5: Tạo Test Endpoint

**File:** `src/main/java/com/studyhub/auth/controller/AuthController.java`

```java
package com.studyhub.auth.controller;

import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @GetMapping("/me")
    public Map<String, Object> getCurrentUser(@AuthenticationPrincipal Jwt jwt) {
        return Map.of(
            "username", jwt.getClaimAsString("preferred_username"),
            "email", jwt.getClaimAsString("email"),
            "roles", jwt.getClaimAsStringList("realm_access.roles"),
            "userId", jwt.getSubject()
        );
    }
}
```

### Bước 3.6: Chạy và Test

1. **Start service:**
   ```bash
   cd studyhub/services/auth-service
   mvn spring-boot:run
   ```

2. **Test với Postman:**
   - URL: `http://localhost:9000/api/auth/me`
   - Method: **GET**
   - Headers:
     ```
     Authorization: Bearer <paste access_token từ bước 2.1>
     ```
   - Click **Send**

✅ **Expected Response:**
```json
{
  "username": "student1",
  "email": "student1@studyhub.local",
  "roles": ["STUDENT"],
  "userId": "abc-123-..."
}
```

---

## Phần 4: Setup User Service (60 phút)

### Bước 4.1: Tạo User Service

Tương tự Auth Service, tạo từ Spring Initializr với thêm:
- Dependencies: Spring Data JPA, PostgreSQL Driver, Flyway Migration

### Bước 4.2: Khởi động PostgreSQL cho User DB

```bash
cd studyhub/ops
docker-compose -f docker-compose.dev.yml up -d postgres-user-db
```

### Bước 4.3: Cấu hình application.yml

**File:** `user-service/src/main/resources/application.yml`

```yaml
server:
  port: 9001

spring:
  application:
    name: user-service
  profiles:
    active: dev

---
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/studyhub_user
    username: user
    password: password
  jpa:
    hibernate:
      ddl-auto: validate  # Dùng Flyway nên để validate
    show-sql: true
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/studyhub
```

### Bước 4.4: Tạo Entity & Repository (Minimal)

**Entity:** `src/main/java/com/studyhub/user/entity/UserProfile.java`

Chỉ các field cơ bản:
- id (UUID, từ Keycloak)
- displayName
- bio
- avatarUrl
- createdAt, updatedAt

**Repository:** Extend `JpaRepository<UserProfile, String>`

### Bước 4.5: Tạo Flyway Migration

**File:** `src/main/resources/db/migration/V1__Create_user_profile_table.sql`

```sql
CREATE TABLE user_profile (
    id VARCHAR(255) PRIMARY KEY,
    display_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Bước 4.6: Service Layer với Auto-Create Profile

**Logic:** Khi user đăng nhập lần đầu, tự động tạo profile từ thông tin trong JWT.

**Service:** `UserProfileService.java`
- Method: `getOrCreateProfile(Jwt jwt)`
- Kiểm tra user đã có profile chưa
- Nếu chưa → tạo mới từ thông tin trong token
- Return profile

### Bước 4.7: Controller với GET và PUT

**Controller:** `UserProfileController.java`
- `GET /api/users/me/profile` → lấy profile của chính mình
- `PUT /api/users/me/profile` → cập nhật profile

### Bước 4.8: Test User Service

1. Start service: `mvn spring-boot:run`
2. Lấy token mới (Postman)
3. Call `GET /api/users/me/profile` với Authorization header
4. Profile tự động được tạo lần đầu
5. Call `PUT /api/users/me/profile` với body JSON để update

---

## Phần 5: Setup Frontend Login Page (90 phút)

### Bước 5.1: Install Dependencies

```bash
cd studyhub/frontend
pnpm install next-auth @auth/core
```

### Bước 5.2: Cấu hình NextAuth.js

**File:** `app/api/auth/[...nextauth]/route.ts`

Config Keycloak provider với:
- `issuer`: `http://localhost:8080/realms/studyhub`
- `clientId`: `studyhub-frontend`
- `authorization`: Authorization Code Flow

**Không cần code chi tiết**, dùng template từ NextAuth docs.

### Bước 5.3: Tạo Login Page

**File:** `app/auth/login/page.tsx`

Minimal UI:
- Logo StudyHub
- Button "Sign in with Keycloak"
- Click button → `signIn('keycloak')`

**Style:** Dùng components có sẵn từ `shadcn/ui`.

### Bước 5.4: Tạo Protected Layout

**File:** `app/(protected)/layout.tsx`

- Check session
- Nếu chưa login → redirect về `/auth/login`
- Nếu đã login → render children

### Bước 5.5: Tạo Profile Page

**File:** `app/(protected)/profile/page.tsx`

**Features:**
1. Fetch profile từ `http://localhost:9001/api/users/me/profile` (qua API Gateway sau này)
2. Hiển thị:
   - Avatar (hoặc default)
   - Display Name
   - Email (từ session)
   - Bio
3. Button "Edit Profile" → mở modal/form
4. Form edit:
   - Input: Display Name
   - Textarea: Bio
   - Upload Avatar (optional, có thể skip sprint 1)
   - Button Save → PUT request

**UI Components:** Dùng `shadcn/ui` Card, Input, Textarea, Button.

### Bước 5.6: Test Flow

1. Start frontend: `pnpm dev`
2. Vào `http://localhost:3000`
3. Click "Sign in"
4. Redirect sang Keycloak
5. Login với `student1 / student123`
6. Redirect về homepage
7. Navigate tới Profile
8. Thấy thông tin user
9. Edit và save
10. Refresh → data đã update

---

## Phần 6: Integration Testing (30 phút)

### Test Case 1: Login Flow
1. ✅ User có thể login qua Keycloak
2. ✅ Sau login, session được tạo
3. ✅ Token có chứa đúng roles

### Test Case 2: Profile Creation
1. ✅ User login lần đầu → profile tự tạo
2. ✅ GET /api/users/me/profile trả về thông tin đúng

### Test Case 3: Profile Update
1. ✅ PUT request với data mới
2. ✅ Data được lưu vào DB
3. ✅ GET lại thấy data mới

### Test Case 4: Authorization
1. ✅ Gọi API không có token → 401
2. ✅ Gọi API với token hết hạn → 401
3. ✅ Gọi API với token hợp lệ → 200

---

## 📝 Checklist hoàn thành Sprint 1

- [ ] Keycloak running với realm `studyhub`
- [ ] Client `studyhub-backend` và `studyhub-frontend` đã tạo
- [ ] 3 realm roles: STUDENT, INSTRUCTOR, ADMIN
- [ ] 3 test users với roles tương ứng
- [ ] Auth Service xác thực token thành công
- [ ] User Service tạo và quản lý profile
- [ ] PostgreSQL user-db hoạt động, có migration
- [ ] Frontend login page hoạt động
- [ ] Frontend profile page hiển thị và edit được
- [ ] Integration test pass

---

## 🎯 Kết quả đạt được

**User Story:** "Là một người dùng, tôi có thể đăng ký/đăng nhập và quản lý profile của mình"

✅ **Hoàn thành:**
- Hệ thống xác thực hoàn chỉnh
- User có thể login qua Keycloak
- Profile tự động tạo và có thể chỉnh sửa
- Frontend có login page và profile page
- Backend services bảo mật với JWT

---

## 🚀 Bước tiếp theo (Sprint 2)

- API Gateway để route requests
- Service-to-service communication
- Refresh token handling
- Email verification
- Password reset flow

---

## 💡 Tips

### Debugging
- Check Keycloak logs: `docker logs keycloak`
- Check service logs trong console
- Dùng jwt.io để decode token
- Enable SQL logging để xem queries

### Common Issues
1. **Token invalid:** Check issuer-uri match với Keycloak realm
2. **CORS error:** Config CORS trong Spring Security
3. **DB connection failed:** Check docker container status
4. **Port conflict:** Đảm bảo ports 8080, 9000, 9001, 3000 available

### Performance
- Trong dev, token lifespan có thể để dài (30 min) để tiện test
- Production nên để ngắn (5 min) và dùng refresh token
