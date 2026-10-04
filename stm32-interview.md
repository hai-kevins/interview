# STM32 Architecture / Memory Map

> Trình bày nền tảng kiến trúc Cortex-M trong STM32, tổ chức bộ nhớ, cơ chế khởi động, Stack và quá trình liên kết chương trình.

---

<a id="muc-luc"></a>
## Mục lục

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
   - 1.10. Startup Code
   - 1.11. Linker Script và các section
2. **RCC + Clock**
   - 2.1. RCC là gì?
   - 2.2. Các nguồn Clock: HSI / HSE / LSI / LSE
   - 2.3. Clock Tree
   - 2.4. PLL
   - 2.5. SYSCLK / HCLK / PCLK1 / PCLK2
   - 2.6. Prescaler và cách tính tần số
   - 2.7. Peripheral Clock Enable và Peripheral Reset
   - 2.8. Clock của Timer
   - 2.9. Clock của các Peripheral quan trọng
   - 2.10. Quy trình cấu hình Clock
   - 2.11. Câu hỏi tự kiểm tra
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

Core có các thành phần/chức năng chính:

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

Core cũng chứa các thanh ghi đặc biệt như:

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

Một **Cortex-M4 processor** có thể được biểu diễn với các khối:

```text
Cortex-M4 processor
├── Cortex-M4 core
└── NVIC
```

Tức là:

> **Core là phần thực thi lệnh, còn processor bao gồm core và các khối hỗ trợ cần thiết để processor hoạt động trong hệ thống.**

Cortex-M4 có thể tích hợp FPU tùy biến thể triển khai.

> **Lưu ý:** Không phải mọi STM32 đều có FPU. Mỗi dòng STM32 có thể sử dụng một biến thể Cortex-M khác nhau và có tập tính năng khác nhau.

---

#### Microcontroller — vi điều khiển

**Microcontroller (MCU)** là một chip hoàn chỉnh tích hợp bộ xử lý cùng bộ nhớ và các ngoại vi.

Một STM32 có thể được biểu diễn ở mức khái niệm như sau:

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

Cortex-M3 / Cortex-M4 / Cortex-M7 / ...
= các processor core thuộc họ Cortex-M của Arm

Armv6-M / Armv7-M / Armv8-M / ...
= kiến trúc mà các core tương ứng triển khai
```

> **Thuật ngữ:** Không nên dùng `Cortex-M` như tên của một *kiến trúc* cụ thể. **STM32 là MCU; bên trong nó tích hợp một Cortex-M processor core; core đó triển khai một kiến trúc Arm-M tương ứng.**

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

**Fetch** là quá trình CPU lấy mã lệnh từ bộ nhớ chương trình, thường là Flash/ROM, thông qua bus để chuẩn bị giải mã và thực thi.

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

Các thanh ghi gồm:

```text
R0 → R15
xPSR
CONTROL
...
```

Thanh ghi là vùng lưu trữ rất gần với khối xử lý và được core sử dụng trực tiếp khi thực thi lệnh.

Có thể phân nhóm như sau:

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

Cortex-M4 có ba đường bus chính:

```text
I-Code bus
D-Code bus
System bus
```

Ý nghĩa:

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



```text
Core cần:
├── lấy Instruction
├── đọc/ghi Data
└── truy cập SRAM / Peripheral
```

nên có thể phân tách thành:

```text
Instruction → I-Code
Data        → D-Code
System      → System bus
```

Không cần ghi nhớ chi tiết bus transaction ở giai đoạn này.

---

### 1.1.7. AHB và APB — chỉ cần nhận diện trước

Hai loại bus chính được sử dụng là:

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

APB thường được nối với AHB thông qua:

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

Ví dụ các peripheral phía APB gồm:

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

### 1.1.10. Điểm cần nhớ

**Câu hỏi:** “Cortex-M và STM32 khác nhau như thế nào?”

> **Cortex-M là lõi/kiến trúc xử lý ARM dùng để thực thi lệnh. STM32 là một vi điều khiển hoàn chỉnh tích hợp Cortex-M cùng Flash, SRAM và các peripheral như GPIO, Timer, UART, SPI, I2C...**

**Câu hỏi:** “Core làm gì?”

> **Core lấy lệnh từ bộ nhớ, giải mã và thực thi lệnh. Bên trong core có ALU, các thanh ghi và các khối điều khiển thực thi.**

**Câu hỏi:** “Fetch là gì?”

> **Fetch là quá trình CPU lấy mã lệnh từ bộ nhớ chương trình thông qua bus để chuẩn bị giải mã và thực thi.**

**Câu hỏi:** “Core truy cập peripheral như thế nào?”

> **Core thực hiện các thao tác đọc/ghi thông qua hệ thống bus. Peripheral của vi điều khiển thường được ánh xạ vào không gian địa chỉ, nên CPU có thể truy cập các thanh ghi của peripheral thông qua địa chỉ tương ứng.**

Khái niệm **memory-mapped peripheral** sẽ được triển khai kỹ ở phần Memory Map.

---

### 1.1.11. Câu hỏi tự kiểm tra

1. Processor Core là gì?
2. Processor khác Processor Core ở điểm nào?
3. Microcontroller khác Processor ở điểm nào?
4. Cortex-M nằm ở đâu trong một vi điều khiển STM32?
5. ALU có nhiệm vụ gì?
6. Core sử dụng thanh ghi để làm gì?
7. Ba giai đoạn cơ bản khi xử lý một lệnh là gì?
8. Fetch nghĩa là gì?
9. CPU thường lấy mã chương trình từ đâu trong hệ thống nhúng?
10. I-Code bus được dùng cho mục đích gì ?
11. D-Code bus được dùng cho mục đích gì ?
12. System bus được dùng cho mục đích gì ?
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

 các bộ xử lý Cortex-M0/M3/M4 có **2 chế độ hoạt động**:

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

**Operational mode of the processor** là một đặc trưng quan trọng của Cortex-M0/M3/M4.

---

### 1.2.2. Thread mode



> Mã ứng dụng sẽ chạy trong **Thread mode**.

Có thể hình dung:

```text
Khởi động processor
        ↓
   Thread mode
        ↓
Chạy mã ứng dụng
```

Ví dụ:

```c
int main(void)
{
    while (1)
    {
        // Mã ứng dụng
    }
}
```

Khi chương trình đang thực thi luồng mã ứng dụng bình thường, processor ở **Thread mode**.

Không nên đồng nhất Thread mode với **User Mode** hay **Unprivileged**, vì đây là các khái niệm khác nhau.

> **Lưu ý:** Trong Cortex-M, **Thread mode không đồng nghĩa với Unprivileged/User**. Thread mode có thể chạy ở **Privileged** hoặc **Unprivileged**, còn Handler mode luôn Privileged. Nên dùng thuật ngữ **Thread mode** để tránh nhầm lẫn.

Phần **Access Level** ở mục 1.3 sẽ tách riêng khái niệm quyền truy cập khỏi Operation Mode.

---

### 1.2.3. Handler mode



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



> Processor luôn bắt đầu ở **Thread mode**.

Có thể ghi nhớ:

```text
Processor bắt đầu hoạt động
          ↓
      Thread mode
```

Ở mục này chưa đi sâu vào **Reset Sequence** hay startup code; các nội dung đó sẽ được triển khai riêng ở phần sau.

---

### 1.2.5. Khi nào Core chuyển từ Thread mode sang Handler mode?

Nội dung mô tả:

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

Cơ chế exception chi tiết được trình bày riêng ở phần **Exception / Interrupt**.

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
| Mục đích chính  | Chạy mã ứng dụng | Chạy exception/interrupt handler |
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

### 1.2.9. Điểm cần nhớ

**Câu hỏi:** “Cortex-M có những operation mode nào?”

> **Trong Cortex-M, processor có hai operation mode là Thread mode và Handler mode. Mã ứng dụng chạy trong Thread mode, còn exception handler và interrupt handler chạy trong Handler mode.**

**Câu hỏi:** “Processor bắt đầu ở mode nào?”

> **Processor bắt đầu ở Thread mode.**

**Câu hỏi:** “Khi interrupt xảy ra thì mode thay đổi như thế nào?”

> **Khi core gặp system exception hoặc external interrupt, core chuyển từ Thread mode sang Handler mode để thực thi handler hoặc ISR tương ứng.**

**Câu hỏi:** “Thread mode và Handler mode khác nhau ở điểm chính nào?”

> **Thread mode dành cho luồng mã ứng dụng bình thường, còn Handler mode dành cho việc xử lý exception và interrupt.**

---

### 1.2.10. Câu hỏi tự kiểm tra

1. Cortex-M0/M3/M4 có bao nhiêu operation mode ?
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

 Cortex-M0/M3/M4 cung cấp **2 mức truy cập**:

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

 khi mã chạy ở **Privileged Access Level (PAL)** thì nó có quyền truy cập đầy đủ hơn đối với:

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

Có thể nhớ:

> **Privileged mode/access level cho phép mã truy cập các tài nguyên và thanh ghi hệ thống mà mã không đặc quyền có thể bị hạn chế.**

---

### 1.3.3. Non-Privileged Access Level — mức truy cập không đặc quyền

 khi mã chạy ở **Non-Privileged Access Level (NPAL)** thì mã có thể **không được phép truy cập một số thanh ghi bị hạn chế của processor**.

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



> **Mặc định, mã bắt đầu chạy ở Privileged Access Level.**

Có thể ghép với phần Operation Modes trước đó:

```text
Processor bắt đầu
      ↓
Thread mode
      ↓
Privileged Access Level
```

Như vậy, ở trạng thái khởi đầu :

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

 khi đang ở Thread mode và Privileged, chương trình có thể chuyển processor sang Non-Privileged.

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

 processor sử dụng thanh ghi:

```text
CONTROL
```

để chuyển đổi mức truy cập khi ở Thread mode.

Cách viết rút gọn `CONTROL = 0/1` dễ gây nhầm vì `CONTROL` còn chứa các bit điều khiển khác.

Cách hiểu chính xác hơn:

```text
CONTROL.nPRIV — bit 0
0 → Thread mode Privileged
1 → Thread mode Unprivileged

CONTROL.SPSEL — bit 1
0 → Thread mode dùng MSP
1 → Thread mode dùng PSP
```

> Một số cách ký hiệu cũ có thể dùng tên trường khác như `TPL`/`ASPSEL`; ý nghĩa cốt lõi vẫn là **bit quyền của Thread mode** và **bit chọn Stack Pointer**.

Trong Handler mode, processor luôn Privileged và luôn sử dụng MSP.

---

### 1.3.8. Tại sao từ Non-Privileged không thể tự quay lại Privileged?

 khi Thread mode đã chuyển từ:

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

Do đó, nội dung mô tả con đường quay lại Privileged thông qua **Handler mode**.

---

### 1.3.9. Từ Non-Privileged quay lại Privileged như thế nào?



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
    │ CONTROL.nPRIV = 1
    ↓
Thread mode
Non-Privileged
    │
    │ Exception / Interrupt
    ↓
Handler mode
Privileged
    │
    │ CONTROL.nPRIV = 0
    ↓
Thread mode
Privileged
```

Điểm cốt lõi:

> **Handler mode luôn có quyền Privileged, nên handler có thể thực hiện thao tác cần quyền cao rồi đưa Thread mode trở về mức Privileged theo cơ chế mô tả trong phần này.**

---

### 1.3.10. Ghép Operation Mode và Access Level

Đây là phần dễ nhầm nhất.

Không nên nghĩ:

```text
Thread mode  = Non-Privileged
Handler mode = Privileged
```

Cách hiểu đúng :

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

### 1.3.11. Luồng chuyển trạng thái

Luồng chuyển trạng thái có thể biểu diễn như sau:

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
   được thực hiện theo cơ chế trên

6. Khi thoát handler:
   quay lại Thread mode
```

Sơ đồ tổng hợp:

```text
Thread / Privileged
        │
        │ CONTROL.nPRIV = 1
        ↓
Thread / Non-Privileged
        │
        │ Exception / Interrupt
        ↓
Handler / Privileged
        │
        │ CONTROL.nPRIV = 0
        ↓
Thread / Privileged
```

---

### 1.3.12. Tại sao cần Access Level?

Ý nghĩa chính:

```text
Privileged
→ truy cập tài nguyên hệ thống đầy đủ hơn

Non-Privileged
→ hạn chế truy cập một số tài nguyên/thanh ghi
```

Nhờ vậy, mã ứng dụng có thể được chạy với quyền thấp hơn, trong khi mã xử lý hệ thống/exception vẫn chạy với quyền cao hơn.

Điểm cốt lõi:

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

### 1.3.14. Điểm cần nhớ

**Câu hỏi:** “Cortex-M có những access level nào?”

> **Có hai mức truy cập: Privileged và Non-Privileged. Privileged có quyền truy cập đầy đủ hơn vào tài nguyên và các thanh ghi bị hạn chế của processor, còn Non-Privileged bị giới hạn một số quyền truy cập.**

**Câu hỏi:** “Thread mode có luôn là Non-Privileged không?”

> **Không. Thread mode có thể chạy ở Privileged hoặc Non-Privileged.**

**Câu hỏi:** “Handler mode chạy ở access level nào?”

> **Handler mode luôn chạy ở Privileged Access Level .**

**Câu hỏi:** “Tại sao mã Non-Privileged không thể tự nâng quyền trở lại?”

> **Vì nếu mã không đặc quyền có thể tự chuyển thành đặc quyền thì cơ chế giới hạn quyền sẽ không còn ý nghĩa.  muốn quay lại Privileged cần đi qua Handler mode.**

**Câu hỏi:** “Thanh ghi nào liên quan tới việc chuyển access level?”

> **Thanh ghi `CONTROL`. Nội dung minh họa `CONTROL.nPRIV = 0` cho Privileged và `CONTROL.nPRIV = 1` cho Non-Privileged; chi tiết từng bit sẽ học ở phần Core Registers.**

---

### 1.3.15. Câu hỏi tự kiểm tra

1. Cortex-M0/M3/M4 có bao nhiêu access level ?
2. Hai access level đó là gì?
3. Privileged Access Level cho phép truy cập những gì?
4. Non-Privileged Access Level bị hạn chế điều gì?
5. Access level mặc định là gì?
6. Thread mode có thể chạy ở những access level nào?
7. Handler mode chạy ở access level nào?
8. Operation Mode và Access Level có phải cùng một khái niệm không?
9. Thanh ghi nào được nội dung sử dụng để chuyển access level?
10. Từ Thread/Privileged có thể chuyển sang Thread/Non-Privileged như thế nào ở mức khái niệm?
11. Vì sao Thread/Non-Privileged không thể tự chuyển trực tiếp về Privileged?
12. Exception/interrupt có vai trò gì trong quá trình quay lại Privileged ?
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
   CONTROL.nPRIV = 1
        ↓
Thread / Non-Privileged
        ↓
Exception / Interrupt
        ↓
Handler / Privileged
        ↓
   CONTROL.nPRIV = 0
        ↓
Thread / Privileged
```

**Ý quan trọng nhất:**

> **Operation Mode cho biết processor đang chạy luồng ứng dụng hay handler; Access Level cho biết mức quyền truy cập của mã đang chạy. Thread mode có thể Privileged hoặc Non-Privileged, còn Handler mode luôn Privileged .**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-04"></a>
## 1.4. Core Registers

