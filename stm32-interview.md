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
   - 1.7. Memory Map
   - 1.8. Flash và SRAM
   - 1.9. Stack cơ bản trên Cortex-M
   - **1.10. Startup Code** ← đang triển khai
   - 1.11. Linker Script và các section
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

Khái niệm **Memory-Mapped Register / Memory-Mapped I/O** đã được giới thiệu ngay trong mục **1.4. Core Registers**; mục **1.7. Memory Map** tiếp tục làm rõ các vùng địa chỉ mà những tài nguyên đó được ánh xạ vào.

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

Khái niệm các thanh ghi memory-mapped liên quan tới processor/peripheral đã được giới thiệu ở mục **1.4. Core Registers**. Phần **1.7. Memory Map** tập trung vào vị trí các vùng đó trong không gian địa chỉ.

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

Cơ chế **Memory-Mapped I/O** đã được giải thích ở mục **1.4. Core Registers**; trong mục **1.7. Memory Map**, ví dụ ADC được dùng để nối khái niệm đó với address bus, data bus và vùng Peripheral.

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


---

<a id="muc-01-08"></a>
## 1.8. Flash và SRAM

### 1.8.1. Hai loại bộ nhớ chính cần phân biệt

Theo tài liệu nguồn, hai loại bộ nhớ quan trọng được đặt cạnh nhau là:

```text
Code memory
→ FLASH

Data memory
→ SRAM
```

Có thể nhớ ngắn:

```text
Flash
→ giữ chương trình và dữ liệu chỉ đọc

SRAM
→ giữ dữ liệu cần đọc/ghi khi chương trình chạy
```

Hai vùng này có đặc điểm và mục đích sử dụng khác nhau.

---

### 1.8.2. Flash Memory là gì?

Theo tài liệu:

- Flash là bộ nhớ **không mất dữ liệu khi mất nguồn** — non-volatile.
- Dùng để lưu:
  - firmware;
  - hằng số;
  - mã chương trình.
- Tài liệu mô tả Flash về bản chất thuộc nhóm ROM nhưng trong thực tế có thể xóa và ghi lại để nạp chương trình.
- Khi chương trình đang chạy, Flash chủ yếu được CPU đọc để lấy lệnh hoặc dữ liệu.
- Muốn ghi lại Flash phải đi qua thủ tục đặc biệt như:

```text
unlock
  ↓
erase
  ↓
program
```

Có thể hình dung:

```text
Mất nguồn
   ↓
Flash vẫn giữ nội dung
```

Đây là lý do firmware có thể vẫn tồn tại sau khi tắt nguồn và chạy lại khi MCU được cấp nguồn trở lại.

---

### 1.8.3. SRAM là gì?

Theo tài liệu:

```text
SRAM
= Static RAM
```

Đặc điểm:

- Là bộ nhớ **mất dữ liệu** — volatile.
- CPU có thể đọc/ghi trong khi chương trình chạy.
- Dùng để lưu các dữ liệu có thể thay đổi trong quá trình thực thi, ví dụ:
  - biến toàn cục;
  - biến `static`;
  - biến cục bộ trên Stack;
  - Heap;
  - context của task.
- Khi mất nguồn hoặc reset, nội dung SRAM không được giữ lại theo tài liệu.

Có thể hình dung:

```text
Chương trình đang chạy
        ↓
SRAM thay đổi liên tục

Mất nguồn / reset
        ↓
dữ liệu SRAM không được giữ lại
```

---

### 1.8.4. So sánh Flash và SRAM

| Tiêu chí | Flash | SRAM |
|---|---|---|
| Khả năng giữ dữ liệu khi mất nguồn | Có | Không |
| Vai trò chính theo tài liệu | Firmware, code, hằng số | Dữ liệu đọc/ghi khi chạy |
| CPU sử dụng khi chạy | Chủ yếu đọc | Đọc và ghi |
| Ghi dữ liệu mới | Cần thủ tục đặc biệt | Có thể đọc/ghi trực tiếp trong quá trình chạy |
| Thành phần điển hình trong hình | Vector Table, `.text`, `.rodata`, bản sao khởi tạo `.data` | `.data`, `.bss`, Heap, Stack |

Cách nhớ:

```text
Flash
→ lưu lâu dài

SRAM
→ vùng làm việc khi chương trình đang chạy
```

---

### 1.8.5. Bố cục Flash trong hình tài liệu

Hình nguồn mô tả Code Memory bắt đầu tại:

```text
0x08000000
```

và có bố cục khái niệm:

```text
Code memory (FLASH)

+-----------------------------+
| Unused code memory          |
+-----------------------------+
| .data                       |
| Initialized global/static   |
| variables — bản lưu ban đầu |
+-----------------------------+
| .rodata                     |
+-----------------------------+
| .text                       |
+-----------------------------+
| Vector Table                |
+-----------------------------+
0x08000000
```

Các thành phần cần nhận diện:

```text
Vector Table
.text
.rodata
.data (bản dữ liệu khởi tạo nằm trong Flash)
```

---

### 1.8.6. Vector Table trong Flash

Hình nguồn đặt:

```text
Vector Table
```

ở đầu vùng Flash minh họa.

Điều này nối với phần **Reset Sequence**:

```text
Reset
  ↓
processor đọc vector
  ↓
initial MSP
Reset Handler address
```

Có thể nhớ:

> **Vector Table là một phần dữ liệu quan trọng được lưu trong Code Memory để processor sử dụng khi reset và khi xử lý các vector tương ứng.**

Chi tiết vector table sẽ tiếp tục xuất hiện ở phần Interrupt/Exception sau này.

---

### 1.8.7. `.text` là gì trong sơ đồ?

Hình nguồn đặt:

```text
.text
```

trong Flash.

Ở mức tài liệu này, có thể hiểu:

```text
.text
→ phần mã chương trình được lưu trong Code Memory
```

Sơ đồ:

```text
C source
   ↓ biên dịch / liên kết
machine instructions
   ↓
.text
   ↓
Flash
```

Khi CPU chạy chương trình:

```text
CPU
 ↓ fetch instruction
Flash / .text
```

---

### 1.8.8. `.rodata` là gì?

Hình nguồn đặt:

```text
.rodata
```

trong Flash, phía trên `.text`.

Tài liệu `Flash & SRAM.txt` giải thích rằng các hằng số không cần đặt trong SRAM nếu chúng không thay đổi:

```text
const / read-only data
→ CPU chỉ cần đọc
→ không cần dùng SRAM để chứa một bản đọc/ghi
```

Nếu chuyển các hằng số từ Flash sang SRAM mỗi lần reset thì theo tài liệu sẽ:

- tốn SRAM;
- tốn thời gian khởi tạo do phải copy.

Do đó trong sơ đồ:

```text
.rodata
→ dữ liệu chỉ đọc
→ đặt trong Flash
```

---

### 1.8.9. `.data` đặc biệt ở điểm nào?

Đây là phần quan trọng nhất của hình.

Hình thể hiện `.data` xuất hiện **cả ở Flash và SRAM**:

```text
FLASH                         SRAM

.data                         .data
(initial values)    ----->    (runtime values)
                       copy
```

`.data` chứa:

```text
Initialized global variables
Initialized static variables
```

Ví dụ khái niệm:

```c
int counter = 10;
static int state = 2;
```

Các biến này cần:

```text
1. Có giá trị khởi tạo ban đầu
2. Có thể thay đổi trong quá trình chạy
```

Do đó tài liệu minh họa:

```text
Giá trị khởi tạo
→ lưu trong Flash

Khi startup
→ copy sang SRAM

Trong lúc chạy
→ chương trình sử dụng bản ở SRAM
```

---

