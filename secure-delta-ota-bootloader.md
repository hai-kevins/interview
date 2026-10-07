# Mục lục

- [1. STM32 Bootloader – từ Reset đến ứng dụng](#1-stm32-bootloader-từ-reset-đến-ứng-dụng)
  - [1.1. Bootloader là gì?](#11-bootloader-là-gì)
  - [1.2. Cortex-M3 khởi động như thế nào?](#12-cortex-m3-khởi-động-như-thế-nào)
  - [1.3. Bảng vector (Vector Table)](#13-bảng-vector-vector-table)
  - [1.4. MSP – Con trỏ ngăn xếp chính (Main Stack Pointer)](#14-msp-con-trỏ-ngăn-xếp-chính-main-stack-pointer)
  - [1.5. Reset_Handler](#15-resethandler)
  - [1.6. `.data` và `.bss`](#16-data-và-bss)
  - [1.7. Linker Script](#17-linker-script)
  - [1.8. VTOR – Thanh ghi độ lệch bảng vector (Vector Table Offset Register)](#18-vtor-thanh-ghi-độ-lệch-bảng-vector-vector-table-offset-register)
  - [1.9. Bit Thumb](#19-bit-thumb)
  - [1.10. Kiểm tra tính hợp lệ của ứng dụng](#110-kiểm-tra-tính-hợp-lệ-của-ứng-dụng)
  - [1.11. Tại sao phải dọn trạng thái trước khi chuyển sang ứng dụng?](#111-tại-sao-phải-dọn-trạng-thái-trước-khi-chuyển-sang-ứng-dụng)
  - [1.12. SysTick và ngắt](#112-systick-và-ngắt)
  - [1.13. Trạng thái xung nhịp](#113-trạng-thái-xung-nhịp)
  - [1.14. Vì sao không nên đổi MSP rồi tiếp tục chạy C?](#114-vì-sao-không-nên-đổi-msp-rồi-tiếp-tục-chạy-c)
  - [1.15. CONTROL, PRIMASK, BASEPRI, FAULTMASK](#115-control-primask-basepri-faultmask)
  - [1.16. Trình tự chuyển giao đầy đủ](#116-trình-tự-chuyển-giao-đầy-đủ)
  - [1.17. Quyết định khởi động](#117-quyết-định-khởi-động)
  - [1.18. Các lỗi Bootloader thường gặp](#118-các-lỗi-bootloader-thường-gặp)
  - [1.19. Mô hình tư duy cần nhớ](#119-mô-hình-tư-duy-cần-nhớ)
  - [1.20. Trọng tâm cần thuộc](#120-trọng-tâm-cần-thuộc)
- [2. Flash nội STM32](#2-flash-nội-stm32)
  - [2.1. Flash nội và bản đồ bộ nhớ](#21-flash-nội-và-bản-đồ-bộ-nhớ)
  - [2.2. Đơn vị xóa (Erase) và lập trình (Program)](#22-đơn-vị-xóa-erase-và-lập-trình-program)
  - [2.3. Các thanh ghi Flash quan trọng](#23-các-thanh-ghi-flash-quan-trọng)
  - [2.4. Quy trình xóa/lập trình Flash](#24-quy-trình-xóalập-trình-flash)
  - [2.5. Xác minh bằng cách đọc lại](#25-xác-minh-bằng-cách-đọc-lại)
  - [2.6. Bộ đệm theo trang trong SRAM](#26-bộ-đệm-theo-trang-trong-sram)
  - [2.7. Điểm kiểm tra theo trang](#27-điểm-kiểm-tra-theo-trang)
  - [2.8. Khôi phục sau mất nguồn](#28-khôi-phục-sau-mất-nguồn)
  - [2.9. Tính lặp an toàn (Idempotency)](#29-tính-lặp-an-toàn-idempotency)
  - [2.10. Sao lưu trước khi cài đặt](#210-sao-lưu-trước-khi-cài-đặt)
  - [2.11. Metadata A/B](#211-metadata-ab)
  - [2.12. CRC ghi cuối Metadata](#212-crc-ghi-cuối-metadata)
  - [2.13. Độ bền Flash](#213-độ-bền-flash)
  - [2.14. Flash nội và W25Q ngoài](#214-flash-nội-và-w25q-ngoài)
  - [2.15. Luồng cài đặt firmware đầy đủ](#215-luồng-cài-đặt-firmware-đầy-đủ)
  - [2.16. Các lỗi thường gặp](#216-các-lỗi-thường-gặp)
  - [2.17. Trọng tâm cần thuộc](#217-trọng-tâm-cần-thuộc)
- [3. Linker Script + Memory Map](#3-linker-script-memory-map)
  - [3.1. Memory Map là gì?](#31-memory-map-là-gì)
  - [3.2. Linker Script là gì?](#32-linker-script-là-gì)
  - [3.3. Linker Script của Bootloader](#33-linker-script-của-bootloader)
  - [3.4. Linker Script của Application](#34-linker-script-của-application)
  - [3.5. `ENTRY(Reset_Handler)`](#35-entryresethandler)
  - [3.6. Các section chính](#36-các-section-chính)
  - [3.7. `.isr_vector`](#37-isrvector)
  - [3.8. Tại sao dùng `KEEP()`?](#38-tại-sao-dùng-keep)
  - [3.9. `.text` và `.rodata`](#39-text-và-rodata)
  - [3.10. `.data`](#310-data)
  - [3.11. `.bss`](#311-bss)
  - [3.12. Linker Symbol](#312-linker-symbol)
  - [3.13. Stack và Heap](#313-stack-và-heap)
  - [3.14. `ASSERT()` trong Linker Script](#314-assert-trong-linker-script)
  - [3.15. Vì sao Bootloader và Application cùng dùng toàn bộ RAM?](#315-vì-sao-bootloader-và-application-cùng-dùng-toàn-bộ-ram)
  - [3.16. External W25Q có nằm trong Linker Script không?](#316-external-w25q-có-nằm-trong-linker-script-không)
  - [3.17. Quan hệ giữa Memory Map, Linker Script và Bootloader](#317-quan-hệ-giữa-memory-map-linker-script-và-bootloader)
  - [3.18. Các lỗi thường gặp](#318-các-lỗi-thường-gặp)
  - [3.19. Mô hình tư duy cần nhớ](#319-mô-hình-tư-duy-cần-nhớ)
  - [3.20. Trọng tâm cần thuộc](#320-trọng-tâm-cần-thuộc)

---

# 1. STM32 Bootloader – từ Reset đến ứng dụng

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

## 1.7. Linker Script

Linker script quyết định firmware được đặt ở đâu trong bộ nhớ.

Bootloader:

```text
FLASH ORIGIN = 0x08000000
```

Application:

```text
FLASH ORIGIN = 0x08006000
```

Vì vậy application phải được build với linker script riêng.

Nếu build application như firmware thông thường tại `0x08000000` nhưng lại flash nó vào `0x08006000`, các địa chỉ bên trong binary sẽ không đúng.

Memory map chính:

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

---

## 1.8. VTOR – Thanh ghi độ lệch bảng vector (Vector Table Offset Register)

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

## 1.9. Bit Thumb

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

## 1.10. Kiểm tra tính hợp lệ của ứng dụng

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

## 1.11. Tại sao phải dọn trạng thái trước khi chuyển sang ứng dụng?

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

## 1.12. SysTick và ngắt

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

## 1.13. Trạng thái xung nhịp

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

## 1.14. Vì sao không nên đổi MSP rồi tiếp tục chạy C?

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

## 1.15. CONTROL, PRIMASK, BASEPRI, FAULTMASK

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

## 1.16. Trình tự chuyển giao đầy đủ

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

## 1.17. Quyết định khởi động

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

## 1.18. Các lỗi Bootloader thường gặp

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

## 1.19. Mô hình tư duy cần nhớ

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

## 1.20. Trọng tâm cần thuộc

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

# 2. Flash nội STM32

## 2.1. Flash nội và bản đồ bộ nhớ

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

---

## 2.2. Đơn vị xóa (Erase) và lập trình (Program)

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

## 2.3. Các thanh ghi Flash quan trọng

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

## 2.4. Quy trình xóa/lập trình Flash

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

## 2.5. Xác minh bằng cách đọc lại

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

## 2.6. Bộ đệm theo trang trong SRAM

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

## 2.7. Điểm kiểm tra theo trang

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

## 2.8. Khôi phục sau mất nguồn

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

## 2.9. Tính lặp an toàn (Idempotency)

Một thao tác khôi phục tốt phải có thể chạy lại mà không làm hệ thống xấu đi.

```text
Xóa Trang N
Lập trình trang N
Xác minh Trang N
```

Nếu khởi động lại rồi chạy lại đúng chuỗi này với cùng dữ liệu, kết quả vẫn giống nhau. Đó là **khôi phục có tính lặp an toàn (idempotent recovery)**.

---

## 2.10. Sao lưu trước khi cài đặt

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

## 2.11. Metadata A/B

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

## 2.12. CRC ghi cuối Metadata

CRC được đặt ở cuối metadata record.

```text
Lập trình các trường
      ↓
Lập trình CRC cuối cùng
```

Nếu reset xảy ra giữa lúc ghi, CRC chưa hoàn chỉnh và record mới sẽ bị coi là invalid. Record cũ vẫn tồn tại.

---

## 2.13. Độ bền Flash

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

## 2.14. Flash nội và W25Q ngoài

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

## 2.15. Luồng cài đặt firmware đầy đủ

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

## 2.16. Các lỗi thường gặp

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

## 2.17. Trọng tâm cần thuộc

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

# 3. Linker Script + Memory Map

## 3.1. Memory Map là gì?

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

## 3.2. Linker Script là gì?

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

## 3.3. Linker Script của Bootloader

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

## 3.4. Linker Script của Application

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

## 3.5. `ENTRY(Reset_Handler)`

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

## 3.6. Các section chính

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

## 3.7. `.isr_vector`

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

## 3.8. Tại sao dùng `KEEP()`?

Khi linker garbage collection được bật, những section không có reference thông thường có thể bị loại bỏ.

Vector table không nhất thiết được code C gọi trực tiếp nên cần:

```ld
KEEP(*(.isr_vector))
```

Ý nghĩa:

> Luôn giữ vector table trong firmware.

Nếu vector table bị loại bỏ, firmware không thể khởi động đúng.

---

## 3.9. `.text` và `.rodata`

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

## 3.10. `.data`

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

## 3.11. `.bss`

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

## 3.12. Linker Symbol

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

## 3.13. Stack và Heap

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

## 3.14. `ASSERT()` trong Linker Script

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

## 3.15. Vì sao Bootloader và Application cùng dùng toàn bộ RAM?

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

## 3.16. External W25Q có nằm trong Linker Script không?

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

## 3.17. Quan hệ giữa Memory Map, Linker Script và Bootloader

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

## 3.18. Các lỗi thường gặp

- Application vẫn link tại `0x08000000`.
- Vector table không nằm đầu application.
- Application vượt `0x0800F800` và ghi đè metadata.
- Bootloader vượt quá 24 KiB.
- `.data/.bss` vượt quá RAM.
- Linker script và địa chỉ hard-code trong source không đồng nhất.
- Flash binary sang địa chỉ khác với địa chỉ mà linker đã dùng.

---

## 3.19. Mô hình tư duy cần nhớ

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

## 3.20. Trọng tâm cần thuộc

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