### 1.4.1. Tổng quan các thanh ghi của Processor Core

Các thanh ghi của processor core được chia thành các nhóm chính:

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

Phân nhóm:

- `R0` đến `R12` được gọi là **general-purpose registers**.
- `R0` đến `R7` được đánh dấu là **low registers**.
- `R8` đến `R12` được đánh dấu là **high registers**.
- `R13` là **Stack Pointer (SP)**.
- `R14` là **Link Register (LR)**.
- `R15` là **Program Counter (PC)**.

Các thanh ghi phía dưới như `PSR`, `PRIMASK`, `FAULTMASK`, `BASEPRI`, `CONTROL` được nhóm thành các **special registers**.

---

### 1.4.2. R0 → R12 — General-Purpose Registers

Sơ đồ:

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

Quy ước truyền tham số, giữ biến cục bộ và bảo toàn thanh ghi được trình bày ở phần AAPCS.

---

### 1.4.3. R13 — Stack Pointer

Theo sơ đồ:

```text
R13
 ↓
SP — Stack Pointer
```

`SP` có hai phiên bản banked:

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

Điểm cần nhớ:

> **R13 là Stack Pointer; hai Stack Pointer banked là PSP và MSP.**

Cách PSP/MSP được chọn, mode nào dùng SP nào và cách Stack hoạt động sẽ được triển khai riêng trong phần **Stack cơ bản trên Cortex-M**.

---

### 1.4.4. R14 — Link Register

Theo sơ đồ:

```text
R14
 ↓
LR — Link Register
```

`LR` giữ vai trò quan trọng khi một hàm gọi hàm khác.

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

Luồng thực thi:

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

> **LR giữ thông tin địa chỉ quay về khi thực hiện lời gọi hàm.**

---

### 1.4.5. R15 — Program Counter



```text
R15
 ↓
PC — Program Counter
```



> `PC` là thanh ghi `R15` và chứa địa chỉ chương trình hiện tại.

Có thể hiểu:

```text
PC
→ cho core biết vị trí lệnh trong luồng chương trình
```

Trong lời gọi hàm:

```text
fun1 gọi fun2
      ↓
PC nhảy tới địa chỉ của fun2
```

Khi `fun2` kết thúc:

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

Trong quá trình Reset:

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

PSR là thanh ghi trạng thái chương trình; cấu trúc chi tiết được trình bày ngay sau đây.

> **PSR là thanh ghi trạng thái chương trình của processor.**

Các thành phần `xPSR`, `APSR`, `IPSR`, `EPSR` và T-bit được trình bày dưới đây.


---

#### `xPSR`, `APSR`, `IPSR`, `EPSR` và T-bit

Ở Cortex-M, `xPSR` là cách nhìn tổng hợp của các nhóm trạng thái:

```text
xPSR
├── APSR → trạng thái kết quả phép toán, ví dụ N/Z/C/V
├── IPSR → exception number hiện tại
└── EPSR → trạng thái thực thi
```

Trong `EPSR` có **T-bit** liên quan tới Thumb state.

Điểm cần nhớ:

```text
Cortex-M
→ chỉ thực thi Thumb/Thumb-2 theo core tương ứng
→ không có ARM state cổ điển
→ T-bit phải ở trạng thái hợp lệ (= 1)
```

Khi một địa chỉ handler được nạp vào `PC`, **bit 0 của giá trị địa chỉ được dùng để biểu thị Thumb state**. Vì vậy các entry trong Vector Table thường có bit 0 bằng `1`.

Ví dụ khái niệm:

```text
Reset_Handler thực thi từ vùng quanh 0x08000100
Vector entry có thể chứa      0x08000101
                                      ↑
                                  bit 0 = 1
                                  Thumb state
```

Không nên hiểu `0x08000101` là CPU thực thi instruction ở một byte lẻ theo nghĩa thông thường; bit thấp nhất mang thông tin trạng thái thực thi.

Nếu T-bit không hợp lệ trên Cortex-M, processor có thể phát sinh **UsageFault** trên các core hỗ trợ fault tương ứng.

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

Do đó:

> **PRIMASK, FAULTMASK và BASEPRI là các thanh ghi liên quan đến việc mask exception.**

Chi tiết từng thanh ghi mask exception được trình bày ở phần Interrupt / Exception.

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
→ được dùng để chuyển
  Thread mode giữa Privileged và Non-Privileged
```

Các bit quan trọng của `CONTROL` được trình bày ở phần Access Level và Stack.

---

### 1.4.10. Non-Memory-Mapped Registers là gì?

Có thể phân biệt:

```text
Non-memory mapped registers
```

và:

```text
Memory mapped registers
```

Các **processor core registers** được đặt phía **Non-memory mapped registers**.

Nội dung ghi:

- Các core register này **không có địa chỉ duy nhất để truy cập như một địa chỉ trong memory map**.
- Vì vậy chúng **không thuộc processor memory map**.
- Không thể truy cập chúng trong chương trình C bằng cách lấy một địa chỉ cố định rồi giải tham chiếu như với peripheral register.
- Để truy cập trực tiếp các thanh ghi này cần sử dụng instruction phù hợp ở mức Assembly.

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

Nhóm **Memory-mapped registers** gồm:

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
| Ví dụ trong phần này | `R0-R15`, `PSR`, `CONTROL`... | NVIC, MPU, SCB, RTC, I2C, TIMER... |
| Có địa chỉ trong memory map | Không theo cách nội dung mô tả | Có |
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

Đây là phần quan trọng cần nắm chắc.



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

### 1.4.15. Điểm cần nhớ

**Câu hỏi:** “Các core register chính của Cortex-M là gì?”

> ** Cortex-M có các general-purpose register R0-R12, R13 là Stack Pointer, R14 là Link Register, R15 là Program Counter, cùng các special register như PSR, PRIMASK, FAULTMASK, BASEPRI và CONTROL.**

**Câu hỏi:** “R14/LR dùng để làm gì?”

> **LR giữ địa chỉ quay về. Khi caller gọi callee, PC chuyển tới hàm được gọi còn LR giữ địa chỉ của lệnh tiếp theo để khi return có thể quay lại caller.**

**Câu hỏi:** “R15/PC dùng để làm gì?”

> **PC là Program Counter, tức R15, chứa địa chỉ chương trình đang được processor dùng để điều khiển luồng thực thi. Khi gọi hàm, PC chuyển tới địa chỉ callee; khi return, PC nhận lại địa chỉ quay về từ LR.**

**Câu hỏi:** “Core register có nằm trong memory map không?”

> ** các core register như R0-R15 là non-memory-mapped register, không có địa chỉ riêng trong processor memory map như peripheral register.**

**Câu hỏi:** “Memory-mapped register là gì?”

> **Là register có địa chỉ trong memory map. Nội dung đưa ví dụ các register của NVIC, MPU, SCB hoặc peripheral của vi điều khiển như I2C, Timer, CAN, USB; chương trình C có thể truy cập chúng thông qua địa chỉ tương ứng.**

**Câu hỏi:** “R13 có gì đặc biệt?”

> **R13 là Stack Pointer và nội dung cho thấy nó có hai phiên bản banked là PSP và MSP.**

---

### 1.4.16. Câu hỏi tự kiểm tra

1. R0 đến R12 thuộc nhóm register nào?
2. R0-R7 và R8-R12 được sơ đồ gọi là gì?
3. R13 là register gì?
4. Hai phiên bản Stack Pointer được nội dung thể hiện là gì?
5. R14 là register gì?
6. R15 là register gì?
7. Khi caller gọi callee, LR giữ thông tin gì?
8. Khi gọi hàm, PC thay đổi như thế nào?
9. Khi hàm return, mối quan hệ `PC = LR` có ý nghĩa gì?
10. Khi reset, nội dung nói PC được nạp từ địa chỉ nào?
11. `PSR` được nội dung mô tả là loại register gì?
12. `PRIMASK`, `FAULTMASK` và `BASEPRI` được nhóm thành loại register gì?
13. `CONTROL` thuộc nhóm register nào?
14. Processor core register có thuộc memory map không ?
15. Peripheral register memory-mapped khác core register ở điểm nào?
16. Hãy kể các ví dụ processor-specific peripheral register.
17. Hãy kể các ví dụ microcontroller-specific peripheral register.
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

> **R0-R12 là các general-purpose register, R13 là Stack Pointer, R14 là Link Register và R15 là Program Counter. Các core register này được nội dung xếp vào nhóm non-memory-mapped, khác với các register của peripheral được ánh xạ vào memory map và có thể truy cập bằng địa chỉ.**

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

 sau reset processor đọc hai vị trí bộ nhớ đầu tiên:

```text
0x00000000
0x00000004
```

Ý nghĩa :

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

#### Boot alias/remap: Vector Table tại `0x00000000` và Main Flash tại `0x08000000`

Đây là điểm rất dễ gây nhầm khi ghép **Memory Map** với **Reset Sequence**.

Ở góc nhìn của Cortex-M khi reset:

```text
0x00000000
→ initial MSP

0x00000004
→ Reset vector
```

Trong nhiều STM32, **Main Flash có địa chỉ vật lý bắt đầu tại `0x08000000`**. Để core vẫn có thể lấy vector ở `0x00000000`, STM32 sử dụng cơ chế **boot mapping / alias / remap**.

Khi boot từ Main Flash, có thể hình dung:

```text
Địa chỉ vật lý Main Flash
0x08000000
      │
      │ được ánh xạ/alias vào boot space
      ↓
0x00000000
      │
      ├── initial MSP
      └── Reset vector
```

Vì vậy hai phát biểu sau **không mâu thuẫn**:

```text
Cortex-M đọc vector tại 0x00000000 khi reset

STM32 Main Flash thường bắt đầu tại 0x08000000
```

Tùy dòng STM32 và cấu hình boot, boot space tại `0x00000000` có thể ánh xạ tới:

```text
Main Flash
System Memory / bootloader
SRAM
```

Sau khi hệ thống đã chạy, Vector Table còn có thể được **relocate** bằng thanh ghi `VTOR` trên các core hỗ trợ tính năng này.

> **Điểm cần nhớ:** `0x00000000` là **reset/boot view mà core nhìn thấy**, còn `0x08000000` là địa chỉ Main Flash điển hình của STM32. Cơ chế alias/remap nối hai phần này lại với nhau.

---

### 1.5.3. Bước 1 — Processor bắt đầu chuỗi reset

bước đầu tiên bằng việc processor bắt đầu tại vùng địa chỉ:

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



```text
MSP = value @ 0x00000000
```

Trong đó:

```text
MSP
= Main Stack Pointer
```

Điểm quan trọng:

> Processor trước tiên khởi tạo Stack Pointer.

Ví dụ minh họa:

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

Địa chỉ `0x20008000` chỉ là **giá trị minh họa** cho initial MSP.

---

### 1.5.5. Tại sao phải khởi tạo MSP trước?

 processor khởi tạo Main Stack Pointer trước khi đi tiếp tới Reset Handler.

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

Sau khi khởi tạo MSP, nội dung mô tả processor đọc giá trị tại:

```text
0x00000004
```

Giá trị này chính là:

```text
địa chỉ của Reset Handler
```



```text
PC = value @ 0x00000004
```

Ví dụ minh họa:

```text
Memory[0x00000004] = 0x20001000
```

thì:

```text
PC = 0x20001000
```

và:

```text
0x20001000
→ địa chỉ bắt đầu của Reset Handler trong ví dụ
```

Địa chỉ trên chỉ là **địa chỉ minh họa trong sơ đồ**, không nên ghi nhớ như một địa chỉ cố định cho mọi STM32.

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

Sơ đồ mô tả:

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

Luồng thực thi:

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

 Reset Handler có các trách nhiệm chính trước khi gọi `main()`:

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

Bước khởi tạo thư viện C có thể gọi:

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

Luồng tổng hợp:

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

### 1.5.13. Ví dụ minh họa

Ví dụ:

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

Hai giá trị `0x20008000` và `0x20001000` chỉ là **ví dụ minh họa cho cơ chế**, không phải giá trị bắt buộc của mọi chương trình STM32.

---

### 1.5.14. Reset Sequence khác `main()` như thế nào?

Một điểm dễ nhầm:

```text
Reset
≠
nhảy thẳng vào main()
```



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

### 1.5.15. Điểm cần nhớ

**Câu hỏi:** “Cortex-M làm gì ngay sau reset?”

> **Processor lấy initial MSP từ giá trị tại địa chỉ `0x00000000`, sau đó lấy địa chỉ Reset Handler từ `0x00000004`, nạp địa chỉ đó vào PC và bắt đầu chạy Reset Handler.**

**Câu hỏi:** “Giá trị tại `0x00000000` dùng làm gì?”

> ** đó là initial value được nạp vào MSP — Main Stack Pointer.**

**Câu hỏi:** “Giá trị tại `0x00000004` là gì?”

> **Đó là địa chỉ của Reset Handler, được processor đọc để đưa luồng thực thi tới Reset Handler.**

**Câu hỏi:** “Reset Handler làm gì?”

> **Reset Handler thực hiện các bước khởi tạo cần thiết trước khi gọi `main()`: khởi tạo data section, bss section, C standard library rồi gọi `main()`.**

**Câu hỏi:** “`main()` có phải lệnh đầu tiên chạy sau reset không?”

> **Không.  processor chạy Reset Handler trước; Reset Handler thực hiện các bước khởi tạo rồi mới gọi `main()`.**

**Câu hỏi:** “MSP và PC liên quan gì tới Reset Sequence?”

> **MSP được khởi tạo từ vector đầu tiên tại `0x00000000`, còn PC nhận địa chỉ Reset Handler từ vector tại `0x00000004`.**

---

### 1.5.16. Câu hỏi tự kiểm tra

1. Reset Sequence là gì?
2. Hai địa chỉ đầu tiên mà nội dung nhấn mạnh là gì?
3. Giá trị tại `0x00000000` được nạp vào thanh ghi nào?
4. MSP viết tắt của gì?
5. Tại sao nội dung nói Stack Pointer được khởi tạo trước?
6. Giá trị tại `0x00000004` đại diện cho gì?
7. Thanh ghi nào nhận địa chỉ Reset Handler?
8. Sau khi PC nhận địa chỉ Reset Handler thì processor làm gì?
9. Reset Handler là gì?
10. Reset Handler có thể được viết bằng ngôn ngữ nào ?
11. Reset Handler chạy trước hay sau `main()`?
12. Vector table có vai trò gì trong quá trình reset?
13. Những bước khởi tạo nào diễn ra trước `main()`?
14. `__libc_init_array()` được gọi ở bước nào?
15. `.data` và `.bss` được xử lý trước hay sau `main()`?
16. Hai địa chỉ `0x20008000` và `0x20001000` có phải giá trị cố định cho mọi STM32 không?
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

Trọng tâm của mục này là:

```text
AMBA
├── AHB-Lite
└── APB
```

---

### 1.6.2. AMBA là gì?



```text
AMBA
= Advanced Microcontroller Bus Architecture
```

AMBA là một đặc tả do ARM thiết kế để quy định chuẩn giao tiếp **on-chip** bên trong một System-on-Chip.

Có thể hiểu:

```text
AMBA
→ bộ quy tắc / đặc tả
→ chuẩn hóa cách các khối trong chip giao tiếp với nhau
```

Sơ đồ cho biết AMBA hỗ trợ nhiều bus protocol, trong đó mục này tập trung vào:

```text
AHB-Lite
APB
```

---

### 1.6.3. AHB-Lite



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

Trong kiến trúc bus, các đường:

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

APB được dùng cho giao tiếp tốc độ thấp hơn AHB.

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

Theo cách tổ chức trong phần này:

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

APB được truy cập thông qua:

```text
AHB-APB Bridge
```

Vai trò:

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

### 1.6.7. Các bus interface của Cortex-M

Bốn đường/interface quan trọng gồm:

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

> **I-CODE phục vụ việc lấy lệnh và đọc vector table từ CODE region trong sơ đồ.**

---

### 1.6.9. D-CODE bus



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

trong vùng CODE.

---

### 1.6.10. System bus



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

> **System bus phục vụ các truy cập đọc/ghi tới các vùng hệ thống như SRAM và peripheral trong sơ đồ.**

---

### 1.6.11. PPB interface

Ngoài ra còn có đường:

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

Ngoài ra còn có một vùng:

```text
MCU Vendor specific region
```

ở phía trên.

Ở đây chỉ cần nhận diện PPB như một vùng/interface riêng của processor; cấu trúc register chi tiết được trình bày ở phần System Control/NVIC.

```text
PPB
→ một bus/interface và vùng truy cập riêng được thể hiện trong sơ đồ
```

Khái niệm các thanh ghi memory-mapped liên quan tới processor/peripheral đã được giới thiệu ở mục **1.4. Core Registers**. Phần **1.7. Memory Map** tập trung vào vị trí các vùng đó trong không gian địa chỉ.

---

### 1.6.12. Sơ đồ tổng hợp

Có thể diễn giải sơ đồ thành:

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

#### Bus của Cortex-M và bus cụ thể của STM32

Cần tách hai lớp kiến thức:

```text
Cortex-M processor side
→ I-CODE / D-CODE / System bus
→ mô hình truy cập của core