### 1.8.10. Vì sao `.data` phải copy từ Flash sang SRAM?

Có hai yêu cầu cùng lúc:

```text
Yêu cầu 1
→ giá trị khởi tạo phải tồn tại sau khi mất nguồn

Yêu cầu 2
→ biến phải sửa được khi chương trình chạy
```

Theo bố cục tài liệu:

```text
Flash
→ giữ giá trị khởi tạo

SRAM
→ giữ bản có thể đọc/ghi khi chạy
```

Do đó khi startup:

```text
Flash .data
     │
     │ Data copy
     ↓
SRAM .data
```

Hình nguồn gọi quá trình này là:

```text
Transferring of .data section to RAM
('C' start-up)
```

---

### 1.8.11. `_etext`, `_sdata`, `_edata` trong hình

Hình minh họa ba boundary symbol:

```text
_etext
_sdata
_edata
```

Chúng được dùng để xác định các mốc phục vụ quá trình copy `.data`.

Có thể đọc sơ đồ theo ý tưởng:

```text
Nguồn trong Flash
→ vị trí chứa initial values của .data

Đích trong SRAM
→ bắt đầu tại _sdata
→ kết thúc tại _edata
```

Sơ đồ khái niệm:

```text
Flash
  |
  | nguồn dữ liệu khởi tạo
  |
  +--------------------+
                       |
                       | copy
                       v
                 _sdata
                    |
                    | .data trong SRAM
                    |
                 _edata
```

Trong phạm vi hình nguồn, chỉ cần hiểu đây là các **boundary** dùng để xác định vùng dữ liệu cần copy.

Chi tiết cách linker tạo các symbol này sẽ được học ở phần **Linker Script**.

---

### 1.8.12. `.bss` nằm ở đâu?

Hình nguồn đặt `.bss` trong SRAM:

```text
SRAM

+-----------------------------+
| Heap                        |
+-----------------------------+
| .bss                        |
| Uninitialized global/static |
| variables                   |
+-----------------------------+
| .data                       |
+-----------------------------+
```

Theo hình:

```text
.bss
→ uninitialized global variables
→ uninitialized static variables
```

Ví dụ:

```c
int counter;
static int status;
```

Mục **Reset Sequence** trước đó đã cho thấy Reset Handler có bước:

```text
Initialize bss section
```

Vì vậy có thể nối kiến thức:

```text
Reset
  ↓
Reset Handler
  ↓
Initialize .bss
  ↓
main()
```

Hình Flash/SRAM hiện tại không thể hiện một bản `.bss` cần copy từ Flash như `.data`.

---

### 1.8.13. Heap và Stack trong SRAM

Hình nguồn đặt cả:

```text
Heap
Stack
```

trong Data Memory — SRAM.

Bố cục minh họa:

```text
SRAM

+-----------------------------+
| Stack                       |
+-----------------------------+
| Unused SRAM                 |
+-----------------------------+
| Heap                        |
+-----------------------------+
| .bss                        |
+-----------------------------+
| .data                       |
+-----------------------------+
0x20000000
```

Điểm cần nhớ:

```text
Stack
→ nằm trong SRAM

Heap
→ nằm trong SRAM
```

Phần Stack sẽ được triển khai kỹ ở mục **1.9. Stack cơ bản trên Cortex-M**.

---

### 1.8.14. Tại sao biến đọc/ghi đặt trong SRAM?

Tài liệu giải thích:

> Các biến của chương trình là dữ liệu đọc/ghi vì giá trị của chúng có thể thay đổi trong quá trình chạy.

Do đó:

```text
Biến cần thay đổi
     ↓
cần vùng có thể đọc/ghi thuận tiện
     ↓
SRAM
```

Ví dụ:

```c
int counter = 0;

counter++;
counter++;
```

Giá trị `counter` thay đổi trong thời gian chạy, nên bản runtime của nó phải nằm ở vùng cho phép CPU đọc/ghi.

---

### 1.8.15. Vì sao `const` thường không cần chiếm SRAM?

Theo tài liệu:

```text
const
→ không thay đổi
→ CPU chỉ cần đọc
```

Nếu vẫn đặt một bản `const` trong SRAM thì tài liệu nêu hai chi phí:

```text
1. Tốn RAM
2. Tốn thời gian khởi tạo vì phải copy khi reset
```

Do đó hình bố trí:

```text
.rodata
→ Flash
```

Có thể nhớ:

> **Dữ liệu chỉ đọc phù hợp với Flash; dữ liệu cần thay đổi khi chạy phù hợp với SRAM.**

---

### 1.8.16. Quá trình từ Reset tới dữ liệu sẵn sàng trong SRAM

Kết hợp hình Flash/SRAM với phần Reset Sequence đã học:

```text
RESET
  ↓
MSP được thiết lập
  ↓
Reset Handler
  ↓
copy .data:
Flash → SRAM
  ↓
initialize .bss
  ↓
khởi tạo thư viện C
  ↓
main()
```

Đến khi vào `main()`:

```text
.text / .rodata
→ vẫn ở Flash theo sơ đồ nguồn

.data
→ đã có bản runtime trong SRAM

.bss
→ đã được khởi tạo trong SRAM

Heap / Stack
→ sử dụng SRAM
```

---

### 1.8.17. Sơ đồ tổng hợp Flash → SRAM

```text
                FLASH
0x08000000
+-----------------------------+
| Vector Table                |
+-----------------------------+
| .text                       |
| program code                |
+-----------------------------+
| .rodata                     |
| read-only data              |
+-----------------------------+
| .data initial values        |
+-----------------------------+
| Unused Flash                |
+-----------------------------+

            |
            | startup copy
            | .data
            v

                SRAM
0x20000000
+-----------------------------+
| .data                       |
| initialized global/static   |
+-----------------------------+
| .bss                        |
| uninitialized global/static |
+-----------------------------+
| Heap                        |
+-----------------------------+
| Unused SRAM                 |
+-----------------------------+
| Stack                       |
+-----------------------------+
```

Ý quan trọng:

```text
Flash
→ giữ nội dung lâu dài

SRAM
→ giữ trạng thái runtime
```

---

### 1.8.18. Liên hệ với Memory Map

Ở mục Memory Map trước đó:

```text
Code Region
→ dành cho code memory

SRAM Region
→ dành cho data memory
```

Hình Flash/SRAM của STM32 minh họa cụ thể hơn:

```text
Flash base
→ 0x08000000

SRAM base
→ 0x20000000
```

Do đó có thể nối:

```text
Memory Map
→ cho biết vùng địa chỉ

Flash / SRAM layout
→ cho biết firmware và các section được bố trí như thế nào trong những vùng đó
```

---

### 1.8.19. Phân biệt các section cần nhớ

| Section / vùng | Nằm ở đâu theo hình | Nội dung |
|---|---|---|
| Vector Table | Flash | Vector dùng bởi processor |
| `.text` | Flash | Mã chương trình |
| `.rodata` | Flash | Dữ liệu chỉ đọc |
| `.data` initial values | Flash | Giá trị khởi tạo ban đầu |
| `.data` runtime | SRAM | Biến global/static đã khởi tạo và có thể thay đổi |
| `.bss` | SRAM | Biến global/static chưa khởi tạo |
| Heap | SRAM | Vùng Heap |
| Stack | SRAM | Vùng Stack |

Cách nhớ:

```text
.text
.rodata
→ Flash

.data
.bss
Heap
Stack
→ SRAM khi chương trình chạy

.data
→ còn có initial values nằm trong Flash
```

---

### 1.8.20. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Flash và SRAM khác nhau như thế nào?”**, có thể trả lời:

> **Flash là bộ nhớ non-volatile dùng để giữ firmware và dữ liệu chỉ đọc; SRAM là bộ nhớ volatile dùng cho dữ liệu đọc/ghi trong lúc chương trình chạy như `.data`, `.bss`, Heap và Stack.**

Nếu hỏi **“Tại sao `.data` xuất hiện cả ở Flash và SRAM?”**:

> **Vì biến `.data` cần có giá trị khởi tạo tồn tại trong firmware nhưng đồng thời phải thay đổi được khi chạy. Giá trị ban đầu được lưu trong Flash, sau reset startup code copy nó sang SRAM để chương trình sử dụng.**

Nếu hỏi **“`.bss` chứa gì?”**:

> **Theo hình tài liệu, `.bss` chứa các biến global và static chưa khởi tạo và nằm trong SRAM. Reset Handler thực hiện bước khởi tạo `.bss` trước khi vào `main()`.**

Nếu hỏi **“Tại sao `const` thường được giữ trong Flash?”**:

> **Vì dữ liệu `const` không thay đổi và CPU chỉ cần đọc. Theo tài liệu, đưa chúng vào SRAM sẽ tốn RAM và còn tốn thời gian copy lúc startup.**

Nếu hỏi **“Stack và Heap nằm ở đâu?”**:

> **Theo sơ đồ nguồn, cả Stack và Heap đều nằm trong SRAM.**

Nếu hỏi **“`_sdata` và `_edata` dùng để làm gì?”**:

> **Trong hình chúng là các boundary xác định vùng `.data` trong SRAM để startup code biết phạm vi cần khởi tạo/copy; chi tiết cách linker tạo các symbol này sẽ học ở phần Linker Script.**

---

### 1.8.21. Câu hỏi phỏng vấn tự kiểm tra

1. Flash là volatile hay non-volatile?
2. SRAM là volatile hay non-volatile?
3. Firmware thường được lưu ở đâu theo tài liệu?
4. Dữ liệu chỉ đọc thường nằm ở section nào trong hình?
5. `.text` nằm ở Flash hay SRAM?
6. `.data` chứa loại biến nào?
7. Tại sao `.data` có bản trong cả Flash và SRAM?
8. Khi startup, `.data` được chuyển theo hướng nào?
9. `.bss` chứa loại biến nào?
10. `.bss` nằm ở Flash hay SRAM theo hình?
11. Stack nằm ở đâu?
12. Heap nằm ở đâu?
13. Tại sao dữ liệu `const` không cần một bản SRAM theo giải thích của tài liệu?
14. Vì sao biến cần thay đổi trong lúc chạy phù hợp với SRAM?
15. Vector Table nằm ở đâu trong sơ đồ nguồn?
16. `_sdata` và `_edata` đại diện cho điều gì ở mức khái niệm?
17. Phần Reset Handler liên quan thế nào tới `.data` và `.bss`?
18. Hãy mô tả luồng `Flash .data → SRAM .data → main()`.
19. Hãy phân biệt `.text`, `.rodata`, `.data` và `.bss`.
20. Flash base và SRAM base trong hình lần lượt là gì?

---

### 1.8.22. Tóm tắt

```text
FLASH
→ non-volatile
→ firmware
→ Vector Table
→ .text
→ .rodata
→ initial values của .data
```

```text
SRAM
→ volatile
→ runtime data
→ .data
→ .bss
→ Heap
→ Stack
```

Điểm quan trọng nhất của `.data`:

```text
Flash
.data initial values
        │
        │ C startup copy
        ↓
SRAM
.data runtime
```

Trước `main()`:

```text
Reset Handler
   ↓
copy .data
   ↓
initialize .bss
   ↓
main()
```

**Ý quan trọng nhất:**

> **Flash giữ firmware và dữ liệu cần tồn tại lâu dài, còn SRAM là vùng làm việc đọc/ghi trong lúc chương trình chạy. `.data` đặc biệt vì giá trị khởi tạo nằm trong Flash nhưng bản runtime được copy sang SRAM trước khi `main()` bắt đầu.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-09"></a>
## 1.9. Stack cơ bản trên Cortex-M

### 1.9.1. Stack Memory là gì?

Theo tài liệu, **Stack Memory** là một phần của bộ nhớ chính, thường nằm trong RAM, được dành cho việc lưu trữ dữ liệu tạm thời trong lúc chương trình chạy.

Nguồn mô tả Stack có thể thuộc:

```text
Internal RAM
hoặc
External RAM
```

và chủ yếu được dùng trong:

```text
Function call
Interrupt handling
Exception handling
```

Stack làm việc theo nguyên tắc:

```text
LIFO
= Last In, First Out
= vào sau, ra trước
```

Ví dụ:

```text
PUSH A
PUSH B
PUSH C

POP → C
POP → B
POP → A
```

---

### 1.9.2. Stack được dùng để lưu những gì?

Theo các slide nguồn, Stack thường được dùng để lưu tạm:

- Giá trị của các thanh ghi processor.
- Biến cục bộ của hàm.
- Dữ liệu tạm thời trong quá trình gọi hàm.
- Context khi xảy ra interrupt/exception, ví dụ:
  - general-purpose registers;
  - processor status register;
  - return address.

Có thể hình dung:

```text
Stack
├── local variables
├── saved registers
├── return information
└── exception / interrupt context
```

Điểm cần nhớ:

> **Stack là vùng lưu trạng thái tạm thời phục vụ luồng thực thi hiện tại.**

---

### 1.9.3. Stack nằm ở đâu trong SRAM?

Một hình trong tài liệu chia SRAM thành các vùng khái niệm:

```text
RAM_START
   ↓
+------------------+
| Global data      |
+------------------+
| Heap             |
+------------------+
| Stack            |
+------------------+
   ↑
RAM_END
```

Trong ví dụ này:

```text
Global data
→ dùng cho dữ liệu toàn cục / static

Heap
→ dùng cho cấp phát động

Stack
→ dùng cho function call, local variables,
  register/context tạm thời
```

Điều này nối trực tiếp với mục **1.8 Flash và SRAM**:

```text
.data / .bss
Heap
Stack
→ đều sử dụng SRAM trong lúc chương trình chạy
```

---

### 1.9.4. Stack Pointer — SP / R13

Stack được theo dõi bằng thanh ghi:

```text
SP
= Stack Pointer
= R13
```

Theo tài liệu:

- `PUSH` và `POP` làm thay đổi Stack Pointer.
- Stack cũng có thể được truy cập bằng các lệnh load/store như `LDR`/`STR` ở mức Assembly.
- SP cho biết vị trí hiện tại của Stack theo mô hình đang sử dụng.

Có thể nhớ:

```text
Stack
   ↑
   │
  SP
```

SP là một trong các core register đã học ở mục **1.4 Core Registers**.

---

### 1.9.5. Bốn mô hình hoạt động của Stack

Tài liệu liệt kê bốn mô hình:

```text
1. Full Ascending
2. Full Descending
3. Empty Ascending
4. Empty Descending
```

Hai từ khóa cần hiểu:

```text
Full / Empty
→ SP đang trỏ tới ô đã có dữ liệu
  hay ô trống kế tiếp

Ascending / Descending
→ Stack mở rộng theo địa chỉ tăng
  hay địa chỉ giảm
```

---

### 1.9.6. Full Ascending Stack

Theo file nguồn:

```text
Full Ascending
```

có đặc điểm:

- SP trỏ tới ô đã chứa dữ liệu cuối cùng.
- Khi `PUSH`, SP tăng lên.
- Khi `POP`, lấy dữ liệu tại SP rồi SP giảm.
- Stack mở rộng theo hướng địa chỉ tăng.

Sơ đồ khái niệm:

```text
Địa chỉ tăng ↑

[ data mới  ] ← SP sau PUSH
[ data cũ   ]
[ ...       ]
```

