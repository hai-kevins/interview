# Ghi chú phỏng vấn STM32

> **Mục tiêu:** Ôn STM32 theo hướng phỏng vấn Embedded Firmware, đi từ kiến trúc nền tảng đến ngoại vi và gỡ lỗi.

---

<a id="muc-luc"></a>
## Lộ trình tài liệu

1. **STM32 architecture / memory map**
   - 1.1. Processor Core vs Processor vs Microcontroller
   - 1.2. Operation Modes
   - 1.3. Access Level
   - 1.4. Core Registers
   - 1.5. Reset Sequence
   - 1.6. Bus Architecture
   - **1.7. Memory Map** ← đang triển khai
   - 1.8. Memory-Mapped I/O
   - 1.9. Flash và SRAM
   - 1.10. Stack cơ bản trên Cortex-M
   - 1.11. Startup Code
   - 1.12. Linker Script và các section
2. **RCC + Clock**
3. **GPIO**
4. **Interrupt + NVIC + EXTI**
5. **Timer + PWM**
6. **UART**
7. **SPI + I2C**
8. **ADC**
9. **DMA**
10. **Debug bằng ST-Link**

---

<a id="chuong-01"></a>
# 1. STM32 architecture / memory map

<a id="muc-01-01"></a>
## 1.1. Processor Core vs Processor vs Microcontroller

### 1.1.1. Ba khái niệm cần phân biệt

Ba khái niệm này có quan hệ lồng nhau nhưng **không đồng nghĩa**.

```text
Microcontroller
│
├── Processor
│   │
│   ├── Processor Core
│   │   ├── ALU
│   │   ├── logic giải mã và thực thi lệnh
│   │   ├── các thanh ghi
│   │   ├── pipeline
│   │   ├── khối nhân/chia phần cứng
│   │   └── khối tạo địa chỉ
│   │
│   └── các khối hỗ trợ xử lý
│       └── ví dụ: NVIC, các khối điều khiển hệ thống...
│
├── Flash
├── SRAM
└── Peripheral
    ├── GPIO
    ├── Timer
    ├── UART
    ├── SPI
    ├── I2C
    └── ...
```

#### Processor Core — lõi xử lý

**Core** là phần trực tiếp thực hiện các lệnh của chương trình.

Theo hình minh họa trong tài liệu, core có các thành phần/chức năng chính:

- **ALU** — thực hiện các phép toán số học và logic.
- Logic để **giải mã và thực thi lệnh**.
- Nhiều **thanh ghi** để lưu và thao tác dữ liệu.
- **Pipeline** để tăng hiệu quả thực thi lệnh.
- Khối **nhân và chia bằng phần cứng**.
- Khối **tạo địa chỉ** để phục vụ truy cập dữ liệu/lệnh.

Có thể hình dung:

```text
Instruction
    ↓
Decode
    ↓
Execute
    ↓
ALU / các khối xử lý
    ↓
Result
```

Core cũng chứa các thanh ghi đặc biệt mà tài liệu nhắc tới như:

```text
R0 → R15
xPSR
CONTROL
...
```

Phần ý nghĩa cụ thể của từng thanh ghi sẽ được học riêng ở mục **Core Registers**.

---

#### Processor — bộ xử lý

**Processor** là khái niệm rộng hơn **processor core**.

Hình nguồn minh họa một **Cortex-M4 processor**, bên trong có:

```text
Cortex-M4 processor
├── Cortex-M4 core
└── NVIC
```

Tức là:

> **Core là phần thực thi lệnh, còn processor bao gồm core và các khối hỗ trợ cần thiết để processor hoạt động trong hệ thống.**

Trong hình nguồn còn xuất hiện FPU trong phần Cortex-M4.

> **Lưu ý khi học STM32:** hình nguồn đang minh họa **Cortex-M4**. Không nên suy ra rằng mọi STM32 đều có FPU. Mỗi dòng STM32 có thể sử dụng một biến thể Cortex-M khác nhau và có tập tính năng khác nhau.

---

#### Microcontroller — vi điều khiển

**Microcontroller (MCU)** là một chip hoàn chỉnh tích hợp bộ xử lý cùng bộ nhớ và các ngoại vi.

Có thể hình dung một STM32 ở mức khái niệm:

```text
+--------------------------------------------------+
|                    STM32 MCU                     |
|                                                  |
|   +------------------+                           |
|   | ARM Cortex-M     |                           |
|   | processor        |                           |
|   +------------------+                           |
|                                                  |
|   Flash             SRAM                         |
|                                                  |
|   GPIO   Timer   UART   SPI   I2C   ADC   ...    |
+--------------------------------------------------+
```

Vì vậy cần tránh cách nói:

```text
STM32 = Cortex-M
```

Cách hiểu đúng hơn:

```text
STM32
= một vi điều khiển hoàn chỉnh

Cortex-M
= kiến trúc/lõi xử lý được tích hợp bên trong vi điều khiển đó
```

### 1.1.2. Quan hệ giữa Core và STM32

Khi chương trình firmware chạy, core là thành phần thực thi mã lệnh, nhưng nó không hoạt động độc lập.

Ví dụ:

```text
Core
 │
 ├── lấy lệnh từ Flash
 │
 ├── đọc/ghi dữ liệu trong SRAM
 │
 └── đọc/ghi thanh ghi của peripheral
          ↓
      GPIO / UART / Timer / ...
```

Core cần giao tiếp với các thành phần khác thông qua hệ thống bus.

Đây là lý do khi học kiến trúc STM32 ta không chỉ học CPU, mà còn phải hiểu:

- Flash.
- SRAM.
- Peripheral.
- Bus.
- Memory map.
- Các thanh ghi ánh xạ bộ nhớ.

---

### 1.1.3. Fetch là gì?

Tài liệu định nghĩa **fetch** là quá trình CPU lấy mã lệnh từ bộ nhớ chương trình, thường là Flash/ROM, thông qua bus để chuẩn bị giải mã và thực thi.

Có thể hình dung quá trình xử lý một lệnh ở mức cơ bản:

```text
Flash / bộ nhớ chương trình
          │
          │ Fetch
          ↓
        Core
          │
          │ Decode
          ↓
   Giải mã lệnh
          │
          │ Execute
          ↓
     Thực thi lệnh
```

Ba bước cần nhớ:

```text
Fetch
  ↓
Decode
  ↓
Execute
```

Ví dụ khi chương trình có:

```c
result = a + b;
```

Ở mức khái niệm:

```text
CPU lấy lệnh máy từ bộ nhớ
        ↓
giải mã lệnh
        ↓
đọc dữ liệu cần thiết
        ↓
ALU thực hiện phép cộng
        ↓
ghi kết quả vào vị trí phù hợp
```

Không nên hiểu rằng CPU trực tiếp thực thi dòng C `result = a + b;`.

Mã C trước đó đã được biên dịch thành các lệnh máy phù hợp với kiến trúc đích.

---

### 1.1.4. Thanh ghi trong Core

Tài liệu nhắc tới các thanh ghi:

```text
R0 → R15
xPSR
CONTROL
...
```

Thanh ghi là vùng lưu trữ rất gần với khối xử lý và được core sử dụng trực tiếp khi thực thi lệnh.

Ở mức hiện tại chỉ cần nhớ:

```text
Core
├── các thanh ghi chứa dữ liệu/tạm thời
├── các thanh ghi phục vụ luồng thực thi
└── các thanh ghi trạng thái/điều khiển
```

Chưa cần học chi tiết từng thanh ghi ở mục này.

Phần sau sẽ tách riêng:

```text
R0-R12
R13 / SP
R14 / LR
R15 / PC
xPSR
CONTROL
MSP
PSP
```

---

### 1.1.5. Core giao tiếp với bộ nhớ bằng bus

Tài liệu Cortex-M4 cung cấp ba đường bus chính:

```text
I-Code bus
D-Code bus
System bus
```

Ý nghĩa được mô tả trong tài liệu:

```text
I-Code bus
→ CPU lấy lệnh từ Flash/ROM

D-Code bus
→ CPU đọc/ghi dữ liệu từ Flash/ROM

System bus
→ CPU truy cập SRAM, peripheral và thiết bị ngoài
```

Có thể hình dung:

```text
                    +--------+
                    |  Core  |
                    +--------+
                     /   |   \
                    /    |    \
                   /     |     \
            I-Code    D-Code   System
               |         |        |
               ↓         ↓        ↓
             Flash     Flash   SRAM / Peripheral
```

Phần này mới chỉ cần hiểu:

> **Bus là đường giao tiếp giúp core trao đổi lệnh, dữ liệu và các thao tác truy cập với những thành phần khác trong hệ thống.**

Chi tiết **AHB, APB và AHB-to-APB Bridge** sẽ được tách sang mục **Bus Architecture**, tránh học quá nhiều khái niệm cùng lúc.

---

### 1.1.6. Tại sao cần nhiều bus?

Nếu mọi truy cập đều đi qua đúng một đường chung thì các hoạt động như lấy lệnh và truy cập dữ liệu có thể phải tranh chấp cùng tài nguyên truyền dẫn.

Việc Cortex-M có các đường bus phục vụ các nhóm truy cập khác nhau giúp tổ chức luồng truy cập hiệu quả hơn.

Ở mức khái niệm:

```text
Core cần:
├── lấy Instruction
├── đọc/ghi Data
└── truy cập SRAM / Peripheral
```

nên tài liệu tách thành:

```text
Instruction → I-Code
Data        → D-Code
System      → System bus
```

Không cần ghi nhớ chi tiết bus transaction ở giai đoạn này.

---

### 1.1.7. AHB và APB — chỉ cần nhận diện trước

Tài liệu nguồn còn đề cập hai loại bus:

```text
AHB
→ Advanced High-performance Bus

APB
→ Advanced Peripheral Bus
```

Ở mục này **chưa học sâu**, chỉ cần nhận diện:

```text
AHB
→ hướng tới kết nối có yêu cầu hiệu năng cao hơn

APB
→ hướng tới peripheral đơn giản hơn
```

Tài liệu mô tả APB thường được nối với AHB thông qua:

```text
AHB-to-APB Bridge
```

Do đó một đường truy cập peripheral có thể được hình dung:

```text
CPU
 ↓
AHB
 ↓
AHB-to-APB Bridge
 ↓
APB
 ↓
Peripheral
```

Ví dụ các peripheral mà tài liệu liệt kê ở phía APB gồm:

```text
UART
SPI
I2C
GPIO
Timer
Watchdog
```

> Chi tiết về băng thông, pipeline, burst transfer và cách AHB/APB được bố trí trong STM32 sẽ để sang mục **Bus Architecture**.

---

### 1.1.8. Phân biệt nhanh

| Khái niệm | Hiểu ngắn gọn |
|---|---|
| **Core** | Phần trực tiếp giải mã và thực thi lệnh. |
| **Processor** | Bao gồm core và các khối hỗ trợ xử lý/hệ thống liên quan. |
| **Microcontroller** | Chip hoàn chỉnh gồm processor + bộ nhớ + peripheral. |
| **Flash** | Thường chứa mã chương trình và dữ liệu không mất khi mất nguồn. |
| **SRAM** | Vùng RAM phục vụ dữ liệu trong lúc chương trình chạy. |
| **Peripheral** | Khối phần cứng chuyên dụng như GPIO, UART, Timer... |
| **Bus** | Hệ thống kết nối để các khối trao đổi lệnh/dữ liệu và thực hiện truy cập. |

---

### 1.1.9. Sơ đồ tổng hợp

```text
                  STM32 Microcontroller
+--------------------------------------------------+
|                                                  |
|              ARM Cortex-M Processor              |
|            +-----------------------+             |
|            |    Processor Core     |             |
|            |                       |             |
|            | ALU                   |             |
|            | Registers             |             |
|            | Decode / Execute      |             |
|            | Pipeline              |             |
|            +-----------------------+             |
|                        │                         |
|                        │ Bus                     |
|          ┌─────────────┼─────────────┐           |
|          ↓             ↓             ↓           |
|        Flash          SRAM       Peripheral      |
|                                  ├── GPIO        |
|                                  ├── Timer       |
|                                  ├── UART        |
|                                  ├── SPI         |
|                                  └── I2C         |
|                                                  |
+--------------------------------------------------+
```

---

### 1.1.10. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Cortex-M và STM32 khác nhau như thế nào?”**, có thể trả lời ngắn:

