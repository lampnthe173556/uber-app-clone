# Microservices Parent – Multi-module Maven Project

> Hướng dẫn từ A-Z để setup một project **Java Spring Boot Microservices theo mô hình Maven Multi-module**, tập trung vào cách tổ chức project, Maven Parent POM, dependency management và các best practice phù hợp cho người mới.

---

## 1. Mục tiêu của project

Chúng ta sẽ xây dựng cấu trúc:

```text
microservices-parent/
├── pom.xml
├── README.md
│
├── common/
│   ├── pom.xml
│   └── src/
│
├── user-service/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/example/userservice/
│       │   └── resources/
│       └── test/
│
├── order-service/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/example/orderservice/
│       │   └── resources/
│       └── test/
│
└── notification-service/
    ├── pom.xml
    └── src/
        ├── main/
        │   ├── java/com/example/notificationservice/
        │   └── resources/
        └── test/
```

Ý tưởng:

- `microservices-parent`: Maven parent/aggregator.
- `common`: code dùng chung thật sự giữa các service.
- `user-service`: quản lý user.
- `order-service`: quản lý order.
- `notification-service`: gửi notification.

Mỗi service là **một Spring Boot application độc lập** và có thể được build/deploy độc lập.

---

# 2. Trước tiên: hiểu Maven Multi-module là gì?

Có 3 khái niệm rất dễ bị nhầm:

## 2.1 Parent POM

Parent POM là nơi tập trung các cấu hình dùng chung.

Ví dụ:

- Java version
- Spring Boot version
- dependency versions
- plugin versions
- encoding
- Maven compiler configuration

Ví dụ:

```xml
<properties>
    <java.version>21</java.version>
</properties>
```

Các module con có thể kế thừa property này.

---

## 2.2 Aggregator POM

Aggregator POM là POM đứng ở root và khai báo các module:

```xml
<modules>
    <module>common</module>
    <module>user-service</module>
    <module>order-service</module>
</modules>
```

Khi chạy:

```bash
mvn clean install
```

ở root, Maven biết phải build các module bên dưới.

---

## 2.3 Parent và Aggregator không hoàn toàn giống nhau

Một project có thể:

- chỉ là parent;
- chỉ là aggregator;
- vừa là parent vừa là aggregator.

Trong project microservices, cách đơn giản và phổ biến cho một repository là:

```text
root POM
   │
   ├── parent configuration
   │
   └── module aggregation
```

Tức root POM vừa:

- cung cấp cấu hình chung;
- quản lý dependency;
- quản lý plugin;
- gom các module.

---

# 3. Kiến trúc Maven đề xuất

```text
                    microservices-parent
                            │
                 ┌──────────┴──────────┐
                 │                     │
              Parent POM          Module Aggregator
                 │                     │
       ┌─────────┼─────────┐           │
       │         │         │           │
     common     user      order    notification
                service    service      service
```

Điểm quan trọng:

```text
Root POM
  ├── common
  ├── user-service
  ├── order-service
  └── notification-service
```

Không nên biến root project thành một Spring Boot application.

Root chỉ có nhiệm vụ quản lý build.

---

# 4. Prerequisites

Nên chuẩn bị:

- JDK 21
- Maven 3.9+
- Git
- IntelliJ IDEA
- Docker Desktop nếu sau này chạy PostgreSQL/Redis/Kafka

Kiểm tra:

```bash
java -version
```

Ví dụ:

```text
openjdk version "21.x.x"
```

Kiểm tra Maven:

```bash
mvn -version
```

Nên thấy:

```text
Apache Maven 3.9.x
Java version: 21
```

---

# 5. Tạo project root

Tạo thư mục:

```bash
mkdir microservices-parent
cd microservices-parent
```

Tạo:

```text
pom.xml
```

Cấu trúc ban đầu:

```text
microservices-parent/
└── pom.xml
```

---

# 6. Root pom.xml

Đây là phần quan trọng nhất.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>microservices-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <name>microservices-parent</name>
    <description>Multi-module Maven project for microservices</description>

    <modules>
        <module>common</module>
        <module>user-service</module>
        <module>order-service</module>
        <module>notification-service</module>
    </modules>

    <properties>
        <java.version>21</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