---

### 1.9.7. Full Descending Stack

Theo tài liệu:

```text
Full Descending
```

có đặc điểm:

- SP trỏ tới ô đã chứa dữ liệu cuối cùng.
- Khi `PUSH`, SP giảm.
- Khi `POP`, lấy dữ liệu tại SP rồi SP tăng.
- Stack mở rộng theo hướng địa chỉ giảm.

File nguồn ghi:

> **Đây là mô hình mặc định của ARM Cortex-M.**

Có thể hình dung:

```text
Địa chỉ cao
   ↑
   |
[ dữ liệu cũ ]
[ dữ liệu cũ ]
[ dữ liệu mới ] ← SP
   |
   ↓ Stack phát triển xuống địa chỉ thấp
Địa chỉ thấp
```

Cách nhớ:

```text
Full
→ SP trỏ vào dữ liệu hiện tại

Descending
→ PUSH làm SP đi xuống địa chỉ thấp hơn
```

---

### 1.9.8. Empty Ascending Stack

Theo tài liệu:

- SP trỏ tới ô trống kế tiếp.
- Khi `PUSH`, dữ liệu được ghi vào ô SP rồi SP tăng.
- Khi `POP`, SP giảm trước rồi mới lấy dữ liệu.
- Stack mở rộng theo địa chỉ tăng.

```text
Empty
→ SP chỉ vào vị trí trống kế tiếp

Ascending
→ Stack phát triển lên địa chỉ cao
```

---

### 1.9.9. Empty Descending Stack

Theo tài liệu:

- SP trỏ tới ô trống kế tiếp.
- Khi `PUSH`, SP giảm trước rồi mới ghi dữ liệu.
- Khi `POP`, lấy dữ liệu rồi SP tăng.
- Stack mở rộng theo địa chỉ giảm.

```text
Empty
→ SP trỏ ô trống

Descending
→ Stack phát triển xuống địa chỉ thấp
```

---

### 1.9.10. So sánh bốn Stack Model

| Mô hình | SP trỏ tới | Hướng Stack phát triển |
|---|---|---|
| Full Ascending | Ô đã có dữ liệu cuối cùng | Địa chỉ tăng |
| Full Descending | Ô đã có dữ liệu cuối cùng | Địa chỉ giảm |
| Empty Ascending | Ô trống kế tiếp | Địa chỉ tăng |
| Empty Descending | Ô trống kế tiếp | Địa chỉ giảm |

Điểm cần nhớ cho Cortex-M:

```text
ARM Cortex-M
→ Full Descending
```

---

### 1.9.11. Ví dụ PUSH/POP trên Full Descending Stack

Một slide nguồn minh họa chuỗi thao tác:

```text
PUSH LR
PUSH R0
PUSH R1
POP  R2
POP  R3
POP  PC
```

Ý chính của ví dụ không phải là ghi nhớ các giá trị cụ thể, mà là quan sát:

```text
PUSH
→ SP dịch xuống địa chỉ thấp hơn

POP
→ SP dịch lên địa chỉ cao hơn
```

với mô hình Full Descending.

Do LIFO:

```text
giá trị được PUSH sau
→ sẽ được POP trước
```

---

### 1.9.12. Stack có thể được đặt ở đâu trong RAM?

Các slide cho thấy Stack có thể được bố trí ở những vị trí khác nhau trong RAM tùy thiết kế/linker script.

Ví dụ 1:

```text
RAM
+-------------------+
| Unused            |
+-------------------+
| Stack             |
+-------------------+
| Heap              |
+-------------------+
| Data              |
+-------------------+
```

Ví dụ 2:

```text
RAM
+-------------------+
| Stack             | ← gần cuối RAM
+-------------------+
| Unused            |
+-------------------+
| Heap              |
+-------------------+
| Data              |
+-------------------+
```

Một slide minh họa symbol:

```text
_estack
```

và mô tả nó là linker symbol dùng để chỉ **cuối RAM làm điểm bắt đầu của Stack** trong cách bố trí đó.

Điểm cần nhớ:

> **Vị trí Stack không phải tự nhiên xuất hiện; linker/startup code phải biết biên Stack hợp lệ.**

---

### 1.9.13. Vì sao Stack thường bắt đầu ở địa chỉ cao?

Do Cortex-M sử dụng Full Descending Stack theo tài liệu:

```text
PUSH
→ SP giảm
```

nên một cách bố trí phổ biến là đặt initial SP gần cuối vùng RAM:

```text
RAM_END / _estack
        ↓
      Stack
        ↓
        ↓ phát triển về địa chỉ thấp
```

Sơ đồ:

```text
Địa chỉ cao
RAM_END / _estack
        ↓
+------------------+
| Stack            |
|        ↓         |
|        ↓         |
+------------------+
| Unused RAM       |
+------------------+
| Heap             |
+------------------+
| Data             |
+------------------+
Địa chỉ thấp
```

---

### 1.9.14. MSP và PSP

Tài liệu về **Banked Stack Pointers** giới thiệu:

```text
MSP
= Main Stack Pointer

PSP
= Process Stack Pointer
```

và:

```text
SP / R13
→ current stack pointer
```

Có thể hình dung:

```text
             SP / R13
                │
                │ chọn một Stack Pointer hiện hành
          ┌─────┴─────┐
          ↓           ↓
         MSP         PSP
```

#### Lưu ý về cách diễn đạt trong nguồn

Trong ZIP có hai cách diễn đạt:

- File `Banked stack pointer registers.txt` gọi `SP`, `MSP`, `PSP` là “3 stack pointers”.
- Slide `MSP, PSP summary` lại ghi Cortex-M có **2 stack pointer registers**, là `MSP` và `PSP`.

Để giữ hai nguồn nhất quán, tài liệu này dùng cách biểu diễn:

```text
MSP và PSP
→ hai banked Stack Pointer

SP / R13
→ Stack Pointer hiện đang được chọn
```

---

### 1.9.15. MSP — Main Stack Pointer

Theo nguồn:

- Sau reset, **MSP được chọn làm current Stack Pointer mặc định**.
- MSP được processor khởi tạo tự động từ giá trị ở:

```text
0x00000000
```

Điều này nối với phần Reset Sequence:

```text
Reset
  ↓
value @ 0x00000000
  ↓
MSP
```

Nguồn cũng mô tả:

```text
Handler mode
→ luôn dùng MSP
```

Điểm cần nhớ:

> **MSP là Stack Pointer mặc định sau reset và là Stack Pointer được Handler mode sử dụng.**

---

### 1.9.16. PSP — Process Stack Pointer

Theo tài liệu:

```text
PSP
→ Stack Pointer thay thế
→ có thể được Thread mode sử dụng
```

Thread mode có thể chọn PSP bằng bit:

```text
CONTROL.SPSEL
```

Nguồn nhấn mạnh:

> Nếu muốn sử dụng PSP, chương trình phải khởi tạo PSP tới một địa chỉ Stack hợp lệ.

Có thể hình dung:

```text
Thread mode
     │
     ├── dùng MSP
     │
     └── dùng PSP
```

Trong khi:

```text
Handler mode
→ luôn MSP
```

---

### 1.9.17. Quan hệ Thread mode, Handler mode, MSP và PSP

Ghép với các phần Operation Modes đã học:

```text
Thread mode
├── MSP
└── PSP

Handler mode
└── MSP
```

Theo tài liệu:

- Thread mode có thể đổi current SP sang PSP bằng `CONTROL.SPSEL`.
- Handler mode luôn dùng MSP.
- Thay đổi `SPSEL` trong Handler mode không có ý nghĩa theo file nguồn và thao tác ghi sẽ bị bỏ qua.