> **Cortex-M là lõi/kiến trúc xử lý ARM dùng để thực thi lệnh. STM32 là một vi điều khiển hoàn chỉnh tích hợp Cortex-M cùng Flash, SRAM và các peripheral như GPIO, Timer, UART, SPI, I2C...**

Nếu hỏi **“Core làm gì?”**:

> **Core lấy lệnh từ bộ nhớ, giải mã và thực thi lệnh. Bên trong core có ALU, các thanh ghi và các khối điều khiển thực thi.**

Nếu hỏi **“Fetch là gì?”**:

> **Fetch là quá trình CPU lấy mã lệnh từ bộ nhớ chương trình thông qua bus để chuẩn bị giải mã và thực thi.**

Nếu hỏi **“Core truy cập peripheral như thế nào?”**:

> **Core thực hiện các thao tác đọc/ghi thông qua hệ thống bus. Peripheral của vi điều khiển thường được ánh xạ vào không gian địa chỉ, nên CPU có thể truy cập các thanh ghi của peripheral thông qua địa chỉ tương ứng.**

Khái niệm **memory-mapped peripheral** sẽ được triển khai kỹ ở phần Memory Map.

---

### 1.1.11. Câu hỏi phỏng vấn tự kiểm tra

1. Processor Core là gì?
2. Processor khác Processor Core ở điểm nào?
3. Microcontroller khác Processor ở điểm nào?
4. Cortex-M nằm ở đâu trong một vi điều khiển STM32?
5. ALU có nhiệm vụ gì?
6. Core sử dụng thanh ghi để làm gì?
7. Ba giai đoạn cơ bản khi xử lý một lệnh là gì?
8. Fetch nghĩa là gì?
9. CPU thường lấy mã chương trình từ đâu trong hệ thống nhúng?
10. I-Code bus được dùng cho mục đích gì theo tài liệu?
11. D-Code bus được dùng cho mục đích gì theo tài liệu?
12. System bus được dùng cho mục đích gì theo tài liệu?
13. AHB và APB khác nhau ở ý tưởng sử dụng như thế nào?
14. AHB-to-APB Bridge có vai trò gì?
15. Tại sao không nên nói “STM32 chính là Cortex-M”?

---

### 1.1.12. Tóm tắt

```text
STM32
= Microcontroller
= Processor + Memory + Peripheral

Processor
= Core + các khối hỗ trợ

Core
= nơi trực tiếp xử lý lệnh
  ├── ALU
  ├── Registers
  ├── Decode / Execute logic
  ├── Pipeline
  └── các khối xử lý liên quan

Chương trình chạy:
Flash
  ↓ Fetch
Core
  ↓ Decode
Core
  ↓ Execute
Kết quả

Core giao tiếp với hệ thống thông qua bus:
I-Code
D-Code
System bus
```

**Ý quan trọng nhất:**

> **STM32 là một vi điều khiển hoàn chỉnh; Cortex-M là bộ xử lý/lõi xử lý được tích hợp bên trong nó. Core chịu trách nhiệm lấy, giải mã và thực thi lệnh, còn Flash, SRAM và peripheral là các thành phần khác của vi điều khiển mà core truy cập thông qua hệ thống bus.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-02"></a>
## 1.2. Operation Modes

### 1.2.1. Cortex-M có những chế độ hoạt động nào?

Theo tài liệu, các bộ xử lý Cortex-M0/M3/M4 có **2 chế độ hoạt động**:

```text
Operational Modes
├── Thread mode
└── Handler mode
```

Hai chế độ này được dùng để phân biệt hai tình huống thực thi chính:

```text
Thread mode
→ thực thi mã ứng dụng bình thường

Handler mode
→ thực thi exception handler / interrupt handler
```

Tài liệu tổng quan cũng liệt kê **Operational mode of the processor** là một trong những nội dung đặc trưng cần hiểu khi học Cortex-M0/M3/M4.

---

### 1.2.2. Thread mode

Theo tài liệu:

> Mã ứng dụng sẽ chạy trong **Thread mode**.

Có thể hình dung:

```text
Khởi động processor
        ↓
   Thread mode
        ↓
Chạy mã ứng dụng
```

Ví dụ ở mức khái niệm:

```c
int main(void)
{
    while (1)
    {
        // Mã ứng dụng
    }
}
```

Trong trạng thái chương trình đang thực thi luồng mã ứng dụng bình thường, tài liệu xếp việc thực thi đó vào **Thread mode**.

Tài liệu cũng gọi Thread mode là **"User Mode"**.

> **Phạm vi của mục này:** ở đây chỉ ghi nhận cách gọi trong tài liệu nguồn.  
> Phần **Access Level** sẽ được học riêng ở mục 1.3, nên chưa trộn khái niệm quyền truy cập vào Operation Modes.

---

### 1.2.3. Handler mode

Theo tài liệu:

> Tất cả **exception handler** hoặc **interrupt handler** sẽ chạy trong **Handler mode**.

Có thể hình dung:

```text
Exception / Interrupt xảy ra
          ↓
       Core
          ↓
   Handler mode
          ↓
Chạy handler / ISR tương ứng
```

Ví dụ về mặt ý tưởng:

```c
void Some_IRQHandler(void)
{
    // Mã xử lý ngắt
}
```

Điểm cần nhớ:

```text
Mã ứng dụng bình thường
→ Thread mode

Exception handler / Interrupt handler
→ Handler mode
```

---

### 1.2.4. Processor bắt đầu ở mode nào?

Theo tài liệu:

> Processor luôn bắt đầu ở **Thread mode**.

Do đó, ở mức khái niệm có thể ghi nhớ:

```text
Processor bắt đầu hoạt động
          ↓
      Thread mode
```

Ở mục này chưa đi sâu vào **Reset Sequence** hay startup code; các nội dung đó sẽ được triển khai riêng ở phần sau.

---

### 1.2.5. Khi nào Core chuyển từ Thread mode sang Handler mode?

Tài liệu mô tả:

- Khi core gặp **system exception**, hoặc
- Khi có **external interrupt**,

thì core sẽ chuyển sang **Handler mode** để phục vụ handler/ISR tương ứng.

Sơ đồ:

```text
              Thread mode
                  │
                  │ System exception
                  │ hoặc External interrupt
                  ↓
              Handler mode
                  │
                  ↓
          Thực thi handler / ISR
```

Có thể nhớ ngắn gọn:

```text
Thread mode
    ↓ exception / interrupt
Handler mode
```

Phần tài liệu Operation Modes hiện tại chỉ mô tả việc chuyển sang Handler mode để phục vụ exception/interrupt. Cơ chế exception chi tiết sẽ được học ở phần **Exception / Interrupt** sau này.

---

### 1.2.6. System exception và external interrupt trong mục này cần hiểu đến đâu?

Ở mục **Operation Modes**, chưa cần học chi tiết từng loại exception hoặc interrupt.

Chỉ cần hiểu mối liên hệ:

```text
Sự kiện xảy ra
      ↓
System exception
hoặc
External interrupt
      ↓
Core chuyển sang Handler mode
      ↓
Handler / ISR được thực thi
```

Các nội dung như:

```text
NVIC
Interrupt priority
Exception number
HardFault
SysTick
PendSV
SVC
...
```

sẽ được để sang phần chuyên về **Interrupt + NVIC + EXTI** và phần exception tương ứng.

---

### 1.2.7. Phân biệt nhanh Thread mode và Handler mode

| Tiêu chí | Thread mode | Handler mode |
|---|---|---|
| Mục đích chính theo tài liệu | Chạy mã ứng dụng | Chạy exception/interrupt handler |
| Mã thường chạy | Application code | Handler / ISR |
| Processor bắt đầu ở đây | Có | Không |
| Khi exception/interrupt xảy ra | Core rời Thread mode để xử lý sự kiện | Core chuyển vào mode này để phục vụ handler |

Cách nhớ:

```text
Thread
→ chương trình đang chạy bình thường

Handler
→ CPU đang xử lý exception / interrupt
```

---

### 1.2.8. Sơ đồ tổng hợp

```text
              Processor bắt đầu
                     │
                     ↓
                Thread mode
                     │
                     │ Chạy application code
                     │
                     ├───────────────┐
                     │               │
                     │        Exception / Interrupt
                     │               │
                     │               ↓
                     │          Handler mode
                     │               │
                     │               ↓
                     │          Handler / ISR
                     │
                     └── Luồng hoạt động bình thường
```

Ý chính của sơ đồ:

```text
Thread mode
= mã ứng dụng

Handler mode
= mã xử lý exception / interrupt
```

---

### 1.2.9. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Cortex-M có những operation mode nào?”**, có thể trả lời:

> **Theo mô hình Cortex-M trong tài liệu, processor có hai operation mode là Thread mode và Handler mode. Mã ứng dụng chạy trong Thread mode, còn exception handler và interrupt handler chạy trong Handler mode.**

Nếu hỏi **“Processor bắt đầu ở mode nào?”**:

> **Processor bắt đầu ở Thread mode.**

Nếu hỏi **“Khi interrupt xảy ra thì mode thay đổi như thế nào?”**:

> **Khi core gặp system exception hoặc external interrupt, core chuyển từ Thread mode sang Handler mode để thực thi handler hoặc ISR tương ứng.**

Nếu hỏi **“Thread mode và Handler mode khác nhau ở điểm chính nào?”**:

> **Thread mode dành cho luồng mã ứng dụng bình thường, còn Handler mode dành cho việc xử lý exception và interrupt.**

---

### 1.2.10. Câu hỏi phỏng vấn tự kiểm tra

1. Cortex-M0/M3/M4 có bao nhiêu operation mode theo tài liệu?
2. Hai operation mode đó là gì?
3. Mã ứng dụng bình thường chạy trong mode nào?
4. Exception handler chạy trong mode nào?
5. Interrupt handler chạy trong mode nào?
6. Processor bắt đầu ở mode nào?
7. Khi system exception xảy ra, core chuyển sang mode nào?
8. Khi external interrupt xảy ra, core chuyển sang mode nào?
9. Thread mode và Handler mode khác nhau ở mục đích chính như thế nào?
10. Trong phần Operation Modes này, tại sao chưa cần đi sâu vào NVIC hay interrupt priority?

---

### 1.2.11. Tóm tắt

```text
Cortex-M Operational Modes
│
├── Thread mode
│   ├── processor bắt đầu ở mode này
│   └── chạy application code
│
└── Handler mode
    └── chạy exception / interrupt handler
```

Khi có sự kiện:

```text
Thread mode
     ↓
System exception
hoặc External interrupt
     ↓
Handler mode
     ↓
Handler / ISR
```

**Ý quan trọng nhất:**

> **Thread mode được dùng cho mã ứng dụng, còn Handler mode được dùng để phục vụ exception và interrupt. Khi core gặp system exception hoặc external interrupt, nó chuyển sang Handler mode để thực thi handler tương ứng.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-03"></a>
## 1.3. Access Level

### 1.3.1. Cortex-M có những mức truy cập nào?

Theo tài liệu, Cortex-M0/M3/M4 cung cấp **2 mức truy cập**:

```text
Access Levels
├── Privileged Access Level (PAL)
└── Non-Privileged Access Level (NPAL)
```

Có thể dịch ngắn gọn:

```text
PAL
→ mức truy cập đặc quyền

NPAL
→ mức truy cập không đặc quyền
```

Mục đích chính của hai mức này là phân biệt **mức quyền mà mã đang chạy có đối với tài nguyên của processor**.

---

### 1.3.2. Privileged Access Level — mức truy cập đặc quyền

Theo tài liệu, khi mã chạy ở **Privileged Access Level (PAL)** thì nó có quyền truy cập đầy đủ hơn đối với:

- Tài nguyên đặc thù của processor.
- Các thanh ghi bị hạn chế truy cập.

Có thể hình dung:

```text
Privileged
     ↓
Có quyền truy cập đầy đủ hơn
     ↓
Processor resources
Restricted registers
```

Ở mức phỏng vấn, có thể nhớ:

> **Privileged mode/access level cho phép mã truy cập các tài nguyên và thanh ghi hệ thống mà mã không đặc quyền có thể bị hạn chế.**

---

### 1.3.3. Non-Privileged Access Level — mức truy cập không đặc quyền

