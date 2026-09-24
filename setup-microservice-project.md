# Hướng Dẫn Setup Dự Án Java Spring Boot Microservices — Hệ Thống Uber App (Maven Multi-Module)

> **Tài liệu hướng dẫn từ A-Z**: Xây dựng kiến trúc hệ thống gọi xe công nghệ (**Uber App Clone**) theo mô hình **Maven Multi-module Monorepo** với **Spring Boot 3**, **Java 17**, tích hợp thư viện dùng chung **`common`**, lưu trữ toạ độ tài xế thời gian thực với **Redis GeoSpatial**, quản lý chuyến đi với **MySQL/JPA**, và xử lý sự kiện bất đồng bộ qua **Apache Kafka**.

---

## 📋 MỤC LỤC

- [PHẦN I: TỔNG QUAN & KIẾN TRÚC PROJECT UBER-APP](#phần-i-tổng-quan--kiến-trúc-project-uber-app)
  - [1.1. Mục tiêu & Cấu trúc Thư mục](#11-mục-tiêu--cấu-trúc-thư-mục)
  - [1.2. Phân biệt Khái niệm: Parent POM vs Aggregator POM](#12-phân-biệt-khái-niệm-parent-pom-vs-aggregator-pom)
  - [1.3. Mô hình Kiến trúc Maven & Luồng Dữ liệu Tổng thể](#13-mô-hình-kiến-trúc-maven--luồng-dữ-liệu-tổng-thể)
- [PHẦN II: CHUẨN BỊ MÔI TRƯỜNG & KHỞI TẠO ROOT PROJECT](#phần-ii-chuẩn-bị-môi-trường--khởi-tạo-root-project)
  - [2.1. Yêu cầu Môi trường (Prerequisites)](#21-yêu-cầu-môi-trường-prerequisites)
  - [2.2. Khởi tạo Root Project & Maven Wrapper](#22-khởi-tạo-root-project--maven-wrapper)
  - [2.3. Cấu hình Root pom.xml Chuẩn hóa (Parent & Aggregator POM)](#23-cấu-hình-root-pomxml-chuẩn-hóa-parent--aggregator-pom)
  - [2.4. Giải thích Chi tiết các Thẻ Cấu hình Root POM](#24-giải-thích-chi-tiết-các-thẻ-cấu-hình-root-pom)
- [PHẦN III: THIẾT LẬP CÁC MODULE CHI TIẾT & KẾT HỢP MODULE COMMON](#phần-iii-thiết-lập-các-module-chi-tiết--kết-hợp-module-common)
  - [3.1. Thiết lập Module common (Plain JAR Library) & Các Model/Event Dùng Chung](#31-thiết-lập-module-common-plain-jar-library--các-modelevent-dùng-chung)
  - [3.2. Thiết lập Module location-service (Redis GeoSpatial)](#32-thiết-lập-module-location-service-redis-geospatial)
  - [3.3. Thiết lập Module matching-service (Kafka Event Engine)](#33-thiết-lập-module-matching-service-kafka-event-engine)
  - [3.4. Thiết lập Module ride-service (Core Business & MySQL)](#34-thiết-lập-module-ride-service-core-business--mysql)
  - [3.5. Chuẩn hóa Cấu trúc Package trong Microservice (Ví dụ ride-service)](#35-chuẩn-hóa-cấu-trúc-package-trong-microservice-ví-dụ-ride-service)
- [PHẦN IV: CẤU HÌNH, DATABASE & GIAO TIẾP GIỮA CÁC SERVICE](#phần-iv-cấu-hình-database--giao-tiếp-giữa-các-service)
  - [4.1. Quản lý File Cấu hình (application.yml & Profiles)](#41-quản-lý-file-cấu-hình-applicationyml--profiles)
  - [4.2. Luồng Giao tiếp Giữa các Service trong Uber App Flow](#42-luồng-giao-tiếp-giữa-các-service-trong-uber-app-flow)
  - [4.3. Nguyên tắc Quản lý Dữ liệu (Database per Service)](#43-nguyên-tắc-quản-lý-dữ-liệu-database-per-service)
- [PHẦN V: THAO TÁC MAVEN & CHUẨN HÓA QUY TRÌNH BUILD](#phần-v-thao-tác-maven--chuẩn-hóa-quy-trình-build)
  - [5.1. Cơ chế Maven Reactor & Thứ tự Build Hệ thống](#51-cơ-chế-maven-reactor--thứ-tự-build-hệ-thống)
  - [5.2. Các Câu lệnh Maven Cốt lõi trong Dự án Multi-module](#52-các-câu-lệnh-maven-cốt-lõi-trong-dự-án-multi-module)
  - [5.3. Quản lý Dependency Scope & Dependency Hygiene]    (#53-quản-lý-dependency-scope--dependency-hygiene)
- [PHẦN VI: CÁCH CHẠY DỰ ÁN (RUNNING GUIDE)](#phần-vi-cách-chạy-dự-án-running-guide)
  - [6.1. Các Bước Chuẩn Bị Bắt Buộc (Bật Hạ Tầng & Build Common)](#61-các-bước-chuẩn-bị-bắt-buộc-bật-hạ-tầng--build-common)
  - [6.2. Cách 1: Chạy Bằng IntelliJ IDEA Services Dashboard (Khuyên Dùng Cho Dev)](#62-cách-1-chạy-bằng-intellij-idea-services-dashboard-khuyên-dùng-cho-dev)
  - [6.3. Cách 2: Chạy Bằng Dòng Lệnh Maven Wrapper (Terminal)](#63-cách-2-chạy-bằng-dòng-lệnh-maven-wrapper-terminal)
  - [6.4. Cách 3: Đóng Gói Fat JAR & Chạy Bằng java -jar](#64-cách-3-đóng-gói-fat-jar--chạy-bằng-java--jar)
  - [6.5. Thứ Tự Khởi Động Khuyến Nghị & Kiểm Tra Trạng Thái Sống (Health Check)](#65-thứ-tự-khởi-động-khuyến-nghị--kiểm-tra-trạng-thái-sống-health-check)
- [PHẦN VII: CONTAINERIZATION, DOCKER COMPOSE & PATTERNS NÂNG CAO](#phần-vii-containerization-docker-compose--patterns-nâng-cao)
  - [7.1. Dockerize Microservices & File docker-compose.yml Thực tế](#71-dockerize-microservices--file-docker-composeyml-thực-tế)
  - [7.2. Kiến trúc Mở rộng: API Gateway, Security & Observability](#72-kiến-trúc-mở-rộng-api-gateway-security--observability)
  - [7.3. Distributed Transactions (Saga & Outbox Pattern) trong Đặt xe](#73-distributed-transactions-saga--outbox-pattern-trong-đặt-xe)
- [PHẦN VIII: ROADMAP VÀ CHECKLIST THỰC HÀNH](#phần-viii-roadmap-và-checklist-thực-hành)
  - [8.1. Lộ trình Học tập & Phát triển Năng lực Java Backend Engineer](#81-lộ-trình-học-tập--phát-triển-năng-lực-java-backend-engineer)
  - [8.2. Checklist Setup Project từ A-Z](#82-checklist-setup-project-từ-a-z)
  - [8.3. 10 Quy tắc Vàng (Core Rules) & Kết luận](#83-10-quy-tắc-vàng-core-rules--kết-luận)

---

# PHẦN I: TỔNG QUAN & KIẾN TRÚC PROJECT UBER-APP

## 1.1. Mục tiêu & Cấu trúc Thư mục

Hệ thống Uber App được thiết kế theo mô hình **Maven Multi-module Monorepo** với **Spring Boot 3.3.4** và **Java 17**.

Cấu trúc cây thư mục của dự án `uber-app`:

```text
uber-app/
├── .mvn/                               # Cấu hình Maven Wrapper
├── mvnw                                # Script chạy Maven trên Linux/macOS
├── mvnw.cmd                            # Script chạy Maven trên Windows
├── pom.xml                             # Root Parent & Aggregator POM
├── docker-compose.yml                  # Môi trường chạy Local (MySQL, Redis, Kafka, Zookeeper)
│
├── common/                             # Thư viện dùng chung (DTO, ApiResponse, Kafka Events, Exceptions)
│   ├── pom.xml
│   └── src/main/java/com/cns/lg/common/
│       ├── dto/                        # ApiResponse<T>, DriverLocationDTO
│       ├── event/                      # RideRequestEvent, MatchFoundEvent
│       └── exception/                  # BusinessException, GlobalExceptionHandler
│
├── location-service/                   # Service quản lý vị trí tài xế theo thời gian thực (Port 8081)
│   ├── pom.xml                         # Phụ thuộc: common, spring-boot-starter-web, redis
│   └── src/main/java/com/cns/lg/locationservice/
│
├── matching-service/                   # Service tính toán & ghép nối tài xế với khách hàng (Port 8082)
│   ├── pom.xml                         # Phụ thuộc: common, spring-boot-starter-web, spring-kafka
│   └── src/main/java/com/cns/lg/matchingservice/
│
└── ride-service/                       # Service quản lý vòng đời cuốc xe (Port 8083)
    ├── pom.xml                         # Phụ thuộc: common, spring-boot-starter-web, jpa, mysql, kafka
    └── src/main/java/com/cns/lg/rideservice/
```

### Vai trò của từng Module trong Hệ thống Uber:
1. **`uber-app` (Root POM)**: Đóng vai trò vừa là **Parent POM** (quản lý phiên bản dependencies, plugin) vừa là **Aggregator POM** (chứa danh sách tất cả các module con để build đồng loạt).
2. **`common`**: Đóng gói dạng thư viện Plain JAR tiêu chuẩn (`.jar`). Chứa các cấu trúc dữ liệu dùng chung (DTO kết quả phản hồi `ApiResponse<T>`, các Kafka Event payload như `RideRequestEvent`, `MatchFoundEvent`). Cả 3 service con đều kế thừa và sử dụng module này.
3. **`location-service`**: Nhận cập nhật toạ độ GPS (latitude, longitude) từ ứng dụng tài xế (Driver App), lưu trữ vào Redis bằng dữ liệu không gian địa lý **Redis GeoSpatial** (`GEOADD`, `GEORADIUS`) để truy vấn tài xế lân cận với tốc độ tính bằng mili-giây.
4. **`matching-service`**: Lắng nghe sự kiện khách đặt xe từ Kafka topic `ride-requests`, truy vấn các tài xế gần nhất, thực hiện thuật toán tìm tài xế phù hợp và bắn sự kiện `MatchFoundEvent` sang Kafka topic `ride-matched`.
5. **`ride-service`**: Core Service tiếp nhận yêu cầu đặt xe từ hành khách (Rider App), lưu trạng thái chuyến đi (`REQUESTED`, `MATCHED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`) vào cơ sở dữ liệu **MySQL**, và kích hoạt quy trình tìm xe bằng cách gửi event lên Kafka.

---

## 1.2. Phân biệt Khái niệm: Parent POM vs Aggregator POM

Trong Maven Monorepo của dự án Uber:

### 1. Parent POM (Kế thừa - Inheritance)
Đóng vai trò như "lớp cha" cung cấp cấu hình dùng chung mà các module con (`common`, `location-service`, `matching-service`, `ride-service`) kế thừa:
- Kế thừa Spring Boot Parent `3.3.4`.
- Khai báo phiên bản Java đồng bộ: `<java.version>17</java.version>`.
- Quản lý phiên bản thư viện trung tâm qua `<dependencyManagement>` (ví dụ `spring-kafka`, `spring-kafka-test`, và chính module `common`).
- Quản lý cấu hình Plugin chung qua `<pluginManagement>` (ví dụ loại bỏ Lombok khỏi Executable JAR).

### 2. Aggregator POM (Gom nhóm - Aggregation)
Đóng vai trò gom nhóm các module để khi đứng tại thư mục root `uber-app`, bạn chỉ cần gõ một lệnh duy nhất `mvn clean install` là Maven tự động build toàn bộ 4 module theo đúng thứ tự logic:
```xml
<modules>
    <module>common</module>
    <module>location-service</module>
    <module>matching-service</module>
    <module>ride-service</module>
</modules>
```

### 3. Sơ đồ Kế thừa & Gom nhóm trong `uber-app`
```text
                         Root POM (uber-app)
                 ┌─────────────────┴─────────────────┐
                 │                                   │
           Parent Component                   Aggregator Component
     (Quản lý Versions & Plugins)         (Khai báo danh sách Modules)
                 │                                   │
      ┌──────────┼──────────────┬──────────────┐     │
      │          │              │              │     │
    common  location-service matching-service ride-service
      ▲          ▲              ▲              ▲     │
      │          │              │              │     │
      └──────────┴──────────────┴──────────────┴─────┘
                 Kế thừa cấu hình từ Root POM
                 
      * ĐẶC BIỆT: location-service, matching-service, ride-service
                  đều IMPORT VÀ SỬ DỤNG module "common"
```

---

## 1.3. Mô hình Kiến trúc Maven & Luồng Dữ liệu Tổng thể

> [!IMPORTANT]
> **Quy tắc quan trọng**: 
> 1. Root POM `uber-app` tuyệt đối **KHÔNG** chứa bất kỳ file code Java nào và **KHÔNG** phải là một ứng dụng Spring Boot. Root POM bắt buộc phải đặt `<packaging>pom</packaging>`.
> 2. Module `common` chỉ là thư viện dùng chung, **KHÔNG** được đóng gói thành Spring Boot Fat JAR mà phải cấu hình `<skip>true</skip>` ở plugin để đóng gói thành Plain JAR tiêu chuẩn.

---

# PHẦN II: CHUẨN BỊ MÔI TRƯỜNG & KHỞI TẠO ROOT PROJECT

## 2.1. Yêu cầu Môi trường (Prerequisites)

- **JDK 17** (hoặc JDK 21 tương thích chuẩn Spring Boot 3.x).
- **Maven 3.9+**.
- **Docker Desktop** (đang bật để chạy Redis, MySQL, Kafka, Zookeeper).
- **IDE**: IntelliJ IDEA hoặc VS Code.

Kiểm tra phiên bản Java và Maven trên máy:
```bash
java -version
mvn -version
```

---

## 2.2. Khởi tạo Root Project & Maven Wrapper

Tại thư mục làm việc, tạo thư mục gốc cho dự án:
```bash
mkdir uber-app
cd uber-app
```

Tạo **Maven Wrapper** để đảm bảo tất cả máy phát triển và CI/CD dùng đúng phiên bản Maven đồng nhất:
```bash
mvn wrapper:wrapper
```

Sau khi chạy lệnh, dự án sẽ sinh ra các file `.mvn/`, `mvnw` (Linux/macOS) và `mvnw.cmd` (Windows).

---

## 2.3. Cấu hình Root pom.xml Chuẩn hóa (Parent & Aggregator POM)

File `uber-app/pom.xml` được cấu hình chuẩn mực với đầy đủ chú thích tiếng Việt:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- =========================================== -->
    <!-- 1. Kế thừa Spring Boot Starter Parent       -->
    <!-- =========================================== -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.4</version>
        <relativePath/> <!-- Tìm parent trực tiếp từ Maven Central Repository -->
    </parent>

    <!-- =========================================== -->
    <!-- 2. Thông tin định danh Root Project (G.A.V) -->
    <!-- =========================================== -->
    <groupId>com.cns.lg</groupId>
    <artifactId>uber-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <!-- Bắt buộc: Root POM phải là packaging pom -->
    <packaging>pom</packaging>

    <name>uber-app</name>
    <description>Root Parent & Aggregator POM for Uber App Microservices</description>

    <!-- =========================================== -->
    <!-- 3. Danh sách các Module con (Aggregator)    -->
    <!-- =========================================== -->
    <modules>
        <module>common</module>
        <module>location-service</module>
        <module>matching-service</module>
        <module>ride-service</module>
    </modules>

    <!-- =========================================== -->
    <!-- 4. Properties cấu hình dùng chung           -->
    <!-- =========================================== -->
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <!-- Thư viện Spring Kafka cho Event-Driven -->
        <spring-kafka.version>3.2.4</spring-kafka.version>
    </properties>

    <!-- =========================================== -->
    <!-- 5. Quản lý phiên bản tập trung              -->
    <!--    (Không tự động thêm vào classpath)       -->
    <!-- =========================================== -->
    <dependencyManagement>
        <dependencies>

            <!-- Module Common dùng chung của dự án Uber -->
            <dependency>
                <groupId>com.cns.lg</groupId>
                <artifactId>common</artifactId>
                <version>${project.version}</version>
            </dependency>

            <!-- Spring Kafka -->
            <dependency>
                <groupId>org.springframework.kafka</groupId>
                <artifactId>spring-kafka</artifactId>
                <version>${spring-kafka.version}</version>
            </dependency>

            <!-- Spring Kafka Test: Hỗ trợ EmbeddedKafka khi viết test -->
            <dependency>
                <groupId>org.springframework.kafka</groupId>
                <artifactId>spring-kafka-test</artifactId>
                <version>${spring-kafka.version}</version>
                <scope>test</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>

    <!-- =========================================== -->
    <!-- 6. Quản lý cấu hình Plugin tập trung        -->
    <!-- =========================================== -->
    <build>
        <pluginManagement>
            <plugins>

                <!-- spring-boot-maven-plugin: Loại trừ Lombok khỏi file Fat JAR -->
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

                <!-- maven-compiler-plugin: Kích hoạt annotation processor cho Lombok -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <annotationProcessorPaths>
                            <path>
                                <groupId>org.projectlombok</groupId>
                                <artifactId>lombok</artifactId>
                            </path>
                        </annotationProcessorPaths>
                    </configuration>
                </plugin>

            </plugins>
        </pluginManagement>
    </build>

</project>
```

---

## 2.4. Giải thích Chi tiết các Thẻ Cấu hình Root POM

### 1. `<modelVersion>`
* **Giá trị**: Luôn là `4.0.0`.
* **Ý nghĩa**: Định nghĩa phiên bản của cấu trúc mô hình đối tượng Maven POM (Project Object Model).

### 2. `<parent>` (Kế thừa Spring Boot Starter Parent)
* **Tác dụng**:
  1. **Dependency Management tự động**: Quản lý sẵn phiên bản chuẩn của hàng trăm thư viện (Spring Web, Spring Data JPA, Redis, MySQL Connector, Jackson, SLF4J, JUnit 5...). Các service con khi khai báo các thư viện này **không cần ghi thẻ `<version>`**.
  2. **Plugin Management mặc định**: Cấu hình sẵn `spring-boot-maven-plugin` để đóng gói Executable Fat JAR.
  3. **Cấu hình compiler chuẩn**: Tự động áp dụng mã hóa UTF-8 và phiên bản Java tương thích.
* **`<relativePath/>` (để trống)**: Bắt buộc Maven bỏ qua tìm kiếm trên máy cục bộ và tải trực tiếp Parent POM từ **Maven Central Repository**.

### 3. Thông tin Định danh G.A.V (`groupId`, `artifactId`, `version`)
- **`groupId`**: `com.cns.lg` — Định danh tổ chức / công ty.
- **`artifactId`**: `uber-app` — Tên đại diện cho Root Monorepo.
- **`version`**: `1.0.0-SNAPSHOT` — Phiên bản đang trong giai đoạn phát triển.

### 4. `<packaging>pom</packaging>`
Bắt buộc đối với Root POM. Maven hiểu đây là POM cha gom nhóm và điều phối cấu hình, sẽ không biên dịch ra file `.jar` tại root.

### 5. `<modules>` (Aggregator Component)
Liệt kê danh sách các thư mục module con (`common`, `location-service`, `matching-service`, `ride-service`) để phục vụ lệnh build toàn bộ dự án từ root.

### 6. `<dependencyManagement>` vs `<dependencies>`
- **`<dependencyManagement>`**: **Chỉ định nghĩa phiên bản chuẩn** cho các thư viện và module nội bộ (`common`, `spring-kafka`). Thẻ này **KHÔNG** tự động add thư viện vào classpath của service con. Module con nào cần dùng thì tự khai báo lại trong thẻ `<dependencies>` của riêng nó mà **không cần ghi `<version>`**.
- **`<dependencies>`**: Thêm trực tiếp thư viện vào classpath. Không nên đặt bừa bãi ở Root POM vì sẽ ép buộc mọi service con phải kế thừa (kể cả service không cần dùng).

### 7. `<pluginManagement>`
Định nghĩa sẵn cấu hình mẫu cho các plugin Maven (ví dụ: loại bỏ Lombok khi build JAR). Cấu hình này chỉ kích hoạt khi service con gọi plugin tương ứng.

---

# PHẦN III: THIẾT LẬP CÁC MODULE CHI TIẾT & KẾT HỢP MODULE COMMON

## 3.1. Thiết lập Module common (Plain JAR Library) & Các Model/Event Dùng Chung

Module `common` là xương sống dùng chung cho toàn bộ hệ thống Uber:
- Chứa wrapper response chuẩn `ApiResponse<T>`.
- Chứa các đối tượng sự kiện Kafka Payload: `RideRequestEvent`, `MatchFoundEvent`.
- Chứa DTO toạ độ tài xế `DriverLocationDTO`.
- Chứa lớp ngoại lệ nghiệp vụ `BusinessException`.

### File `common/pom.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- =========================================== -->
    <!-- Kế thừa từ Root Parent POM của dự án        -->
    <!-- =========================================== -->
    <parent>
        <groupId>com.cns.lg</groupId>
        <artifactId>uber-app</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>common</artifactId>
    <packaging>jar</packaging>
    <name>common</name>
    <description>Common shared library for Uber App Microservices (DTOs, Events, Exceptions)</description>

    <!-- =========================================== -->
    <!-- Dependencies dùng chung cho common          -->
    <!-- =========================================== -->
    <dependencies>

        <!-- Spring Web: Sử dụng HttpStatus, ResponseEntity nếu cần -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Jackson Databind: Serialize/Deserialize JSON khi gửi Kafka Event -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

        <!-- Lombok: Tự sinh Getter, Setter, Builder -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

    </dependencies>

    <!-- =========================================== -->
    <!-- CẤU HÌNH QUAN TRỌNG: Đóng gói Plain JAR     -->
    <!-- =========================================== -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- BẮT BUỘC: skip repackage để tạo Plain JAR cho các service khác import -->
                    <skip>true</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

> [!WARNING]
> **BẪY KINH ĐIỂN VỚI MODULE COMMON**:
> Nếu không có cấu hình `<skip>true</skip>` ở `spring-boot-maven-plugin`, Maven sẽ đóng gói `common` thành **Executable Fat JAR** (các class bị giấu trong thư mục `BOOT-INF/classes`). Khi `ride-service` hay `matching-service` gọi `import com.cns.lg.common...`, Maven sẽ báo lỗi:
> `[ERROR] Package com.cns.lg.common does not exist`.

---

### Các Class Mẫu trong Module `common`:

#### 1. Wrapper chuẩn phản hồi API — `ApiResponse.java`:
```java
package com.cns.lg.common.dto;

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

#### 2. Sự kiện Khách Đặt Xe gửi qua Kafka — `RideRequestEvent.java`:
```java
package com.cns.lg.common.event;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class RideRequestEvent implements Serializable {
    private Long rideId;
    private Long passengerId;
    private Double pickupLatitude;
    private Double pickupLongitude;
    private Double dropoffLatitude;
    private Double dropoffLongitude;
    private BigDecimal estimatedFare;
    @Builder.Default
    private LocalDateTime requestedAt = LocalDateTime.now();
}
```

#### 3. Sự kiện Ghép Xe Thành Công gửi qua Kafka — `MatchFoundEvent.java`:
```java
package com.cns.lg.common.event;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.time.LocalDateTime;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class MatchFoundEvent implements Serializable {
    private Long rideId;
    private Long driverId;
    private String driverName;
    private String licensePlate;
    private Double driverLatitude;
    private Double driverLongitude;
    private Integer estimatedArrivalMinutes;
    @Builder.Default
    private LocalDateTime matchedAt = LocalDateTime.now();
}
```

#### 4. DTO Vị trí Tài xế — `DriverLocationDTO.java`:
```java
package com.cns.lg.common.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class DriverLocationDTO {
    private Long driverId;
    private Double latitude;
    private Double longitude;
    private Boolean isAvailable;
}
```

---

## 3.2. Thiết lập Module location-service (Redis GeoSpatial)

Module chịu trách nhiệm tiếp nhận toạ độ tài xế thời gian thực và lưu trữ vào Redis GeoSpatial.

### File `location-service/pom.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- =========================================== -->
    <!-- Kế thừa từ Root Parent POM của dự án        -->
    <!-- =========================================== -->
    <parent>
        <groupId>com.cns.lg</groupId>
        <artifactId>uber-app</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>location-service</artifactId>
    <name>location-service</name>
    <description>
        Location Service - Quản lý và theo dõi vị trí tài xế theo thời gian thực.
        Sử dụng Redis GeoSpatial để lưu trữ và truy vấn toạ độ (kinh độ, vĩ độ) với độ trễ thấp.
    </description>

    <!-- =========================================== -->
    <!-- Dependencies riêng của location-service     -->
    <!-- (version kế thừa từ Root POM / Spring BOM)  -->
    <!-- =========================================== -->
    <dependencies>

        <!-- KẾT HỢP: Module common dùng chung (DTO, ApiResponse) -->
        <dependency>
            <groupId>com.cns.lg</groupId>
            <artifactId>common</artifactId>
        </dependency>

        <!-- Spring Web MVC: Cung cấp REST API cho Driver App đẩy toạ độ GPS -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Data Redis: Thao tác Redis GeoSpatial (GEOADD, GEORADIUS) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- Actuator: Giám sát trạng thái hoạt động (Health check & Metrics) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Lombok: Giảm boilerplate code (getter/setter/builder...) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- DevTools: Hỗ trợ tự động reload khi sửa code trong môi trường dev -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Test: Bộ test chuẩn (JUnit 5, Mockito, MockMvc) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <!-- =========================================== -->
    <!-- Build: Kích hoạt plugin từ Root pluginMgmt  -->
    <!-- =========================================== -->
    <build>
        <plugins>
            <!-- Đóng gói thành Executable Fat JAR để chạy độc lập -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- Compiler với Lombok processor từ Root POM -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 3.3. Thiết lập Module matching-service (Kafka Event Engine)

Module này đóng vai trò bộ não ghép xe:
- Lắng nghe Kafka Topic `ride-requests` phát ra từ `ride-service`.
- Gọi sang `location-service` để lấy danh sách tài xế trong bán kính 3-5km.
- Chọn tài xế tối ưu và phát sự kiện `MatchFoundEvent` vào Kafka Topic `ride-matched`.

### File `matching-service/pom.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- =========================================== -->
    <!-- Kế thừa từ Root Parent POM của dự án        -->
    <!-- =========================================== -->
    <parent>
        <groupId>com.cns.lg</groupId>
        <artifactId>uber-app</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>matching-service</artifactId>
    <name>matching-service</name>
    <description>
        Matching Service - Ghép cặp hành khách với tài xế gần nhất.
        Tiêu thụ sự kiện từ Kafka (ride-requests), truy vấn Location Service để tìm tài xế,
        và phát sự kiện ghép cặp thành công (ride-matched) về Ride Service.
    </description>

    <!-- =========================================== -->
    <!-- Dependencies riêng của matching-service     -->
    <!-- (version kế thừa từ Root POM / Spring BOM)  -->
    <!-- =========================================== -->
    <dependencies>

        <!-- KẾT HỢP: Module common dùng chung (RideRequestEvent, MatchFoundEvent) -->
        <dependency>
            <groupId>com.cns.lg</groupId>
            <artifactId>common</artifactId>
        </dependency>

        <!-- Spring Web MVC: REST API nội bộ và gọi sang location-service -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Kafka: Lắng nghe ride-requests và gửi match-found-event -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <!-- Actuator: Health check & Metrics giám sát trạng thái service -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Lombok: Tối ưu hóa code DTO, Service -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- DevTools: Tự động hot reload khi code -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Test: Bộ test chuẩn -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Kafka Test: Kiểm thử giao tiếp Kafka (EmbeddedKafka) -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <!-- =========================================== -->
    <!-- Build: Kích hoạt plugin từ Root pluginMgmt  -->
    <!-- =========================================== -->
    <build>
        <plugins>
            <!-- Đóng gói thành Executable Fat JAR -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- Compiler với Lombok processor từ Root POM -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 3.4. Thiết lập Module ride-service (Core Business & MySQL)

Module chịu trách nhiệm xử lý toàn bộ vòng đời cuốc xe:
- Nhận yêu cầu tạo chuyến từ khách hàng, validate dữ liệu đầu vào.
- Lưu cuốc xe vào cơ sở dữ liệu **MySQL** với trạng thái ban đầu là `REQUESTED`.
- Bắn sự kiện `RideRequestEvent` lên Kafka topic `ride-requests`.
- Lắng nghe Kafka topic `ride-matched` từ `matching-service` để cập nhật trạng thái cuốc xe thành `MATCHED`, gán mã tài xế và biển số xe.

### File `ride-service/pom.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- =========================================== -->
    <!-- Kế thừa từ Root Parent POM của dự án        -->
    <!-- =========================================== -->
    <parent>
        <groupId>com.cns.lg</groupId>
        <artifactId>uber-app</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>ride-service</artifactId>
    <name>ride-service</name>
    <description>
        Ride Service - Quản lý vòng đời toàn bộ chuyến đi (Ride lifecycle).
        Xử lý: tạo yêu cầu đặt xe -> chờ ghép cặp -> xác nhận tài xế -> đang đón
        -> đang di chuyển -> hoàn thành/huỷ chuyến đi.
        Lưu trữ bền vững vào MySQL qua JPA/Hibernate, giao tiếp async qua Kafka.
    </description>

    <!-- =========================================== -->
    <!-- Dependencies riêng của ride-service         -->
    <!-- (version kế thừa từ Root POM / Spring BOM)  -->
    <!-- =========================================== -->
    <dependencies>

        <!-- KẾT HỢP: Module common dùng chung (ApiResponse, RideRequestEvent, MatchFoundEvent) -->
        <dependency>
            <groupId>com.cns.lg</groupId>
            <artifactId>common</artifactId>
        </dependency>

        <!-- Spring Web MVC: REST API cho Client (Rider App / Driver App) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Data JPA: Tương tác với cơ sở dữ liệu qua ORM Hibernate -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Spring Kafka: Bắn event đặt xe và lắng nghe event ghép xe thành công -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <!-- Spring Validation: Kiểm tra tính hợp lệ của toạ độ, số tiền, thông tin đặt xe -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Actuator: Endpoint kiểm tra tình trạng kết nối DB, Kafka, liveness/readiness -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- MySQL Connector: Driver kết nối cơ sở dữ liệu MySQL 8.0 -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok: Tự sinh getter/setter/builder/log cho Entity & Service -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- DevTools: Hỗ trợ hot reload trong môi trường phát triển -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Test: Bộ test chuẩn -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- Kafka Test: Kiểm thử tích hợp với Kafka Consumer/Producer -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <!-- =========================================== -->
    <!-- Build: Kích hoạt plugin từ Root pluginMgmt  -->
    <!-- =========================================== -->
    <build>
        <plugins>
            <!-- Đóng gói thành Executable Fat JAR -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- Compiler với Lombok processor từ Root POM -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 3.5. Chuẩn hóa Cấu trúc Package trong Microservice (Ví dụ `ride-service`)

Khuyên dùng cấu trúc tổ chức mã nguồn chuẩn doanh nghiệp:

```text
ride-service/
└── src/main/java/com/cns/lg/rideservice/
    ├── RideServiceApplication.java             # Entry point (@SpringBootApplication)
    │
    ├── config/                                 # Cấu hình KafkaProducer, KafkaConsumer, MySQL JPA
    │   ├── KafkaTopicConfig.java
    │   └── KafkaProducerConfig.java
    │
    ├── controller/                             # REST API cho client gọi
    │   └── RideController.java                 # POST /api/v1/rides (Đặt xe)
    │
    ├── service/                                # Nghiệp vụ xử lý cuốc xe
    │   ├── RideService.java
    │   └── impl/RideServiceImpl.java
    │
    ├── repository/                             # Spring Data JPA Repository
    │   └── RideRepository.java
    │
    ├── entity/                                 # MySQL Table Entity
    │   └── Ride.java
    │
    ├── kafka/                                  # Tương tác với Message Broker
    │   ├── RideKafkaProducer.java              # Bắn RideRequestEvent
    │   └── RideMatchedConsumer.java            # Lắng nghe MatchFoundEvent
    │
    └── dto/                                    # DTO nội bộ của Ride Service
        ├── CreateRideRequest.java
        └── RideResponse.java
```

---

# PHẦN IV: CẤU HÌNH, DATABASE & GIAO TIẾP GIỮA CÁC SERVICE

## 4.1. Quản lý File Cấu hình (application.yml & Profiles)

Mỗi service sở hữu cổng mạng riêng để tránh xung đột khi chạy cùng lúc trên máy dev:

### 1. Phân bổ Port mặc định:
| Service | Cổng (Port) | Công nghệ / Database | Vai trò |
| :--- | :--- | :--- | :--- |
| **`location-service`** | `8081` | Spring Boot + Redis Geo | Quản lý toạ độ tài xế |
| **`matching-service`** | `8082` | Spring Boot + Kafka | Khớp nối khách & tài xế |
| **`ride-service`** | `8083` | Spring Boot + MySQL + Kafka | Quản lý vòng đời cuốc xe |
| **API Gateway** *(Mở rộng)* | `8080` | Spring Cloud Gateway | Cửa ngõ API tập trung |

---

### 2. File cấu hình mẫu `ride-service/src/main/resources/application.yml`:
```yaml
server:
  port: 8083

spring:
  application:
    name: ride-service

  # Cấu hình kết nối MySQL (chạy từ docker-compose)
  datasource:
    url: jdbc:mysql://localhost:3306/ride_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

  # Cấu hình kết nối Apache Kafka
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: ride-service-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.cns.lg.common.event"

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

---

### 3. File cấu hình mẫu `location-service/src/main/resources/application.yml`:
```yaml
server:
  port: 8081

spring:
  application:
    name: location-service

  # Cấu hình kết nối Redis (chạy từ docker-compose)
  data:
    redis:
      host: localhost
      port: 6379

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

---

## 4.2. Luồng Giao tiếp Giữa các Service trong Uber App Flow

Hệ thống Uber áp dụng kết hợp cả **Event-Driven Asynchronous** (Kafka) cho luồng đặt xe và **Synchronous REST** khi cần truy vấn toạ độ tức thì:

```text
 [Rider App] 
      │ (1) POST /api/v1/rides (Đặt xe)
      ▼
┌──────────────┐
│ ride-service │ (Lưu trạng thái REQUESTED vào MySQL)
└──────┬───────┘
       │ (2) Phát sự kiện: RideRequestEvent
       ▼
 ┌───────────┐
 │   Kafka   │  Topic: "ride-requests"
 └─────┬─────┘
       │ (3) Lắng nghe sự kiện
       ▼
┌──────────────────┐  (4) REST GET /api/v1/drivers/nearby?lat=...&lng=...  ┌──────────────────┐
│ matching-service ├──────────────────────────────────────────────────────►│ location-service │
└────────┬─────────┘                                                       └────────┬─────────┘
         │ (5) Tìm được tài xế phù hợp                                              │ Truy vấn GEORADIUS
         │                                                                          ▼
         │ (6) Phát sự kiện: MatchFoundEvent                                   ┌─────────┐
         ▼                                                                     │  Redis  │
   ┌───────────┐                                                               └─────────┘
   │   Kafka   │  Topic: "ride-matched"
   └─────┬─────┘
         │ (7) Lắng nghe sự kiện
         ▼
  ┌──────────────┐
  │ ride-service │ (Cập nhật MySQL: trạng thái MATCHED, gán driverId & biển số)
  └──────────────┘
```

1. **Khách hàng đặt xe**: Gửi request qua REST API tới `ride-service`.
2. **Khởi tạo cuốc xe**: `ride-service` tạo bản ghi chuyến đi trong MySQL với trạng thái `REQUESTED`, sau đó phát ngay `RideRequestEvent` lên Kafka topic `ride-requests`.
3. **Tiêu thụ sự kiện tìm xe**: `matching-service` nhận được `RideRequestEvent`.
4. **Truy vấn vị trí tài xế**: `matching-service` gọi sang `location-service` để lấy toạ độ các tài xế đang rảnh trong bán kính lân cận (được lưu bằng Redis GeoSpatial).
5. **Khớp cặp**: `matching-service` chọn ra tài xế gần nhất và khả thi nhất.
6. **Thông báo kết quả**: `matching-service` phát sự kiện `MatchFoundEvent` lên Kafka topic `ride-matched`.
7. **Cập nhật cuốc xe**: `ride-service` tiêu thụ sự kiện `MatchFoundEvent`, cập nhật trạng thái chuyến đi sang `MATCHED` và lưu thông tin tài xế vào MySQL.

---

## 4.3. Nguyên tắc Quản lý Dữ liệu (Database per Service)

> [!IMPORTANT]
> **Database per Service**: Mỗi service quản lý hoàn toàn cơ sở dữ liệu riêng:
> - **`ride-service`** quản lý **MySQL Database** (`ride_db`) để lưu trữ dữ liệu quan hệ, lịch sử cuốc xe, thanh toán và trạng thái chuyến đi cần tính toàn vẹn (ACID).
> - **`location-service`** quản lý **Redis In-memory Storage** để đọc/ghi hàng triệu toạ độ GPS tài xế mỗi giây với độ trễ cực thấp.
> - **`matching-service`** sử dụng **Apache Kafka** làm message log phân tán để xử lý luồng sự kiện.
> - **CẤM**: Tuyệt đối không để `matching-service` chọc thẳng vào MySQL của `ride-service` để đọc dữ liệu; mọi dữ liệu đều phải truyền qua Kafka Event hoặc REST API.

---

# PHẦN V: THAO TÁC MAVEN & CHUẨN HÓA QUY TRÌNH BUILD

## 5.1. Cơ chế Maven Reactor & Thứ tự Build Hệ thống

Nhờ cấu hình `<modules>` và các phụ thuộc `<dependency>`, **Maven Reactor** sẽ tự động tính toán đồ thị phụ thuộc và build theo thứ tự chuẩn xác:

```text
[INFO] Reactor Build Order:
[INFO]   uber-app (Root Parent POM)
[INFO]   common   (Biên dịch trước vì cả 3 service đều phụ thuộc vào common)
[INFO]   location-service
[INFO]   matching-service
[INFO]   ride-service
```

---

## 5.2. Các Câu lệnh Maven Cốt lõi trong Dự án Multi-module

| Mục tiêu thao tác | Lệnh thực thi từ thư mục `uber-app` |
| :--- | :--- |
| **Build toàn bộ 4 module** | `./mvnw clean install` |
| **Build nhanh (bỏ qua Unit Test)** | `./mvnw clean install -DskipTests` |
| **Chỉ build module `common`** | `./mvnw clean install -pl common` |
| **Build `ride-service` VÀ module `common`** | `./mvnw clean install -pl ride-service -am` |
| **Chạy trực tiếp `ride-service`** | `./mvnw spring-boot:run -pl ride-service` |
| **Chạy trực tiếp `location-service`** | `./mvnw spring-boot:run -pl location-service` |
| **Kiểm tra cây Dependency của `matching-service`** | `./mvnw dependency:tree -pl matching-service` |

- `-pl` (`--projects`): Chỉ định chính xác module cần thao tác.
- `-am` (`--also-make`): Tự động tìm và build các module phụ thuộc trước (ví dụ tự build `common` trước khi build `ride-service`).

---

## 5.3. Quản lý Dependency Scope & Dependency Hygiene

- **`compile`** *(Mặc định)*: Có mặt ở cả compile time và runtime (ví dụ: `common`, `spring-boot-starter-web`).
- **`provided`**: Cần khi biên dịch, môi trường runtime tự cấp (ví dụ: `lombok`).
- **`runtime`**: Không cần khi viết code, nhưng cần khi ứng dụng chạy (ví dụ: `mysql-connector-j`).
- **`test`**: Chỉ sử dụng khi chạy unit/integration test (ví dụ: `spring-boot-starter-test`, `spring-kafka-test`).

---

# PHẦN VI: CÁCH CHẠY DỰ ÁN (RUNNING GUIDE)

Để khởi chạy toàn bộ hệ sinh thái Microservices `uber-app`, bạn có thể lựa chọn một trong 3 cách: chạy trực tiếp từ **IntelliJ IDEA** (khuyên dùng khi phát triển code), chạy bằng dòng lệnh **Terminal (Maven Wrapper)**, hoặc đóng gói Fat JAR để chạy bằng **`java -jar`**.

Trước khi khởi chạy bất kỳ service Spring Boot nào, bạn **bắt buộc phải thực hiện 2 bước chuẩn bị nền tảng** bên dưới.

---

## 6.1. Các Bước Chuẩn Bị Bắt Buộc (Bật Hạ Tầng & Build Common)

### Bước 1: Khởi động Hạ tầng Docker (MySQL, Redis, Kafka, Zookeeper)
Các service của bạn phụ thuộc trực tiếp vào MySQL (lưu cuốc xe), Redis (lưu toạ độ tài xế), và Kafka (bắn/nhận event). Nếu chưa bật các container này, Spring Boot khi start sẽ lập tức báo lỗi `Connection refused` và crash.

Mở Terminal tại thư mục `uber-app/` và chạy:
```bash
docker compose up -d
```

> 🔍 **Kiểm tra trạng thái**: Gõ lệnh `docker ps` để đảm bảo 4 container sau đều ở trạng thái `Up`:
> - `mysql-rideshare` (Port `3306`)
> - `redis-geo` (Port `6379`)
> - `kafka` (Port `9092`)
> - `zookeeper` (Port `2181`)

### Bước 2: Build Cài đặt Module `common` vào Local Maven Cache
Vì cả 3 service (`location-service`, `matching-service`, `ride-service`) đều sử dụng các DTO và Event từ module `common`, bạn cần thực hiện build toàn bộ Monorepo 1 lần từ thư mục root để Maven sinh file `.jar` của `common` và cài đặt vào thư mục `.m2` local:

```bash
# Đứng tại thư mục root uber-app/
./mvnw clean install -DskipTests
```
*(Trên hệ điều hành Windows Command Prompt, bạn chạy lệnh `mvnw.cmd clean install -DskipTests`)*.

---

## 6.2. Cách 1: Chạy Bằng IntelliJ IDEA Services Dashboard (Khuyên Dùng Cho Dev)

Đây là cách trực quan, tiện lợi và hỗ trợ debug tốt nhất khi bạn đang viết code:

1. **Mở cửa sổ Services trong IntelliJ**:
   - Menu: **View ➔ Tool Windows ➔ Services** (hoặc phím tắt `Alt + 8`).
2. **Thêm các Spring Boot Application**:
   - Nếu IntelliJ chưa tự động nhận diện, bấm vào biểu tượng dấu **`+`** (Add Service) ➔ Chọn **Spring Boot**.
   - Cửa sổ sẽ hiển thị danh sách 3 ứng dụng:
     - `LocationServiceApplication`
     - `MatchingServiceApplication`
     - `RideServiceApplication`
3. **Thao tác Khởi chạy & Debug**:
   - Chuột phải vào từng service và chọn **Run** hoặc **Debug** (đặt breakpoint xem luồng dữ liệu).
   - Hoặc chọn cả 3 service và bấm nút **Play All** (hình tam giác kép màu xanh).
   - Mỗi service sẽ có một tab Console Log riêng biệt bên dưới, giúp bạn dễ dàng theo dõi log độc lập.

---

## 6.3. Cách 2: Chạy Bằng Dòng Lệnh Maven Wrapper (Terminal)

Mở **3 tab Terminal riêng biệt** (hoặc 3 cửa sổ CMD/PowerShell) tại thư mục `uber-app/`:

* **Terminal 1 — Chạy `location-service` (Port 8081)**:
  ```bash
  ./mvnw spring-boot:run -pl location-service
  ```

* **Terminal 2 — Chạy `matching-service` (Port 8082)**:
  ```bash
  ./mvnw spring-boot:run -pl matching-service
  ```

* **Terminal 3 — Chạy `ride-service` (Port 8083)**:
  ```bash
  ./mvnw spring-boot:run -pl ride-service
  ```

> 💡 **Ý nghĩa tham số `-pl` (`--projects`)**: Cho phép bạn đứng ngay tại thư mục root mà vẫn khởi chạy được từng service cụ thể mà không cần phải gõ lệnh `cd` chuyển thư mục.

---

## 6.4. Cách 3: Đóng Gói Fat JAR & Chạy Bằng `java -jar` (Mô phỏng Production)

Cách này được dùng khi bạn muốn kiểm tra xem file đóng gói cuối cùng có chạy ổn định hay không trước khi đẩy lên máy chủ hoặc đóng container:

1. **Biên dịch và đóng gói toàn bộ dự án**:
   ```bash
   ./mvnw clean package -DskipTests
   ```
2. **Chạy từng file JAR trong các cửa sổ dòng lệnh riêng**:
   ```bash
   # Chạy location-service
   java -jar location-service/target/location-service-0.0.1-SNAPSHOT.jar

   # Chạy matching-service
   java -jar matching-service/target/matching-service-0.0.1-SNAPSHOT.jar

   # Chạy ride-service
   java -jar ride-service/target/ride-service-0.0.1-SNAPSHOT.jar
   ```

---

## 6.5. Thứ Tự Khởi Động Khuyến Nghị & Kiểm Tra Trạng Thái Sống (Health Check)

### 1. Thứ tự khởi động khuyến nghị (Startup Order):
1. **Hạ tầng Docker**: MySQL, Redis, Kafka, Zookeeper (Bật đầu tiên).
2. **`location-service`**: Khởi động trước vì độc lập, chỉ phụ thuộc vào Redis.
3. **`matching-service`**: Khởi động tiếp theo để sẵn sàng kết nối Kafka và gọi sang location-service.
4. **`ride-service`**: Khởi động sau cùng (kết nối MySQL và Kafka).

### 2. Kiểm tra trạng thái hoạt động qua Spring Boot Actuator:
Sau khi các service khởi động xong, bạn có thể kiểm tra xem từng service đã kết nối thành công với cơ sở dữ liệu / broker tương ứng chưa thông qua endpoint Actuator:

- **Location Service**: [http://localhost:8081/actuator/health](http://localhost:8081/actuator/health)
- **Matching Service**: [http://localhost:8082/actuator/health](http://localhost:8082/actuator/health)
- **Ride Service**: [http://localhost:8083/actuator/health](http://localhost:8083/actuator/health)

👉 Nếu kết quả trả về JSON có dạng:
```json
{
  "status": "UP"
}
```
tức là service đã chạy hoàn toàn ổn định và sẵn sàng tiếp nhận request!

---

# PHẦN VII: CONTAINERIZATION, DOCKER COMPOSE & PATTERNS NÂNG CAO

## 7.1. Dockerize Microservices & File docker-compose.yml Thực tế

### 1. File `docker-compose.yml` tại thư mục root `uber-app`:
Dự án đã có sẵn file cấu hình chạy toàn bộ hạ tầng cơ sở dữ liệu và message broker:

```yaml
version: '3.8'

services:

  # ─────────────────────────────────────────
  # REDIS — Lưu trữ toạ độ tài xế thời gian thực
  # ─────────────────────────────────────────
  redis:
    image: redis:latest
    container_name: redis-geo
    ports:
      - "6379:6379"
    networks:
      - rideshare-network

  # ─────────────────────────────────────────
  # MYSQL — Lưu trữ dữ liệu cuốc xe (Ride Data)
  # ─────────────────────────────────────────
  mysql:
    image: mysql:8.0
    container_name: mysql-rideshare
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ride_db
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - rideshare-network

  # ─────────────────────────────────────────
  # ZOOKEEPER — Quản lý cụm Kafka
  # ─────────────────────────────────────────
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    networks:
      - rideshare-network

  # ─────────────────────────────────────────
  # KAFKA — Event Streaming phân tán
  # ─────────────────────────────────────────
  kafka:
    image: confluentinc/cp-kafka:7.4.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
    networks:
      - rideshare-network

volumes:
  mysql-data:

networks:
  rideshare-network:
    driver: bridge
```

Khởi động toàn bộ hạ tầng bằng một lệnh duy nhất:
```bash
docker compose up -d
```

---

### 2. Mẫu Dockerfile tối ưu (Multi-stage Build) cho `ride-service`:

Tạo file `ride-service/Dockerfile`:
```dockerfile
# Stage 1: Build JAR bằng JDK 17
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw clean package -pl ride-service -am -DskipTests

# Stage 2: Runtime image siêu nhẹ bằng JRE 17
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/ride-service/target/ride-service-*.jar app.jar
EXPOSE 8083
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 7.2. Kiến trúc Mở rộng: API Gateway, Security & Observability

Khi hoàn thiện hệ sinh thái Uber App, kiến trúc nên được bổ sung:

1. **Spring Cloud Gateway (Cổng 8080)**:
   - Là điểm tiếp nhận request duy nhất từ ứng dụng di động (Rider App, Driver App).
   - Định tuyến `/api/v1/rides/**` về `ride-service`.
   - Định tuyến `/api/v1/locations/**` về `location-service`.
   - Xác thực JWT Token và Rate Limiting chống spam request.
2. **Distributed Tracing (Micrometer + Zipkin/Jaeger)**:
   - Gắn `traceId` vào mỗi request đặt xe để theo dõi hành trình của request đi xuyên suốt từ Gateway -> `ride-service` -> Kafka -> `matching-service` -> `location-service`.
3. **Metrics & Giám sát**:
   - Spring Boot Actuator + Prometheus + Grafana để theo dõi số lượng cuốc xe đặt mỗi phút, thời gian phản hồi khớp xe, tải CPU/RAM của từng service.

---

## 7.3. Distributed Transactions (Saga & Outbox Pattern) trong Đặt xe

Khi một cuốc xe diễn ra qua nhiều microservice:
- **Saga Pattern**: Nếu `matching-service` sau 30 giây không tìm thấy tài xế nào khả dụng trong khu vực, nó sẽ phát sự kiện `RideMatchTimeoutEvent`. `ride-service` lắng nghe event này và thực hiện **Giao dịch bù trừ** (Compensating Transaction): chuyển trạng thái cuốc xe sang `NO_DRIVER_FOUND` và hoàn lại tiền nếu khách đã trừ ví.
- **Transactional Outbox Pattern**: Đảm bảo tính nguyên tử giữa việc lưu trạng thái cuốc xe vào MySQL và gửi event lên Kafka, tránh tình trạng lưu database thành công nhưng mạng chập chờn làm mất event gửi sang Kafka.

---

# PHẦN VIII: ROADMAP VÀ CHECKLIST THỰC HÀNH

## 8.1. Lộ trình Học tập & Phát triển Năng lực Java Backend Engineer

```text
Level 1: Maven Multi-module Monorepo & Quản lý Parent POM
   │
   ▼
Level 2: Spring Boot 3 Core, Validation & REST API
   │
   ▼
Level 3: MySQL, Spring Data JPA & Database per Service
   │
   ▼
Level 4: Redis Caching & Redis GeoSpatial (Quản lý toạ độ thời gian thực)
   │
   ▼
Level 5: Event-Driven Architecture với Apache Kafka (Producer, Consumer, Topics)
   │
   ▼
Level 6: API Gateway, Microservice Security (JWT, OAuth2) & Distributed Tracing
   │
   ▼
Level 7: Docker Containerization, Docker Compose & CI/CD Deployment
```

---

## 8.2. Checklist Setup Project từ A-Z

### Phase 1 — Khởi tạo Skeleton & Maven
- [x] Cài đặt JDK 17 và Maven 3.9+.
- [x] Tạo thư mục Root `uber-app`.
- [x] Chạy `mvn wrapper:wrapper` để tạo Maven Wrapper.
- [x] Cấu hình Root `pom.xml` chuẩn với Spring Boot Parent `3.3.4`, Spring Kafka version, dependencyManagement và pluginManagement.
- [x] Tạo module `common` với cấu hình plugin `<skip>true</skip>`.
- [x] Tạo các modules nghiệp vụ: `location-service`, `matching-service`, `ride-service`.
- [x] Thêm dependency `common` vào từng module con.

### Phase 2 — Mã nguồn & Cấu hình App
- [x] Viết các class dùng chung trong `common` (`ApiResponse`, `RideRequestEvent`, `MatchFoundEvent`).
- [x] Cấu hình file `application.yml` cho từng Service với Port và Database riêng biệt (MySQL, Redis, Kafka).
- [x] Kiểm tra lệnh build toàn bộ Monorepo: `./mvnw clean install`.
- [x] Chạy thử một service: `./mvnw spring-boot:run -pl ride-service`.

### Phase 3 — Hạ tầng Local & Docker
- [x] Khởi động MySQL, Redis, Kafka, Zookeeper qua `docker compose up -d`.
- [x] Tạo file `.gitignore` đầy đủ cho Java, Maven, IDE, Docker, Office.

---

## 8.3. 10 Quy tắc Vàng (Core Rules) & Kết luận

> [!TIP]
> 1. **Root POM luôn là `<packaging>pom</packaging>`** và không chứa code Java.
> 2. **Dùng `<dependencyManagement>` ở Root POM** để quản lý thống nhất phiên bản thư viện chung (`common`, `spring-kafka`).
> 3. **Module `common` bắt buộc cấu hình `<skip>true</skip>`** ở `spring-boot-maven-plugin` để đóng gói thành Plain JAR.
> 4. **Mỗi Microservice là một ứng dụng độc lập** sở hữu Database riêng biệt (MySQL cho `ride-service`, Redis cho `location-service`).
> 5. **Không bao giờ phụ thuộc chéo trực tiếp class giữa các Microservice nghiệp vụ**; giao tiếp phải thông qua Kafka Event hoặc REST.
> 6. **Module `common` chỉ chứa DTO/Event thực sự dùng chung**, không biến `common` thành nơi chứa code lung tung.
> 7. **Dùng Maven Wrapper (`./mvnw`)** để đảm bảo tính đồng bộ môi trường trên mọi máy dev.
> 8. **Không bao giờ hard-code Secret/Password** trong code; dùng biến môi trường trong `application.yml`.
> 9. **Mỗi Microservice sở hữu bộ Unit & Integration Test riêng biệt**.
> 10. **Bám sát domain nghiệp vụ Uber**: Luôn kiểm soát chặt chẽ luồng sự kiện từ đặt xe -> ghép xe -> nhận chuyến -> hoàn thành.

---

### Kết luận

Bằng việc kết hợp kiến trúc **Maven Multi-module Monorepo** với mô hình dịch vụ thực tế của **Uber App** (`common`, `location-service`, `matching-service`, `ride-service`), dự án của bạn đã có một bộ khung chuẩn doanh nghiệp, rõ ràng về mặt trách nhiệm từng module, dễ dàng bảo trì và sẵn sàng mở rộng quy mô.