Sơ đồ:

```text
                 Processor
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
   Thread mode               Handler mode
        │                         │
   ┌────┴────┐                    ↓
   ↓         ↓                   MSP
  MSP       PSP
```

---

### 1.9.18. Tại sao Thread mode dùng PSP lại hữu ích?

File `Question.txt` đưa ra hai ý chính.

#### 1. Phân tách Stack ứng dụng và Stack hệ thống

```text
Thread / task
→ PSP

Interrupt / exception
→ MSP
```

Nhờ vậy:

```text
Stack ứng dụng
≠
Stack xử lý exception
```

Nguồn giải thích rằng sự tách biệt này giúp Stack của ứng dụng không “đụng” trực tiếp vào Stack dành cho ngắt/exception.

#### 2. Hỗ trợ RTOS / đa nhiệm

Theo file nguồn:

```text
Task A → PSP riêng
Task B → PSP riêng
Task C → PSP riêng

ISR → MSP
```

Khi chuyển task, kernel có thể thay đổi PSP để chuyển sang Stack của task khác.

Ở mức hiện tại chỉ cần biết ý tưởng:

> **PSP giúp tách Stack của từng task/application khỏi Stack hệ thống dùng cho exception.**

FreeRTOS chi tiết sẽ được học sau nếu cần.

---

### 1.9.19. Thay đổi Stack Pointer

Một slide trong nguồn ghi rằng ở Assembly có thể truy cập MSP và PSP bằng các instruction:

```text
MRS
MSR
```

và trong C có thể cần mã mức thấp/naked function nếu muốn trực tiếp thay đổi Stack Pointer.

Ở mức Intern, không cần nhớ cú pháp Assembly cụ thể trong mục này.

Điểm cần nhớ:

```text
MSP / PSP
→ là các thanh ghi đặc biệt
→ việc đổi current Stack Pointer là thao tác mức thấp
```

---

### 1.9.20. AAPCS là gì?

Bộ tài liệu Stack còn có phần:

```text
AAPCS
= Procedure Call Standard for the Arm Architecture
```

Đây là chuẩn quy định **cách các hàm gọi nhau trên ARM**.

Nguồn mô tả nó như một “hợp đồng” giữa:

```text
Caller
→ hàm gọi

Callee
→ hàm được gọi
```

AAPCS thuộc ABI:

```text
ABI
= Application Binary Interface
```

Mục tiêu:

> Các module được viết/biên dịch riêng vẫn có thể gọi nhau đúng cách nếu cùng tuân thủ quy ước.

---

### 1.9.21. Caller và Callee

Ví dụ:

```c
void caller(void)
{
    int ret;
    ret = callee(1, 2, 4, 5);
}
```

```c
int callee(int a, int b, int c, int d)
{
    return a + b + c + d;
}
```

Trong đó:

```text
caller()
→ Caller

callee()
→ Callee
```

AAPCS quy định cách truyền:

```text
tham số
register
Stack
giá trị trả về
```

giữa hai hàm.

---

### 1.9.22. Truyền tham số theo AAPCS trong tài liệu

File nguồn nêu:

```text
4 tham số đầu
→ R0, R1, R2, R3

Nếu có thêm tham số
→ Stack
```

Sơ đồ:

```text
Caller
  │
  ├── arg1 → R0
  ├── arg2 → R1
  ├── arg3 → R2
  ├── arg4 → R3
  └── arg5... → Stack
        ↓
      Callee
```

#### Lưu ý về một hình nguồn

Một slide minh họa bốn tham số nhưng nhãn register trên hình bị ghi thành:

```text
R0, R1, R3, R4
```

trong khi file AAPCS dạng text và các slide giải thích khác ghi rõ:

```text
R0-R3
```

Tài liệu này giữ quy tắc **R0-R3** vì đó là nội dung được nguồn text mô tả trực tiếp; hình trên được xem là không nhất quán về nhãn.

---

### 1.9.23. Caller-saved Registers

Theo slide AAPCS:

```text
R0
R1
R2
R3
R12
R14 / LR
```

được mô tả là nhóm **caller-saved registers**.

Ý tưởng:

```text
Caller cần giữ giá trị nào
sau khi gọi hàm?
       │
       └── caller tự lưu trước khi call
```

Callee có thể sử dụng các register này mà không phải khôi phục lại đúng giá trị cũ theo cách mô tả của nguồn.

---

### 1.9.24. Callee-saved Registers

Theo tài liệu:

```text
R4 → R11
```

được gọi là:

```text
callee-saved registers
```

Nếu callee muốn thay đổi chúng:

```text
Callee
  ↓
save giá trị cũ
  ↓
sử dụng register
  ↓
restore giá trị
  ↓
return
```

Stack thường được dùng để lưu/khôi phục các giá trị này.

Có thể nhớ:

```text
R0-R3, R12, LR
→ caller-saved theo slide

R4-R11
→ callee-saved
```

---

### 1.9.25. Giá trị trả về theo tài liệu AAPCS

Các slide nguồn mô tả kết quả trả về qua:

```text
R0
```

và có slide nói `R0`/`R1` có thể được dùng để gửi result về caller.

Ở mức này nên nhớ:

```text
Giá trị trả về
→ bắt đầu từ R0 theo ví dụ nguồn
```

Việc chính xác cần bao nhiêu register phụ thuộc kiểu dữ liệu/kích thước kết quả và sẽ không đi sâu trong mục Stack cơ bản.

---

### 1.9.26. Luồng gọi hàm theo AAPCS

Có thể tóm tắt tài liệu thành:

```text
1. Caller đặt tham số vào R0-R3
   nếu nhiều hơn → Stack

2. Caller lưu những caller-saved value
   mà nó còn cần sau lời gọi

3. Nhảy tới callee

4. Callee có thể dùng R0-R3, R12...
   nếu dùng R4-R11 thì phải bảo toàn

5. Callee đặt result vào register trả về

6. Callee return

7. Caller tiếp tục thực thi
```

Sơ đồ:

```text
Caller
  │
  │ args
  ↓
Registers / Stack
  │
  ↓
Callee
  │
  │ result
  ↓
R0...
  │
  ↓
Caller
```

---

### 1.9.27. Stack trong Interrupt và Exception

Một nhóm slide nguồn mô tả **Stack activities during interrupt and exception**.

Khi exception xảy ra, processor tự động lưu một nhóm register để tạo:

```text
Stack Frame
```

Nguồn liệt kê:

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

Sơ đồ:

```text
Thread mode đang chạy
        ↓
Exception / Interrupt
        ↓
Hardware tự động stacking
        ↓
R0-R3, R12, LR, PC, xPSR
        ↓
Handler mode
```

---

### 1.9.28. Vì sao Hardware tự động Stacking?

Slide nguồn giải thích rằng việc tự động lưu context cho phép một hàm C thông thường được dùng làm exception/interrupt handler mà không phải tự xử lý toàn bộ quy ước lưu các caller-saved register ngay từ đầu.

Có thể hiểu:

```text
Exception xảy ra
      ↓
Processor tự save context cần thiết
      ↓
Handler chạy
      ↓
Processor restore context khi thoát
      ↓
Chương trình bị ngắt tiếp tục
```

Mục tiêu:

> Khi quay lại chương trình cũ, các giá trị cần thiết phải trở về trạng thái như trước lúc exception.

---

### 1.9.29. Stack Frame khi Exception

Theo hình `Analyzing stack frame`, một Stack Frame không có FPU chứa tám word:

```text
xPSR
PC
LR
R12
R3
R2
R1
R0
```

Tổng cộng:

```text
8 register
× 4 byte
= 32 byte
```

Trong hình Full Descending:

```text
SP trước exception
        ↓
processor stacking
        ↓
SP sau exception thấp hơn 32 byte
```

Sơ đồ:

```text
Địa chỉ cao
+------------------+
| Previous stack   |
+------------------+
| xPSR             |
| PC               |
| LR               |
| R12              |
| R3               |
| R2               |
| R1               |
| R0               | ← SP sau stacking
+------------------+
Địa chỉ thấp
```

Hình nguồn ghi rõ:

```text
Stack Frame (No FPU)
```

nên cấu trúc này đang nói tới trường hợp minh họa **không có FPU context**.

---

### 1.9.30. Un-stacking khi thoát Exception

Khi handler kết thúc, slide mô tả processor tự động:

```text
Un-stacking
```

Tức là các giá trị trước đó được khôi phục từ Stack.

Luồng:

```text
Thread mode
   ↓ exception
Stacking
   ↓
Handler mode
   ↓ handler exit
Un-stacking
   ↓
Thread mode tiếp tục
```

Có thể nhớ:

```text
Exception entry
→ stacking

Exception return
→ un-stacking
```

---

### 1.9.31. Stack Frame hữu ích khi Debug Fault

Phần `Analyzing stack frame` cho thấy khi exception/fault xảy ra, Stack Frame chứa:

```text
PC
LR
xPSR
R0-R3
R12
```

Vì vậy ở mức khái niệm, Stack Frame có thể giúp xác định:

```text
PC
→ chương trình đang ở đâu khi fault xảy ra

LR
→ thông tin liên quan tới luồng quay về

R0-R3 / R12
→ dữ liệu register tại thời điểm exception

xPSR
→ trạng thái processor
```

Đây là nền tảng để sau này debug HardFault/UsageFault bằng debugger.

---

### 1.9.32. Khởi tạo Stack

Nguồn `Stack initialization` nhấn mạnh:

> Trước khi `main()` chạy, Stack Pointer đã phải được khởi tạo.

Điều này nối với Reset Sequence:

```text
Reset
  ↓
processor đọc vector table
  ↓
MSP được khởi tạo
  ↓
Reset Handler
  ↓
main()
```

Nguồn cũng nói sau khi vào `main()` chương trình có thể cấu hình lại Stack Pointer nếu thiết kế cần.

---

### 1.9.33. Các lưu ý khi thiết kế Stack theo tài liệu

Slide `Stack initialization tips` đưa ra các ý:

1. Ước lượng lượng Stack cần trong tình huống xấu nhất của ứng dụng.
2. Biết mô hình Stack mà processor sử dụng:
   - Full Ascending;
   - Full Descending;
   - Empty Ascending;
   - Empty Descending.
3. Quyết định vị trí Stack trong RAM:
   - đầu;
   - giữa;
   - cuối vùng RAM.
4. Có thể đặt Stack trong internal RAM hoặc external memory nếu hệ thống đã khởi tạo vùng đó phù hợp.
5. ARM Cortex-M lấy initial MSP từ entry đầu của vector table.
6. Linker script thường quyết định biên Stack, Heap và các vùng RAM.
7. Trong RTOS, kernel có thể dùng MSP cho Stack hệ thống và cấu hình PSP cho các task.

Các chi tiết về Linker Script sẽ được học ở mục **1.11**.

---

### 1.9.34. Sơ đồ tổng hợp

```text
                         STACK
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
  Function call       Exception/IRQ       RTOS task
        │                  │                  │
        ↓                  ↓                  ↓
 local variables      HW stacking           PSP
 saved registers      R0-R3, R12             │
 return info          LR, PC, xPSR            │
        │                  │                  │
        └──────────────┬───┴──────────────────┘
                       ↓
                    SRAM
```

Stack Pointer:

```text
SP / R13
   │
   ├── MSP
   │   ├── default sau reset
   │   └── Handler mode luôn dùng
   │
   └── PSP
       └── Thread mode có thể dùng
```

Cortex-M Stack model:

```text
Full Descending
→ PUSH: SP giảm
→ POP : SP tăng
```

---

### 1.9.35. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Stack là gì?”**, có thể trả lời:

> **Stack là vùng RAM dùng để lưu dữ liệu tạm thời theo nguyên tắc LIFO, như biến cục bộ, register cần bảo toàn, return information và context khi xảy ra exception/interrupt.**

Nếu hỏi **“Cortex-M dùng Stack model nào?”**:

> **Theo tài liệu, Cortex-M sử dụng Full Descending Stack: SP trỏ vào phần tử đang ở đỉnh Stack và Stack phát triển về địa chỉ thấp hơn.**

Nếu hỏi **“MSP và PSP khác nhau thế nào?”**:

> **MSP là Main Stack Pointer, được chọn mặc định sau reset và Handler mode luôn dùng MSP. PSP là Process Stack Pointer mà Thread mode có thể chọn sử dụng, thường hữu ích để tách Stack ứng dụng/task khỏi Stack hệ thống.**

Nếu hỏi **“MSP được khởi tạo khi nào?”**:

> **Sau reset, processor tự động lấy initial MSP từ entry đầu tại địa chỉ `0x00000000` theo tài liệu.**

Nếu hỏi **“AAPCS liên quan Stack như thế nào?”**:

> **AAPCS quy định cách caller và callee sử dụng register và Stack khi gọi hàm, ví dụ bốn tham số đầu đi qua R0-R3, các tham số dư có thể đặt trên Stack và R4-R11 là nhóm callee phải bảo toàn nếu sử dụng theo tài liệu.**

Nếu hỏi **“Exception entry lưu những register nào?”**:

> **Theo slide nguồn, hardware tự động stacking R0-R3, R12, LR, PC và xPSR để tạo Stack Frame trước khi handler chạy.**

Nếu hỏi **“Stack Frame khi exception có ích gì cho debug?”**:

> **Nó giữ PC, LR và các register tại thời điểm exception, nên debugger có thể dùng chúng để phân tích vị trí và trạng thái chương trình khi fault xảy ra.**

---

### 1.9.36. Câu hỏi phỏng vấn tự kiểm tra

1. Stack Memory là gì?
2. Stack thường nằm trong loại bộ nhớ nào?
3. LIFO nghĩa là gì?
4. Stack thường lưu những loại dữ liệu nào?
5. SP là register nào?
6. Cortex-M sử dụng Stack model nào theo tài liệu?
7. Full và Empty khác nhau ở điểm nào?
8. Ascending và Descending khác nhau ở điểm nào?
9. Khi PUSH trên Full Descending Stack thì SP tăng hay giảm?
10. Khi POP trên Full Descending Stack thì SP tăng hay giảm?
11. `_estack` được hình nguồn dùng để biểu diễn gì?
12. MSP viết tắt của gì?
13. PSP viết tắt của gì?
14. Stack Pointer mặc định sau reset là MSP hay PSP?
15. Thread mode có thể dùng những Stack Pointer nào?
16. Handler mode dùng Stack Pointer nào?
17. Bit nào trong `CONTROL` được tài liệu nhắc tới để chọn PSP?
18. Vì sao phải khởi tạo PSP tới địa chỉ Stack hợp lệ trước khi dùng?
19. Dùng PSP cho Thread mode có lợi gì?
20. RTOS có thể sử dụng PSP như thế nào theo tài liệu?
21. AAPCS là gì?
22. Caller và Callee là gì?
23. Bốn tham số đầu của hàm được truyền qua những register nào theo nguồn text?
24. Nếu có nhiều hơn bốn tham số thì phần dư có thể được đặt ở đâu?
25. Những register nào được slide gọi là caller-saved?
26. Những register nào được gọi là callee-saved?
27. Khi callee sử dụng R4-R11 thì phải làm gì?
28. Khi exception xảy ra, hardware tự động lưu những register nào?
29. Một basic exception Stack Frame không FPU trong hình có bao nhiêu word?
30. Stacking và un-stacking xảy ra khi nào?
31. PC trong exception Stack Frame có ích gì khi debug?
32. MSP được processor khởi tạo từ vị trí nào trong vector table?
33. Linker script liên quan gì tới Stack?
34. Hãy mô tả luồng `Thread → exception → stacking → Handler → un-stacking → Thread`.