</project>
```

> Lưu ý: version Spring Boot ở ví dụ trên chỉ là version minh họa. Khi bắt đầu project thực tế, nên chọn một version Spring Boot đang được team/company chuẩn hóa và kiểm tra compatibility với JDK.

---

# 7. Giải thích từng phần của root POM

## 7.1 modelVersion

```xml
<modelVersion>4.0.0</modelVersion>
```

Đây là model version của Maven POM.

Hiện tại gần như luôn là:

```xml
4.0.0
```

Không cần thay đổi.

---

# 8. groupId

```xml
<groupId>com.example</groupId>
```

Thường dùng để định danh tổ chức/project.

Ví dụ công ty:

```text
com.company
```

Project:

```text
com.company.payment
```

Một convention tốt:

```text
com.company.project
```

Ví dụ:

```text
com.fpt.payment
```

---

# 9. artifactId

```xml
<artifactId>microservices-parent</artifactId>
```

Đây là tên artifact Maven.

Với root project:

```text
microservices-parent
```

Các service:

```text
user-service
order-service
notification-service
```

---

# 10. version

```xml
<version>1.0.0-SNAPSHOT</version>
```

Ý nghĩa:

```text
1.0.0-SNAPSHOT
```

thường đại diện cho version đang phát triển.

Ví dụ:

```text
1.0.0-SNAPSHOT
1.0.0
1.1.0
2.0.0
```

Trong quá trình development:

```text
1.0.0-SNAPSHOT
```

Khi release:

```text
1.0.0
```

---

# 11. packaging

Đây là phần rất quan trọng.

```xml
<packaging>pom</packaging>
```

Root project **không phải application**.

Nó chỉ là:

```text
Parent
+
Aggregator
```

Do đó:

```xml
<packaging>pom</packaging>
```

Nếu không khai báo `packaging`, Maven mặc định là:

```text
jar
```

Nhưng root multi-module project thường nên khai báo rõ:

```xml
<packaging>pom</packaging>
```

---

# 12. Vì sao service dùng jar?

Ví dụ `user-service`:

```xml
<packaging>jar</packaging>
```

Spring Boot tạo:

```text
user-service-1.0.0.jar
```

Sau đó có thể chạy:

```bash
java -jar user-service-1.0.0.jar
```

Đây là cách rất phổ biến để deploy Spring Boot microservice.

---

# 13. modules

Root POM:

```xml
<modules>
    <module>common</module>
    <module>user-service</module>
    <module>order-service</module>
    <module>notification-service</module>
</modules>
```

Maven sẽ hiểu:

```text
microservices-parent/
│
├── common/
├── user-service/
├── order-service/
└── notification-service/
```

Khi chạy:

```bash
mvn clean install
```

Maven sẽ build các module theo dependency graph.

---

# 14. Tạo common module

Tạo:

```text
common/
└── pom.xml
```

POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>common</artifactId>

</project>
```

Maven sẽ tự hiểu parent nằm ở:

```text
../pom.xml
```

---

# 15. Common nên chứa gì?

Có thể chứa:

```text
common/
└── src/main/java/com/example/common/
    ├── exception/
    ├── response/
    ├── constant/
    ├── util/
    └── dto/
```

Ví dụ:

```java
public record ApiResponse<T>(
        boolean success,
        T data,
        String message
) {
}
```

Hoặc exception:

```java
public class BusinessException extends RuntimeException {

    public BusinessException(String message) {
        super(message);
    }
}
```

---

# 16. Nhưng đừng biến common thành "thùng rác"

Đây là best practice cực kỳ quan trọng.

Không nên:

```text
common/
├── UserEntity
├── OrderEntity
├── UserRepository
├── OrderRepository
├── UserService
├── OrderService
└── ...
```

Nếu làm như vậy, các microservice bắt đầu phụ thuộc chặt vào nhau.

Microservices mất tính độc lập.

Nên common chỉ chứa những thứ thực sự generic.

Ví dụ:

```text
common
├── ApiResponse
├── BusinessException
├── ErrorCode
└── common utilities
```

---

# 17. Tạo user-service

Cấu trúc:

```text
user-service/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
```

POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>user-service</artifactId>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

# 18. Tại sao user-service không cần khai báo version Spring Boot?

Không viết:

```xml
<version>3.5.5</version>
```

cho từng dependency Spring Boot.

Ví dụ:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot parent đã quản lý version.

Đây chính là một trong những lợi ích lớn của dependency management.

---

# 19. Parent POM quản lý version như thế nào?

Root:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.5</version>
</parent>
```

Sau đó:

```text
microservices-parent
        │
        ▼
Spring Boot Parent
        │
        ▼
dependency management
        │
        ├── spring-web
        ├── spring-core
        ├── jackson
        ├── tomcat
        └── ...
```

Các module con kế thừa:

```text
microservices-parent
        │
        ├── user-service
        ├── order-service
        └── notification-service
```

---

# 20. Thêm dependency common vào service

Nếu `user-service` cần dùng:

```text
ApiResponse
BusinessException
```

thì thêm:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
</dependency>
```

Nhưng cần đảm bảo version được quản lý.

Có thể khai báo version trong parent:

```xml
<dependencyManagement>
    <dependencies>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
            <version>${project.version}</version>
        </dependency>

    </dependencies>
</dependencyManagement>
```

Sau đó service chỉ cần:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
</dependency>
```

---

# 21. dependencyManagement vs dependencies

Đây là một điểm người mới rất hay nhầm.

## dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

Có nghĩa:

> Module này thực sự sử dụng dependency.

---

## dependencyManagement

```xml
<dependencyManagement>
    <dependencies>
        ...
    </dependencies>
</dependencyManagement>
```

Có nghĩa:

> Chỉ quản lý version/configuration. Không tự động đưa dependency vào classpath.

Ví dụ parent:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

`user-service` vẫn phải khai báo:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
</dependency>
```

---

# 22. Tạo main class

```text
user-service/
└── src/main/java/com/example/userservice/
    └── UserServiceApplication.java
```

Code:

```java
package com.example.userservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

---

# 23. Tạo Controller test

```java
package com.example.userservice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @GetMapping("/api/users/hello")
    public String hello() {
        return "Hello from User Service";
    }
}
```

Chạy:

```bash
mvn spring-boot:run -pl user-service
```

Hoặc:

```bash
cd user-service
mvn spring-boot:run
```

---

# 24. Tạo order-service

POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>order-service</artifactId>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

# 25. Thứ tự dependency

Maven tự xử lý dependency graph.

Ví dụ:

```text
order-service
      │
      ▼
    common
```

Khi chạy:

```bash
mvn clean install
```

Maven sẽ build:

```text
common
   ↓
order-service
```

Nếu có:

```text
user-service
notification-service
```

nhưng chúng không phụ thuộc nhau, Maven có thể build chúng độc lập/phù hợp với dependency graph.

---

# 26. Tạo notification-service

POM tương tự:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>notification-service</artifactId>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

# 27. Cấu trúc hoàn chỉnh

Sau khi setup:

```text
microservices-parent/
│
├── pom.xml
├── README.md
│
├── common/
│   ├── pom.xml
│   └── src/
│       └── main/
│           └── java/
│               └── com/example/common/
│                   ├── exception/
│                   ├── response/
│                   └── constant/
│
├── user-service/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/
│       │   │   └── com/example/userservice/
│       │   │       ├── UserServiceApplication.java
│       │   │       ├── controller/
│       │   │       ├── service/
│       │   │       ├── repository/
│       │   │       ├── entity/
│       │   │       └── dto/
│       │   │
│       │   └── resources/
│       │       └── application.yml
│       │
│       └── test/
│
├── order-service/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/example/orderservice/
│       │   └── resources/application.yml
│       └── test/
│
└── notification-service/
    ├── pom.xml
    └── src/
        ├── main/
        │   ├── java/com/example/notificationservice/
        │   └── resources/application.yml
        └── test/
```

---

# 28. Best practice package structure

Không nên để toàn bộ code trong:

```text
controller/
service/
repository/
```

ở cấp root nếu project lớn.

Một hướng dễ maintain:

```text
user-service/
└── src/main/java/com/example/userservice/
    │
    ├── UserServiceApplication.java
    │
    ├── user/
    │   ├── controller/
    │   ├── service/
    │   ├── repository/
    │   ├── entity/
    │   └── dto/
    │
    ├── config/
    ├── exception/
    └── security/
```

Hoặc nếu muốn tiến tới Clean/Hexagonal Architecture:

```text
user-service/
└── user/
    ├── domain/
    ├── application/
    ├── infrastructure/
    └── adapter/
```

Với người mới, nên bắt đầu đơn giản:

```text
controller
service
repository
entity
dto
config
exception
```

Sau khi hiểu Spring Boot/Spring Security/JPA thì mới chuyển dần sang Clean/Hexagonal.

---

# 29. application.yml

Ví dụ:

```yaml
spring:
  application:
    name: user-service

server:
  port: 8081
```

Order:

```yaml
spring:
  application:
    name: order-service

server:
  port: 8082
```

Notification:

```yaml
spring:
  application:
    name: notification-service

server:
  port: 8083
```

---

# 30. Chạy từng service

## User

```bash
mvn spring-boot:run -pl user-service
```

Port:

```text
8081
```

## Order

```bash
mvn spring-boot:run -pl order-service
```

Port:

```text
8082
```

## Notification

```bash
mvn spring-boot:run -pl notification-service
```

Port:

```text
8083
```

---

# 31. Maven reactor

Khi chạy:

```bash
mvn clean install
```

root Maven sử dụng cơ chế gọi là **Maven Reactor**.

Nó sẽ:

1. đọc root POM;
2. đọc modules;
3. xác định dependency graph;
4. xác định thứ tự build;
5. compile;
6. test;
7. package;
8. install artifact.

Ví dụ:

```text
common
   │
   ├──────► order-service
   │
   └──────► user-service
```

Maven hiểu dependency graph này.

---

# 32. Các command Maven quan trọng

## Build toàn bộ

```bash
mvn clean install
```

---

## Build không chạy test

```bash
mvn clean install -DskipTests
```

Lưu ý:

```bash
-DskipTests
```

thường vẫn compile test nhưng không execute test.

Nếu thực sự muốn bỏ qua test compilation:

```bash
-Dmaven.test.skip=true
```

Không nên dùng trong CI bình thường.

---

## Chỉ build một module

```bash
mvn clean install -pl user-service
```

---

## Build module và dependency của nó

```bash
mvn clean install -pl order-service -am
```

`-am` = also make required projects.

Ví dụ:

```text
order-service
      ↓
    common