Theo tài liệu, khi mã chạy ở **Non-Privileged Access Level (NPAL)** thì mã có thể **không được phép truy cập một số thanh ghi bị hạn chế của processor**.

Có thể hình dung:

```text
Non-Privileged
      ↓
Quyền bị hạn chế
      ↓
Một số restricted registers
không được truy cập
```

Điểm quan trọng:

> **Non-Privileged không có nghĩa là CPU ngừng chạy mã ứng dụng. Nó chỉ có nghĩa là mã đó đang chạy với mức quyền thấp hơn.**

---

### 1.3.4. Mức truy cập mặc định

Theo tài liệu:

> **Mặc định, mã bắt đầu chạy ở Privileged Access Level.**

Có thể ghép với phần Operation Modes trước đó:

```text
Processor bắt đầu
      ↓
Thread mode
      ↓
Privileged Access Level
```

Như vậy, ở trạng thái khởi đầu theo tài liệu:

```text
Operational mode = Thread mode
Access level     = Privileged
```

Đây là hai khái niệm khác nhau:

```text
Operation Mode
→ Thread / Handler

Access Level
→ Privileged / Non-Privileged
```

---

### 1.3.5. Thread mode có thể chạy ở mức truy cập nào?

Trong **Thread mode**, processor có thể chạy ở:

```text
Thread mode
├── Privileged
└── Non-Privileged
```

Theo tài liệu, khi đang ở Thread mode và Privileged, chương trình có thể chuyển processor sang Non-Privileged.

Có thể hình dung:

```text
Thread mode
Privileged
    │
    │ thay đổi CONTROL
    ↓
Thread mode
Non-Privileged
```

Đây là điểm rất quan trọng vì:

> **Thread mode không đồng nghĩa với Non-Privileged.**

Thread mode có thể là:

```text
Thread + Privileged
```

hoặc:

```text
Thread + Non-Privileged
```

---

### 1.3.6. Handler mode luôn chạy ở mức nào?

Theo tài liệu:

> **Handler mode luôn chạy ở Privileged Access Level.**

Có thể nhớ:

```text
Handler mode
     ↓
Always Privileged
```

Do đó:

```text
Thread mode
→ có thể Privileged hoặc Non-Privileged

Handler mode
→ luôn Privileged
```

Đây là mối liên hệ quan trọng giữa **Operation Mode** và **Access Level**.

---

### 1.3.7. Vai trò của thanh ghi `CONTROL`

Theo tài liệu, processor sử dụng thanh ghi:

```text
CONTROL
```

để chuyển đổi mức truy cập khi ở Thread mode.

Hình minh họa sử dụng cách viết rút gọn:

```text
CONTROL = 0
→ Privileged

CONTROL = 1
→ Non-Privileged
```

Ở phần này chỉ cần hiểu ý tưởng:

```text
CONTROL
→ ảnh hưởng tới access level của Thread mode
```

Chi tiết các bit cụ thể trong thanh ghi `CONTROL` sẽ được học ở phần **Core Registers** để tránh trộn quá nhiều kiến thức vào một mục.

---

### 1.3.8. Tại sao từ Non-Privileged không thể tự quay lại Privileged?

Theo tài liệu, khi Thread mode đã chuyển từ:

```text
Privileged
    ↓
Non-Privileged
```

thì mã đang chạy ở Non-Privileged **không thể tự chuyển trực tiếp trở lại Privileged**.

Lý do về mặt ý tưởng:

```text
Nếu mã không đặc quyền
có thể tự nâng quyền
        ↓
cơ chế phân quyền
sẽ mất ý nghĩa
```

Do đó, tài liệu mô tả con đường quay lại Privileged thông qua **Handler mode**.

---

### 1.3.9. Từ Non-Privileged quay lại Privileged như thế nào?

Theo tài liệu và hình minh họa:

```text
Thread mode
Non-Privileged
      │
      │ Interrupt / Exception
      ↓
Handler mode
Privileged
      │
      │ thay đổi CONTROL
      ↓
Return to Thread mode
Privileged
```

Có thể hình dung đầy đủ:

```text
Thread mode
Privileged
    │
    │ CONTROL = 1
    ↓
Thread mode
Non-Privileged
    │
    │ Exception / Interrupt
    ↓
Handler mode
Privileged
    │
    │ CONTROL = 0
    ↓
Thread mode
Privileged
```

Điểm cốt lõi:

> **Handler mode luôn có quyền Privileged, nên handler có thể thực hiện thao tác cần quyền cao rồi đưa Thread mode trở về mức Privileged theo cơ chế mô tả trong tài liệu.**

---

### 1.3.10. Ghép Operation Mode và Access Level

Đây là phần dễ nhầm nhất.

Không nên nghĩ:

```text
Thread mode  = Non-Privileged
Handler mode = Privileged
```

Cách hiểu đúng theo tài liệu:

| Operation Mode | Access Level có thể có |
|---|---|
| Thread mode | Privileged hoặc Non-Privileged |
| Handler mode | Privileged |

Sơ đồ:

```text
                    Processor
                        │
          ┌─────────────┴─────────────┐
          ↓                           ↓
     Thread mode                 Handler mode
          │                           │
     ┌────┴────┐                      ↓
     ↓         ↓                  Privileged
Privileged  Non-Privileged
```

Vì vậy:

```text
Operation Mode
≠
Access Level
```

Hai cơ chế này có liên quan nhưng không phải cùng một khái niệm.

---

### 1.3.11. Luồng chuyển trạng thái theo hình tài liệu

Hình nguồn có thể được diễn giải thành chuỗi sau:

```text
1. Processor bắt đầu:
   Thread mode + Privileged

2. Thread mode chuyển:
   Privileged → Non-Privileged

3. Khi có interrupt/exception:
   Thread mode → Handler mode

4. Handler mode:
   luôn Privileged

5. Handler có thể thiết lập lại CONTROL theo cơ chế
   được minh họa trong tài liệu

6. Khi thoát handler:
   quay lại Thread mode
```

Sơ đồ tổng hợp:

```text
Thread / Privileged
        │
        │ CONTROL = 1
        ↓
Thread / Non-Privileged
        │
        │ Exception / Interrupt
        ↓
Handler / Privileged
        │
        │ CONTROL = 0
        ↓
Thread / Privileged
```

---

### 1.3.12. Tại sao cần Access Level?

Từ nội dung tài liệu có thể rút ra ý nghĩa chính:

```text
Privileged
→ truy cập tài nguyên hệ thống đầy đủ hơn

Non-Privileged
→ hạn chế truy cập một số tài nguyên/thanh ghi
```

Nhờ vậy, mã ứng dụng có thể được chạy với quyền thấp hơn, trong khi mã xử lý hệ thống/exception vẫn chạy với quyền cao hơn.

Ở mức Intern chỉ cần nắm:

> **Access Level là cơ chế kiểm soát quyền truy cập của mã đang chạy đối với các tài nguyên nhạy cảm của processor.**

---

### 1.3.13. Phân biệt nhanh Operation Mode và Access Level

| Khái niệm | Câu hỏi nó trả lời |
|---|---|
| **Operation Mode** | Processor đang chạy mã ứng dụng hay đang xử lý exception/interrupt? |
| **Access Level** | Mã hiện tại có mức quyền truy cập cao hay bị hạn chế? |

Ví dụ:

```text
Thread mode + Privileged
→ mã ứng dụng đang chạy với quyền cao

Thread mode + Non-Privileged
→ mã ứng dụng đang chạy với quyền bị hạn chế

Handler mode + Privileged
→ processor đang xử lý exception/interrupt với quyền cao
```

---

### 1.3.14. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Cortex-M có những access level nào?”**, có thể trả lời:

> **Có hai mức truy cập: Privileged và Non-Privileged. Privileged có quyền truy cập đầy đủ hơn vào tài nguyên và các thanh ghi bị hạn chế của processor, còn Non-Privileged bị giới hạn một số quyền truy cập.**

Nếu hỏi **“Thread mode có luôn là Non-Privileged không?”**:

> **Không. Thread mode có thể chạy ở Privileged hoặc Non-Privileged.**

Nếu hỏi **“Handler mode chạy ở access level nào?”**:

> **Handler mode luôn chạy ở Privileged Access Level theo tài liệu.**

Nếu hỏi **“Tại sao mã Non-Privileged không thể tự nâng quyền trở lại?”**:

> **Vì nếu mã không đặc quyền có thể tự chuyển thành đặc quyền thì cơ chế giới hạn quyền sẽ không còn ý nghĩa. Theo tài liệu, muốn quay lại Privileged cần đi qua Handler mode.**

Nếu hỏi **“Thanh ghi nào liên quan tới việc chuyển access level?”**:

> **Thanh ghi `CONTROL`. Tài liệu minh họa `CONTROL = 0` cho Privileged và `CONTROL = 1` cho Non-Privileged; chi tiết từng bit sẽ học ở phần Core Registers.**

---

### 1.3.15. Câu hỏi phỏng vấn tự kiểm tra

1. Cortex-M0/M3/M4 có bao nhiêu access level theo tài liệu?
2. Hai access level đó là gì?
3. Privileged Access Level cho phép truy cập những gì?
4. Non-Privileged Access Level bị hạn chế điều gì?
5. Access level mặc định là gì?
6. Thread mode có thể chạy ở những access level nào?
7. Handler mode chạy ở access level nào?
8. Operation Mode và Access Level có phải cùng một khái niệm không?
9. Thanh ghi nào được tài liệu sử dụng để chuyển access level?
10. Từ Thread/Privileged có thể chuyển sang Thread/Non-Privileged như thế nào ở mức khái niệm?
11. Vì sao Thread/Non-Privileged không thể tự chuyển trực tiếp về Privileged?
12. Exception/interrupt có vai trò gì trong quá trình quay lại Privileged theo tài liệu?
13. Sau khi vào Handler mode, access level là gì?
14. Hãy phân biệt `Thread + Privileged`, `Thread + Non-Privileged` và `Handler + Privileged`.

---

### 1.3.16. Tóm tắt

```text
Access Levels
│
├── Privileged
│   └── quyền truy cập đầy đủ hơn
│
└── Non-Privileged
    └── bị hạn chế một số quyền truy cập
```

Kết hợp với Operation Modes:

```text
Thread mode
├── Privileged
└── Non-Privileged

Handler mode
└── Privileged
```

Luồng quan trọng:

```text
Thread / Privileged
        ↓
   CONTROL = 1
        ↓
Thread / Non-Privileged
        ↓
Exception / Interrupt
        ↓
Handler / Privileged
        ↓
   CONTROL = 0
        ↓
Thread / Privileged
```

**Ý quan trọng nhất:**

> **Operation Mode cho biết processor đang chạy luồng ứng dụng hay handler; Access Level cho biết mức quyền truy cập của mã đang chạy. Thread mode có thể Privileged hoặc Non-Privileged, còn Handler mode luôn Privileged theo tài liệu.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-04"></a>
## 1.4. Core Registers

### 1.4.1. Tổng quan các thanh ghi của Processor Core

Theo sơ đồ tài liệu, các thanh ghi của processor core được chia thành các nhóm chính:

```text
Core Registers
│
├── General-purpose registers
│   ├── R0
│   ├── R1
│   ├── ...
│   └── R12
│
├── R13
│   └── SP — Stack Pointer
│       ├── PSP
│       └── MSP
│
├── R14
│   └── LR — Link Register
│
├── R15
│   └── PC — Program Counter
│
└── Special registers
    ├── PSR
    ├── PRIMASK
    ├── FAULTMASK
    ├── BASEPRI
    └── CONTROL
```

Trong sơ đồ nguồn:

- `R0` đến `R12` được gọi là **general-purpose registers**.
- `R0` đến `R7` được đánh dấu là **low registers**.
- `R8` đến `R12` được đánh dấu là **high registers**.
- `R13` là **Stack Pointer (SP)**.
- `R14` là **Link Register (LR)**.
- `R15` là **Program Counter (PC)**.

Các thanh ghi phía dưới như `PSR`, `PRIMASK`, `FAULTMASK`, `BASEPRI`, `CONTROL` được nhóm thành các **special registers**.

---

### 1.4.2. R0 → R12 — General-Purpose Registers

Theo sơ đồ tài liệu:

```text
R0  ┐
R1  │
R2  │
R3  │
R4  ├── General-purpose registers
R5  │
R6  │
R7  │
R8  │
R9  │
R10 │
R11 │
R12 ┘
```