---

### 1.9.37. Tóm tắt

```text
Stack
→ vùng RAM tạm thời
→ LIFO
→ function / interrupt / exception
```

Cortex-M:

```text
Full Descending Stack

PUSH
→ SP giảm

POP
→ SP tăng
```

Stack Pointer:

```text
SP / R13
   │
   ├── MSP
   │   ├── mặc định sau reset
   │   └── Handler mode
   │
   └── PSP
       └── Thread mode có thể dùng
```

AAPCS:

```text
R0-R3
→ argument registers

Tham số dư
→ Stack

R0-R3, R12, LR
→ caller-saved theo slide

R4-R11
→ callee-saved
```

Exception entry:

```text
Thread mode
   ↓
Exception
   ↓
Hardware stacking
   ↓
R0 R1 R2 R3
R12 LR PC xPSR
   ↓
Handler mode
```

Exception return:

```text
Handler
   ↓
Hardware un-stacking
   ↓
Thread tiếp tục
```

**Ý quan trọng nhất:**

> **Stack trên Cortex-M là vùng RAM hoạt động theo mô hình Full Descending. MSP là Stack Pointer mặc định và luôn được Handler mode sử dụng, còn Thread mode có thể dùng MSP hoặc PSP. Stack còn đóng vai trò trung tâm trong AAPCS khi gọi hàm và trong cơ chế exception khi hardware tự động lưu/khôi phục context.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-10"></a>
## 1.10. Startup Code

### 1.10.1. Phạm vi của tài liệu Startup Code hiện có

Tài liệu nguồn `Startup_Code.docx` tập trung vào một câu hỏi chính:

> **Tại sao không được đặt Vector Table vào vùng `.data` đã khởi tạo trong RAM?**

Nguồn không trình bày toàn bộ nội dung của một file `startup.s` hay `startup.c`.

Vì vậy mục này chỉ triển khai các ý mà tài liệu hiện tại hỗ trợ:

```text
Reset
→ CPU cần Vector Table ngay lập tức

Vector Table
→ phải tồn tại trước khi bất kỳ code khởi tạo nào chạy

.data trong SRAM
→ chỉ được tạo/copy sau khi Reset_Handler đã bắt đầu chạy

Do đó:
Vector Table không thể phụ thuộc vào .data trong RAM
```

Các phần như weak handler, toàn bộ danh sách vector, cú pháp Assembly của startup file hoặc implementation cụ thể của vendor chưa có trong tài liệu nguồn này nên chưa được triển khai ở đây.

---

### 1.10.2. Startup Code nằm ở đâu trong luồng khởi động?

Có thể nối tài liệu này với các phần đã học:

```text
RESET
  ↓
CPU đọc Vector Table
  ↓
lấy initial SP
  ↓
lấy địa chỉ Reset_Handler
  ↓
nhảy vào Reset_Handler
  ↓
thực hiện khởi tạo
  ↓
main()
```

Điểm quan trọng:

> **CPU phải đọc được Vector Table trước khi Reset_Handler có cơ hội chạy bất kỳ đoạn code khởi tạo nào.**

Đây là nguyên nhân cốt lõi của toàn bộ mục này.

---

### 1.10.3. Hai word đầu của Vector Table được CPU dùng khi nào?

Theo tài liệu:

> Ngay khi reset, CPU đọc **2 word đầu của Vector Table**.

Hai giá trị đó là:

```text
Word đầu tiên
→ SP khởi tạo

Word thứ hai
→ địa chỉ Reset_Handler
```

Sơ đồ:

```text
Vector Table
+--------------------------+
| Initial Stack Pointer    | ← CPU đọc ngay sau reset
+--------------------------+
| Reset_Handler address    | ← CPU đọc ngay sau reset
+--------------------------+
| ...                      |
+--------------------------+
```

Điểm cần nhớ:

```text
CPU cần hai giá trị này
TRƯỚC
khi code startup bắt đầu chạy
```

---

### 1.10.4. Địa chỉ Vector Table trong ví dụ STM32F1

Nguồn ghi:

```text
STM32F1:
Flash được map ở 0x08000000
```

và mô tả CPU đọc Vector Table tại địa chỉ cố định trong quá trình reset.

Ở mức tài liệu hiện tại, cần nhớ mối quan hệ:

```text
Flash
→ chứa Vector Table

Vector Table
→ phải sẵn sàng ngay khi reset

CPU
→ đọc nó trước khi chạy code
```

Mục này không mở rộng thêm cơ chế remap hoặc các trường hợp boot khác vì nguồn chưa mô tả.

---

### 1.10.5. `.data` trong SRAM tồn tại khi nào?

Tài liệu nhấn mạnh:

> Vùng dữ liệu đã khởi tạo `.data` trong RAM chỉ được copy từ Flash xuống bởi `Reset_Handler` **sau khi CPU đã lấy được Vector Table và nhảy vào Reset_Handler**.

Do đó thứ tự là:

```text
1. CPU đọc Vector Table
        ↓
2. CPU lấy SP và Reset_Handler
        ↓
3. CPU nhảy vào Reset_Handler
        ↓
4. Reset_Handler mới copy .data
   từ Flash xuống SRAM
```

Điểm này rất quan trọng:

```text
Vector Table
→ phải tồn tại trước

.data trong SRAM
→ chỉ có nội dung đúng sau đó
```

---

### 1.10.6. Vì sao `.data` cần được copy từ Flash xuống SRAM?

Phần Flash/SRAM trước đó đã chỉ ra:

```text
.data
→ chứa initialized global/static variables
```

Các biến này cần:

```text
giá trị khởi tạo
+
khả năng đọc/ghi khi chạy
```

Do đó bố cục đã học là:

```text
FLASH
.data initial values
        │
        │ startup copy
        ↓
SRAM
.data runtime values
```

Startup Code hiện tại không mô tả chi tiết vòng lặp copy, nhưng xác nhận rõ:

> **Reset_Handler là phần thực hiện việc copy `.data` từ Flash xuống RAM.**

---

### 1.10.7. Tại sao không thể đặt Vector Table vào `.data`?

Đây là câu hỏi chính của tài liệu nguồn.

Giả sử Vector Table được đặt vào:

```text
.data
→ SRAM
```

Nhưng `.data` chỉ được copy sau khi:

```text
CPU đã đọc Vector Table
và
đã nhảy vào Reset_Handler
```

Điều này tạo ra vòng phụ thuộc không thể thực hiện:

```text
CPU muốn chạy Reset_Handler
        ↓
phải đọc Vector Table trước
        ↓
nhưng Vector Table lại ở .data trong SRAM
        ↓
.data chỉ được tạo bởi Reset_Handler
        ↓
Reset_Handler chưa thể chạy
```

Kết quả theo tài liệu:

```text
CPU không tìm được SP / Reset
→ không boot được
```

---

### 1.10.8. Vòng phụ thuộc sai nếu Vector Table nằm trong `.data`

Có thể biểu diễn rõ hơn:

```text
                RESET
                  │
                  ↓
        CPU cần Vector Table
                  │
                  ↓
       Vector Table ở .data?
                  │
                 Có
                  ↓
     .data trong SRAM chưa được copy
                  │
                  ↓
       CPU chưa có SP / Reset_Handler
                  │
                  ↓
        Không vào được Reset_Handler
                  │
                  ↓
         Không thể copy .data
                  │
                  ↓
              KHÔNG BOOT
```

