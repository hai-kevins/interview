# Mục lục

- [1. Cortex-M3 Boot Process](#1-cortex-m3-boot-process)
  - [1.1. Bootloader là gì?](#11-bootloader-là-gì)
  - [1.2. Cortex-M3 khởi động như thế nào?](#12-cortex-m3-khởi-động-như-thế-nào)
  - [1.3. Bảng vector (Vector Table)](#13-bảng-vector-vector-table)
  - [1.4. MSP – Con trỏ ngăn xếp chính (Main Stack Pointer)](#14-msp-con-trỏ-ngăn-xếp-chính-main-stack-pointer)
  - [1.5. Reset_Handler](#15-resethandler)
  - [1.6. `.data` và `.bss`](#16-data-và-bss)
  - [1.7. VTOR – Thanh ghi độ lệch bảng vector (Vector Table Offset Register)](#17-vtor-thanh-ghi-độ-lệch-bảng-vector-vector-table-offset-register)
  - [1.8. Bit Thumb](#18-bit-thumb)
  - [1.9. Mô hình tư duy cần nhớ](#19-mô-hình-tư-duy-cần-nhớ)
  - [1.10. Trọng tâm cần thuộc](#110-trọng-tâm-cần-thuộc)
- [2. Linker Script + STM32 Memory Map](#2-linker-script-+-stm32-memory-map)
  - [2.1. Memory Map là gì?](#21-memory-map-là-gì)
  - [2.2. Linker Script là gì?](#22-linker-script-là-gì)
  - [2.3. Linker Script của Bootloader](#23-linker-script-của-bootloader)
  - [2.4. Linker Script của Application](#24-linker-script-của-application)
  - [2.5. `ENTRY(Reset_Handler)`](#25-entryresethandler)
  - [2.6. Các section chính](#26-các-section-chính)
  - [2.7. `.isr_vector`](#27-isrvector)
  - [2.8. Tại sao dùng `KEEP()`?](#28-tại-sao-dùng-keep)
  - [2.9. `.text` và `.rodata`](#29-text-và-rodata)
  - [2.10. `.data`](#210-data)
  - [2.11. `.bss`](#211-bss)
  - [2.12. Linker Symbol](#212-linker-symbol)
  - [2.13. Stack và Heap](#213-stack-và-heap)
  - [2.14. `ASSERT()` trong Linker Script](#214-assert-trong-linker-script)
  - [2.15. Vì sao Bootloader và Application cùng dùng toàn bộ RAM?](#215-vì-sao-bootloader-và-application-cùng-dùng-toàn-bộ-ram)
  - [2.16. External W25Q có nằm trong Linker Script không?](#216-external-w25q-có-nằm-trong-linker-script-không)
  - [2.17. Quan hệ giữa Memory Map, Linker Script và Bootloader](#217-quan-hệ-giữa-memory-map-linker-script-và-bootloader)
  - [2.18. Các lỗi thường gặp](#218-các-lỗi-thường-gặp)
  - [2.19. Mô hình tư duy cần nhớ](#219-mô-hình-tư-duy-cần-nhớ)
  - [2.20. Trọng tâm cần thuộc](#220-trọng-tâm-cần-thuộc)
- [3. Internal Flash Erase / Program / Verify](#3-internal-flash-erase-program-verify)
  - [3.1. Flash nội và bản đồ bộ nhớ](#31-flash-nội-và-bản-đồ-bộ-nhớ)
  - [3.2. Đơn vị xóa (Erase) và lập trình (Program)](#32-đơn-vị-xóa-erase-và-lập-trình-program)
  - [3.3. Các thanh ghi Flash quan trọng](#33-các-thanh-ghi-flash-quan-trọng)
  - [3.4. Quy trình xóa/lập trình Flash](#34-quy-trình-xóalập-trình-flash)
  - [3.5. Xác minh bằng cách đọc lại](#35-xác-minh-bằng-cách-đọc-lại)
  - [3.6. Bộ đệm theo trang trong SRAM](#36-bộ-đệm-theo-trang-trong-sram)
  - [3.7. Điểm kiểm tra theo trang](#37-điểm-kiểm-tra-theo-trang)
  - [3.8. Khôi phục sau mất nguồn](#38-khôi-phục-sau-mất-nguồn)
  - [3.9. Tính lặp an toàn (Idempotency)](#39-tính-lặp-an-toàn-idempotency)
  - [3.10. Sao lưu trước khi cài đặt](#310-sao-lưu-trước-khi-cài-đặt)
  - [3.11. Metadata A/B](#311-metadata-ab)
  - [3.12. CRC ghi cuối Metadata](#312-crc-ghi-cuối-metadata)
  - [3.13. Độ bền Flash](#313-độ-bền-flash)
  - [3.14. Flash nội và W25Q ngoài](#314-flash-nội-và-w25q-ngoài)
  - [3.15. Luồng cài đặt firmware đầy đủ](#315-luồng-cài-đặt-firmware-đầy-đủ)
  - [3.16. Các lỗi thường gặp](#316-các-lỗi-thường-gặp)
  - [3.17. Trọng tâm cần thuộc](#317-trọng-tâm-cần-thuộc)
- [4. Bootloader → Application Jump](#4-bootloader-application-jump)
  - [4.1. Kiểm tra tính hợp lệ của ứng dụng](#41-kiểm-tra-tính-hợp-lệ-của-ứng-dụng)
  - [4.2. Tại sao phải dọn trạng thái trước khi chuyển sang ứng dụng?](#42-tại-sao-phải-dọn-trạng-thái-trước-khi-chuyển-sang-ứng-dụng)
  - [4.3. SysTick và ngắt](#43-systick-và-ngắt)
  - [4.4. Trạng thái xung nhịp](#44-trạng-thái-xung-nhịp)
  - [4.5. Vì sao không nên đổi MSP rồi tiếp tục chạy C?](#45-vì-sao-không-nên-đổi-msp-rồi-tiếp-tục-chạy-c)
  - [4.6. CONTROL, PRIMASK, BASEPRI, FAULTMASK](#46-control-primask-basepri-faultmask)
  - [4.7. Trình tự chuyển giao đầy đủ](#47-trình-tự-chuyển-giao-đầy-đủ)
  - [4.8. Quyết định khởi động](#48-quyết-định-khởi-động)
  - [4.9. Các lỗi Bootloader thường gặp](#49-các-lỗi-bootloader-thường-gặp)
- [5. W25Q SPI NOR: Erase / Program / Read / Status](#5-w25q-spi-nor-erase-program-read-status)
  - [5.1. Giao tiếp SPI](#51-giao-tiếp-spi)
  - [5.2. JEDEC ID](#52-jedec-id)
  - [5.3. Các lệnh quan trọng](#53-các-lệnh-quan-trọng)
  - [5.4. Quy tắc của NOR Flash](#54-quy-tắc-của-nor-flash)
  - [5.5. Page và Sector](#55-page-và-sector)
  - [5.6. Write Enable và WEL](#56-write-enable-và-wel)
  - [5.7. Status Register và BUSY](#57-status-register-và-busy)
  - [5.8. Read Data](#58-read-data)
  - [5.9. Page Program](#59-page-program)
  - [5.10. Page Boundary](#510-page-boundary)
  - [5.11. Sector Erase](#511-sector-erase)
  - [5.12. Đọc lại để xác minh](#512-đọc-lại-để-xác-minh)
  - [5.13. Bản đồ phân vùng W25Q và vai trò từng vùng](#513-bản-đồ-phân-vùng-w25q-và-vai-trò-từng-vùng)
  - [5.14. Vai trò của W25Q trong OTA](#514-vai-trò-của-w25q-trong-ota)
  - [5.15. Flash nội và W25Q khác nhau](#515-flash-nội-và-w25q-khác-nhau)
  - [5.16. Các lỗi thường gặp](#516-các-lỗi-thường-gặp)
  - [5.17. Mô hình tư duy cần nhớ](#517-mô-hình-tư-duy-cần-nhớ)
  - [5.18. Trọng tâm cần thuộc](#518-trọng-tâm-cần-thuộc)
- [6. CRC32 + UART Framing + COBS](#6-crc32-+-uart-framing-+-cobs)
  - [6.1. Tại sao UART cần Framing?](#61-tại-sao-uart-cần-framing)
  - [6.2. COBS và byte phân cách `0x00`](#62-cobs-và-byte-phân-cách-0x00)
  - [6.3. Vì sao không dùng trực tiếp `0x00` nếu không có COBS?](#63-vì-sao-không-dùng-trực-tiếp-0x00-nếu-không-có-cobs)
  - [6.4. Luồng nhận COBS trên STM32](#64-luồng-nhận-cobs-trên-stm32)
  - [6.5. Cấu trúc Raw Packet](#65-cấu-trúc-raw-packet)
  - [6.6. Ý nghĩa các field quan trọng](#66-ý-nghĩa-các-field-quan-trọng)
  - [6.7. CRC32 dùng để làm gì?](#67-crc32-dùng-để-làm-gì)
  - [6.8. Thứ tự đóng gói và giải mã](#68-thứ-tự-đóng-gói-và-giải-mã)
  - [6.9. DATA Packet và `offset`](#69-data-packet-và-offset)
  - [6.10. CRC phải đúng trước khi ghi W25Q](#610-crc-phải-đúng-trước-khi-ghi-w25q)
  - [6.11. ACK / NACK](#611-ack-nack)
  - [6.12. Retry và packet trùng lặp](#612-retry-và-packet-trùng-lặp)
  - [6.13. Packet CRC và Artifact CRC](#613-packet-crc-và-artifact-crc)
  - [6.14. CRC32 không phải cơ chế bảo mật](#614-crc32-không-phải-cơ-chế-bảo-mật)
  - [6.15. Vai trò của từng thành phần](#615-vai-trò-của-từng-thành-phần)
  - [6.16. Luồng OTA UART hoàn chỉnh](#616-luồng-ota-uart-hoàn-chỉnh)
  - [6.17. Ví dụ đầy đủ: gửi một DATA packet từ ESP32 xuống W25Q](#617-ví-dụ-đầy-đủ-gửi-một-data-packet-từ-esp32-xuống-w25q)
  - [6.18. Trọng tâm cần thuộc](#618-trọng-tâm-cần-thuộc)
- [7. OTA State Machine + Persistent Metadata A/B](#7-ota-state-machine-+-persistent-metadata-ab)
- [8. Power-Loss Recovery + Checkpoint + Rollback](#8-power-loss-recovery-+-checkpoint-+-rollback)
- [9. SHA-256 + ECDSA P-256 + Public / Private Key](#9-sha-256-+-ecdsa-p-256-+-public-private-key)
- [10. Firmware Container + Anti-Rollback](#10-firmware-container-+-anti-rollback)
- [11. Delta Patch / JojoDiff Concept](#11-delta-patch-jojodiff-concept)
- [12. ESP32 + MQTT + HTTPS + Server / Release Tooling](#12-esp32-+-mqtt-+-https-+-server-release-tooling)

---

# 1. Cortex-M3 Boot Process

## 1.1. Bootloader là gì?

Bootloader là firmware chạy đầu tiên sau khi STM32 reset khi MCU boot từ Main Flash. Trong dự án này, bootloader nằm tại `0x08000000`, còn application được đặt tại `0x08006000`.

Bootloader có nhiệm vụ đọc trạng thái hệ thống, kiểm tra firmware, xử lý OTA/khôi phục/rollback và chỉ chuyển quyền điều khiển sang ứng dụng khi ứng dụng hợp lệ.

```text
Reset
  ↓
Bootloader
  ↓
Kiểm tra trạng thái / firmware
  ↓
Application
```

---

## 1.2. Cortex-M3 khởi động như thế nào?

Sau reset, Cortex-M3 không chạy trực tiếp vào `main()`.

CPU đọc hai word đầu tiên của vector table:

```text
vector[0] = Initial MSP
vector[1] = Reset_Handler
```

Sau đó CPU:

1. Nạp `vector[0]` vào Main Stack Pointer (MSP).
2. Nạp `vector[1]` vào Program Counter.
3. Bắt đầu thực thi `Reset_Handler`.

Với bootloader:

```text
0x08000000 -> Initial MSP của bootloader
0x08000004 -> Reset_Handler của bootloader
```

---

## 1.3. Bảng vector (Vector Table)

Vector table chứa địa chỉ các exception và interrupt handler.

```text
+0x00   Initial MSP
+0x04   Reset_Handler
+0x08   NMI_Handler
+0x0C   HardFault_Handler
...
       SysTick_Handler
       USART_IRQHandler
       Timer_IRQHandler
```

Phần tử đầu tiên không phải function pointer mà là giá trị stack pointer ban đầu.

Ứng dụng trong dự án có bảng vector riêng tại:

```text
0x08006000
```

Do đó bootloader biết:

```text
[0x08006000] -> MSP của ứng dụng
[0x08006004] -> Reset_Handler của ứng dụng
```

---

## 1.4. MSP – Con trỏ ngăn xếp chính (Main Stack Pointer)

STM32F103C8T6 có SRAM:

```text
0x20000000 -> 0x20004FFF
```

Initial MSP thường được đặt ngay phía trên SRAM:

```text
0x20005000
```

Điều này hợp lệ vì stack phát triển theo hướng địa chỉ giảm.

```text
MSP ban đầu = 0x20005000

push dữ liệu
      ↓
MSP = 0x20004FFC
```

Trước khi chuyển sang ứng dụng, bootloader phải kiểm tra MSP của application:

- nằm trong vùng SRAM;
- có alignment hợp lệ.

Nếu MSP sai, application có thể fault ngay khi sử dụng stack.

---

## 1.5. Reset_Handler

`Reset_Handler` là entry point thật sự của firmware.

Luồng thông thường:

```text
Reset
  ↓
Reset_Handler
  ↓
Khởi tạo .data
  ↓
Zero .bss
  ↓
SystemInit()
  ↓
C runtime
  ↓
main()
```

Vì vậy bootloader phải chuyển quyền điều khiển tới `Reset_Handler`, không nên chuyển thẳng tới `main()`.

---

## 1.6. `.data` và `.bss`

### `.data`

Là các biến global/static có giá trị khởi tạo.

```c
uint32_t counter = 123;
```

Giá trị ban đầu được lưu trong Flash, sau đó mã khởi động sao chép vào RAM.

```text
Flash -> RAM
```

### `.bss`

Là các biến global/static không có giá trị khởi tạo rõ ràng.

```c
uint32_t errors;
```

Mã khởi động đưa vùng `.bss` về 0 trước khi vào `main()`.

---

## 1.7. VTOR – Thanh ghi độ lệch bảng vector (Vector Table Offset Register)

`SCB->VTOR` cho CPU biết vector table hiện tại nằm ở đâu.

Khi bootloader đang chạy:

```text
VTOR = 0x08000000
```

Khi chuyển sang application:

```text
VTOR = 0x08006000
```

Nếu quên đổi VTOR, application có thể chạy được cho đến khi interrupt xảy ra. Khi đó CPU có thể lấy handler từ vector table của bootloader thay vì application.

Do đó trước khi chuyển sang ứng dụng:

```c
SCB->VTOR = APPLICATION_ADDRESS;
```

Sau đó thường dùng:

```c
__DSB();
__ISB();
```

để đồng bộ thay đổi trạng thái hệ thống.

---

## 1.8. Bit Thumb

Cortex-M3 chỉ chạy Thumb/Thumb-2.

Địa chỉ exception handler trong vector table phải có bit 0 bằng `1`.

Ví dụ:

```text
Reset_Handler thực tế: 0x08006230
Vector chứa:           0x08006231
```

Khi kiểm tra range:

```c
code_address = reset_handler & ~1U;
```

Bootloader nên kiểm tra:

```text
reset_handler & 1 == 1
```

Nếu không, reset vector không hợp lệ cho Cortex-M.

---

## 1.9. Mô hình tư duy cần nhớ

```text
RESET
  ↓
CPU đọc Bootloader Vector Table
  ↓
Load MSP
  ↓
Bootloader Reset_Handler
  ↓
Bootloader main()
  ↓
Đọc metadata khởi động
  ↓
Quyết định khởi động
  ↓
kiểm tra tính hợp lệ của ứng dụng
  ↓
Read:
[0x08006000] -> MSP của ứng dụng
[0x08006004] -> Reset_Handler của ứng dụng
  ↓
Dừng SysTick
Tắt/xóa trạng thái ngắt
Đặt lại trạng thái xung nhịp
  ↓
VTOR = 0x08006000
  ↓
MSP = MSP của ứng dụng
  ↓
BX Reset_Handler của ứng dụng
  ↓
quá trình khởi động ứng dụng
  ↓
main()
```

---

## 1.10. Trọng tâm cần thuộc

```text
Reset
↓
Vector Table
↓
MSP
↓
Reset_Handler
↓
Linker Script
↓
địa chỉ ứng dụng
↓
VTOR
↓
Thumb Bit
↓
kiểm tra tính hợp lệ của ứng dụng
↓
Dừng SysTick / Clear Interrupt
↓
Reset CPU Context
↓
Set MSP
↓
BX Reset_Handler
↓
main() của ứng dụng
```

Nếu giải thích được toàn bộ chuỗi trên mà không nhìn tài liệu, bạn đã nắm phần cốt lõi của STM32 bootloader handoff.

---

# 2. Linker Script + STM32 Memory Map

## 2.1. Memory Map là gì?

Memory Map là bản đồ phân chia toàn bộ bộ nhớ của MCU thành các vùng có mục đích rõ ràng.

Trong dự án này:

```text
0x08000000
+---------------------------+
| Bootloader - 24 KiB       |
+---------------------------+ 0x08006000
| Application - 38 KiB      |
+---------------------------+ 0x0800F800
| Metadata A - 1 KiB        |
+---------------------------+ 0x0800FC00
| Metadata B - 1 KiB        |
+---------------------------+ 0x08010000
```

SRAM:

```text
0x20000000
      ↓
    20 KiB
      ↓
0x20004FFF
```

Memory Map là **hợp đồng bộ nhớ** của toàn hệ thống. Bootloader, application, linker script và OTA installer phải dùng cùng các địa chỉ.

---

## 2.2. Linker Script là gì?

Compiler tạo ra các object file nhưng chưa quyết định chính xác code và dữ liệu sẽ nằm ở đâu trong bộ nhớ.

Linker Script `.ld` quy định:

```text
Flash bắt đầu ở đâu?
RAM bắt đầu ở đâu?
Vector table nằm ở đâu?
.text / .rodata nằm ở đâu?
.data / .bss nằm ở đâu?
Stack nằm ở đâu?
Firmware được phép lớn tới đâu?
```

Bootloader và application nằm ở hai vùng Flash khác nhau nên phải dùng linker script khác nhau.

---

## 2.3. Linker Script của Bootloader

Bootloader được liên kết tại đầu Flash:

```ld
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 24K
    RAM   (xrw) : ORIGIN = 0x20000000, LENGTH = 20K
}
```

Vùng Flash hợp lệ của bootloader:

```text
0x08000000
      ↓
0x08005FFF
```

Bootloader không được vượt sang vùng application tại `0x08006000`.

---

## 2.4. Linker Script của Application

Application được liên kết tại:

```ld
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08006000, LENGTH = 38K
    RAM   (xrw) : ORIGIN = 0x20000000, LENGTH = 20K
}
```

Vùng Flash của application:

```text
0x08006000
      ↓
0x0800F7FF
```

Từ `0x0800F800` trở đi là metadata, vì vậy application tuyệt đối không được vượt vào vùng này.

Không thể build application cho `0x08000000` rồi chỉ flash binary sang `0x08006000`, vì các địa chỉ bên trong binary đã được linker tính theo địa chỉ gốc.

---

## 2.5. `ENTRY(Reset_Handler)`

Linker script thường có:

```ld
ENTRY(Reset_Handler)
```

Nó xác định `Reset_Handler` là entry symbol của ELF.

Luồng khởi động vẫn là:

```text
Vector Table
    ↓
Reset_Handler
    ↓
Khởi tạo runtime
    ↓
main()
```

Trên Cortex-M, CPU vẫn lấy `Reset_Handler` từ vector table khi reset.

---

## 2.6. Các section chính

Firmware thường được chia thành:

```text
.isr_vector
.text
.rodata
.data
.bss
```

Cách bố trí điển hình:

```text
FLASH
+----------------+
| .isr_vector    |
| .text          |
| .rodata        |
| initial .data  |
+----------------+

RAM
+----------------+
| .data          |
| .bss           |
| heap / stack   |
+----------------+
```

---

## 2.7. `.isr_vector`

Vector table phải nằm ngay đầu vùng Flash của firmware.

Application:

```ld
.isr_vector :
{
    . = ALIGN(0x200);
    __vector_table_start__ = .;
    KEEP(*(.isr_vector))
} > FLASH
```

Do đó:

```text
0x08006000 -> Initial MSP
0x08006004 -> Reset_Handler
```

Bootloader dựa vào đúng hai địa chỉ này để kiểm tra và chuyển quyền điều khiển sang application.

---

## 2.8. Tại sao dùng `KEEP()`?

Khi linker garbage collection được bật, những section không có reference thông thường có thể bị loại bỏ.

Vector table không nhất thiết được code C gọi trực tiếp nên cần:

```ld
KEEP(*(.isr_vector))
```

Ý nghĩa:

> Luôn giữ vector table trong firmware.

Nếu vector table bị loại bỏ, firmware không thể khởi động đúng.

---

## 2.9. `.text` và `.rodata`

`.text` chứa machine code.

`.rodata` chứa dữ liệu chỉ đọc, ví dụ:

```c
const uint32_t table[] = {1, 2, 3};
```

Cả hai thường nằm trong Flash:

```ld
.text :
{
    *(.text)
    *(.text*)
    *(.rodata)
    *(.rodata*)
} > FLASH
```

---

## 2.10. `.data`

`.data` chứa các biến global/static có giá trị khởi tạo và cần thay đổi lúc chạy.

Ví dụ:

```c
uint32_t counter = 100;
```

Linker script:

```ld
.data :
{
    _sdata = .;
    *(.data)
    *(.data*)
    _edata = .;
} > RAM AT > FLASH
```

Ý nghĩa:

```text
Giá trị ban đầu lưu trong Flash
            ↓
Startup code sao chép
            ↓
Biến chạy trong RAM
```

Các linker symbol thường dùng:

```text
_sidata
_sdata
_edata
```

---

## 2.11. `.bss`

`.bss` chứa các biến global/static chưa có giá trị khởi tạo rõ ràng.

Ví dụ:

```c
uint32_t error_count;
```

Linker script:

```ld
.bss (NOLOAD) :
{
    _sbss = .;
    *(.bss)
    *(.bss*)
    _ebss = .;
} > RAM
```

Startup code đưa vùng:

```text
_sbss -> _ebss
```

về `0` trước khi vào `main()`.

---

## 2.12. Linker Symbol

Các symbol như:

```text
_estack
_sidata
_sdata
_edata
_sbss
_ebss
_etext
```

được linker tạo ra để startup code biết các ranh giới bộ nhớ.

Ví dụ:

```ld
_estack = ORIGIN(RAM) + LENGTH(RAM);
```

Với RAM 20 KiB:

```text
_estack = 0x20005000
```

Đây chính là Initial MSP.

---

## 2.13. Stack và Heap

Linker script có thể dành trước dung lượng tối thiểu cho stack/heap.

Ví dụ:

```ld
_Min_Heap_Size  = 0x000;
_Min_Stack_Size = 0x400;
```

Tức là dành tối thiểu:

```text
0x400 = 1024 byte
```

cho stack.

Nếu `.data + .bss + stack` vượt quá RAM, linker nên báo lỗi ngay lúc build.

---

## 2.14. `ASSERT()` trong Linker Script

`ASSERT()` dùng để kiểm tra Memory Map ngay lúc build.

Ví dụ:

```ld
ASSERT(ORIGIN(FLASH) == 0x08006000, ...)
ASSERT(__vector_table_start__ == 0x08006000, ...)
ASSERT(_etext <= 0x0800F800, ...)
ASSERT(_ebss <= (_estack - _Min_Stack_Size), ...)
```

Nhờ đó có thể phát hiện sớm:

```text
Bootloader quá lớn
Application quá lớn
Vector table sai địa chỉ
RAM bị tràn
```

Thay vì để lỗi xuất hiện khi thiết bị đã chạy thực tế.

---

## 2.15. Vì sao Bootloader và Application cùng dùng toàn bộ RAM?

Cả bootloader và application đều có thể khai báo:

```ld
RAM : ORIGIN = 0x20000000, LENGTH = 20K
```

Điều này hợp lệ vì chúng không chạy đồng thời.

```text
Bootloader dùng SRAM
        ↓
Chuyển quyền điều khiển
        ↓
Application startup
        ↓
Application dùng lại SRAM
```

Persistent state nên lưu trong Flash/metadata, không dựa vào nội dung SRAM còn sót lại.

---

## 2.16. External W25Q có nằm trong Linker Script không?

Thông thường không.

W25Q được truy cập qua SPI, không phải vùng bộ nhớ mà CPU trực tiếp thực thi code như internal Flash.

Có thể phân biệt:

```text
Linker Script
    ↓
Internal Flash + SRAM

Storage Layout
    ↓
External W25Q
```

W25Q vẫn có Memory Map riêng, nhưng đó là bố cục lưu trữ chứ không phải linker memory region của CPU.

---

## 2.17. Quan hệ giữa Memory Map, Linker Script và Bootloader

Ba thành phần phải hoàn toàn nhất quán:

```text
Memory Map
    ↓
Linker Script
    ↓
Bootloader constants
```

Ví dụ:

```text
Application start = 0x08006000
```

thì:

- Memory Map phải quy định đúng địa chỉ này.
- Linker Script phải link application tại địa chỉ này.
- Bootloader phải đọc MSP/Reset_Handler và đặt `VTOR` từ địa chỉ này.

Sai một trong ba là hệ thống có thể không boot được.

---

## 2.18. Các lỗi thường gặp

- Application vẫn link tại `0x08000000`.
- Vector table không nằm đầu application.
- Application vượt `0x0800F800` và ghi đè metadata.
- Bootloader vượt quá 24 KiB.
- `.data/.bss` vượt quá RAM.
- Linker script và địa chỉ hard-code trong source không đồng nhất.
- Flash binary sang địa chỉ khác với địa chỉ mà linker đã dùng.

---

## 2.19. Mô hình tư duy cần nhớ

```text
                MEMORY MAP

Internal Flash
0x08000000
+------------------------+
| Bootloader             |
+------------------------+ 0x08006000
| Application            |
+------------------------+ 0x0800F800
| Metadata A             |
+------------------------+ 0x0800FC00
| Metadata B             |
+------------------------+

RAM
0x20000000
+------------------------+
| .data                  |
| .bss                   |
| heap / stack           |
+------------------------+
0x20005000
```

Application linker:

```text
FLASH ORIGIN = 0x08006000
RAM   ORIGIN = 0x20000000
        ↓
.isr_vector -> FLASH
.text       -> FLASH
.rodata     -> FLASH
.data       -> RAM, giá trị ban đầu trong FLASH
.bss        -> RAM
stack       -> đỉnh RAM
```

---

## 2.20. Trọng tâm cần thuộc

```text
Memory Map
    ↓
Linker Script
    ↓
FLASH ORIGIN / LENGTH
RAM ORIGIN / LENGTH
    ↓
.isr_vector
.text / .rodata
.data / .bss
    ↓
Linker Symbols
    ↓
ASSERT()
    ↓
Bootloader và Application dùng đúng địa chỉ
```

Các điểm cần nhớ:

- Memory Map là hợp đồng bộ nhớ.
- Bootloader bắt đầu tại `0x08000000`, tối đa 24 KiB.
- Application bắt đầu tại `0x08006000`, tối đa 38 KiB.
- Metadata bắt đầu tại `0x0800F800`.
- `.isr_vector` phải nằm ngay đầu application.
- `.text/.rodata` nằm trong Flash.
- `.data` chạy trong RAM nhưng giá trị khởi tạo nằm trong Flash.
- `.bss` nằm trong RAM và được đưa về `0` khi startup.
- `_estack`, `_sidata`, `_sdata`, `_edata`, `_sbss`, `_ebss` là các linker symbol quan trọng.
- `ASSERT()` giúp phát hiện lỗi bố cục bộ nhớ ngay lúc build.
- Memory Map, Linker Script và Bootloader phải dùng cùng một bộ địa chỉ.

---

# 3. Internal Flash Erase / Program / Verify

## 3.1. Flash nội và bản đồ bộ nhớ

Flash nội là bộ nhớ không mất dữ liệu khi mất nguồn. Trong dự án này, nó chứa bootloader, ứng dụng và hai trang metadata.

```text
0x08000000
+---------------------------+
| Bootloader - 24 KiB       |
+---------------------------+ 0x08006000
| Application - 38 KiB      |
+---------------------------+ 0x0800F800
| Metadata A - 1 KiB        |
+---------------------------+ 0x0800FC00
| Metadata B - 1 KiB        |
+---------------------------+
```

Khác RAM, Flash không thể ghi đè tùy ý. Quy trình cơ bản luôn là:

```text
Xóa -> Lập trình -> Xác minh
```

### Hiểu nhanh Internal Metadata A/B

Hai vùng `Metadata A` và `Metadata B` trong Flash nội dùng cơ chế A/B để cập nhật metadata an toàn khi có nguy cơ mất điện.

Internal Metadata lưu **trạng thái boot/update quan trọng của STM32**.

Ví dụ metadata hiện tại:

```text
A:
generation = 10
state = INSTALLING
VALID
```

Khi cần cập nhật trạng thái mới, bootloader không ghi đè trực tiếp lên A mà dùng B:

```text
Giữ nguyên A
    ↓
Erase B
    ↓
Ghi metadata mới vào B
generation = 11
state = TRIAL_BOOT
    ↓
Verify B
```

Nếu mất điện khi đang erase hoặc ghi B:

```text
A vẫn VALID
B có thể INVALID
→ bootloader dùng A
```

Nếu B đã ghi và verify thành công:

```text
A: generation = 10
B: generation = 11

→ B là bản mới nhất
```

Cần nhớ:

```text
generation
= số thứ tự để biết metadata hợp lệ nào mới hơn

state
= trạng thái boot/update hiện tại
  ví dụ INSTALLING, TRIAL_BOOT, ROLLBACK

A/B
= giữ ít nhất một bản metadata hợp lệ khi mất điện
```

---

## 3.2. Đơn vị xóa (Erase) và lập trình (Program)

Với STM32F103C8:

```text
Kích thước trang Flash = 1 KiB
Đơn vị lập trình       = 16 bit = 2 byte
```

Nghĩa là xóa theo cả trang 1024 byte nhưng lập trình từng half-word 2 byte.

Sau khi xóa, Flash trở về trạng thái:

```text
0xFF 0xFF 0xFF ...
```

Ở mức bit:

```text
Xóa:     0 -> 1
Lập trình: 1 -> 0
```

Muốn một bit đã là `0` quay lại `1`, phải xóa lại cả trang. Vì vậy firmware mới không thể đơn giản ghi đè trực tiếp lên firmware cũ.

---

## 3.3. Các thanh ghi Flash quan trọng

### `FLASH_KEYR`

Dùng để mở khóa bộ điều khiển Flash trước khi xóa/lập trình.

```text
KEY1 = 0x45670123
KEY2 = 0xCDEF89AB
```

Sau khi thao tác xong nên khóa Flash lại.

### `FLASH_CR`

Các bit cần nhớ:

```text
PG     Lập trình (Program)
PER    Xóa trang (Page Erase)
MER    Xóa toàn bộ (Mass Erase)
STRT   Bắt đầu xóa
LOCK   Khóa Flash
```

### `FLASH_AR`

Chứa địa chỉ trang cần xóa.

### `FLASH_SR`

Các status flag quan trọng:

```text
BSY       Flash đang bận
EOP       Thao tác hoàn tất
PGERR     Lỗi lập trình
WRPRTERR  Lỗi bảo vệ ghi
```

---

## 3.4. Quy trình xóa/lập trình Flash

Mô hình tư duy:

```text
Wait Flash idle
      ↓
FLASH_Unlock()
      ↓
Clear status flags
      ↓
FLASH_ErasePage(page_address)
      ↓
Lập trình từng 16 bit
      ↓
Đọc lại để xác minh
      ↓
FLASH_Lock()
```

Với SPL thường gặp:

```c
FLASH_Unlock();
FLASH_ClearFlag(...);
FLASH_ErasePage(page_address);
FLASH_ProgramHalfWord(address, value);
FLASH_Lock();
```

Địa chỉ lập trình tăng theo 2 byte:

```text
0x08006000
0x08006002
0x08006004
...
```

Nếu image có số byte lẻ, byte còn thiếu của half-word cuối thường được đệm bằng `0xFF`.

---

## 3.5. Xác minh bằng cách đọc lại

Không nên coi việc API trả về thành công là bằng chứng cuối cùng rằng dữ liệu đã đúng.

Sau program nên:

```text
Lập trình
   ↓
Đọc lại Flash
   ↓
So sánh với dữ liệu mong đợi
```

Nguyên tắc cần nhớ:

> Ghi thành công không đồng nghĩa tính toàn vẹn dữ liệu đã được chứng minh.

Repo xác minh lại từng trang sau khi ghi.

---

## 3.6. Bộ đệm theo trang trong SRAM

Trước khi xóa một trang ứng dụng, repo đọc firmware ứng viên từ W25Q vào bộ đệm SRAM 1 KiB.

```text
Firmware ứng viên trong W25Q
      ↓
Bộ đệm SRAM 1 KiB
      ↓
Xóa trang Flash nội
      ↓
Lập trình trang
      ↓
Xác minh với bộ đệm SRAM
```

Một trang Flash trở thành đơn vị cập nhật tự nhiên.

---

## 3.7. Điểm kiểm tra theo trang

Ứng dụng 38 KiB tương ứng khoảng 38 trang, mỗi trang 1 KiB.

Trình cài đặt xử lý lần lượt:

```text
Trang N
  ↓
Xóa
  ↓
Lập trình
  ↓
Xác minh
  ↓
Ghi nhận điểm kiểm tra
  ↓
Trang N+1
```

Điểm kiểm tra chỉ được cập nhật sau khi cả trang đã được xác minh thành công.

---

## 3.8. Khôi phục sau mất nguồn

Nếu mất điện giữa khi lập trình một trang, trang đó có thể bị ghi dở.

Ví dụ:

```text
Trang N đang được ghi
      ↓
Mất nguồn
```

Điểm kiểm tra vẫn chỉ trỏ tới trang cuối cùng đã được xác minh. Sau khi khởi động lại, bootloader:

```text
Xóa lại Trang N
      ↓
Lập trình lại từ đầu
      ↓
Xác minh
      ↓
Tiếp tục
```

Đây là cơ chế khôi phục theo ranh giới trang.

---

## 3.9. Tính lặp an toàn (Idempotency)

Một thao tác khôi phục tốt phải có thể chạy lại mà không làm hệ thống xấu đi.

```text
Xóa Trang N
Lập trình trang N
Xác minh Trang N
```

Nếu khởi động lại rồi chạy lại đúng chuỗi này với cùng dữ liệu, kết quả vẫn giống nhau. Đó là **khôi phục có tính lặp an toàn (idempotent recovery)**.

---

## 3.10. Sao lưu trước khi cài đặt

Không nên:

```text
Xóa ứng dụng đang hoạt động
        ↓
Lập trình firmware mới
```

vì nếu lỗi giữa chừng thì firmware cũ đã mất.

Repo làm:

```text
Ứng dụng đang hoạt động
        ↓
Sao lưu sang W25Q
        ↓
Xác minh bản sao lưu
        ↓
Cài đặt firmware mới
```

Nếu firmware mới không hoạt động, bootloader còn dữ liệu để rollback.

---

## 3.11. Metadata A/B

Metadata không được cập nhật tại chỗ trên một trang duy nhất. Repo dùng hai trang độc lập:

```text
Metadata A
Metadata B
```

Ví dụ:

```text
A generation 10, valid
        ↓
Xóa B
        ↓
Lập trình B, generation 11
        ↓
Xác minh B
        ↓
B trở thành newest valid metadata
```

Nếu mất điện khi ghi B, A vẫn còn hợp lệ.

Đây là một dạng:

```text
giao dịch sao chép-khi-ghi (copy-on-write)
```

---

## 3.12. CRC ghi cuối Metadata

CRC được đặt ở cuối metadata record.

```text
Lập trình các trường
      ↓
Lập trình CRC cuối cùng
```

Nếu reset xảy ra giữa lúc ghi, CRC chưa hoàn chỉnh và record mới sẽ bị coi là invalid. Record cũ vẫn tồn tại.

---

## 3.13. Độ bền Flash

Flash nội có số chu kỳ xóa/lập trình hữu hạn, vì vậy không nên dùng như RAM hoặc vùng lưu log ghi liên tục.

Metadata chỉ nên được cập nhật tại các lần chuyển trạng thái hoặc điểm kiểm tra quan trọng như:

```text
VERIFYING
INSTALLING
TRIAL_BOOT
CONFIRMED
ROLLBACK
```

---

## 3.14. Flash nội và W25Q ngoài

Trong dự án:

```text
Flash nội
  -> Bootloader
  -> Ứng dụng
  -> Metadata
```

W25Q ngoài:

```text
  -> Gói OTA nhận vào
  -> Image đã tái tạo
  -> Image sao lưu
  -> Vùng tạm / log
```

Đơn vị xóa khác nhau:

```text
Flash nội STM32      : trang 1 KiB
Flash W25Q ngoài      : sector 4 KiB
```

Điểm kiểm tra nên bám theo đơn vị xóa vật lý của từng loại Flash.

---

## 3.15. Luồng cài đặt firmware đầy đủ

Mental model quan trọng nhất:

```text
Firmware ứng viên đã xác minh trong W25Q
          ↓
Sao lưu ứng dụng đang hoạt động
          ↓
Xác minh bản sao lưu
          ↓
Đọc trang ứng viên 1 KiB vào SRAM
          ↓
Tắt ngắt
          ↓
FLASH Unlock
          ↓
Xóa các cờ trạng thái
          ↓
Xóa trang Flash nội
          ↓
Lập trình 16 bit/lần
          ↓
FLASH Lock
          ↓
Đọc lại để xác minh
          ↓
Ghi nhận điểm kiểm tra
          ↓
Trang tiếp theo
```

Sau khi cài toàn bộ image:

```text
Whole-image Xác minh
      ↓
TRIAL_BOOT
      ↓
CONFIRM
hoặc
ROLLBACK
```

---

## 3.16. Các lỗi thường gặp

- Không xóa trước khi lập trình.
- Nhầm kích thước trang Flash.
- Lập trình sai căn chỉnh địa chỉ.
- Không kiểm tra `BSY` hoặc các cờ lỗi.
- Không khóa Flash sau khi ghi.
- Không đọc lại để xác minh.
- Xóa ứng dụng đang hoạt động trước khi có bản sao lưu.
- Ghi nhận điểm kiểm tra trước khi trang được xác minh.
- Dùng một trang metadata duy nhất.
- Ghi metadata quá thường xuyên.
- Dùng xóa toàn bộ (mass erase) cho OTA.
- Không xử lý mất nguồn giữa lúc ghi một trang.
- Không phân biệt xóa theo trang và lập trình theo half-word.

---

## 3.17. Trọng tâm cần thuộc

```text
Trang Flash = 1 KiB
        ↓
Xóa -> 0xFF
        ↓
Lập trình 16 bit/lần
        ↓
Mở khóa / Khóa
        ↓
BSY / EOP / PGERR / WRPRTERR
        ↓
Đọc lại để xác minh
        ↓
Bộ đệm SRAM 1 KiB
        ↓
Điểm kiểm tra theo trang
        ↓
Khôi phục sau mất nguồn
        ↓
Sao lưu trước khi cài đặt
        ↓
Metadata A/B
        ↓
Khởi động thử / Rollback
```

Nếu giải thích được vì sao phải xóa, vì sao xóa theo 1 KiB nhưng lập trình chỉ 2 byte, vì sao phải xác minh và làm sao khôi phục khi mất điện giữa lúc ghi, bạn đã nắm phần cốt lõi của Flash nội STM32 trong dự án này.

---

# 4. Bootloader → Application Jump

## 4.1. Kiểm tra tính hợp lệ của ứng dụng

Trước khi chuyển sang ứng dụng, bootloader nên kiểm tra tối thiểu:

```text
MSP của ứng dụng
    ↓
Có nằm trong SRAM?
    ↓
Có alignment đúng?
    ↓
Reset_Handler có Thumb bit?
    ↓
Reset_Handler có nằm trong vùng application?
```

Các kiểm tra này chỉ xác nhận rằng vector table có cấu trúc hợp lý.

Chúng không thay thế kiểm tra bảo mật như:

```text
SHA-256
ECDSA signature
anti-rollback
```

---

## 4.2. Tại sao phải dọn trạng thái trước khi chuyển sang ứng dụng?

Ứng dụng không nhận một lần reset phần cứng thật sự.

Nó kế thừa trạng thái CPU và peripheral từ bootloader.

Bootloader có thể đã sử dụng:

- SysTick;
- UART;
- SPI;
- Timer;
- NVIC;
- PLL/cây xung nhịp;
- interrupt masks.

Do đó phải tạo một trạng thái gần giống reset trước khi chuyển quyền điều khiển.

---

## 4.3. SysTick và ngắt

Bootloader nên stop SysTick:

```c
SysTick->CTRL = 0;
SysTick->LOAD = 0;
SysTick->VAL  = 0;
```

Sau đó tạm thời mask interrupt:

```c
__disable_irq();
```

Ngoài ra phải disable và clear pending IRQ trong NVIC.

```text
Disable IRQ
+
Clear pending IRQ
```

Nếu chỉ tắt ngắt mà không xóa trạng thái chờ, một IRQ cũ của bootloader có thể chạy ngay sau khi ứng dụng bật ngắt trở lại.

PendSV và SysTick pending cũng nên được clear.

---

## 4.4. Trạng thái xung nhịp

Bootloader có thể đã thay đổi cây xung nhịp.

Ví dụ:

```text
HSI
HSE
PLL
AHB prescaler
APB prescaler
```

Trước khi chuyển sang ứng dụng có thể đưa xung nhịp về trạng thái gần reset, ví dụ:

```c
RCC_DeInit();
```

Ứng dụng sau đó tự cấu hình xung nhịp trong `SystemInit()`.

Nguyên tắc:

> Ứng dụng không nên phụ thuộc vào trạng thái xung nhịp do bootloader để lại.

---

## 4.5. Vì sao không nên đổi MSP rồi tiếp tục chạy C?

Ví dụ nguy hiểm:

```c
__set_MSP(app_msp);
app_reset_handler();
```

Sau khi MSP thay đổi, compiler vẫn có thể sinh lệnh sử dụng stack của function C hiện tại.

Điều này có thể làm hỏng stack.

Vì vậy final handoff thường dùng một assembly helper rất ngắn:

```asm
msr msp, r0
bx  r1
```

Trong dự án, hàm hỗ trợ còn đặt lại:

```text
PSP
CONTROL
BASEPRI
FAULTMASK
PRIMASK
```

để ứng dụng bắt đầu trong ngữ cảnh gần giống sau reset phần cứng.

---

## 4.6. CONTROL, PRIMASK, BASEPRI, FAULTMASK

Các register này ảnh hưởng tới privilege, stack selection và interrupt masking.

Trước khi branch sang application, bootloader nên đảm bảo application không vô tình kế thừa trạng thái bất thường.

Mục tiêu cuối cùng thường là:

```text
MSP      = application MSP
PSP      = 0
CONTROL  = 0
BASEPRI  = 0
FAULTMASK= 0
PRIMASK  = 0
```

Sau đó:

```asm
bx application_reset_handler
```

---

## 4.7. Trình tự chuyển giao đầy đủ

Luồng chuyển từ bootloader sang application:

```text
Bootloader đang chạy
        ↓
Đọc MSP của ứng dụng
Đọc Reset_Handler của ứng dụng
        ↓
Validate MSP / Reset_Handler
        ↓
Dừng SysTick
        ↓
Che ngắt
        ↓
Tắt + xóa IRQ đang chờ trong NVIC
        ↓
Xóa trạng thái chờ của SysTick / PendSV
        ↓
Đưa xung nhịp về trạng thái phù hợp
        ↓
VTOR = bảng vector của ứng dụng
        ↓
DSB / ISB
        ↓
MSP = MSP của ứng dụng
        ↓
Đặt lại PSP / CONTROL / các mặt nạ ngắt
        ↓
BX Reset_Handler của ứng dụng
        ↓
quá trình khởi động ứng dụng
        ↓
.data initialization
.bss initialization
SystemInit()
        ↓
main()
```

---

## 4.8. Quyết định khởi động

Bootloader không phải lúc nào cũng chuyển sang ứng dụng ngay sau reset.

Trong hệ thống OTA bảo mật, nó phải đọc trạng thái lưu bền vững trước.

Ví dụ:

```text
IDLE
RECEIVING
VERIFYING
PATCHING
BACKING_UP
INSTALLING
TRIAL_BOOT
ROLLBACK
```

Nếu state là:

```text
INSTALLING
```

bootloader không nên chạy một application đang được ghi dở.

Nếu state là:

```text
IDLE
```

và application hợp lệ, bootloader có thể thực hiện handoff.

Vì vậy bootloader thực chất là:

> Bộ xử lý quyết định khởi động + thành phần có quyền xử lý khôi phục/cập nhật firmware.

---

## 4.9. Các lỗi Bootloader thường gặp

- Ứng dụng được liên kết (link) ở sai địa chỉ.
- Vector table không nằm tại application base.
- Quên đổi `SCB->VTOR`.
- Chuyển thẳng tới `main()`.
- Không kiểm tra MSP.
- Không kiểm tra Reset_Handler range.
- Không kiểm tra Thumb bit.
- Đổi MSP trong C rồi tiếp tục sử dụng stack.
- Không stop SysTick.
- Không clear pending NVIC interrupt.
- Để `PRIMASK` hoặc `BASEPRI` ở trạng thái sai.
- Ứng dụng vượt quá phân vùng Flash.
- Chuyển sang ứng dụng dù trạng thái OTA đang ở giữa quá trình cài đặt.

---

---

---

# 5. W25Q SPI NOR: Erase / Program / Read / Status

W25Q là Flash NOR ngoài giao tiếp SPI. Trong dự án này, nó đóng vai trò vùng lưu trữ bền vững cho OTA: chứa gói firmware nhận vào, image đã tái tạo, bản sao lưu application và một số metadata/checkpoint. Firmware cuối cùng vẫn được chạy từ Flash nội của STM32.

## 5.1. Giao tiếp SPI

Các tín hiệu chính:

```text
SCK   -> xung nhịp
MOSI  -> STM32 gửi dữ liệu
MISO  -> STM32 nhận dữ liệu
CS    -> chọn chip
```

Một giao dịch điển hình:

```text
CS = LOW
   ↓
Command
   ↓
Address
   ↓
Data
   ↓
CS = HIGH
```

`CS` xác định ranh giới của một lệnh gửi tới W25Q.

---

## 5.2. JEDEC ID

Trước khi sử dụng Flash, driver nên đọc `JEDEC ID` bằng lệnh:

```text
0x9F
```

Kết quả cho biết:

```text
Manufacturer
Memory Type
Capacity
```

Mục đích là xác nhận đúng chip Flash và dung lượng được hỗ trợ trước khi thực hiện OTA.

---

## 5.3. Các lệnh quan trọng

Nhóm lệnh cốt lõi:

```text
0x9F   Read JEDEC ID
0x03   Read Data
0x06   Write Enable
0x05   Read Status Register-1
0x02   Page Program
0x20   Sector Erase 4 KiB
```

Mental model:

```text
READ
  -> 0x03

PROGRAM
  -> Write Enable
  -> Page Program

ERASE
  -> Write Enable
  -> Sector Erase

STATUS
  -> Read Status Register
```

---

## 5.4. Quy tắc của NOR Flash

Sau khi erase:

```text
FF FF FF FF ...
```

Ở mức bit:

```text
Erase   : đưa bit về 1
Program : chỉ chuyển 1 -> 0
```

Nếu dữ liệu mới cần một bit chuyển:

```text
0 -> 1
```

thì sector phải được erase trước.

Vì vậy không thể tùy ý ghi đè dữ liệu mới lên dữ liệu cũ.

---

## 5.5. Page và Sector

Hai kích thước cần nhớ:

```text
Page   = 256 byte
Sector = 4 KiB
```

`Page` là giới hạn quan trọng của `Page Program`.

`Sector` là đơn vị erase.

```text
1 Sector = 16 Page
          = 4096 byte
```

---

## 5.6. Write Enable và WEL

Trước `Page Program` hoặc `Sector Erase`, phải gửi:

```text
Write Enable = 0x06
```

Lệnh này đặt bit:

```text
WEL = Write Enable Latch
```

Driver nên kiểm tra `WEL = 1` trước khi tiếp tục ghi/xóa.

Flow:

```text
Write Enable
     ↓
Đọc Status Register
     ↓
WEL = 1?
   /     \
 no      yes
 ↓        ↓
lỗi   Program/Erase
```

---

## 5.7. Status Register và BUSY

Lệnh:

```text
0x05
```

đọc `Status Register-1`.

Hai bit quan trọng:

```text
BUSY
WEL
```

`BUSY = 1` nghĩa là Flash đang program/erase.

`BUSY = 0` nghĩa là Flash đã sẵn sàng.

Sau mỗi operation ghi/xóa, driver phải poll `BUSY` cho tới khi clear và nên có timeout để tránh treo vô hạn.

---

## 5.8. Read Data

Đọc dữ liệu dùng:

```text
0x03
```

Flow:

```text
CS LOW
  ↓
0x03
  ↓
24-bit Address
  ↓
Nhận Data
  ↓
CS HIGH
```

Read không cần `Write Enable` và không làm thay đổi dữ liệu Flash.

---

## 5.9. Page Program

Quy trình ghi:

```text
Wait Ready
    ↓
Write Enable
    ↓
Kiểm tra WEL
    ↓
Page Program 0x02
    ↓
24-bit Address
    ↓
Data
    ↓
CS HIGH
    ↓
Poll BUSY
    ↓
Đọc lại để xác minh
```

Sau khi gửi lệnh program, không được thực hiện operation tiếp theo cho đến khi `BUSY = 0`.

---

## 5.10. Page Boundary

`Page Program` không nên vượt qua ranh giới page 256 byte.

Ví dụ nếu bắt đầu tại offset:

```text
0xF0
```

thì page hiện tại chỉ còn:

```text
0x100 - 0xF0 = 16 byte
```

Driver phải chia dữ liệu:

```text
16 byte
  ↓
Program page hiện tại

phần còn lại
  ↓
Program page tiếp theo
```

Đây là lý do một hàm ghi lớn phải tự chia thành nhiều chunk theo page boundary.

---

## 5.11. Sector Erase

Erase dùng:

```text
0x20
```

với sector 4 KiB.

Flow:

```text
Kiểm tra địa chỉ sector
      ↓
Write Enable
      ↓
Sector Erase 0x20
      ↓
24-bit Address
      ↓
Poll BUSY
      ↓
Ready
```

Địa chỉ erase nên căn theo boundary 4 KiB:

```text
0x000000
0x001000
0x002000
0x003000
...
```

Không nên tự động làm tròn một địa chỉ sai vì có thể xóa nhầm sector.

---

## 5.12. Đọc lại để xác minh

Sau khi program:

```text
Program
   ↓
Poll BUSY
   ↓
Read lại
   ↓
So sánh dữ liệu
```

Nếu dữ liệu đọc lại khác dữ liệu mong đợi, operation phải được coi là thất bại.

Nguyên tắc giống Flash nội:

> Gửi lệnh thành công không có nghĩa dữ liệu đã được chứng minh là đúng.

---

## 5.13. Bản đồ phân vùng W25Q và vai trò từng vùng

W25Q không được dùng như một vùng lưu trữ chung duy nhất. Dự án chia Flash ngoài thành các phân vùng cố định để mỗi loại dữ liệu OTA có vị trí riêng.

```text
0x000000  External Metadata A   4 KiB
0x001000  External Metadata B   4 KiB

0x002000  Incoming Artifact   128 KiB
0x022000  Reconstructed Image 128 KiB
0x042000  Backup Image        128 KiB

0x062000  Update Logs          64 KiB

0x072000  Reserved
   ...
0x3FF000  Flash Self-Test       4 KiB
0x400000  End
```

Vai trò chính:

- **External Metadata A/B**: lưu metadata/checkpoint ngoài theo cơ chế A/B để vẫn có một bản hợp lệ nếu mất nguồn khi đang cập nhật bản còn lại.
- **Incoming Artifact**: lưu nguyên gói OTA vừa nhận, có thể là full firmware hoặc delta package.
- **Reconstructed Image**: lưu firmware đích hoàn chỉnh sau khi copy full image hoặc áp dụng delta patch.
- **Backup Image**: lưu bản sao application đang chạy trước khi cài firmware mới, dùng cho rollback nếu trial boot thất bại.
- **Update Logs**: lưu thông tin chẩn đoán liên quan quá trình cập nhật.
- **Reserved**: dành cho mở rộng sau này.
- **Flash Self-Test**: sector riêng để kiểm tra driver W25Q mà không phá dữ liệu OTA.

### Hiểu nhanh External Metadata A/B

`External Metadata A/B` dùng **cùng cơ chế A/B như Internal Metadata A/B** đã học ở phần Flash nội: giữ một bản metadata hợp lệ trong khi cập nhật bản còn lại, để nếu mất điện giữa lúc erase/ghi thì hệ thống vẫn còn bản cũ để phục hồi.

Điểm khác là **nội dung mà metadata quản lý**:

```text
Internal Metadata A/B
= trạng thái boot/update của STM32

External Metadata A/B
= trạng thái/checkpoint của quá trình OTA trên W25Q
```

`Incoming Artifact` chứa **dữ liệu firmware thật**. `External Metadata A/B` chỉ là hai bản **ghi chú trạng thái** để hệ thống biết quá trình OTA đã tiến tới đâu và có thể tiếp tục từ đâu sau khi reset/mất điện.

Ví dụ:

```text
Incoming Artifact:
đã ghi dữ liệu tới 36 KiB

External Metadata hiện tại:
checkpoint = 32 KiB
```

Điều này có nghĩa dữ liệu vật lý có thể đã tới 36 KiB, nhưng hệ thống mới chỉ **tin chắc** mốc 32 KiB. Nếu mất điện trước khi checkpoint mới được ghi hợp lệ, sau reboot hệ thống sẽ tiếp tục từ mốc 32 KiB.

Cơ chế cập nhật giống Internal Metadata A/B:

```text
A: generation = 10, checkpoint = 32 KiB, VALID
        ↓
Giữ nguyên A
        ↓
Erase + ghi B
        ↓
B: generation = 11, checkpoint = 36 KiB
        ↓
Verify B
```

Nếu mất điện khi đang erase/ghi B thì A vẫn còn hợp lệ. Chỉ khi B ghi xong và verify thành công, B mới trở thành bản mới nhất.

Cần nhớ:

```text
Incoming Artifact      = dữ liệu firmware thật
External Metadata A/B  = bookmark trạng thái OTA trên W25Q
generation             = số thứ tự để biết metadata nào mới hơn
checkpoint             = mốc an toàn cuối cùng có thể tiếp tục
```

Mental model:

```text
Incoming Artifact
      ↓
Verify / Reconstruct
      ↓
Reconstructed Image
      ↓
Backup Active Application
      ↓
Backup Image
      ↓
Install vào Flash nội
```

Phân vùng giúp bootloader biết rõ dữ liệu nào đang là **gói nhận vào**, **firmware đích**, **bản sao lưu** và **metadata phục hồi**, đồng thời giảm nguy cơ các module OTA ghi đè nhầm lên nhau.

---

## 5.14. Vai trò của W25Q trong OTA

Có thể xem W25Q như vùng làm việc bền vững giữa quá trình nhận firmware và quá trình cài vào Flash nội.

```text
ESP32 / UART
      ↓
Incoming Artifact
      ↓
W25Q
      ↓
Verify
      ↓
Reconstructed Image
      ↓
Backup Application
      ↓
Install vào Flash nội STM32
```

W25Q cho phép giữ cả candidate firmware và backup mà không phải phá ngay active application.

---

## 5.15. Flash nội và W25Q khác nhau

Trong dự án:

```text
STM32 Internal Flash
Erase   = 1 KiB page
Program = 16-bit half-word

W25Q SPI NOR
Erase   = 4 KiB sector
Program = theo page tối đa 256 byte
```

Vai trò:

```text
Internal Flash
  -> Bootloader
  -> Application
  -> Metadata quan trọng

W25Q
  -> OTA staging
  -> Reconstructed image
  -> Backup
  -> Checkpoint / storage
```

---

## 5.16. Các lỗi thường gặp

- Quên `Write Enable` trước Program/Erase.
- Không kiểm tra `WEL`.
- Không chờ `BUSY` clear.
- Không có timeout khi poll `BUSY`.
- Ghi xuyên page boundary mà không chia chunk.
- Erase địa chỉ không căn theo sector 4 KiB.
- Cố program bit `0 -> 1` mà chưa erase.
- Không đọc lại để xác minh.
- Không kiểm tra `JEDEC ID`.
- Cho module OTA ghi tùy ý bằng địa chỉ tuyệt đối mà không kiểm tra phạm vi.

---

## 5.17. Mô hình tư duy cần nhớ

```text
READ
CS LOW
  ↓
0x03
  ↓
Address
  ↓
Data
  ↓
CS HIGH


PROGRAM
Write Enable 0x06
  ↓
WEL = 1
  ↓
Page Program 0x02
  ↓
Address + Data
  ↓
BUSY = 1
  ↓
Wait BUSY = 0
  ↓
Readback Verify


ERASE
Write Enable 0x06
  ↓
Sector Erase 0x20
  ↓
4 KiB-aligned Address
  ↓
BUSY = 1
  ↓
Wait BUSY = 0
```

---

## 5.18. Trọng tâm cần thuộc

```text
SPI
↓
CS / SCK / MOSI / MISO
↓
JEDEC ID 0x9F
↓
Read 0x03
↓
Write Enable 0x06
↓
WEL
↓
Page Program 0x02
↓
Page = 256 byte
↓
Sector Erase 0x20
↓
Sector = 4 KiB
↓
Status 0x05
↓
BUSY
↓
Đọc lại để xác minh
```

Nếu giải thích được ba luồng `Read`, `Page Program`, `Sector Erase`, đồng thời hiểu vì sao cần `WEL`, `BUSY`, page boundary và erase trước khi chuyển bit `0 -> 1`, bạn đã nắm phần cốt lõi của W25Q SPI NOR trong dự án này.

---

---

# 6. CRC32 + UART Framing + COBS

Phần này mô tả cách ESP32 gửi OTA artifact sang STM32 qua UART sao cho STM32 biết **đâu là một frame hoàn chỉnh, frame có bị lỗi hay không và dữ liệu nào được phép ghi xuống W25Q**.

Luồng tổng quát:

```text
ESP32
  ↓
Tạo packet
  ↓
CRC32
  ↓
COBS encode
  ↓
UART
  ↓
STM32
  ↓
COBS decode
  ↓
Kiểm tra CRC32
  ↓
Kiểm tra sequence / offset
  ↓
SPI → W25Q
  ↓
ACK / NACK
```

---

## 6.1. Tại sao UART cần Framing?

UART chỉ truyền một dòng byte liên tục và không tự biết packet bắt đầu/kết thúc ở đâu.

```text
12 A4 55 21 00 73 ...
```

Vì vậy protocol phải tự định nghĩa:

```text
ranh giới frame
cấu trúc packet
kiểm tra lỗi
thứ tự packet
```

Trong dự án, COBS + byte `0x00` được dùng để xác định ranh giới frame.

---

## 6.2. COBS và byte phân cách `0x00`

COBS là:

```text
Consistent Overhead Byte Stuffing
```

Sau khi COBS encode, bên trong encoded frame không còn byte:

```text
0x00
```

Do đó project có thể dùng:

```text
[COBS encoded frame] 0x00
```

với `0x00` là byte phân cách cuối frame.

STM32 nhận UART:

```text
frame 1 ... 00
frame 2 ... 00
frame 3 ... 00
```

Khi gặp `0x00`, STM32 biết đã nhận xong một frame và có thể COBS decode.

---

## 6.3. Vì sao không dùng trực tiếp `0x00` nếu không có COBS?

Firmware binary hoàn toàn có thể chứa:

```text
0x00
```

Nếu chỉ dùng `0x00` làm byte kết thúc:

```text
12 35 00 A7 82
```

receiver không biết `00` là:

```text
dữ liệu firmware
```

hay:

```text
kết thúc frame
```

COBS loại vấn đề này bằng cách đảm bảo `0x00` chỉ xuất hiện ở cuối encoded frame.

---

## 6.4. Luồng nhận COBS trên STM32

STM32 tích lũy các byte UART vào buffer:

```text
UART byte
   ↓
byte == 0x00 ?
   |
   +-- NO  → lưu vào encoded buffer
   |
   +-- YES
        ↓
    frame hoàn chỉnh
        ↓
    COBS decode
        ↓
    raw packet
```

Nếu một frame lỗi, receiver có thể bỏ frame đó và đồng bộ lại ở byte `0x00` tiếp theo.

---

## 6.5. Cấu trúc Raw Packet

Sau COBS decode, packet có dạng:

```text
Offset   Size   Field

0        2      magic
2        1      protocol_version
3        1      command
4        4      update_id
8        4      offset
12       2      sequence
14       2      payload_length
16       N      payload
16+N     4      packet_crc32
```

Các giá trị quan trọng:

```text
magic            = 0xA55A
protocol_version = 1
header           = 16 byte
DATA payload max = 256 byte
CRC32            = 4 byte
```

---

## 6.6. Ý nghĩa các field quan trọng

`magic` giúp xác nhận đây là packet của OTA protocol.

`protocol_version` đảm bảo ESP32 và STM32 đang dùng cùng phiên bản giao thức.

`command` xác định loại packet, ví dụ:

```text
START
DATA
FINISH
RESUME
INSTALL
ACK
NACK
```

`update_id` xác định OTA session hiện tại.

`offset` cho biết payload thuộc vị trí nào trong artifact.

`sequence` là số thứ tự packet.

`payload_length` cho biết số byte dữ liệu trong payload.

---

## 6.7. CRC32 dùng để làm gì?

CRC32 dùng để phát hiện dữ liệu bị hỏng trong quá trình truyền hoặc lưu trữ.

Sender:

```text
Header + Payload
      ↓
tính CRC32
```

Receiver:

```text
Header + Payload
      ↓
tính CRC32 lại
      ↓
so sánh với CRC nhận được
```

Nếu khác nhau:

```text
packet bị lỗi
→ không ghi xuống W25Q
→ trả NACK
```

CRC32 packet được tính trên header + payload, không bao gồm chính trường CRC32 và không tính trên dữ liệu sau COBS encode.

---

## 6.8. Thứ tự đóng gói và giải mã

ESP32:

```text
Header
  +
Payload
  ↓
CRC32
  ↓
Raw Packet
  ↓
COBS encode
  ↓
thêm 0x00
  ↓
UART Send
```

STM32 làm ngược lại:

```text
UART Receive
  ↓
tìm 0x00
  ↓
COBS decode
  ↓
Raw Packet
  ↓
kiểm tra header
  ↓
CRC32
  ↓
xử lý command
```

Thứ tự này cần được giữ đúng.

---

## 6.9. DATA Packet và `offset`

OTA artifact được chia thành nhiều DATA packet.

Ví dụ payload 256 byte:

```text
Packet 0:
offset = 0
payload = 256 B

Packet 1:
offset = 256
payload = 256 B

Packet 2:
offset = 512
payload = 256 B
```

STM32 kiểm tra `offset` và `sequence` để bảo đảm dữ liệu đến đúng thứ tự.

```text
offset nhận được
      =
next_expected_offset
```

Packet sai thứ tự không được ghi tùy ý xuống W25Q.

---

## 6.10. CRC phải đúng trước khi ghi W25Q

Luồng đúng:

```text
UART DATA
   ↓
COBS decode
   ↓
kiểm tra header
   ↓
CRC32
   ↓
sequence / offset
   ↓
hợp lệ?
 /      \
NO      YES
↓        ↓
NACK    SPI
         ↓
       W25Q
```

Nhờ vậy dữ liệu UART bị lỗi không làm hỏng `Incoming Artifact`.

---

## 6.11. ACK / NACK

`ACK` nghĩa là packet đã được chấp nhận.

`NACK` nghĩa là packet bị từ chối do lỗi như:

```text
CRC sai
offset sai
sequence sai
length sai
protocol sai
```

Với DATA packet, ACK nên được gửi sau khi dữ liệu đã được ghi và xác minh ở W25Q.

```text
CRC OK
  ↓
Write W25Q
  ↓
Verify
  ↓
ACK
```

---

## 6.12. Retry và packet trùng lặp

Nếu ESP32 không nhận được ACK, nó có thể gửi lại cùng packet.

Ví dụ:

```text
DATA N
  ↓
STM32 ghi thành công
  ↓
ACK bị mất
  ↓
ESP32 timeout
  ↓
gửi lại DATA N
```

STM32 cần nhận biết packet trùng lặp và nếu dữ liệu giống packet đã chấp nhận thì có thể gửi lại ACK.

Điều này giúp retry an toàn khi ACK bị mất.

---

## 6.13. Packet CRC và Artifact CRC

Có hai mức CRC khác nhau.

### Packet CRC32

Kiểm tra một packet UART:

```text
Header + Payload
```

Nó trả lời:

> Packet này có bị lỗi trong quá trình truyền không?

### Artifact CRC32

Sau khi nhận toàn bộ OTA artifact, STM32 kiểm tra CRC của toàn bộ dữ liệu trong `Incoming Artifact`.

Nó trả lời:

> Toàn bộ file OTA đã nhận có đúng không?

Mental model:

```text
Packet CRC
= bảo vệ từng packet

Artifact CRC
= kiểm tra toàn bộ file
```

---

## 6.14. CRC32 không phải cơ chế bảo mật

CRC32 chỉ phát hiện lỗi dữ liệu ngẫu nhiên như:

```text
bit flip
nhiễu truyền
corruption
```

CRC32 không xác thực người phát hành firmware vì một attacker có thể sửa firmware rồi tính CRC mới.

```text
CRC32
= kiểm tra tính toàn vẹn do lỗi

SHA-256 + ECDSA
= xác thực firmware
```

---

## 6.15. Vai trò của từng thành phần

```text
COBS
= xác định ranh giới frame

CRC32
= phát hiện dữ liệu bị lỗi

sequence / offset
= bảo đảm packet đúng thứ tự/vị trí

ACK / NACK
= phản hồi packet đã thành công hay thất bại

W25Q
= lưu OTA artifact sau khi packet đã hợp lệ
```

COBS và CRC32 không mã hóa dữ liệu và không thay thế ECDSA.

---

## 6.16. Luồng OTA UART hoàn chỉnh

```text
Server
  ↓
ESP32
  ↓
chia artifact thành DATA packet
  ↓
Header + Payload
  ↓
CRC32
  ↓
COBS encode
  ↓
0x00
  ↓
UART
  ↓
STM32
  ↓
COBS decode
  ↓
CRC32 verify
  ↓
sequence / offset verify
  ↓
SPI
  ↓
W25Q Incoming Artifact
  ↓
Verify write
  ↓
ACK
```

---

## 6.17. Ví dụ đầy đủ: gửi một DATA packet từ ESP32 xuống W25Q

Giả sử server có OTA artifact:

```text
firmware.bin = 1024 byte
```

ESP32 chia thành 4 DATA packet, mỗi packet 256 byte:

```text
Packet 0 → offset =   0
Packet 1 → offset = 256
Packet 2 → offset = 512   ← xét packet này
Packet 3 → offset = 768
```

Với Packet 2, ESP32 lấy:

```text
payload = firmware[512 ... 767]
payload_length = 256
sequence = 2
offset = 512
```

Sau đó tạo header:

```text
magic            = 0xA55A
protocol_version = 1
command          = DATA
update_id        = OTA session hiện tại
offset           = 512
sequence         = 2
payload_length   = 256
```

ESP32 ghép:

```text
Header 16 B
+
Payload 256 B
```

rồi tính `CRC32` trên `Header + Payload` và gắn thêm 4 byte CRC:

```text
[ Header ][ Payload ][ CRC32 ]
```

Vì raw packet có thể chứa `0x00`, ESP32 chạy:

```text
COBS encode
```

để encoded frame không chứa `0x00`, rồi thêm một byte `0x00` ở cuối:

```text
[ COBS encoded frame ][ 0x00 ]
```

Frame được gửi qua UART sang STM32.

STM32 nhận từng byte cho tới khi gặp `0x00`, sau đó:

```text
COBS decode
    ↓
khôi phục Raw Packet
    ↓
kiểm tra magic / version / length
    ↓
tính lại CRC32
```

Nếu CRC sai:

```text
NACK
→ không ghi xuống W25Q
```

Nếu CRC đúng, STM32 tiếp tục kiểm tra:

```text
sequence = 2
offset   = 512
```

Giả sử đây đúng là packet tiếp theo cần nhận, STM32 tính địa chỉ ghi trong phân vùng `Incoming Artifact`:

```text
Incoming Artifact base = 0x002000
offset                  = 0x000200

địa chỉ ghi = 0x002000 + 0x000200
            = 0x002200
```

Sau đó:

```text
Payload 256 B
    ↓
STM32
    ↓
SPI
    ↓
W25Q address 0x002200
    ↓
Page Program
    ↓
Readback Verify
```

Nếu dữ liệu đọc lại đúng với payload vừa nhận:

```text
Verify PASS
    ↓
ACK
```

Lúc này STM32 có thể báo:

```text
next_expected_offset = 768
```

để ESP32 gửi Packet 3.

Toàn bộ ví dụ có thể nhớ theo một chuỗi:

```text
Server
  ↓
ESP32 lấy 256 B tại offset 512
  ↓
Header + Payload
  ↓
CRC32
  ↓
COBS Encode
  ↓
0x00
  ↓
UART
  ↓
STM32
  ↓
COBS Decode
  ↓
CRC32 Verify
  ↓
sequence / offset Verify
  ↓
SPI
  ↓
W25Q Incoming Artifact @ 0x002200
  ↓
Readback Verify
  ↓
ACK
  ↓
next_expected_offset = 768
```

Cách hiểu vai trò từng bước:

```text
COBS
= xác định đúng ranh giới frame

CRC32
= phát hiện packet bị lỗi

sequence / offset
= xác nhận đúng packet và đúng vị trí

W25Q Readback Verify
= xác nhận dữ liệu đã thực sự được ghi đúng

ACK
= cho phép ESP32 gửi packet tiếp theo
```

---

## 6.18. Trọng tâm cần thuộc

```text
Raw Packet
↓
Header + Payload
↓
CRC32
↓
COBS Encode
↓
0x00 delimiter
↓
UART
↓
COBS Decode
↓
CRC32 Verify
↓
Sequence / Offset
↓
Write W25Q
↓
ACK / NACK
```

Các giá trị chính:

```text
UART              = 115200, 8-N-1
magic             = 0xA55A
protocol_version  = 1
header            = 16 byte
DATA payload max  = 256 byte
CRC32             = 4 byte
COBS delimiter    = 0x00
```

Cần hiểu rõ ba ý:

```text
COBS
= giúp tìm đúng ranh giới frame

CRC32
= phát hiện packet bị lỗi

ACK chỉ gửi sau khi
packet hợp lệ và dữ liệu đã được lưu/xác minh
```

Nếu giải thích được ba ý này cùng với `sequence`, `offset` và luồng ghi vào W25Q, bạn đã nắm phần cốt lõi của CRC32 + UART Framing + COBS trong dự án.

---

# 7. OTA State Machine + Persistent Metadata A/B

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 8. Power-Loss Recovery + Checkpoint + Rollback

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 9. SHA-256 + ECDSA P-256 + Public / Private Key

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 10. Firmware Container + Anti-Rollback

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 11. Delta Patch / JojoDiff Concept

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 12. ESP32 + MQTT + HTTPS + Server / Release Tooling

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.