STM32 chip interconnect
→ AHB / APB1 / APB2 / bus matrix ...
→ cách ST kết nối bộ nhớ và peripheral trong từng dòng MCU
```

Do đó không nên suy ra rằng mọi STM32 có cấu trúc bus giống hệt nhau chỉ vì chúng cùng dùng Cortex-M.

Ở phần **RCC + Clock**, các bus như AHB/APB sẽ còn liên quan tới:

```text
clock source
prescaler
peripheral clock
bus clock
```

> **Cách nhớ:** phần này giải thích **đường truy cập dữ liệu/lệnh**; phần RCC giải thích **cách các bus/peripheral được cấp clock** trên từng STM32 cụ thể.

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
| Vai trò  | Main bus interface | Peripheral bus |
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

| Bus | Mục đích |
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

### 1.6.17. Điểm cần nhớ

**Câu hỏi:** “AMBA là gì?”

> **AMBA là đặc tả bus do ARM thiết kế để chuẩn hóa giao tiếp on-chip. Trong phần này, hai protocol chính được nhắc tới là AHB-Lite và APB.**

**Câu hỏi:** “AHB-Lite và APB khác nhau thế nào?”

> **AHB-Lite được dùng cho main bus interface và các giao tiếp tốc độ cao hơn, còn APB được dùng cho nhiều peripheral không cần tốc độ cao và thường được nối với AHB qua AHB-APB Bridge.**

**Câu hỏi:** “I-CODE dùng để làm gì?”

> ** I-CODE dùng cho instruction fetch và vector table read từ CODE region.**

**Câu hỏi:** “D-CODE dùng để làm gì?”

> **D-CODE dùng để truy cập dữ liệu trong CODE region.**

**Câu hỏi:** “System bus dùng để làm gì?”

> **System bus dùng cho các truy cập đọc/ghi tới các vùng như SRAM, peripheral, external RAM và device region.**

**Câu hỏi:** “AHB-APB Bridge có vai trò gì?”

> **Nó làm cầu nối giữa phía AHB và phía APB để processor có thể truy cập các peripheral được kết nối trên APB.**

---

### 1.6.18. Câu hỏi tự kiểm tra

1. Bus Architecture dùng để giải quyết vấn đề gì?
2. AMBA là gì ?
3. AMBA do ai thiết kế?
4. Hai bus protocol được nội dung nhắc tới là gì?
5. AHB-Lite được dùng chủ yếu cho loại giao tiếp nào?
6. APB được dùng chủ yếu cho loại giao tiếp nào?
7. Vì sao hệ thống cần cả AHB và APB?
8. AHB-APB Bridge có vai trò gì?
9. I-CODE bus dùng để làm gì?
10. I-CODE đọc những gì từ CODE region?
11. D-CODE bus dùng để làm gì?
12. System bus truy cập những vùng nào?
13. Các bus trong sơ đồ có độ rộng bao nhiêu bit?
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

Các bus interface:

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

Nội dung nêu:

> Phạm vi địa chỉ mà processor có thể truy cập phụ thuộc vào kích thước của address bus.

Có thể biểu diễn:

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

Không gian địa chỉ được chia thành các vùng lớn:

| Vùng | Khoảng địa chỉ | Kích thước |
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



```text
Code Region
0x00000000 → 0x1FFFFFFF
512 MB
```

Đây là vùng mà nhà sản xuất MCU có thể kết nối **CODE memory**, ví dụ:

```text
Embedded Flash
ROM
OTP
EEPROM
...
```



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
→ nơi chứa bộ nhớ chương trình
→ processor fetch thông tin vector table từ đây sau reset
```

---

### 1.7.5. Quan hệ Code Region với I-CODE và D-CODE

Liên hệ với Bus Architecture:

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



```text
SRAM Region
0x20000000 → 0x3FFFFFFF
512 MB
```

Nội dung mô tả:

- Đây là 512 MB tiếp theo sau CODE region.
- Chủ yếu dùng để kết nối SRAM, thường là on-chip SRAM.
- Có thể thực thi program code từ vùng này.
- Phần đầu của vùng có thể hỗ trợ bit-band tùy core/implementation.

Có thể nhớ:

```text
SRAM Region
→ vùng dành chủ yếu cho SRAM
→ processor có thể đọc/ghi dữ liệu ở đây
→ có thể thực thi code từ vùng này
```

---

### 1.7.7. Bit-Band trong SRAM Region — chỉ nhận diện

Sơ đồ thể hiện:

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

Cơ chế bit-band được trình bày ngay sau đây.

Chỉ cần nhận diện:

```text
Bit-Band Region
Bit-Band Alias
```

và các địa chỉ tương ứng.

---

### 1.7.8. Peripheral Region



```text
Peripheral Region
0x40000000 → 0x5FFFFFFF
512 MB
```

Nội dung mô tả:

- Chủ yếu dành cho các on-chip peripherals.
- Phần 1 MB đầu có thể là vùng bit-addressable nếu tính năng bit-band tùy chọn được hỗ trợ.
- Đây là vùng **Execute Never (XN)**.
- Cố thực thi code từ vùng này sẽ gây fault exception trong sơ đồ.

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

Cơ chế và công thức ánh xạ bit-band được trình bày ngay sau đây.


---

#### Cơ chế Bit-band

Bit-band cho phép phần mềm thao tác **một bit** trong một vùng nhớ gốc thông qua một **word riêng trong alias region**.

Không dùng bit-band, việc sửa một bit thường có dạng:

```text
READ cả byte/word
    ↓
MODIFY bit cần đổi
    ↓
WRITE cả byte/word trở lại
```

Đây là chuỗi **read-modify-write**. Nếu một tác nhân khác thay đổi các bit khác giữa lúc đọc và ghi, phần mềm có nguy cơ ghi đè trạng thái mới đó.

Với bit-band:

```text
1 bit trong bit-band region
        ↕ ánh xạ
1 word trong alias region
```

Ghi vào alias word:

```text
0 → clear bit tương ứng
1 → set bit tương ứng
```

Công thức ánh xạ khái niệm:

```text
alias_address
= alias_base
+ byte_offset × 32
+ bit_number × 4
```

Ví dụ ý tưởng:

```text
bit thứ n của một byte trong SRAM bit-band region
        ↓
được ánh xạ thành một word 32-bit riêng trong alias region
        ↓
CPU đọc/ghi alias word để đọc/đổi bit đó
```

Điểm quan trọng là phần mềm **không phải tự viết chuỗi read-modify-write** để đổi một bit.

> **Lưu ý:** không được mặc định mọi Cortex-M hoặc mọi dòng STM32 đều hỗ trợ bit-band. Chỉ sử dụng khi programming manual/reference manual của core/MCU cụ thể xác nhận có vùng bit-band.

---

### 1.7.10. External RAM Region



```text
External RAM Region
0x60000000 → 0x9FFFFFFF
1 GB
```

Nội dung mô tả:

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



```text
External Device Region
0xA0000000 → 0xDFFFFFFF
1 GB
```

Nội dung mô tả:

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

Sơ đồ tổng thể của nội dung đặt phần cuối của không gian địa chỉ từ:

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

Bên trong vùng PPB có các khối như:

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

### 1.7.13. Phân biệt External Device và PPB

Trong Memory Map của Cortex-M:

```text
0xA0000000 → 0xDFFFFFFF
→ External Device Region

0xE0000000 trở lên
→ PPB / System Region
```

Vùng PPB chứa các khối hệ thống của processor như:

```text
NVIC
System timer
System Control Block
ITM
DWT
FPB
...
```

Các vùng system/private peripheral không được dùng như vùng thực thi chương trình thông thường.

---

### 1.7.14. Memory Map và Peripheral Register

Ví dụ với ADC:

```text
ADC có dữ liệu
     ↓
dữ liệu nằm trong một thanh ghi của ADC
     ↓
CPU cần đọc thanh ghi đó
```

CPU thực hiện:

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

Luồng dữ liệu có thể tóm tắt:

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

Processor có:

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

Các vùng 512 MB hoặc 1 GB là **phạm vi được dành trong bản đồ địa chỉ** cho từng loại tài nguyên.

Có thể nhớ:

```text
Address space
≠
dung lượng bộ nhớ vật lý thực tế
```

---

### 1.7.17. Tóm tắt các vùng chính

| Vùng | Base address | Vai trò chính  |
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
| sơ đồ                           |
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

### 1.7.19. Điểm cần nhớ

**Câu hỏi:** “Memory Map là gì?”

> **Memory Map là cách processor chia không gian địa chỉ thành các vùng dành cho code, SRAM, peripheral, external memory và system resources. Nó cho biết một địa chỉ cụ thể tương ứng với loại tài nguyên nào.**

**Câu hỏi:** “Cortex-M có không gian địa chỉ bao nhiêu?”

> ** processor có kênh địa chỉ 32-bit nên có tối đa 4 GB không gian địa chỉ, từ `0x00000000` đến `0xFFFFFFFF`.**

**Câu hỏi:** “Code, SRAM và Peripheral bắt đầu ở đâu?”

> **Theo sơ đồ: Code bắt đầu tại `0x00000000`, SRAM tại `0x20000000`, và Peripheral tại `0x40000000`.**

**Câu hỏi:** “Peripheral Region dùng để làm gì?”

> **Đây là vùng chủ yếu dành cho on-chip peripheral và là vùng Execute Never, vì vậy không dùng để thực thi code.**

**Câu hỏi:** “Memory Map liên quan gì đến peripheral register?”

> **Mỗi peripheral register có thể được gắn với một địa chỉ trong không gian địa chỉ. CPU đưa địa chỉ đó lên bus để truy cập đúng register; nội dung minh họa bằng việc CPU đọc dữ liệu từ ADC register rồi chuyển dữ liệu vào memory.**

**Câu hỏi:** “4 GB addressable memory có nghĩa STM32 có 4 GB RAM không?”

> **Không. 4 GB là không gian địa chỉ mà processor có thể biểu diễn; từng MCU thực tế chỉ triển khai một phần trong các vùng đó.**

---

### 1.7.20. Câu hỏi tự kiểm tra

1. Memory Map là gì?
2. Kích thước address bus ảnh hưởng tới điều gì?
3. Kênh địa chỉ có độ rộng bao nhiêu bit?
4. 32-bit address space cho tối đa bao nhiêu không gian địa chỉ?
5. Không gian địa chỉ bắt đầu và kết thúc ở đâu?
6. Code Region bắt đầu tại địa chỉ nào?
7. Code Region có kích thước bao nhiêu?
8. Processor dùng Code Region cho những loại memory nào ?
9. Vector table liên quan gì tới Code Region?
10. SRAM Region bắt đầu tại địa chỉ nào?
11. Peripheral Region bắt đầu tại địa chỉ nào?
12. Peripheral Region là Execute Never có nghĩa là gì?
13. External RAM Region nằm trong khoảng nào?
14. External Device Region nằm trong khoảng nào?
15. System/PPB region bắt đầu từ vùng địa chỉ nào theo sơ đồ tổng thể?
16. Bit-Band Region của SRAM bắt đầu tại đâu?
17. Bit-Band Alias của SRAM bắt đầu tại đâu?
18. Bit-Band Region của Peripheral bắt đầu tại đâu?
19. Bit-Band Alias của Peripheral bắt đầu tại đâu?
20. Memory Map và Bus Architecture khác nhau ở điểm nào?
21. CPU đọc ADC register bằng địa chỉ như thế nào?
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

Hai loại bộ nhớ chính cần phân biệt là:

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



- Flash là bộ nhớ **không mất dữ liệu khi mất nguồn** — non-volatile.
- Dùng để lưu:
  - firmware;
  - hằng số;
  - mã chương trình.
- Flash là bộ nhớ không mất dữ liệu, có thể xóa và lập trình lại theo quy trình của MCU.
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
- Cần phân biệt hai trường hợp:
  - **Mất nguồn:** SRAM là volatile nên không được kỳ vọng giữ dữ liệu.
  - **Reset:** không nên kết luận rằng toàn bộ SRAM luôn bị phần cứng xóa sạch. Sau reset, startup code thường **khởi tạo lại `.data` và `.bss`**, còn nội dung những vùng RAM khác phụ thuộc loại reset, dòng MCU và thiết kế hệ thống.

Có thể hình dung:

```text
Chương trình đang chạy
        ↓
SRAM thay đổi liên tục

Mất nguồn
        ↓
không giữ dữ liệu

Reset
        ↓
startup code tái khởi tạo .data / .bss
        ↓
không nên dựa vào dữ liệu RAM cũ nếu đặc tả của MCU không bảo đảm
```

---

### 1.8.4. So sánh Flash và SRAM

| Tiêu chí | Flash | SRAM |
|---|---|---|
| Khả năng giữ dữ liệu khi mất nguồn | Có | Không |
| Vai trò chính  | Firmware, code, hằng số | Dữ liệu đọc/ghi khi chạy |
| CPU sử dụng khi chạy | Chủ yếu đọc | Đọc và ghi |
| Ghi dữ liệu mới | Cần thủ tục đặc biệt | Có thể đọc/ghi trực tiếp trong quá trình chạy |
| Thành phần điển hình | Vector Table, `.text`, `.rodata`, bản sao khởi tạo `.data` | `.data`, `.bss`, Heap, Stack |

Cách nhớ:

```text
Flash
→ lưu lâu dài

SRAM
→ vùng làm việc khi chương trình đang chạy
```

---

### 1.8.5. Bố cục Flash

Code Memory bắt đầu tại:

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

Sơ đồ đặt:

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

Sơ đồ đặt:

```text
.text
```

trong Flash.

Có thể hiểu:

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

Sơ đồ đặt:

```text
.rodata
```

trong Flash, phía trên `.text`.

Các hằng số không cần đặt trong SRAM nếu chúng không thay đổi:

```text
const / read-only data
→ CPU chỉ cần đọc
→ không cần dùng SRAM để chứa một bản đọc/ghi
```

Nếu chuyển các hằng số từ Flash sang SRAM mỗi lần reset sẽ:

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

Đây là điểm quan trọng nhất.

`.data` xuất hiện **cả ở Flash và SRAM**:

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

Do đó:

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

Theo bố cục bộ nhớ:

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

Quá trình này là:

```text
Transferring of .data section to RAM
('C' start-up)
```

---

### 1.8.11. `_etext`, `_sdata`, `_edata`

Ba boundary symbol chính:

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

Đây là các **boundary symbol** dùng để xác định vùng dữ liệu cần copy.

Chi tiết cách linker tạo các symbol này sẽ được học ở phần **Linker Script**.

---

### 1.8.12. `.bss` nằm ở đâu?

`.bss` nằm trong SRAM:

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

Theo sơ đồ:

```text
.bss
→ uninitialized global variables
→ uninitialized static variables
```

Hiệu chỉnh theo cách hiểu C/linker chính xác hơn:

> **`.bss` thường chứa các đối tượng có static storage duration được zero-initialize.** Điều này bao gồm biến global/static không ghi initializer; compiler/linker cũng thường có thể đặt các biến khởi tạo bằng `0` vào `.bss`.

Ví dụ:

```c
int counter;          // zero-initialized
static int status;    // zero-initialized
static int error = 0; // thường cũng có thể nằm trong .bss
```

Trước khi vào `main()`, startup code phải bảo đảm vùng `.bss` được điền `0`.

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

`.bss` không cần một bản dữ liệu khởi tạo trong Flash như `.data`; startup code chỉ cần zero-initialize vùng này.

---

### 1.8.13. Heap và Stack trong SRAM

Cả hai vùng sau đều nằm trong SRAM:

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

Nội dung giải thích:

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



```text
const
→ không thay đổi
→ CPU chỉ cần đọc
```

Nếu vẫn đặt một bản `const` trong SRAM thì có hai chi phí:

```text
1. Tốn RAM
2. Tốn thời gian khởi tạo vì phải copy khi reset
```

Do đó:

```text
.rodata
→ Flash
```

Có thể nhớ:

> **Dữ liệu chỉ đọc phù hợp với Flash; dữ liệu cần thay đổi khi chạy phù hợp với SRAM.**

---

### 1.8.16. Quá trình từ Reset tới dữ liệu sẵn sàng trong SRAM

Kết hợp với Reset Sequence:

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
→ vẫn ở Flash

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

Bố cục STM32 có thể biểu diễn cụ thể hơn:

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

| Section / vùng | Vị trí | Nội dung |
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

### 1.8.20. Điểm cần nhớ

**Câu hỏi:** “Flash và SRAM khác nhau như thế nào?”

> **Flash là bộ nhớ non-volatile dùng để giữ firmware và dữ liệu chỉ đọc; SRAM là bộ nhớ volatile dùng cho dữ liệu đọc/ghi trong lúc chương trình chạy như `.data`, `.bss`, Heap và Stack.**

**Câu hỏi:** “Tại sao `.data` xuất hiện cả ở Flash và SRAM?”

> **Vì biến `.data` cần có giá trị khởi tạo tồn tại trong firmware nhưng đồng thời phải thay đổi được khi chạy. Giá trị ban đầu được lưu trong Flash, sau reset startup code copy nó sang SRAM để chương trình sử dụng.**

**Câu hỏi:** “`.bss` chứa gì?”

> ** `.bss` chứa các biến global và static chưa khởi tạo và nằm trong SRAM. Reset Handler thực hiện bước khởi tạo `.bss` trước khi vào `main()`.**

**Câu hỏi:** “Tại sao `const` thường được giữ trong Flash?”

> **Vì dữ liệu `const` không thay đổi và CPU chỉ cần đọc.  đưa chúng vào SRAM sẽ tốn RAM và còn tốn thời gian copy lúc startup.**

**Câu hỏi:** “Stack và Heap nằm ở đâu?”

> **Theo sơ đồ, cả Stack và Heap đều nằm trong SRAM.**

**Câu hỏi:** “`_sdata` và `_edata` dùng để làm gì?”

> **Đây là các boundary xác định vùng `.data` trong SRAM để startup code biết phạm vi cần khởi tạo/copy; cách linker tạo các symbol này được trình bày ở phần Linker Script.**

---

### 1.8.21. Câu hỏi tự kiểm tra

1. Flash là volatile hay non-volatile?
2. SRAM là volatile hay non-volatile?
3. Firmware thường được lưu ở đâu ?
4. Dữ liệu chỉ đọc thường nằm ở section nào?
5. `.text` nằm ở Flash hay SRAM?
6. `.data` chứa loại biến nào?
7. Tại sao `.data` có bản trong cả Flash và SRAM?
8. Khi startup, `.data` được chuyển theo hướng nào?
9. `.bss` chứa loại biến nào?
10. `.bss` nằm ở Flash hay SRAM?
11. Stack nằm ở đâu?
12. Heap nằm ở đâu?
13. Tại sao dữ liệu `const` không cần một bản SRAM theo giải thích của nội dung?
14. Vì sao biến cần thay đổi trong lúc chạy phù hợp với SRAM?
15. Vector Table nằm ở đâu trong sơ đồ?
16. `_sdata` và `_edata` đại diện cho điều gì ở mức khái niệm?
17. Phần Reset Handler liên quan thế nào tới `.data` và `.bss`?
18. Hãy mô tả luồng `Flash .data → SRAM .data → main()`.
19. Hãy phân biệt `.text`, `.rodata`, `.data` và `.bss`.
20. Flash base và SRAM base lần lượt là gì?

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

**Stack Memory** là một phần của bộ nhớ chính, thường nằm trong RAM, dùng để lưu dữ liệu tạm thời trong lúc chương trình chạy.

Stack có thể nằm trong:

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

Stack thường được dùng để lưu tạm:

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

SRAM có thể được tổ chức khái niệm thành các vùng:

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

Có bốn mô hình Stack:

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



```text
Full Descending
```

có đặc điểm:

- SP trỏ tới ô đã chứa dữ liệu cuối cùng.
- Khi `PUSH`, SP giảm.
- Khi `POP`, lấy dữ liệu tại SP rồi SP tăng.
- Stack mở rộng theo hướng địa chỉ giảm.

Nội dung ghi:

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

Ví dụ chuỗi thao tác:

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

Stack có thể được bố trí ở những vị trí khác nhau trong RAM tùy thiết kế/linker script.

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

Một symbol thường dùng là:

```text
_estack
```

và mô tả nó là linker symbol dùng để chỉ **cuối RAM làm điểm bắt đầu của Stack** trong cách bố trí đó.

Điểm cần nhớ:

> **Vị trí Stack không phải tự nhiên xuất hiện; linker/startup code phải biết biên Stack hợp lệ.**

---

### 1.9.13. Vì sao Stack thường bắt đầu ở địa chỉ cao?

Do Cortex-M sử dụng Full Descending Stack:

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

Các **Banked Stack Pointer** gồm:

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

#### Cách hiểu thống nhất

Cortex-M có hai Stack Pointer banked:

```text
MSP
→ Main Stack Pointer

PSP
→ Process Stack Pointer
```

`SP / R13` biểu diễn Stack Pointer đang được chọn tại thời điểm thực thi:

```text
MSP và PSP
→ hai banked Stack Pointer

SP / R13
→ Stack Pointer hiện hành
```

---

### 1.9.15. MSP — Main Stack Pointer



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



```text
Handler mode
→ luôn dùng MSP
```

Điểm cần nhớ:

> **MSP là Stack Pointer mặc định sau reset và là Stack Pointer được Handler mode sử dụng.**

---

### 1.9.16. PSP — Process Stack Pointer



```text
PSP
→ Stack Pointer thay thế
→ có thể được Thread mode sử dụng
```

Thread mode có thể chọn PSP bằng bit:

```text
CONTROL.SPSEL
```



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



- Thread mode có thể đổi current SP sang PSP bằng `CONTROL.SPSEL`.
- Handler mode luôn dùng MSP.
- Thay đổi `SPSEL` trong Handler mode không có ý nghĩa; thao tác ghi bị bỏ qua.

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

Hai lợi ích chính:

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

Sự tách biệt này giúp Stack của ứng dụng độc lập với Stack dành cho ngắt/exception.

#### 2. Hỗ trợ RTOS / đa nhiệm



```text
Task A → PSP riêng
Task B → PSP riêng
Task C → PSP riêng

ISR → MSP
```

Khi chuyển task, kernel có thể thay đổi PSP để chuyển sang Stack của task khác.

Ý chính:

> **PSP giúp tách Stack của từng task/application khỏi Stack hệ thống dùng cho exception.**

FreeRTOS chi tiết sẽ được học sau nếu cần.

---

### 1.9.19. Thay đổi Stack Pointer

Trong Assembly có thể truy cập MSP và PSP bằng các instruction:

```text
MRS
MSR
```

và trong C có thể cần mã mức thấp/naked function nếu muốn trực tiếp thay đổi Stack Pointer.

Không cần ghi nhớ cú pháp Assembly cụ thể; cần hiểu vai trò của các thanh ghi và luồng chuyển Stack.

Điểm cần nhớ:

```text
MSP / PSP
→ là các thanh ghi đặc biệt
→ việc đổi current Stack Pointer là thao tác mức thấp
```

---

### 1.9.20. AAPCS là gì?

Một khái niệm liên quan là:

```text
AAPCS
= Procedure Call Standard for the Arm Architecture
```

Đây là chuẩn quy định **cách các hàm gọi nhau trên ARM**.

Có thể xem đây là một “hợp đồng” giữa:

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

### 1.9.22. Truyền tham số theo AAPCS

Quy tắc cơ bản:

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

#### Lưu ý về quy ước thanh ghi

Theo AAPCS, bốn tham số đầu tiên được truyền qua:

```text
R0
R1
R2
R3
```

Các tham số vượt quá khả năng truyền qua nhóm thanh ghi này có thể được truyền qua Stack.

---

### 1.9.23. Caller-saved Registers

Theo AAPCS:

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

Callee có thể sử dụng các register này mà không phải khôi phục lại đúng giá trị cũ, trừ khi caller cần bảo toàn chúng.

---

### 1.9.24. Callee-saved Registers



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
→ caller-saved

R4-R11
→ callee-saved
```


---

#### Lưu ý về AAPCS và căn chỉnh Stack

Cách nhớ đơn giản:

```text
R0-R3, R12, LR
→ caller-saved

R4-R11
→ callee-saved
```

Khi đối chiếu AAPCS32 chính thức, cần thêm hai lưu ý:

1. **`r9` có vai trò phụ thuộc nền tảng.** AAPCS32 yêu cầu callee bảo toàn `r4-r8`, `r10`, `r11` và `SP`; `r9` được bảo toàn trong các PCS variant coi `r9` là một biến callee-saved. Vì vậy câu “R4-R11 luôn callee-saved” chỉ nên dùng như cách nhớ đơn giản.
2. **Stack phải được căn chỉnh.** AAPCS yêu cầu `SP` luôn ít nhất word-aligned; tại public interface, Stack phải **8-byte aligned** (`SP mod 8 = 0`).

Cách diễn đạt ngắn gọn:

> **R0-R3 và R12 là các scratch/argument register nên caller không được kỳ vọng chúng được giữ nguyên qua một lời gọi hàm. R4-R11 thường được dùng làm callee-saved, nhưng r9 có thể có vai trò platform-specific; Stack phải tuân thủ alignment của ABI.**

Với `LR`:

```text
BL / lời gọi hàm
→ ghi return address vào LR
```

nên một hàm **không phải leaf function** thường phải bảo toàn `LR` trước khi gọi tiếp hàm khác nếu còn cần địa chỉ quay về cũ.

---

### 1.9.25. Giá trị trả về theo AAPCS

Kết quả trả về bắt đầu qua:

```text
R0
```

và `R0`/`R1` có thể được dùng để gửi result về caller tùy kiểu/kích thước giá trị trả về.

Ở mức này nên nhớ:

```text
Giá trị trả về
→ bắt đầu từ R0
```

Việc chính xác cần bao nhiêu register phụ thuộc kiểu dữ liệu/kích thước kết quả và sẽ không đi sâu trong mục Stack cơ bản.

---

### 1.9.26. Luồng gọi hàm theo AAPCS

Có thể tóm tắt:

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

**Stack activities during interrupt and exception** diễn ra như sau.

Khi exception xảy ra, processor tự động lưu một nhóm register để tạo:

```text
Stack Frame
```

Các thanh ghi được lưu gồm:

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

Việc tự động lưu context cho phép một hàm C thông thường được dùng làm exception/interrupt handler mà không phải tự lưu toàn bộ caller-saved register ngay từ đầu.

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

Một basic Stack Frame không có FPU chứa tám word:

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

Với Full Descending Stack:

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

Trường hợp minh họa:

```text
Stack Frame (No FPU)
```

nên cấu trúc này đang nói tới trường hợp minh họa **không có FPU context**.

---

### 1.9.30. Un-stacking khi thoát Exception

Khi handler kết thúc, processor tự động:

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

Stack Frame có thể giúp xác định:

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

Nguyên tắc khởi tạo Stack:

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

Sau khi vào `main()`, chương trình có thể cấu hình lại Stack Pointer nếu thiết kế cần.

---

### 1.9.33. Các lưu ý khi thiết kế Stack

Các lưu ý khi khởi tạo Stack:

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

### 1.9.35. Điểm cần nhớ

**Câu hỏi:** “Stack là gì?”

> **Stack là vùng RAM dùng để lưu dữ liệu tạm thời theo nguyên tắc LIFO, như biến cục bộ, register cần bảo toàn, return information và context khi xảy ra exception/interrupt.**

**Câu hỏi:** “Cortex-M dùng Stack model nào?”

> ** Cortex-M sử dụng Full Descending Stack: SP trỏ vào phần tử đang ở đỉnh Stack và Stack phát triển về địa chỉ thấp hơn.**

**Câu hỏi:** “MSP và PSP khác nhau thế nào?”

> **MSP là Main Stack Pointer, được chọn mặc định sau reset và Handler mode luôn dùng MSP. PSP là Process Stack Pointer mà Thread mode có thể chọn sử dụng, thường hữu ích để tách Stack ứng dụng/task khỏi Stack hệ thống.**

**Câu hỏi:** “MSP được khởi tạo khi nào?”

> **Sau reset, processor tự động lấy initial MSP từ entry đầu tại địa chỉ `0x00000000` .**

**Câu hỏi:** “AAPCS liên quan Stack như thế nào?”

> **AAPCS quy định cách caller và callee sử dụng register và Stack khi gọi hàm, ví dụ bốn tham số đầu đi qua R0-R3, các tham số dư có thể đặt trên Stack và R4-R11 là nhóm callee phải bảo toàn nếu sử dụng .**

**Câu hỏi:** “Exception entry lưu những register nào?”

> ** hardware tự động stacking R0-R3, R12, LR, PC và xPSR để tạo Stack Frame trước khi handler chạy.**

**Câu hỏi:** “Stack Frame khi exception có ích gì cho debug?”

> **Nó giữ PC, LR và các register tại thời điểm exception, nên debugger có thể dùng chúng để phân tích vị trí và trạng thái chương trình khi fault xảy ra.**

---

### 1.9.36. Câu hỏi tự kiểm tra

1. Stack Memory là gì?
2. Stack thường nằm trong loại bộ nhớ nào?
3. LIFO nghĩa là gì?
4. Stack thường lưu những loại dữ liệu nào?
5. SP là register nào?
6. Cortex-M sử dụng Stack model nào ?
7. Full và Empty khác nhau ở điểm nào?
8. Ascending và Descending khác nhau ở điểm nào?
9. Khi PUSH trên Full Descending Stack thì SP tăng hay giảm?
10. Khi POP trên Full Descending Stack thì SP tăng hay giảm?
11. `_estack` được sơ đồ dùng để biểu diễn gì?
12. MSP viết tắt của gì?
13. PSP viết tắt của gì?
14. Stack Pointer mặc định sau reset là MSP hay PSP?
15. Thread mode có thể dùng những Stack Pointer nào?
16. Handler mode dùng Stack Pointer nào?
17. Bit nào trong `CONTROL` được nội dung nhắc tới để chọn PSP?
18. Vì sao phải khởi tạo PSP tới địa chỉ Stack hợp lệ trước khi dùng?
19. Dùng PSP cho Thread mode có lợi gì?
20. RTOS có thể sử dụng PSP như thế nào ?
21. AAPCS là gì?
22. Caller và Callee là gì?
23. Bốn tham số đầu của hàm được truyền qua những register nào theo quy tắc?
24. Nếu có nhiều hơn bốn tham số thì phần dư có thể được đặt ở đâu?
25. Những register nào thuộc nhóm caller-saved?
26. Những register nào được gọi là callee-saved?
27. Khi callee sử dụng R4-R11 thì phải làm gì?
28. Khi exception xảy ra, hardware tự động lưu những register nào?
29. Một basic exception Stack Frame không FPU có bao nhiêu word?
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
→ caller-saved

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

### 1.10.1. Vai trò của Startup Code

Một yêu cầu quan trọng của Startup Code là:

> **Tại sao không được đặt Vector Table vào vùng `.data` đã khởi tạo trong RAM?**

Startup Code hoàn chỉnh thường bao gồm Vector Table, `Reset_Handler`, khởi tạo dữ liệu runtime và các handler mặc định.

Luồng khởi động cốt lõi:

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

Các khái niệm weak handler, danh sách vector, cú pháp Assembly và triển khai cụ thể theo vendor được tách thành các tiểu mục riêng để giữ luồng khởi động rõ ràng.

---

### 1.10.2. Startup Code nằm ở đâu trong luồng khởi động?

Liên hệ với các phần trước:

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



```text
STM32F1:
Flash được map ở 0x08000000
```

và mô tả CPU đọc Vector Table tại địa chỉ cố định trong quá trình reset.

Mối quan hệ cần nhớ:

```text
Flash
→ chứa Vector Table