Đây chính là lý do tài liệu yêu cầu Vector Table không phụ thuộc vào vùng `.data` runtime trong RAM.

---

### 1.10.9. Thứ tự đúng

Theo nội dung nguồn, thứ tự đúng phải là:

```text
Flash đã chứa Vector Table
        ↓
RESET
        ↓
CPU đọc initial SP
        ↓
CPU đọc Reset_Handler address
        ↓
CPU nhảy vào Reset_Handler
        ↓
Reset_Handler copy .data
Flash → SRAM
        ↓
sau đó chương trình mới tiếp tục các bước khởi tạo tiếp theo
```

Điểm phân biệt:

```text
Vector Table
→ phải tồn tại từ trước reset

.data trong SRAM
→ được chuẩn bị sau khi Reset_Handler chạy
```

---

### 1.10.10. Quan hệ giữa Startup Code và Reset Sequence

Phần **1.5 Reset Sequence** trả lời:

```text
CPU làm gì ngay sau reset?
```

Phần **1.10 Startup Code** hiện tại bổ sung:

```text
Tại sao dữ liệu mà CPU cần ngay lúc reset
không thể phụ thuộc vào vùng RAM
chỉ được startup code khởi tạo sau đó?
```

Ghép hai phần:

```text
Reset Sequence
      ↓
CPU đọc Vector Table
      ↓
CPU vào Reset_Handler
      ↓
Startup Code
      ↓
khởi tạo runtime data như .data
```

---

### 1.10.11. Quan hệ giữa Startup Code và Flash/SRAM

Phần **1.8 Flash và SRAM** đã cho thấy:

```text
Flash
├── Vector Table
├── .text
├── .rodata
└── initial values của .data

SRAM
├── .data runtime
├── .bss
├── Heap
└── Stack
```

Startup Code là cầu nối:

```text
Flash
  │
  │ copy initialized data
  ↓
SRAM
```

Trong phạm vi tài liệu hiện tại:

```text
Reset_Handler
→ copy .data từ Flash xuống SRAM
```

---

### 1.10.12. Vì sao Vector Table phải có sẵn trước code?

Theo nguồn:

> Việc CPU đọc hai word đầu của Vector Table xảy ra **trước khi bất kỳ code nào chạy**.

Do đó:

```text
Vector Table
≠ dữ liệu có thể chờ startup code tạo ra
```

Nó phải thuộc phần firmware mà processor có thể truy cập ngay khi reset.

Cách nhớ:

```text
Vector Table
→ điều kiện để bắt đầu chạy code

.data initialization
→ một công việc do code thực hiện sau đó
```

---

### 1.10.13. Sai lầm thường gặp về thứ tự

Sai:

```text
Reset_Handler chạy
     ↓
tạo Vector Table
     ↓
CPU đọc Vector Table
```

Đúng theo tài liệu:

```text
CPU đọc Vector Table
     ↓
CPU tìm Reset_Handler
     ↓
Reset_Handler chạy
     ↓
Reset_Handler copy .data
```

Không được đảo ngược thứ tự này.

---

### 1.10.14. Sơ đồ tổng hợp

```text
                 FLASH
+----------------------------------+
| Vector Table                     |
| ├── Initial SP                   |
| └── Reset_Handler address        |
+----------------------------------+
| .data initial values             |
+----------------------------------+

          │ RESET
          ↓

CPU đọc Vector Table
          │
          ├── lấy SP
          └── lấy Reset_Handler
                  │
                  ↓
            Reset_Handler
                  │
                  │ copy
                  ↓

                 SRAM
+----------------------------------+
| .data runtime                    |
+----------------------------------+
```

Thứ tự:

```text
Vector Table có trước
        ↓
Reset_Handler chạy
        ↓
.data mới được copy sang SRAM
```

---

### 1.10.15. Ý cần nhớ khi phỏng vấn

Nếu nhà tuyển dụng hỏi **“Tại sao Vector Table không được đặt trong `.data` ở RAM?”**, có thể trả lời:

> **Vì CPU cần đọc initial Stack Pointer và địa chỉ Reset_Handler từ Vector Table ngay khi reset, trước khi bất kỳ code nào chạy. Trong khi `.data` trong SRAM chỉ được Reset_Handler copy từ Flash xuống sau đó. Nếu Vector Table phụ thuộc vào `.data` trong RAM thì CPU chưa thể lấy SP và Reset_Handler để boot.**

Nếu hỏi **“`.data` được chuẩn bị khi nào?”**:

> **Theo tài liệu, `.data` trong RAM được Reset_Handler copy từ Flash xuống sau khi CPU đã đọc Vector Table và nhảy vào Reset_Handler.**

Nếu hỏi **“CPU dùng gì từ Vector Table ngay sau reset?”**:

> **Hai word đầu: initial Stack Pointer và địa chỉ Reset_Handler.**

Nếu hỏi **“Startup Code có liên hệ thế nào với Flash và SRAM?”**:

> **Trong nội dung nguồn hiện tại, Reset_Handler thuộc luồng startup và thực hiện việc copy `.data` từ Flash xuống SRAM để tạo bản dữ liệu runtime.**

---

### 1.10.16. Câu hỏi phỏng vấn tự kiểm tra

1. Startup Code trong tài liệu hiện tại tập trung vào vấn đề gì?
2. CPU đọc bao nhiêu word đầu của Vector Table ngay khi reset?
3. Word đầu tiên của Vector Table dùng để làm gì?
4. Word thứ hai dùng để làm gì?
5. Việc đọc Vector Table xảy ra trước hay sau khi code bắt đầu chạy?
6. `.data` trong SRAM được tạo/copy khi nào?
7. Ai thực hiện việc copy `.data` từ Flash xuống SRAM theo nguồn?
8. Vì sao Vector Table không thể phụ thuộc vào `.data` trong SRAM?
9. Điều gì xảy ra nếu CPU không lấy được initial SP và Reset_Handler?
10. Hãy mô tả vòng phụ thuộc sai nếu Vector Table nằm trong `.data`.
11. Hãy mô tả thứ tự đúng từ Reset tới lúc `.data` sẵn sàng trong SRAM.
12. Startup Code liên hệ thế nào với Reset Sequence?
13. Startup Code liên hệ thế nào với Flash/SRAM?
14. Tại sao Vector Table phải tồn tại trước khi Reset_Handler chạy?
15. Hãy trả lời câu phỏng vấn: “Tại sao không đặt Vector Table vào `.data`?”

---

### 1.10.17. Tóm tắt

```text
RESET
  ↓
CPU đọc Vector Table
  ↓
Initial SP
+
Reset_Handler address
  ↓
CPU nhảy vào Reset_Handler
  ↓
Reset_Handler copy .data
Flash → SRAM
```

Không được làm:

```text
Vector Table
→ đặt phụ thuộc vào .data trong SRAM
```

vì:

```text
.data chưa được copy
cho tới khi Reset_Handler chạy

nhưng

Reset_Handler chưa thể chạy
nếu CPU chưa đọc được Vector Table
```

Kết quả:

```text
không có SP / Reset_Handler
→ không boot được
```

**Ý quan trọng nhất:**

> **Vector Table phải tồn tại và truy cập được ngay khi reset, vì CPU cần nó để lấy initial Stack Pointer và địa chỉ Reset_Handler. `.data` trong SRAM chỉ được tạo sau đó bởi Reset_Handler, nên Vector Table không thể phụ thuộc vào `.data`.**

[↑ Về mục lục](#muc-luc)
