# Hướng Dẫn Setup Dự Án Java Spring Boot Microservices (Maven Multi-Module)

> **Tài liệu hướng dẫn từ A-Z**: Xây dựng project Java Spring Boot Microservices theo mô hình **Maven Multi-module Monorepo**, tối ưu hóa cấu hình Parent POM, Quản lý Dependency, Containerization và áp dụng các Best Practices nâng cao.

---

## 📋 MỤC LỤC

- [PHẦN I: TỔNG QUAN & KIẾN TRÚC PROJECT](#phần-i-tổng-quan--kiến-trúc-project)
  - [1.1. Mục tiêu & Cấu trúc Thư mục](#11-mục-tiêu--cấu-trúc-thư-mục)
  - [1.2. Phân biệt Khái niệm: Parent POM vs Aggregator POM](#12-phân-biệt-khái-niệm-parent-pom-vs-aggregator-pom)
  - [1.3. Mô hình Kiến trúc Maven Đề xuất](#13-mô-hình-kiến-trúc-maven-đề-xuất)
- [PHẦN II: CHUẨN BỊ MÔI TRƯỜNG & KHỞI TẠO ROOT PROJECT](#phần-ii-chuẩn-bị-môi-trường--khởi-tạo-root-project)
  - [2.1. Yêu cầu Môi trường (Prerequisites)](#21-yêu-cầu-môi-trường-prerequisites)
  - [2.2. Khởi tạo Root Project & Maven Wrapper](#22-khởi-tạo-root-project--maven-wrapper)
  - [2.3. Cấu hình Root pom.xml Chuẩn hóa (Parent POM)](#23-cấu-hình-root-pomxml-chuẩn-hóa-parent-pom)
  - [2.4. Giải thích Chi tiết các Thẻ Cấu hình Root POM](#24-giải-thích-chi-tiết-các-thẻ-cấu-hình-root-pom)
- [PHẦN III: THIẾT LẬP CÁC MODULE CHI TIẾT](#phần-iii-thiết-lập-các-module-chi-tiết)
  - [3.1. Thiết lập Module common & Cải tiến Quan trọng (Tránh Bẫy Plugin Build)](#31-thiết-lập-module-common--cải-tiến-quan-trọng-tránh-bẫy-plugin-build)
  - [3.2. Thiết lập Microservice Đầu tiên: user-service](#32-thiết-lập-microservice-đầu-tiên-user-service)
  - [3.3. Khởi tạo các Microservice Tiếp theo: order-service & notification-service](#33-khởi-tạo-các-microservice-tiếp-theo-order-service--notification-service)
  - [3.4. Chuẩn hóa Cấu trúc Package trong Microservice](#34-chuẩn-hóa-cấu-trúc-package-trong-microservice)
- [PHẦN IV: CẤU HÌNH, DATABASE & GIAO TIẾP GIỮA CÁC SERVICE](#phần-iv-cấu-hình-database--giao-tiếp-giữa-các-service)
  - [4.1. Quản lý File Cấu hình (application.yml & Profiles)](#41-quản-lý-file-cấu-hình-applicationyml--profiles)
  - [4.2. Giao tiếp giữa các Microservice (Synchronous REST/Feign vs Asynchronous Event-Driven)](#42-giao-tiếp-giữa-các-microservice-synchronous-restfeign-vs-asynchronous-event-driven)
  - [4.3. Nguyên tắc Quản lý Dữ liệu (Database per Service)](#43-nguyên-tắc-quản-lý-dữ-liệu-database-per-service)
- [PHẦN V: THAO TÁC MAVEN & CHUẨN HÓA QUY TRÌNH BUILD](#phần-v-thao-tác-maven--chuẩn-hóa-quy-trình-build)
  - [5.1. Cơ chế Maven Reactor & Thứ tự Build](#51-cơ-chế-maven-reactor--thứ-tự-build)
  - [5.2. Các Câu lệnh Maven Cốt lõi trong Dự án Multi-module](#52-các-câu-lệnh-maven-cốt-lõi-trong-dự-án-multi-module)
  - [5.3. Quản lý Dependency Scope & Dependency Hygiene](#53-quản-lý-dependency-scope--dependency-hygiene)
- [PHẦN VI: CONTAINERIZATION, DOCKER COMPOSE & PATTERNS NÂNG CAO](#phần-vi-containerization-docker-compose--patterns-nâng-cao)
  - [6.1. Dockerize Microservices & File docker-compose.yml Mẫu](#61-dockerize-microservices--file-docker-composeyml-mẫu)
  - [6.2. Kiến trúc Nâng cao: API Gateway, Security & Observability](#62-kiến-trúc-nâng-cao-api-gateway-security--observability)
  - [6.3. Distributed Transactions (Saga Pattern & Outbox Pattern) & Resilience](#63-distributed-transactions-saga-pattern--outbox-pattern--resilience)
- [PHẦN VII: ROADMAP VÀ CHECKLIST THỰC HÀNH](#phần-vii-roadmap-và-checklist-thực-hành)
  - [7.1. Lộ trình Học tập & Phát triển Năng lực Java Backend Engineer](#71-lộ-trình-học-tập--phát-triển-năng-lực-java-backend-engineer)
  - [7.2. Checklist Setup Project từ A-Z](#72-checklist-setup-project-từ-a-z)
  - [7.3. 10 Quy tắc Vàng (Core Rules) & Kết luận](#73-10-quy-tắc-vàng-core-rules--kết-luận)

---

# PHẦN I: TỔNG QUAN & KIẾN TRÚC PROJECT

## 1.1. Mục tiêu & Cấu trúc Thư mục

Chúng ta sẽ xây dựng một dự án Microservices chuẩn doanh nghiệp sử dụng **Spring Boot 3**, **Java 21** và **Maven Multi-module Monorepo**.

Cấu trúc cây thư mục mục tiêu:

```text
microservices-parent/
├── .mvn/                       # Cấu hình Maven Wrapper
├── mvnw                        # Script chạy Maven trên Linux/macOS
├── mvnw.cmd                    # Script chạy Maven trên Windows
├── pom.xml                     # Root POM (Parent & Aggregator)
├── README.md                   # Tài liệu dự án
├── docker-compose.yml          # Môi trường chạy Local (DB, Kafka, Redis)
│
├── common/                     # Thư viện code/DTO/Utils dùng chung
│   ├── pom.xml
│   └── src/
│
├── user-service/               # Service Quản lý Nguời dùng (Port 8081)
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/example/userservice/
│       │   └── resources/application.yml
│       └── test/
│
├── order-service/              # Service Quản lý Đơn hàng (Port 8082)
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/example/orderservice/
│       │   └── resources/application.yml
│       └── test/
│
└── notification-service/       # Service Gửi Thông báo (Port 8083)
    ├── pom.xml
    └── src/
        ├── main/
        │   ├── java/com/example/notificationservice/
        │   └── resources/application.yml
        └── test/
```

### Ý nghĩa của các Module:
1. **`microservices-parent`**: Đóng vai trò là Root POM (quản lý dependency, plugin và danh sách module con).
2. **`common`**: Module chứa các Class/DTO/Exception/Response dùng chung thực sự. Module này được đóng gói dưới dạng thư viện JAR (`.jar`).
3. **`user-service`, `order-service`, `notification-service`**: Mỗi service là một **Spring Boot Application độc lập**, có thể build, package và deploy riêng biệt.

---

## 1.2. Phân biệt Khái niệm: Parent POM vs Aggregator POM

Trong Maven có 3 khái niệm rất dễ gây nhầm lẫn:

### 1. Parent POM (Kế thừa - Inheritance)
Parent POM đóng vai trò như một "lớp cha" cung cấp cấu hình dùng chung cho các module con kế thừa:
- Khai báo phiên bản Java (`java.version`).
- Quản lý phiên bản thư viện trung tâm (`<dependencyManagement>`).
- Quản lý cấu hình Plugin chung (`<pluginManagement>`).
- Cấu hình encoding, compiler options.

### 2. Aggregator POM (Gom nhóm - Aggregation)
Aggregator POM đóng vai trò gom nhóm các module để có thể build toàn bộ project chỉ bằng một lệnh duy nhất:
```xml
<modules>
    <module>common</module>
    <module>user-service</module>
    <module>order-service</module>
    <module>notification-service</module>
</modules>
```
Khi đứng ở Root và chạy `mvn clean install`, Maven sẽ tự động đọc danh sách này và build toàn bộ dự án.

### 3. Kết hợp Parent & Aggregator trong Root POM
Trong mô hình Monorepo Microservices, cách tiếp cận chuẩn và gọn nhất là kết hợp cả **Parent** và **Aggregator** vào cùng một Root `pom.xml`.

```text
               Root POM (microservices-parent)
            ┌──────────────────┴──────────────────┐
            │                                     │
      Parent Component                     Aggregator Component
(Quản lý Versions & Plugins)            (Khai báo danh sách Modules)
            │                                     │
    ┌───────┴───────┬───────────────┐             │
  common       user-service    order-service      │
    ▲               ▲               ▲             │
    └───────────────┴───────────────┴─────────────┘
                  Kế thừa cấu hình
```

---

## 1.3. Mô hình Kiến trúc Maven Đề xuất

> [!IMPORTANT]
> **Quy tắc quan trọng**: Root POM tuyệt đối **KHÔNG** chứa bất kỳ mã nguồn Java nào và **KHÔNG** phải là một Spring Boot Application. Root POM phải đặt thẻ `<packaging>pom</packaging>`.

---

# PHẦN II: CHUẨN BỊ MÔI TRƯỜNG & KHỞI TẠO ROOT PROJECT

## 2.1. Yêu cầu Môi trường (Prerequisites)

Trước khi bắt đầu, hãy đảm bảo hệ thống của bạn đã cài đặt các công cụ sau:
- **JDK 21** (hoặc tối thiểu JDK 17 cho Spring Boot 3.x).
- **Maven 3.9+**.
- **Git**.
- **Docker & Docker Desktop** (để chạy PostgreSQL, Redis, Kafka local).
- **IDE**: IntelliJ IDEA Ultimate / VS Code với Java Extension Pack.

Kiểm tra phiên bản Java & Maven:
```bash
java -version
mvn -version
```

---

## 2.2. Khởi tạo Root Project & Maven Wrapper

Tạo thư mục root cho dự án:
```bash
mkdir microservices-parent
cd microservices-parent
```

Tạo **Maven Wrapper** để đảm bảo tất cả các thành viên trong team và hệ thống CI/CD sử dụng chung một phiên bản Maven thống nhất:
```bash
mvn wrapper:wrapper
```
Sau khi chạy lệnh trên, dự án sẽ sinh ra các file `.mvn/`, `mvnw`, và `mvnw.cmd`.

---

## 2.3. Cấu hình Root pom.xml Chuẩn hóa (Parent POM)

Tạo file `pom.xml` tại thư mục root `microservices-parent/`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- 1. Kế thừa Spring Boot Starter Parent -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.4</version>
        <relativePath/> <!-- Tìm parent từ remote repository -->
    </parent>

    <!-- 2. Thông tin G.A.V của Root Project -->
    <groupId>com.example</groupId>
    <artifactId>microservices-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <!-- QUAN TRỌNG: Root bắt buộc phải là packaging pom -->
    <packaging>pom</packaging>

    <name>microservices-parent</name>
    <description>Root Parent POM for Java Spring Boot Microservices</description>

    <!-- 3. Danh sách các Module con (Aggregator) -->
    <modules>
        <module>common</module>
        <module>user-service</module>
        <module>order-service</module>
        <module>notification-service</module>
    </modules>

    <!-- 4. Quản lý Properties dùng chung -->
    <properties>
        <java.version>21</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        
        <!-- Versions của các thư viện bổ sung -->
        <spring-cloud.version>2023.0.3</spring-cloud.version>
        <lombok.version>1.18.34</lombok.version>
    </properties>

    <!-- 5. Quản lý Version Dependency trung tâm (Dependency Management) -->
    <dependencyManagement>
        <dependencies>
            <!-- Spring Cloud BOM -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- Module Common dùng chung của dự án -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- 6. Quản lý Cấu hình Plugin tập trung (Plugin Management) -->
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <excludes>
                            <exclude>
                                <groupId>org.projectlombok</groupId>
                                <artifactId>lombok</artifactId>
                            </exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
```

---

## 2.4. Giải thích Chi tiết các Thẻ Cấu hình Root POM

### `<modelVersion>`
Luôn để `4.0.0` theo tiêu chuẩn Maven POM hiện tại.

### `<parent>`
Kế thừa `spring-boot-starter-parent` giúp tự động quản lý phiên bản cho hàng trăm thư viện phổ biến (Spring MVC, Hibernate, Jackson, Tomcat, Logback, JUnit 5...).

### `<groupId>`, `<artifactId>`, `<version>` (G.A.V)
- **`groupId`**: Tên tổ chức/công ty (ví dụ: `com.company.project`).
- **`artifactId`**: Định danh artifact (ví dụ: `microservices-parent`).
- **`version`**: Định danh phiên bản (`1.0.0-SNAPSHOT` đại diện cho phiên bản đang phát triển).

### `<packaging>pom</packaging>`
Thông báo cho Maven biết đây là một Root/Aggregator POM. Maven sẽ không đóng gói root thành file `.jar` hay `.war`.

### `<dependencyManagement>` vs `<dependencies>`
> [!NOTE]
> - **`<dependencyManagement>`**: Chỉ **định nghĩa và quản lý phiên bản** cho dependencies. Khai báo ở đây **KHÔNG** làm tự động thêm JAR vào classpath của các module con.
> - **`<dependencies>`**: Thêm trực tiếp JAR vào classpath. Nếu khai báo `<dependencies>` ở Root POM, tất cả module con đều bị ép buộc kế thừa thư viện đó.

### `<spring-cloud-dependencies>` BOM
Giúp quản lý phiên bản tương thích giữa Spring Boot 3 và các module Spring Cloud (Eureka, OpenFeign, Gateway, CircuitBreaker).

---

# PHẦN III: THIẾT LẬP CÁC MODULE CHI TIẾT

## 3.1. Thiết lập Module common & Cải tiến Quan trọng (Tránh Bẫy Plugin Build)

Module `common` chứa các mã nguồn dùng chung như: `ApiResponse<T>`, `BusinessException`, `ErrorCode`, Utilities.

Tạo thư mục và file `common/pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- Kế thừa từ Root Parent -->
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>common</artifactId>
    <packaging>jar</packaging>

    <name>common</name>
    <description>Common shared library for microservices</description>

    <dependencies>
        <!-- Spring Boot Starter Web để dùng HttpStatus, ResponseEntity nếu cần -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <!-- CẢI TIẾN QUAN TRỌNG: Cấu hình Build cho Thư viện Shared -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- QUAN TRỌNG: Disable repackage để module common đóng gói đúng chuẩn Plain JAR -->
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

> [!WARNING]
> **BẪY KINH ĐIỂN VỚI MODULE COMMON**:
> Mặc định, `spring-boot-maven-plugin` sẽ đóng gói file JAR thành dạng **Executable Fat JAR** (chứa các class trong `BOOT-INF/classes`). Khi một service khác (như `user-service`) phụ thuộc vào `common`, Maven sẽ **KHÔNG THỂ** import được các class từ `BOOT-INF/classes` và báo lỗi `Package com.example.common does not exist` khi compile!
> 
> **Giải pháp**: Trong `common/pom.xml`, bắt buộc phải thêm cấu hình `<skip>true</skip>` cho `spring-boot-maven-plugin` để `common` được đóng gói thành file JAR thư viện tiêu chuẩn.

---

### Mẫu Code trong Module `common`:

#### 1. File `ApiResponse.java`:
```java
package com.example.common.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    @Builder.Default
    private LocalDateTime timestamp = LocalDateTime.now();

    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
                .success(true)
                .message("Success")
                .data(data)
                .build();
    }

    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .build();
    }
}
```

#### 2. File `BusinessException.java`:
```java
package com.example.common.exception;

public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```

---

## 3.2. Thiết lập Microservice Đầu tiên: user-service

Cấu trúc `user-service/pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>user-service</artifactId>
    <packaging>jar</packaging>

    <name>user-service</name>

    <dependencies>
        <!-- Dependency dùng chung internal common -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
        </dependency>

        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Plugin sinh file Executable JAR cho user-service -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 3.3. Khởi tạo các Microservice Tiếp theo: order-service & notification-service

Cấu trúc `order-service/pom.xml` và `notification-service/pom.xml` tương tự như `user-service`. Mỗi service chỉ bổ sung các dependencies thực sự cần thiết (ví dụ `order-service` cần `spring-kafka` hoặc `postgresql`).

---

## 3.4. Chuẩn hóa Cấu trúc Package trong Microservice

Tránh dồn tất cả code vào các package phẳng ở cấp root (`controller/`, `service/`, `repository/`).

Khuyên dùng cấu trúc **Feature-based Packaging** hoặc **Layered by Feature**:

```text
user-service/
└── src/main/java/com/example/userservice/
    ├── UserServiceApplication.java             # Entry point
    │
    ├── config/                                 # App Configurations (Swagger, Security, Beans)
    ├── exception/                              # Global Exception Handler
    │
    └── user/                                   # Feature Package: User Management
        ├── controller/                         # REST Controllers
        │   └── UserController.java
        ├── service/                            # Service Interfaces & Implementations
        │   ├── UserService.java
        │   └── impl/UserServiceImpl.java
        ├── repository/                         # Spring Data JPA Repositories
        │   └── UserRepository.java
        ├── entity/                             # JPA Entities
        │   └── UserEntity.java
        └── dto/                                # Request/Response DTOs
            ├── CreateUserRequest.java
            └── UserResponse.java
```

---

# PHẦN IV: CẤU HÌNH, DATABASE & GIAO TIẾP GIỮA CÁC SERVICE

## 4.1. Quản lý File Cấu hình (application.yml & Profiles)

Mỗi microservice cần khai báo cổng (`server.port`) và tên ứng dụng (`spring.application.name`) riêng biệt trong file `src/main/resources/application.yml`.

### File `user-service/src/main/resources/application.yml`:
```yaml
server:
  port: 8081

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
    url: jdbc:postgresql://localhost:5432/user_db
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}
```

### Bảng phân bổ Port mặc định cho Dev Local:
| Service | Application Name | Port |
| :--- | :--- | :--- |
| **API Gateway** | `api-gateway` | `8080` |
| **User Service** | `user-service` | `8081` |
| **Order Service** | `order-service` | `8082` |
| **Notification Service** | `notification-service` | `8083` |

---

## 4.2. Giao tiếp giữa các Microservice (Synchronous REST/Feign vs Asynchronous Event-Driven)

```text
┌──────────────┐     Synchronous (REST / OpenFeign)     ┌──────────────┐
│ Order        ├───────────────────────────────────────►│ User         │
│ Service      │                                        │ Service      │
└──────┬───────┘                                        └──────────────┘
       │
       │ Asynchronous Event (OrderCreatedEvent)
       ▼
┌──────────────┐
│ Apache Kafka │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Notification │
│ Service      │
└──────────────┘
```

1. **Giao tiếp Đồng bộ (Synchronous)**: Dùng **Spring Cloud OpenFeign** hoặc `RestClient` khi `Order Service` cần gọi trực tiếp `User Service` để kiểm tra thông tin user trước khi tạo đơn hàng.
2. **Giao tiếp Bất đồng bộ (Asynchronous Event-Driven)**: Dùng **Apache Kafka** hoặc **RabbitMQ** khi `Order Service` bắn ra sự kiện `OrderCreatedEvent` để `Notification Service` lắng nghe và gửi email/push notification mà không gây blocking cho flow tạo đơn hàng.

---

## 4.3. Nguyên tắc Quản lý Dữ liệu (Database per Service)

> [!IMPORTANT]
> **Database per Service Pattern**: Mỗi Microservice sở hữu hoàn toàn cơ sở dữ liệu của chính nó. 
> - `user-service` kết nối tới `user_db`.
> - `order-service` kết nối tới `order_db`.
> - `user-service` **KHÔNG BAO GIỜ** được truy cập trực tiếp vào `order_db` hoặc ngược lại.

---

# PHẦN V: THAO TÁC MAVEN & CHUẨN HÓA QUY TRÌNH BUILD

## 5.1. Cơ chế Maven Reactor & Thứ tự Build

Khi bạn đứng ở thư mục Root và thực hiện lệnh build, **Maven Reactor** sẽ phân tích đồ thị phụ thuộc (Dependency Graph) giữa các module để quyết định thứ tự biên dịch:

```text
1. microservices-parent (Root POM)
2. common (Được biên dịch trước vì user-service & order-service phụ thuộc vào common)
3. user-service
4. order-service
5. notification-service
```

---

## 5.2. Các Câu lệnh Maven Cốt lõi trong Dự án Multi-module

| Mục tiêu lệnh | Lệnh thực thi |
| :--- | :--- |
| **Build toàn bộ dự án** | `./mvnw clean install` |
| **Build nhanh bỏ qua Test** | `./mvnw clean install -DskipTests` |
| **Chỉ build 1 module cụ thể** | `./mvnw clean install -pl user-service` |
| **Build 1 module VÀ các module phụ thuộc** | `./mvnw clean install -pl order-service -am` |
| **Chạy 1 Spring Boot Service** | `./mvnw spring-boot:run -pl user-service` |
| **Kiểm tra cây Dependency** | `./mvnw dependency:tree -pl user-service` |

- `-pl` (`--projects`): Chỉ định danh sách module muốn thao tác.
- `-am` (`--also-make`): Tự động build thêm các module phụ thuộc upstream.

---

## 5.3. Quản lý Dependency Scope & Dependency Hygiene

- **`compile`** *(Mặc định)*: Có mặt ở cả compile time và runtime classpath.
- **`provided`**: Cần để compile nhưng môi trường chạy sẽ cung cấp (ví dụ Lombok).
- **`runtime`**: Không cần để compile code, nhưng cần khi chạy ứng dụng (ví dụ Database Drivers: `postgresql`).
- **`test`**: Chỉ có hiệu lực khi biên dịch và chạy Unit/Integration Test (ví dụ `spring-boot-starter-test`).

---

# PHẦN VI: CONTAINERIZATION, DOCKER COMPOSE & PATTERNS NÂNG CAO

## 6.1. Dockerize Microservices & File docker-compose.yml Mẫu

### 1. File `user-service/Dockerfile` (Multi-stage Build tối ưu dung lượng):

```dockerfile
# Stage 1: Build JAR
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw clean package -pl user-service -am -DskipTests

# Stage 2: Runtime Image
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/user-service/target/user-service-*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. File `docker-compose.yml` tại Root Project:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: microservices-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgrespassword
      POSTGRES_MULTIPLE_DATABASES: user_db,order_db,notification_db
    volumes:
      - postgres_data:/var/lib/postgresql/data

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    container_name: microservices-kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_PROCESS_ROLES: 'broker,controller'
      KAFKA_CONTROLLER_QUORUM_VOTERS: '1@kafka:29093'
      KAFKA_LISTENERS: 'PLAINTEXT://0.0.0.0:29092,CONTROLLER://0.0.0.0:29093,PLAINTEXT_HOST://0.0.0.0:9092'
      KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'

volumes:
  postgres_data:
```

Chạy toàn bộ môi trường hạ tầng local bằng 1 lệnh:
```bash
docker compose up -d
```

---

## 6.2. Kiến trúc Nâng cao: API Gateway, Security & Observability

Khi hệ thống phát triển lớn hơn, dự án nên được mở rộng thêm các thành phần:

1. **API Gateway (Spring Cloud Gateway)**:
   - Điểm truy cập trung tâm (Single Entry Point) cho Clients.
   - Xử lý Routing, Rate Limiting, Authentication/Token Verification, CORS configuration.
2. **Security (Spring Security + OAuth2 / JWT)**:
   - Xác thực người dùng tại API Gateway hoặc chuyển Token tới các Service xử lý Authorization.
3. **Observability Stack**:
   - **Distributed Tracing**: Spring Boot Actuator + Micrometer + OpenTelemetry + Tempo/Jaeger.
   - **Metrics**: Micrometer + Prometheus + Grafana.
   - **Centralized Logging**: Logback + ELK Stack (Elasticsearch, Logstash, Kibana) hoặc Grafana Loki.

---

## 6.3. Distributed Transactions (Saga Pattern & Outbox Pattern) & Resilience

Trong môi trường Microservices với nhiều Database riêng biệt:

- **Saga Pattern**: Quản lý giao dịch phân tán qua một chuỗi các local transactions. Nếu 1 bước thất bại, Saga sẽ kích hoạt các **Compensating Transactions** (Giao dịch bù trừ) để hoàn tác dữ liệu trước đó.
- **Transactional Outbox Pattern**: Đảm bảo tính nhất quán giữa việc lưu Database và gửi Event lên Message Broker (Kafka) mà không bị rơi vào tình trạng "DB lưu thành công nhưng Kafka bị ngắt kết nối gây mất Event".
- **Resilience (Resilience4j)**: Áp dụng Circuit Breaker, Retry, Rate Limiter và TimeLimiter khi gọi các REST APIs inter-service.

---

# PHẦN VII: ROADMAP VÀ CHECKLIST THỰC HÀNH

## 7.1. Lộ trình Học tập & Phát triển Năng lực Java Backend Engineer

```text
Level 1: Maven Multi-module & Project Structure
   │
   ▼
Level 2: Spring Boot Core, REST APIs & Validation
   │
   ▼
Level 3: JPA, Hibernate, Flyway & Database Per Service
   │
   ▼
Level 4: Security (Spring Security, JWT, OAuth2)
   │
   ▼
Level 5: Event-Driven Architecture (Kafka, Redis Cache)
   │
   ▼
Level 6: System Design, API Gateway, Resilience & Patterns (Saga, Outbox)
   │
   ▼
Level 7: Docker, Kubernetes, CI/CD Pipelines & Observability
```

---

## 7.2. Checklist Setup Project từ A-Z

### Phase 1 — Khởi tạo Skeleton & Maven
- [x] Cài đặt JDK 21 và Maven 3.9+.
- [x] Tạo thư mục Root `microservices-parent`.
- [x] Chạy `mvn wrapper:wrapper` để tạo Maven Wrapper.
- [x] Cấu hình Root `pom.xml` với `<packaging>pom</packaging>`, Spring Boot Parent 3.3.x, và Spring Cloud BOM.
- [x] Tạo module `common` với cấu hình plugin `<skip>true</skip>`.
- [x] Tạo các modules `user-service`, `order-service`, `notification-service`.

### Phase 2 — Mã nguồn & Cấu hình App
- [x] Tạo Main Application Class cho từng Service.
- [x] Cấu hình file `application.yml` cho từng Service với Port và Application Name riêng biệt.
- [x] Kiểm tra lệnh build toàn hệ thống: `./mvnw clean install`.
- [x] Chạy thử một service: `./mvnw spring-boot:run -pl user-service`.

### Phase 3 — Hạ tầng Local & Continuous Integration
- [x] Viết file `docker-compose.yml` chạy PostgreSQL và Kafka local.
- [x] Tạo `Dockerfile` cho các microservices.
- [x] Thiết lập file `.gitignore` bỏ qua `target/`, `.idea/`, `.env`.

---

## 7.3. 10 Quy tắc Vàng (Core Rules) & Kết luận

> [!TIP]
> 1. **Root POM luôn là `<packaging>pom</packaging>`** và không chứa code Java.
> 2. **Dùng `<dependencyManagement>` ở Root POM** để quản lý thống nhất phiên bản thư viện.
> 3. **Module `common` bắt buộc disable repackage plugin** (`<skip>true</skip>`) để các service khác có thể import.
> 4. **Mỗi Microservice là một ứng dụng độc lập** sở hữu Database riêng biệt.
> 5. **Không bao giờ phụ thuộc chéo trực tiếp (Java Class Dependency)** giữa các Microservice nghiệp vụ.
> 6. **Module `common` chỉ chứa mã nguồn thực sự generic**, không biến `common` thành "thùng rác chứa code".
> 7. **Dùng Maven Wrapper (`./mvnw`)** thay vì phụ thuộc vào Maven toàn cục trên máy dev.
> 8. **Không bao giờ hard-code Secret/Password** trong source code; sử dụng Environment Variables.
> 9. **Mỗi Microservice phải có bộ Unit & Integration Test riêng biệt**.
> 10. **Không over-engineer ngay từ đầu**: Bắt đầu đơn giản với Monorepo Multi-module trước khi phân tách thành các hạ tầng phức tạp.

---

### Kết luận

Mô hình **Maven Multi-module Monorepo** là giải pháp lý tưởng giúp bạn dễ dàng quản lý, biên dịch và phát triển hệ thống Microservices bằng Java Spring Boot. Bằng việc tuân thủ các quy chuẩn cấu hình Parent POM, phân tách module hợp lý và quản lý dependencies tập trung, bạn sẽ xây dựng được một nền tảng mã nguồn sạch, dễ mở rộng và sẵn sàng cho môi trường sản xuất.