Vector Table
→ phải sẵn sàng ngay khi reset

CPU
→ đọc nó trước khi chạy code
```

Cơ chế remap/boot alias đã được trình bày ở phần Reset Sequence.

---

### 1.10.5. `.data` trong SRAM tồn tại khi nào?

Điểm quan trọng:

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

Trong Startup Code, `Reset_Handler` thực hiện việc copy `.data` từ Flash xuống RAM.

> **Reset_Handler là phần thực hiện việc copy `.data` từ Flash xuống RAM.**

---

### 1.10.7. Tại sao không thể đặt Vector Table vào `.data`?

Đây là một ràng buộc quan trọng của quá trình khởi động.

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

Kết quả:

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

Vì vậy Vector Table không được phụ thuộc vào vùng `.data` runtime trong RAM.

---

### 1.10.9. Thứ tự đúng

Thứ tự đúng:

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

Phần **1.10 Startup Code** làm rõ:

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

Trong luồng startup:

```text
Reset_Handler
→ copy .data từ Flash xuống SRAM
```

---

### 1.10.12. Vì sao Vector Table phải có sẵn trước code?



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

Đúng:

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

### 1.10.15. Điểm cần nhớ

**Câu hỏi:** “Tại sao Vector Table không được đặt trong `.data` ở RAM?”

> **Vì CPU cần đọc initial Stack Pointer và địa chỉ Reset_Handler từ Vector Table ngay khi reset, trước khi bất kỳ code nào chạy. Trong khi `.data` trong SRAM chỉ được Reset_Handler copy từ Flash xuống sau đó. Nếu Vector Table phụ thuộc vào `.data` trong RAM thì CPU chưa thể lấy SP và Reset_Handler để boot.**

**Câu hỏi:** “`.data` được chuẩn bị khi nào?”

> ** `.data` trong RAM được Reset_Handler copy từ Flash xuống sau khi CPU đã đọc Vector Table và nhảy vào Reset_Handler.**

**Câu hỏi:** “CPU dùng gì từ Vector Table ngay sau reset?”

> **Hai word đầu: initial Stack Pointer và địa chỉ Reset_Handler.**

**Câu hỏi:** “Startup Code có liên hệ thế nào với Flash và SRAM?”

> **Trong nội dung phần này, Reset_Handler thuộc luồng startup và thực hiện việc copy `.data` từ Flash xuống SRAM để tạo bản dữ liệu runtime.**

---

### 1.10.16. Câu hỏi tự kiểm tra

1. Startup Code trong phần này hiện tại tập trung vào vấn đề gì?
2. CPU đọc bao nhiêu word đầu của Vector Table ngay khi reset?
3. Word đầu tiên của Vector Table dùng để làm gì?
4. Word thứ hai dùng để làm gì?
5. Việc đọc Vector Table xảy ra trước hay sau khi code bắt đầu chạy?
6. `.data` trong SRAM được tạo/copy khi nào?
7. Ai thực hiện việc copy `.data` từ Flash xuống SRAM?
8. Vì sao Vector Table không thể phụ thuộc vào `.data` trong SRAM?
9. Điều gì xảy ra nếu CPU không lấy được initial SP và Reset_Handler?
10. Hãy mô tả vòng phụ thuộc sai nếu Vector Table nằm trong `.data`.
11. Hãy mô tả thứ tự đúng từ Reset tới lúc `.data` sẵn sàng trong SRAM.
12. Startup Code liên hệ thế nào với Reset Sequence?
13. Startup Code liên hệ thế nào với Flash/SRAM?
14. Tại sao Vector Table phải tồn tại trước khi Reset_Handler chạy?
15. Hãy trả lời: “Tại sao không đặt Vector Table vào `.data`?”

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


---

### 1.10.18. Startup Code hoàn chỉnh

> Startup Code liên kết trực tiếp Reset Sequence, Vector Table, Flash/SRAM và Linker Script thành một chuỗi khởi động hoàn chỉnh.

Một startup file STM32 điển hình thường có các thành phần khái niệm sau:

```text
Startup file
├── Vector Table
├── Reset_Handler
├── Default_Handler
└── các handler yếu (weak) / alias mặc định
```

#### Vector Table

Vector Table chứa:

```text
Entry 0 → Initial MSP
Entry 1 → Reset_Handler
Entry 2... → exception / interrupt handlers
```

Linker thường giữ Vector Table trong một section riêng như:

```text
.isr_vector
```

và đặt section này ở đầu vùng boot/Flash thích hợp.

#### `Reset_Handler`

Sau khi phần cứng đã lấy initial MSP và Reset vector, `Reset_Handler` thực hiện phần **khởi tạo phần mềm**.

Luồng điển hình:

```text
RESET
  ↓
Hardware lấy MSP + Reset vector
  ↓
Reset_Handler
  ↓
copy .data: Flash → SRAM
  ↓
zero .bss
  ↓
SystemInit() / khởi tạo hệ thống cần thiết
  ↓
khởi tạo C/C++ runtime nếu toolchain yêu cầu
  ↓
main()
```

> Thứ tự chính xác của `SystemInit()` so với một số bước runtime có thể khác giữa startup file/toolchain/vendor. Điều cần nắm là **`.data` phải có giá trị runtime đúng, `.bss` phải được zero-initialize và môi trường cần thiết phải sẵn sàng trước khi mã ứng dụng phụ thuộc vào chúng**.

Pseudocode khái niệm:

```c
extern unsigned int _sidata;
extern unsigned int _sdata;
extern unsigned int _edata;
extern unsigned int _sbss;
extern unsigned int _ebss;

void Reset_Handler(void)
{
    // 1. Copy .data từ Flash sang SRAM
    unsigned int *src = &_sidata;
    unsigned int *dst = &_sdata;
    while (dst < &_edata) {
        *dst++ = *src++;
    }

    // 2. Zero .bss
    for (dst = &_sbss; dst < &_ebss; ++dst) {
        *dst = 0;
    }

    // 3. Các bước khởi tạo hệ thống/runtime tùy project
    SystemInit();
    __libc_init_array();

    // 4. Vào ứng dụng
    main();

    while (1) {
        // main() thường không được kỳ vọng return trong bare-metal firmware
    }
}
```

Đây chỉ là pseudocode để hiểu luồng; tên symbol và thứ tự cụ thể phụ thuộc linker script/startup file thật của project.

#### `Default_Handler` và weak handler

Startup file của vendor thường cung cấp handler mặc định:

```text
IRQ_Handler cụ thể chưa được user định nghĩa
        ↓
weak alias
        ↓
Default_Handler
        ↓
thường lặp vô hạn để debugger có thể dừng lại
```

Khi người dùng định nghĩa một handler cùng tên mạnh hơn, linker sẽ chọn implementation của người dùng thay cho weak default.

Cách hiểu này giúp giải thích tại sao bạn chỉ cần viết, ví dụ:

```c
void USART2_IRQHandler(void)
{
    // xử lý ngắt
}
```

mà không phải sửa trực tiếp startup file trong nhiều project.

#### Startup Code phụ thuộc Linker Script như thế nào?

Startup code không tự biết `.data` và `.bss` nằm ở đâu. Nó sử dụng các symbol do linker/linker script tạo ra:

```text
_sidata → địa chỉ chứa initial values của .data trong Flash
_sdata  → đầu .data trong SRAM
_edata  → cuối .data trong SRAM
_sbss   → đầu .bss
_ebss   → cuối .bss
_estack → initial Stack boundary / top of stack theo project
```

Do đó:

```text
Linker Script
→ tạo layout + boundary symbols
        ↓
Startup Code
→ dùng symbols để khởi tạo RAM
        ↓
main()
```

### 1.10.19. Cách diễn đạt ngắn gọn

**Câu hỏi:** “Startup code làm gì trước `main()`?”

> **Sau khi hardware lấy MSP và Reset vector, `Reset_Handler` chuẩn bị môi trường runtime: copy `.data` từ Flash sang RAM, zero `.bss`, thực hiện các bước system/C runtime initialization cần thiết rồi gọi `main()`. Startup file cũng thường chứa Vector Table và các handler mặc định.**

[↑ Về mục lục](#muc-luc)


---

<a id="muc-01-11"></a>
## 1.11. Linker Script và các section

### 1.11.1. Từ file `.c` đến object file `.o`



```text
main.c
  ↓
main.o

led.c
  ↓
led.o
```

Mỗi file nguồn sau khi biên dịch tạo ra một **object file** riêng.

Ví dụ:

```text
main.c → main.o
led.c  → led.o
```

Mỗi object file có thể chứa nhiều section khác nhau, ví dụ:

```text
.text
.data
.bss
.rodata
```

Có thể hình dung:

```text
main.o
├── .text
├── .data
├── .bss
└── .rodata

led.o
├── .text
├── .data
├── .bss
└── .rodata
```

---

### 1.11.2. Section là gì?

Một object file ở định dạng ELF có thể chứa nhiều loại section:

```text
.text
.data
.bss
.rodata
User defined sections
Some special sections
```

Mỗi section chứa một loại nội dung khác nhau.

Sơ đồ:

```text
main.o
│
├── .text
├── .data
├── .bss
├── .rodata
├── User defined sections
└── Some special sections
```

---

### 1.11.3. `.text`



```text
.text
→ chứa code / instructions
```

Có thể nhớ:

```text
.text
→ mã máy của chương trình
```

Ví dụ:

```c
void led_on(void)
{
    // code
}
```

sau khi biên dịch, phần instruction tương ứng sẽ được đặt trong section `.text` của object file.

---

### 1.11.4. `.data`



```text
.data
→ chứa initialized data
```

Có thể hiểu:

```text
.data
→ dữ liệu đã có giá trị khởi tạo
```

Ví dụ khái niệm:

```c
int counter = 10;
```

Phần trước về Flash/SRAM đã cho thấy:

```text
.data initial values
→ lưu trong Flash

.data runtime
→ được copy sang SRAM khi startup
```

Ở mục Linker Script này, trọng tâm là:

> **Object file nào cũng có thể đóng góp một phần `.data`, và linker sẽ ghép các phần đó lại.**

---

### 1.11.5. `.bss`



```text
.bss
→ contains data which are uninitialized
```

Tức là:

```text
.bss
→ dữ liệu chưa khởi tạo
```

Ví dụ:

```c
int counter;
static int state;
```

Khi build nhiều object file:

```text
main.o  → có thể có .bss
led.o   → có thể có .bss
```

và linker sẽ ghép các phần `.bss` phù hợp vào section `.bss` cuối cùng.

---

### 1.11.6. `.rodata`



```text
.rodata
→ contains read-only data
```

Có thể nhớ:

```text
.rodata
→ dữ liệu chỉ đọc
```

Phần Flash/SRAM trước đó đã liên hệ `.rodata` với Flash vì dữ liệu này không cần thay đổi trong thời gian chạy.

---

### 1.11.7. User-defined sections

Ngoài các section chuẩn còn có:

```text
User defined sections
```

và mô tả:

> chứa data/code mà lập trình viên yêu cầu đưa vào section do người dùng tự định nghĩa.

Điều này có nghĩa về mặt khái niệm:

```text
Programmer
   ↓
yêu cầu một số code/data
được đặt vào section riêng
   ↓
User-defined section
```

Cú pháp khai báo user-defined section phụ thuộc compiler/linker; phần này tập trung vào cách linker script gom và đặt các section trong bộ nhớ.

---

### 1.11.8. Special sections

Ngoài ra còn có:

```text
Some special sections
```

với mô tả:

> compiler có thể thêm một số section đặc biệt chứa dữ liệu đặc biệt.

Ngoài các section quen thuộc, ELF còn có thể chứa các special section phục vụ compiler, linker và runtime:

```text
ELF object
→ ngoài các section quen thuộc
  còn có thể chứa special sections
```

---

### 1.11.9. Linker làm gì?



> Linker dùng để merge các section cùng loại của nhiều object file và resolve các undefined symbol.

Có thể hình dung:

```text
main.o
├── .text
├── .data
├── .bss
└── .rodata