```

thì Maven sẽ build:

```text
common
order-service
```

---

# 33. `-pl` và `-am`

Đây là command rất hữu ích.

```bash
mvn clean install -pl order-service
```

Có nghĩa:

> Build order-service.

Nếu `order-service` cần `common` nhưng common chưa có artifact phù hợp trong local repository, có thể dùng:

```bash
mvn clean install -pl order-service -am
```

Nghĩa:

```text
-pl = projects list
-am = also make dependencies
```

---

# 34. Package vs Install

## package

```bash
mvn package
```

Tạo:

```text
target/*.jar
```

Nhưng chưa đưa artifact vào local Maven repository.

---

## install

```bash
mvn install
```

Sau package, Maven đưa artifact vào:

```text
~/.m2/repository
```

Ví dụ:

```text
~/.m2/repository/com/example/common/1.0.0-SNAPSHOT/
```

---

# 35. Khi nào cần `install`?

Trong multi-module project, nếu build từ root:

```bash
mvn clean install
```

thường không có vấn đề vì Maven Reactor biết các module.

Nếu build riêng:

```bash
cd order-service
mvn package
```

mà `common` chưa có artifact cần thiết trong local repository, có thể gặp lỗi dependency.

Khi đó:

```bash
mvn clean install
```

ở root là cách đơn giản nhất.

---

# 36. Một best practice quan trọng: không copy version

Không nên:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.x.x</version>
</dependency>
```

ở 5 service nếu cùng dùng một version.

Nên quản lý centralized.

Ví dụ parent:

```xml
<properties>
    <java.version>21</java.version>
    <postgresql.version>42.x.x</postgresql.version>
</properties>
```

và:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>${postgresql.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Service:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

---

# 37. Nhưng đừng quản lý mọi thứ thủ công

Nếu Spring Boot đã quản lý version:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

không cần tự thêm version.

Best practice:

```text
Spring Boot BOM/parent
        ↓
quản lý version framework ecosystem
        ↓
Root POM
        ↓
service POM
```

Chỉ override khi có lý do rõ ràng.

---

# 38. Dependency scope

Các scope quan trọng:

```text
compile
provided
runtime
test
```

## compile

Mặc định:

```xml
<scope>compile</scope>
```

Dependency có trong compile/runtime.

---

## test

```xml
<scope>test</scope>
```

Ví dụ:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Chỉ dùng trong test.

---

## runtime

Ví dụ database driver trong một số trường hợp:

```xml
<scope>runtime</scope>
```

Dependency cần khi chạy nhưng không nhất thiết cần để compile.

---

# 39. Không nên phụ thuộc chéo giữa microservices

Không nên:

```text
user-service
    ↓
order-service
```

bằng cách import Java class:

```java
import com.example.orderservice.entity.Order;
```

Điều này phá vỡ ranh giới service.

Thay vào đó:

```text
user-service
      │
      │ HTTP / REST
      ▼
order-service
```

hoặc:

```text
user-service
      │
      │ event
      ▼
Kafka
      │
      ▼
order-service
```

---

# 40. Database per service

Microservices nên hướng tới:

```text
user-service
    ↓
user-db

order-service
    ↓
order-db

notification-service
    ↓
notification-db
```

Không nên:

```text
                   ┌─────────────┐
user-service ──────┤             │
order-service ─────┤  ONE DB     │
notification ──────┤             │
                   └─────────────┘
```

Nếu mọi service truy cập trực tiếp cùng database/schema, coupling tăng mạnh.

---

# 41. Communication giữa services

Có hai kiểu chính.

## Synchronous

```text
Order Service
     │
     │ REST
     ▼
User Service
```

Ví dụ:

```http
GET /api/users/{id}
```

Dùng khi cần response ngay.

---

## Asynchronous

```text
Order Service
      │
      │ OrderCreated
      ▼
    Kafka
      │
      ├────────► Notification Service
      │
      └────────► Statistic Service
```

Dùng cho event-driven workflow.

---

# 42. Khi nào nên tạo module `common`?

Chỉ tạo nếu có code thực sự dùng chung.

Ví dụ:

```text
common
├── exception
├── response
├── error
└── utility
```

Không nên đưa business logic vào common.

Không nên:

```text
common/
└── UserService.java
```

vì User Service phải thuộc về `user-service`.

---

# 43. Một cách tổ chức nâng cao hơn

Khi project lớn, có thể tách:

```text
microservices-parent/
│
├── pom.xml
│
├── common/
│
├── common-web/
│
├── common-security/
│
├── user-service/
├── order-service/
└── notification-service/
```

Trong đó:

```text
common
```

chứa generic domain-independent code.

```text
common-web
```

chứa:

```text
ExceptionHandler
ApiResponse
Web configuration
```

```text
common-security
```

chứa những thành phần security dùng chung nếu thật sự cần.

Tuy nhiên, không nên tách quá sớm.

---

# 44. Parent POM nên quản lý những gì?

Nên quản lý:

```text
Java version
Spring Boot version
dependency versions
plugin versions
encoding
compiler configuration
test configuration
```

Ví dụ:

```xml
<properties>
    <java.version>21</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

Có thể quản lý:

```text
Maven Compiler Plugin
Maven Surefire Plugin
JaCoCo
Checkstyle
Spotless
```

nếu team sử dụng.

---

# 45. Không nên nhét business logic vào parent

Root POM chỉ quản lý build.

Không có:

```text
Java source code
business logic
database entity
controller
service
repository
```

---

# 46. Plugin management

Nếu có nhiều module và muốn quản lý plugin version tập trung, có thể dùng:

```xml
<build>
    <pluginManagement>
        <plugins>

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

        </plugins>
    </pluginManagement>
</build>
```

Điểm cần hiểu:

```text
pluginManagement
```

chỉ quản lý cấu hình/version.

Nó không nhất thiết kích hoạt plugin.

Module vẫn có thể cần:

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
</plugins>
```

---

# 47. Maven Wrapper

Nên commit Maven Wrapper:

```text
mvnw
mvnw.cmd
.mvn/
```

Sau đó developer không nhất thiết phải cài đúng Maven version bằng tay.

Linux/macOS:

```bash
./mvnw clean install
```

Windows:

```bash
mvnw.cmd clean install
```

Hoặc:

```bash
./mvnw
```

tùy shell/environment.

Best practice:

```text
Project
  ↓
Maven Wrapper
  ↓
đảm bảo Maven version nhất quán
```

---

# 48. Gitignore

Nên có:

```gitignore
target/
.idea/
*.iml
.classpath
.project
.settings/
.vscode/
*.log
.env
```

Không commit:

```text
target/
```

và secret:

```text
.env
application-local.yml
credentials
private keys
```

---

# 49. Configuration và secrets

Không hard-code:

```yaml
spring:
  datasource:
    username: admin
    password: 123456
```

Đặc biệt không commit password thật lên Git.

Development có thể dùng:

```yaml
spring:
  datasource:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Sau đó inject environment variable.

Production có thể dùng:

- Kubernetes Secrets
- Vault
- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager

tùy infrastructure.

---

# 50. Profile

Có thể dùng:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Ví dụ:

```yaml
spring:
  profiles:
    active: dev
```

Tuy nhiên tránh hard-code:

```yaml
spring:
  profiles:
    active: prod
```

trong source code.

Nên để deployment environment quyết định profile.

---

# 51. Database dependency

Ví dụ service sử dụng PostgreSQL:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Chỉ thêm database dependency vào service thực sự sử dụng database.

Không nên đưa JPA/PostgreSQL vào `common`.

---

# 52. Redis

Service nào dùng Redis thì service đó dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Không nên:

```text
common
  └── Redis dependency
```

nếu tất cả service không cần Redis.

---

# 53. Kafka

Service nào producer/consumer Kafka thì service đó mới khai báo:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

Ví dụ:

```text
order-service
     │
     │ publish
     ▼
   Kafka
     │
     ▼
notification-service
```

Không nhất thiết phải tạo:

```text
common-kafka
```

ngay từ đầu.

Chỉ tạo khi có abstraction thực sự dùng chung và đã xác định rõ boundary.

---

# 54. Test strategy

Mỗi service nên có:

```text
src/test/java
```

Ví dụ:

```text
user-service/
└── src/test/java/com/example/userservice/
    ├── controller/
    ├── service/
    └── repository/
```

Có thể dùng:

- JUnit 5
- Mockito
- Spring Boot Test
- Testcontainers

---

# 55. Unit test

Ví dụ:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldFindUser() {
        // arrange
        // act
        // assert
    }
}
```

Không cần load toàn bộ Spring context nếu chỉ test business logic.

---

# 56. Integration test

Khi cần test:

```text
Spring
+
Database
+
Repository
```

có thể dùng:

```java
@SpringBootTest
class UserIntegrationTest {
}
```

Với database thật trong test, có thể cân nhắc Testcontainers.

---

# 57. Build pipeline

Một CI pipeline cơ bản:

```text
Git Push
   │
   ▼
Compile
   │
   ▼
Unit Test
   │
   ▼
Integration Test
   │
   ▼
Package
   │
   ▼
Docker Build
   │
   ▼
Push Image
   │
   ▼
Deploy
```

Maven command:

```bash
./mvnw clean verify
```

Sau đó Docker build từng service.

---

# 58. Docker cho từng service

Không nên tạo một Docker image chứa tất cả service.

Nên:

```text
user-service
    ↓
user-service:1.0.0

order-service
    ↓
order-service:1.0.0

notification-service
    ↓
notification-service:1.0.0
```

Mỗi service deploy độc lập.

---

# 59. Dockerfile mẫu

Ví dụ:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/user-service-*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build:

```bash
mvn clean package
```

Sau đó:

```bash
docker build -t user-service:1.0.0 user-service/
```

---

# 60. Một lưu ý quan trọng về microservices

Maven multi-module **không đồng nghĩa** với microservices.

Ví dụ:

```text
multi-module Maven
```

chỉ là cách tổ chức source/build.

Microservices thực sự còn liên quan đến:

```text
Service boundary
Database ownership
Deployment independence
Communication
Observability
Resilience
Security
Scaling
CI/CD
Infrastructure
```

Do đó:

```text
Maven Multi-module
        ≠
Microservices Architecture
```

Nó chỉ là một cách rất tiện để quản lý nhiều service trong cùng repository.

---

# 61. Monorepo vs Polyrepo

Có hai cách phổ biến.

## Monorepo

```text
git repository
│
├── user-service
├── order-service
├── payment-service
└── notification-service
```

Ưu điểm:

- dễ tìm code;
- refactor dễ;
- dependency chung dễ quản lý;
- CI có thể tập trung;
- phù hợp team nhỏ/vừa.

Nhược điểm:

- repository lớn;
- pipeline cần tối ưu;
- boundary phải được giữ kỷ luật.

---

## Polyrepo

```text
user-service.git
order-service.git
payment-service.git
notification-service.git
```

Mỗi service một repository.

Ưu điểm:

- deployment độc lập rõ;
- ownership rõ;
- repository nhỏ.

Nhược điểm:

- dependency management khó hơn;
- thay đổi cross-service phức tạp;
- nhiều repository phải quản lý.

Với project học tập hoặc team nhỏ, multi-module monorepo thường dễ bắt đầu hơn.

---

# 62. Best practice về version

Có thể bắt đầu:

```text
1.0.0-SNAPSHOT
```

Các module dùng cùng version:

```text
common              1.0.0-SNAPSHOT
user-service        1.0.0-SNAPSHOT
order-service       1.0.0-SNAPSHOT
notification-service 1.0.0-SNAPSHOT
```

Khi release:

```text
1.0.0
```

Sau đó phát triển:

```text
1.1.0-SNAPSHOT
```

Nếu team cần release độc lập từng service, có thể dùng version riêng cho service.

Không nên áp dụng versioning phức tạp trước khi có nhu cầu.

---

# 63. Naming convention

Nên nhất quán:

```text
user-service
order-service
payment-service
notification-service
```

Artifact:

```text
user-service
```

Application name:

```yaml
spring:
  application:
    name: user-service
```

Package:

```text
com.company.userservice
```

Class:

```text
UserServiceApplication
```

---

# 64. Không nên đặt package như thế này

Không nên:

```text
com.example.service
```

vì quá generic.

Nên:

```text
com.example.userservice
```

hoặc:

```text
com.example.user
```

Điều này giúp Spring component scanning và ownership rõ ràng hơn.

---

# 65. Dependency hygiene

Mỗi service chỉ nên khai báo dependency nó thực sự dùng.

Ví dụ `notification-service` không dùng JPA:

Không thêm:

```xml
spring-boot-starter-data-jpa
```

Không thêm PostgreSQL.

Không thêm Redis.

Không thêm Kafka nếu chưa sử dụng.

Nguyên tắc:

```text
Dependency should follow responsibility.
```

---

# 66. Kiểm tra dependency tree

Rất hữu ích:

```bash
mvn dependency:tree
```

Hoặc:

```bash
mvn dependency:tree -pl user-service
```

Dùng khi gặp:

```text
ClassNotFoundException
NoSuchMethodError
version conflict
```

---

# 67. Dependency convergence

Một project lớn có thể có:

```text
library A
   ↓
jackson 2.x

library B
   ↓
jackson 2.y
```

Maven phải resolve dependency tree.

Có thể dùng:

```bash
mvn dependency:tree
```

và các Maven Enforcer rules để phát hiện vấn đề sớm.

---

# 68. Maven Enforcer

Có thể thêm vào parent để enforce:

- Java version;
- Maven version;
- dependency convergence;
- banned dependencies.

Ví dụ concept:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
</plugin>
```

Không nhất thiết phải cấu hình phức tạp ngay khi mới bắt đầu.

Khi team/project lớn hơn, đây là một cải tiến đáng cân nhắc.

---

# 69. Formatting

Nên thống nhất code format.

Có thể dùng:

```text
Spotless
Checkstyle
EditorConfig
```

Mục tiêu:

```text
Developer A
Developer B
Developer C
        ↓
same code style
```

CI có thể fail nếu code không đúng format.

---

# 70. Logging

Không nên:

```java
System.out.println("User created");
```

Nên dùng logging:

```java
private static final Logger log =
        LoggerFactory.getLogger(UserService.class);
```

Hoặc Lombok:

```java
@Slf4j
@Service
public class UserService {
}
```

Sau đó:

```java
log.info("Creating user with email={}", email);
```

Không log:

```text
password
JWT
access token
credit card
secret
```

---

# 71. Actuator

Microservice thực tế thường cần health check.

Thêm:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Có thể expose:

```text
/actuator/health
```

Kubernetes có thể dùng health endpoint cho:

```text
liveness
readiness
```

---

# 72. Observability về sau

Khi project lớn hơn nên nghĩ tới:

```text
Logging
Metrics
Tracing
```

Stack có thể là:

```text
Micrometer
Prometheus
Grafana
OpenTelemetry
Jaeger/Tempo
ELK/OpenSearch
```

Không cần setup toàn bộ ngay khi mới tạo project.

---

# 73. Security

Sau khi skeleton chạy ổn mới thêm:

```text
Spring Security
JWT
OAuth2/OIDC
API Gateway
```

Không nên ngay từ bước đầu nhét tất cả vào mọi service.

Có thể bắt đầu:

```text
user-service
      │
      ▼
Spring Security
      │
      ▼
JWT
```

Sau đó mới mở rộng.

---

# 74. API Gateway

Khi có nhiều service:

```text
Client
   │
   ▼
API Gateway
   │
   ├──► User Service
   ├──► Order Service
   └──► Notification Service
```

Gateway có thể chịu trách nhiệm:

- routing;
- authentication;
- rate limiting;
- CORS;
- request correlation;
- TLS termination.

Nhưng gateway không nên chứa business logic của service.

---

# 75. Service Discovery

Ở hệ thống lớn có thể có:

```text
Service Discovery
```

Ví dụ:

```text
Eureka
Consul
Kubernetes Service Discovery
```

Nhưng nếu deploy trên Kubernetes, Kubernetes DNS/service discovery thường đã giải quyết phần lớn nhu cầu cơ bản.

Không nên thêm Eureka chỉ vì "microservices phải có Eureka".

---

# 76. Configuration Management

Khi nhiều service:

```text
user-service
order-service
payment-service
```

sẽ có rất nhiều config.

Có thể dùng:

```text
Spring Cloud Config
Vault
Kubernetes ConfigMap
Kubernetes Secret
```

Tùy hạ tầng.

---

# 77. Resilience

Khi service gọi service khác:

```text
order-service
     │
     ▼
payment-service
```

Payment có thể down.

Không nên để:

```text
payment down
    ↓
order-service treo vô hạn
```

Nên nghĩ tới:

```text
Timeout
Retry
Circuit Breaker
Bulkhead
Fallback
```

Có thể sử dụng Resilience4j.

---

# 78. Transaction trong microservices

Một transaction không nên mặc định trải dài:

```text
user DB
+
order DB
+
payment DB
```

với một transaction database duy nhất.

Thay vào đó có thể dùng:

```text
Saga Pattern
Outbox Pattern
Event-driven architecture
```

đặc biệt khi workflow phức tạp.

---

# 79. Outbox Pattern

Ví dụ:

```text
Order Service
    │
    ├── Save Order
    │
    └── Save Outbox Event
              │
              ▼
          Outbox table
              │
              ▼
           Publisher
              │
              ▼
            Kafka
```

Mục tiêu là tránh tình trạng:

```text
DB commit thành công
Kafka publish thất bại
```

dẫn đến mất event.

Đây là kiến thức nên học sau khi đã nắm chắc Spring Boot + JPA + Kafka.

---

# 80. Saga Pattern

Ví dụ order:

```text
Create Order
     │
     ▼
Reserve Inventory
     │
     ▼
Process Payment
     │
     ▼
Create Shipment
```

Nếu payment fail:

```text
compensating action
```

Ví dụ:

```text
Release Inventory
```

Saga giúp xử lý distributed transaction ở mức business workflow.

---

# 81. Roadmap học từ project này

Nếu bạn đang từ Fresher Java Backend tiến lên Junior/Middle, có thể học theo thứ tự:

## Level 1 — Maven

Nắm:

```text
pom.xml
groupId
artifactId
version
packaging
parent
modules
dependencies
dependencyManagement
plugin
lifecycle
repository
```

---

## Level 2 — Spring Boot

Nắm:

```text
IoC
DI
Bean
Configuration
REST API
Validation
Exception Handling
Profiles
Actuator
```

---

## Level 3 — Database

Nắm:

```text
PostgreSQL/MySQL
JPA
Hibernate
Transaction
Index
Query optimization
Migration
Flyway/Liquibase
```

---

## Level 4 — Security

Nắm:

```text
Spring Security
Authentication
Authorization
JWT
OAuth2/OIDC
CORS
CSRF
```

---

## Level 5 — Distributed System

Nắm:

```text
REST
Kafka
Redis
Idempotency
Retry
Timeout
Circuit Breaker
Outbox
Saga
```

---

## Level 6 — System Design

Nắm:

```text
Load Balancer
API Gateway
Caching
Database scaling
Replication
Partitioning
Message broker
Observability
CAP
Consistency
Availability
```

---

## Level 7 — Deployment

Nắm:

```text
Docker
Docker Compose
CI/CD
Kubernetes
Ingress
ConfigMap
Secret
HPA
Rolling Update
```

---

# 82. Checklist setup project

## Phase 1 — Project skeleton

- [ ] Cài JDK 21
- [ ] Cài Maven
- [ ] Tạo root project
- [ ] Tạo root `pom.xml`
- [ ] `packaging=pom`
- [ ] Khai báo `<modules>`
- [ ] Tạo `common`
- [ ] Tạo `user-service`
- [ ] Tạo `order-service`
- [ ] Tạo `notification-service`

---

## Phase 2 — Maven

- [ ] Hiểu parent
- [ ] Hiểu aggregator
- [ ] Hiểu dependencies
- [ ] Hiểu dependencyManagement
- [ ] Hiểu Maven lifecycle
- [ ] Hiểu Reactor
- [ ] Biết `-pl`
- [ ] Biết `-am`
- [ ] Biết `package`
- [ ] Biết `install`
- [ ] Thêm Maven Wrapper

---

## Phase 3 — Spring Boot

- [ ] Main class
- [ ] Controller
- [ ] Service
- [ ] Repository
- [ ] DTO
- [ ] Validation
- [ ] Global Exception Handler
- [ ] Configuration
- [ ] Profiles
- [ ] Actuator

---

## Phase 4 — Infrastructure

- [ ] PostgreSQL
- [ ] Redis
- [ ] Kafka
- [ ] Docker
- [ ] Docker Compose

---

## Phase 5 — Production

- [ ] Unit test
- [ ] Integration test
- [ ] Testcontainers
- [ ] CI/CD
- [ ] Logging
- [ ] Metrics
- [ ] Tracing
- [ ] Security
- [ ] Rate limiting
- [ ] Resilience
- [ ] Kubernetes

---

# 83. Recommended final structure

Một structure thực tế có thể là:

```text
microservices-parent/
│
├── .mvn/
│   └── wrapper/
│
├── .gitignore
├── mvnw
├── mvnw.cmd
├── pom.xml
├── README.md
│
├── common/
│   ├── pom.xml
│   └── src/
│
├── user-service/
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/company/userservice/
│       │   │   ├── UserServiceApplication.java
│       │   │   ├── user/
│       │   │   │   ├── controller/
│       │   │   │   ├── service/
│       │   │   │   ├── repository/
│       │   │   │   ├── entity/
│       │   │   │   └── dto/
│       │   │   ├── config/
│       │   │   ├── exception/
│       │   │   └── security/
│       │   └── resources/
│       │       └── application.yml
│       └── test/
│
├── order-service/
│   ├── pom.xml
│   └── src/
│
└── notification-service/
    ├── pom.xml
    └── src/
```

---

# 84. Root POM khuyến nghị cho giai đoạn đầu

Một root POM tương đối sạch:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
            http://maven.apache.org/POM/4.0.0
            https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>microservices-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>microservices-parent</name>
    <description>Microservices Maven Multi-module Project</description>

    <modules>
        <module>common</module>
        <module>user-service</module>
        <module>order-service</module>
        <module>notification-service</module>
    </modules>

    <properties>
        <java.version>21</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <dependencyManagement>
        <dependencies>

            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common</artifactId>
                <version>${project.version}</version>
            </dependency>

        </dependencies>
    </dependencyManagement>

</project>
```

Đây là điểm bắt đầu tốt. Đừng cố đưa tất cả công nghệ vào parent ngay lập tức.

---

# 85. Các nguyên tắc quan trọng nhất cần nhớ

## Rule 1

Root:

```xml
<packaging>pom</packaging>
```

---

## Rule 2

Root quản lý:

```text
version
dependency
plugin
module
build
```

---

## Rule 3

Service là application độc lập:

```text
user-service
order-service
notification-service
```

---

## Rule 4

Không share business logic giữa các service bằng Java dependency.

Không:

```text
order-service → user-service Java classes
```

Nên:

```text
order-service → REST/Kafka → user-service
```

---

## Rule 5

`common` chỉ chứa code generic.

Không biến nó thành:

```text
everything-common
```

---

## Rule 6

Mỗi service sở hữu data của nó.

```text
User Service → User DB
Order Service → Order DB
```

---

## Rule 7

Dependency phải thuộc về service sử dụng nó.

```text
Redis → service cần Redis
Kafka → service cần Kafka
JPA → service cần JPA
```

---

## Rule 8

Không hard-code secret.

Dùng:

```text
Environment Variable
Secret Manager
Kubernetes Secret
Vault
```

---

## Rule 9

Mỗi service có test riêng.

```text
user-service/src/test
order-service/src/test
notification-service/src/test
```

---

## Rule 10

Đừng over-engineer từ ngày đầu.

Bắt đầu:

```text
Maven
+
Spring Boot
+
REST
+
Database
```

Sau đó:

```text
Redis
Kafka
Security
Docker
CI/CD
Kubernetes
Observability
```

---

# 86. Thứ tự thực hành đề xuất

Nếu mục tiêu của bạn là trở thành Java Backend Engineer và hiểu Microservices thực tế, hãy build theo thứ tự:

```text
Step 1
Maven Multi-module
        ↓
Step 2
Spring Boot services
        ↓
Step 3
REST API
        ↓
Step 4
PostgreSQL + JPA
        ↓
Step 5
Flyway
        ↓
Step 6
Spring Security + JWT
        ↓
Step 7
Redis
        ↓
Step 8
Kafka
        ↓
Step 9
Docker Compose
        ↓
Step 10
API Gateway
        ↓
Step 11
Resilience
        ↓
Step 12
Observability
        ↓
Step 13
CI/CD
        ↓
Step 14
Kubernetes
```

Đây là thứ tự dễ học hơn so với việc dựng ngay một hệ thống có:

```text
Gateway
Eureka
Kafka
Redis
Kubernetes
Prometheus
Grafana
ELK
Vault
```

ngay từ ngày đầu.

---

# 87. Kết luận

Mô hình Maven Multi-module phù hợp để bắt đầu một monorepo microservices:

```text
                  Root POM
                     │
       ┌─────────────┼─────────────┐
       │             │             │
     common       user-service  order-service
                                   │
                                   │
                            notification-service
```

Hãy nhớ:

```text
Parent POM
    =
quản lý build/configuration

Aggregator
    =
quản lý modules

Service
    =
một application độc lập

Common
    =
chỉ chứa code generic

Microservices
    =
service boundary + data ownership
+ communication + deployment independence
```

Nếu nắm chắc các phần trên, bạn đã có nền tảng tốt để chuyển sang:

```text
Spring Boot
→ JPA
→ Security
→ Redis
→ Kafka
→ Docker
→ System Design
→ Kubernetes
```

và tiến dần từ Fresher → Junior → Middle Java Backend.
