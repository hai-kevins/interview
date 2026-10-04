# Ghi chú phỏng vấn STM32

> **Mục tiêu:** Ôn STM32 theo hướng phỏng vấn Embedded Firmware, đi từ kiến trúc nền tảng đến ngoại vi và gỡ lỗi.

---

<a id="muc-luc"></a>
## Lộ trình tài liệu

1. **STM32 architecture / memory map**
   - 1.1. Processor Core vs Processor vs Microcontroller
   - 1.2. Operation Modes
   - **1.3. Access Level** ← đang triển khai
   - 1.4. Core Registers
   - 1.5. Reset Sequence
   - 1.6. Bus Architecture
   - 1.7. Memory Map
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