led.o
├── .text
├── .data
├── .bss
└── .rodata

        ↓ linker

final.elf
├── .text
├── .data
├── .bss
└── .rodata
```

Hai nhiệm vụ chính:

```text
1. Merge similar sections
2. Resolve undefined symbols
```

---

### 1.11.10. Merge các section giống nhau



```text
.text
← .text(main.o)
← .text(led.o)

.data
← .data(main.o)
← .data(led.o)

.bss
← các phần .bss tương ứng

.rodata
← .rodata(main.o)
← .rodata(led.o)
```

Có thể hiểu:

```text
Các input section
từ từng object file

        ↓

Linker merge

        ↓

Output section
trong final ELF
```

Ví dụ:

```text
.text(main.o)
+
.text(led.o)
        ↓
     .text
```

---

### 1.11.11. Quy tắc ghép section

Linker gom các input section cùng loại từ nhiều object file vào output section tương ứng.

Ví dụ:

```text
.text(main.o)
+
.text(led.o)
        ↓
      .text
```

Tương tự:

```text
.data(*)   → .data
.bss(*)    → .bss
.rodata(*) → .rodata
```

Quy tắc bố trí cụ thể được điều khiển bởi linker script.

---

### 1.11.12. Resolve undefined symbols là gì?

Linker còn có nhiệm vụ:

```text
resolve all undefined symbols
of different object files
```

Có thể hiểu:

```text
main.o
→ gọi một symbol
→ nhưng chưa có phần định nghĩa trong main.o

led.o
→ chứa định nghĩa symbol đó

linker
→ nối hai phần lại
```

Ví dụ:

```c
// main.c
void led_on(void);

int main(void)
{
    led_on();
}
```

```c
// led.c
void led_on(void)
{
}
```

Sau khi biên dịch:

```text
main.o
→ biết có symbol led_on
→ chưa có body trong main.o

led.o
→ chứa định nghĩa led_on
```

Linker:

```text
main.o + led.o
        ↓
resolve led_on
```

---

### 1.11.13. Locator làm gì?

Thuật ngữ sử dụng là:

```text
Linker and Locator
```

và mô tả:

> Locator là một phần của linker và dùng linker script để hiểu bạn muốn merge các section như thế nào và gán địa chỉ nào cho từng section.

Có thể tóm tắt:

```text
Linker
→ ghép section
→ resolve symbol

Locator
→ dựa vào linker script
→ gán địa chỉ cho section
```

---

### 1.11.14. Linker Script là gì?



```text
Linker Script
→ mô tả cách merge các section
→ mô tả địa chỉ được gán cho các section
```

Có thể hiểu:

> **Linker Script là tệp cấu hình cho linker biết các section cuối cùng phải được bố trí ở đâu trong không gian bộ nhớ.**

Sơ đồ:

```text
Object files
   ↓
Linker
   ↓
Linker Script
   ↓
Merge sections
+
Address relocation
   ↓
final.elf
```

---

### 1.11.15. Address Relocation



```text
Merging and address relocation
```

Điều này cho thấy sau khi ghép section, linker/locator còn phải xử lý địa chỉ.

Có thể hiểu:

```text
.text
.data
.bss
.rodata
```

không chỉ cần được ghép lại mà còn phải được đặt vào địa chỉ thích hợp trong chương trình cuối.

Ví dụ khái niệm:

```text
.text
→ vùng code

.data / .bss
→ vùng data

.rodata
→ vùng read-only
```

Việc section cụ thể nằm ở Flash hay SRAM và tại địa chỉ nào được quyết định bởi cách bố trí trong linker script.

---

### 1.11.16. final ELF

Kết quả là:

```text
final.elf
```

Đây là executable sau khi linker/locator thực hiện:

```text
merge
+
symbol resolution
+
address relocation
```

Sơ đồ tổng thể:

```text
main.c
  ↓
main.o
  ┐
  │
led.c
  ↓
led.o
  │
  ┘
   ↓
Linker + Locator
   ↓
Linker Script
   ↓
final.elf
```

---

### 1.11.17. Liên hệ với Build Process

Có thể nối mục này với kiến thức C trước đó:

```text
Source
  ↓ Compile
Object files
  ↓ Link
Executable / ELF
```

Ở đây phần Link được mở rộng thành:

```text
Object files
   ↓
merge sections
   ↓
resolve symbols
   ↓
assign addresses
   ↓
final ELF
```

---

### 1.11.18. Liên hệ với Flash và SRAM

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

Linker Script là nơi giúp mô tả cách các section cuối cùng được bố trí vào các vùng bộ nhớ đó.

Có thể hình dung:

```text
.text
.rodata
        ↓
Linker Script
        ↓
Flash region

.data
.bss
        ↓
Linker Script
        ↓
SRAM region
```

Riêng `.data` có liên hệ kép:

```text
Load image
→ giá trị khởi tạo nằm trong Flash

Runtime
→ được copy sang SRAM
```

Chi tiết này đã được triển khai ở mục Flash/SRAM.

---

### 1.11.19. Liên hệ với Startup Code

Startup Code cần biết các boundary của section để thực hiện công việc như:

```text
copy .data
Flash → SRAM
```

và:

```text
initialize .bss
```

Các symbol biên thường gặp:

```text
_etext
_sdata
_edata
```

Linker Script quyết định vị trí section và tạo các symbol biên phục vụ startup code:

```text
Linker Script
→ quyết định vị trí section
→ từ đó linker có thể tạo các boundary/symbol liên quan
```

Một linker script rút gọn ở cuối mục cho thấy cách tạo `_sdata`, `_edata` và cách startup code sử dụng các symbol này.

---

### 1.11.20. Sơ đồ tổng hợp

```text
main.c                    led.c
  │                         │
  ↓                         ↓
main.o                    led.o
  │                         │
  ├── .text                 ├── .text
  ├── .data                 ├── .data
  ├── .bss                  ├── .bss
  └── .rodata               └── .rodata
        \                   /
         \                 /
          \               /
           ↓             ↓
              Linker
                │
                ├── merge similar sections
                ├── resolve symbols
                │
                ↓
             Locator
                │
                └── dùng Linker Script
                    để gán địa chỉ
                ↓
             final.elf
                │
                ├── .text
                ├── .data
                ├── .bss
                └── .rodata
```

---

### 1.11.21. Điểm cần nhớ

**Câu hỏi:** “Linker làm gì?”

> **Linker ghép các section cùng loại từ nhiều object file, resolve các undefined symbol và tạo chương trình cuối. Linker script quyết định cách merge và địa chỉ của các section.**

**Câu hỏi:** “Linker Script dùng để làm gì?”

> **Linker Script mô tả cách bố trí các section và gán địa chỉ cho chúng trong bộ nhớ. Nó giúp linker/locator biết `.text`, `.data`, `.bss`, `.rodata` và các section khác phải được đặt ở đâu.**

**Câu hỏi:** “Mỗi file `.o` có những section nào?”

> **Một object file ELF có thể có `.text`, `.data`, `.bss`, `.rodata`, cùng user-defined sections và special sections.**

**Câu hỏi:** “`.text`, `.data`, `.bss`, `.rodata` chứa gì?”

> ** `.text` chứa code/instruction, `.data` chứa initialized data, `.bss` chứa uninitialized data, còn `.rodata` chứa read-only data.**

**Câu hỏi:** “final ELF được tạo như thế nào?”

> **Các object file được đưa vào linker; linker merge các section cùng loại, resolve symbol, locator dùng linker script để gán địa chỉ và kết quả là final ELF.**

---

### 1.11.22. Câu hỏi tự kiểm tra

1. File `.c` sau khi biên dịch tạo ra file gì?
2. Một object file ELF có thể chứa những section nào?
3. `.text` chứa gì?
4. `.data` chứa gì?
5. `.bss` chứa gì?
6. `.rodata` chứa gì?
7. User-defined section dùng để làm gì?
8. Special section là gì ở mức khái niệm?
9. Linker có nhiệm vụ chính gì?
10. Merge similar sections nghĩa là gì?
11. Undefined symbol là gì ở mức khái niệm?
12. Linker resolve symbol giữa các object file như thế nào?
13. Locator là gì ?
14. Locator dùng gì để biết cách bố trí section?
15. Linker Script dùng để làm gì?
16. Address relocation nghĩa là gì?
17. final ELF là gì?
18. `.text(main.o)` và `.text(led.o)` cuối cùng được xử lý thế nào?
19. Linker Script liên hệ thế nào với Flash và SRAM?
20. Linker Script liên hệ thế nào với Startup Code?
21. Vì sao boundary symbol như `_sdata`, `_edata` cần thông tin từ linker?
22. Hãy mô tả luồng `main.c + led.c → object files → linker → final.elf`.
23. Sơ đồ có điểm không nhất quán nào ở dòng `.bss`?

---

### 1.11.23. Tóm tắt

```text
Source files
main.c
led.c
   ↓
Compile
   ↓
Object files
main.o
led.o
   ↓
mỗi object có:
.text
.data
.bss
.rodata
...
```

Linker:

```text
merge similar sections
+
resolve undefined symbols
```

Locator:

```text
dùng Linker Script
→ quyết định cách bố trí
→ gán địa chỉ
```

Kết quả:

```text
final.elf
├── .text
├── .data
├── .bss
├── .rodata
└── ...
```

Liên hệ bộ nhớ:

```text
Linker Script
→ quyết định section nằm ở đâu
→ Flash / SRAM
```

**Ý quan trọng nhất:**

> **Mỗi object file có các section riêng như `.text`, `.data`, `.bss`, `.rodata`. Linker ghép các section tương ứng và resolve symbol; locator dùng linker script để gán địa chỉ cho các section, từ đó tạo ra file ELF cuối cùng có bố cục bộ nhớ xác định.**


---

### 1.11.24. Linker Script tối thiểu cần đọc được

> Mục tiêu là đọc được một file `.ld` cơ bản và hiểu section nào được đặt vào Flash hoặc RAM; không cần học thuộc toàn bộ cú pháp.

Một ví dụ rút gọn:

```ld
MEMORY
{
    FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 1024K
    RAM   (xrw) : ORIGIN = 0x20000000, LENGTH = 128K
}

_estack = ORIGIN(RAM) + LENGTH(RAM);

SECTIONS
{
    .isr_vector :
    {
        KEEP(*(.isr_vector))
    } > FLASH

    .text :
    {
        *(.text*)
        *(.rodata*)
    } > FLASH

    _sidata = LOADADDR(.data);

    .data :
    {
        _sdata = .;
        *(.data*)
        _edata = .;
    } > RAM AT > FLASH

    .bss (NOLOAD) :
    {
        _sbss = .;
        *(.bss*)
        *(COMMON)
        _ebss = .;
    } > RAM
}
```

> `1024K` Flash và `128K` RAM ở trên chỉ là **ví dụ**. Dung lượng thật phải lấy từ datasheet/linker script của MCU cụ thể.

#### `MEMORY`

```text
MEMORY
→ khai báo các vùng bộ nhớ mà linker được phép sử dụng
```

Ví dụ:

```text
FLASH
→ bắt đầu 0x08000000

RAM
→ bắt đầu 0x20000000
```

`ORIGIN` là địa chỉ đầu, `LENGTH` là kích thước vùng.

#### `SECTIONS`

```text
SECTIONS
→ mô tả input sections từ các .o
→ được gom thành output section nào
→ output section được đặt vào MEMORY region nào
```

Ví dụ:

```text
*(.text*)
*(.rodata*)
      ↓
.text output section
      ↓
FLASH
```

#### `KEEP(*(.isr_vector))`

Khi linker bật loại bỏ section không được tham chiếu, Vector Table có thể không được gọi như một hàm thông thường.

```text
KEEP(...)
→ yêu cầu linker không loại section quan trọng này
```

Vì vậy startup/linker script STM32 thường có dạng tương tự:

```ld
KEEP(*(.isr_vector))
```

#### VMA và LMA — chìa khóa để hiểu `.data`

`.data` có hai địa chỉ cần phân biệt:

```text
VMA — Virtual/Runtime Memory Address
→ nơi code sử dụng .data khi chương trình chạy
→ SRAM

LMA — Load Memory Address
→ nơi initial values của .data được lưu trong firmware image
→ Flash
```

Có thể hình dung:

```text
FLASH                            SRAM
LMA                              VMA
.data initial values  ───────→   .data runtime
                         startup copy
```

Trong linker script:

```ld
.data : { ... } > RAM AT > FLASH
```

có ý nghĩa khái niệm:

```text
.data chạy ở RAM
nhưng dữ liệu khởi tạo của nó được load/lưu trong Flash
```

`LOADADDR(.data)` giúp lấy địa chỉ load của `.data` để tạo symbol như `_sidata` cho startup code.

#### Boundary symbols

Các dòng:

```ld
_sdata = .;
_edata = .;
_sbss  = .;
_ebss  = .;
```

không tạo biến C thông thường. Chúng tạo **linker symbols** đánh dấu địa chỉ.

Startup code có thể tham chiếu chúng để biết:

```text
copy từ đâu
copy tới đâu
zero vùng nào
```

Kết nối toàn bộ kiến thức:

```text
Linker Script
  ↓ tạo
_sidata / _sdata / _edata / _sbss / _ebss / _estack
  ↓ được dùng bởi
Startup Code
  ↓ chuẩn bị
.data / .bss / Stack
  ↓
main()
```

#### ELF, BIN và HEX khác nhau như thế nào?

Sau khi link:

```text
final.elf
→ chứa code/data + section table + symbol + có thể có debug information
```

Công cụ như `objcopy` có thể tạo thêm:

```text
.bin
→ raw binary image

.hex
→ image có địa chỉ theo định dạng Intel HEX
```

Vì vậy khi debug bằng GDB/ST-Link, file ELF rất hữu ích vì còn symbol/debug info; còn khi nạp firmware, tool có thể dùng ELF/HEX/BIN tùy workflow.

### 1.11.25. Cách diễn đạt ngắn gọn

**Câu hỏi:** “Linker Script quan trọng gì trong embedded?”

> **Linker Script mô tả bản đồ bộ nhớ thực của firmware: Flash/RAM bắt đầu ở đâu, section nào nằm ở đâu, Stack boundary ở đâu và các linker symbol nào được startup code sử dụng. Nó là cầu nối giữa object sections và memory map thật của MCU.**

**Câu hỏi:** “Tại sao `.data` vừa ở Flash vừa ở RAM?”

> **Linker đặt runtime address của `.data` ở RAM nhưng lưu initial image của nó trong Flash. Startup code dùng linker symbols để copy initial values từ LMA trong Flash tới VMA trong RAM trước khi vào `main()`.**

[↑ Về mục lục](#muc-luc)


---

<a id="chuong-02"></a>
# 2. RCC + Clock

> Phạm vi của chương này tập trung vào STM32F101xx, STM32F102xx và STM32F103xx thuộc các nhóm low-, medium-, high- và XL-density. STM32F105xx/STM32F107xx connectivity line có clock tree riêng với nhiều PLL hơn, vì vậy không áp dụng trực tiếp toàn bộ công thức cấu hình của phần này.

<a id="muc-02-01"></a>
## 2.1. RCC là gì?

`RCC` là viết tắt của:

```text
Reset and Clock Control
```

RCC chịu trách nhiệm chính cho hai nhóm chức năng:

```text
RCC
├── Clock Control
│   ├── bật/tắt các nguồn clock
│   ├── chọn SYSCLK
│   ├── cấu hình PLL
│   ├── cấu hình AHB/APB prescaler
│   ├── cấp clock cho peripheral
│   └── theo dõi trạng thái clock
│
└── Reset Control
    ├── reset peripheral trên APB1
    ├── reset peripheral trên APB2
    └── reset Backup domain
