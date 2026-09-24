# Quy Chuẩn Git Commit & Code Comment — Dự Án Uber App Microservices

> **Mục đích**: Tài liệu này quy định chuẩn hóa cách viết **Git Commit Message** (theo chuẩn quốc tế **Conventional Commits**) và **Quy tắc Comment Code** trong toàn bộ hệ thống dự án `uber-app`. Giúp lịch sử Git rõ ràng, dễ truy vết lỗi, hỗ trợ tự động sinh Changelog và làm việc nhóm chuyên nghiệp.

---

## 📋 MỤC LỤC

- [1. Cấu Trúc Chuẩn Của Một Git Commit Message](#1-cấu-trúc-chuẩn-của-một-git-commit-message)
- [2. Bảng Tra Cứu Các Loại Commit (Commit Types)](#2-bảng-tra-cứu-các-loại-commit-commit-types)
- [3. Quy Ước Phạm Vi (Scope) Cho Dự Án Uber App](#3-quy-ước-phạm-vi-scope-cho-dự-án-uber-app)
- [4. Chi Tiết Cách Viết Commit Theo Từng Tình Huống Cụ Thể](#4-chi-tiết-cách-viết-commit-theo-từng-tình-huống-cụ-thể)
  - [4.1. Thêm tính năng mới (feat)](#41-thêm-tính-năng-mới-feat)
  - [4.2. Sửa lỗi / Bug Fix (fix)](#42-sửa-lỗi--bug-fix-fix)
  - [4.3. Cập nhật tài liệu (docs)](#43-cập-nhật-tài-liệu-docs)
  - [4.4. Tái cấu trúc code (refactor)](#44-tái-cấu-trúc-code-refactor)
  - [4.5. Nâng cấp cấu hình & Thư viện (chore)](#45-nâng-cấp-cấu-hình--thư-viện-chore)
  - [4.6. Viết Unit / Integration Test (test)](#46-viết-unit--integration-test-test)
  - [4.7. Tối ưu hiệu năng (perf)](#47-tối-ưu-hiệu-năng-perf)
  - [4.8. Hoàn tác commit cũ (revert)](#48-hoàn-tác-commit-cũ-revert)
- [5. 7 Quy Tắc Vàng Khi Đặt Tiêu Đề Commit](#5-7-quy-tắc-vàng-khi-đặt-tiêu-đề-commit)
- [6. Quy Chuẩn Comment Trong Source Code Java](#6-quy-chuẩn-comment-trong-source-code-java)
  - [6.1. Javadoc Comment (Class, Interface & Public Method)](#61-javadoc-comment-class-interface--public-method)
  - [6.2. Inline Comment (Giải thích logic phức tạp)](#62-inline-comment-giải-thích-logic-phức-tạp)
  - [6.3. Đánh dấu TODO và FIXME](#63-đánh-dấu-todo-và-fixme)
- [7. Cheat Sheet Tra Cứu Nhanh](#7-cheat-sheet-tra-cứu-nhanh)

---

## 1. Cấu Trúc Chuẩn Của Một Git Commit Message

Một commit message chuẩn bao gồm 3 phần: **Header**, **Body** (tuỳ chọn), và **Footer** (tuỳ chọn):

```text
<type>(<scope>): <subject>

[optional <body>: Mô tả chi tiết lý do và sự thay đổi]

[optional <footer>: Mã task Jira/GitHub Issue, Breaking Changes]
```

### Ví dụ tổng quan:
```text
feat(ride): add api to create new ride request

- Implement POST /api/v1/rides in RideController
- Publish RideRequestEvent to Kafka topic "ride-requests"
- Persist ride entity to MySQL with initial status REQUESTED

Closes #12
```

---

## 2. Bảng Tra Cứu Các Loại Commit (Commit Types)

| Type | Ý nghĩa & Khi nào sử dụng | Biểu tượng gợi ý |
| :--- | :--- | :--- |
| **`feat`** | Thêm một tính năng mới cho ứng dụng | ✨ Feature |
| **`fix`** | Sửa một lỗi (bug) trong code hoặc hệ thống | 🐛 Bug Fix |
| **`docs`** | Thêm, sửa tài liệu (README, Markdown, Swagger doc) | 📝 Documentation |
| **`refactor`** | Sửa code nhưng không thêm tính năng, không sửa bug (tối ưu cấu trúc) | ♻️ Refactoring |
| **`perf`** | Cải thiện hiệu năng (tăng tốc độ truy vấn, giảm bộ nhớ) | ⚡ Performance |
| **`test`** | Thêm mới hoặc bổ sung Unit Test, Integration Test | 🧪 Tests |
| **`chore`** | Thay đổi phụ thuộc (POM, Gradle), tooling, scripts, file `.gitignore` | 🔧 Chores |
| **`style`** | Chỉnh sửa khoảng trắng, format code, import thừa (không đổi logic) | 💄 Formatting |
| **`ci`** | Thay đổi cấu hình CI/CD, GitHub Actions, Dockerfile | 👷 CI/CD |
| **`revert`** | Hoàn tác lại một commit trước đó do lỗi | ⏪ Revert |

---

## 3. Quy Ước Phạm Vi (Scope) Cho Dự Án Uber App

Phạm vi (`scope`) là module hoặc thành phần chịu ảnh hưởng trực tiếp của thay đổi:

| Scope | Áp dụng cho | Ví dụ |
| :--- | :--- | :--- |
| **`root`** | Thư mục gốc, file root POM, config toàn dự án | `chore(root): add maven wrapper configuration` |
| **`common`** | Module thư viện dùng chung `common` | `feat(common): add MatchFoundEvent payload` |
| **`location`** | Module `location-service` | `feat(location): add redis geospatial tracking for drivers` |
| **`matching`** | Module `matching-service` | `fix(matching): handle timeout when finding nearby drivers` |
| **`ride`** | Module `ride-service` | `feat(ride): create ride lifecycle state machine` |
| **`docker`** | File `docker-compose.yml`, Dockerfile hạ tầng | `chore(docker): configure kafka and redis container networks` |
| **`doc`** | Tài liệu hướng dẫn, file `.md` | `docs(doc): add system architecture and running guide` |

---

## 4. Chi Tiết Cách Viết Commit Theo Từng Tình Huống Cụ Thể

### 4.1. Thêm tính năng mới (`feat`)
Dùng khi bạn tạo mới một API, một Model, một Event Producer/Consumer, hoặc một logic nghiệp vụ.

* **Ví dụ 1 (Đơn giản - 1 dòng)**:
  ```bash
  git commit -m "feat(ride): add endpoint to cancel ongoing ride"
  ```
* **Ví dụ 2 (Đầy đủ Header & Body)**:
  ```text
  feat(location): implement driver coordinate update via redis geo

  - Add PUT /api/v1/drivers/location endpoint
  - Use Redis GEOADD command to store driver longitude and latitude
  - Set TTL expiration for inactive drivers
  ```

---

### 4.2. Sửa lỗi / Bug Fix (`fix`)
Dùng khi phát hiện lỗi logic, crash ứng dụng, sai dữ liệu, hoặc lỗi kết nối.

* **Ví dụ 1 (Sửa lỗi crash khi không tìm thấy tài xế)**:
  ```text
  fix(matching): prevent null pointer exception when no driver found

  Add validation check to ensure nearby driver list is not empty
  before attempting to extract the closest driver ID.
  
  Fixes #45
  ```
* **Ví dụ 2 (Sửa lỗi sai toạ độ)**:
  ```bash
  git commit -m "fix(location): swap inverted longitude and latitude parameters in redis geoadd"
  ```

---

### 4.3. Cập nhật tài liệu (`docs`)
Dùng khi bạn viết hoặc sửa file `.md`, tài liệu hướng dẫn cài đặt, thiết kế kiến trúc, chú thích API.

* **Ví dụ 1 (Cập nhật hướng dẫn chạy dự án)**:
  ```bash
  git commit -m "docs(doc): add step-by-step running guide with docker and intellij"
  ```
* **Ví dụ 2 (Cập nhật setup project)**:
  ```text
  docs(setup): update pom configuration explanation for common module

  - Clarify why spring-boot-maven-plugin needs skip true in common
  - Add sequence diagram for ride booking event flow
  ```

---

### 4.4. Tái cấu trúc code (`refactor`)
Dùng khi bạn dọn dẹp code, tách class, đổi tên hàm cho rõ nghĩa nhưng **không làm thay đổi hành vi** của chương trình.

* **Ví dụ 1 (Tách class xử lý logic)**:
  ```bash
  git commit -m "refactor(ride): extract kafka producer logic from RideServiceImpl to RideEventProducer"
  ```
* **Ví dụ 2 (Tối ưu cấu trúc package)**:
  ```text
  refactor(common): reorganize dtos and events into dedicated packages

  Move ApiResponse to dto package and RideRequestEvent to event package
  for cleaner modular separation.
  ```

---

### 4.5. Nâng cấp cấu hình & Thư viện (`chore`)
Dùng khi bạn chỉnh sửa file `pom.xml`, thêm/xóa thư viện, sửa file `.gitignore`, hoặc cập nhật file cấu hình YAML.

* **Ví dụ 1 (Xóa thư viện thừa trong POM)**:
  ```bash
  git commit -m "chore(deps): remove unused h2 and embedded-redis test dependencies"
  ```
* **Ví dụ 2 (Cập nhật .gitignore)**:
  ```bash
  git commit -m "chore(git): add ignore rules for target, idea and office temp files"
  ```
* **Ví dụ 3 (Nâng cấp phiên bản thư viện)**:
  ```bash
  git commit -m "chore(pom): upgrade spring-kafka version to 3.2.4 in root pom"
  ```

---

### 4.6. Viết Unit / Integration Test (`test`)
Dùng khi bạn viết thêm các ca kiểm thử hoặc sửa đổi test cũ.

* **Ví dụ 1**:
  ```bash
  git commit -m "test(ride): add unit tests for ride creation and fare calculation"
  ```
* **Ví dụ 2**:
  ```bash
  git commit -m "test(matching): add embedded kafka test for match-found event listener"
  ```

---

### 4.7. Tối ưu hiệu năng (`perf`)
Dùng khi bạn sửa code để ứng dụng chạy nhanh hơn, tốn ít RAM/CPU hơn hoặc giảm số query DB.

* **Ví dụ 1**:
  ```bash
  git commit -m "perf(location): use redis pipelining to batch driver coordinate updates"
  ```
* **Ví dụ 2**:
  ```bash
  git commit -m "perf(ride): add index on passenger_id and status columns in mysql rides table"
  ```

---

### 4.8. Hoàn tác commit cũ (`revert`)
Dùng khi commit trước bị lỗi nặng và cần quay xe về trạng thái cũ:

* **Cú pháp chuẩn**: `revert: <tiêu đề commit cũ>`
  ```bash
  git commit -m "revert: feat(matching): auto-assign driver without confirmation"
  ```

---

## 5. 7 Quy Tắc Vàng Khi Đặt Tiêu Đề Commit

> [!IMPORTANT]
> 1. **Dùng thể mệnh lệnh ở thì hiện tại**: Viết `add`, `fix`, `update`, `remove` (KHÔNG viết `added`, `fixed`, `updating`).
> 2. **Chữ cái đầu không viết hoa**: `feat(ride): add ride request api` (tránh viết `feat(ride): Add...`).
> 3. **Không đặt dấu chấm `.` ở cuối tiêu đề**: Viết ngắn gọn súc tích.
> 4. **Giới hạn tiêu đề <= 50-72 ký tự**: Nếu cần giải thích chi tiết, hãy xuống dòng viết ở phần **Body**.
> 5. **Tách biệt giữa What và Why**: Header nói lên *cái gì đã đổi*, phần Body giải thích *tại sao lại đổi*.
> 6. **Commit nhỏ và có mục đích rõ ràng**: Tránh gom 5 việc khác nhau vào 1 commit khổng lồ (ví dụ vừa sửa bug vừa format code vừa sửa doc).
> 7. **Ngôn ngữ nhất quán**: Toàn bộ dự án nên chọn tiếng Anh (chuẩn quốc tế) hoặc tiếng Việt thống nhất theo quy ước team.

---

## 6. Quy Chuẩn Comment Trong Source Code Java

Bên cạnh Git commit, comment trong mã nguồn Java của dự án cũng cần tuân thủ quy tắc rõ ràng:

### 6.1. Javadoc Comment (Class, Interface & Public Method)
Dùng cú pháp `/** ... */` cho các Class, Interface, Controller và Service methods công khai:

```java
/**
 * Service xử lý toàn bộ vòng đời cuốc xe (Ride Lifecycle).
 * Tiếp nhận yêu cầu đặt xe, lưu trữ vào MySQL và phát sự kiện sang Kafka.
 *
 * @author Team Uber
 * @version 1.0.0
 */
public interface RideService {

    /**
     * Tạo mới một yêu cầu đặt xe và kích hoạt quy trình tìm tài xế.
     *
     * @param request DTO chứa toạ độ điểm đón, điểm đến và mã hành khách
     * @return Thông tin cuốc xe vừa tạo với trạng thái REQUESTED
     * @throws BusinessException nếu toạ độ không hợp lệ hoặc khách hàng đang có cuốc chưa hoàn thành
     */
    RideResponse createRide(CreateRideRequest request);
}
```

---

### 6.2. Inline Comment (Giải thích logic phức tạp)
Dùng cú pháp `// ...` để giải thích **TẠI SAO** làm như vậy (Why), không giải thích lại **CÁI ĐANG LÀM** (What) vì code đã tự nói lên điều đó:

```java
// ❌ SAI: Comment những điều hiển nhiên, vô nghĩa
// Tăng biến i lên 1
i++;

// Đặt trạng thái chuyến đi thành MATCHED
ride.setStatus(RideStatus.MATCHED);


// ✅ ĐÚNG: Giải thích nguyên nhân hoặc quyết định kỹ thuật
// Giới hạn bán kính quét tối đa 5km để tránh quá tải Redis Geo và đảm bảo thời gian đón < 10 phút
double searchRadiusKm = 5.0;

// Sử dụng Outbox Pattern: Lưu event vào bảng trung gian trước khi gửi Kafka 
// nhằm đảm bảo tính toàn vẹn dữ liệu nếu Kafka broker tạm thời mất kết nối
outboxRepository.save(new OutboxEvent("ride-requests", payload));
```

---

### 6.3. Đánh dấu TODO và FIXME
Sử dụng tiền tố chuẩn để các IDE (IntelliJ, VS Code) nhận diện và gom nhóm:

* **`// TODO: <nội dung>`**: Đánh dấu tính năng cần bổ sung trong tương lai.
  ```java
  // TODO: Tích hợp Google Maps Distance Matrix API để tính giá cước chính xác theo quãng đường thực tế
  ```
* **`// FIXME: <nội dung>`**: Đánh dấu đoạn code tạm bợ cần sửa gấp hoặc tiềm ẩn rủi ro.
  ```java
  // FIXME: Đang hardcode phí cước cơ bản là 20.000 VND, cần chuyển sang Dynamic Pricing Service
  ```

---

## 7. Cheat Sheet Tra Cứu Nhanh

Hãy copy và áp dụng ngay các mẫu commit thông dụng dưới đây:

| Tình huống thực tế | Mẫu câu commit chuẩn |
| :--- | :--- |
| Cập nhật file markdown hướng dẫn | `docs(setup): update running guide for intellij and docker` |
| Thêm DTO/Response mới vào `common` | `feat(common): add driver location dto and api response wrapper` |
| Thêm API tìm tài xế lân cận | `feat(location): add GET /api/v1/drivers/nearby endpoint` |
| Sửa lỗi Kafka deserializer | `fix(matching): resolve json deserializer trusted package error` |
| Sửa lỗi kết nối MySQL | `fix(ride): fix database url timezone parameter in application.yml` |
| Tách nhỏ hàm dài trong Service | `refactor(ride): split ride validation into separate validator class` |
| Xóa dependency thừa khỏi file POM | `chore(deps): remove unnecessary test dependencies in matching-service` |
| Cấu hình container Redis trong docker | `chore(docker): update redis port mapping and persistence volume` |
| Thêm test cho API tạo cuốc xe | `test(ride): add integration test for POST /api/v1/rides` |
| Format lại code toàn bộ service | `style(location): reformat code according to project style guide` |
