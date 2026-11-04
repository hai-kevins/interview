# Secure Delta OTA Bootloader

> **Tài liệu ôn tập & trình bày project**  
> Nền tảng: **STM32F103 + ESP32 Gateway + External Flash W25Q**  
> Nội dung được chuyển từ tài liệu ODT, bố cục lại để đọc trên GitHub, VS Code hoặc Obsidian.

---

## Mục lục

1. [Trình bày project](#1-trình-bày-project)
2. [Trình bày System Architecture](#2-trình-bày-system-architecture)
3. [Phân tích Developer/CI](#3-phân-tích-developerci)
4. [Kiến thức trong project](#4-kiến-thức-trong-project)
5. [Câu hỏi trọng tâm](#5-câu-hỏi-trọng-tâm)

---

## 1. Trình bày project

### Phần 1. Giới thiệu tổng quan

Dạ, project đầu tiên của em là **Secure Delta OTA Bootloader**, được phát triển trên **STM32F103** kết hợp với **ESP32 Gateway**.

Mục tiêu của project là cho phép cập nhật firmware từ xa một cách an toàn, đồng thời giảm dung lượng dữ liệu truyền khi cập nhật bằng cơ chế **Delta OTA**.

### Phần 2. Kiến trúc và cách hoạt động

Về kiến trúc, **ESP32** đóng vai trò Gateway, kết nối với server để tải firmware, sau đó truyền xuống STM32 thông qua **UART**.

Trên STM32, **Application** tiếp nhận dữ liệu cập nhật và lưu vào Flash ngoài **W25Q** thông qua **SPI**.

Sau đó, **Bootloader** sẽ xác thực firmware, xử lý bản cập nhật, sao lưu firmware cũ và cài đặt firmware mới vào **Internal Flash**.

### Phần 3. Công việc thực hiện

Trong project này, em triển khai hai cơ chế là **Full OTA** và **Delta OTA**.

Với Delta OTA, hệ thống chỉ truyền phần thay đổi giữa hai phiên bản firmware, sau đó tái tạo firmware mới từ bản cũ.

Em sử dụng **ECDSA P-256** kết hợp **SHA-256** để xác thực và kiểm tra tính toàn vẹn của firmware, đồng thời triển khai cơ chế chống cập nhật xuống phiên bản cũ không được phép.

### Phần 4. Khó khăn và cách giải quyết

Một khó khăn lớn trong project là xử lý trường hợp thiết bị mất nguồn khi đang cập nhật, khiến firmware có thể bị ghi chưa hoàn chỉnh.

Để giải quyết, em sử dụng **State Machine** kết hợp **Metadata** và **Checkpoint**, giúp Bootloader khôi phục quá trình cập nhật sau khi khởi động lại.

Ngoài ra, em triển khai cơ chế **Rollback** để khôi phục firmware đã sao lưu nếu phiên bản mới không hoạt động đúng.

### Phần 5. Kết quả đạt được

Kết quả thử nghiệm với **GCC** cho thấy Delta OTA giảm khoảng **85,3%** dung lượng dữ liệu cập nhật so với Full OTA.

Hệ thống cũng ghi nhận **9/9 kịch bản kiểm thử trên phần cứng thành công**, bao gồm các tình huống mất nguồn và cập nhật thất bại.

Qua project này, em hiểu rõ hơn về bootloader, quản lý bộ nhớ Flash, giao tiếp UART, SPI và cách xây dựng hệ thống cập nhật firmware an toàn.

---

## 2. Trình bày System Architecture

Về **System Architecture**, hệ thống của em bao gồm **Developer/CI**, **Release Server**, **ESP32 Gateway**, **STM32 Application** và **STM32 Bootloader**. Em sẽ trình bày luồng hoạt động từ trên xuống.

- **Developer/CI:** Đầu tiên, Developer/CI sẽ build firmware, tạo gói cập nhật **Full OTA** hoặc **Delta OTA**, sau đó đóng gói theo định dạng **SDOT** và ký số bằng **ECDSA P-256**. Đồng thời, hệ thống tạo **Manifest** chứa thông tin bản phát hành trước khi đưa lên Release Server.

- **Release Server:** Tiếp theo, Release Server chịu trách nhiệm xác minh và phân phối các gói cập nhật. Server sử dụng **MQTTS** để thông báo bản cập nhật mới và **HTTPS** để cung cấp gói cập nhật cho ESP32 Gateway.

- **ESP32 Gateway:** Khi nhận được thông báo qua MQTTS, ESP32 Gateway sẽ tải gói cập nhật SDOT thông qua HTTPS, sau đó truyền xuống STM32 Application qua **UART**, sử dụng **COBS** để đóng khung dữ liệu và **CRC32** để phát hiện lỗi truyền.

- **STM32 Application:** STM32 Application tiếp nhận các gói dữ liệu qua UART và lưu vào **External Flash W25Q** thông qua **SPI**. Sau khi nhận đủ gói cập nhật, STM32 Application cập nhật **Metadata** và reset thiết bị để chuyển quyền xử lý cho STM32 Bootloader.

- **STM32 Bootloader:** Khi khởi động, STM32 Bootloader đọc Metadata để xác định trạng thái cập nhật, sau đó xác thực chữ ký **ECDSA P-256** và kiểm tra tính toàn vẹn bằng **SHA-256**. Nếu là Delta OTA, Bootloader kiểm tra firmware gốc và tái tạo firmware mới từ Delta Patch. Sau khi firmware mới được kiểm tra hợp lệ, Bootloader sao lưu firmware hiện tại, cài đặt firmware mới vào **Internal Flash** và kiểm tra lại kết quả.

  Cuối cùng, Bootloader cho STM32 Application mới chạy ở chế độ **Trial Boot**. Nếu Application xác nhận hoạt động thành công, bản cập nhật được chấp nhận. Ngược lại, nếu không được xác nhận sau số lần thử cho phép, Bootloader sẽ thực hiện **Rollback** để khôi phục firmware đã sao lưu.

  Ngoài ra, hệ thống sử dụng **Metadata** và **Checkpoint** để khôi phục quá trình cập nhật khi xảy ra mất nguồn hoặc reset bất ngờ.

### Tóm tắt luồng dữ liệu

```text
Developer/CI
    │ Tạo Full/Delta OTA, đóng gói SDOT, ký số, tạo Manifest
    ▼
Release Server
    │ MQTTS: thông báo cập nhật
    │ HTTPS: cung cấp gói cập nhật
    ▼
ESP32 Gateway
    │ UART + COBS + CRC32
    ▼
STM32 Application
    │ SPI: lưu SDOT vào External Flash W25Q
    │ Metadata + Reset
    ▼
STM32 Bootloader
    │ Xác thực → Tái tạo Delta (nếu có) → Sao lưu → Cài đặt
    ▼
Trial Boot → Xác nhận / Rollback
```

---

## 3. Phân tích Developer/CI

> **Tình huống minh họa:** STM32 đang chạy firmware **v1**, Developer muốn cập nhật lên **v2**.

### Bước 1. Build firmware

Developer chỉnh sửa mã nguồn Application, sau đó hệ thống build tạo ra firmware mới: **`application-v2.bin`**. Đây là file binary sẽ được ghi vào vùng Application của STM32 Internal Flash.

**CI (Continuous Integration)** có thể tự động thực hiện các bước build, kiểm tra và chuẩn bị phát hành firmware.

### Bước 2. Tạo Full OTA và Delta OTA

| Full OTA | Delta OTA |
|---|---|
| Chứa toàn bộ firmware v2 | Chứa patch từ firmware v1 sang v2 |
| Không cần firmware gốc để tạo gói | Cần firmware v1 chính xác |
| Dung lượng thường lớn hơn | Có thể giảm đáng kể dung lượng |

Với **Delta OTA**, công cụ sử dụng **JojoDiff** để tạo patch từ firmware v1 và v2.

Repository cũng kiểm tra lại bằng cách áp dụng patch lên firmware v1, sau đó so sánh kết quả với firmware v2.

**Lưu ý:** Repository luôn tạo Full OTA, nhưng chỉ đưa Delta OTA vào release nếu mức tiết kiệm đạt ngưỡng cấu hình, mặc định là **20%**.

### Bước 3. Đóng gói và ký firmware

Cả Full OTA và Delta OTA đều được đóng gói theo định dạng **SDOT (Secure Delta OTA Container)**.

**Cấu trúc cơ bản của SDOT Container:**

```text
+-----------------------------+
| Header                      |
| - Firmware type             |
| - Base version              |
| - Target version            |
| - Base SHA-256              |
| - Target SHA-256            |
| - Product ID                |
+-----------------------------+
| Payload                     |
| Full firmware / Delta patch |
+-----------------------------+
| ECDSA P-256 Signature       |
+-----------------------------+
```

Hệ thống sử dụng **SHA-256** để tạo các giá trị hash phục vụ kiểm tra tính toàn vẹn và **ECDSA P-256** để ký dữ liệu container bằng **private key**.

> **Ghi nhớ:** Private key chỉ được sử dụng ở phía phát hành firmware, **không truyền xuống ESP32 hoặc STM32**.

### Bước 4. Tạo Manifest

Sau khi có các gói SDOT, release tooling tạo file **`manifest.json`**.

Manifest chứa thông tin mô tả bản phát hành, chẳng hạn:

- Phiên bản firmware mới.
- Product ID và hardware revision.
- Thông tin các gói Full/Delta.
- Kích thước và SHA-256 của artifact.
- URL để tải artifact.
- Thông tin khóa ký.

Manifest cũng được ký ECDSA và lưu chữ ký riêng trong **`manifest.json.sig`**.

> **Phân biệt:** **SDOT** chứa firmware hoặc patch đã ký số; **Manifest** mô tả các artifact và thông tin phát hành.

### Bước 5. Chuyển sang Release Server

Một release được tạo có cấu trúc tương tự:

```text
dist/releases/fw-v2/
│
├── application-v2.bin
├── application-v2.full.sdot
├── application-v1-to-v2.delta.sdot
│
├── manifest.json
├── manifest.json.sig
├── signing-public.pem
├── checksums.txt
└── release-notes.md
```

Sau khi tạo release, công cụ kiểm tra tính hợp lệ của các file.

**Release Server** sau đó sử dụng thư mục release đã được triển khai để:

- Xác minh chữ ký Manifest và các artifact.
- Cung cấp Manifest và firmware qua HTTPS.
- Chuẩn bị và gửi thông báo cập nhật qua MQTTS.

### Câu hỏi ôn tập: Developer/CI

| Câu hỏi | Ý chính cần trả lời |
|---|---|
| CI là gì? | Hệ thống tự động hóa build, kiểm thử và chuẩn bị release. |
| Tại sao cần firmware v1 khi tạo Delta OTA? | Vì patch được tạo từ sự khác biệt giữa firmware v1 và v2. |
| Tại sao cần SDOT Container? | Để đóng gói firmware cùng metadata bảo mật và chữ ký số. |
| Tại sao phải sử dụng ECDSA? | Để xác thực firmware được ký bằng private key tương ứng với public key tin cậy. |
| SHA-256 dùng để làm gì? | Kiểm tra tính toàn vẹn của dữ liệu firmware. |
| Manifest dùng để làm gì? | Mô tả bản phát hành và giúp hệ thống lựa chọn, tải artifact. |
| Server có tạo firmware không? | Không. Release tooling tạo artifact; server xác minh và phân phối. |
| Server có giữ private key không? | Không cần private key ký firmware để xác minh và phân phối release. |

---

## 4. Kiến thức trong project

*Chưa có nội dung trong tài liệu gốc.*

## 5. Câu hỏi trọng tâm

*Chưa có nội dung trong tài liệu gốc.*

---

> **Ghi chú chuyển đổi:** Giữ nguyên nội dung và số liệu trong tài liệu ODT; chuẩn hóa tiêu đề, danh sách, bảng và khối mã cho Markdown. Sửa lỗi gõ `Buil firmware` thành `Build firmware`.