Trong đó:

```text
R0 → R7
→ Low registers

R8 → R12
→ High registers
```

Ở mục này chỉ cần nhớ:

> **R0 đến R12 là các thanh ghi đa dụng mà core dùng trong quá trình xử lý dữ liệu và thực thi chương trình.**

Chi tiết quy ước thanh ghi nào thường dùng để truyền tham số, giữ biến cục bộ hay phải được caller/callee bảo toàn chưa được hình nguồn này mô tả đầy đủ, nên chưa triển khai ở đây.

---

### 1.4.3. R13 — Stack Pointer

Theo sơ đồ:

```text
R13
 ↓
SP — Stack Pointer
```

Tài liệu còn cho thấy `SP` có hai phiên bản:

```text
SP
├── PSP
└── MSP
```

và ghi chú đây là **banked version of SP**.

Có thể hình dung:

```text
R13 / SP
   │
   ├── PSP
   │
   └── MSP
```

Ở phần Core Registers hiện tại, chỉ cần ghi nhận rằng:

> **R13 là Stack Pointer và tài liệu thể hiện hai phiên bản của Stack Pointer là PSP và MSP.**

Cách PSP/MSP được chọn, mode nào dùng SP nào và cách Stack hoạt động sẽ được triển khai riêng trong phần **Stack cơ bản trên Cortex-M**.

---

### 1.4.4. R14 — Link Register

Theo sơ đồ:

```text
R14
 ↓
LR — Link Register
```

Hình Caller/Callee minh họa vai trò của `LR` khi một hàm gọi hàm khác.

Ví dụ:

```c
void fun1(void)
{
    fun1_ins_1;

    fun2();

    fun1_ins_2;
}
```

và:

```c
void fun2(void)
{
    fun2_ins_1;
    fun2_ins_2;
}
```

Theo hình:

```text
fun1 gọi fun2
      ↓
PC nhảy tới địa chỉ của fun2
      ↓
LR giữ địa chỉ quay về
      ↓
fun2 thực thi
      ↓
khi hàm kết thúc:
PC = LR
      ↓
quay lại fun1
```

Hình mô tả:

```text
LR = return address
   = địa chỉ của lệnh tiếp theo
```

Có thể hình dung:

```text
fun1
│
├── fun1_ins_1
│
├── gọi fun2
│      │
│      ├── LR ← địa chỉ quay về
│      └── PC ← địa chỉ fun2
│
│   fun2
│   ├── fun2_ins_1
│   ├── fun2_ins_2
│   └── return
│         ↓
│       PC = LR
│
└── fun1_ins_2
```

Ý cần nhớ:

> **LR giữ thông tin địa chỉ quay về khi thực hiện lời gọi hàm theo mô hình minh họa trong tài liệu.**

---

### 1.4.5. R15 — Program Counter

Theo tài liệu:

```text
R15
 ↓
PC — Program Counter
```

Hình Program Counter ghi rõ:

> `PC` là thanh ghi `R15` và chứa địa chỉ chương trình hiện tại.

Có thể hiểu ở mức khái niệm:

```text
PC
→ cho core biết vị trí lệnh trong luồng chương trình
```

Trong hình Caller/Callee:

```text
fun1 gọi fun2
      ↓
PC nhảy tới địa chỉ của fun2
```

Khi `fun2` kết thúc, hình minh họa:

```text
PC = LR
```

để quay về tiếp tục thực thi `fun1`.

Do đó mối liên hệ quan trọng giữa PC và LR là:

```text
Gọi hàm:
PC → địa chỉ hàm được gọi
LR → giữ địa chỉ quay về

Return:
PC ← LR
```

---

### 1.4.6. Program Counter khi Reset

Hình tài liệu về `Program Counter` còn mô tả quá trình Reset:

```text
PC = R15
```

và khi reset:

> Processor nạp `PC` bằng giá trị của **reset vector** tại địa chỉ `0x00000004`.

Có thể hình dung:

```text
Reset
  ↓
đọc giá trị tại 0x00000004
  ↓
nạp giá trị đó vào PC
  ↓
bắt đầu thực thi tại địa chỉ Reset Handler
```

Tài liệu cũng ghi:

```text
Bit[0] của giá trị reset vector
→ được nạp vào T-bit của EPSR lúc reset
→ bit này phải bằng 1
```

Ở mục Core Registers chỉ cần ghi nhận thông tin trên.

Cơ chế Reset Vector và trình tự khởi động sẽ được triển khai kỹ hơn trong phần **Reset Sequence**.

---

### 1.4.7. PSR — Program Status Register

Theo sơ đồ Core Registers:

```text
PSR
→ Program status register
```

PSR được xếp vào nhóm **special registers**.

Trong phạm vi hình nguồn hiện tại, tài liệu chưa giải thích chi tiết từng trường bit của PSR, vì vậy ở đây chỉ cần nhớ:

> **PSR là thanh ghi trạng thái chương trình của processor.**

Chi tiết cấu trúc PSR sẽ chỉ nên bổ sung khi có tài liệu nguồn tương ứng.

---

### 1.4.8. PRIMASK, FAULTMASK và BASEPRI

Theo sơ đồ:

```text
PRIMASK
FAULTMASK
BASEPRI
```

được nhóm chung dưới nhãn:

```text
Exception mask registers
```

Do đó, ở mức tài liệu hiện có:

> **PRIMASK, FAULTMASK và BASEPRI là các thanh ghi liên quan đến việc mask exception.**

Phần hình nguồn chưa mô tả cụ thể từng thanh ghi mask loại exception nào hay cách đặt bit, nên chưa đi sâu hơn trong mục này.

Nội dung chi tiết phù hợp hơn với phần **Interrupt / Exception** sau này.

---

### 1.4.9. CONTROL Register

Theo sơ đồ:

```text
CONTROL
→ CONTROL register
```

và nó thuộc nhóm **special registers**.

Mục Access Level trước đó đã sử dụng `CONTROL` để mô tả việc thay đổi mức truy cập trong Thread mode.

Trong phần Core Registers, chỉ cần nối lại kiến thức:

```text
CONTROL
→ một special register của processor core
→ có liên quan tới trạng thái điều khiển processor
→ trong tài liệu Access Level, được dùng để chuyển
  Thread mode giữa Privileged và Non-Privileged
```

Chi tiết từng bit trong `CONTROL` chưa xuất hiện trong nhóm hình Core Registers hiện tại nên chưa bổ sung ngoài phạm vi nguồn.

---

### 1.4.10. Non-Memory-Mapped Registers là gì?

Một hình trong tài liệu phân biệt:

```text
Non-memory mapped registers
```

và:

```text
Memory mapped registers
```

Các **processor core registers** được đặt phía **Non-memory mapped registers**.

Tài liệu ghi:

- Các core register này **không có địa chỉ duy nhất để truy cập như một địa chỉ trong memory map**.
- Vì vậy chúng **không thuộc processor memory map**.
- Không thể truy cập chúng trong chương trình C bằng cách lấy một địa chỉ cố định rồi giải tham chiếu như với peripheral register.
- Tài liệu chỉ ra rằng để truy cập trực tiếp các thanh ghi này cần sử dụng instruction phù hợp ở mức Assembly.

Có thể hình dung:

```text
R0, R1, ..., R15
PSR
PRIMASK
FAULTMASK
BASEPRI
CONTROL

→ Core Registers
→ Non-Memory-Mapped
→ không truy cập theo kiểu:

volatile uint32_t *reg = (uint32_t *)ADDRESS;
```

Điểm quan trọng:

> **Core register và peripheral register không phải cùng một loại register về cách được đặt trong không gian địa chỉ.**

---

### 1.4.11. Memory-Mapped Registers là gì?

Hình tài liệu đặt ở phía **Memory mapped registers** hai nhóm:

```text
Processor-specific peripheral registers
├── NVIC
├── MPU
├── SCB
├── DEBUG
└── ...

Microcontroller-specific peripheral registers
├── RTC
├── I2C
├── TIMER
├── CAN
├── USB
└── ...
```

Theo tài liệu:

> **Mỗi memory-mapped register có một địa chỉ trong processor memory map.**

Do đó chương trình C có thể truy cập chúng bằng cơ chế địa chỉ và giải tham chiếu.

Ý tưởng:

```c
volatile uint32_t *reg =
    (volatile uint32_t *)ADDRESS;

uint32_t value = *reg;
```

Sơ đồ:

```text
Memory Map
    │
    ├── địa chỉ A → register của peripheral
    ├── địa chỉ B → register khác
    └── ...
```

CPU đọc/ghi vào địa chỉ đó:

```text
CPU
 ↓
địa chỉ trong Memory Map
 ↓
Memory-Mapped Register
 ↓
Peripheral
```

Khái niệm này sẽ được triển khai sâu hơn ở mục **Memory-Mapped I/O**.

---

### 1.4.12. Core Register và Memory-Mapped Register khác nhau như thế nào?

| Tiêu chí | Core Register | Memory-Mapped Register |
|---|---|---|
| Ví dụ trong tài liệu | `R0-R15`, `PSR`, `CONTROL`... | NVIC, MPU, SCB, RTC, I2C, TIMER... |
| Có địa chỉ trong memory map | Không theo cách tài liệu mô tả | Có |
| Truy cập bằng địa chỉ + giải tham chiếu trong C | Không | Có |
| Thuộc processor core | Có | Thường là thanh ghi của khối/peripheral |

Cách nhớ:

```text
Core Register
→ nằm trong core
→ dùng trực tiếp bởi instruction
→ không phải một ô địa chỉ trong memory map

Peripheral Register
→ được ánh xạ vào memory map
→ có địa chỉ
→ C có thể đọc/ghi thông qua địa chỉ
```

---

### 1.4.13. Quan hệ giữa PC và LR khi gọi hàm

Đây là phần rất dễ được hỏi trong phỏng vấn.

Theo hình:

```text
Caller = fun1
Callee = fun2
```

Khi `fun1` gọi `fun2`:

```text
Trước lời gọi:

PC → lệnh trong fun1

        ↓ gọi fun2()

LR ← địa chỉ lệnh tiếp theo của fun1
PC ← địa chỉ của fun2

        ↓

fun2 thực thi

        ↓ return

PC ← LR

        ↓

fun1 tiếp tục tại lệnh sau lời gọi
```

Sơ đồ ngắn:

```text
Caller
  │
  │ call
  ↓
Callee
  │
  │ return
  ↓
Caller

LR = return address
PC = địa chỉ đang/tiếp tục được thực thi
```

---

### 1.4.14. Sơ đồ tổng hợp Core Registers

```text
                    Processor Core Registers
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ↓                   ↓                   ↓
   General-purpose       Control Flow         Special
      Registers           Registers           Registers
          │                   │                   │
       R0-R12             R13 = SP             PSR
                              │                 PRIMASK
                         ┌────┴────┐            FAULTMASK
                         ↓         ↓             BASEPRI
                        PSP       MSP            CONTROL

                         R14 = LR
                         R15 = PC
```

Mối liên hệ khi gọi hàm:

```text
PC
↓
địa chỉ hàm đang chạy

LR
↓
địa chỉ quay về
```

Mối liên hệ với Memory Map:

```text
Core Registers
→ Non-Memory-Mapped

Peripheral Registers
→ Memory-Mapped
```

---

### 1.4.15. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Các core register chính của Cortex-M là gì?”**, có thể trả lời:

> **Theo sơ đồ tài liệu, Cortex-M có các general-purpose register R0-R12, R13 là Stack Pointer, R14 là Link Register, R15 là Program Counter, cùng các special register như PSR, PRIMASK, FAULTMASK, BASEPRI và CONTROL.**

Nếu hỏi **“R14/LR dùng để làm gì?”**:

> **Trong mô hình gọi hàm của tài liệu, LR giữ địa chỉ quay về. Khi caller gọi callee, PC chuyển tới hàm được gọi còn LR giữ địa chỉ của lệnh tiếp theo để khi return có thể quay lại caller.**

Nếu hỏi **“R15/PC dùng để làm gì?”**:

> **PC là Program Counter, tức R15, chứa địa chỉ chương trình đang được processor dùng để điều khiển luồng thực thi. Khi gọi hàm PC chuyển tới địa chỉ callee, và hình tài liệu minh họa khi return thì PC nhận lại giá trị từ LR.**

Nếu hỏi **“Core register có nằm trong memory map không?”**:

> **Theo tài liệu, các core register như R0-R15 là non-memory-mapped register, không có địa chỉ riêng trong processor memory map như peripheral register.**

Nếu hỏi **“Memory-mapped register là gì?”**:

> **Là register có địa chỉ trong memory map. Tài liệu đưa ví dụ các register của NVIC, MPU, SCB hoặc peripheral của vi điều khiển như I2C, Timer, CAN, USB; chương trình C có thể truy cập chúng thông qua địa chỉ tương ứng.**

Nếu hỏi **“R13 có gì đặc biệt?”**:

> **R13 là Stack Pointer và tài liệu cho thấy nó có hai phiên bản banked là PSP và MSP.**

---

### 1.4.16. Câu hỏi phỏng vấn tự kiểm tra

1. R0 đến R12 thuộc nhóm register nào?
2. R0-R7 và R8-R12 được sơ đồ gọi là gì?
3. R13 là register gì?
4. Hai phiên bản Stack Pointer được tài liệu thể hiện là gì?
5. R14 là register gì?
6. R15 là register gì?
7. Khi caller gọi callee, LR giữ thông tin gì?
8. Khi gọi hàm, PC thay đổi như thế nào theo hình?
9. Khi hàm return, mối quan hệ `PC = LR` trong hình có ý nghĩa gì?
10. Khi reset, tài liệu nói PC được nạp từ địa chỉ nào?
11. `PSR` được tài liệu mô tả là loại register gì?
12. `PRIMASK`, `FAULTMASK` và `BASEPRI` được nhóm thành loại register gì?
13. `CONTROL` thuộc nhóm register nào?
14. Processor core register có thuộc memory map không theo tài liệu?
15. Peripheral register memory-mapped khác core register ở điểm nào?
16. Hãy kể các ví dụ processor-specific peripheral register trong hình.
17. Hãy kể các ví dụ microcontroller-specific peripheral register trong hình.
18. Vì sao có thể truy cập memory-mapped register trong C bằng địa chỉ?
19. Hãy phân biệt ngắn gọn `SP`, `LR` và `PC`.
20. Hãy mô tả luồng caller → callee → caller bằng `PC` và `LR`.

---

### 1.4.17. Tóm tắt

```text
R0-R12
→ General-purpose registers

R13
→ SP
→ PSP / MSP

R14
→ LR
→ giữ return address theo mô hình lời gọi hàm

R15
→ PC
→ Program Counter
```

Special registers trong sơ đồ:

```text
PSR
PRIMASK
FAULTMASK
BASEPRI
CONTROL
```

Khi gọi hàm:

```text
Caller
  ↓ call

LR ← return address
PC ← callee address

Callee
  ↓ return

PC ← LR

Caller tiếp tục
```

Phân biệt loại register:

```text
Core Registers
→ Non-Memory-Mapped
→ không có địa chỉ riêng trong processor memory map

Peripheral Registers
→ Memory-Mapped
→ có địa chỉ trong memory map
→ có thể truy cập trong C bằng địa chỉ
```

**Ý quan trọng nhất:**

> **R0-R12 là các general-purpose register, R13 là Stack Pointer, R14 là Link Register và R15 là Program Counter. Các core register này được tài liệu xếp vào nhóm non-memory-mapped, khác với các register của peripheral được ánh xạ vào memory map và có thể truy cập bằng địa chỉ.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-05"></a>
## 1.5. Reset Sequence

### 1.5.1. Reset Sequence là gì?

**Reset Sequence** là chuỗi bước processor thực hiện ngay sau khi reset để:

```text
Khởi tạo Stack Pointer
        ↓
Xác định địa chỉ Reset Handler
        ↓
Nhảy tới Reset Handler
        ↓
Thực hiện các bước khởi tạo
        ↓
Gọi main()
```

Phần này nối trực tiếp với mục Core Registers trước đó vì Reset Sequence sử dụng hai thanh ghi rất quan trọng:

```text
MSP
→ Main Stack Pointer

PC
→ Program Counter
```

---

### 1.5.2. Hai giá trị đầu tiên trong vùng vector

Theo tài liệu, sau reset processor đọc hai vị trí bộ nhớ đầu tiên:

```text
0x00000000
0x00000004
```

Ý nghĩa theo tài liệu:

```text
Địa chỉ 0x00000000
→ chứa giá trị khởi tạo cho MSP

Địa chỉ 0x00000004
→ chứa địa chỉ của Reset Handler
```

Có thể hình dung:

```text
Memory Address        Nội dung
────────────────────────────────────────
0x00000000        →   Initial MSP value
0x00000004        →   Reset Handler address
```

Đây là phần đầu của **vector table** mà processor sử dụng khi reset.

---

### 1.5.3. Bước 1 — Processor bắt đầu chuỗi reset

Tài liệu mô tả bước đầu tiên bằng việc processor bắt đầu tại vùng địa chỉ:

```text
0x00000000
```

Sau đó processor dùng các giá trị đầu tiên ở vùng vector để khởi tạo trạng thái cần thiết trước khi chạy Reset Handler.

Ở mức học hiện tại, ý quan trọng không phải là ghi nhớ cách diễn đạt từng vi bước của PC, mà là nhớ thứ tự:

```text
Reset
  ↓
đọc vector đầu tiên
  ↓
khởi tạo MSP
  ↓
đọc vector tiếp theo
  ↓
nạp địa chỉ Reset Handler vào PC
```

---

### 1.5.4. Bước 2 — Khởi tạo MSP

Theo tài liệu:

```text
MSP = value @ 0x00000000
```

Trong đó:

```text
MSP
= Main Stack Pointer
```

Tài liệu nhấn mạnh:

> Processor trước tiên khởi tạo Stack Pointer.

Ví dụ minh họa trong hình:

```text
Memory[0x00000000] = 0x20008000
```

thì:

```text
MSP = 0x20008000
```

Sơ đồ:

```text
0x00000000
     │
     │ đọc giá trị
     ↓
0x20008000
     │
     ↓
MSP = 0x20008000
```

Địa chỉ `0x20008000` trong hình chỉ là **giá trị minh họa** cho initial MSP.

---

### 1.5.5. Tại sao phải khởi tạo MSP trước?

Theo tài liệu, processor khởi tạo Main Stack Pointer trước khi đi tiếp tới Reset Handler.

Có thể hiểu luồng:

```text
Reset
  ↓
MSP được thiết lập
  ↓
Processor đã có Stack Pointer ban đầu
  ↓
mới tiếp tục vào Reset Handler
```

Chi tiết Stack hoạt động ra sao, MSP khác PSP thế nào và Stack frame được tổ chức như thế nào sẽ được triển khai riêng ở mục **Stack cơ bản trên Cortex-M**.

Ở đây chỉ cần nhớ:

> **Initial MSP là giá trị đầu tiên processor lấy từ vector table khi reset.**

---

### 1.5.6. Bước 3 — Đọc địa chỉ Reset Handler

Sau khi khởi tạo MSP, tài liệu mô tả processor đọc giá trị tại:

```text
0x00000004
```

Giá trị này chính là:

```text
địa chỉ của Reset Handler
```

Theo tài liệu:

```text
PC = value @ 0x00000004
```

Ví dụ minh họa:

```text
Memory[0x00000004] = 0x20001000
```

thì hình minh họa:

```text
PC = 0x20001000
```

và:

```text
0x20001000
→ địa chỉ bắt đầu của Reset Handler trong ví dụ
```

Địa chỉ trên chỉ là **địa chỉ minh họa trong hình nguồn**, không nên ghi nhớ như một địa chỉ cố định cho mọi STM32.

---

### 1.5.7. Bước 4 — PC nhảy tới Reset Handler

Sau khi PC nhận địa chỉ Reset Handler:

```text
PC
 ↓
Reset Handler address
```

processor bắt đầu thực thi lệnh tại Reset Handler.

Sơ đồ:

```text
0x00000004
     │
     │ chứa địa chỉ Reset Handler
     ↓
    PC
     │
     │ jump
     ↓
Reset_Handler
```

Hình nguồn mô tả:

```text
First instruction
      ↓
Next instruction
      ↓
...
```

tức là processor bắt đầu chạy các lệnh của Reset Handler.

---

### 1.5.8. Reset Handler là gì?

Theo tài liệu:

> **Reset Handler là một hàm C hoặc Assembly dùng để thực hiện các bước khởi tạo cần thiết sau reset.**

Có thể hình dung:

```text
Reset_Handler()
{
    // các bước khởi tạo cần thiết

    main();
}
```

Reset Handler là phần mã chạy **trước `main()`**.

Do đó:

```text
Reset
  ↓
Reset Handler
  ↓
Initialization
  ↓
main()
```

---

### 1.5.9. Vector Table đưa processor tới Reset Handler như thế nào?

Một hình trong tài liệu mô tả:

```text
Vector Table
     ↓
chỉ ra địa chỉ Reset Handler
     ↓
Processor thực thi Reset Handler sau reset
```

Sơ đồ:

```text
Vector Table
     │
     │ Reset vector
     ↓
Reset_Handler()
     │
     │ Initialization
     ↓
main()
```

Ở mức này cần nhớ:

> **Vector table chứa thông tin để processor tìm được Reset Handler sau reset.**

Chi tiết đầy đủ của vector table và các entry exception/interrupt sẽ được học ở phần Interrupt/Exception sau này.

---

### 1.5.10. Reset Handler làm gì trước `main()`?

Theo hình tài liệu, Reset Handler có các trách nhiệm chính trước khi gọi `main()`:

```text
Processor reset
      ↓
Initialize data section
      ↓
Initialize bss section
      ↓
Initialize C standard library
      ↓
main()
```

Hình còn chỉ ra lời gọi:

```c
__libc_init_array();
```

ở bước khởi tạo thư viện C.

Ở phần này chỉ cần nhớ thứ tự khái niệm:

```text
.data
  ↓
.bss
  ↓
C library
  ↓
main()
```

Chi tiết `.data`, `.bss`, cách chúng nằm trong Flash/RAM và linker bố trí chúng sẽ được để sang các mục:

```text
Startup Code
Linker Script
Flash và SRAM
```

---

### 1.5.11. Luồng Reset Sequence hoàn chỉnh

Ghép các hình và file lý thuyết lại:

```text
                 RESET
                   │
                   ↓
     đọc value tại 0x00000000
                   │
                   ↓
            khởi tạo MSP
                   │
                   ↓
     đọc value tại 0x00000004
                   │
                   ↓
        lấy địa chỉ Reset Handler
                   │
                   ↓
            nạp địa chỉ vào PC
                   │
                   ↓
            Reset_Handler
                   │
                   ↓
          Initialize .data
                   │
                   ↓
          Initialize .bss
                   │
                   ↓
      Initialize C standard library
                   │
                   ↓
                 main()
```

Đây là sơ đồ quan trọng nhất của mục này.

---

### 1.5.12. Mối liên hệ giữa MSP, PC và Reset Handler

Có thể tóm tắt bằng bảng:

| Thành phần | Vai trò trong Reset Sequence |
|---|---|
| `MSP` | Nhận initial Main Stack Pointer từ giá trị tại `0x00000000`. |
| `PC` | Nhận địa chỉ Reset Handler từ giá trị tại `0x00000004`. |
| Vector table | Cung cấp initial MSP và địa chỉ Reset Handler. |
| Reset Handler | Thực hiện các bước khởi tạo trước `main()`. |
| `main()` | Điểm bắt đầu của mã ứng dụng sau các bước khởi tạo. |

Sơ đồ ngắn:

```text
0x00000000 → MSP

0x00000004 → PC → Reset_Handler → main()
```

---

### 1.5.13. Ví dụ theo hình minh họa

Hình nguồn sử dụng ví dụ:

```text
Memory[0x00000000] = 0x20008000
Memory[0x00000004] = 0x20001000
```

Processor thực hiện:

```text
MSP = 0x20008000

PC = 0x20001000
```

Sau đó:

```text
PC
 ↓
0x20001000
 ↓
First instruction của Reset Handler
 ↓
Next instruction
 ↓
...
 ↓
main()
```

Hai giá trị `0x20008000` và `0x20001000` trong hình là **ví dụ minh họa cho cơ chế**, không phải giá trị bắt buộc của mọi chương trình STM32.

---

### 1.5.14. Reset Sequence khác `main()` như thế nào?

