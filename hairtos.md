# Mục lục

- [1. Cortex-M3 RTOS Foundation](#1-cortex-m3-rtos-foundation)
  - [1.1. Cortex-M3 cung cấp gì cho RTOS?](#11-cortex-m3-cung-cấp-gì-cho-rtos)
  - [1.2. Thread Mode và Handler Mode](#12-thread-mode-và-handler-mode)
  - [1.3. MSP và PSP](#13-msp-và-psp)
  - [1.4. Exception là nền tảng của context switch](#14-exception-là-nền-tảng-của-context-switch)
  - [1.5. Hardware Exception Stack Frame](#15-hardware-exception-stack-frame)
  - [1.6. SysTick](#16-systick)
  - [1.7. SVC](#17-svc)
  - [1.8. PendSV](#18-pendsv)
  - [1.9. NVIC Priority](#19-nvic-priority)
  - [1.10. Critical Section và PRIMASK](#110-critical-section-và-primask)
  - [1.11. Bare-metal và RTOS khác nhau ở đâu?](#111-bare-metal-và-rtos-khác-nhau-ở-đâu)
  - [1.12. Luồng Cortex-M3 khi `hairtos` chạy](#112-luồng-cortex-m3-khi-hairtos-chạy)
  - [1.13. Mô hình tư duy cần nhớ](#113-mô-hình-tư-duy-cần-nhớ)
  - [1.14. Trọng tâm cần thuộc](#114-trọng-tâm-cần-thuộc)
- [2. Task + TCB + Stack](#2-task-tcb-stack)
- [3. SVC + PendSV Context Switch](#3-svc-pendsv-context-switch)
- [4. Scheduler + Ready Queue + Bitmap](#4-scheduler-ready-queue-bitmap)
- [5. Preemption + Round-Robin](#5-preemption-round-robin)
- [6. SysTick + Delay + Timeout](#6-systick-delay-timeout)
- [7. Queue + Blocking IPC](#7-queue-blocking-ipc)
- [8. Semaphore + ISR Synchronization](#8-semaphore-isr-synchronization)
- [9. Mutex + Priority Inversion + Priority Inheritance](#9-mutex-priority-inversion-priority-inheritance)
- [10. Suspend / Resume + Software Timer](#10-suspend-resume-software-timer)
- [11. haievent + Event Ownership + Reference Counting](#11-haievent-event-ownership-reference-counting)
- [12. Active Object + FSM + Publish / Subscribe](#12-active-object-fsm-publish-subscribe)
- [13. Diagnostics + Fault + Stack Safety](#13-diagnostics-fault-stack-safety)
- [14. Architecture Port + Testing + Benchmark](#14-architecture-port-testing-benchmark)

---

# 1. Cortex-M3 RTOS Foundation

`hairtos` là RTOS preemptive chạy trên ARM Cortex-M. Target hoàn chỉnh hiện tại của repo là STM32F103C8T6 / Cortex-M3, với SysTick mặc định 1 kHz và context start/switch dùng `SVC` / `PendSV`.

Muốn hiểu kernel, trước tiên phải hiểu Cortex-M3 cung cấp cơ chế gì để RTOS có thể tạo task, chuyển task và xử lý interrupt.

```text
Cortex-M3
   ↓
Thread / Handler Mode
   ↓
MSP / PSP
   ↓
Exception
   ↓
SysTick / SVC / PendSV
   ↓
RTOS Context Switch
```

---

## 1.1. Cortex-M3 cung cấp gì cho RTOS?

Cortex-M3 có sẵn các cơ chế rất phù hợp để xây RTOS:

```text
Thread Mode
Handler Mode

MSP
PSP

SysTick
SVC
PendSV

NVIC
Exception stacking
```

RTOS không tự tạo một CPU context switch hoàn toàn bằng phần mềm. Nó tận dụng cơ chế exception của Cortex-M3 để CPU tự lưu một phần thanh ghi, sau đó kernel chỉ lưu phần còn lại.

Trong `hairtos`, architecture port chịu trách nhiệm:

```text
PSP / MSP
SVC
PendSV
critical section
context save / restore
```

Kernel phía trên không cần biết chi tiết thanh ghi CPU.

---

## 1.2. Thread Mode và Handler Mode

Cortex-M3 có hai mode thực thi quan trọng:

```text
Thread Mode
= code bình thường / task

Handler Mode
= exception / interrupt handler
```

Sau reset, `main()` chạy trong Thread Mode.

Khi SysTick, SVC, PendSV hoặc một interrupt xảy ra:

```text
Thread Mode
    ↓
Exception
    ↓
Handler Mode
```

Khi exception hoàn tất:

```text
Handler Mode
    ↓
Exception Return
    ↓
Thread Mode
```

Trong `hairtos`, task chạy trong Thread Mode, còn `SVC_Handler`, `PendSV_Handler`, `SysTick_Handler` chạy trong Handler Mode.

---

## 1.3. MSP và PSP

Cortex-M3 có hai stack pointer:

```text
MSP
= Main Stack Pointer

PSP
= Process Stack Pointer
```

Sau reset:

```text
CPU dùng MSP
```

Trong `hairtos`, trước khi kernel bắt đầu task đầu tiên:

```text
main()
→ dùng MSP
```

Sau `SVC`, task chạy bằng:

```text
PSP
```

Trong khi exception handler vẫn dùng:

```text
MSP
```

Mental model:

```text
Kernel startup / Handler
        ↓
       MSP

Task A
Task B
Task C
        ↓
       PSP
```

Mỗi task có stack riêng, và PSP sẽ trỏ vào stack của task đang chạy.

---

## 1.4. Exception là nền tảng của context switch

Context switch trên Cortex-M3 được xây dựa trên exception.

Khi CPU vào exception, hardware tự động lưu một phần context của task hiện tại lên stack.

Sau đó code exception handler có thể:

```text
lưu phần context còn lại
      ↓
chọn task tiếp theo
      ↓
đổi PSP
      ↓
khôi phục context task mới
      ↓
Exception Return
```

Vì vậy context switch không đơn giản là:

```text
gọi function Task B
```

mà là chuyển toàn bộ trạng thái CPU từ task này sang task khác.

---

## 1.5. Hardware Exception Stack Frame

Khi Cortex-M3 vào exception từ task đang dùng PSP, hardware tự động lưu:

```text
R0
R1
R2
R3
R12
LR
PC
xPSR
```

Có thể hình dung:

```text
Task đang chạy
     ↓
Exception xảy ra
     ↓
CPU tự push:

R0-R3
R12
LR
PC
xPSR
```

Các thanh ghi:

```text
R4-R11
```

không được hardware tự lưu.

Vì vậy `PendSV` của `hairtos` phải tự:

```text
Save R4-R11
      ↓
Switch task
      ↓
Restore R4-R11
```

Sau đó Cortex-M3 tự khôi phục hardware frame khi exception return.

Đây là ý quan trọng nhất để hiểu context switch ở các chương sau.

---

## 1.6. SysTick

`SysTick` là timer nằm trong Cortex-M core và thường được RTOS dùng tạo nhịp thời gian.

Trong target STM32F103 của `hairtos`:

```text
Kernel Tick = 1 kHz
```

tức khoảng:

```text
1 tick = 1 ms
```

SysTick giúp kernel xử lý các chức năng như:

```text
task delay
timeout
time slicing
software timer
```

Trước khi kernel tồn tại, example bare-metal của repo cũng dùng một millisecond timebase cho:

```text
board_millis()
board_delay_ms()
```

Nhưng lúc đó chưa có task, scheduler hay context switch.

---

## 1.7. SVC

`SVC` là:

```text
Supervisor Call
```

Trong `hairtos`, SVC được dùng để bắt đầu task đầu tiên.

Luồng:

```text
main()
  ↓
MSP
  ↓
Start Kernel
  ↓
SVC
  ↓
SVC_Handler
  ↓
khôi phục context task đầu tiên
  ↓
PSP
  ↓
Exception Return
  ↓
Task đầu tiên chạy
```

Điểm quan trọng:

> `SVC` chủ yếu giải quyết bước chuyển từ môi trường startup dùng MSP sang task đầu tiên dùng PSP.

Sau khi task đầu tiên đã chạy, các context switch tiếp theo chủ yếu dùng `PendSV`.

---

## 1.8. PendSV

`PendSV` là exception được thiết kế rất phù hợp cho context switch.

Trong `hairtos`:

```text
Context switch
→ PendSV
```

Luồng cơ bản:

```text
Task A đang chạy
      ↓
cần chuyển task
      ↓
PendSV pending
      ↓
CPU vào PendSV
      ↓
hardware đã lưu
R0-R3, R12, LR, PC, xPSR
      ↓
PendSV lưu R4-R11
      ↓
lưu PSP của Task A vào TCB
      ↓
kernel chọn Task B
      ↓
nạp PSP của Task B
      ↓
restore R4-R11
      ↓
Exception Return
      ↓
Task B tiếp tục chạy
```

Chương 3 sẽ đi sâu toàn bộ quá trình này.

---

## 1.9. NVIC Priority

Cortex-M3 dùng NVIC để quản lý interrupt và exception priority.

Trong `hairtos`, `PendSV` được đặt ở priority thấp nhất để context switch không chen ngang các interrupt quan trọng hơn.

Mental model:

```text
Interrupt quan trọng
      ↓
xử lý trước

PendSV
      ↓
chờ

Interrupt kết thúc
      ↓
PendSV chạy
      ↓
Context Switch
```

Điều này giúp scheduler không chuyển task ở giữa lúc một ISR quan trọng đang xử lý.

---

## 1.10. Critical Section và PRIMASK

Kernel có những đoạn code cập nhật dữ liệu dùng chung như:

```text
ready list
wait list
timeout list
scheduler state
```

Các thao tác này phải được bảo vệ khỏi interrupt chen ngang tại thời điểm không an toàn.

`hairtos` Cortex-M3 hiện dùng:

```text
PRIMASK
```

để tạo critical section.

Có thể hiểu đơn giản:

```text
Enter Critical
      ↓
mask interrupt
      ↓
thay đổi kernel state
      ↓
Exit Critical
      ↓
khôi phục interrupt state
```

Critical section phải ngắn và có thời gian hữu hạn.

Không được gọi operation có thể block bên trong critical section.

Repo v1 chưa dùng `BASEPRI` làm interrupt ceiling cho Cortex-M3.

---

## 1.11. Bare-metal và RTOS khác nhau ở đâu?

Example đầu tiên của repo cố tình chưa dùng kernel.

Bare-metal:

```text
Reset
  ↓
board_init()
  ↓
while (1)
{
    toggle LED
    print UART
    delay 500 ms
}
```

Khi `board_delay_ms()` đang blocking:

```text
CPU không chuyển sang task khác
```

Trong RTOS:

```text
Task A delay
    ↓
Task A BLOCKED
    ↓
Scheduler chọn Task B
    ↓
CPU làm công việc khác
```

Đây là thay đổi tư duy quan trọng:

```text
Bare-metal
= một luồng điều khiển chính

RTOS
= nhiều task chia sẻ CPU
```

---

## 1.12. Luồng Cortex-M3 khi `hairtos` chạy

Có thể hình dung toàn bộ nền tảng như sau:

```text
Reset
  ↓
Startup
  ↓
main()
  ↓
MSP
  ↓
Kernel init
  ↓
Create tasks
  ↓
Start scheduler
  ↓
SVC
  ↓
Task đầu tiên
  ↓
PSP
  ↓
SysTick / IRQ / Yield / Wakeup
  ↓
Scheduler quyết định cần đổi task?
       /            \
     NO              YES
     ↓                ↓
 tiếp tục           PendSV
                      ↓
                Save Context
                      ↓
                Select Task
                      ↓
                Restore Context
                      ↓
                Exception Return
                      ↓
                   Task mới
```

---

## 1.13. Mô hình tư duy cần nhớ

```text
Thread Mode
= nơi task chạy

Handler Mode
= nơi exception / ISR chạy

MSP
= startup + Handler Mode

PSP
= stack của task

SysTick
= nhịp thời gian kernel

SVC
= bắt đầu task đầu tiên

PendSV
= context switch

PRIMASK
= bảo vệ critical section
```

Quan hệ quan trọng:

```text
Task
  ↓
PSP
  ↓
Exception xảy ra
  ↓
Hardware tự lưu một phần context
  ↓
Handler dùng MSP
  ↓
PendSV lưu phần còn lại
  ↓
đổi task
  ↓
Exception Return
  ↓
Task mới tiếp tục bằng PSP
```

---

## 1.14. Trọng tâm cần thuộc

Chuỗi quan trọng nhất:

```text
Cortex-M3
↓
Thread Mode / Handler Mode
↓
MSP / PSP
↓
Exception Stacking
↓
SysTick
↓
SVC
↓
PendSV
↓
Context Switch
```

Các ý phải giải thích được:

- Sau reset CPU bắt đầu với MSP.
- `hairtos` cho task chạy bằng PSP.
- Handler Mode tiếp tục dùng MSP.
- Khi vào exception, hardware tự lưu `R0-R3`, `R12`, `LR`, `PC`, `xPSR`.
- `PendSV` lưu/khôi phục thêm `R4-R11`.
- `SVC` được dùng để bắt đầu task đầu tiên.
- `PendSV` xử lý các context switch sau đó.
- `SysTick` cung cấp kernel tick 1 kHz trên target tham chiếu.
- `PendSV` được đặt priority thấp để context switch diễn ra sau các interrupt quan trọng.
- Critical section của Cortex-M3 port hiện dùng `PRIMASK`.
- Cortex-M3 target không có FPU nên v1 không cần lưu FPU context.

Nếu giải thích được chuỗi:

```text
main/MSP
→ SVC
→ task/PSP
→ SysTick hoặc sự kiện scheduling
→ PendSV
→ save context
→ chọn task mới
→ restore context
→ exception return
```

thì bạn đã có nền tảng cần thiết để học `Task + TCB + Stack` và context switch của `hairtos`.

---

# 2. Task + TCB + Stack

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 3. SVC + PendSV Context Switch

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 4. Scheduler + Ready Queue + Bitmap

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 5. Preemption + Round-Robin

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 6. SysTick + Delay + Timeout

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 7. Queue + Blocking IPC

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 8. Semaphore + ISR Synchronization

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 9. Mutex + Priority Inversion + Priority Inheritance

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 10. Suspend / Resume + Software Timer

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 11. haievent + Event Ownership + Reference Counting

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 12. Active Object + FSM + Publish / Subscribe

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 13. Diagnostics + Fault + Stack Safety

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 14. Architecture Port + Testing + Benchmark

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.
