# 1. STM32 Bootloader – từ Reset đến Application

## 1. Bootloader là gì?

Bootloader là firmware chạy đầu tiên sau khi STM32 reset khi MCU boot từ Main Flash. Trong project này, bootloader nằm tại `0x08000000`, còn application được đặt tại `0x08006000`.

Bootloader có nhiệm vụ đọc trạng thái hệ thống, kiểm tra firmware, xử lý OTA/recovery/rollback và chỉ chuyển quyền điều khiển sang application khi application hợp lệ.

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

## 2. Cortex-M3 khởi động như thế nào?

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

## 3. Vector Table

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

Application trong project có vector table riêng tại:

```text
0x08006000
```

Do đó bootloader biết:

```text
[0x08006000] -> Application MSP
[0x08006004] -> Application Reset_Handler
```

---

## 4. MSP – Main Stack Pointer

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

Trước khi jump, bootloader phải kiểm tra MSP của application:

- nằm trong vùng SRAM;
- có alignment hợp lệ.

Nếu MSP sai, application có thể fault ngay khi sử dụng stack.

---

## 5. Reset_Handler

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

Vì vậy bootloader phải jump tới `Reset_Handler`, không nên jump thẳng tới `main()`.

---

## 6. `.data` và `.bss`

### `.data`

Là các biến global/static có giá trị khởi tạo.

```c
uint32_t counter = 123;
```

Giá trị ban đầu được lưu trong Flash, sau đó startup code copy vào RAM.

```text
Flash -> RAM
```

### `.bss`

Là các biến global/static không có giá trị khởi tạo rõ ràng.

```c
uint32_t errors;
```

Startup code zero hóa vùng `.bss` trước khi vào `main()`.

---

## 7. Linker Script

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

## 8. VTOR – Vector Table Offset Register

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

Do đó trước khi jump:

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

## 9. Thumb Bit

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

## 10. Application Validation

Trước khi jump, bootloader nên kiểm tra tối thiểu:

```text
Application MSP
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

## 11. Tại sao phải dọn trạng thái trước khi jump?

Application không nhận một hardware reset thật sự.

Nó kế thừa trạng thái CPU và peripheral từ bootloader.

Bootloader có thể đã sử dụng:

- SysTick;
- UART;
- SPI;
- Timer;
- NVIC;
- PLL/clock tree;
- interrupt masks.

Do đó phải tạo một trạng thái gần giống reset trước khi chuyển quyền điều khiển.

---

## 12. SysTick và Interrupt

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

Nếu chỉ disable interrupt mà không clear pending state, một IRQ cũ của bootloader có thể chạy ngay sau khi application bật interrupt trở lại.

PendSV và SysTick pending cũng nên được clear.

---

## 13. Clock State

Bootloader có thể đã thay đổi clock tree.

Ví dụ:

```text
HSI
HSE
PLL
AHB prescaler
APB prescaler
```

Trước khi jump có thể đưa clock về trạng thái gần reset, ví dụ:

```c
RCC_DeInit();
```

Application sau đó tự cấu hình clock trong `SystemInit()`.

Nguyên tắc:

> Application không nên phụ thuộc vào clock state do bootloader để lại.

---

## 14. Vì sao không nên đổi MSP rồi tiếp tục chạy C?

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

Trong project, helper còn reset thêm:

```text
PSP
CONTROL
BASEPRI
FAULTMASK
PRIMASK
```

để application bắt đầu trong context gần giống sau hardware reset.

---

## 15. CONTROL, PRIMASK, BASEPRI, FAULTMASK

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

## 16. Full Handoff Sequence

Luồng chuyển từ bootloader sang application:

```text
Bootloader đang chạy
        ↓
Đọc Application MSP
Đọc Application Reset_Handler
        ↓
Validate MSP / Reset_Handler
        ↓
Stop SysTick
        ↓
Mask interrupt
        ↓
Disable + clear NVIC IRQ
        ↓
Clear pending SysTick / PendSV
        ↓
Reset clock về trạng thái phù hợp
        ↓
VTOR = Application Vector Table
        ↓
DSB / ISB
        ↓
MSP = Application MSP
        ↓
Reset PSP / CONTROL / interrupt masks
        ↓
BX Application Reset_Handler
        ↓
Application startup
        ↓
.data initialization
.bss initialization
SystemInit()
        ↓
main()
```

---

## 17. Boot Decision

Bootloader không phải lúc nào cũng jump application ngay sau reset.

Trong secure OTA system, nó phải đọc persistent state trước.

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

> Boot decision engine + firmware recovery/update authority.

---

## 18. Các lỗi Bootloader thường gặp

- Application link ở sai địa chỉ.
- Vector table không nằm tại application base.
- Quên đổi `SCB->VTOR`.
- Jump thẳng tới `main()`.
- Không kiểm tra MSP.
- Không kiểm tra Reset_Handler range.
- Không kiểm tra Thumb bit.
- Đổi MSP trong C rồi tiếp tục sử dụng stack.
- Không stop SysTick.
- Không clear pending NVIC interrupt.
- Để `PRIMASK` hoặc `BASEPRI` ở trạng thái sai.
- Application vượt quá partition Flash.
- Jump application dù OTA state đang ở giữa quá trình install.

---

## 19. Mental Model cần nhớ

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
Read boot metadata
  ↓
Boot decision
  ↓
Validate Application
  ↓
Read:
[0x08006000] -> Application MSP
[0x08006004] -> Application Reset_Handler
  ↓
Stop SysTick
Disable/Clear Interrupt
Reset clock state
  ↓
VTOR = 0x08006000
  ↓
MSP = Application MSP
  ↓
BX Application Reset_Handler
  ↓
Application startup
  ↓
main()
```

---

## 20. Cách trả lời ngắn khi phỏng vấn

> STM32 boots into the bootloader because the bootloader owns the vector table at the beginning of internal Flash. The application is linked at a separate Flash address. Before transferring control, the bootloader validates the application's initial stack pointer and reset vector, stops SysTick, disables and clears pending interrupts, restores a reset-like CPU and clock state, relocates `SCB->VTOR` to the application's vector table, loads the application's MSP, and branches to its `Reset_Handler`. The application's startup code then initializes the C runtime and eventually enters `main()`.

---

## Trọng tâm cần thuộc

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
Application Address
↓
VTOR
↓
Thumb Bit
↓
Validate Application
↓
Stop SysTick / Clear Interrupt
↓
Reset CPU Context
↓
Set MSP
↓
BX Reset_Handler
↓
Application main()
```

Nếu giải thích được toàn bộ chuỗi trên mà không nhìn tài liệu, bạn đã nắm phần cốt lõi của STM32 bootloader handoff.