Một điểm dễ nhầm:

```text
Reset
≠
nhảy thẳng vào main()
```

Theo tài liệu:

```text
Reset
  ↓
MSP initialization
  ↓
Reset Handler
  ↓
các bước initialization
  ↓
main()
```

Vì vậy:

> **`main()` không phải đoạn mã đầu tiên được chạy ngay sau reset; Reset Handler chạy trước và chuẩn bị môi trường cần thiết rồi mới gọi `main()`.**

---

### 1.5.15. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Cortex-M làm gì ngay sau reset?”**, có thể trả lời:

> **Processor lấy initial MSP từ giá trị tại địa chỉ `0x00000000`, sau đó lấy địa chỉ Reset Handler từ `0x00000004`, nạp địa chỉ đó vào PC và bắt đầu chạy Reset Handler.**

Nếu hỏi **“Giá trị tại `0x00000000` dùng làm gì?”**:

> **Theo tài liệu, đó là initial value được nạp vào MSP — Main Stack Pointer.**

Nếu hỏi **“Giá trị tại `0x00000004` là gì?”**:

> **Đó là địa chỉ của Reset Handler, được processor đọc để đưa luồng thực thi tới Reset Handler.**

Nếu hỏi **“Reset Handler làm gì?”**:

> **Reset Handler thực hiện các bước khởi tạo cần thiết trước khi gọi `main()`. Hình tài liệu minh họa việc khởi tạo data section, bss section, C standard library rồi gọi `main()`.**

Nếu hỏi **“`main()` có phải lệnh đầu tiên chạy sau reset không?”**:

> **Không. Theo tài liệu, processor chạy Reset Handler trước; Reset Handler thực hiện các bước khởi tạo rồi mới gọi `main()`.**

Nếu hỏi **“MSP và PC liên quan gì tới Reset Sequence?”**:

> **MSP được khởi tạo từ vector đầu tiên tại `0x00000000`, còn PC nhận địa chỉ Reset Handler từ vector tại `0x00000004`.**

---

### 1.5.16. Câu hỏi phỏng vấn tự kiểm tra

1. Reset Sequence là gì?
2. Hai địa chỉ đầu tiên mà tài liệu nhấn mạnh là gì?
3. Giá trị tại `0x00000000` được nạp vào thanh ghi nào?
4. MSP viết tắt của gì?
5. Tại sao tài liệu nói Stack Pointer được khởi tạo trước?
6. Giá trị tại `0x00000004` đại diện cho gì?
7. Thanh ghi nào nhận địa chỉ Reset Handler?
8. Sau khi PC nhận địa chỉ Reset Handler thì processor làm gì?
9. Reset Handler là gì?
10. Reset Handler có thể được viết bằng ngôn ngữ nào theo tài liệu?
11. Reset Handler chạy trước hay sau `main()`?
12. Vector table có vai trò gì trong quá trình reset?
13. Hình tài liệu liệt kê những bước khởi tạo nào trước `main()`?
14. `__libc_init_array()` xuất hiện ở bước nào trong hình?
15. `.data` và `.bss` được xử lý trước hay sau `main()`?
16. Hai địa chỉ `0x20008000` và `0x20001000` trong hình có phải giá trị cố định cho mọi STM32 không?
17. Hãy mô tả Reset Sequence bằng MSP và PC.
18. Hãy vẽ lại chuỗi `Reset → MSP → PC → Reset_Handler → main()`.

---

### 1.5.17. Tóm tắt

```text
RESET
  ↓
đọc 0x00000000
  ↓
MSP = initial stack pointer
  ↓
đọc 0x00000004
  ↓
PC = Reset Handler address
  ↓
Reset_Handler
  ↓
Initialize data section
  ↓
Initialize bss section
  ↓
Initialize C standard library
  ↓
main()
```

Hai vector đầu tiên cần nhớ:

```text
0x00000000
→ Initial MSP

0x00000004
→ Reset Handler address
```

Quan hệ với Core Registers:

```text
MSP
→ thiết lập Stack Pointer ban đầu

PC
→ đưa luồng thực thi tới Reset Handler
```

**Ý quan trọng nhất:**

> **Sau reset, processor lấy initial MSP từ `0x00000000`, lấy địa chỉ Reset Handler từ `0x00000004`, chạy Reset Handler để thực hiện các bước khởi tạo cần thiết, rồi mới gọi `main()`.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-06"></a>
## 1.6. Bus Architecture

### 1.6.1. Bus Architecture dùng để làm gì?

Processor core không hoạt động độc lập. Nó cần trao đổi với:

```text
Flash / CODE region
SRAM
Peripheral
PPB
Vùng đặc thù của nhà sản xuất MCU
...
```

Việc trao đổi này được thực hiện thông qua các **bus interface**.

Theo tài liệu:

> Các bus interface của Cortex-Mx dựa trên đặc tả **AMBA**.

Có thể hình dung:

```text
Processor Core
     │
     │ Bus
     ↓
Memory / Peripheral / System resources
```

Bus có nhiệm vụ truyền các thông tin phục vụ truy cập, ví dụ:

```text
Địa chỉ
Dữ liệu
Tín hiệu điều khiển
```

Trong phạm vi tài liệu nguồn của mục này, trọng tâm là:

```text
AMBA
├── AHB-Lite
└── APB
```

---

### 1.6.2. AMBA là gì?

Theo hình tài liệu:

```text
AMBA
= Advanced Microcontroller Bus Architecture
```

Tài liệu mô tả AMBA là một đặc tả do ARM thiết kế để quy định chuẩn giao tiếp **on-chip** bên trong một System-on-Chip.

Có thể hiểu ở mức khái niệm:

```text
AMBA
→ bộ quy tắc / đặc tả
→ chuẩn hóa cách các khối trong chip giao tiếp với nhau
```

Hình nguồn cho biết AMBA hỗ trợ nhiều bus protocol, trong đó mục này tập trung vào:

```text
AHB-Lite
APB
```

---

### 1.6.3. AHB-Lite

Theo tài liệu:

```text
AHB-Lite
= AMBA High-performance Bus
```

AHB-Lite được dùng chủ yếu cho:

```text
Main bus interfaces
High-speed communication
Các peripheral yêu cầu tốc độ hoạt động cao
```

Có thể nhớ ngắn gọn:

> **AHB-Lite là bus chính, hướng tới truy cập tốc độ cao trong hệ thống.**

Trong hình kiến trúc bus, các đường:

```text
PPB
System
D-CODE
I-CODE
```

đều được biểu diễn với giao tiếp:

```text
AHB
32 bit
```

---

### 1.6.4. APB

Theo tài liệu:

```text
APB
= AMBA Peripheral Bus
```

APB được mô tả là bus dùng cho:

```text
PPB access
một số on-chip peripheral access
```

và các truy cập này đi thông qua:

```text
AHB-APB bridge
```

Tài liệu cũng nhấn mạnh rằng APB được dùng cho giao tiếp tốc độ thấp hơn AHB.

Có thể nhớ:

```text
AHB
→ tốc độ cao hơn

APB
→ tốc độ thấp hơn
→ phù hợp với nhiều peripheral không cần tốc độ cao
```

---

### 1.6.5. Tại sao cần AHB và APB?

Theo cách tổ chức trong tài liệu:

```text
Các khối cần tốc độ cao
        ↓
      AHB-Lite

Các peripheral không cần tốc độ cao
        ↓
       APB
```

Điều này giúp hệ thống không phải dùng cùng một kiểu bus cho mọi thiết bị.

Sơ đồ khái niệm:

```text
                 Processor
                     │
                     ↓
                  AHB-Lite
                     │
          ┌──────────┴──────────┐
          │                     │
          ↓                     ↓
   Khối tốc độ cao        AHB-APB Bridge
                                │
                                ↓
                               APB
                                │
                                ↓
                      Peripheral tốc độ thấp hơn
```

---

### 1.6.6. AHB-APB Bridge

Tài liệu mô tả APB được truy cập thông qua:

```text
AHB-APB Bridge
```

Vai trò ở mức khái niệm:

```text
AHB side
   │
   ↓
AHB-APB Bridge
   │
   ↓
APB side
```

Có thể hình dung khi processor muốn truy cập một peripheral nằm phía APB:

```text
Processor
   ↓
AHB
   ↓
AHB-APB Bridge
   ↓
APB
   ↓
Peripheral
```

Ở mục này chưa đi sâu vào timing hay protocol transaction của bridge.

---

### 1.6.7. Các bus interface được thể hiện trong hình Cortex-M

Hình nguồn thể hiện bốn đường/interface quan trọng:

```text
PPB
System
D-CODE
I-CODE
```

và các đường này giao tiếp ra các vùng tương ứng.

Sơ đồ rút gọn:

```text
ARM Cortex-Mx Processor
│
├── PPB
├── System
├── D-CODE
└── I-CODE
```

---

### 1.6.8. I-CODE bus

Theo hình:

```text
I-CODE
→ AHB 32-bit
→ CODE region
```

Mục đích được ghi rõ:

```text
Instruction fetch
Vector table read
```

Có thể hình dung:

```text
CODE region
    │
    │ instruction fetch
    │ vector table read
    ↓
 I-CODE
    ↓
Processor
```

Điểm cần nhớ:

> **I-CODE phục vụ việc lấy lệnh và đọc vector table từ CODE region theo hình tài liệu.**

---

### 1.6.9. D-CODE bus

Theo hình:

```text
D-CODE
→ AHB 32-bit
→ CODE region
```

Mục đích được ghi:

```text
Data access
```

Tức là:

```text
CODE region
    │
    │ data access
    ↓
 D-CODE
    ↓
Processor
```

Có thể nhớ ngắn:

```text
I-CODE
→ instruction fetch

D-CODE
→ data access
```

trong vùng CODE theo sơ đồ nguồn.

---

### 1.6.10. System bus

Hình tài liệu thể hiện:

```text
System
→ AHB 32-bit
```

và ghi:

```text
Any access (read/write)
```

System bus nối tới vùng:

```text
SRAM
Peripheral
External RAM
Device regions
```

Có thể hình dung:

```text
Processor
   │
   │ System bus
   ↓
SRAM / Peripheral / External RAM / Device regions
```

Điểm cần nhớ:

> **System bus phục vụ các truy cập đọc/ghi tới các vùng hệ thống như SRAM và peripheral theo hình tài liệu.**

---

### 1.6.11. PPB interface

Hình tài liệu còn thể hiện đường:

```text
PPB
→ AHB 32-bit
```

với chú thích:

```text
Data access
```

và nó nối tới vùng:

```text
PPB
```

Ngoài ra hình còn thể hiện một vùng:

```text
MCU Vendor specific region
```

ở phía trên.

Trong phạm vi nguồn hiện tại, tài liệu không giải thích chi tiết PPB là gì hoặc cấu trúc các register bên trong vùng đó, nên ở mục này chỉ cần ghi nhận:

```text
PPB
→ một bus/interface và vùng truy cập riêng được thể hiện trong sơ đồ
```

Chi tiết về các thanh ghi memory-mapped liên quan tới processor/peripheral sẽ được học ở mục **Memory-Mapped I/O**.

---

### 1.6.12. Sơ đồ tổng hợp từ tài liệu

Có thể diễn giải hình nguồn thành:

```text
                    ARM Cortex-Mx Processor
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ↓                    ↓                    ↓
      I-CODE               D-CODE               System
        │                    │                    │
        │ AHB 32-bit         │ AHB 32-bit         │ AHB 32-bit
        │                    │                    │
        ↓                    ↓                    ↓
   CODE region          CODE region       SRAM / Peripheral /
                                              External RAM /
                                              Device regions

                             │
                             ↓
                            PPB
                             │
                             │ AHB 32-bit
                             ↓
                         PPB region
```

Mục đích chính:

```text
I-CODE
→ Instruction fetch
→ Vector table read

D-CODE
→ Data access tới CODE region

System
→ Read/Write tới SRAM / Peripheral / Device regions

PPB
→ Data access tới PPB region
```

---

### 1.6.13. Quan hệ giữa Bus Architecture và Memory Map

Bus Architecture trả lời câu hỏi:

> **Processor đi bằng đường nào để truy cập các vùng trong hệ thống?**

Memory Map sẽ trả lời câu hỏi:

> **Các vùng đó nằm ở những địa chỉ nào trong không gian địa chỉ?**

