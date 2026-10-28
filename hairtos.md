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
  - [8.1. Semaphore là gì?](#81-semaphore-là-gì)
  - [8.2. Binary Semaphore và Counting Semaphore](#82-binary-semaphore-và-counting-semaphore)
  - [8.3. Semaphore không có Ownership](#83-semaphore-không-có-ownership)
  - [8.4. `hr_semaphore_take()`](#84-hr_semaphore_take)
  - [8.5. `hr_semaphore_give()`](#85-hr_semaphore_give)
  - [8.6. Vì sao `give()` ưu tiên Waiter?](#86-vì-sao-give-ưu-tiên-waiter)
  - [8.7. Blocking và Timeout](#87-blocking-và-timeout)
  - [8.8. `HR_NO_WAIT` và `HR_WAIT_FOREVER`](#88-hr_no_wait-và-hr_wait_forever)
  - [8.9. `hr_semaphore_give_from_isr()`](#89-hr_semaphore_give_from_isr)
  - [8.10. `higher_priority_task_woken`](#810-higher_priority_task_woken)
  - [8.11. ISR không được Block](#811-isr-không-được-block)
  - [8.12. Pattern ISR → Semaphore → Task](#812-pattern-isr--semaphore--task)
  - [8.13. Binary Semaphore có thể mất Event như thế nào?](#813-binary-semaphore-có-thể-mất-event-như-thế-nào)
  - [8.14. Counting Semaphore dùng khi nào?](#814-counting-semaphore-dùng-khi-nào)
  - [8.15. Semaphore và Mutex khác nhau](#815-semaphore-và-mutex-khác-nhau)
  - [8.16. Ví dụ từ `hairtos`](#816-ví-dụ-từ-hairtos)
  - [8.17. Mô hình tư duy cần nhớ](#817-mô-hình-tư-duy-cần-nhớ)
  - [8.18. Trọng tâm cần thuộc](#818-trọng-tâm-cần-thuộc)
- [9. Mutex + Priority Inversion + Priority Inheritance](#9-mutex-priority-inversion-priority-inheritance)
  - [9.1. Mutex là gì?](#91-mutex-là-gì)
  - [9.2. Mutex có Ownership](#92-mutex-có-ownership)
  - [9.3. Normal Mutex và Recursive Mutex](#93-normal-mutex-và-recursive-mutex)
  - [9.4. `hr_mutex_lock()`](#94-hr_mutex_lock)
  - [9.5. `hr_mutex_unlock()`](#95-hr_mutex_unlock)
  - [9.6. Priority Inversion là gì?](#96-priority-inversion-là-gì)
  - [9.7. Priority Inheritance giải quyết thế nào?](#97-priority-inheritance-giải-quyết-thế-nào)
  - [9.8. `base_priority` và `effective_priority`](#98-base_priority-và-effective_priority)
  - [9.9. Owner READY phải được Requeue](#99-owner-ready-phải-được-requeue)
  - [9.10. Một Task giữ nhiều Mutex](#910-một-task-giữ-nhiều-mutex)
  - [9.11. Chained Priority Inheritance](#911-chained-priority-inheritance)
  - [9.12. Direct Ownership Handoff khi Unlock](#912-direct-ownership-handoff-khi-unlock)
  - [9.13. Timeout khi chờ Mutex](#913-timeout-khi-chờ-mutex)
  - [9.14. Mutex không dùng trong ISR](#914-mutex-không-dùng-trong-isr)
  - [9.15. Semaphore và Mutex khác nhau](#915-semaphore-và-mutex-khác-nhau)
  - [9.16. Ví dụ đầy đủ L / M / H](#916-ví-dụ-đầy-đủ-l--m--h)
  - [9.17. Mô hình tư duy cần nhớ](#917-mô-hình-tư-duy-cần-nhớ)
  - [9.18. Trọng tâm cần thuộc](#918-trọng-tâm-cần-thuộc)
- [10. Suspend / Resume + Software Timer](#10-suspend-resume-software-timer)
  - [10.1. Suspend khác BLOCKED như thế nào?](#101-suspend-khác-blocked-như-thế-nào)
  - [10.2. `hr_task_suspend()`](#102-hr_task_suspend)
  - [10.3. Suspend một Task đang BLOCKED](#103-suspend-một-task-đang-blocked)
  - [10.4. Deferred Wakeup khi đang SUSPENDED](#104-deferred-wakeup-khi-đang-suspended)
  - [10.5. `hr_task_resume()`](#105-hr_task_resume)
  - [10.6. Self-Suspend](#106-self-suspend)
  - [10.7. Idle Task không được Suspend](#107-idle-task-không-được-suspend)
  - [10.8. Software Timer là gì?](#108-software-timer-là-gì)
  - [10.9. Timer Object trong `hairtos`](#109-timer-object-trong-hairtos)
  - [10.10. Timer-Service Task](#1010-timer-service-task)
  - [10.11. Timer Expiry Flow](#1011-timer-expiry-flow)
  - [10.12. One-Shot và Periodic Timer](#1012-one-shot-và-periodic-timer)
  - [10.13. `start`, `reset`, `change_period`, `stop`](#1013-start-reset-change_period-stop)
  - [10.14. `pending_count`](#1014-pending_count)
  - [10.15. Vì sao Callback không chạy trong SysTick ISR?](#1015-vì-sao-callback-không-chạy-trong-systick-isr)
  - [10.16. Ví dụ Suspend / Resume](#1016-ví-dụ-suspend--resume)
  - [10.17. Ví dụ Software Timer](#1017-ví-dụ-software-timer)
  - [10.18. Mô hình tư duy cần nhớ](#1018-mô-hình-tư-duy-cần-nhớ)
  - [10.19. Trọng tâm cần thuộc](#1019-trọng-tâm-cần-thuộc)
- [11. haievent + Event Ownership + Reference Counting](#11-haievent-event-ownership-reference-counting)
  - [11.1. `haievent` là gì?](#111-haievent-là-gì)
  - [11.2. Event và Signal](#112-event-và-signal)
  - [11.3. `he_event_t` chứa gì?](#113-he_event_t-chứa-gì)
  - [11.4. Static Event](#114-static-event)
  - [11.5. Dynamic Event](#115-dynamic-event)
  - [11.6. Fixed-Block Event Pool](#116-fixed-block-event-pool)
  - [11.7. Reference Counting](#117-reference-counting)
  - [11.8. `he_event_retain()`](#118-he_event_retain)
  - [11.9. `he_event_release()`](#119-he_event_release)
  - [11.10. Ownership khi `he_active_post()`](#1110-ownership-khi-he_active_post)
  - [11.11. `he_active_post_shared()`](#1111-he_active_post_shared)
  - [11.12. Event được release sau Dispatch](#1112-event-được-release-sau-dispatch)
  - [11.13. Publish/Subscribe và nhiều Owner](#1113-publishsubscribe-và-nhiều-owner)
  - [11.14. Ownership khi Post thất bại](#1114-ownership-khi-post-thất-bại)
  - [11.15. Static Event Lifetime](#1115-static-event-lifetime)
  - [11.16. Thread Safety của Event Pool](#1116-thread-safety-của-event-pool)
  - [11.17. Các lỗi Ownership thường gặp](#1117-các-lỗi-ownership-thường-gặp)
  - [11.18. Mô hình tư duy cần nhớ](#1118-mô-hình-tư-duy-cần-nhớ)
  - [11.19. Trọng tâm cần thuộc](#1119-trọng-tâm-cần-thuộc)
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

Semaphore là primitive đồng bộ dùng để biểu diễn số lượng token đang có sẵn.

Trong `hairtos`, semaphore có:

```text
count
max_count
waiter list
```

Có thể hiểu:

```text
count > 0
→ có token để take

count = 0
→ task có thể phải chờ
```

Semaphore đặc biệt hữu ích để đồng bộ giữa ISR và task:

```text
ISR
  ↓
Give Semaphore
  ↓
Task đang chờ được đánh thức
  ↓
Task xử lý phần công việc nặng
```

---

## 8.1. Semaphore là gì?

Semaphore là một counter dùng cho synchronization.

Hai thao tác chính:

```text
Take
= lấy một token

Give
= trả / tạo một token
```

Nếu:

```text
count > 0
```

thì `take()` thành công và:

```text
count--
```

Nếu:

```text
count = 0
```

thì task có thể block tùy timeout.

---

## 8.2. Binary Semaphore và Counting Semaphore

`hairtos` hỗ trợ:

```c
hr_semaphore_create_binary()

hr_semaphore_create_counting()
```

### Binary Semaphore

Binary semaphore có:

```text
max_count = 1
```

nên chỉ có hai trạng thái logic:

```text
0 = chưa có tín hiệu
1 = có tín hiệu
```

API:

```c
hr_semaphore_create_binary(
    semaphore,
    initially_available
);
```

### Counting Semaphore

Counting semaphore có:

```text
0 <= count <= max_count
```

Ví dụ:

```text
initial_count = 0
max_count     = 4
```

có thể biểu diễn tối đa 4 event hoặc 4 resource đang sẵn sàng.

---

## 8.3. Semaphore không có Ownership

Điểm rất quan trọng:

```text
Semaphore
= không có owner
```

Kernel không ghi nhận:

```text
"Task nào đang sở hữu semaphore?"
```

Vì vậy semaphore phù hợp cho:

```text
event signaling
resource counting
ISR → Task synchronization
```

nhưng không phải primitive tốt để bảo vệ shared resource cần ownership.

Semaphore cũng không có:

```text
Priority Inheritance
```

Nếu cần mutual exclusion có ownership và Priority Inheritance:

```text
→ dùng Mutex
```

---

## 8.4. `hr_semaphore_take()`

API:

```c
hr_semaphore_take(
    semaphore,
    timeout
);
```

Nếu:

```text
count > 0
```

luồng đơn giản:

```text
Take
 ↓
count > 0
 ↓
count--
 ↓
HR_OK
```

Nếu:

```text
count = 0
```

thì kết quả phụ thuộc timeout.

---

## 8.5. `hr_semaphore_give()`

`give()` có hai trường hợp chính.

### Có Task đang chờ

```text
Give
 ↓
Waiter tồn tại?
 ↓ YES
đánh thức Waiter
```

Count không được tăng.

### Không có Task đang chờ

```text
Give
 ↓
không có Waiter
 ↓
count < max_count?
 ↓
count++
```

Nếu count đã bằng max:

```text
HR_ERROR_SEMAPHORE_FULL
```

---

## 8.6. Vì sao `give()` ưu tiên Waiter?

Giả sử:

```text
count = 0
Task B đang BLOCKED chờ semaphore
```

Task A gọi:

```text
give()
```

Không nên:

```text
count++
+
wake Task B
```

vì như vậy cùng một token tồn tại hai lần về mặt logic.

`hairtos` làm:

```text
Give
  ↓
Waiter tồn tại
  ↓
Wake Waiter trực tiếp
  ↓
count vẫn không tăng
```

Có thể hiểu đây là:

```text
direct token handoff
```

Nguyên tắc:

> Một token không được vừa nằm trong `count` vừa được trao cho một waiter trong cùng một operation.

---

## 8.7. Blocking và Timeout

Nếu task gọi:

```c
hr_semaphore_take(&sem, timeout);
```

khi:

```text
count = 0
```

và timeout cho phép block:

```text
Task RUNNING
    ↓
Semaphore Wait List
    ↓
nếu timeout hữu hạn:
Timeout List
    ↓
BLOCKED
    ↓
PendSV
```

Task được đánh thức khi:

```text
Semaphore được give
```

hoặc:

```text
Timeout hết
```

---

## 8.8. `HR_NO_WAIT` và `HR_WAIT_FOREVER`

Nếu:

```text
timeout = HR_NO_WAIT
```

và semaphore không có token:

```text
→ HR_ERROR_SEMAPHORE_EMPTY
```

Task không block.

Nếu:

```text
timeout = HR_WAIT_FOREVER
```

task có thể chờ vô hạn:

```text
Semaphore Wait List
```

mà không cần timeout node.

Với finite timeout:

```text
Semaphore Wait List
+
Timeout List
```

---

## 8.9. `hr_semaphore_give_from_isr()`

ISR không được gọi `hr_semaphore_give()` thông thường.

`hairtos` cung cấp:

```c
hr_semaphore_give_from_isr(
    semaphore,
    &higher_priority_task_woken
);
```

API này:

```text
không block
+
có thể đánh thức một waiter
+
trả về thông tin có cần context switch không
```

Nếu gọi `hr_semaphore_give()` từ ISR:

```text
HR_ERROR_FROM_ISR
```

Ngược lại, `give_from_isr()` được thiết kế riêng cho ISR context.

---

## 8.10. `higher_priority_task_woken`

Giả sử:

```text
Task A priority = 4
đang RUNNING

Task B priority = 1
BLOCKED chờ semaphore
```

ISR gọi:

```text
give_from_isr()
```

Task B trở thành READY.

Vì:

```text
1 < 4
```

nên:

```text
higher_priority_task_woken = true
```

Sau đó ISR dùng cơ chế yield-from-ISR của port để request context switch.

Luồng:

```text
ISR
 ↓
give_from_isr()
 ↓
wake Task B
 ↓
higher_priority_task_woken = true
 ↓
request PendSV
 ↓
ISR kết thúc
 ↓
PendSV
 ↓
Task B RUNNING
```

Context switch không diễn ra giữa chừng trong ISR.

---

## 8.11. ISR không được Block

ISR phải chạy ngắn và có thời gian hữu hạn.

Do đó ISR không được:

```text
chờ semaphore
delay
block trên queue
block trên mutex
```

`hr_semaphore_take()` kiểm tra ISR context và từ chối nếu được gọi trong ISR.

Mental model:

```text
ISR
= phát hiện sự kiện + cập nhật tối thiểu + signal task

Task
= xử lý logic dài / phức tạp
```

---

## 8.12. Pattern ISR → Semaphore → Task

Đây là pattern rất quan trọng trong embedded RTOS:

```text
Hardware Event
      ↓
ISR
      ↓
clear interrupt flag
      ↓
Give Semaphore From ISR
      ↓
ISR kết thúc
      ↓
Task được đánh thức
      ↓
xử lý dữ liệu / protocol / control logic
```

Ví dụ:

```text
UART RX interrupt
      ↓
ISR ghi byte / cập nhật buffer
      ↓
Give Semaphore
      ↓
Parser Task thức dậy
      ↓
parse frame
```

Mục tiêu là giữ ISR ngắn và chuyển công việc dài sang task context.

---

## 8.13. Binary Semaphore có thể mất Event như thế nào?

Binary semaphore chỉ chứa tối đa:

```text
count = 1
```

Giả sử hai event xảy ra trước khi task kịp `take()`:

```text
Event 1
→ count = 1

Event 2
→ semaphore đã full
```

Event thứ hai không thể làm count thành `2`.

Do đó binary semaphore phù hợp khi chỉ cần biết:

```text
"đã có ít nhất một event"
```

Nếu cần đếm chính xác nhiều event xảy ra:

```text
→ Counting Semaphore
```

---

## 8.14. Counting Semaphore dùng khi nào?

Counting semaphore phù hợp cho hai kiểu bài toán.

### Đếm Event

```text
ISR Event
 ↓
Give
 ↓
count++
```

Task:

```text
Take
 ↓
count--
 ↓
xử lý một event
```

Count biểu diễn:

```text
số event đã xảy ra nhưng chưa xử lý
```

### Đếm Resource

Giả sử có:

```text
4 buffer
```

thì:

```text
max_count = 4
initial_count = 4
```

Task lấy resource:

```text
Take
→ count--
```

trả resource:

```text
Give
→ count++
```

---

## 8.15. Semaphore và Mutex khác nhau

Hai primitive có vẻ giống nhau vì đều có:

```text
Take / Give
```

nhưng ý nghĩa rất khác.

Semaphore:

```text
không ownership
không Priority Inheritance
có thể dùng để signal từ ISR
phù hợp event / resource count
```

Mutex:

```text
có owner
dùng bảo vệ shared resource
có Priority Inheritance
không dùng như ISR signal
```

Vì vậy:

> Không nên dùng semaphore thay mutex khi mục tiêu là mutual exclusion có ownership.

Mutex sẽ được học ở Chương 9.

---

## 8.16. Ví dụ từ `hairtos`

Repo có example:

```text
10-01-semaphore-from-isr
```

Mô hình:

```text
Waiter Task
priority = 1
BLOCKED trên Binary Semaphore

Trigger Task
priority = 3
      ↓
kích hoạt EXTI0
      ↓
EXTI0 ISR
      ↓
hr_semaphore_give_from_isr()
      ↓
Waiter Task READY
```

Vì waiter priority cao hơn:

```text
higher_priority_task_woken = true
```

ISR request yield:

```text
hr_port_yield_from_isr()
```

Sau khi ISR kết thúc:

```text
PendSV
 ↓
Waiter Task RUNNING
```

Đây là ví dụ trực tiếp về:

```text
ISR → Semaphore → High-Priority Task
```

---

## 8.17. Mô hình tư duy cần nhớ

Take:

```text
TAKE
 ↓
count > 0?
 /       \
YES       NO
↓          ↓
count--   NO_WAIT?
HR_OK      /    \
         YES    NO
         ↓       ↓
      EMPTY    BLOCK
```

Give:

```text
GIVE
 ↓
Waiter?
 /     \
YES     NO
↓        ↓
Wake   count < max?
Waiter   /      \
        YES      NO
        ↓         ↓
      count++    FULL
```

ISR synchronization:

```text
Hardware IRQ
    ↓
ISR
    ↓
give_from_isr()
    ↓
Task BLOCKED → READY
    ↓
higher_priority_task_woken?
    ↓
PendSV after ISR
    ↓
Task RUNNING
```

---

## 8.18. Trọng tâm cần thuộc

API chính:

```text
hr_semaphore_create_binary()

hr_semaphore_create_counting()

hr_semaphore_take()

hr_semaphore_give()

hr_semaphore_give_from_isr()
```

Các invariant chính:

```text
0 <= count <= max_count

max_count > 0
```

Các quy tắc:

- Binary semaphore là counting semaphore với `max_count = 1`.
- `take()` giảm `count` khi có token.
- `take()` có thể block nếu `count = 0`.
- `HR_NO_WAIT` khi không có token trả `HR_ERROR_SEMAPHORE_EMPTY`.
- `give()` ưu tiên đánh thức waiter thay vì tăng count.
- Nếu không có waiter, `give()` tăng count.
- Nếu `count == max_count`, `give()` trả `HR_ERROR_SEMAPHORE_FULL`.
- Semaphore không có owner.
- Semaphore không có Priority Inheritance.
- Dùng mutex nếu cần ownership-aware mutual exclusion.
- ISR không được dùng API blocking.
- ISR dùng `hr_semaphore_give_from_isr()`.
- `higher_priority_task_woken` cho biết có cần request PendSV hay không.
- Context switch chỉ xảy ra sau khi ISR kết thúc.
- Binary semaphore phù hợp signaling; counting semaphore phù hợp đếm event hoặc resource.

Điểm quan trọng nhất:

> **Semaphore là counter đồng bộ không có ownership. `take()` tiêu thụ token; `give()` tạo token hoặc trao trực tiếp quyền tiến lên cho task đang chờ.**

> **Trong ISR, chỉ nên signal bằng API `_from_isr`, sau đó để PendSV chuyển sang task priority cao hơn sau khi ISR hoàn tất.**


---

# 9. Mutex + Priority Inversion + Priority Inheritance

Mutex dùng để bảo vệ shared resource sao cho tại một thời điểm chỉ một task được sở hữu resource đó.

Khác semaphore, mutex có:

```text
owner
+
recursion policy
+
waiter list
+
Priority Inheritance
```

Trong `hairtos`, mỗi task còn theo dõi danh sách các mutex mà nó đang sở hữu để kernel có thể tính đúng `effective_priority`.

---

## 9.1. Mutex là gì?

Mutex là:

```text
Mutual Exclusion
```

Mục tiêu:

```text
Task A
Task B
Task C
   ↓
Shared Resource
```

chỉ cho phép:

```text
1 owner tại một thời điểm
```

Ví dụ shared resource:

```text
SPI peripheral
I2C bus
shared data structure
driver state
```

Luồng cơ bản:

```text
Lock Mutex
   ↓
Critical Resource
   ↓
Unlock Mutex
```

---

## 9.2. Mutex có Ownership

Điểm quan trọng nhất khác semaphore:

```text
Mutex
= có owner
```

Khi Task A lock thành công:

```text
mutex.owner = Task A
```

Chỉ owner mới được unlock.

```text
Task B unlock mutex của Task A
→ không hợp lệ
```

Trong `hairtos`, status enum có:

```text
HR_ERROR_NOT_OWNER
```

để biểu diễn lỗi ownership.

---

## 9.3. Normal Mutex và Recursive Mutex

`hairtos` có hai kiểu:

```c
hr_mutex_create()

hr_mutex_create_recursive()
```

### Normal Mutex

Nếu owner hiện tại lock lại chính mutex đó:

```text
Task A owns M
Task A lock M lần nữa
→ bị từ chối
```

### Recursive Mutex

Recursive mutex cho phép cùng owner lock nhiều lần.

```text
Task A lock
recursion_count = 1

Task A lock lần nữa
recursion_count = 2
```

Unlock:

```text
2 → 1
```

mutex vẫn thuộc Task A.

Chỉ khi:

```text
recursion_count
→ 0
```

ownership mới thực sự được nhả.

---

## 9.4. `hr_mutex_lock()`

API:

```c
hr_mutex_lock(
    mutex,
    timeout
);
```

Nếu mutex đang free:

```text
Lock
 ↓
owner = current task
 ↓
recursion_count = 1
 ↓
link mutex vào owned_mutexes của task
 ↓
HR_OK
```

Nếu mutex đang thuộc task khác:

```text
Mutex Busy
   ↓
HR_NO_WAIT?
 /          \
YES          NO
↓             ↓
BUSY       đưa task vào Wait List
            ↓
          BLOCKED
```

Waiter được quản lý theo `effective_priority`.

---

## 9.5. `hr_mutex_unlock()`

Chỉ owner mới được unlock.

Với recursive mutex:

```text
recursion_count > 1
        ↓
recursion_count--
        ↓
owner vẫn giữ mutex
```

Khi recursion count về `0`:

```text
release ownership
```

Nếu không có waiter:

```text
owner = NULL
```

Nếu có waiter:

```text
ownership có thể được handoff trực tiếp
cho waiter được chọn
```

Sau đó priority của owner cũ phải được tính lại.

---

## 9.6. Priority Inversion là gì?

Giả sử ba task:

```text
H = High Priority
M = Medium Priority
L = Low Priority
```

L đang giữ mutex:

```text
L owns Mutex X
```

Sau đó H chạy và cần Mutex X:

```text
H lock X
   ↓
X đang thuộc L
   ↓
H BLOCKED
```

Nếu M trở thành READY:

```text
M priority > L
```

thì M có thể preempt L.

Kết quả:

```text
H chờ L
L lại không chạy được vì M
```

Luồng:

```text
L giữ Mutex
   ↓
H cần Mutex
   ↓
H BLOCKED
   ↓
M chạy
   ↓
L không được CPU
   ↓
H bị trì hoãn bởi M
```

Đây là:

```text
Priority Inversion
```

---

## 9.7. Priority Inheritance giải quyết thế nào?

Khi H block trên mutex thuộc L:

```text
H priority cao hơn L
```

kernel tạm boost L:

```text
L.base_priority      = 5
L.effective_priority = 1
```

nếu H có priority `1`.

Lúc này M priority `3` không còn preempt được L:

```text
L effective = 1
M effective = 3
```

L có cơ hội chạy nhanh để:

```text
hoàn thành critical section
↓
unlock mutex
```

Sau đó H nhận mutex và tiếp tục.

---

## 9.8. `base_priority` và `effective_priority`

Mỗi TCB trong `hairtos` lưu riêng:

```text
base_priority
effective_priority
```

`base_priority`:

```text
priority được cấu hình cho task
```

`effective_priority`:

```text
priority scheduler thực sự sử dụng
```

Bình thường:

```text
base = effective
```

Khi Priority Inheritance:

```text
effective
→ được boost
```

nhưng:

```text
base
→ không thay đổi
```

Scheduler và wait policy sử dụng:

```text
effective_priority
```

---

## 9.9. Owner READY phải được Requeue

Giả sử Task L đang READY tại:

```text
Priority 5 Queue
```

Sau khi H chờ mutex của L:

```text
L.effective_priority
5 → 1
```

Không thể chỉ đổi một field trong TCB.

Nếu vẫn để L trong queue priority 5:

```text
Ready Set
→ sai
```

Do đó `hairtos` phải:

```text
remove L khỏi Ready Queue cũ
      ↓
update effective_priority
      ↓
insert L vào Ready Queue mới
```

Mental model:

```text
Priority thay đổi
=
Ready Queue membership cũng phải thay đổi
```

---

## 9.10. Một Task giữ nhiều Mutex

Một task có thể sở hữu nhiều mutex.

TCB có:

```text
owned_mutexes
owned_mutex_count
```

Ví dụ:

```text
Task L owns M1
Task L owns M2
```

M1 có waiter priority `1`.

M2 có waiter priority `3`.

L có:

```text
base_priority = 5
```

thì effective priority phải dựa trên waiter cao nhất:

```text
effective_priority = 1
```

Nếu L unlock M1 nhưng vẫn giữ M2:

```text
không được restore ngay về 5
```

vì waiter priority `3` trên M2 vẫn còn.

Kernel phải tính lại:

```text
effective_priority
=
min(
    base_priority,
    highest-priority waiter
    trên mọi mutex đang sở hữu
)
```

Với quy ước:

```text
số nhỏ hơn = priority cao hơn
```

---

## 9.11. Chained Priority Inheritance

Priority Inheritance có thể lan qua nhiều mutex.

Ví dụ:

```text
Task A priority 1
Task B priority 3
Task C priority 5
```

Quan hệ:

```text
C owns M1

B owns M2
B đang chờ M1

A đang chờ M2
```

Chuỗi:

```text
A
 ↓ waits
M2
 ↓ owned by
B
 ↓ waits
M1
 ↓ owned by
C
```

Khi A priority `1` chờ B:

```text
B effective → 1
```

Nhưng B đang chờ mutex của C, nên inheritance phải lan tiếp:

```text
C effective → 1
```

Có thể nhớ:

```text
A → M2 → B → M1 → C
```

`hairtos` hỗ trợ chained inheritance bằng quá trình recompute/propagate có giới hạn để tránh recursion bất thường hoặc chuỗi bệnh lý.

---

## 9.12. Direct Ownership Handoff khi Unlock

Giả sử:

```text
L owns Mutex
H BLOCKED chờ Mutex
```

Khi L unlock, thay vì:

```text
owner = NULL
↓
wake H
↓
H chạy
↓
H lock lại Mutex
```

kernel có thể:

```text
L unlock
   ↓
chọn waiter H
   ↓
owner = H
   ↓
wake H
```

Đây là:

```text
Direct Ownership Handoff
```

Nó giữ trạng thái mutex nhất quán và tránh khoảng trống không cần thiết giữa unlock và lock lại.

Sau handoff:

```text
H trở thành owner
```

và priority của owner cũ được recompute.

---

## 9.13. Timeout khi chờ Mutex

Nếu task lock mutex với timeout hữu hạn:

```c
hr_mutex_lock(&mutex, 100);
```

khi mutex đang bận:

```text
Mutex Wait List
+
Timeout List
```

Hai khả năng:

```text
Mutex được unlock trước
→ task nhận ownership
→ HR_OK

Timeout trước
→ task rời Wait List
→ HR_ERROR_TIMEOUT
```

Điểm quan trọng:

Khi waiter timeout, owner có thể không còn cần boost mạnh như trước.

Do đó kernel phải:

```text
remove waiter
      ↓
recompute owner effective_priority
```

và nếu owner nằm trong dependency chain thì việc recompute có thể cần lan tiếp.

---

## 9.14. Mutex không dùng trong ISR

Public mutex API:

```text
hr_mutex_lock()
hr_mutex_unlock()
```

không được dùng từ ISR context.

Nếu gọi trong ISR:

```text
HR_ERROR_FROM_ISR
```

Lý do:

```text
Mutex có ownership theo task
+
lock có thể block
+
Priority Inheritance phụ thuộc TCB
```

ISR không phải task và không được ngủ chờ mutex.

Nếu ISR cần báo sự kiện cho task:

```text
Semaphore
hoặc
Queue _from_isr
```

phù hợp hơn.

---

## 9.15. Semaphore và Mutex khác nhau

Semaphore:

```text
không owner
có count
có thể signal từ ISR
không Priority Inheritance
```

Mutex:

```text
có owner
bảo vệ shared resource
có recursion policy
có Priority Inheritance
không dùng từ ISR
```

Mental model:

```text
Semaphore
= "có bao nhiêu token / event?"

Mutex
= "task nào đang sở hữu resource?"
```

---

## 9.16. Ví dụ đầy đủ L / M / H

Giả sử:

```text
H priority = 1
M priority = 3
L priority = 5
```

### Bước 1

L chạy và lock mutex:

```text
L
↓
Lock X
↓
owner = L
```

### Bước 2

H trở thành READY và preempt L:

```text
H
↓
Lock X
↓
X owned by L
↓
H BLOCKED
```

### Nếu không có Priority Inheritance

M có thể chạy:

```text
M READY
↓
M priority 3 > L priority 5
↓
M preempt L
```

H tiếp tục phải chờ.

### Với Priority Inheritance

Khi H block:

```text
L.base_priority      = 5
L.effective_priority = 1
```

L trở thành task ưu tiên cao:

```text
L chạy
↓
hoàn thành critical section
↓
Unlock X
```

Kernel:

```text
ownership X
L → H
```

Sau đó:

```text
recompute L priority
→ 5
```

H trở thành READY và chạy:

```text
H owns X
↓
H RUNNING
```

Flow:

```text
L owns X
   ↓
H waits X
   ↓
Boost L
5 → 1
   ↓
L finishes
   ↓
Unlock
   ↓
Handoff X → H
   ↓
Restore / Recompute L
   ↓
H runs
```

---

## 9.17. Mô hình tư duy cần nhớ

Lock:

```text
LOCK
 ↓
Mutex free?
 /       \
YES       NO
↓          ↓
Become    Current owner?
Owner      /         \
         YES         NO
         ↓            ↓
      Recursive?    NO_WAIT?
       /    \        /     \
     YES    NO     YES     NO
     ↓       ↓      ↓       ↓
 count++   reject  BUSY    BLOCK
                              ↓
                         Wait List
                              ↓
                    Priority Inheritance
```

Unlock:

```text
UNLOCK
  ↓
Caller == Owner?
 /           \
NO           YES
↓             ↓
NOT_OWNER   Recursive count > 1?
              /          \
            YES          NO
            ↓             ↓
         count--      Waiter?
                     /      \
                   YES      NO
                   ↓         ↓
               Handoff     Free
                   ↓
          Recompute old owner
```

Priority Inheritance:

```text
High waits on Mutex
      ↓
Low owns Mutex
      ↓
Boost Low effective_priority
      ↓
Low completes sooner
      ↓
Unlock / Handoff
      ↓
Recompute Low priority
```

---

## 9.18. Trọng tâm cần thuộc

API chính:

```text
hr_mutex_create()

hr_mutex_create_recursive()

hr_mutex_lock()

hr_mutex_unlock()
```

Thông tin quan trọng trong mutex:

```text
owner
recursion_count
recursive
waiters
owner_node
```

Thông tin quan trọng trong TCB:

```text
base_priority
effective_priority

owned_mutexes
owned_mutex_count
```

Các quy tắc:

- Mutex có ownership.
- Chỉ owner được unlock.
- Normal mutex không cho owner lock lại chính nó.
- Recursive mutex tăng `recursion_count`.
- Mutex bận + `HR_NO_WAIT` trả busy.
- Lock có timeout có thể đưa task vào `BLOCKED`.
- Waiter được quản lý theo `effective_priority`.
- Task priority cao hơn chờ mutex có thể boost owner.
- Scheduler sử dụng `effective_priority`.
- READY owner đổi priority phải được requeue.
- Owner giữ nhiều mutex phải recompute priority từ tất cả mutex đang sở hữu.
- Unlock một mutex chưa chắc làm owner trở lại `base_priority`.
- Priority Inheritance có thể lan theo chuỗi mutex.
- Unlock có thể direct handoff ownership cho waiter được chọn.
- Waiter timeout phải làm owner recompute lại priority.
- Mutex API không dùng trong ISR.
- Mutex khác semaphore vì có owner và Priority Inheritance.

Điểm quan trọng nhất:

> **Priority Inheritance không làm task Low trở thành High vĩnh viễn. Nó chỉ tạm nâng `effective_priority` của owner để owner có CPU hoàn thành critical section và nhả resource mà task High đang cần.**

> **Trong `hairtos`, effective priority phải được tính từ `base_priority` và waiter ưu tiên cao nhất trên tất cả mutex mà task đang sở hữu; vì vậy unlock một mutex không có nghĩa priority lập tức trở về base.**

> **Nếu owner đang block trên mutex khác, việc boost có thể lan theo chuỗi, tạo Chained Priority Inheritance.**


---

# 10. Suspend / Resume + Software Timer

Chương này gồm hai cơ chế khác nhau:

```text
Suspend / Resume
= chủ động loại / đưa Task trở lại scheduling

Software Timer
= lập lịch callback theo Kernel Tick
```

Điểm cần phân biệt là `SUSPENDED` không giống `BLOCKED`, còn callback của Software Timer không chạy trực tiếp trong `SysTick_Handler`.

---

## 10.1. Suspend khác BLOCKED như thế nào?

`BLOCKED` nghĩa là task đang chờ một điều kiện cụ thể:

```text
delay
queue
semaphore
mutex
timeout
```

Ví dụ:

```text
RUNNING
  ↓
hr_task_delay(100)
  ↓
BLOCKED
```

`SUSPENDED` là trạng thái quản trị:

```text
"Task này tạm thời không được scheduler chạy"
```

Do đó:

```text
BLOCKED
= đang chờ một event

SUSPENDED
= bị chủ động dừng khỏi scheduling
```

Trong `hairtos`, thông tin suspend được giữ tách biệt với lý do task đang chờ.

---

## 10.2. `hr_task_suspend()`

API:

```c
hr_task_suspend(task);
```

Luồng khái niệm:

```text
Task hiện tại
READY / RUNNING / BLOCKED
        ↓
hr_task_suspend()
        ↓
lưu trạng thái cần Resume
        ↓
state = SUSPENDED
```

Nếu task đang nằm trong Ready Set:

```text
remove khỏi Ready Set
```

Nếu suspend chính task đang chạy:

```text
current task
→ SUSPENDED
→ phải reschedule
→ PendSV
```

---

## 10.3. Suspend một Task đang BLOCKED

Đây là phần quan trọng trong `hairtos`.

Giả sử:

```text
Worker
  ↓
hr_task_delay(100)
  ↓
BLOCKED
```

Supervisor suspend Worker khi Worker vẫn đang chờ delay.

Kernel phải nhớ:

```text
Worker đang SUSPENDED
nhưng trạng thái trước suspend là BLOCKED
```

Có thể hình dung:

```text
BLOCKED
   ↓
suspend()
   ↓
SUSPENDED(BLOCKED)
```

Suspend không được làm mất wait reason của task.

---

## 10.4. Deferred Wakeup khi đang SUSPENDED

Giả sử Worker:

```text
SUSPENDED(BLOCKED)
```

Trong lúc nó đang suspended:

```text
timeout hết
```

Kernel không được đưa Worker ngay vào Ready Set.

Thay vào đó phải ghi nhận rằng điều kiện chờ đã hoàn thành:

```text
timeout / event completed
        ↓
Worker vẫn SUSPENDED
        ↓
deferred wake
```

Có thể hiểu task lúc này là:

```text
SUSPENDED(READY)
```

Nghĩa là:

```text
nếu không bị suspend
thì nó đã READY
```

---

## 10.5. `hr_task_resume()`

API:

```c
hr_task_resume(task);
```

Kết quả phụ thuộc trạng thái được lưu.

### Nếu event chưa hoàn thành

```text
SUSPENDED(BLOCKED)
       ↓
resume()
       ↓
BLOCKED
```

Task tiếp tục chờ.

### Nếu event đã hoàn thành

```text
SUSPENDED(READY)
       ↓
resume()
       ↓
READY
```

Task được đưa trở lại Ready Set.

Nếu task vừa resume có priority cao hơn current task:

```text
READY
  ↓
Preemption Required
  ↓
PendSV
```

---

## 10.6. Self-Suspend

Một task cũng có thể suspend chính nó:

```text
Task A RUNNING
    ↓
hr_task_suspend(Task A)
    ↓
Task A SUSPENDED
    ↓
Scheduler chọn Task khác
```

Task A không thể tự chạy lại vì nó đã mất CPU.

Do đó phải có task khác:

```text
Supervisor
    ↓
hr_task_resume(Task A)
```

để đưa A trở lại scheduling.

---

## 10.7. Idle Task không được Suspend

Idle task là fallback cuối cùng của scheduler:

```text
không có application task READY
        ↓
Idle Task RUNNING
```

Do đó `hairtos` không cho suspend Idle Task.

Nếu Idle cũng bị loại khỏi scheduler:

```text
Ready Set có thể không còn task hợp lệ
```

điều này phá invariant của scheduler.

---

## 10.8. Software Timer là gì?

Software Timer là timer do kernel quản lý dựa trên Kernel Tick.

Ví dụ:

```text
Start Timer
period = 100 tick
      ↓
100 tick trôi qua
      ↓
Timer Expired
      ↓
Callback được lập lịch
```

Nó không phải hardware timer peripheral như:

```text
TIM2
TIM3
TIM4
```

Software Timer dùng hạ tầng thời gian của RTOS.

---

## 10.9. Timer Object trong `hairtos`

Timer được tạo tĩnh bằng:

```c
hr_timer_create_static();
```

Timer object chứa các thông tin chính như:

```text
name
period
auto_reload
callback
argument
timeout node
pending state
```

Object và storage do caller sở hữu.

Có thể hiểu:

```text
Timer
=
deadline
+
period
+
callback
+
state
```

---

## 10.10. Timer-Service Task

`hairtos` không chạy callback trong SysTick ISR.

Thay vào đó có:

```text
Timer-Service Task
```

Task này được tạo khi timer subsystem được khởi tạo lần đầu và dùng priority/stack từ cấu hình.

Luồng:

```text
SysTick
   ↓
phát hiện Timer hết hạn
   ↓
đưa Timer vào Pending Queue
   ↓
đánh thức Timer-Service Task
```

Sau đó:

```text
Timer-Service Task
   ↓
lấy Timer pending
   ↓
gọi Callback
```

Callback vì vậy chạy trong:

```text
Task Context
```

không phải:

```text
Handler Mode
```

---

## 10.11. Timer Expiry Flow

Luồng đầy đủ:

```text
Kernel Tick
   ↓
Timer deadline tới
   ↓
SysTick ISR
   ↓
đánh dấu / queue pending callback
   ↓
signal Timer-Service Task
   ↓
ISR kết thúc
   ↓
Scheduler
   ↓
Timer-Service Task RUNNING
   ↓
pop pending Timer
   ↓
invoke callback
```

Có thể nhớ:

```text
SysTick
= phát hiện expiry

Timer-Service Task
= chạy callback
```

---

## 10.12. One-Shot và Periodic Timer

### One-Shot

```text
Start
 ↓
Expire
 ↓
Callback
 ↓
Inactive
```

Callback chỉ chạy một lần cho lần start đó.

### Periodic / Auto-Reload

```text
Start
 ↓
Expire
 ↓
Callback
 ↓
Re-arm
 ↓
Expire
 ↓
Callback
 ↓
...
```

Periodic timer trong `hairtos` được re-arm tự động.

Repo re-arm dựa trên deadline để hạn chế drift thay vì đơn giản lấy thời điểm callback hoàn tất làm mốc mới.

---

## 10.13. `start`, `reset`, `change_period`, `stop`

Các API chính:

```c
hr_timer_start()

hr_timer_reset()

hr_timer_change_period()

hr_timer_stop()
```

Có thể hiểu:

### `start`

```text
Timer inactive
    ↓
start
    ↓
deadline = now + period
```

### `reset`

```text
Timer
  ↓
reset
  ↓
tính lại expiry từ thời điểm reset
```

### `change_period`

```text
period cũ
   ↓
change_period
   ↓
period mới
```

### `stop`

```text
Timer active
   ↓
stop
   ↓
không còn active timeout
```

Implementation hiện tại còn phải xử lý nhất quán cả:

```text
active timeout node
+
pending callback
```

khi stop/reset/change period.

---

## 10.14. `pending_count`

Một điểm quan trọng của `hairtos` là mỗi timer có:

```text
pending_count
```

Giả sử Timer-Service Task bị trễ trong khi periodic timer đã expire nhiều lần.

Không nên:

```text
3 expiration
→ chỉ còn 1 callback pending
```

vì sẽ làm mất thông tin expiration.

`pending_count` giúp giữ số lần expiry đang chờ xử lý:

```text
Expire
→ pending_count++

Service xử lý callback
→ pending_count--
```

Nhờ vậy nhiều expiration không bị âm thầm gộp mất khi service task chưa chạy kịp.

---

## 10.15. Vì sao Callback không chạy trong SysTick ISR?

Nếu callback chạy trực tiếp trong SysTick:

```text
SysTick ISR
   ↓
User Callback
   ↓
logic application
```

thì ISR có thể bị kéo dài và làm tăng interrupt latency.

`hairtos` tách:

```text
ISR
= cập nhật timer state + queue pending work

Task
= chạy callback
```

Nhờ đó callback chạy trong task context và không kéo dài trực tiếp SysTick Handler.

Điểm cần nhớ:

> Timer expiry xảy ra trong nhịp kernel, nhưng callback execution được hoãn sang Timer-Service Task.

---

## 10.16. Ví dụ Suspend / Resume

Giả sử:

```text
Worker priority = 1
Supervisor priority = 2
```

Worker:

```text
hr_task_delay(100)
→ BLOCKED
```

Supervisor:

```text
suspend Worker
```

Trạng thái:

```text
SUSPENDED(BLOCKED)
```

Sau đó 100 tick trôi qua:

```text
timeout hết
```

nhưng Worker vẫn:

```text
SUSPENDED
```

Kernel chỉ ghi nhận:

```text
deferred wake
```

Khi Supervisor gọi:

```text
resume Worker
```

Worker trở thành:

```text
READY
```

Vì priority `1` cao hơn Supervisor `2`:

```text
Preemption
   ↓
PendSV
   ↓
Worker RUNNING
```

---

## 10.17. Ví dụ Software Timer

Giả sử periodic timer:

```text
period = 250 tick
auto_reload = true
```

Luồng:

```text
hr_timer_start()
      ↓
250 tick
      ↓
SysTick phát hiện expiry
      ↓
queue pending Timer
      ↓
wake Timer-Service Task
      ↓
Callback
      ↓
re-arm Timer
```

Sau đó đổi period:

```text
250 tick
   ↓
hr_timer_change_period()
   ↓
500 tick
```

Timer tiếp tục hoạt động với period mới.

---

## 10.18. Mô hình tư duy cần nhớ

Suspend / Resume:

```text
READY / RUNNING / BLOCKED
        ↓
     suspend()
        ↓
    SUSPENDED
        ↓
event có thể vẫn xảy ra
        ↓
deferred wake nếu cần
        ↓
      resume()
      /      \
BLOCKED     READY
```

Software Timer:

```text
Timer
  ↓
Timeout List
  ↓
SysTick detects expiry
  ↓
Pending Queue
  ↓
Timer-Service Task
  ↓
Callback
```

Phân biệt:

```text
SysTick ISR
= phát hiện Timer hết hạn

Timer-Service Task
= chạy callback
```

---

## 10.19. Trọng tâm cần thuộc

Suspend / Resume:

```text
BLOCKED
= chờ event

SUSPENDED
= bị chủ động loại khỏi scheduling
```

Các API:

```text
hr_task_suspend()
hr_task_resume()
```

Các nguyên tắc:

- Suspend là trạng thái quản trị độc lập với wait reason.
- Có thể suspend task đang `READY`, `RUNNING` hoặc `BLOCKED`.
- Suspend task đang `RUNNING` phải dẫn tới reschedule.
- Task đang `SUSPENDED` không được đưa vào Ready Set chỉ vì timeout/event xảy ra.
- Event xảy ra trong lúc suspend phải được ghi nhận dưới dạng deferred wake.
- Resume trả task về `BLOCKED` nếu wait chưa hoàn thành.
- Resume trả task về `READY` nếu wait đã hoàn thành.
- Task vừa resume có thể preempt current task nếu priority cao hơn.
- Idle Task không được suspend.

Software Timer:

```text
hr_timer_create_static()
hr_timer_start()
hr_timer_reset()
hr_timer_change_period()
hr_timer_stop()
```

Các nguyên tắc:

- Software Timer dựa trên Kernel Tick.
- Timer object là static/caller-owned storage.
- Callback không chạy trong SysTick ISR.
- SysTick chỉ phát hiện expiry và queue pending work.
- Timer-Service Task chạy callback trong task context.
- One-shot timer inactive sau expiry.
- Periodic timer tự re-arm.
- Periodic rearm dựa trên deadline để hạn chế drift.
- `pending_count` giữ số expiration chưa được xử lý.
- `stop/reset/change_period` phải xử lý cả active timeout và pending callback một cách nhất quán.

Điểm quan trọng nhất:

> **`SUSPENDED` không có nghĩa là “đang chờ timeout”. Nó là một lớp điều khiển độc lập; event có thể hoàn thành trong lúc task bị suspend nhưng task chỉ được chạy lại sau `resume()`.**

> **Software Timer của `hairtos` tách “timer hết hạn” khỏi “chạy callback”: SysTick xử lý expiry tối thiểu, còn callback chạy trong Timer-Service Task.**


---

# 11. haievent + Event Ownership + Reference Counting

`haievent` là event-driven framework nằm phía trên kernel `hairtos`.

Nó không tạo scheduler riêng mà dùng các primitive đã học:

```text
Task
Queue
Software Timer
Critical Section
```

để xây dựng:

```text
Event
Active Object
State Machine
Time Event
Publish / Subscribe
```

Chương này tập trung vào nền tảng quan trọng nhất trước khi học Active Object và FSM:

```text
Event là gì?
Ai sở hữu Event?
Event sống tới khi nào?
Khi nào Event được trả lại Pool?
```

---

## 11.1. `haievent` là gì?

Có thể hình dung kiến trúc:

```text
Application
    ↓
 haievent
    ↓
 hairtos
    ↓
Cortex-M3
```

`haievent` không thay thế RTOS.

Nó cung cấp mô hình:

```text
Event
  ↓
Queue
  ↓
Active Object
  ↓
State Machine
```

thay vì application tự viết nhiều vòng lặp task và cờ trạng thái rời rạc.

---

## 11.2. Event và Signal

Mỗi event có một:

```text
signal
```

để mô tả loại sự kiện.

Trong `hairtos`:

```text
he_signal_t = uint16_t
```

Các signal framework dành riêng:

```text
HE_SIG_NONE    = 0
HE_SIG_ENTRY   = 1
HE_SIG_EXIT    = 2
HE_SIG_INIT    = 3
HE_SIG_TIMEOUT = 4
```

Application-defined signal bắt đầu từ:

```text
HE_SIG_USER = 32
```

Ví dụ:

```c
enum
{
    SIG_BUTTON_PRESSED = HE_SIG_USER,
    SIG_SENSOR_READY,
    SIG_UART_FRAME
};
```

---

## 11.3. `he_event_t` chứa gì?

Header event hiện tại có dạng khái niệm:

```text
he_event_t
├── pool
├── magic
├── size
├── reference_count
├── signal
└── flags
```

Các field quan trọng:

```text
pool
= event thuộc pool nào

size
= kích thước event

reference_count
= số reference đang giữ dynamic event

signal
= loại event

flags
= static hay dynamic
```

Application có thể mở rộng event:

```c
typedef struct
{
    he_event_t super;

    uint16_t adc_value;
    uint8_t channel;
} sensor_event_t;
```

Điểm quan trọng:

```text
he_event_t
```

thường nằm ở đầu struct mở rộng.

---

## 11.4. Static Event

Static event được khởi tạo bằng:

```c
he_event_init_static();
```

Đặc điểm:

```text
pool = NULL
reference_count = 0
flag = STATIC
```

Framework không trả static event về pool.

Lifetime do application chịu trách nhiệm.

Ví dụ phù hợp:

```text
event không payload thay đổi
+
được dùng lặp lại nhiều lần
+
storage tồn tại lâu dài
```

Ví dụ:

```c
static he_event_t button_pressed_event;
```

---

## 11.5. Dynamic Event

Dynamic event được tạo bằng:

```c
he_event_new();
```

Nó lấy một block từ:

```text
Event Pool
```

Khi tạo thành công:

```text
reference_count = 1
flag = DYNAMIC
pool = owning pool
```

Mental model:

```text
Pool
 ↓ allocate
Dynamic Event
ref = 1
```

Reference đầu tiên thuộc code vừa gọi `he_event_new()`.

---

## 11.6. Fixed-Block Event Pool

`haievent` không dùng general-purpose heap để cấp phát event.

Pool được tạo bằng:

```c
he_event_pool_init(
    pool,
    storage,
    block_size,
    block_count
);
```

Storage do application cung cấp.

Ví dụ khái niệm:

```text
Event Pool

+---------+
| Block 0 |
+---------+
| Block 1 |
+---------+
| Block 2 |
+---------+
| Block 3 |
+---------+
```

Mỗi block có cùng kích thước.

Ưu điểm:

```text
bounded memory
không external fragmentation
allocation / free đơn giản
memory usage dễ dự đoán
```

Nhược điểm:

```text
event nhỏ vẫn chiếm nguyên block
```

nên có thể có internal fragmentation.

Dynamic event chỉ được tạo nếu:

```text
event_size <= block_size
```

và pool còn free block.

---

## 11.7. Reference Counting

Reference Counting trả lời:

> **Hiện còn bao nhiêu nơi có quyền giữ dynamic event này sống?**

Ví dụ:

```text
Event E
ref = 1
```

Nếu một consumer khác cũng cần giữ E:

```text
retain
↓
ref = 2
```

Khi một owner dùng xong:

```text
release
↓
ref = 1
```

Owner cuối cùng release:

```text
ref = 0
↓
Event trả về Pool
```

Flow:

```text
FREE
 ↓ he_event_new()
OWNED
ref = 1
 ↓ retain
SHARED
ref > 1
 ↓ release
OWNED
ref = 1
 ↓ final release
FREE
```

---

## 11.8. `he_event_retain()`

API:

```c
he_event_retain(event);
```

Với dynamic event:

```text
reference_count++
```

Với static event:

```text
không thay đổi lifetime
→ HR_OK
```

Refcount là:

```text
uint16_t
```

nên implementation kiểm tra overflow.

Nếu:

```text
reference_count == UINT16_MAX
```

thì retain thất bại với lỗi overflow thay vì wrap về `0`.

---

## 11.9. `he_event_release()`

API:

```c
he_event_release(event);
```

Với dynamic event:

```text
reference_count--
```

Nếu sau decrement:

```text
reference_count > 0
```

event vẫn còn owner.

Nếu:

```text
reference_count == 0
```

kernel/framework:

```text
invalidate event metadata
      ↓
đưa block trở lại free list
      ↓
pool.free_count++
```

Với static event:

```text
release()
```

không reclaim storage.

---

## 11.10. Ownership khi `he_active_post()`

Đây là quy tắc rất quan trọng.

```c
he_active_post(
    active,
    event,
    timeout
);
```

implementation hiện tại đưa:

```text
event pointer
```

vào queue của Active Object mà không tự `retain()`.

Do đó với dynamic event, cách hiểu đúng là:

```text
Caller đang giữ 1 reference
        ↓
he_active_post() thành công
        ↓
reference đó được chuyển cho AO queue
```

Sau post thành công:

> Caller không được tiếp tục coi reference đó là reference riêng của mình trừ khi đã retain thêm trước đó.

Mental model:

```text
Caller owns E
ref = 1
      ↓ post success
AO Queue owns E
ref = 1
```

---

## 11.11. `he_active_post_shared()`

Nếu caller muốn:

```text
vẫn giữ event
+
đồng thời đưa event cho AO
```

thì dùng:

```c
he_active_post_shared();
```

Flow:

```text
Caller owns E
ref = 1
      ↓
retain()
      ↓
ref = 2
      ↓
enqueue vào AO
```

Sau post:

```text
Caller reference
+
AO reference
```

Nếu enqueue thất bại:

```text
retain
↓
post fail
↓
release rollback
```

nên refcount quay về giá trị trước đó.

---

## 11.12. Event được release sau Dispatch

Active Object task lấy event pointer từ queue:

```text
AO Queue
   ↓
receive Event
   ↓
State Machine Dispatch
```

Sau khi dispatch hoàn tất, `he_active` gọi:

```c
he_event_release(event);
```

Do đó với dynamic event:

```text
Queue ownership
      ↓
Dispatch
      ↓
release
```

Nếu đó là reference cuối cùng:

```text
ref = 0
→ block trả về Event Pool
```

Điều này tạo ownership flow rõ ràng:

```text
Producer
   ↓ post
AO Queue
   ↓ receive
State Machine
   ↓ dispatch done
release
```

---

## 11.13. Publish/Subscribe và nhiều Owner

Publish/Subscribe là trường hợp reference counting đặc biệt quan trọng.

Giả sử một dynamic event được publish tới:

```text
AO A
AO B
AO C
```

Mỗi subscriber nhận cùng một event pointer.

Framework phải giữ event sống cho tới khi tất cả consumer dùng xong.

Flow khái niệm:

```text
Publisher
ref = 1
   ↓
post_shared → A
ref = 2
   ↓
post_shared → B
ref = 3
   ↓
post_shared → C
ref = 4
```

Sau khi publish hoàn tất, publisher release reference của mình:

```text
ref = 3
```

Mỗi AO dispatch xong:

```text
A release → 2
B release → 1
C release → 0
```

Cuối cùng:

```text
Event → Pool
```

Trong implementation hiện tại, publish còn snapshot subscriber list trước rồi mới post ra ngoài critical section.

Chi tiết Pub/Sub sẽ học sâu ở Chương 12.

---

## 11.14. Ownership khi Post thất bại

Giả sử:

```text
dynamic event
ref = 1
```

Caller dùng:

```text
he_active_post_shared()
```

Implementation:

```text
retain
↓
ref = 2
↓
enqueue
```

Nếu queue full hoặc post thất bại:

```text
release reference vừa retain
↓
ref = 1
```

Caller vẫn giữ reference gốc.

Đây là invariant quan trọng:

> **Operation thất bại không được làm rò rỉ reference mà nó vừa tạo.**

Với `he_active_post()` thông thường, chỉ khi post thành công thì caller mới coi ownership đã chuyển sang AO.

---

## 11.15. Static Event Lifetime

Reference counting không quản lý lifetime của static event.

Ví dụ nguy hiểm:

```c
void send_event(void)
{
    he_event_t event;

    he_event_init_static(&event, SIG_DATA, sizeof(event));

    he_active_post(&ao, &event, 0);

}   // stack variable chết ở đây
```

AO có thể xử lý event sau khi function đã return.

Khi đó pointer trong queue không còn hợp lệ.

Do đó static event phải có lifetime dài hơn mọi consumer còn giữ pointer tới nó.

Phù hợp:

```text
global static storage
static const-like event
application-owned object có lifetime đủ dài
```

Không phù hợp:

```text
automatic local variable
rồi post asynchronous
```

---

## 11.16. Thread Safety của Event Pool

Các operation như:

```text
allocate block
return block
retain
release
```

có thể được gọi từ các execution context khác nhau.

Implementation bảo vệ các cập nhật:

```text
free list
free_count
reference_count
```

bằng critical section.

Trên Cortex-M3 hiện tại:

```text
critical section
→ PRIMASK
```

Nguyên tắc vẫn giống kernel:

```text
critical section phải ngắn
không block bên trong
```

---

## 11.17. Các lỗi Ownership thường gặp

### Release quá sớm

```text
new()
ref = 1
↓
post()
↓
caller release ngay dù ownership đã transfer
↓
AO giữ pointer tới block đã free
```

Có thể dẫn đến:

```text
use-after-free
```

### Quên Release

```text
new()
ref = 1
↓
retain
ref = 2
↓
chỉ release một lần
```

Kết quả:

```text
ref = 1 mãi
→ block không về pool
→ event leak
```

### Double Release

```text
ref = 1
↓
release
ref = 0
→ free
↓
release lần nữa
```

Pointer không còn là một event dynamic hợp lệ.

### Static Event chết quá sớm

```text
local static-style event trên stack
↓
post asynchronous
↓
function return
↓
AO dùng dangling pointer
```

### Dùng chung Dynamic Event nhưng không Retain

```text
Producer
 ├→ AO A
 └→ AO B
```

Nếu chỉ có một reference mà hai consumer cùng nghĩ mình sở hữu:

```text
consumer đầu release
→ event có thể free
→ consumer sau dùng invalid event
```

Khi cùng event được chia sẻ:

```text
mỗi lifetime owner
→ cần một reference
```

---

## 11.18. Mô hình tư duy cần nhớ

Static Event:

```text
Application Storage
      ↓
init_static
      ↓
post / dispatch
      ↓
Framework không free
      ↓
Application chịu trách nhiệm lifetime
```

Dynamic Event:

```text
Fixed-Block Pool
      ↓
he_event_new()
      ↓
ref = 1
      ↓
retain khi có thêm owner
      ↓
release khi owner dùng xong
      ↓
ref = 0
      ↓
return block to Pool
```

Ownership transfer:

```text
he_active_post()
= chuyển reference hiện tại cho AO

he_active_post_shared()
= giữ reference caller
  + tạo thêm reference cho AO
```

AO execution:

```text
Queue
 ↓
receive Event
 ↓
Dispatch
 ↓
Release
```

---

## 11.19. Trọng tâm cần thuộc

Các signal:

```text
HE_SIG_NONE    = 0
HE_SIG_ENTRY   = 1
HE_SIG_EXIT    = 2
HE_SIG_INIT    = 3
HE_SIG_TIMEOUT = 4

HE_SIG_USER
= 32
```

Event header:

```text
pool
magic
size
reference_count
signal
flags
```

API chính:

```text
he_event_pool_init()

he_event_new()

he_event_init_static()

he_event_retain()

he_event_release()

he_event_is_valid()

he_event_is_dynamic()

he_event_get_reference_count()
```

Các nguyên tắc:

- `haievent` nằm trên `hairtos`, không có scheduler riêng.
- Static event không được framework reclaim.
- Static event có `reference_count = 0`.
- Dynamic event được cấp từ fixed-block pool.
- Dynamic event mới có `reference_count = 1`.
- Mỗi owner giữ dynamic event phải tương ứng với một reference.
- `retain()` tăng refcount của dynamic event.
- `release()` giảm refcount.
- Final release đưa block trở lại pool.
- Refcount là `uint16_t` và có overflow protection.
- `he_active_post()` không tự retain; post thành công chuyển reference hiện tại sang AO.
- `he_active_post_shared()` retain trước khi enqueue.
- Shared post thất bại phải rollback reference vừa retain.
- AO release event sau khi state machine dispatch xong.
- Publish giữ một reference cho mỗi subscriber được post thành công rồi consume publisher reference.
- Static event phải có storage sống đủ lâu cho mọi asynchronous consumer.
- Pool/free-list/refcount update được bảo vệ bằng critical section.
- Sai ownership dễ gây use-after-free, double release hoặc leak pool block.

Điểm quan trọng nhất:

> **Dynamic event không thuộc về một function hay một task cố định; ownership được biểu diễn bằng `reference_count`. Mỗi nơi cần giữ event sống phải có một reference, và owner cuối cùng `release()` sẽ trả block về fixed-block pool.**

> **`he_active_post()` là ownership transfer, còn `he_active_post_shared()` là ownership sharing. Phân biệt hai semantics này là chìa khóa để tránh use-after-free và memory leak trong `haievent`.**


---

# 12. Active Object + FSM + Publish / Subscribe

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 13. Diagnostics + Fault + Stack Safety

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.

---

# 14. Architecture Port + Testing + Benchmark

> Chưa bổ sung nội dung. Phần này sẽ được điền khi tiếp tục ôn theo thứ tự tài liệu.
