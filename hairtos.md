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
  - [2.1. Task là gì?](#21-task-là-gì)
  - [2.2. TCB là gì?](#22-tcb-là-gì)
  - [2.3. Vì sao TCB cần lưu Stack Pointer?](#23-vì-sao-tcb-cần-lưu-stack-pointer)
  - [2.4. Mỗi Task có Stack riêng](#24-mỗi-task-có-stack-riêng)
  - [2.5. `stack_words` nghĩa là gì?](#25-stack_words-nghĩa-là-gì)
  - [2.6. Task được tạo như thế nào?](#26-task-được-tạo-như-thế-nào)
  - [2.7. Initial Stack Frame là gì?](#27-initial-stack-frame-là-gì)
  - [2.8. Initial Stack Frame trong `hairtos`](#28-initial-stack-frame-trong-hairtos)
  - [2.9. Vì sao `R0` chứa Task Argument?](#29-vì-sao-r0-chứa-task-argument)
  - [2.10. `PC`, `LR` và `xPSR`](#210-pc-lr-và-xpsr)
  - [2.11. Task State](#211-task-state)
  - [2.12. Base Priority và Effective Priority](#212-base-priority-và-effective-priority)
  - [2.13. Static Task Allocation](#213-static-task-allocation)
  - [2.14. Quan hệ Task – TCB – Stack](#214-quan-hệ-task--tcb--stack)
  - [2.15. Mô hình tư duy cần nhớ](#215-mô-hình-tư-duy-cần-nhớ)
  - [2.16. Trọng tâm cần thuộc](#216-trọng-tâm-cần-thuộc)
- [3. SVC + PendSV Context Switch](#3-svc-pendsv-context-switch)
  - [3.1. Vì sao Cortex-M dùng SVC và PendSV?](#31-vì-sao-cortex-m-dùng-svc-và-pendsv)
  - [3.2. `hr_port_start_first_task()` làm gì?](#32-hr_port_start_first_task-làm-gì)
  - [3.3. `SVC_Handler` khởi động task đầu tiên](#33-svc_handler-khởi-động-task-đầu-tiên)
  - [3.4. `EXC_RETURN = 0xFFFFFFFD`](#34-exc_return--0xfffffffd)
  - [3.5. Khi nào yêu cầu Context Switch?](#35-khi-nào-yêu-cầu-context-switch)
  - [3.6. `PendSV_Handler` lưu Task hiện tại](#36-pendsv_handler-lưu-task-hiện-tại)
  - [3.7. Vì sao `stack_pointer` phải ở offset 0 của TCB?](#37-vì-sao-stack_pointer-phải-ở-offset-0-của-tcb)
  - [3.8. Kernel chọn Task tiếp theo](#38-kernel-chọn-task-tiếp-theo)
  - [3.9. Khôi phục Task mới](#39-khôi-phục-task-mới)
  - [3.10. Vì sao PendSV có Priority thấp nhất?](#310-vì-sao-pendsv-có-priority-thấp-nhất)
  - [3.11. SysTick / ISR và PendSV phối hợp thế nào?](#311-systick--isr-và-pendsv-phối-hợp-thế-nào)
  - [3.12. Ví dụ Task A → Task B](#312-ví-dụ-task-a--task-b)
  - [3.13. Mô hình tư duy cần nhớ](#313-mô-hình-tư-duy-cần-nhớ)
  - [3.14. Trọng tâm cần thuộc](#314-trọng-tâm-cần-thuộc)
- [4. Scheduler + Ready Queue + Bitmap](#4-scheduler-ready-queue-bitmap)
  - [4.1. Scheduler có nhiệm vụ gì?](#41-scheduler-có-nhiệm-vụ-gì)
  - [4.2. Fixed-Priority Preemptive Scheduling](#42-fixed-priority-preemptive-scheduling)
  - [4.3. Priority trong `hairtos`](#43-priority-trong-hairtos)
  - [4.4. Ready Set](#44-ready-set)
  - [4.5. Ready Queue theo từng Priority](#45-ready-queue-theo-từng-priority)
  - [4.6. Ready Bitmap](#46-ready-bitmap)
  - [4.7. Scheduler chọn Task tiếp theo thế nào?](#47-scheduler-chọn-task-tiếp-theo-thế-nào)
  - [4.8. Khi nào xảy ra Preemption?](#48-khi-nào-xảy-ra-preemption)
  - [4.9. Task cùng Priority](#49-task-cùng-priority)
  - [4.10. READY, RUNNING và BLOCKED](#410-ready-running-và-blocked)
  - [4.11. `effective_priority` và Scheduler](#411-effective_priority-và-scheduler)
  - [4.12. Scheduler và PendSV khác nhau](#412-scheduler-và-pendsv-khác-nhau)
  - [4.13. Ví dụ cụ thể](#413-ví-dụ-cụ-thể)
  - [4.14. Mô hình tư duy cần nhớ](#414-mô-hình-tư-duy-cần-nhớ)
  - [4.15. Trọng tâm cần thuộc](#415-trọng-tâm-cần-thuộc)
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

Trong RTOS, một `Task` không chỉ là một function. Để task có thể bị dừng giữa chừng rồi tiếp tục đúng vị trí cũ, kernel phải lưu được **trạng thái thực thi và thông tin quản lý của task**.

```text
Task
=
Task Function
+
TCB
+
Stack riêng
```

Trong `hairtos`, task được tạo tĩnh: application cung cấp sẵn `hr_task_t` và vùng stack, kernel không cần cấp phát động bộ nhớ cho task.

---

## 2.1. Task là gì?

Task là một luồng thực thi độc lập. Một task thường có function dạng:

```c
void task(void *argument)
{
    while (1)
    {
        /* xử lý */
    }
}
```

Nhưng để scheduler quản lý được, task còn cần:

```text
stack riêng
priority
state
CPU context
thông tin scheduler
```

Nếu chỉ có function mà không có TCB và stack, kernel không thể dừng Task A rồi tiếp tục Task A tại đúng instruction trước đó.

---

## 2.2. TCB là gì?

TCB là:

```text
Task Control Block
```

Có thể hiểu là hồ sơ quản lý của task.

```text
Task A
 ├── Task Function
 ├── Stack
 └── TCB
      ├── saved stack pointer
      ├── priority
      ├── state
      └── scheduler information
```

Trong `hairtos`, `hr_task_t` là vùng storage do application sở hữu, còn cấu trúc quản lý chi tiết bên trong thuộc kernel.

---

## 2.3. Vì sao TCB cần lưu Stack Pointer?

Giả sử Task A đang chạy bằng PSP:

```text
PSP
 ↓
Stack Task A
```

Khi context switch:

```text
Task A
  ↓
Save CPU Context
  ↓
lưu PSP hiện tại vào TCB A
```

Scheduler chọn Task B:

```text
TCB B
  ↓
lấy saved PSP
  ↓
PSP = Stack Task B
  ↓
Restore CPU Context
```

Điểm cần nhớ:

> **Saved Stack Pointer trong TCB là điểm để kernel tìm lại context đã lưu của task.**

---

## 2.4. Mỗi Task có Stack riêng

Các task không dùng chung một stack.

```text
Task A → Stack A
Task B → Stack B
Task C → Stack C
```

Stack của task chứa:

```text
local variables
return address
function call frame
saved registers
CPU context
```

Khi chuyển task, kernel đổi PSP từ stack của task hiện tại sang stack của task tiếp theo.

---

## 2.5. `stack_words` nghĩa là gì?

Trong `hairtos`:

```text
hr_stack_t = uint32_t
```

nên:

```text
1 stack word = 4 byte
```

Ví dụ:

```text
stack_words = 256
```

thì stack có kích thước:

```text
256 × 4 = 1024 byte
```

`stack_words` là số word, không phải số byte.

---

## 2.6. Task được tạo như thế nào?

API chính có dạng:

```c
hr_task_create_static(
    task,
    name,
    entry,
    argument,
    stack,
    stack_words,
    priority
);
```

Luồng khái niệm:

```text
Task Function
+
Argument
+
Stack Memory
+
Priority
      ↓
Create Task
      ↓
Initialize TCB
      ↓
Build Initial Stack Frame
      ↓
Task sẵn sàng cho scheduler
```

---

## 2.7. Initial Stack Frame là gì?

Task mới chưa từng thực sự chạy nên chưa có CPU context thật để restore.

Kernel giải quyết bằng cách dựng sẵn một context trên stack:

```text
Task chưa từng chạy
        ↓
Kernel dựng stack giống như
task vừa bị exception
        ↓
Context Restore
        ↓
CPU bắt đầu tại task entry
```

Đây là kỹ thuật cốt lõi để khởi động một task mới trên Cortex-M.

---

## 2.8. Initial Stack Frame trong `hairtos`

Có thể hình dung initial frame như sau:

```text
Saved PSP
  ↓
+----------------+
| R4             |
| R5             |
| R6             |
| R7             |
| R8             |
| R9             |
| R10            |
| R11            |
+----------------+
| R0             |
| R1             |
| R2             |
| R3             |
| R12            |
| LR             |
| PC             |
| xPSR           |
+----------------+
```

Phần:

```text
R4-R11
```

là context mà phần mềm phải lưu/khôi phục.

Phần:

```text
R0-R3, R12, LR, PC, xPSR
```

có hình dạng giống hardware exception frame của Cortex-M3.

---

## 2.9. Vì sao `R0` chứa Task Argument?

Theo ARM calling convention, argument đầu tiên của function nằm trong:

```text
R0
```

Task entry có dạng:

```c
void task(void *argument);
```

nên initial frame được dựng:

```text
R0 = argument
PC = task entry
```

Khi task được restore lần đầu, CPU bắt đầu tại task entry và nhận đúng argument.

---

## 2.10. `PC`, `LR` và `xPSR`

Initial frame cần chuẩn bị:

```text
PC
= task entry

R0
= task argument

LR
= task exit handler

xPSR
= Thumb state
```

Cortex-M3 chỉ chạy Thumb/Thumb-2 nên bit Thumb trong xPSR phải hợp lệ.

Nếu task function vô tình return, `LR` phải dẫn tới một handler hợp lệ thay vì địa chỉ ngẫu nhiên.

---

## 2.11. Task State

Các trạng thái cần hiểu:

```text
CREATED
READY
RUNNING
BLOCKED
SUSPENDED
```

Ý nghĩa:

```text
CREATED
= đã tạo

READY
= sẵn sàng chạy

RUNNING
= đang dùng CPU

BLOCKED
= đang chờ timeout / queue / semaphore / sự kiện

SUSPENDED
= bị chủ động loại khỏi scheduler
```

Luồng thường gặp:

```text
CREATED
   ↓
READY
   ↓
RUNNING
   ↓
READY
```

hoặc:

```text
RUNNING
   ↓
BLOCKED
   ↓
READY
```

---

## 2.12. Base Priority và Effective Priority

Để hỗ trợ Priority Inheritance, kernel cần phân biệt:

```text
base_priority
effective_priority
```

Bình thường:

```text
base_priority
=
effective_priority
```

Khi một task priority thấp giữ mutex mà task priority cao đang chờ, `effective_priority` có thể được nâng tạm thời.

Phần này sẽ được học kỹ ở chương Mutex + Priority Inheritance.

---

## 2.13. Static Task Allocation

`hairtos` dùng:

```text
caller-owned TCB storage
+
caller-owned stack
```

Ví dụ:

```c
static hr_task_t task_a;
static hr_stack_t task_a_stack[256];
```

Ưu điểm:

```text
RAM xác định trước
không phụ thuộc heap
không fragmentation runtime
dễ tính bộ nhớ
hành vi dễ dự đoán
```

Đây là thiết kế phù hợp với embedded RTOS cần tính xác định cao.

---

## 2.14. Quan hệ Task – TCB – Stack

```text
                 TASK A

       +----------------------+
       | Task Function        |
       +----------------------+

       +----------------------+
       | TCB A                |
       | saved PSP -----------+------+
       | priority             |      |
       | state                |      |
       | scheduler info       |      |
       +----------------------+      |
                                     |
                                     ↓
       +-----------------------------+
       | Stack A                     |
       | local variables             |
       | function frames             |
       | saved CPU context           |
       +-----------------------------+
```

Context switch về bản chất là:

```text
Save PSP → TCB Task A
        ↓
Scheduler chọn Task B
        ↓
Load PSP ← TCB Task B
```

---

## 2.15. Mô hình tư duy cần nhớ

```text
TASK
= code muốn chạy

TCB
= hồ sơ quản lý task

STACK
= nơi giữ trạng thái thực thi của task
```

Khi tạo task:

```text
Task Function
+
Argument
+
Stack
+
Priority
      ↓
Initialize TCB
      ↓
Build Initial Stack Frame
      ↓
Saved PSP
```

Khi context switch:

```text
Task A
  ↓
Save Context vào Stack A
  ↓
Save PSP vào TCB A
  ↓
Select Task B
  ↓
Load PSP từ TCB B
  ↓
Restore Context từ Stack B
  ↓
Task B
```

---

## 2.16. Trọng tâm cần thuộc

```text
Task
↓
TCB
↓
Private Stack
↓
Initial Stack Frame
↓
PSP
↓
Context Save / Restore
```

Các ý quan trọng:

```text
Task
= function + TCB + stack

TCB
= quản lý task + saved PSP

Stack
= call frame + CPU context

R0
= argument đầu tiên

PC
= task entry

R4-R11
= software context

R0-R3, R12, LR, PC, xPSR
= hardware-style exception frame
```

Điểm quan trọng nhất:

> **TCB không cần chứa toàn bộ register của task; phần lớn CPU context nằm trên stack của task. TCB giữ thông tin quản lý và saved PSP để kernel tìm lại stack đó.**

> **Task mới được khởi động bằng cách dựng sẵn Initial Stack Frame, sau đó Cortex-M restore context và bắt đầu chạy tại task entry.**


---

# 3. SVC + PendSV Context Switch

Trong `hairtos`, context switch của Cortex-M3 dựa vào hai exception chính:

```text
SVC
= bắt đầu task đầu tiên

PendSV
= chuyển giữa các task sau đó
```

Cortex-M3 tự lưu một phần CPU context khi vào exception, còn `hairtos` chỉ cần lưu phần còn lại và đổi PSP sang stack của task tiếp theo.

```text
Hardware save
+
Software save
+
Switch PSP
=
Context Switch
```

---

## 3.1. Vì sao Cortex-M dùng SVC và PendSV?

Nếu chỉ gọi trực tiếp một task như function bình thường:

```text
main()
  ↓
task_a()
```

thì kernel chưa có cơ chế để:

```text
dừng Task A
khôi phục Task B
quay lại đúng vị trí cũ của Task A
```

Cortex-M3 cung cấp exception mechanism để xử lý việc này.

Trong `hairtos`:

```text
SVC
→ chuyển từ startup dùng MSP sang task đầu tiên dùng PSP

PendSV
→ thực hiện các context switch tiếp theo
```

---

## 3.2. `hr_port_start_first_task()` làm gì?

Khi scheduler đã chọn task đầu tiên, `hairtos` gọi:

```text
hr_port_start_first_task()
```

Luồng chính:

```text
Disable interrupt
      ↓
MSP = _estack
      ↓
DSB / ISB
      ↓
Enable interrupt
      ↓
svc #0
```

Việc đặt lại MSP về `_estack` giúp Handler Mode bắt đầu từ stack kernel sạch trước khi SVC chạy.

Sau lệnh:

```asm
svc #0
```

CPU vào:

```text
SVC_Handler
```

---

## 3.3. `SVC_Handler` khởi động task đầu tiên

`SVC_Handler` lấy:

```text
g_hr_current_task_control_block
```

tức TCB của task mà scheduler đã chọn.

Vì `stack_pointer` nằm ở đầu TCB:

```text
TCB
+0x00 → stack_pointer
```

handler có thể lấy saved PSP trực tiếp.

Luồng:

```text
Current TCB
    ↓
saved PSP
    ↓
Restore R4-R11
    ↓
PSP = vị trí hardware frame
    ↓
CONTROL.SPSEL = 1
    ↓
Thread Mode sẽ dùng PSP
```

Sau đó exception return sẽ khôi phục phần context còn lại và bắt đầu task.

---

## 3.4. `EXC_RETURN = 0xFFFFFFFD`

Trong `SVC_Handler`, `hairtos` nạp:

```text
LR = 0xFFFFFFFD
```

rồi:

```asm
bx lr
```

Giá trị này không phải địa chỉ function thông thường. Nó là một mã:

```text
EXC_RETURN
```

Nó yêu cầu Cortex-M:

```text
return về Thread Mode
+
dùng PSP
+
khôi phục standard exception frame
```

Do đó CPU tự pop:

```text
R0-R3
R12
LR
PC
xPSR
```

từ stack của task.

Kết quả:

```text
PC = task entry
R0 = task argument
```

và task đầu tiên bắt đầu chạy.

---

## 3.5. Khi nào yêu cầu Context Switch?

Khi kernel quyết định cần đổi task, nó không thực hiện toàn bộ context switch ngay tại chỗ.

Port gọi:

```text
hr_port_request_context_switch()
```

Hàm này set bit:

```text
PENDSVSET
```

trong:

```text
SCB->ICSR
```

Mental model:

```text
Kernel:
"Cần đổi task"
      ↓
Set PendSV Pending
      ↓
CPU chạy PendSV khi phù hợp
```

Điều này tách:

```text
quyết định scheduling
```

khỏi:

```text
cơ chế save / restore CPU context
```

---

## 3.6. `PendSV_Handler` lưu Task hiện tại

Khi CPU vào PendSV từ một task đang dùng PSP, hardware đã tự lưu:

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

lên stack của task.

`PendSV_Handler` đọc:

```asm
mrs r0, psp
```

lấy PSP hiện tại.

Sau đó `hairtos` tự lưu:

```text
R4-R11
```

bằng:

```asm
stmdb r0!, {r4-r11}
```

Lúc này toàn bộ CPU context cần thiết đã nằm trên stack task.

Sau đó:

```text
saved PSP
→ ghi vào TCB của task hiện tại
```

Có thể hình dung:

```text
Stack Task A

+----------------+
| R4-R11         | ← software save
+----------------+
| R0-R3          |
| R12            |
| LR             |
| PC             |
| xPSR           | ← hardware save
+----------------+
        ↑
saved PSP trong TCB A
```

---

## 3.7. Vì sao `stack_pointer` phải ở offset 0 của TCB?

Internal TCB của `hairtos` bắt đầu bằng:

```c
hr_stack_t *stack_pointer;
```

và repository còn dùng `_Static_assert` để bảo đảm:

```text
offsetof(stack_pointer) == 0
```

Lý do là assembly cần thao tác cực ngắn:

```text
TCB pointer
    ↓
[TCB + 0]
    ↓
saved PSP
```

PendSV không cần biết toàn bộ layout của struct C.

Điều này tạo một hợp đồng rõ ràng:

```text
C kernel
↔
Assembly port
```

---

## 3.8. Kernel chọn Task tiếp theo

Sau khi lưu Task hiện tại, `PendSV_Handler` gọi:

```text
hr_kernel_select_next_from_pendsv()
```

Trước lời gọi C, port:

```text
disable interrupt
```

và giữ `EXC_RETURN` an toàn trên MSP.

Kernel selector có nhiệm vụ chọn task tiếp theo và cập nhật:

```text
g_hr_current_task_control_block
```

Sau khi function C trả về:

```text
Current TCB
= TCB của task mới
```

PendSV không tự triển khai thuật toán scheduler.

Nó chỉ hỏi kernel:

```text
"Task tiếp theo là ai?"
```

Phần scheduler sẽ được học ở Chương 4.

---

## 3.9. Khôi phục Task mới

Sau khi kernel chọn Task B:

```text
TCB B
   ↓
saved PSP
```

PendSV lấy saved PSP của Task B rồi:

```asm
ldmia r0!, {r4-r11}
```

để khôi phục:

```text
R4-R11
```

Sau đó:

```text
PSP = vị trí hardware frame của Task B
```

và:

```asm
bx lr
```

thực hiện exception return.

Cortex-M tự khôi phục:

```text
R0-R3
R12
LR
PC
xPSR
```

Kết quả:

```text
Task B tiếp tục đúng vị trí
mà nó từng bị dừng
```

---

## 3.10. Vì sao PendSV có Priority thấp nhất?

`hairtos` cấu hình:

```text
SVC     → priority cao
SysTick → priority trung bình
PendSV  → priority thấp nhất
```

PendSV thấp nhất để nó không chen ngang interrupt hoặc exception quan trọng hơn.

Ví dụ:

```text
UART ISR đang chạy
      ↓
phát hiện Task High cần được đánh thức
      ↓
PendSV được set pending
      ↓
UART ISR vẫn hoàn thành
      ↓
CPU thoát ISR
      ↓
PendSV mới chạy
      ↓
Context Switch
```

Điều này rất quan trọng vì context switch cần được thực hiện tại thời điểm CPU context ổn định.

---

## 3.11. SysTick / ISR và PendSV phối hợp thế nào?

SysTick hoặc một ISR có thể làm thay đổi trạng thái task.

Ví dụ:

```text
SysTick
   ↓
timeout của Task B hết
   ↓
Task B chuyển BLOCKED → READY
```

Nếu Task B có priority cao hơn Task A:

```text
cần context switch
```

Nhưng ISR không cần tự save/restore toàn bộ task.

Nó chỉ yêu cầu:

```text
PendSV pending
```

Sau khi ISR kết thúc:

```text
PendSV
   ↓
Save Task A
   ↓
Select Task B
   ↓
Restore Task B
```

Trong `hairtos`, API:

```text
hr_port_yield_from_isr(required)
```

sẽ request PendSV nếu:

```text
required = true
```

---

## 3.12. Ví dụ Task A → Task B

Giả sử:

```text
Task A
priority = 5
RUNNING

Task B
priority = 2
BLOCKED
```

Một interrupt làm Task B trở thành READY.

Luồng:

```text
Task A đang chạy bằng PSP_A
        ↓
IRQ xảy ra
        ↓
hardware save context Task A
        ↓
ISR đánh thức Task B
        ↓
Task B priority cao hơn
        ↓
PendSV pending
        ↓
ISR kết thúc
        ↓
PendSV_Handler
        ↓
save R4-R11 của Task A
        ↓
TCB A.stack_pointer = PSP_A
        ↓
hr_kernel_select_next_from_pendsv()
        ↓
Current TCB = Task B
        ↓
load PSP_B
        ↓
restore R4-R11 của Task B
        ↓
Exception Return
        ↓
hardware restore Task B
        ↓
Task B RUNNING
```

Task A không mất context.

Khi được chọn lại sau này:

```text
TCB A
→ saved PSP_A
→ restore context
→ Task A tiếp tục đúng instruction cũ
```

---

## 3.13. Mô hình tư duy cần nhớ

Khởi động task đầu tiên:

```text
main()
  ↓
MSP
  ↓
Scheduler chọn Task A
  ↓
hr_port_start_first_task()
  ↓
SVC
  ↓
Restore R4-R11
  ↓
PSP = Stack A
  ↓
EXC_RETURN
  ↓
Restore hardware frame
  ↓
Task A
```

Context switch bình thường:

```text
Task A
  ↓
PendSV Pending
  ↓
Hardware Save
R0-R3, R12, LR, PC, xPSR
  ↓
PendSV Save
R4-R11
  ↓
Save PSP → TCB A
  ↓
Select Task B
  ↓
Load PSP ← TCB B
  ↓
Restore R4-R11
  ↓
Exception Return
  ↓
Hardware Restore
  ↓
Task B
```

---

## 3.14. Trọng tâm cần thuộc

Chuỗi quan trọng nhất:

```text
SVC
= start first task

PendSV
= switch task
```

Phải giải thích được:

```text
Hardware tự save:
R0-R3, R12, LR, PC, xPSR

PendSV tự save:
R4-R11

TCB:
lưu saved PSP

Scheduler:
chọn next TCB

EXC_RETURN:
khôi phục Thread Mode bằng PSP
```

Các function quan trọng của `hairtos`:

```text
hr_port_start_first_task()
SVC_Handler
hr_port_request_context_switch()
PendSV_Handler
hr_kernel_select_next_from_pendsv()
```

Các điểm cần nhớ:

- Task chạy Thread Mode bằng PSP.
- Exception handler chạy bằng MSP.
- `SVC` chỉ cần cho bước bắt đầu task đầu tiên.
- Các context switch sau dùng `PendSV`.
- `PendSV` có priority thấp nhất.
- Assembly chỉ save/restore context; thuật toán chọn task nằm ở kernel.
- `stack_pointer` là field đầu tiên của TCB để assembly truy cập trực tiếp.
- `EXC_RETURN = 0xFFFFFFFD` đưa CPU về Thread Mode dùng PSP.
- Cortex-M3 target không có FPU nên không có FPU context cần save/restore.

Câu cần nhớ nhất:

> **Context switch trên Cortex-M3 không phải copy toàn bộ register giữa hai TCB. CPU context được lưu trên stack riêng của từng task; TCB chỉ giữ saved PSP để kernel đổi từ stack task này sang stack task khác.**

> **SVC đưa hệ thống từ `main()`/MSP sang task đầu tiên/PSP, còn PendSV thực hiện mọi context switch tiếp theo.**


---

# 4. Scheduler + Ready Queue + Bitmap

Scheduler trả lời một câu hỏi cốt lõi:

> **Trong các task đang sẵn sàng chạy, task nào phải được CPU chạy tiếp theo?**

`hairtos` dùng **fixed-priority preemptive scheduler**. Mỗi priority có một FIFO ready queue riêng, còn bitmap cho biết priority nào hiện đang có task sẵn sàng.

```text
Task trở thành READY
        ↓
đưa vào Ready Queue
        ↓
cập nhật Bitmap
        ↓
Scheduler chọn Priority cao nhất
        ↓
chọn Task đầu queue
        ↓
PendSV thực hiện Context Switch
```

---

## 4.1. Scheduler có nhiệm vụ gì?

Scheduler không trực tiếp save/restore register. Việc đó thuộc Cortex-M3 port và `PendSV`.

Scheduler chỉ quyết định:

```text
Task nào đang READY?
Task nào có priority cao nhất?
Task hiện tại có cần bị preempt không?
Task tiếp theo là task nào?
```

Có thể phân biệt:

```text
Scheduler
= quyết định chạy ai

PendSV
= thực hiện việc chuyển CPU sang task đó
```

---

## 4.2. Fixed-Priority Preemptive Scheduling

`hairtos` dùng:

```text
Fixed Priority
+
Preemption
```

`Fixed Priority` nghĩa là mỗi task có một priority được cấu hình và scheduler luôn ưu tiên task có priority cao hơn.

`Preemption` nghĩa là nếu một task priority cao hơn trở thành READY, nó có thể giành CPU từ task priority thấp hơn đang chạy.

Ví dụ:

```text
Task A priority = 5
Task B priority = 2
```

Nếu A đang chạy và B trở thành READY:

```text
B priority cao hơn
        ↓
Scheduler yêu cầu Context Switch
        ↓
Task B chạy
```

---

## 4.3. Priority trong `hairtos`

Cấu hình hiện tại có:

```text
HR_CFG_PRIORITY_COUNT = 8
```

tức:

```text
priority 0
priority 1
...
priority 7
```

Quy tắc rất quan trọng:

```text
Số càng nhỏ
=
Priority càng cao
```

Do đó:

```text
0 = cao nhất
7 = thấp nhất
```

Idle task chạy ở:

```text
priority 7
```

Nên khi không có application task nào cần CPU, idle task là phương án cuối cùng.

---

## 4.4. Ready Set

Scheduler quản lý các task có khả năng được chạy thông qua:

```text
Ready Set
```

Trong `hairtos`, cấu trúc khái niệm là:

```text
Ready Set
├── queues[0]
├── queues[1]
├── ...
├── queues[7]
├── bitmap
└── count
```

Mỗi task có một:

```text
ready_node
```

được kernel dùng để đưa task vào ready set.

Ready set không chứa bản copy của task; nó liên kết trực tiếp node nằm trong TCB.

Đây là:

```text
intrusive list
```

---

## 4.5. Ready Queue theo từng Priority

Mỗi mức priority có một FIFO queue riêng.

Ví dụ:

```text
Priority 0:
[ Task A ] → [ Task B ]

Priority 1:
[ Task C ]

Priority 2:
empty

Priority 3:
[ Task D ] → [ Task E ]
```

Scheduler không cần so sánh tất cả task với nhau.

Nó chỉ cần:

```text
1. tìm priority cao nhất đang có task
2. lấy task đầu FIFO của priority đó
```

Task cùng priority giữ thứ tự FIFO.

---

## 4.6. Ready Bitmap

Nếu chỉ có các queue, scheduler vẫn cần biết queue nào không rỗng.

`hairtos` dùng:

```text
bitmap
```

Mỗi bit đại diện cho một priority.

Ví dụ:

```text
priority 0 → empty
priority 1 → có task
priority 2 → empty
priority 3 → có task
```

Bitmap có thể hình dung:

```text
bit 0 = 0
bit 1 = 1
bit 2 = 0
bit 3 = 1
```

Khi queue của một priority từ rỗng thành có task:

```text
set bit tương ứng
```

Khi task cuối cùng của queue đó bị remove:

```text
clear bit
```

Nhờ bitmap, scheduler nhanh chóng biết priority nào đang hoạt động.

---

## 4.7. Scheduler chọn Task tiếp theo thế nào?

Luồng lựa chọn:

```text
Ready Bitmap
     ↓
tìm priority nhỏ nhất có bit = 1
     ↓
Ready Queue của priority đó
     ↓
lấy node đầu FIFO
     ↓
Task được chọn
```

Ví dụ:

```text
P0: empty
P1: Task B
P2: Task C
P3: Task D
```

Scheduler luôn chọn:

```text
Task B
```

vì:

```text
priority 1
```

cao hơn `2` và `3`.

Thứ tự task được tạo không quan trọng nếu chúng có priority khác nhau.

---

## 4.8. Khi nào xảy ra Preemption?

Giả sử task hiện tại:

```text
Task A
effective_priority = 4
RUNNING
```

Task B vừa được đánh thức:

```text
Task B
effective_priority = 2
READY
```

Vì:

```text
2 < 4
```

nên B có priority cao hơn.

Scheduler xác định:

```text
should_preempt = true
```

sau đó yêu cầu:

```text
PendSV
```

để thực hiện context switch.

Ngược lại:

```text
Task A priority = 2
Task B priority = 4
```

B trở thành READY nhưng không được giành CPU ngay.

Task A tiếp tục chạy cho tới khi:

```text
block
yield
hoặc có lý do scheduling khác
```

---

## 4.9. Task cùng Priority

Giả sử:

```text
Task A priority = 3
Task B priority = 3
```

Task B trở thành READY trong khi A đang chạy.

Vì hai task có cùng priority:

```text
không preempt chỉ vì B vừa READY
```

Các task cùng priority dùng FIFO và có thể đổi lượt khi:

```text
yield
hoặc
time slice hết
```

Khi đó ready queue có thể được rotate:

```text
[A][B][C]
    ↓
[B][C][A]
```

Chi tiết Round-Robin và Time Slicing sẽ học ở Chương 5.

---

## 4.10. READY, RUNNING và BLOCKED

Ba state quan trọng nhất đối với scheduler:

```text
READY
= có thể chạy

RUNNING
= đang dùng CPU

BLOCKED
= chưa thể chạy vì đang chờ
```

Ví dụ một task gọi delay:

```text
RUNNING
   ↓
hr_task_delay()
   ↓
BLOCKED
```

Task đó phải rời ready scheduling path.

Khi timeout hết:

```text
BLOCKED
   ↓
READY
   ↓
đưa lại vào Ready Queue
```

Scheduler sau đó kiểm tra xem task vừa thức có cần preempt task hiện tại không.

Các sự kiện có thể làm task trở lại READY gồm:

```text
timeout hết
queue có dữ liệu
semaphore được give
mutex được unlock / handoff
ISR đánh thức task
```

---

## 4.11. `effective_priority` và Scheduler

Scheduler không chỉ nhìn:

```text
base_priority
```

mà dùng:

```text
effective_priority
```

Điều này cần cho Priority Inheritance.

Ví dụ:

```text
base_priority      = 5
effective_priority = 2
```

Scheduler phải coi task này như priority `2`.

Nếu một task đang READY mà `effective_priority` thay đổi:

```text
remove khỏi Ready Queue cũ
        ↓
cập nhật priority
        ↓
insert vào Ready Queue mới
```

Nếu không làm vậy, bitmap và queue sẽ không phản ánh đúng priority hiện tại.

---

## 4.12. Scheduler và PendSV khác nhau

Đây là điểm rất hay bị nhầm.

Scheduler:

```text
Ready Set
   ↓
chọn Task B
   ↓
Current TCB = Task B
```

PendSV:

```text
Save Context Task A
   ↓
gọi Scheduler
   ↓
Load Context Task B
```

Có thể nhớ:

```text
Scheduler
= policy

PendSV
= mechanism
```

Hay nói cách khác:

> Scheduler quyết định **ai được chạy**, PendSV quyết định **CPU được chuyển sang task đó như thế nào**.

---

## 4.13. Ví dụ cụ thể

Giả sử ready set:

```text
Priority 0:
empty

Priority 1:
Task B

Priority 2:
Task C → Task D

Priority 3:
Task A
```

Task A hiện đang chạy.

Một sự kiện làm Task B READY.

Bitmap:

```text
P0 P1 P2 P3
 0  1  1  1
```

Scheduler:

```text
tìm priority cao nhất
        ↓
priority 1
        ↓
Task B
```

Vì:

```text
Task B effective_priority = 1
Task A effective_priority = 3
```

nên:

```text
1 < 3
```

Kết quả:

```text
Task B READY
    ↓
Preemption required
    ↓
PendSV pending
    ↓
Save Task A
    ↓
Scheduler chọn Task B
    ↓
Restore Task B
    ↓
Task B RUNNING
```

Nếu sau đó Task B block:

```text
Task B
   ↓
remove khỏi Ready Queue
   ↓
bitmap cập nhật nếu queue P1 rỗng
   ↓
Scheduler chọn priority cao nhất còn lại
   ↓
Task C
```

---

## 4.14. Mô hình tư duy cần nhớ

```text
TASK READY
    ↓
Ready Queue theo Priority
    ↓
Ready Bitmap
    ↓
Highest Priority
    ↓
FIFO Head
    ↓
Next Task
```

Quy tắc priority:

```text
Số nhỏ hơn
=
Priority cao hơn
```

Preemption:

```text
New READY Task
      ↓
priority cao hơn Current?
   /            \
 YES            NO
 ↓               ↓
PendSV         tiếp tục
 ↓
Context Switch
```

Phân chia trách nhiệm:

```text
Scheduler
= chọn task

Ready Queue
= giữ thứ tự task runnable

Bitmap
= cho biết priority nào đang có task

PendSV
= chuyển CPU sang task được chọn
```

---

## 4.15. Trọng tâm cần thuộc

Các giá trị chính:

```text
Priority Count = 8

Priority 0
= cao nhất

Priority 7
= thấp nhất / Idle
```

Các cấu trúc chính:

```text
Ready Set
=
FIFO Queue cho từng priority
+
Bitmap
+
Task Count
```

Các function khái niệm quan trọng:

```text
add_ready
remove_ready
select_highest
should_preempt
yield_current
rotate_highest
```

Các nguyên tắc phải giải thích được:

- Scheduler của `hairtos` là fixed-priority và preemptive.
- Số priority nhỏ hơn nghĩa là priority cao hơn.
- Mỗi priority có một FIFO ready queue.
- Bitmap đánh dấu priority nào có task runnable.
- Scheduler chọn priority cao nhất rồi lấy task đầu FIFO.
- Task priority cao hơn có thể preempt task thấp hơn.
- Task cùng priority không preempt nhau chỉ vì vừa READY.
- `yield` hoặc time slicing mới rotate các task cùng priority.
- Scheduler dùng `effective_priority`, không chỉ `base_priority`.
- Task BLOCKED không được scheduler chọn.
- Task được wake phải quay lại ready set trước khi có thể chạy.
- Scheduler chỉ đưa ra quyết định; `PendSV` mới thực hiện context switch.

Câu cần nhớ nhất:

> **`hairtos` không tìm task tiếp theo bằng cách quét toàn bộ task. Nó tổ chức task theo từng FIFO ready queue của mỗi priority và dùng bitmap để biết priority cao nhất đang có task.**

> **Preemption xảy ra khi một task READY có `effective_priority` cao hơn task hiện tại; việc chuyển CPU thực tế sau đó được thực hiện bằng `PendSV`.**


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