Có thể nối hai khái niệm:

```text
Processor
   ↓
Bus Interface
   ↓
Memory Map
   ↓
CODE / SRAM / Peripheral / ...
```

Do đó:

```text
Bus Architecture
≠
Memory Map
```

nhưng hai phần liên quan trực tiếp tới nhau.

---

### 1.6.14. Phân biệt nhanh AHB-Lite và APB

| Tiêu chí | AHB-Lite | APB |
|---|---|---|
| Vai trò theo tài liệu | Main bus interface | Peripheral bus |
| Tốc độ tương đối | Cao hơn | Thấp hơn |
| Đối tượng sử dụng | Giao tiếp chính, peripheral cần tốc độ cao | Nhiều peripheral không yêu cầu tốc độ cao |
| Kết nối với nhau | Phía chính | Thường đi qua AHB-APB Bridge |

Cách nhớ:

```text
AHB-Lite
→ High-performance

APB
→ Peripheral
```

---

### 1.6.15. Phân biệt I-CODE, D-CODE và System bus

| Bus | Mục đích theo hình tài liệu |
|---|---|
| `I-CODE` | Instruction fetch và vector table read từ CODE region |
| `D-CODE` | Data access tới CODE region |
| `System` | Read/write tới SRAM, Peripheral, External RAM và Device regions |

Cách nhớ:

```text
I = Instruction
D = Data
System = các truy cập hệ thống khác
```

---

### 1.6.16. Luồng truy cập ví dụ

#### Trường hợp 1 — Processor lấy lệnh

```text
CODE region
    ↓
I-CODE
    ↓
Processor
```

#### Trường hợp 2 — Processor đọc dữ liệu trong CODE region

```text
CODE region
    ↓
D-CODE
    ↓
Processor
```

#### Trường hợp 3 — Processor đọc/ghi SRAM

```text
Processor
    ↓
System bus
    ↓
SRAM
```

#### Trường hợp 4 — Processor truy cập peripheral

Theo sơ đồ kiến trúc bus:

```text
Processor
    ↓
System / AHB
    ↓
Peripheral region
```

Nếu peripheral nằm phía APB theo cách tổ chức của hệ thống:

```text
Processor
    ↓
AHB
    ↓
AHB-APB Bridge
    ↓
APB
    ↓
Peripheral
```

---

### 1.6.17. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“AMBA là gì?”**, có thể trả lời:

> **AMBA là đặc tả bus do ARM thiết kế để chuẩn hóa giao tiếp on-chip. Trong tài liệu này, hai protocol chính được nhắc tới là AHB-Lite và APB.**

Nếu hỏi **“AHB-Lite và APB khác nhau thế nào?”**:

> **AHB-Lite được dùng cho main bus interface và các giao tiếp tốc độ cao hơn, còn APB được dùng cho nhiều peripheral không cần tốc độ cao và thường được nối với AHB qua AHB-APB Bridge.**

Nếu hỏi **“I-CODE dùng để làm gì?”**:

> **Theo hình tài liệu, I-CODE dùng cho instruction fetch và vector table read từ CODE region.**

Nếu hỏi **“D-CODE dùng để làm gì?”**:

> **D-CODE dùng để truy cập dữ liệu trong CODE region.**

Nếu hỏi **“System bus dùng để làm gì?”**:

> **System bus dùng cho các truy cập đọc/ghi tới các vùng như SRAM, peripheral, external RAM và device region.**

Nếu hỏi **“AHB-APB Bridge có vai trò gì?”**:

> **Nó làm cầu nối giữa phía AHB và phía APB để processor có thể truy cập các peripheral được kết nối trên APB.**

---

### 1.6.18. Câu hỏi phỏng vấn tự kiểm tra

1. Bus Architecture dùng để giải quyết vấn đề gì?
2. AMBA là gì theo tài liệu?
3. AMBA do ai thiết kế?
4. Hai bus protocol được tài liệu nhắc tới là gì?
5. AHB-Lite được dùng chủ yếu cho loại giao tiếp nào?
6. APB được dùng chủ yếu cho loại giao tiếp nào?
7. Vì sao hệ thống cần cả AHB và APB?
8. AHB-APB Bridge có vai trò gì?
9. I-CODE bus dùng để làm gì?
10. I-CODE đọc những gì từ CODE region theo hình?
11. D-CODE bus dùng để làm gì?
12. System bus truy cập những vùng nào?
13. Các bus trong hình có độ rộng bao nhiêu bit?
14. PPB xuất hiện ở đâu trong sơ đồ?
15. Bus Architecture và Memory Map khác nhau ở câu hỏi mà chúng trả lời như thế nào?
16. Hãy mô tả đường đi khi processor fetch instruction.
17. Hãy mô tả đường đi khi processor đọc/ghi SRAM.
18. Hãy mô tả đường đi khái niệm khi processor truy cập một peripheral nằm phía APB.

---

### 1.6.19. Tóm tắt

```text
AMBA
├── AHB-Lite
│   ├── main bus interface
│   └── high-speed communication
│
└── APB
    ├── peripheral bus
    ├── tốc độ thấp hơn AHB
    └── thường nối qua AHB-APB Bridge
```

Các bus interface trong hình:

```text
I-CODE
→ Instruction fetch
→ Vector table read

D-CODE
→ Data access tới CODE region

System
→ Read/Write tới SRAM / Peripheral /
  External RAM / Device regions

PPB
→ Data access tới PPB region
```

Sơ đồ tổng hợp:

```text
Processor
   │
   ├── I-CODE ──→ CODE region
   ├── D-CODE ──→ CODE region
   ├── System ──→ SRAM / Peripheral / Device regions
   └── PPB ─────→ PPB region
```

**Ý quan trọng nhất:**

> **Cortex-Mx sử dụng các bus interface dựa trên AMBA. AHB-Lite phục vụ các giao tiếp chính và tốc độ cao hơn, APB phục vụ nhiều peripheral tốc độ thấp hơn thông qua AHB-APB Bridge; còn I-CODE, D-CODE và System bus đảm nhiệm các loại truy cập khác nhau giữa processor và các vùng trong hệ thống.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-07"></a>
## 1.7. Memory Map

### 1.7.1. Memory Map là gì?

Theo tài liệu:

> **Memory map mô tả cách các vùng bộ nhớ và các thanh ghi peripheral được ánh xạ vào không gian địa chỉ mà processor có thể truy cập.**

Có thể hiểu:

```text
Không gian địa chỉ của processor
        ↓
được chia thành nhiều vùng
        ↓
mỗi vùng dành cho một loại tài nguyên
```

Ví dụ:

```text
Code
SRAM
Peripheral
External RAM
External Device
System / PPB
```

Nói ngắn gọn:

> **Memory Map là bản đồ cho biết mỗi loại bộ nhớ hoặc thiết bị nằm ở vùng địa chỉ nào.**

---

### 1.7.2. Không gian địa chỉ phụ thuộc vào Address Bus

Tài liệu nêu:

> Phạm vi địa chỉ mà processor có thể truy cập phụ thuộc vào kích thước của address bus.

Một hình nguồn thể hiện:

```text
32-bit address channel
32-bit data channel
```

và ghi:

```text
The processor has a fixed default memory map
that provides up to 4GB of addressable memory
```

Từ kênh địa chỉ 32-bit:

```text
2^32 địa chỉ byte
= 4 GB không gian địa chỉ
```

Có thể hình dung:

```text
0x00000000
    ↓
    ↓  toàn bộ không gian địa chỉ
    ↓
0xFFFFFFFF
```

Tổng phạm vi:

```text
0x00000000 → 0xFFFFFFFF
```

---

### 1.7.3. Bản đồ tổng thể 4 GB

Theo sơ đồ tổng thể trong tài liệu, không gian địa chỉ được chia thành các vùng lớn:

| Vùng | Khoảng địa chỉ theo hình | Kích thước |
|---|---:|---:|
| Code | `0x00000000` → `0x1FFFFFFF` | 0.5 GB |
| SRAM | `0x20000000` → `0x3FFFFFFF` | 0.5 GB |
| Peripheral | `0x40000000` → `0x5FFFFFFF` | 0.5 GB |
| External RAM | `0x60000000` → `0x9FFFFFFF` | 1 GB |
| External Device | `0xA0000000` → `0xDFFFFFFF` | 1 GB |
| System / PPB | `0xE0000000` → `0xFFFFFFFF` | phần còn lại của không gian địa chỉ |

Sơ đồ rút gọn:

```text
0xFFFFFFFF  +-----------------------------+
            | System / PPB                |
0xE0000000  +-----------------------------+
            | External Device             |
0xA0000000  +-----------------------------+
            | External RAM                |
0x60000000  +-----------------------------+
            | Peripheral                  |
0x40000000  +-----------------------------+
            | SRAM                        |
0x20000000  +-----------------------------+
            | Code                        |
0x00000000  +-----------------------------+
```

Đây là sơ đồ trung tâm của phần Memory Map.

---

### 1.7.4. Code Region

Theo hình tài liệu:

```text
Code Region
0x00000000 → 0x1FFFFFFF
512 MB
```

Tài liệu mô tả đây là vùng mà nhà sản xuất MCU có thể kết nối **CODE memory**, ví dụ:

```text
Embedded Flash
ROM
OTP
EEPROM
...
```

Hình cũng ghi:

> Processor mặc định lấy thông tin vector table từ vùng này ngay sau reset.

Điều này nối trực tiếp với phần Reset Sequence trước đó:

```text
Reset
  ↓
đọc vector table
  ↓
initial MSP
Reset Handler address
```

Có thể nhớ:

```text
Code Region
→ nơi chứa bộ nhớ chương trình theo mô hình tài liệu
→ processor fetch thông tin vector table từ đây sau reset
```

---

### 1.7.5. Quan hệ Code Region với I-CODE và D-CODE

Ở phần Bus Architecture trước đó, tài liệu mô tả:

```text
I-CODE
→ instruction fetch
→ vector table read

D-CODE
→ data access tới CODE region
```

Do đó có thể nối hai phần:

```text
                    CODE region
                   /           \
                  /             \
             I-CODE           D-CODE
                  \             /
                   \           /
                    Processor
```

Trong đó:

```text
I-CODE
→ lấy lệnh / đọc vector table

D-CODE
→ đọc dữ liệu trong CODE region
```

---

### 1.7.6. SRAM Region

Theo hình tài liệu:

```text
SRAM Region
0x20000000 → 0x3FFFFFFF
512 MB
```

Tài liệu mô tả:

- Đây là 512 MB tiếp theo sau CODE region.
- Chủ yếu dùng để kết nối SRAM, thường là on-chip SRAM.
- Có thể thực thi program code từ vùng này.
- Phần đầu của vùng có liên quan tới bit-band theo sơ đồ nguồn.

Có thể nhớ:

```text
SRAM Region
→ vùng dành chủ yếu cho SRAM
→ processor có thể đọc/ghi dữ liệu ở đây
→ tài liệu cho phép thực thi code từ vùng này
```

---

### 1.7.7. Bit-Band trong SRAM Region — chỉ nhận diện

Hình nguồn thể hiện:

```text
0x20000000
    ↓
1 MB Bit-Band Region
    ↓
0x20100000
```

và vùng alias:

```text
0x22000000
    ↓
32 MB Bit-Band Alias
    ↓
0x24000000
```

Có thể biểu diễn:

```text
SRAM region

0x20000000
+----------------------+
| 1 MB Bit-Band Region |
+----------------------+
0x20100000
|                      |
|      31 MB           |
|                      |
0x22000000
+----------------------+
| 32 MB Bit-Band Alias |
+----------------------+
0x24000000
```

Ở mục Memory Map hiện tại **chưa học cơ chế bit-band hoạt động như thế nào**.

Chỉ cần nhận diện:

```text
Bit-Band Region
Bit-Band Alias
```

và địa chỉ của chúng theo sơ đồ tài liệu.

---

### 1.7.8. Peripheral Region

Theo hình:

```text
Peripheral Region
0x40000000 → 0x5FFFFFFF
512 MB
```

Tài liệu mô tả:

- Chủ yếu dành cho các on-chip peripherals.
- Phần 1 MB đầu có thể là vùng bit-addressable nếu tính năng bit-band tùy chọn được hỗ trợ.
- Đây là vùng **Execute Never (XN)**.
- Cố thực thi code từ vùng này sẽ gây fault exception theo hình tài liệu.