```

Các thanh ghi quan trọng:

| Thanh ghi | Vai trò chính |
|---|---|
| `RCC_CR` | Bật/tắt HSI, HSE, PLL; đọc các cờ `RDY`; bật CSS |
| `RCC_CFGR` | Chọn SYSCLK, cấu hình PLL, AHB/APB/ADC prescaler, MCO |
| `RCC_CIR` | Cờ/ngắt liên quan tới trạng thái nguồn clock và CSS |
| `RCC_AHBENR` | Bật clock các peripheral trên AHB |
| `RCC_APB2ENR` | Bật clock các peripheral trên APB2 |
| `RCC_APB1ENR` | Bật clock các peripheral trên APB1 |
| `RCC_APB2RSTR` | Reset các peripheral trên APB2 |
| `RCC_APB1RSTR` | Reset các peripheral trên APB1 |
| `RCC_BDCR` | Clock/Reset của Backup domain và RTC |
| `RCC_CSR` | Điều khiển LSI và các cờ reset |

Có thể hình dung:

```text
Clock source
    ↓
   RCC
    ↓
Clock Tree
    ↓
CPU / Bus / Peripheral
```

---

<a id="muc-02-02"></a>
## 2.2. Các nguồn Clock: HSI / HSE / LSI / LSE

STM32F10xxx có bốn nguồn clock thường gặp:

```text
High-speed
├── HSI
└── HSE

Low-speed
├── LSI
└── LSE
```

### HSI — High-Speed Internal

HSI là bộ dao động RC tốc độ cao nằm bên trong MCU.

Đối với STM32F10xxx trong phạm vi chương này:

```text
HSI = 8 MHz
```

HSI có thể được dùng:

```text
HSI
├── trực tiếp làm SYSCLK
└── chia 2 làm đầu vào PLL
```

Ưu điểm:

- Không cần linh kiện dao động ngoài.
- Khởi động nhanh.
- Chi phí phần cứng thấp.

Hạn chế:

- Độ chính xác thấp hơn crystal/resonator ngoài.
- Tần số chịu ảnh hưởng bởi nhiệt độ và điện áp.
- Có thể tinh chỉnh bằng `HSITRIM`.

Các bit quan trọng trong `RCC_CR`:

```text
HSION
→ bật HSI

HSIRDY
→ HSI đã ổn định

HSICAL
→ giá trị hiệu chuẩn

HSITRIM
→ tinh chỉnh HSI
```

Sau system reset:

```text
SYSCLK = HSI
```

Do đó MCU luôn có một nguồn clock nội bộ để bắt đầu chạy trước khi phần mềm cấu hình clock khác.

### HSE — High-Speed External

HSE là nguồn clock tốc độ cao từ bên ngoài.

Có hai cách sử dụng:

```text
HSE
├── Crystal / ceramic resonator
└── External clock bypass
```

Với crystal/resonator:

```text
4 MHz → 16 MHz
```

Với HSE bypass, một tín hiệu clock bên ngoài được đưa trực tiếp vào `OSC_IN`.

Các bit quan trọng trong `RCC_CR`:

```text
HSEON
→ bật HSE

HSERDY
→ HSE đã ổn định

HSEBYP
→ chọn chế độ external clock bypass
```

HSE thường được dùng khi cần clock chính xác hơn HSI.

### LSI — Low-Speed Internal

LSI là bộ dao động RC tốc độ thấp bên trong MCU.

Tần số danh định:

```text
xấp xỉ 40 kHz
```

Khoảng tần số thực tế có thể rộng hơn do đặc tính RC.

Ứng dụng chính:

```text
LSI
├── Independent Watchdog — IWDG
└── có thể cấp RTC / Auto-Wakeup
```

Các bit liên quan nằm trong `RCC_CSR`:

```text
LSION
LSIRDY
```

### LSE — Low-Speed External

LSE là nguồn dao động ngoài tốc độ thấp:

```text
32.768 kHz
```

Ứng dụng chính:

```text
LSE
→ RTC
```

LSE phù hợp cho RTC vì:

- tần số thấp;
- tiêu thụ năng lượng thấp;
- độ chính xác tốt;
- có thể tiếp tục hoạt động trong Backup domain khi nguồn chính bị tắt nếu `VBAT` vẫn còn.

Các bit liên quan nằm trong `RCC_BDCR`:

```text
LSEON
LSERDY
LSEBYP
```

### So sánh nhanh

| Nguồn | Loại | Tần số điển hình | Mục đích chính |
|---|---|---:|---|
| HSI | Internal RC | 8 MHz | SYSCLK, đầu vào PLL |
| HSE | External | 4–16 MHz với crystal | SYSCLK, đầu vào PLL |
| LSI | Internal RC | ~40 kHz | IWDG, RTC/AWU |
| LSE | External crystal | 32.768 kHz | RTC |

---

<a id="muc-02-03"></a>
## 2.3. Clock Tree

Clock Tree mô tả đường đi của clock từ nguồn dao động tới CPU, bus và peripheral.

Sơ đồ rút gọn:

```text
              HSI 8 MHz
                 │
                 ├───────────────┐
                 │               │
                 │            HSI / 2
                 │               │
                 │               ↓
                 │              PLL
                 │               │
HSE ─────────────┼───────────────┤
                 │               │
                 └──────┬────────┘
                        ↓
                      SYSCLK
                        │
                  AHB Prescaler
                        │
                        ↓
                       HCLK
                        │
           ┌────────────┴────────────┐
           ↓                         ↓
    APB1 Prescaler             APB2 Prescaler
           ↓                         ↓
         PCLK1                     PCLK2
           │                         │
           ↓                         ↓
    APB1 Peripheral            APB2 Peripheral
```

Đường clock chính:

```text
Clock source
    ↓
SYSCLK
    ↓
AHB Prescaler
    ↓
HCLK
    ↓
APB1 / APB2 Prescaler
    ↓
PCLK1 / PCLK2
    ↓
Peripheral
```

Ngoài ra còn có các nhánh riêng:

```text
PCLK2
  ↓
ADC Prescaler
  ↓
ADCCLK

PCLK1 / PCLK2
  ↓
Timer clock logic
  ↓
TIMxCLK

PLLCLK
  ↓
USB Prescaler
  ↓
USBCLK

HCLK
  ├── Core / AHB / Memory / DMA
  └── HCLK / 8 → một lựa chọn clock cho SysTick
```

Clock Tree cần được đọc theo ba câu hỏi:

```text
1. Nguồn clock ban đầu là gì?
2. Clock đã đi qua bộ nhân/chia nào?
3. Peripheral cuối cùng nhận tần số bao nhiêu?
```

---

<a id="muc-02-04"></a>
## 2.4. PLL

`PLL` là viết tắt của:

```text
Phase-Locked Loop
```

PLL được dùng để nhân tần số clock đầu vào.

Đối với STM32F10xxx trong phạm vi chương này, đầu vào PLL có thể là:

```text
HSI / 2
hoặc
HSE
hoặc
HSE / 2
```

Sơ đồ:

```text
HSI / 2 ───┐
           ├──→ PLL ──→ PLLCLK
HSE ───────┤
HSE / 2 ───┘
```

Tần số đầu ra có dạng:

```text
PLLCLK = PLL input × PLLMUL
```

`PLLMUL` cho phép hệ số:

```text
×2 → ×16
```

Ví dụ:

```text
HSE = 8 MHz
PLLMUL = ×9

PLLCLK = 8 MHz × 9
       = 72 MHz
```

Các bit cấu hình chính trong `RCC_CFGR`:

```text
PLLSRC
→ chọn HSI/2 hoặc HSE

PLLXTPRE
→ HSE hoặc HSE/2 trước PLL

PLLMUL
→ hệ số nhân PLL
```

Các bit trạng thái/điều khiển trong `RCC_CR`:

```text
PLLON
→ bật PLL

PLLRDY
→ PLL đã khóa và ổn định
```

Điểm quan trọng:

```text
Cấu hình PLL
→ phải thực hiện khi PLL đang tắt
```

Sau khi PLL được bật:

```text
PLLON = 1
    ↓
chờ PLLRDY = 1
    ↓
PLL mới sẵn sàng để chọn làm SYSCLK
```

Giới hạn cần nhớ:

```text
PLLCLK ≤ 72 MHz
```

Khi HSI/2 được dùng làm đầu vào PLL, tần số hệ thống tối đa đạt được thấp hơn trường hợp HSE phù hợp.

---

<a id="muc-02-05"></a>
## 2.5. SYSCLK / HCLK / PCLK1 / PCLK2

### SYSCLK

`SYSCLK` là system clock.

Nguồn có thể là:

```text
SYSCLK
├── HSI
├── HSE
└── PLLCLK
```

Chọn nguồn bằng:

```text
RCC_CFGR.SW
```

Kiểm tra nguồn thực sự đang được dùng bằng:

```text
RCC_CFGR.SWS
```

Sau reset:

```text
SYSCLK = HSI = 8 MHz
```

### HCLK

`HCLK` là clock của AHB domain:

```text
HCLK = SYSCLK / AHB Prescaler
```

HCLK cấp clock cho:

```text
Cortex-M3 core
AHB bus
Memory
DMA
```

Giới hạn:

```text
HCLK ≤ 72 MHz
```

### PCLK1

`PCLK1` là clock của APB1:

```text
PCLK1 = HCLK / APB1 Prescaler
```

Giới hạn:

```text
PCLK1 ≤ 36 MHz
```

APB1 thường chứa các peripheral như:

```text
TIM2-TIM7
I2C1/I2C2
SPI2/SPI3
USART2/USART3
...
```

Khả năng có mặt của từng peripheral phụ thuộc từng mã MCU.

### PCLK2

`PCLK2` là clock của APB2:

```text
PCLK2 = HCLK / APB2 Prescaler
```

Giới hạn:

```text
PCLK2 ≤ 72 MHz
```

APB2 thường chứa:

```text
AFIO
GPIOA-G
ADC
SPI1
USART1
TIM1 / TIM8
...
```

### Quan hệ tổng quát

```text
SYSCLK
   │
   ↓ AHB Prescaler
 HCLK
   │
   ├───────↓ APB1 Prescaler
   │      PCLK1
   │
   └───────↓ APB2 Prescaler
          PCLK2
```

### Clock khác cần nhận diện

```text
FCLK
→ Cortex-M3 free-running clock

SysTick
→ có thể dùng HCLK
  hoặc HCLK / 8

ADCCLK
→ PCLK2 / 2
  PCLK2 / 4
  PCLK2 / 6
  PCLK2 / 8

ADCCLK ≤ 14 MHz
```

---

<a id="muc-02-06"></a>
## 2.6. Prescaler và cách tính tần số

Prescaler là bộ chia tần số.

### AHB Prescaler

`HPRE` trong `RCC_CFGR`:

```text
SYSCLK
  ↓ HPRE
HCLK
```

Các hệ số chia:

```text
/1
/2
/4
/8
/16
/64
/128
/256
/512
```

Công thức:

```text
HCLK = SYSCLK / HPRE
```

### APB1 Prescaler

`PPRE1`:

```text
HCLK
  ↓ PPRE1
PCLK1
```

Các hệ số:

```text
/1
/2
/4
/8
/16
```

Công thức:

```text
PCLK1 = HCLK / PPRE1
```

Điều kiện:

```text
PCLK1 ≤ 36 MHz
```

### APB2 Prescaler

`PPRE2`:

```text
HCLK
  ↓ PPRE2
PCLK2
```

Các hệ số:

```text
/1
/2
/4
/8
/16
```

Công thức:

```text
PCLK2 = HCLK / PPRE2
```

Điều kiện:

```text
PCLK2 ≤ 72 MHz
```

### ADC Prescaler

`ADCPRE`:

```text
PCLK2
  ↓
/2, /4, /6 hoặc /8
  ↓
ADCCLK
```

Điều kiện:

```text
ADCCLK ≤ 14 MHz
```

### Ví dụ: hệ thống 72 MHz từ HSE 8 MHz

Giả sử:

```text
HSE = 8 MHz

PLL input = HSE
PLLMUL = ×9
```

Ta có:

```text
PLLCLK = 8 × 9
       = 72 MHz
```

Chọn:

```text
SYSCLK = PLLCLK
HPRE   = /1
PPRE1  = /2
PPRE2  = /1
ADCPRE = /6
```

Kết quả:

```text
SYSCLK = 72 MHz

HCLK
= 72 / 1
= 72 MHz

PCLK1
= 72 / 2
= 36 MHz

PCLK2
= 72 / 1
= 72 MHz

ADCCLK
= 72 / 6
= 12 MHz
```

Sơ đồ:

```text
HSE 8 MHz
    ↓
PLL ×9
    ↓
72 MHz SYSCLK
    ↓ HPRE /1
72 MHz HCLK
   ┌───────────────┐
   ↓               ↓
PPRE1 /2        PPRE2 /1
   ↓               ↓
36 MHz           72 MHz
PCLK1            PCLK2
                    ↓ ADCPRE /6
                  12 MHz
                  ADCCLK
```

---

<a id="muc-02-07"></a>
## 2.7. Peripheral Clock Enable và Peripheral Reset

Peripheral không tự động nhận clock chỉ vì CPU đang chạy.

RCC có các thanh ghi để bật clock riêng cho từng peripheral.

### AHB

```text
RCC_AHBENR
```

Ví dụ các khối:

```text
DMA1
DMA2
SRAM interface
CRC
FSMC
SDIO
```

### APB2

```text
RCC_APB2ENR
```

Ví dụ:

```text
AFIO
GPIOA
GPIOB
GPIOC
...
ADC1
ADC2
SPI1
USART1
TIM1
...
```

Ví dụ bật GPIOA:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
```

Ý nghĩa:

```text
IOPAEN = 0
→ clock GPIOA bị tắt

IOPAEN = 1
→ clock GPIOA được bật
```

Đối với STM32F1, GPIO nằm trên APB2.

### APB1

```text
RCC_APB1ENR
```

Ví dụ:

```text
TIM2
TIM3
TIM4
TIM5
I2C1
I2C2
SPI2
SPI3
USART2
USART3
...
```

### Vì sao phải bật Clock trước?

Khi peripheral clock không hoạt động, các thanh ghi của peripheral có thể không đọc được như bình thường; một số trường hợp giá trị đọc về có thể là `0`.

Quy trình:

```text
1. Bật peripheral clock
2. Cấu hình peripheral register
3. Cho peripheral hoạt động
```

Không nên đảo thành:

```text
cấu hình peripheral
→ rồi mới bật clock
```

### Peripheral Reset

RCC còn có thể reset riêng từng peripheral.

APB2:

```text
RCC_APB2RSTR
```

APB1:

```text
RCC_APB1RSTR
```

Ví dụ khái niệm:

```text
set RESET bit
    ↓
peripheral được reset
    ↓
clear RESET bit
    ↓
peripheral trở lại trạng thái hoạt động
```

Ví dụ:

```c
RCC->APB2RSTR |= RCC_APB2RSTR_IOPARST;
RCC->APB2RSTR &= ~RCC_APB2RSTR_IOPARST;
```

Reset peripheral hữu ích khi cần đưa một khối phần cứng về trạng thái reset mà không reset toàn MCU.

---

<a id="muc-02-08"></a>
## 2.8. Clock của Timer

Timer trên STM32F1 có một quy tắc đặc biệt.

### Trường hợp APB Prescaler = 1

Nếu:

```text
APB Prescaler = /1
```

thì:

```text
TIMxCLK = PCLKx
```

Ví dụ:

```text
PCLK2 = 72 MHz
PPRE2 = /1

TIM1CLK = 72 MHz
```

### Trường hợp APB Prescaler khác 1

Nếu:

```text
APB Prescaler = /2, /4, /8 hoặc /16
```

thì:

```text
TIMxCLK = 2 × PCLKx
```

Ví dụ:

```text
HCLK = 72 MHz
PPRE1 = /2

PCLK1 = 36 MHz

TIM2CLK
= 2 × PCLK1
= 72 MHz
```

Quy tắc tổng quát:

```text
if APB prescaler == 1:
    TIMxCLK = PCLKx
else:
    TIMxCLK = 2 × PCLKx
```

Bảng:

| APB Prescaler | `PCLKx` | `TIMxCLK` |
|---|---:|---:|
| `/1` | `HCLK` | `PCLKx` |
| `/2` | `HCLK/2` | `2 × PCLKx` |
| `/4` | `HCLK/4` | `2 × PCLKx` |
| `/8` | `HCLK/8` | `2 × PCLKx` |
| `/16` | `HCLK/16` | `2 × PCLKx` |

Đây là nguyên nhân rất phổ biến khiến phép tính Timer/PWM sai nếu chỉ lấy `PCLK1` hoặc `PCLK2` làm timer clock.

---

<a id="muc-02-09"></a>
## 2.9. Clock của các Peripheral quan trọng

Clock của peripheral quyết định trực tiếp các thông số thời gian.

### ADC

ADC nhận clock từ:

```text
PCLK2
  ↓
ADC Prescaler
  ↓
ADCCLK
```

Các lựa chọn:

```text
/2
/4
/6
/8
```

Giới hạn:

```text
ADCCLK ≤ 14 MHz
```

Ví dụ:

```text
PCLK2 = 72 MHz
ADCPRE = /6

ADCCLK = 12 MHz
```

### USB

USB cần:

```text
USBCLK = 48 MHz
```

Nguồn USB clock lấy từ PLL thông qua USB prescaler.

Hai cách điển hình:

```text
PLLCLK = 72 MHz
USB prescaler = /1.5
→ USBCLK = 48 MHz
```

hoặc:

```text
PLLCLK = 48 MHz
USB prescaler = /1
→ USBCLK = 48 MHz
```

### SysTick

SysTick có thể dùng:

```text
HCLK
```

hoặc:

```text
HCLK / 8
```

Sai cấu hình clock hệ thống có thể làm các hàm delay hoặc hệ thống thời gian dựa trên SysTick sai theo.

### RTC

Nguồn RTC có thể chọn:

```text
LSE
LSI
HSE / 128
```

LSE thường phù hợp khi cần thời gian chính xác và duy trì bằng `VBAT`.

### IWDG

IWDG dùng:

```text
LSI
```

Khi IWDG đã được khởi động, LSI được giữ hoạt động để cung cấp clock cho watchdog.

### UART / SPI / I2C / Timer

Các peripheral này phụ thuộc vào clock bus tương ứng và bộ chia nội bộ của chính peripheral.

Có thể hình dung:

```text
Bus Clock
    ↓
Peripheral divider / baud generator / prescaler
    ↓
Tốc độ hoạt động thực tế
```

Do đó nếu xác định sai `PCLK1`, `PCLK2` hoặc `TIMxCLK`, các thông số sau có thể sai:

```text
UART baud rate
SPI SCK
I2C timing
Timer period
PWM frequency
delay
ADC timing
```

---

<a id="muc-02-10"></a>
## 2.10. Quy trình cấu hình Clock

Ví dụ mục tiêu:

```text
HSE = 8 MHz
SYSCLK = 72 MHz
HCLK = 72 MHz
PCLK1 = 36 MHz
PCLK2 = 72 MHz
ADCCLK = 12 MHz
```

### Bước 1 — MCU bắt đầu bằng HSI

Sau reset:

```text
SYSCLK = HSI = 8 MHz
```

Đây là trạng thái an toàn để bắt đầu cấu hình hệ thống clock.

### Bước 2 — Cấu hình Flash latency

Khi tăng SYSCLK, Flash phải có số wait state phù hợp.

Đối với STM32F10xxx:

```text
0 < SYSCLK ≤ 24 MHz
→ 0 wait state

24 MHz < SYSCLK ≤ 48 MHz
→ 1 wait state

48 MHz < SYSCLK ≤ 72 MHz
→ 2 wait states
```

Với 72 MHz:

```text
FLASH_ACR.LATENCY = 2 wait states
```

Prefetch buffer nên được giữ bật khi chạy tần số cao và phải được giữ bật nếu dùng AHB prescaler khác `/1`.

### Bước 3 — Bật HSE

```text
RCC_CR.HSEON = 1
```

Chờ:

```text
RCC_CR.HSERDY = 1
```

Không nên sử dụng HSE làm nguồn hệ thống trước khi nó ổn định.

### Bước 4 — Cấu hình Prescaler

Với ví dụ 72 MHz:

```text
HPRE  = /1
PPRE1 = /2
PPRE2 = /1
ADCPRE = /6
```

Kết quả dự kiến:

```text
HCLK  = 72 MHz
PCLK1 = 36 MHz
PCLK2 = 72 MHz
ADCCLK = 12 MHz
```

### Bước 5 — Cấu hình PLL

Khi PLL đang tắt:

```text
PLLSRC   = HSE
PLLXTPRE = HSE không chia
PLLMUL   = ×9
```

Ta có:

```text
PLLCLK = 8 × 9
       = 72 MHz
```

### Bước 6 — Bật PLL

```text
RCC_CR.PLLON = 1
```

Chờ:

```text
RCC_CR.PLLRDY = 1
```

### Bước 7 — Chuyển SYSCLK sang PLL

Thiết lập:

```text
RCC_CFGR.SW = PLL
```

Sau đó kiểm tra:

```text
RCC_CFGR.SWS = PLL
```

Chỉ khi `SWS` xác nhận PLL đang được dùng thì có thể coi quá trình chuyển SYSCLK hoàn tất.

### Bước 8 — Bật clock cho peripheral cần dùng

Ví dụ:

```text
GPIOA
→ RCC_APB2ENR.IOPAEN

USART2
→ RCC_APB1ENR.USART2EN

DMA1
→ RCC_AHBENR.DMA1EN
```

### Bước 9 — Kiểm tra Clock Tree thực tế

Sau cấu hình, phải tự tính lại:

```text
SYSCLK
HCLK
PCLK1
PCLK2
TIMxCLK
ADCCLK
USBCLK nếu dùng
```

Không nên chỉ nhìn vào giá trị `SYSCLK`.

### Luồng hoàn chỉnh

```text
Reset
  ↓
HSI 8 MHz
  ↓
cấu hình Flash latency
  ↓
bật HSE
  ↓
chờ HSERDY
  ↓
cấu hình AHB/APB/ADC prescaler
  ↓
cấu hình PLL khi PLL đang OFF
  ↓
bật PLL
  ↓
chờ PLLRDY
  ↓
SW = PLL
  ↓
chờ SWS = PLL
  ↓
bật clock cho peripheral
```

### Các lỗi thường gặp

#### Không chờ `HSERDY`

```text
HSEON = 1
→ chuyển nguồn ngay
```

Cách đúng:

```text
HSEON = 1
→ chờ HSERDY
→ mới sử dụng HSE
```

#### Không chờ `PLLRDY`

```text
PLLON = 1
→ chọn PLL ngay
```

Cách đúng:

```text
PLLON = 1
→ chờ PLLRDY
→ mới chọn PLL
```

#### Cấu hình PLL khi PLL đang chạy

Các tham số như:

```text
PLLSRC
PLLXTPRE
PLLMUL
```

phải được cấu hình khi PLL đang tắt.

#### Vượt giới hạn APB1

Sai:

```text
HCLK = 72 MHz
PPRE1 = /1

PCLK1 = 72 MHz
```

Vì:

```text
PCLK1 tối đa = 36 MHz
```

Cách phù hợp:

```text
PPRE1 = /2
→ PCLK1 = 36 MHz
```

#### Tính sai Timer clock

Sai:

```text
PCLK1 = 36 MHz
→ TIM2CLK = 36 MHz
```

Nếu `PPRE1 != /1`:

```text
TIM2CLK = 2 × PCLK1
        = 72 MHz
```

#### Quên bật Peripheral Clock

Ví dụ:

```text
cấu hình GPIOA
nhưng IOPAEN = 0
```

Peripheral có thể không hoạt động đúng vì clock của nó chưa được cấp.

#### Tăng SYSCLK nhưng không cấu hình Flash latency phù hợp

Khi SYSCLK tăng, Flash cần số wait state tương ứng.

Với 72 MHz:

```text
2 wait states
```

phải được thiết lập trước khi chạy ở tần số đó.

---

### Clock Security System — CSS

CSS dùng để phát hiện lỗi HSE.

Nếu HSE hỏng khi đang được dùng trực tiếp hoặc gián tiếp làm SYSCLK:

```text
HSE failure
    ↓
CSS phát hiện lỗi
    ↓
chuyển SYSCLK sang HSI
    ↓
HSE bị tắt
    ↓
PLL cũng bị tắt nếu đang dùng HSE làm đầu vào
    ↓
NMI được tạo
```

CSS phù hợp với hệ thống cần xử lý tình huống mất external clock.

---

### MCO — Microcontroller Clock Output

MCO cho phép đưa một clock bên trong MCU ra chân ngoài để đo bằng oscilloscope hoặc logic analyzer.

Có thể chọn:

```text
SYSCLK
HSI
HSE
PLLCLK / 2
```

MCO hữu ích khi kiểm tra:

```text
Clock source có hoạt động không?
Tần số thực tế có đúng không?
PLL có tạo đúng clock không?
```

---

<a id="muc-02-11"></a>
## 2.11. Câu hỏi tự kiểm tra

1. RCC viết tắt của gì?
2. RCC có hai nhóm chức năng chính nào?
3. HSI của STM32F10xxx trong phạm vi này có tần số bao nhiêu?
4. HSI có thể đi vào PLL theo đường nào?
5. HSE crystal nằm trong khoảng tần số nào?
6. HSE và HSI khác nhau về phần cứng và độ chính xác như thế nào?
7. LSI thường được dùng cho peripheral nào?
8. LSE có tần số bao nhiêu và thường dùng cho gì?
9. Ba nguồn nào có thể được chọn làm SYSCLK?
10. Sau reset, nguồn nào được chọn làm SYSCLK?
11. `SYSCLK`, `HCLK`, `PCLK1`, `PCLK2` khác nhau như thế nào?
12. `HCLK` được tính từ `SYSCLK` bằng gì?
13. `PCLK1` tối đa bao nhiêu MHz?
14. `PCLK2` tối đa bao nhiêu MHz?
15. PLL của STM32F1 nhận những nguồn đầu vào nào?
16. Vì sao phải cấu hình PLL trước khi bật `PLLON`?
17. `PLLRDY` dùng để kiểm tra điều gì?
18. `SW` và `SWS` trong `RCC_CFGR` khác nhau như thế nào?
19. Tại sao phải chờ `HSERDY` trước khi dùng HSE?
20. Với HSE 8 MHz và PLL ×9, `PLLCLK` bằng bao nhiêu?
21. Nếu `SYSCLK = 72 MHz` và `HPRE = /1`, `HCLK` bằng bao nhiêu?
22. Nếu `HCLK = 72 MHz` và `PPRE1 = /2`, `PCLK1` bằng bao nhiêu?
23. Nếu `PCLK1 = 36 MHz` và APB1 prescaler khác `/1`, `TIM2CLK` bằng bao nhiêu?
24. Nếu `PCLK2 = 72 MHz` và `ADCPRE = /6`, `ADCCLK` bằng bao nhiêu?
25. Vì sao GPIOA phải được bật clock trước khi cấu hình?
26. GPIO trên STM32F1 nằm trên bus nào?
27. `RCC_APB2ENR` và `RCC_APB2RSTR` khác nhau thế nào?
28. USB cần clock bao nhiêu MHz?
29. SysTick có thể dùng những nguồn clock nào trong Clock Tree?
30. CSS làm gì khi HSE bị lỗi?
31. MCO có tác dụng gì?
32. Với `SYSCLK = 72 MHz`, Flash cần bao nhiêu wait state?
33. Hãy mô tả đầy đủ đường đi `HSE → PLL → SYSCLK → HCLK → PCLK1/PCLK2`.
34. Hãy mô tả trình tự cấu hình hệ thống từ HSI sau reset sang PLL 72 MHz.
35. Vì sao không thể chỉ nhìn `PCLK1` để xác định Timer clock?

---

## 2.12. Tóm tắt

```text
Clock sources
├── HSI = 8 MHz
├── HSE = external high-speed
├── LSI ≈ 40 kHz
└── LSE = 32.768 kHz
```

Clock hệ thống:

```text
HSI / HSE / PLL
       ↓
     SYSCLK
       ↓
  AHB Prescaler
       ↓
      HCLK
       ↓
 ┌─────┴─────┐
 ↓           ↓
APB1        APB2
 ↓           ↓
PCLK1       PCLK2
```

Giới hạn chính:

```text
HCLK  ≤ 72 MHz
PCLK2 ≤ 72 MHz
PCLK1 ≤ 36 MHz
ADCCLK ≤ 14 MHz
```

Timer:

```text
APB prescaler = /1
→ TIMxCLK = PCLKx

APB prescaler != /1
→ TIMxCLK = 2 × PCLKx
```

Ví dụ 72 MHz:

```text
HSE 8 MHz
  ↓ PLL ×9
SYSCLK 72 MHz
  ↓ HPRE /1
HCLK 72 MHz
  ├── PPRE1 /2 → PCLK1 36 MHz → TIMxCLK 72 MHz
  └── PPRE2 /1 → PCLK2 72 MHz → TIMxCLK 72 MHz
```

Trình tự cấu hình:

```text
HSI sau reset
  ↓
Flash latency
  ↓
HSEON → HSERDY
  ↓
Prescaler
  ↓
PLL config
  ↓
PLLON → PLLRDY
  ↓
SW = PLL
  ↓
SWS = PLL
  ↓
Peripheral Clock Enable
```

**Điểm cần nhớ:**

> **RCC không chỉ tạo clock cho CPU mà còn quyết định clock của toàn bộ bus và peripheral. Khi phân tích một peripheral, luôn xác định nguồn clock, prescaler của bus, clock thực tế của peripheral và bit Clock Enable tương ứng.**

[↑ Về mục lục](#muc-luc)
