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
  - [5.1. Preemption là gì?](#51-preemption-là-gì)
  - [5.2. Priority Preemption trong `hairtos`](#52-priority-preemption-trong-hairtos)
  - [5.3. Round-Robin là gì?](#53-round-robin-là-gì)
  - [5.4. Time Slice](#54-time-slice)
  - [5.5. `time_slice_remaining`](#55-time_slice_remaining)
  - [5.6. SysTick xử lý Time Slicing thế nào?](#56-systick-xử-lý-time-slicing-thế-nào)
  - [5.7. Ready Queue được Rotate thế nào?](#57-ready-queue-được-rotate-thế-nào)
  - [5.8. `hr_task_yield()`](#58-hr_task_yield)
  - [5.9. `hr_task_delay(0)`](#59-hr_task_delay0)
  - [5.10. Preemption và Round-Robin khác nhau](#510-preemption-và-round-robin-khác-nhau)
  - [5.11. Khi chỉ có một Task ở Priority hiện tại](#511-khi-chỉ-có-một-task-ở-priority-hiện-tại)
  - [5.12. Khi Task Priority cao hơn thức dậy](#512-khi-task-priority-cao-hơn-thức-dậy)
  - [5.13. Ví dụ cụ thể](#513-ví-dụ-cụ-thể)
  - [5.14. Mô hình tư duy cần nhớ](#514-mô-hình-tư-duy-cần-nhớ)
  - [5.15. Trọng tâm cần thuộc](#515-trọng-tâm-cần-thuộc)
- [6. SysTick + Delay + Timeout](#6-systick-delay-timeout)
  - [6.1. Kernel Tick là gì?](#61-kernel-tick-là-gì)
  - [6.2. SysTick 1 kHz trong `hairtos`](#62-systick-1-khz-trong-hairtos)
  - [6.3. Tick Counter 32-bit](#63-tick-counter-32-bit)
  - [6.4. `hr_time_now()`](#64-hr_time_now)
  - [6.5. `hr_task_delay()`](#65-hr_task_delay)
  - [6.6. Delay không phải Busy-Wait](#66-delay-không-phải-busy-wait)
  - [6.7. `hr_task_delay_until()`](#67-hr_task_delay_until)
  - [6.8. Timeout List](#68-timeout-list)
  - [6.9. Vì sao cần Current List và Overflow List?](#69-vì-sao-cần-current-list-và-overflow-list)
  - [6.10. Chèn một Timeout](#610-chèn-một-timeout)
  - [6.11. SysTick đánh thức Task như thế nào?](#611-systick-đánh-thức-task-như-thế-nào)
  - [6.12. Timeout của IPC](#612-timeout-của-ipc)
  - [6.13. Race giữa Event Wakeup và Timeout](#613-race-giữa-event-wakeup-và-timeout)
  - [6.14. Timeout và Preemption](#614-timeout-và-preemption)
  - [6.15. Ví dụ Tick Wrap-Around](#615-ví-dụ-tick-wrap-around)
  - [6.16. Mô hình tư duy cần nhớ](#616-mô-hình-tư-duy-cần-nhớ)
  - [6.17. Trọng tâm cần thuộc](#617-trọng-tâm-cần-thuộc)
- [7. Queue + Blocking IPC](#7-queue-blocking-ipc)
  - [7.1. Queue dùng để làm gì?](#71-queue-dùng-để-làm-gì)
  - [7.2. Queue trong `hairtos`](#72-queue-trong-hairtos)
  - [7.3. Static Queue Storage](#73-static-queue-storage)
  - [7.4. Send khi Queue còn chỗ](#74-send-khi-queue-còn-chỗ)
  - [7.5. Receive khi Queue có dữ liệu](#75-receive-khi-queue-có-dữ-liệu)
  - [7.6. Send khi Queue đầy](#76-send-khi-queue-đầy)
  - [7.7. Receive khi Queue rỗng](#77-receive-khi-queue-rỗng)
  - [7.8. Wait List của Sender và Receiver](#78-wait-list-của-sender-và-receiver)
  - [7.9. Direct Handoff](#79-direct-handoff)
  - [7.10. Timeout khi chờ Queue](#710-timeout-khi-chờ-queue)
  - [7.11. `HR_NO_WAIT` và `HR_WAIT_FOREVER`](#711-hr_no_wait-và-hr_wait_forever)
  - [7.12. Queue API từ ISR](#712-queue-api-từ-isr)
  - [7.13. Khi Queue đánh thức Task Priority cao hơn](#713-khi-queue-đánh-thức-task-priority-cao-hơn)
  - [7.14. Queue và Critical Section](#714-queue-và-critical-section)
  - [7.15. Ví dụ Producer → Consumer](#715-ví-dụ-producer--consumer)
  - [7.16. Mô hình tư duy cần nhớ](#716-mô-hình-tư-duy-cần-nhớ)
  - [7.17. Trọng tâm cần thuộc](#717-trọng-tâm-cần-thuộc)
- [8. Semaphore + ISR Synchronization](#8-semaphore-isr-synchronization)
- [9. Mutex + Priority Inversion + Priority Inheritance](#9-mutex-priority-inversion-priority-inheritance)
- [10. Suspend / Resume + Software Timer](#10-suspend-resume-software-timer)
- [11. haievent + Event Ownership + Reference Counting](#11-haievent-event-ownership-reference-counting)
- [12. Active Object + FSM + Publish / Subscribe](#12-active-object-fsm-publish-subscribe)
- [13. Diagnostics + Fault + Stack Safety](#13-diagnostics-fault-stack-safety)
- [14. Architecture Port + Testing + Benchmark](#14-architecture-port-testing-benchmark)

---

# 1. Cortex-M3 RTOS Foundation

`hairtos` là RTOS preemptive chạy trên ARM Cortex-M. Target hoàn chỉnh hiện tại là STM32F103C8T6 / Cortex-M3, với SysTick mặc định 1 kHz và context start/switch dùng `SVC` / `PendSV`.

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

## 1.1. Cortex-M3 cung cấp gì cho RTOS?

Cortex-M3 cung cấp:

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

RTOS tận dụng exception để CPU tự lưu một phần context, còn kernel lưu phần còn lại.

## 1.2. Thread Mode và Handler Mode

```text
Thread Mode
= code bình thường / task

Handler Mode
= exception / interrupt handler
```

Task chạy trong Thread Mode; `SVC_Handler`, `PendSV_Handler`, `SysTick_Handler` chạy trong Handler Mode.

## 1.3. MSP và PSP

```text
MSP = Main Stack Pointer
PSP = Process Stack Pointer
```

Sau reset CPU dùng MSP. Trong `hairtos`, task chạy bằng PSP, còn exception handler dùng MSP.

```text
Kernel startup / Handler → MSP
Task A/B/C              → PSP
```

## 1.4. Exception là nền tảng của context switch

```text
Task đang chạy
  ↓
Exception
  ↓
Save context
  ↓
Select task mới
  ↓
Restore context
  ↓
Exception Return
```

## 1.5. Hardware Exception Stack Frame

Cortex-M3 tự lưu:

```text
R0-R3
R12
LR
PC
xPSR
```

`R4-R11` phải được phần mềm lưu/khôi phục.

## 1.6. SysTick

SysTick tạo kernel tick:

```text
1 kHz
→ 1 tick ≈ 1 ms
```

Nó phục vụ delay, timeout, time slicing và software timer.

## 1.7. SVC

SVC được dùng để bắt đầu task đầu tiên:

```text
main / MSP
  ↓
SVC
  ↓
restore task
  ↓
PSP
  ↓
Task đầu tiên
```

## 1.8. PendSV

PendSV thực hiện context switch:

```text
Save R4-R11
↓
Save PSP vào TCB
↓
Select Task
↓
Load PSP
↓
Restore R4-R11
↓
Exception Return
```

## 1.9. NVIC Priority

PendSV được đặt priority thấp để các interrupt quan trọng xử lý xong trước khi context switch.

## 1.10. Critical Section và PRIMASK

Cortex-M3 port hiện dùng `PRIMASK` để bảo vệ các cập nhật kernel state ngắn:

```text
Enter Critical
↓
mask interrupt
↓
update kernel state
↓
Exit Critical
```

Không được block trong critical section.

## 1.11. Bare-metal và RTOS khác nhau ở đâu?

Bare-metal thường có một luồng điều khiển chính. Trong RTOS:

```text
Task A delay
→ BLOCKED
→ Scheduler chạy Task B
```

CPU không phải đứng chờ.

## 1.12. Luồng Cortex-M3 khi `hairtos` chạy

```text
Reset
↓
main / MSP
↓
Kernel init
↓
Create tasks
↓
SVC
↓
Task đầu tiên / PSP
↓
SysTick / IRQ / Yield
↓
PendSV nếu cần
↓
Save → Select → Restore
```

## 1.13. Mô hình tư duy cần nhớ

```text
Thread Mode = task
Handler Mode = exception
MSP = startup + handler
PSP = task
SysTick = kernel time
SVC = start first task
PendSV = context switch
PRIMASK = critical section
```

## 1.14. Trọng tâm cần thuộc

- Sau reset CPU bắt đầu với MSP.
- Task dùng PSP.
- Handler dùng MSP.
- Hardware lưu `R0-R3, R12, LR, PC, xPSR`.
- PendSV lưu `R4-R11`.
- SVC bắt đầu task đầu tiên.
- PendSV xử lý context switch.
- Cortex-M3 target không có FPU context.

---

# 2. Task + TCB + Stack

Trong RTOS:

```text
Task
=
Task Function
+
TCB
+
Stack riêng
```

## 2.1. Task là gì?

Task là một luồng thực thi độc lập. Ngoài function, task còn cần stack riêng, priority, state và CPU context.

## 2.2. TCB là gì?

TCB là `Task Control Block`, tức hồ sơ quản lý task:

```text
TCB
├── saved stack pointer
├── priority
├── state
└── scheduler information
```

## 2.3. Vì sao TCB cần lưu Stack Pointer?

```text
Task A
↓
Save context vào Stack A
↓
Save PSP vào TCB A
↓
Select Task B
↓
Load PSP từ TCB B
↓
Restore Task B
```

Saved PSP là điểm để kernel tìm lại context của task.

## 2.4. Mỗi Task có Stack riêng

Stack chứa local variable, return address, function frame và saved CPU context.

## 2.5. `stack_words` nghĩa là gì?

Trong `hairtos`:

```text
hr_stack_t = uint32_t
```

nên:

```text
256 stack words = 1024 byte
```

## 2.6. Task được tạo như thế nào?

```text
Task Function
+
Argument
+
Stack
+
Priority
↓
hr_task_create_static()
↓
Initialize TCB
↓
Build Initial Stack Frame
```

## 2.7. Initial Stack Frame là gì?

Task mới chưa từng chạy, nên kernel dựng sẵn một context giống như task vừa bị exception.

## 2.8. Initial Stack Frame trong `hairtos`

```text
R4-R11
+
R0-R3
R12
LR
PC
xPSR
```

`R4-R11` là software context; phần còn lại là hardware-style frame.

## 2.9. Vì sao `R0` chứa Task Argument?

ARM calling convention truyền argument đầu tiên trong `R0`:

```text
R0 = argument
PC = task entry
```

## 2.10. `PC`, `LR` và `xPSR`

```text
PC = task entry
LR = task exit handler
xPSR = Thumb state
```

## 2.11. Task State

```text
CREATED
READY
RUNNING
BLOCKED
SUSPENDED
```

## 2.12. Base Priority và Effective Priority

Kernel tách:

```text
base_priority
effective_priority
```

để hỗ trợ Priority Inheritance.

## 2.13. Static Task Allocation

Task dùng caller-owned TCB storage và caller-owned stack, giúp bộ nhớ xác định trước và tránh fragmentation runtime.

## 2.14. Quan hệ Task – TCB – Stack

```text
Task Function
   +
TCB ---- saved PSP ----+
                       |
                       ↓
                    Stack
                 CPU Context
```

## 2.15. Mô hình tư duy cần nhớ

```text
TASK = code
TCB = hồ sơ quản lý
STACK = trạng thái thực thi
```

## 2.16. Trọng tâm cần thuộc

> **TCB không chứa toàn bộ register; phần lớn context nằm trên stack của task. TCB giữ saved PSP để tìm lại stack đó.**

---

# 3. SVC + PendSV Context Switch

```text
SVC = bắt đầu task đầu tiên
PendSV = chuyển giữa các task
```

## 3.1. Vì sao Cortex-M dùng SVC và PendSV?

Exception mechanism cho phép kernel save/restore toàn bộ trạng thái thực thi thay vì chỉ gọi function.

## 3.2. `hr_port_start_first_task()` làm gì?

```text
MSP = _estack
↓
DSB / ISB
↓
svc #0
↓
SVC_Handler
```

## 3.3. `SVC_Handler` khởi động task đầu tiên

SVC lấy current TCB, lấy saved PSP, restore `R4-R11`, đặt PSP và trả về Thread Mode dùng PSP.

## 3.4. `EXC_RETURN = 0xFFFFFFFD`

`0xFFFFFFFD` yêu cầu Cortex-M:

```text
return về Thread Mode
+
dùng PSP
+
restore standard exception frame
```

## 3.5. Khi nào yêu cầu Context Switch?

Kernel set `PENDSVSET` trong `SCB->ICSR`.

## 3.6. `PendSV_Handler` lưu Task hiện tại

Hardware đã lưu:

```text
R0-R3, R12, LR, PC, xPSR
```

PendSV lưu thêm:

```text
R4-R11
```

và ghi PSP vào TCB.

## 3.7. Vì sao `stack_pointer` phải ở offset 0 của TCB?

Assembly có thể truy cập nhanh:

```text
TCB + 0
→ saved PSP
```

## 3.8. Kernel chọn Task tiếp theo

PendSV gọi:

```text
hr_kernel_select_next_from_pendsv()
```

Scheduler cập nhật current TCB.

## 3.9. Khôi phục Task mới

```text
Load PSP
↓
Restore R4-R11
↓
Exception Return
↓
Hardware restore phần còn lại
```

## 3.10. Vì sao PendSV có Priority thấp nhất?

Để context switch xảy ra sau khi các interrupt quan trọng hơn đã xử lý xong.

## 3.11. SysTick / ISR và PendSV phối hợp thế nào?

ISR chỉ làm task READY và request PendSV nếu cần; PendSV mới thực hiện switch.

## 3.12. Ví dụ Task A → Task B

```text
Task A RUNNING
↓
Task B priority cao hơn READY
↓
PendSV
↓
Save A
↓
Select B
↓
Restore B
↓
B RUNNING
```

## 3.13. Mô hình tư duy cần nhớ

```text
Task A
↓
Hardware Save
↓
PendSV Save R4-R11
↓
Save PSP → TCB A
↓
Select B
↓
Load PSP ← TCB B
↓
Restore
↓
Task B
```

## 3.14. Trọng tâm cần thuộc

> **Context nằm trên stack từng task; TCB giữ saved PSP. SVC bắt đầu task đầu tiên, PendSV thực hiện các context switch sau đó.**

---

# 4. Scheduler + Ready Queue + Bitmap

Scheduler trả lời:

> **Task READY nào phải được CPU chạy tiếp?**

## 4.1. Scheduler có nhiệm vụ gì?

Scheduler chọn task; PendSV thực hiện context switch.

## 4.2. Fixed-Priority Preemptive Scheduling

Task priority cao hơn có thể preempt task thấp hơn.

## 4.3. Priority trong `hairtos`

```text
8 priority levels
0 = cao nhất
7 = thấp nhất / Idle
```

## 4.4. Ready Set

```text
Ready Set
├── queues[0..7]
├── bitmap
└── count
```

## 4.5. Ready Queue theo từng Priority

Mỗi priority có một FIFO queue riêng.

## 4.6. Ready Bitmap

Bitmap cho biết priority nào đang có task READY.

## 4.7. Scheduler chọn Task tiếp theo thế nào?

```text
Bitmap
↓
priority nhỏ nhất có bit = 1
↓
FIFO head
↓
Next Task
```

## 4.8. Khi nào xảy ra Preemption?

Nếu task mới READY có:

```text
effective_priority < current effective_priority
```

thì request PendSV.

## 4.9. Task cùng Priority

Task cùng priority không preempt nhau chỉ vì vừa READY; chúng đổi lượt qua yield hoặc time slicing.

## 4.10. READY, RUNNING và BLOCKED

```text
READY = có thể chạy
RUNNING = đang dùng CPU
BLOCKED = đang chờ
```

## 4.11. `effective_priority` và Scheduler

Scheduler dùng `effective_priority`, không chỉ `base_priority`.

## 4.12. Scheduler và PendSV khác nhau

```text
Scheduler = policy
PendSV = mechanism
```

## 4.13. Ví dụ cụ thể

Task priority 1 READY sẽ được chọn trước task priority 3.

## 4.14. Mô hình tư duy cần nhớ

```text
READY
↓
Queue theo Priority
↓
Bitmap
↓
Highest Priority
↓
FIFO Head
```

## 4.15. Trọng tâm cần thuộc

> **`hairtos` không quét toàn bộ task; nó dùng FIFO ready queue theo priority và bitmap để chọn nhanh task có priority cao nhất.**

---

# 5. Preemption + Round-Robin

```text
Priority Preemption
+
Round-Robin cùng Priority
```

## 5.1. Preemption là gì?

Task priority cao hơn có thể giành CPU từ task thấp hơn.

## 5.2. Priority Preemption trong `hairtos`

```text
HR_CFG_PREEMPTION = 1
```

## 5.3. Round-Robin là gì?

Các task cùng priority chia CPU bằng cách rotate FIFO:

```text
[A][B][C]
→
[B][C][A]
```

## 5.4. Time Slice

```text
HR_CFG_TIME_SLICING = 1
HR_CFG_TIME_SLICE_TICKS = 1
```

Với SysTick 1 kHz:

```text
1 slice ≈ 1 ms
```

## 5.5. `time_slice_remaining`

TCB giữ số tick còn lại của lượt chạy.

## 5.6. SysTick xử lý Time Slicing thế nào?

Chỉ giảm time slice khi có peer cùng priority. Nếu hết slice, mark `TIME_SLICE` và request PendSV.

## 5.7. Ready Queue được Rotate thế nào?

```text
[A][B][C]
→
[B][C][A]
```

## 5.8. `hr_task_yield()`

Task chủ động nhường CPU, scheduler rotate queue cùng priority.

## 5.9. `hr_task_delay(0)`

Tương đương `yield`.

## 5.10. Preemption và Round-Robin khác nhau

```text
Preemption = giữa các priority
Round-Robin = trong cùng priority
```

## 5.11. Khi chỉ có một Task ở Priority hiện tại

Kernel không context switch vô ích sang chính task đó.

## 5.12. Khi Task Priority cao hơn thức dậy

Priority preemption được ưu tiên trước time slicing.

## 5.13. Ví dụ cụ thể

Nếu A/B cùng priority 2 và H priority 0 trở thành READY, H preempt ngay dù slice của A/B chưa hết.

## 5.14. Mô hình tư duy cần nhớ

```text
Task priority cao hơn READY?
YES → Preemption
NO → peer cùng priority?
       YES → Time Slice / Yield
       NO → tiếp tục task hiện tại
```

## 5.15. Trọng tâm cần thuộc

> **Preemption quyết định priority nào chạy; Round-Robin quyết định task nào trong cùng priority chạy tiếp.**

---

# 6. SysTick + Delay + Timeout

SysTick tạo nhịp thời gian cho kernel.

```text
SysTick
↓
Tick++
↓
Timeout
↓
BLOCKED → READY
↓
Scheduler
↓
PendSV nếu cần
```

## 6.1. Kernel Tick là gì?

Tick là đơn vị thời gian logic của RTOS.

## 6.2. SysTick 1 kHz trong `hairtos`

```text
1 tick ≈ 1 ms
```

## 6.3. Tick Counter 32-bit

Tick dùng `uint32_t` và sẽ wrap.

Ở 1 kHz:

```text
wrap ≈ 49.7 ngày
```

## 6.4. `hr_time_now()`

Trả về kernel tick hiện tại.

## 6.5. `hr_task_delay()`

```text
RUNNING
↓
remove Ready
↓
insert Timeout
↓
BLOCKED
↓
PendSV
```

## 6.6. Delay không phải Busy-Wait

Task BLOCKED không giữ CPU; scheduler chạy task khác.

## 6.7. `hr_task_delay_until()`

Dùng mốc tuyệt đối:

```text
next_wake = last_wake + period
```

giúp giảm phase drift cho task tuần hoàn.

## 6.8. Timeout List

Task có `timeout_node`. Kernel dùng hai sorted list:

```text
current
overflow
```

## 6.9. Vì sao cần Current List và Overflow List?

Deadline trước lần wrap kế tiếp nằm ở current; deadline sau wrap nằm ở overflow.

Khi tick wrap:

```text
current ↔ overflow
```

## 6.10. Chèn một Timeout

```text
wake_tick = now + timeout
```

rồi chèn vào current hoặc overflow tùy có wrap hay không.

## 6.11. SysTick đánh thức Task như thế nào?

```text
Tick++
↓
Expire due nodes
↓
BLOCKED → READY
↓
add Ready Set
```

## 6.12. Timeout của IPC

Task chờ queue/semaphore/mutex có thể nằm đồng thời trong:

```text
Object Wait List
+
Timeout List
```

## 6.13. Race giữa Event Wakeup và Timeout

Chỉ một wake path thắng. Path thắng phải cleanup wait/timeout metadata còn lại.

## 6.14. Timeout và Preemption

Task priority cao hơn vừa hết timeout có thể preempt task hiện tại qua PendSV.

## 6.15. Ví dụ Tick Wrap-Around

```text
now = 0xFFFFFFFC

Task A +2 → 0xFFFFFFFE → Current
Task B +6 → 0x00000002 → Overflow
```

Sau wrap, swap list rồi B hết hạn đúng lúc.

## 6.16. Mô hình tư duy cần nhớ

```text
RUNNING
↓
BLOCKED + Timeout
↓
SysTick
↓
deadline hết
↓
READY
↓
Scheduler
```

## 6.17. Trọng tâm cần thuộc

> **`hairtos` xử lý tick wrap bằng hai timeout list `current` và `overflow`, rồi swap khi tick wrap.**

---

# 7. Queue + Blocking IPC

Queue là cơ chế IPC dùng để truyền dữ liệu giữa các task hoặc giữa ISR và task.

Trong `hairtos`, queue là một FIFO có kích thước cố định:

```text
Producer
   ↓
 Queue
   ↓
Consumer
```

Nếu queue đầy, sender có thể block. Nếu queue rỗng, receiver có thể block. Khi điều kiện thay đổi, kernel đánh thức task đang chờ và đưa nó trở lại `READY`.

## 7.1. Queue dùng để làm gì?

Queue giải quyết hai việc:

```text
truyền dữ liệu
+
đồng bộ task
```

Sender không cần biết receiver đang chạy ở thời điểm nào.

## 7.2. Queue trong `hairtos`

API chính:

```c
hr_queue_create_static()

hr_queue_send()
hr_queue_receive()

hr_queue_send_from_isr()
hr_queue_receive_from_isr()
```

Queue là bounded FIFO.

## 7.3. Static Queue Storage

Application cung cấp:

```text
queue object
+
storage
+
item_size
+
capacity
```

Storage cần:

```text
item_size × capacity
```

và phải sống lâu bằng queue.

## 7.4. Send khi Queue còn chỗ

```text
Sender
↓
copy item vào FIFO
↓
count++
↓
HR_OK
```

Queue copy dữ liệu, không chỉ giữ con trỏ.

## 7.5. Receive khi Queue có dữ liệu

```text
Receiver
↓
copy item đầu FIFO ra ngoài
↓
count--
↓
HR_OK
```

Queue giữ thứ tự FIFO.

## 7.6. Send khi Queue đầy

Nếu queue đầy và cho phép block:

```text
Sender RUNNING
↓
remove Ready Set
↓
Sender Wait List
↓
optional Timeout List
↓
BLOCKED
↓
PendSV
```

## 7.7. Receive khi Queue rỗng

Nếu queue rỗng và cho phép block:

```text
Receiver RUNNING
↓
remove Ready Set
↓
Receiver Wait List
↓
optional Timeout List
↓
BLOCKED
↓
PendSV
```

## 7.8. Wait List của Sender và Receiver

Queue có hai wait list:

```text
Sender Wait List
Receiver Wait List
```

Waiter được xếp theo `effective_priority`, FIFO trong cùng priority.

## 7.9. Direct Handoff

Nếu receiver đang chờ:

```text
Sender
↓
copy trực tiếp item vào buffer Receiver
↓
wake Receiver
```

không cần enqueue rồi dequeue.

Receive cũng có thể lấy trực tiếp từ blocked sender khi phù hợp.

## 7.10. Timeout khi chờ Queue

Task có thể đồng thời nằm trong:

```text
Object Wait List
+
Timeout List
```

Data đến trước → `HR_OK`; timeout đến trước → `HR_ERROR_TIMEOUT`.

Chỉ một wake path được thắng.

## 7.11. `HR_NO_WAIT` và `HR_WAIT_FOREVER`

```text
HR_NO_WAIT
= không block, trả ngay

HR_WAIT_FOREVER
= chờ vô hạn, không cần timeout node
```

## 7.12. Queue API từ ISR

ISR chỉ dùng:

```c
hr_queue_send_from_isr()
hr_queue_receive_from_isr()
```

Các API này luôn non-blocking.

Nếu đánh thức task priority cao hơn:

```text
higher_priority_task_woken = true
```

Context switch vẫn được hoãn tới PendSV.

## 7.13. Khi Queue đánh thức Task Priority cao hơn

```text
Task B BLOCKED
↓
Queue event
↓
Task B READY
↓
priority cao hơn current?
↓
PendSV
↓
Task B RUNNING
```

## 7.14. Queue và Critical Section

Queue state như `count`, head/tail và wait list được bảo vệ trong critical section ngắn bằng `PRIMASK`.

Không được block trong critical section.

## 7.15. Ví dụ Producer → Consumer

```text
Consumer Receive
↓
Queue Empty
↓
BLOCKED

Producer Send 42
↓
Receiver đang chờ
↓
Direct Handoff
↓
Consumer READY
↓
Preempt nếu priority cao hơn
↓
Consumer nhận 42
```

## 7.16. Mô hình tư duy cần nhớ

Send:

```text
Receiver đang chờ?
YES → Direct Handoff
NO  → Queue còn chỗ?
      YES → Enqueue
      NO  → NO_WAIT hoặc BLOCK
```

Receive:

```text
Queue có item?
YES → Dequeue
NO  → Sender đang chờ?
      YES → Direct Handoff
      NO  → NO_WAIT hoặc BLOCK
```

## 7.17. Trọng tâm cần thuộc

```text
Queue
=
bounded FIFO
+
caller-owned storage
+
blocking send/receive
+
timeout
+
priority-ordered waiters
+
direct handoff
+
ISR-safe non-blocking API
```

Các điểm cần nhớ:

- Queue storage có kích thước `item_size × capacity`.
- Queue copy item vào storage.
- Queue đầy → sender có thể `BLOCKED`.
- Queue rỗng → receiver có thể `BLOCKED`.
- Sender và receiver có wait list riêng.
- Waiter ưu tiên theo `effective_priority`, FIFO trong cùng priority.
- `HR_NO_WAIT` không block.
- `HR_WAIT_FOREVER` chờ vô hạn.
- Direct handoff có thể truyền thẳng giữa sender và receiver đang chờ.
- Task API chỉ block khi kernel đang RUNNING và không ở ISR context.
- ISR API luôn non-blocking.
- `higher_priority_task_woken` chỉ báo cần PendSV sau ISR.
- Wait buffer phải còn hợp lệ trong suốt thời gian task bị block.

> **Queue không chỉ là ring buffer. Trong RTOS, nó còn quản lý blocking, timeout, wait list và đánh thức task theo priority.**

> **`hairtos` dùng Direct Handoff để dữ liệu có thể đi thẳng giữa sender và receiver đang chờ mà không cần vòng qua FIFO storage.**

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