Có thể nhớ:

```text
Peripheral Region
→ register / vùng của peripheral
→ không phải nơi để chạy chương trình
```

---

### 1.7.9. Bit-Band trong Peripheral Region — chỉ nhận diện

Theo sơ đồ tổng thể:

```text
0x40000000
    ↓
1 MB Bit-Band Region
    ↓
0x40100000
```

và:

```text
0x42000000
    ↓
32 MB Bit-Band Alias
    ↓
0x44000000
```

Sơ đồ:

```text
Peripheral region

0x40000000
+----------------------+
| 1 MB Bit-Band Region |
+----------------------+
0x40100000
|                      |
|      31 MB           |
|                      |
0x42000000
+----------------------+
| 32 MB Bit-Band Alias |
+----------------------+
0x44000000
```

Chi tiết công thức bit-band sẽ để sang phần riêng nếu sau này cần học.

---

### 1.7.10. External RAM Region

Theo hình tài liệu:

```text
External RAM Region
0x60000000 → 0x9FFFFFFF
1 GB
```

Tài liệu mô tả:

- Dành cho memory on-chip hoặc off-chip.
- Có thể thực thi code trong vùng này.
- Ví dụ: kết nối external SDRAM.

Có thể nhớ:

```text
External RAM
→ vùng dành cho RAM ngoài / memory mở rộng
→ ví dụ SDRAM
```

---

### 1.7.11. External Device Region

Theo hình:

```text
External Device Region
0xA0000000 → 0xDFFFFFFF
1 GB
```

Tài liệu mô tả:

- Dành cho external devices và/hoặc shared memory.
- Đây là vùng **Execute Never (XN)**.

Có thể nhớ:

```text
External Device
→ thiết bị ngoài / shared memory
→ XN
```

---

### 1.7.12. Private Peripheral Bus và System Region

Sơ đồ tổng thể của tài liệu đặt phần cuối của không gian địa chỉ từ:

```text
0xE0000000
```

trở lên cho các vùng liên quan tới:

```text
Private Peripheral Bus - Internal
Private Peripheral Bus - External
System
```

Sơ đồ tổng thể thể hiện:

```text
0xE0000000
    ↓
Private Peripheral Bus - Internal

0xE0040000
    ↓
Private Peripheral Bus - External

0xE0100000
    ↓
System

...
0xFFFFFFFF
```

Bên trong vùng PPB, hình tổng thể còn minh họa các khối như:

```text
ITM
DWT
FPB
SCS
TPIU
ETM
ROM Table
```

Trong phạm vi mục này chỉ cần nhận diện rằng:

> **Phần địa chỉ từ `0xE0000000` trở lên chứa các vùng system/private peripheral liên quan tới processor.**

---

### 1.7.13. Lưu ý về một điểm không nhất quán trong hình nguồn

Một hình riêng có tiêu đề **Private Peripheral Bus Region** nhưng lại gắn khoảng:

```text
0xA0000000 → 0xDFFFFFFF
```

trong khi sơ đồ tổng thể của cùng bộ tài liệu đặt khoảng đó là:

```text
External Device Region
```

và đặt PPB từ:

```text
0xE0000000
```

trở lên.

Vì nguồn hình ảnh không nhất quán ở điểm này, tài liệu này:

- Dùng **sơ đồ Memory Map tổng thể** làm nguồn cho địa chỉ vùng PPB.
- Chỉ lấy từ hình riêng ý nghĩa rằng vùng PPB có thể chứa các khối như:

```text
NVIC
System timer
System Control Block
```

và được hình mô tả là vùng **Execute Never**.

---

### 1.7.14. Memory Map và Peripheral Register

File `readme.txt` đưa ra ví dụ với ADC:

```text
ADC có dữ liệu
     ↓
dữ liệu nằm trong một thanh ghi của ADC
     ↓
CPU cần đọc thanh ghi đó
```

Tài liệu mô tả CPU thực hiện:

```text
CPU tạo địa chỉ của thanh ghi ADC
        ↓
đưa địa chỉ lên address bus
        ↓
địa chỉ khớp register tương ứng
        ↓
dữ liệu được đưa lên data bus
        ↓
CPU nhận dữ liệu
        ↓
CPU có thể lưu dữ liệu vào memory
```

Có thể hình dung:

```text
ADC register
     ↓
   CPU
     ↓
 Memory
```

File nguồn tóm tắt:

```text
ADC → CPU → Memory
```

Phần này mới chỉ dùng để giải thích **tại sao Memory Map quan trọng**:

> CPU phải biết **địa chỉ** của register/peripheral để truy cập đúng tài nguyên.

Cơ chế **Memory-Mapped I/O** sẽ được triển khai kỹ hơn ở mục 1.8.

---

### 1.7.15. Memory Map và Bus Architecture liên hệ thế nào?

Hai phần trả lời hai câu hỏi khác nhau:

```text
Bus Architecture
→ đi bằng đường nào?

Memory Map
→ tài nguyên nằm ở địa chỉ nào?
```

Ví dụ:

```text
CPU muốn đọc ADC
        ↓
Memory Map cho biết địa chỉ ADC register
        ↓
Bus truyền địa chỉ và dữ liệu
        ↓
CPU đọc được register
```

Sơ đồ:

```text
CPU
 ↓
Address
 ↓
Bus
 ↓
Memory Map
 ↓
Peripheral Register
```

---

### 1.7.16. Memory Map không có nghĩa MCU có thật 4 GB RAM

Hình tài liệu ghi processor có:

```text
up to 4GB of addressable memory
```

Điều này nói về:

```text
không gian địa chỉ
```

chứ không có nghĩa chip STM32 thực tế phải chứa:

```text
4 GB SRAM
hoặc
4 GB Flash
```

Trong phạm vi nguồn, các vùng 512 MB hoặc 1 GB thể hiện **phạm vi được dành trong bản đồ địa chỉ** cho từng loại tài nguyên.

Có thể nhớ:

```text
Address space
≠
dung lượng bộ nhớ vật lý thực tế
```

---

### 1.7.17. Tóm tắt các vùng chính

| Vùng | Base address | Vai trò chính theo tài liệu |
|---|---:|---|
| Code | `0x00000000` | Code memory, vector table |
| SRAM | `0x20000000` | SRAM / data memory |
| Peripheral | `0x40000000` | On-chip peripherals |
| External RAM | `0x60000000` | RAM ngoài / SDRAM |
| External Device | `0xA0000000` | Thiết bị ngoài / shared memory |
| PPB / System | `0xE0000000` | Processor/system-specific region |

Cách nhớ các base address:

```text
Code        → 0x00000000
SRAM        → 0x20000000
Peripheral  → 0x40000000
External RAM→ 0x60000000
Ext Device  → 0xA0000000
System/PPB  → 0xE0000000
```

---

### 1.7.18. Sơ đồ tổng hợp

```text
0xFFFFFFFF
+----------------------------------+
| System / processor-specific      |
| PPB                              |
+----------------------------------+ 0xE0000000
| External Device                  |
| XN                               |
+----------------------------------+ 0xA0000000
| External RAM                     |
| Có thể chứa/excute code theo     |
| hình tài liệu                    |
+----------------------------------+ 0x60000000
| Peripheral                       |
| XN                               |
+----------------------------------+ 0x40000000
| SRAM                             |
| Data / có thể execute code       |
+----------------------------------+ 0x20000000
| Code                             |
| Program memory / vector table    |
+----------------------------------+ 0x00000000
```

Luồng tư duy:

```text
Địa chỉ
  ↓
Memory Map xác định vùng
  ↓
Bus đưa truy cập tới vùng đó
  ↓
Memory / Peripheral phản hồi
```

---

### 1.7.19. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Memory Map là gì?”**, có thể trả lời:

> **Memory Map là cách processor chia không gian địa chỉ thành các vùng dành cho code, SRAM, peripheral, external memory và system resources. Nó cho biết một địa chỉ cụ thể tương ứng với loại tài nguyên nào.**

Nếu hỏi **“Cortex-M có không gian địa chỉ bao nhiêu?”**:

> **Theo tài liệu, processor có kênh địa chỉ 32-bit nên có tối đa 4 GB không gian địa chỉ, từ `0x00000000` đến `0xFFFFFFFF`.**

Nếu hỏi **“Code, SRAM và Peripheral bắt đầu ở đâu?”**:

> **Theo sơ đồ nguồn: Code bắt đầu tại `0x00000000`, SRAM tại `0x20000000`, và Peripheral tại `0x40000000`.**

Nếu hỏi **“Peripheral Region dùng để làm gì?”**:

> **Đây là vùng chủ yếu dành cho on-chip peripheral. Hình tài liệu mô tả vùng này là Execute Never, vì vậy không dùng để thực thi code.**

Nếu hỏi **“Memory Map liên quan gì đến peripheral register?”**:

> **Mỗi peripheral register có thể được gắn với một địa chỉ trong không gian địa chỉ. CPU đưa địa chỉ đó lên bus để truy cập đúng register; file nguồn minh họa bằng việc CPU đọc dữ liệu từ ADC register rồi chuyển dữ liệu vào memory.**

Nếu hỏi **“4 GB addressable memory có nghĩa STM32 có 4 GB RAM không?”**:

> **Không. 4 GB là không gian địa chỉ mà processor có thể biểu diễn; từng MCU thực tế chỉ triển khai một phần trong các vùng đó.**

---

### 1.7.20. Câu hỏi phỏng vấn tự kiểm tra

1. Memory Map là gì?
2. Kích thước address bus ảnh hưởng tới điều gì?
3. Kênh địa chỉ trong hình có độ rộng bao nhiêu bit?
4. 32-bit address space cho tối đa bao nhiêu không gian địa chỉ?
5. Không gian địa chỉ bắt đầu và kết thúc ở đâu?
6. Code Region bắt đầu tại địa chỉ nào?
7. Code Region có kích thước bao nhiêu theo hình?
8. Processor dùng Code Region cho những loại memory nào theo tài liệu?
9. Vector table liên quan gì tới Code Region?
10. SRAM Region bắt đầu tại địa chỉ nào?
11. Peripheral Region bắt đầu tại địa chỉ nào?
12. Peripheral Region được hình mô tả là Execute Never nghĩa là gì ở mức khái niệm?
13. External RAM Region nằm trong khoảng nào?
14. External Device Region nằm trong khoảng nào?
15. System/PPB region bắt đầu từ vùng địa chỉ nào theo sơ đồ tổng thể?
16. Bit-Band Region của SRAM bắt đầu tại đâu theo hình?
17. Bit-Band Alias của SRAM bắt đầu tại đâu?
18. Bit-Band Region của Peripheral bắt đầu tại đâu?
19. Bit-Band Alias của Peripheral bắt đầu tại đâu?
20. Memory Map và Bus Architecture khác nhau ở điểm nào?
21. CPU đọc ADC register bằng địa chỉ như thế nào theo `readme.txt`?
22. Vì sao 4 GB address space không có nghĩa MCU có 4 GB bộ nhớ vật lý?
23. Hãy kể các base address chính: Code, SRAM, Peripheral, External RAM, External Device và PPB/System.

---

### 1.7.21. Tóm tắt

```text
32-bit address bus
        ↓
4 GB address space
        ↓
0x00000000 → 0xFFFFFFFF
```

Các vùng chính:

```text
0x00000000
→ Code

0x20000000
→ SRAM

0x40000000
→ Peripheral

0x60000000
→ External RAM

0xA0000000
→ External Device

0xE0000000
→ PPB / System
```

Memory Map trả lời:

```text
“Địa chỉ này thuộc vùng nào?”
```

Bus Architecture trả lời:

```text
“Processor truy cập vùng đó qua đường nào?”
```

Ví dụ với peripheral:

```text
CPU tạo địa chỉ register
        ↓
Address Bus
        ↓
Peripheral register được chọn
        ↓
Data Bus
        ↓
CPU nhận dữ liệu
```

**Ý quan trọng nhất:**

> **Memory Map chia không gian địa chỉ 32-bit của processor thành các vùng Code, SRAM, Peripheral, External RAM, External Device và System/PPB. CPU dùng địa chỉ để xác định chính xác bộ nhớ hoặc peripheral register cần truy cập, còn bus là đường truyền địa chỉ và dữ liệu tới vùng đó.**

[↑ Về mục lục](#muc-luc)
