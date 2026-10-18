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
