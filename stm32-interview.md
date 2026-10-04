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
   - 3.1. GPIO là gì? Port và Pin
   - 3.2. Bật Clock cho GPIO
   - 3.3. Cấu trúc một GPIO Pin
   - 3.4. Các chế độ Input
   - 3.5. Các chế độ Output
   - 3.6. Push-Pull và Open-Drain
   - 3.7. Pull-Up / Pull-Down / Floating
   - 3.8. Output Speed: 2 / 10 / 50 MHz
   - 3.9. CRL / CRH và MODE / CNF
   - 3.10. IDR / ODR
   - 3.11. BSRR / BRR và thao tác Atomic
   - 3.12. Alternate Function
   - 3.13. AFIO và Pin Remapping
   - 3.14. Analog Mode
   - 3.15. GPIO cho UART / SPI / I2C / Timer / ADC
   - 3.16. GPIO Locking
   - 3.17. Quy trình cấu hình GPIO
   - 3.18. Câu hỏi tự kiểm tra
4. **Interrupt + NVIC + EXTI**
   - 4.1. Interrupt là gì? Polling và Interrupt
   - 4.2. Exception và Interrupt
   - 4.3. Vector Table và ISR / Handler
   - 4.4. Luồng xử lý Interrupt trên Cortex-M3
   - 4.5. NVIC là gì?
   - 4.6. Enable / Disable / Pending / Active
   - 4.7. Interrupt Priority
   - 4.8. Preemption và Nested Interrupt
   - 4.9. Priority Grouping
   - 4.10. EXTI là gì?
   - 4.11. EXTI Line và GPIO Mapping
   - 4.12. Rising Edge / Falling Edge
   - 4.13. IMR / EMR / RTSR / FTSR / SWIER / PR
   - 4.14. AFIO_EXTICR
   - 4.15. GPIO → AFIO → EXTI → NVIC
   - 4.16. Shared IRQ: EXTI5_9 và EXTI10_15
   - 4.17. Clear Pending Flag
   - 4.18. Quy trình cấu hình EXTI Interrupt
   - 4.19. Quy ước thiết kế ISR
   - 4.20. Ví dụ Button → EXTI → ISR
   - 4.21. Câu hỏi tự kiểm tra
5. **Timer + PWM**
   - 5.1. Timer là gì?
   - 5.2. Các loại Timer trên STM32F1
   - 5.3. Timer Clock
   - 5.4. Counter: CNT
   - 5.5. Prescaler: PSC
   - 5.6. Auto-Reload Register: ARR
   - 5.7. Up / Down / Center-Aligned Counting
   - 5.8. Update Event và Update Interrupt
   - 5.9. Công thức tính Timer Period / Frequency
   - 5.10. Capture/Compare Channel
   - 5.11. Output Compare
   - 5.12. Input Capture
   - 5.13. PWM là gì?
   - 5.14. PWM Frequency và Duty Cycle
   - 5.15. PWM Mode 1 / PWM Mode 2
   - 5.16. CCRx và Compare Match
   - 5.17. Preload: ARPE / OCxPE
   - 5.18. GPIO Alternate Function cho PWM
   - 5.19. Timer Interrupt và DMA
   - 5.20. Advanced Timer: Complementary PWM / Dead-Time / Break
   - 5.21. Quy trình cấu hình Timer
   - 5.22. Quy trình cấu hình PWM
   - 5.23. Ví dụ tính PSC / ARR / CCR
   - 5.24. Câu hỏi tự kiểm tra
6. **UART / USART**
   - 6.1. UART và USART là gì?
   - 6.2. Truyền nối tiếp bất đồng bộ
   - 6.3. UART Frame: Start / Data / Parity / Stop
   - 6.4. 8N1 và các cấu hình Frame
   - 6.5. Baud Rate
   - 6.6. USART Clock và USART_BRR
   - 6.7. TX / RX và GPIO
   - 6.8. USART_DR và cơ chế truyền dữ liệu
   - 6.9. TXE và TC
   - 6.10. RXNE và quá trình nhận dữ liệu
   - 6.11. Error Flags: ORE / FE / NE / PE
   - 6.12. USART Interrupt
   - 6.13. IDLE Line
   - 6.14. UART bằng Polling
   - 6.15. UART bằng Interrupt
   - 6.16. UART bằng DMA
   - 6.17. Hardware Flow Control: CTS / RTS
   - 6.18. Half-Duplex và Synchronous Mode
   - 6.19. Quy trình cấu hình UART
   - 6.20. Ví dụ USART1 115200 8N1
   - 6.21. Câu hỏi tự kiểm tra
7. **SPI + I2C**
   - 7.1. Tổng quan SPI và I2C
   - 7.2. SPI là gì?
   - 7.3. SCK / MOSI / MISO / NSS
   - 7.4. Master / Slave và Full-Duplex
   - 7.5. SPI Clock và Baud Rate Prescaler
   - 7.6. CPOL / CPHA và 4 SPI Mode
   - 7.7. Data Frame: 8/16-bit, MSB/LSB First
   - 7.8. NSS Hardware / Software
   - 7.9. SPI_DR và cơ chế Shift Register
   - 7.10. TXE / RXNE / BSY
   - 7.11. OVR / MODF / CRCERR
   - 7.12. SPI bằng Polling / Interrupt / DMA
   - 7.13. GPIO cho SPI
   - 7.14. Quy trình cấu hình SPI Master
   - 7.15. Ví dụ một SPI Transaction
   - 7.16. I2C là gì?
   - 7.17. SDA / SCL và Open-Drain
   - 7.18. START / STOP / Address / R/W / ACK / NACK
   - 7.19. 7-bit / 10-bit Addressing
   - 7.20. Master / Slave / Transmitter / Receiver
   - 7.21. Standard Mode / Fast Mode
   - 7.22. I2C Clock: CR2.FREQ / CCR / TRISE
   - 7.23. Các I2C Status Flag quan trọng
   - 7.24. Clock Stretching
   - 7.25. Arbitration và Multi-Master
   - 7.26. Repeated START
   - 7.27. Master Transmit / Master Receive
   - 7.28. I2C Interrupt / DMA
   - 7.29. Quy trình cấu hình I2C
   - 7.30. So sánh SPI và I2C
   - 7.31. Câu hỏi tự kiểm tra
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


---

<a id="chuong-03"></a>
# 3. GPIO

`GPIO` là viết tắt của:

```text
General-Purpose Input/Output
```

GPIO là giao diện số giữa vi điều khiển và thế giới bên ngoài. Một chân GPIO có thể được cấu hình để đọc tín hiệu, xuất tín hiệu hoặc được kết nối với một peripheral bên trong MCU thông qua Alternate Function.

Luồng cấu hình tổng quát:

```text
RCC
 ↓
bật clock cho GPIO / AFIO
 ↓
chọn Mode và Configuration
 ↓
đọc / ghi chân
hoặc
kết nối chân với peripheral
```

<a id="muc-03-01"></a>
## 3.1. GPIO là gì? Port và Pin

Các chân GPIO được tổ chức thành từng **port**:

```text
GPIOA
GPIOB
GPIOC
GPIOD
GPIOE
GPIOF
GPIOG
```

Mỗi port có tối đa 16 pin:

```text
GPIOA
├── PA0
├── PA1
├── PA2
├── ...
└── PA15
```

Cách đặt tên:

```text
PA5
│ │
│ └── Pin 5
└──── Port A
```

Không phải mọi port và mọi pin đều tồn tại trên mọi mã STM32F10xxx hoặc mọi package. Pin thực tế phải được kiểm tra theo pinout của MCU đang sử dụng.

### Trạng thái sau Reset

Đối với GPIO thông thường, ngay sau reset:

```text
MODE = 00
CNF  = 01
```

tương ứng:

```text
Input Floating
```

Alternate Function chưa hoạt động mặc định.

Một số chân JTAG/SWD là ngoại lệ vì được dành cho giao diện debug sau reset:

```text
PA13 → JTMS / SWDIO
PA14 → JTCK / SWCLK
PA15 → JTDI
PB3  → JTDO / TRACESWO
PB4  → NJTRST
```

Do đó, không nên giả định tất cả pin đều hoàn toàn tự do ngay sau reset.

---

<a id="muc-03-02"></a>
## 3.2. Bật Clock cho GPIO

Trên STM32F1, GPIO nằm trên bus `APB2`.

Trước khi cấu hình một port GPIO, phải bật clock cho port đó trong:

```text
RCC_APB2ENR
```

Ví dụ:

```text
IOPAEN
→ GPIOA clock enable

IOPBEN
→ GPIOB clock enable

IOPCEN
→ GPIOC clock enable
```

Ví dụ bật clock GPIOA:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
```

Luồng đúng:

```text
RCC
 ↓
bật GPIO clock
 ↓
cấu hình CRL / CRH
 ↓
đọc hoặc ghi GPIO
```

Nếu cần cấu hình `AFIO_MAPR`, `AFIO_EXTICR` hoặc các thanh ghi AFIO khác thì phải bật thêm:

```text
AFIOEN
```

Ví dụ:

```c
RCC->APB2ENR |= RCC_APB2ENR_AFIOEN;
```

Có thể bật nhiều clock trong một lần ghi:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN
               | RCC_APB2ENR_AFIOEN;
```

---

<a id="muc-03-03"></a>
## 3.3. Cấu trúc một GPIO Pin

Một GPIO pin có thể hình dung với các khối chính:

```text
                    +------------------+
External pin ───────┤ Protection       |
                    +------------------+
                             │
             ┌───────────────┴───────────────┐
             ↓                               ↓
       Input Driver                    Output Driver
             │                               │
      Schmitt Trigger                  P-MOS / N-MOS
             │                               │
             ↓                               ↑
           IDR                         ODR / Peripheral
```

Các thành phần quan trọng:

```text
Input path
→ đọc trạng thái trên chân

Output path
→ điều khiển mức logic trên chân

Pull-Up / Pull-Down
→ tạo mức mặc định cho input

Alternate Function
→ kết nối pin với peripheral

Analog path
→ kết nối pin với khối analog khi phù hợp
```

### Input Driver

Ở Digital Input, tín hiệu từ chân đi qua input driver và Schmitt trigger rồi được đưa vào `GPIOx_IDR`.

### Output Driver

Output driver sử dụng transistor P-MOS và N-MOS để tạo các kiểu:

```text
Push-Pull
Open-Drain
```

### Chân 5-V tolerant

Một số GPIO có đặc tính `5-V tolerant`, được biểu diễn bằng miền `VDD_FT`. Không được giả định mọi GPIO đều chịu được 5 V; phải kiểm tra đúng pin và đặc tính điện của MCU.

---

<a id="muc-03-04"></a>
## 3.4. Các chế độ Input

Khi:

```text
MODE[1:0] = 00
```

pin được cấu hình làm Input.

`CNF[1:0]` quyết định loại Input:

| `CNF[1:0]` | Input mode |
|---|---|
| `00` | Analog |
| `01` | Floating |
| `10` | Pull-Up / Pull-Down |
| `11` | Reserved |

Khi pin ở Digital Input:

```text
Output Buffer
→ OFF

Schmitt Trigger
→ ON

Input Data Register
→ nhận trạng thái chân
```

Trạng thái chân được lấy mẫu vào `GPIOx_IDR` theo clock APB2.

### Input Floating

```text
MODE = 00
CNF  = 01
```

Pin không sử dụng pull-up hoặc pull-down nội bộ.

```text
External signal
     ↓
GPIO pin
     ↓
Input driver
     ↓
IDR
```

Nếu bên ngoài không chủ động điều khiển chân, mức logic có thể không xác định.

### Input Pull-Up / Pull-Down

```text
MODE = 00
CNF  = 10
```

STM32F1 sử dụng bit tương ứng trong `ODR` để chọn hướng kéo:

```text
ODR bit = 1
→ Pull-Up

ODR bit = 0
→ Pull-Down
```

Ví dụ PA0 Input Pull-Up:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

/* PA0: MODE=00, CNF=10 */
GPIOA->CRL &= ~(0xFU << 0);
GPIOA->CRL |=  (0x8U << 0);

/* Chọn Pull-Up */
GPIOA->ODR |= (1U << 0);
```

Đây là đặc điểm quan trọng của GPIO STM32F1: `ODR` còn tham gia chọn Pull-Up/Pull-Down khi pin ở chế độ Input Pull-Up/Pull-Down.

---

<a id="muc-03-05"></a>
## 3.5. Các chế độ Output

Khi:

```text
MODE[1:0] != 00
```

pin được cấu hình làm Output hoặc Alternate Function Output.

Trong General-Purpose Output:

```text
CNF = 00
→ Push-Pull

CNF = 01
→ Open-Drain
```

Khi pin ở Output:

```text
Output Buffer
→ ON

Schmitt Trigger
→ ON

Weak Pull-Up / Pull-Down
→ OFF
```

Giá trị xuất được điều khiển bởi `GPIOx_ODR` hoặc các thanh ghi set/reset tương ứng.

Ví dụ PA5 làm General-Purpose Push-Pull Output:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

/* PA5: MODE=10 → 2 MHz, CNF=00 → Push-Pull */
GPIOA->CRL &= ~(0xFU << (5U * 4U));
GPIOA->CRL |=  (0x2U << (5U * 4U));
```

---

<a id="muc-03-06"></a>
## 3.6. Push-Pull và Open-Drain

### Push-Pull

Push-Pull sử dụng cả P-MOS và N-MOS.

```text
ODR = 1
→ P-MOS ON
→ pin được kéo lên mức cao

ODR = 0
→ N-MOS ON
→ pin được kéo xuống mức thấp
```

Sơ đồ:

```text
        VDD
         │
       P-MOS
         │
         ├──── GPIO Pin
         │
       N-MOS
         │
        GND
```

Đặc điểm:

```text
0 → chủ động kéo Low
1 → chủ động kéo High
```

Phù hợp với các tín hiệu số thông thường cần MCU chủ động điều khiển cả hai mức logic.

### Open-Drain

Open-Drain chỉ dùng N-MOS ở output driver.

```text
ODR = 0
→ N-MOS ON
→ pin bị kéo xuống Low

ODR = 1
→ N-MOS OFF
→ pin ở High-Z
```

Sơ đồ:

```text
External Pull-Up
      │
     VDD
      │
      R
      │
      ├──── GPIO Pin
      │
    N-MOS
      │
     GND
```

Open-Drain không tự chủ động tạo mức High. Khi cần mức High, đường tín hiệu thường cần pull-up thích hợp.

Đây là cấu hình tiêu biểu cho I2C:

```text
SCL → Alternate Function Open-Drain
SDA → Alternate Function Open-Drain
```

### So sánh

| Đặc điểm | Push-Pull | Open-Drain |
|---|---|---|
| Chủ động kéo Low | Có | Có |
| Chủ động kéo High | Có | Không |
| Trạng thái khi xuất `1` | High | High-Z |
| Thường cần pull-up ngoài | Không | Có trong các bus như I2C |
| Ví dụ | LED, UART TX, SPI output | I2C SDA/SCL |

---

<a id="muc-03-07"></a>
## 3.7. Pull-Up / Pull-Down / Floating

### Pull-Up

Pull-Up tạo xu hướng mặc định:

```text
không có tín hiệu ngoài
→ logic 1
```

Sơ đồ:

```text
VDD
 │
 R
 │
 ├──── GPIO Input
```

### Pull-Down

Pull-Down tạo xu hướng mặc định:

```text
không có tín hiệu ngoài
→ logic 0
```

Sơ đồ:

```text
GPIO Input
 │
 R
 │
GND
```

### Floating

Floating không dùng điện trở kéo nội bộ:

```text
GPIO Input
→ không Pull-Up
→ không Pull-Down
```

Floating phù hợp khi nguồn tín hiệu bên ngoài luôn chủ động điều khiển mức logic.

Không nên để một input không được điều khiển ở trạng thái floating nếu ứng dụng cần mức logic xác định.

---

<a id="muc-03-08"></a>
## 3.8. Output Speed: 2 / 10 / 50 MHz

Ở Output mode, `MODE[1:0]` vừa cho biết pin là Output vừa chọn **maximum output speed**.

| `MODE[1:0]` | Chế độ |
|---|---|
| `00` | Input |
| `01` | Output, max speed 10 MHz |
| `10` | Output, max speed 2 MHz |
| `11` | Output, max speed 50 MHz |

Điểm cần phân biệt:

```text
Output Speed
≠
tần số mà phần mềm bắt buộc phải toggle pin
```

Đây là lựa chọn khả năng tốc độ của output driver.

Ví dụ một LED không cần cấu hình 50 MHz chỉ vì CPU chạy ở 72 MHz.

Cấu hình tốc độ nên phù hợp với nhu cầu tín hiệu và đặc tính phần cứng của hệ thống.

---

<a id="muc-03-09"></a>
## 3.9. CRL / CRH và MODE / CNF

STM32F1 dùng hai thanh ghi cấu hình cho mỗi port:

```text
GPIOx_CRL
→ Pin 0 → Pin 7

GPIOx_CRH
→ Pin 8 → Pin 15
```

Mỗi pin sử dụng 4 bit:

```text
CNF[1:0] MODE[1:0]
```

Sơ đồ:

```text
4 bit / pin

+------+------+------+------+
| CNF1 | CNF0 | MODE1| MODE0|
+------+------+------+------+
```

### CRL

`GPIOx_CRL` chứa cấu hình:

```text
Pin 0
Pin 1
...
Pin 7
```

Ví dụ bit của pin 5:

```text
Pin 5
→ CRL[23:20]
```

### CRH

`GPIOx_CRH` chứa cấu hình:

```text
Pin 8
Pin 9
...
Pin 15
```

Ví dụ pin 13:

```text
Pin 13
→ CRH[23:20]
```

### Bảng MODE/CNF tổng hợp

| Chức năng | MODE | CNF |
|---|---|---|
| Analog Input | `00` | `00` |
| Floating Input | `00` | `01` |
| Pull-Up/Pull-Down Input | `00` | `10` |
| General-Purpose Push-Pull | `01/10/11` | `00` |
| General-Purpose Open-Drain | `01/10/11` | `01` |
| Alternate Function Push-Pull | `01/10/11` | `10` |
| Alternate Function Open-Drain | `01/10/11` | `11` |

Với Output, `MODE` chọn:

```text
01 → 10 MHz
10 → 2 MHz
11 → 50 MHz
```

### Công thức vị trí bit

Với pin `n` từ `0` đến `7`:

```text
shift = n × 4
→ cấu hình trong CRL
```

Với pin `n` từ `8` đến `15`:

```text
shift = (n - 8) × 4
→ cấu hình trong CRH
```

Ví dụ PA9:

```text
PA9
→ CRH
→ shift = (9 - 8) × 4
        = 4
```

---

<a id="muc-03-10"></a>
## 3.10. IDR / ODR

### GPIOx_IDR — Input Data Register

`IDR` chứa trạng thái input của các pin:

```text
IDR0  → Pin 0
IDR1  → Pin 1
...
IDR15 → Pin 15
```

Ví dụ đọc PA0:

```c
uint32_t state = (GPIOA->IDR >> 0) & 1U;
```

Hoặc:

```c
if (GPIOA->IDR & (1U << 0))
{
    /* PA0 đang ở mức logic 1 */
}
```

### GPIOx_ODR — Output Data Register

`ODR` chứa output latch của các pin:

```text
ODR0  → Pin 0
...
ODR15 → Pin 15
```

Ví dụ set PA5 bằng ODR:

```c
GPIOA->ODR |= (1U << 5);
```

Reset PA5:

```c
GPIOA->ODR &= ~(1U << 5);
```

Toggle PA5:

```c
GPIOA->ODR ^= (1U << 5);
```

### IDR và ODR không giống nhau

```text
ODR
→ giá trị output latch

IDR
→ trạng thái được input path lấy từ chân
```

Ở chế độ Output, input path vẫn có thể hoạt động, vì vậy có thể đọc trạng thái chân qua `IDR`.

---

<a id="muc-03-11"></a>
## 3.11. BSRR / BRR và thao tác Atomic

### Vấn đề của Read-Modify-Write trên ODR

Lệnh:

```c
GPIOA->ODR |= (1U << 5);
```

về bản chất gồm:

```text
READ ODR
   ↓
MODIFY
   ↓
WRITE ODR
```

Nếu nhiều ngữ cảnh cùng sửa `ODR`, thao tác read-modify-write có thể tác động tới các bit khác ngoài ý muốn nếu không được kiểm soát đúng.

### GPIOx_BSRR

`BSRR` cho phép set/reset pin bằng một lần ghi APB2.

Cấu trúc:

```text
BSRR[15:0]
→ SET pin 0 → 15

BSRR[31:16]
→ RESET pin 0 → 15
```

Set PA5:

```c
GPIOA->BSRR = (1U << 5);
```

Reset PA5:

```c
GPIOA->BSRR = (1U << (5 + 16));
```

Có thể set/reset nhiều pin trong cùng một lần ghi:

```c
GPIOA->BSRR = (1U << 5)
            | (1U << 6)
            | (1U << (7 + 16));
```

Ý nghĩa:

```text
PA5 → Set
PA6 → Set
PA7 → Reset
```

Nếu cả bit Set và Reset của cùng một pin cùng được ghi `1`, thao tác Set có ưu tiên.

### GPIOx_BRR

`BRR` chỉ dùng để reset các bit:

```text
ghi 1
→ reset bit tương ứng trong ODR
```

Ví dụ:

```c
GPIOA->BRR = (1U << 5);
```

### Vì sao BSRR quan trọng?

```text
ODR read-modify-write
→ nhiều bước

BSRR
→ một lần ghi
→ atomic ở mức thao tác set/reset GPIO
```

Khi chỉ cần Set/Reset pin, ưu tiên `BSRR` giúp tránh read-modify-write không cần thiết.

---

<a id="muc-03-12"></a>
## 3.12. Alternate Function

Một chân GPIO có thể được kết nối với tín hiệu từ peripheral bên trong MCU.

Ví dụ:

```text
USART1
  │
  └── TX
       ↓
      PA9
```

Khi PA9 được cấu hình Alternate Function Output:

```text
USART1 peripheral
       ↓
Alternate Function
       ↓
GPIO output driver
       ↓
PA9
```

Khi đó output không còn được điều khiển như General-Purpose Output thông thường mà được điều khiển bởi peripheral.

### Alternate Function Output

Hai kiểu:

```text
Alternate Function Push-Pull
Alternate Function Open-Drain
```

Trong Alternate Function Output:

```text
Output Buffer
→ ON

Nguồn điều khiển Output Buffer
→ peripheral

Input path
→ vẫn có thể hoạt động
```

### Alternate Function Input

Đối với các tín hiệu peripheral là input, chân được cấu hình theo input mode phù hợp:

```text
Floating
Pull-Up
Pull-Down
```

Ví dụ:

```text
USART RX
→ Input Floating hoặc Input Pull-Up
```

---

<a id="muc-03-13"></a>
## 3.13. AFIO và Pin Remapping

`AFIO` là:

```text
Alternate Function I/O
```

AFIO cho phép thay đổi mapping của một số peripheral sang các chân khác.

Thanh ghi quan trọng:

```text
AFIO_MAPR
```

Trước khi truy cập các thanh ghi AFIO:

```c
RCC->APB2ENR |= RCC_APB2ENR_AFIOEN;
```

### USART1 Remap

Mặc định:

```text
USART1_TX → PA9
USART1_RX → PA10
```

Remap:

```text
USART1_TX → PB6
USART1_RX → PB7
```

### I2C1 Remap

Mặc định:

```text
I2C1_SCL → PB6
I2C1_SDA → PB7
```

Remap:

```text
I2C1_SCL → PB8
I2C1_SDA → PB9
```

### SPI1 Remap

Mặc định:

```text
SPI1_NSS  → PA4
SPI1_SCK  → PA5
SPI1_MISO → PA6
SPI1_MOSI → PA7
```

Remap:

```text
SPI1_NSS  → PA15
SPI1_SCK  → PB3
SPI1_MISO → PB4
SPI1_MOSI → PB5
```

### Timer Remap

Một số Timer hỗ trợ:

```text
No Remap
Partial Remap
Full Remap
```

Ví dụ TIM3 có thể chuyển các channel từ nhóm chân mặc định sang một nhóm chân khác tùy lựa chọn remap.

Không được suy đoán pin Alternate Function theo tên peripheral. Phải kiểm tra pin mapping và remap của đúng MCU/package.

### JTAG / SWD Remap

Các chân debug mặc định:

```text
PA13 → JTMS / SWDIO
PA14 → JTCK / SWCLK
PA15 → JTDI
PB3  → JTDO / TRACESWO
PB4  → NJTRST
```

`AFIO_MAPR.SWJ_CFG` cho phép thay đổi cấu hình debug:

```text
Full JTAG + SWD
JTAG không NJTRST
JTAG OFF, SWD ON
JTAG OFF, SWD OFF
```

Một cấu hình thường dùng khi cần giải phóng PA15/PB3/PB4 nhưng vẫn giữ khả năng debug là:

```text
JTAG OFF
SWD ON
```

Không nên tắt cả SWD khi vẫn cần ST-Link để debug/nạp chương trình.

---

<a id="muc-03-14"></a>
## 3.14. Analog Mode

Analog mode:

```text
MODE = 00
CNF  = 00
```

Khi pin ở Analog mode:

```text
Output Buffer
→ OFF

Schmitt Trigger / Digital Input
→ OFF

Weak Pull-Up / Pull-Down
→ OFF

IDR
→ đọc về 0
```

Sơ đồ:

```text
Analog signal
     ↓
GPIO Pin
     ↓
Analog peripheral
ADC / DAC
```

GPIO dùng làm ADC input phải được cấu hình Analog.

Ví dụ PA0 Analog Input:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

/* PA0: MODE=00, CNF=00 */
GPIOA->CRL &= ~(0xFU << 0);
```

Analog mode cũng tránh để digital input path hoạt động không cần thiết trên một tín hiệu analog.

---

<a id="muc-03-15"></a>
## 3.15. GPIO cho UART / SPI / I2C / Timer / ADC

GPIO mode phụ thuộc vào hướng và chức năng của peripheral.

### USART

Ví dụ Full-Duplex:

```text
USART_TX
→ Alternate Function Push-Pull

USART_RX
→ Input Floating
  hoặc Input Pull-Up
```

Ví dụ USART1 mặc định:

```text
PA9  → TX → AF Push-Pull
PA10 → RX → Input
```

### SPI

Ở Master mode:

```text
SCK
→ Alternate Function Push-Pull

MOSI
→ Alternate Function Push-Pull

MISO
→ Input Floating / Pull-Up
```

`NSS` phụ thuộc cách quản lý NSS bằng hardware/software.

Ở Slave mode, hướng của SCK/MOSI/MISO thay đổi theo vai trò của peripheral.

### I2C

```text
SCL
→ Alternate Function Open-Drain

SDA
→ Alternate Function Open-Drain
```

Hai đường I2C cần cơ chế kéo lên phù hợp để tạo mức High.

### Timer

Input Capture:

```text
TIMx_CHy
→ Input Floating
```

Output Compare / PWM:

```text
TIMx_CHy
→ Alternate Function Push-Pull
```

### ADC

```text
ADC input
→ Analog Mode
```

### EXTI

GPIO dùng làm external interrupt phải ở Input mode:

```text
Input Floating
hoặc
Input Pull-Up
hoặc
Input Pull-Down
```

Luồng khái niệm:

```text
GPIO Input
    ↓
AFIO / EXTI mapping
    ↓
EXTI
    ↓
NVIC
    ↓
ISR
```

Cấu hình chi tiết EXTI được xử lý ở chương **Interrupt + NVIC + EXTI**.

### Bảng tóm tắt

| Peripheral / Signal | GPIO mode thường dùng trên STM32F1 |
|---|---|
| UART TX | Alternate Function Push-Pull |
| UART RX | Input Floating / Pull-Up |
| SPI Master SCK | Alternate Function Push-Pull |
| SPI Master MOSI | Alternate Function Push-Pull |
| SPI Master MISO | Input Floating / Pull-Up |
| I2C SCL | Alternate Function Open-Drain |
| I2C SDA | Alternate Function Open-Drain |
| Timer Input Capture | Input Floating |
| Timer PWM / Output Compare | Alternate Function Push-Pull |
| ADC Input | Analog |
| EXTI Input | Input Floating / Pull-Up / Pull-Down |

---

<a id="muc-03-16"></a>
## 3.16. GPIO Locking

`GPIOx_LCKR` cho phép khóa cấu hình của một hoặc nhiều pin.

Sau khi lock sequence hoàn tất:

```text
CRL / CRH của pin bị khóa
→ không thể thay đổi
→ cho tới lần reset tiếp theo
```

Các bit:

```text
LCK0 → Pin 0
...
LCK15 → Pin 15

LCKK → Lock Key
```

Lock sequence:

```text
1. Write LCKK = 1
2. Write LCKK = 0
3. Write LCKK = 1
4. Read LCKK → 0
5. Read LCKK → 1   (xác nhận)
```

Trong toàn bộ chuỗi, các bit `LCK[15:0]` không được thay đổi.

GPIO Lock phù hợp khi ứng dụng muốn cố định cấu hình chân sau giai đoạn khởi tạo.

---

<a id="muc-03-17"></a>
## 3.17. Quy trình cấu hình GPIO

Quy trình tổng quát:

```text
1. Xác định Port và Pin
        ↓
2. Bật RCC clock cho GPIO
        ↓
3. Nếu cần remap → bật AFIO clock
        ↓
4. Chọn Input / Output / AF / Analog
        ↓
5. Cấu hình MODE + CNF trong CRL/CRH
        ↓
6. Nếu Input Pull-Up/Pull-Down
   → cấu hình ODR
        ↓
7. Nếu Alternate Function
   → cấu hình peripheral và AFIO remap nếu cần
        ↓
8. Đọc IDR hoặc ghi BSRR/ODR
```

### Ví dụ 1 — PA5 Output Push-Pull, 2 MHz

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

/* Xóa 4 bit cấu hình PA5 */
GPIOA->CRL &= ~(0xFU << (5U * 4U));

/* MODE=10: Output 2 MHz
   CNF=00 : General-Purpose Push-Pull */
GPIOA->CRL |=  (0x2U << (5U * 4U));
```

Set PA5:

```c
GPIOA->BSRR = (1U << 5);
```

Reset PA5:

```c
GPIOA->BSRR = (1U << (5 + 16));
```

### Ví dụ 2 — PA0 Input Pull-Up

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;

/* MODE=00, CNF=10 */
GPIOA->CRL &= ~(0xFU << 0);
GPIOA->CRL |=  (0x8U << 0);

/* Pull-Up */
GPIOA->ODR |= (1U << 0);
```

Đọc PA0:

```c
if (GPIOA->IDR & (1U << 0))
{
    /* High */
}
else
{
    /* Low */
}
```

### Ví dụ 3 — USART1 TX/RX mặc định

Bật clock:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
RCC->APB2ENR |= RCC_APB2ENR_USART1EN;
```

PA9 — TX:

```text
Alternate Function Push-Pull
```

PA10 — RX:

```text
Input Floating
hoặc
Input Pull-Up
```

Ví dụ cấu hình PA9 AF Push-Pull 50 MHz:

```c
/* PA9 nằm trong CRH, shift = 4 */
GPIOA->CRH &= ~(0xFU << 4);

/* MODE=11, CNF=10 → 0b1011 = 0xB */
GPIOA->CRH |=  (0xBU << 4);
```

PA10 Input Floating:

```c
/* PA10 nằm trong CRH, shift = 8 */
GPIOA->CRH &= ~(0xFU << 8);

/* MODE=00, CNF=01 → 0b0100 = 0x4 */
GPIOA->CRH |=  (0x4U << 8);
```

### Ví dụ 4 — I2C1 SCL/SDA

Mặc định:

```text
PB6 → I2C1_SCL
PB7 → I2C1_SDA
```

Cả hai:

```text
Alternate Function Open-Drain
```

Ví dụ 50 MHz output configuration:

```text
MODE = 11
CNF  = 11
→ 0b1111
```

Cần đảm bảo hệ thống có pull-up phù hợp trên SCL/SDA.

---

### Lỗi thường gặp

#### Quên bật RCC Clock

```text
cấu hình GPIO register
nhưng GPIO clock chưa bật
```

→ GPIO không hoạt động như mong muốn.

#### Cấu hình sai CRL / CRH

```text
Pin 0–7
→ CRL

Pin 8–15
→ CRH
```

#### Nhầm MODE và CNF

```text
MODE
→ Input hay Output + maximum output speed

CNF
→ kiểu Input / Output / Alternate Function
```

#### Input Pull-Up nhưng quên ODR

```text
MODE = 00
CNF  = 10
```

chưa đủ.

Cần:

```text
ODR = 1 → Pull-Up
ODR = 0 → Pull-Down
```

#### Dùng ODR read-modify-write khi chỉ cần Set/Reset

Nếu chỉ cần thay đổi pin riêng lẻ:

```text
BSRR
→ phù hợp hơn cho thao tác atomic set/reset
```

#### Dùng I2C ở Push-Pull

I2C SCL/SDA trên STM32F1 phải được cấu hình:

```text
Alternate Function Open-Drain
```

#### Dùng ADC nhưng để Digital Input

ADC input nên được cấu hình:

```text
Analog Mode
```

#### Remap nhưng quên bật AFIO Clock

Trước khi sửa `AFIO_MAPR`:

```text
AFIOEN = 1
```

#### Dùng chân JTAG/SWD như GPIO mà không xử lý debug mapping

PA13/PA14/PA15/PB3/PB4 có liên quan đến JTAG/SWD. Nếu cần dùng chúng cho GPIO phải xem `SWJ_CFG` và giữ đường debug cần thiết.

---

<a id="muc-03-18"></a>
## 3.18. Câu hỏi tự kiểm tra

1. GPIO viết tắt của gì?
2. `PA5` có ý nghĩa gì?
3. GPIO trên STM32F1 nằm trên bus nào?
4. Thanh ghi nào dùng để bật clock GPIOA?
5. Sau reset, GPIO thông thường ở mode nào?
6. Vì sao PA13/PA14/PA15/PB3/PB4 cần chú ý đặc biệt?
7. `CRL` cấu hình những pin nào?
8. `CRH` cấu hình những pin nào?
9. Mỗi pin dùng bao nhiêu bit cấu hình trong CRL/CRH?
10. `MODE[1:0] = 00` có nghĩa gì?
11. Khi Input, `CNF=00`, `01`, `10` lần lượt là gì?
12. Khi Output, `CNF=00`, `01`, `10`, `11` lần lượt là gì?
13. Output speed có những lựa chọn nào?
14. Push-Pull hoạt động như thế nào khi xuất `0` và `1`?
15. Open-Drain hoạt động như thế nào khi xuất `0` và `1`?
16. Vì sao I2C dùng Open-Drain?
17. Floating Input khác Pull-Up/Pull-Down ở điểm nào?
18. Trên STM32F1, chọn Pull-Up hay Pull-Down bằng thanh ghi nào?
19. `IDR` dùng để làm gì?
20. `ODR` dùng để làm gì?
21. `BSRR` khác `ODR` ở điểm nào?
22. 16 bit thấp của `BSRR` làm gì?
23. 16 bit cao của `BSRR` làm gì?
24. Vì sao `BSRR` phù hợp cho atomic set/reset?
25. `BRR` dùng để làm gì?
26. Alternate Function là gì?
27. UART TX thường dùng GPIO mode nào?
28. UART RX thường dùng GPIO mode nào?
29. I2C SCL/SDA dùng GPIO mode nào?
30. SPI Master SCK/MOSI dùng GPIO mode nào?
31. Timer PWM output dùng GPIO mode nào?
32. ADC input dùng GPIO mode nào?
33. AFIO dùng để làm gì?
34. Trước khi truy cập `AFIO_MAPR` cần bật clock nào?
35. USART1 mặc định và remap dùng những chân nào?
36. SPI1 mặc định dùng PA4–PA7; sau remap dùng những chân nào?
37. `SWJ_CFG` liên quan tới chức năng gì?
38. Vì sao không nên tắt SWD khi vẫn cần ST-Link?
39. GPIO Locking có tác dụng gì?
40. Hãy mô tả đầy đủ quy trình cấu hình một GPIO output.
41. Hãy mô tả đầy đủ quy trình cấu hình một GPIO input có pull-up.
42. Hãy cấu hình về mặt khái niệm PA9/PA10 cho USART1.
43. Hãy giải thích sự khác nhau giữa `IDR`, `ODR` và `BSRR`.
44. Hãy giải thích vì sao Input Pull-Up trên STM32F1 vẫn cần thiết lập bit `ODR`.
45. Hãy giải thích luồng `Peripheral → Alternate Function → GPIO Pin`.

---

## 3.19. Tóm tắt

GPIO STM32F1:

```text
GPIO
├── Input
│   ├── Analog
│   ├── Floating
│   └── Pull-Up / Pull-Down
│
└── Output
    ├── General-Purpose
    │   ├── Push-Pull
    │   └── Open-Drain
    │
    └── Alternate Function
        ├── Push-Pull
        └── Open-Drain
```

Thanh ghi chính:

```text
CRL
→ Pin 0–7

CRH
→ Pin 8–15

IDR
→ đọc trạng thái pin

ODR
→ output latch / chọn Pull-Up-Pull-Down khi Input PU/PD

BSRR
→ atomic Set / Reset

BRR
→ Reset

LCKR
→ khóa cấu hình
```

Cấu hình một pin:

```text
RCC Clock Enable
      ↓
CRL / CRH
      ↓
MODE + CNF
      ↓
IDR / ODR / BSRR
```

Peripheral:

```text
UART TX
→ AF Push-Pull

UART RX
→ Input

SPI Master Output
→ AF Push-Pull

I2C
→ AF Open-Drain

Timer PWM
→ AF Push-Pull

ADC
→ Analog
```

AFIO:

```text
Peripheral
   ↓
Default mapping
hoặc
Remapping
   ↓
GPIO Pin
```

**Điểm cần nhớ:**

> **Trên STM32F1, muốn sử dụng GPIO phải bắt đầu từ RCC, sau đó cấu hình `MODE/CNF` trong `CRL/CRH`. `IDR` dùng để đọc chân, `ODR` lưu trạng thái output, còn `BSRR` cho phép set/reset pin bằng thao tác atomic. Khi pin phục vụ peripheral, phải chọn đúng Alternate Function và xử lý AFIO/remap nếu cần.**

[↑ Về mục lục](#muc-luc)


---

<a id="chuong-04"></a>
# 4. Interrupt + NVIC + EXTI

Interrupt cho phép CPU phản ứng với một sự kiện mà không phải liên tục kiểm tra trạng thái bằng polling.

Luồng tổng quát:

```text
Sự kiện
  ↓
Peripheral / EXTI tạo interrupt request
  ↓
NVIC
  ↓
Enable + Priority + Pending
  ↓
CPU chuyển sang Handler mode
  ↓
ISR / Handler
  ↓
xử lý nguyên nhân interrupt
  ↓
clear flag cần thiết
  ↓
Exception Return
  ↓
CPU tiếp tục code trước đó
```

Trên STM32F10xxx, NVIC hỗ trợ nhiều nguồn interrupt từ peripheral và sử dụng 4 bit priority, tương ứng 16 mức priority lập trình được.

EXTI chịu trách nhiệm phát hiện các cạnh tín hiệu trên external interrupt/event line và có thể tạo interrupt request hoặc event request.

<a id="muc-04-01"></a>
## 4.1. Interrupt là gì? Polling và Interrupt

### Polling

Polling là cách CPU chủ động kiểm tra trạng thái liên tục.

Ví dụ:

```c
while (1)
{
    if (GPIOC->IDR & (1U << 13))
    {
        /* xử lý */
    }
}
```

Luồng:

```text
CPU
 ↓
kiểm tra trạng thái
 ↓
chưa có sự kiện?
 ↓
kiểm tra lại
 ↓
kiểm tra lại
 ↓
...
```

Ưu điểm:

- Luồng chương trình đơn giản.
- Dễ theo dõi khi hệ thống nhỏ.

Hạn chế:

- CPU phải liên tục kiểm tra.
- Có thể lãng phí thời gian xử lý.
- Nếu vòng polling quá chậm, một số sự kiện ngắn có thể bị bỏ lỡ.

### Interrupt

Với interrupt:

```text
CPU chạy công việc chính
        ↓
sự kiện xảy ra
        ↓
hardware tạo interrupt request
        ↓
CPU tạm chuyển sang ISR
        ↓
ISR xử lý
        ↓
CPU quay lại công việc trước đó
```

Ví dụ:

```c
volatile uint8_t button_event = 0;

void EXTI15_10_IRQHandler(void)
{
    if (EXTI->PR & (1U << 13))
    {
        EXTI->PR = (1U << 13);
        button_event = 1;
    }
}
```

Main loop không cần liên tục đọc chân PC13 để phát hiện cạnh:

```c
while (1)
{
    if (button_event)
    {
        button_event = 0;

        /* xử lý sự kiện */
    }
}
```

### So sánh

| Đặc điểm | Polling | Interrupt |
|---|---|---|
| Ai chủ động kiểm tra? | CPU | Hardware báo CPU |
| CPU phải kiểm tra liên tục | Có | Không |
| Phản ứng với sự kiện | Phụ thuộc chu kỳ polling | Theo cơ chế interrupt |
| Độ phức tạp | Thấp | Cao hơn |
| Phù hợp | Hệ thống đơn giản | Nhiều sự kiện bất đồng bộ |

---

<a id="muc-04-02"></a>
## 4.2. Exception và Interrupt

Trong Cortex-M, **exception** là khái niệm tổng quát cho các sự kiện làm thay đổi luồng điều khiển sang một handler.

Có thể chia ở mức khái niệm:

```text
Exception
├── System Exception
│   ├── Reset
│   ├── NMI
│   ├── HardFault
│   ├── MemManage
│   ├── BusFault
│   ├── UsageFault
│   ├── SVCall
│   ├── PendSV
│   └── SysTick
│
└── External Interrupt / IRQ
    ├── EXTI
    ├── Timer
    ├── USART
    ├── SPI
    ├── I2C
    ├── ADC
    ├── DMA
    └── ...
```

Điểm cần phân biệt:

```text
External Interrupt
→ một loại Exception

Exception
→ không chỉ có External Interrupt
```

Một số exception có priority cố định hoặc đặc biệt:

```text
Reset
NMI
HardFault
```

Các IRQ từ peripheral được quản lý thông qua NVIC và có priority lập trình được.

---

<a id="muc-04-03"></a>
## 4.3. Vector Table và ISR / Handler

Vector Table đã xuất hiện ở phần Reset Sequence và Startup Code.

Các entry đầu tiên có dạng:

```text
Vector Table

0x00000000 → Initial Stack Pointer
0x00000004 → Reset_Handler
0x00000008 → NMI_Handler
0x0000000C → HardFault_Handler
...
```

Các entry phía sau chứa địa chỉ handler của từng exception và IRQ.

Ví dụ:

```text
EXTI0_IRQn
→ EXTI0_IRQHandler

TIM2_IRQn
→ TIM2_IRQHandler

USART1_IRQn
→ USART1_IRQHandler
```

### ISR là gì?

`ISR`:

```text
Interrupt Service Routine
```

ISR là hàm được thực thi để xử lý một interrupt.

Trong hệ sinh thái STM32/CMSIS, các ISR thường có tên handler tương ứng với vector:

```c
void EXTI0_IRQHandler(void)
{
    /* xử lý EXTI0 */
}
```

### Từ IRQ tới Handler

```text
Peripheral / EXTI
      ↓
IRQ request
      ↓
NVIC
      ↓
Vector Table
      ↓
Handler address
      ↓
ISR
```

Do đó, Vector Table là cầu nối giữa số exception/IRQ và địa chỉ hàm xử lý.

---

<a id="muc-04-04"></a>
## 4.4. Luồng xử lý Interrupt trên Cortex-M3

Khi một interrupt hợp lệ được CPU chấp nhận:

```text
Thread mode
    ↓
Interrupt accepted
    ↓
Hardware stacking
    ↓
Handler mode
    ↓
ISR
    ↓
Exception Return
    ↓
Hardware un-stacking
    ↓
Thread mode tiếp tục
```

Phần Stack đã mô tả exception stack frame cơ bản:

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

Ý nghĩa:

```text
CPU tự lưu context tối thiểu
→ handler có thể bắt đầu chạy

khi handler kết thúc
→ CPU khôi phục context
→ tiếp tục chương trình trước interrupt
```

Không cần tự viết code push/pop cho exception frame cơ bản.

Handler chạy ở:

```text
Handler mode
→ Privileged
→ dùng MSP
```

---

<a id="muc-04-05"></a>
## 4.5. NVIC là gì?

`NVIC` là:

```text
Nested Vectored Interrupt Controller
```

NVIC được tích hợp chặt với Cortex-M3 để quản lý exception và external interrupt.

Các chức năng quan trọng:

```text
NVIC
├── Enable IRQ
├── Disable IRQ
├── Set / Clear Pending
├── theo dõi Active
├── quản lý Priority
├── chọn IRQ được phục vụ tiếp theo
└── hỗ trợ Nested Interrupt
```

STM32F10xxx sử dụng:

```text
4 priority bits
→ 16 mức priority lập trình được
```

### Các nhóm thanh ghi NVIC thường gặp

Ở mức kiến trúc Cortex-M:

```text
ISER
→ Interrupt Set-Enable Register

ICER
→ Interrupt Clear-Enable Register

ISPR
→ Interrupt Set-Pending Register

ICPR
→ Interrupt Clear-Pending Register

IABR
→ Interrupt Active Bit Register

IPR
→ Interrupt Priority Register
```

Trong CMSIS thường thao tác thông qua các hàm:

```c
NVIC_EnableIRQ(IRQn);
NVIC_DisableIRQ(IRQn);
NVIC_SetPriority(IRQn, priority);
NVIC_SetPendingIRQ(IRQn);
NVIC_ClearPendingIRQ(IRQn);
```

Khi lập trình STM32, nên hiểu ý nghĩa phần cứng phía dưới thay vì chỉ thuộc tên API.

---

<a id="muc-04-06"></a>
## 4.6. Enable / Disable / Pending / Active

Bốn khái niệm cần phân biệt:

### Enable

```text
IRQ Enabled
→ NVIC cho phép IRQ được đưa tới CPU
```

Ví dụ:

```c
NVIC_EnableIRQ(EXTI0_IRQn);
```

### Disable

```text
IRQ Disabled
→ NVIC không cho IRQ đó được CPU phục vụ
```

Peripheral vẫn có thể tạo request hoặc giữ flag của nó tùy peripheral.

### Pending

```text
Pending
→ interrupt request đã xuất hiện
→ đang chờ được xử lý
```

Một IRQ có thể Pending vì:

- IRQ vừa xuất hiện;
- CPU đang xử lý exception priority cao hơn;
- interrupt tạm thời chưa được phục vụ.

### Active

```text
Active
→ CPU đang thực thi handler của IRQ đó
```

### Quan hệ

Một IRQ có thể:

```text
Enabled nhưng chưa Pending
Pending nhưng chưa Active
Active
```

Ví dụ:

```text
IRQ B đang Active
       ↓
IRQ A xảy ra
       ↓
IRQ A trở thành Pending
       ↓
nếu A đủ priority để preempt B
→ A trở thành Active

nếu không
→ A chờ B kết thúc
```

---

<a id="muc-04-07"></a>
## 4.7. Interrupt Priority

STM32F10xxx triển khai 4 bit priority:

```text
2^4 = 16 mức
```

Có thể biểu diễn:

```text
Priority 0
Priority 1
...
Priority 15
```

Quy tắc quan trọng:

```text
Số priority nhỏ hơn
→ độ ưu tiên cao hơn
```

Ví dụ:

```text
IRQ A: priority = 2
IRQ B: priority = 5

A có priority cao hơn B
```

Không được hiểu theo giá trị số thông thường:

```text
15 không cao hơn 2 về độ ưu tiên
```

### Priority dùng để quyết định gì?

Khi nhiều IRQ cùng yêu cầu xử lý:

```text
NVIC
 ↓
so sánh priority
 ↓
chọn IRQ có priority phù hợp
 ↓
CPU chạy handler
```

Priority còn quyết định interrupt mới có thể preempt handler hiện tại hay phải chờ.

---

<a id="muc-04-08"></a>
## 4.8. Preemption và Nested Interrupt

`Nested Interrupt` nghĩa là một interrupt có thể xảy ra khi CPU đang xử lý interrupt khác.

Ví dụ:

```text
Main
 ↓
IRQ B priority 5
 ↓
ISR_B
   │
   └── IRQ A priority 2 xảy ra
           ↓
        ISR_A
           ↓
        ISR_A kết thúc
           ↓
        quay lại ISR_B
           ↓
        ISR_B kết thúc
           ↓
        quay lại Main
```

Đây là **preemption**:

```text
IRQ priority cao hơn
→ có thể ngắt handler priority thấp hơn
```

Nếu IRQ mới không có đủ priority để preempt:

```text
IRQ mới
→ Pending
→ chờ handler hiện tại kết thúc
```

Nested interrupt giúp hệ thống phản ứng nhanh với sự kiện quan trọng hơn, nhưng làm tăng độ phức tạp của:

- chia sẻ dữ liệu;
- thời gian thực thi ISR;
- Stack usage;
- phân tích timing.

---

<a id="muc-04-09"></a>
## 4.9. Priority Grouping

Priority field của Cortex-M có thể được chia thành hai phần khái niệm:

```text
Priority
├── Preemption Priority
└── Subpriority
```

### Preemption Priority

Quyết định khả năng:

```text
IRQ A
→ có thể preempt IRQ B hay không
```

### Subpriority

Dùng để quyết định thứ tự phục vụ khi các interrupt có cùng mức preemption và cùng ở trạng thái chờ.

Có thể hình dung:

```text
Priority bits
+-------------------+----------------+
| Preemption part   | Subpriority    |
+-------------------+----------------+
```

Cách chia cụ thể được điều khiển bởi `PRIGROUP` của Cortex-M và không nên học cứng một cách chia duy nhất nếu chưa xác định cấu hình hệ thống.

Điểm cần nhớ:

```text
Priority number
≠ chỉ có một ý nghĩa duy nhất
```

khi priority grouping được sử dụng.

Trong hệ thống đơn giản, có thể chọn cấu hình priority sao cho phần preemption là yếu tố chính để dễ phân tích.

---

<a id="muc-04-10"></a>
## 4.10. EXTI là gì?

`EXTI` là:

```text
External Interrupt/Event Controller
```

EXTI nhận các tín hiệu đầu vào từ GPIO hoặc một số nguồn nội bộ và phát hiện cạnh.

Mỗi line có thể cấu hình độc lập:

```text
EXTI Line
├── Interrupt hoặc Event
├── Mask / Unmask
├── Rising Edge
├── Falling Edge
└── Rising + Falling
```

Đối với STM32F10xxx:

```text
Low/Medium/High/XL-density
→ 19 EXTI lines

Connectivity line
→ tối đa 20 EXTI lines
```

EXTI có thể tạo:

```text
Interrupt Request
→ đi tới NVIC
→ chạy ISR

Event Request
→ tạo event
→ không bắt buộc chạy ISR
```

---

<a id="muc-04-11"></a>
## 4.11. EXTI Line và GPIO Mapping

GPIO pin number quyết định EXTI line number.

Ví dụ:

```text
PA0 ─┐
PB0 ─┤
PC0 ─┤
PD0 ─┤
...  ┘
      ↓
    EXTI0
```

Tương tự:

```text
PA1 / PB1 / PC1 / ...
→ EXTI1

PA13 / PB13 / PC13 / ...
→ EXTI13
```

Do đó:

```text
Pin number
→ chọn EXTI line

Port
→ được chọn bằng AFIO_EXTICR
```

### GPIO EXTI lines

```text
EXTI0  → pin 0
EXTI1  → pin 1
...
EXTI15 → pin 15
```

### Internal EXTI lines

Ngoài GPIO:

```text
EXTI16 → PVD output
EXTI17 → RTC Alarm
EXTI18 → USB Wakeup
EXTI19 → Ethernet Wakeup
          (connectivity line)
```

### Một EXTI line chỉ chọn một GPIO port source

Ví dụ EXTI0:

```text
PA0
PB0
PC0
...
```

đều có khả năng nối vào EXTI0, nhưng mapping được AFIO chọn theo một port source cụ thể.

Không nên hiểu:

```text
PA0 và PB0
→ hai EXTI0 độc lập
```

---

<a id="muc-04-12"></a>
## 4.12. Rising Edge / Falling Edge

EXTI phát hiện sự thay đổi mức logic theo cạnh.

### Rising Edge

```text
0 → 1
```

Sơ đồ:

```text
____|‾‾‾‾
    ↑
 Rising
```

Thanh ghi:

```text
EXTI_RTSR
```

### Falling Edge

```text
1 → 0
```

Sơ đồ:

```text
‾‾‾‾|____
    ↓
 Falling
```

Thanh ghi:

```text
EXTI_FTSR
```

### Cả hai cạnh

Có thể bật đồng thời:

```text
RTSR bit = 1
FTSR bit = 1
```

Khi đó:

```text
0 → 1
hoặc
1 → 0
→ đều tạo trigger
```

### Ví dụ Button Active-Low

Giả sử:

```text
Pull-Up
→ không nhấn = 1

nhấn button
→ chân bị kéo xuống 0
```

Sự kiện nhấn:

```text
1 → 0
→ Falling Edge
```

Vì vậy thường chọn:

```text
FTSR = 1
RTSR = 0
```

nếu chỉ muốn phát hiện lúc nhấn.

---

<a id="muc-04-13"></a>
## 4.13. IMR / EMR / RTSR / FTSR / SWIER / PR

Các thanh ghi EXTI chính:

| Thanh ghi | Chức năng |
|---|---|
| `EXTI_IMR` | Interrupt Mask Register |
| `EXTI_EMR` | Event Mask Register |
| `EXTI_RTSR` | Rising Trigger Selection Register |
| `EXTI_FTSR` | Falling Trigger Selection Register |
| `EXTI_SWIER` | Software Interrupt/Event Register |
| `EXTI_PR` | Pending Register |

### EXTI_IMR

```text
IMR bit = 0
→ interrupt line bị mask

IMR bit = 1
→ interrupt line được unmask
```

Ví dụ:

```c
EXTI->IMR |= (1U << 13);
```

cho phép EXTI13 tạo interrupt request.

### EXTI_EMR

```text
EMR
→ điều khiển đường Event
```

Interrupt và Event là hai đường riêng:

```text
IMR
→ Interrupt

EMR
→ Event
```

### EXTI_RTSR

```text
RTSR bit = 1
→ enable Rising Edge
```

Ví dụ:

```c
EXTI->RTSR |= (1U << 0);
```

### EXTI_FTSR

```text
FTSR bit = 1
→ enable Falling Edge
```

Ví dụ:

```c
EXTI->FTSR |= (1U << 13);
```

### EXTI_SWIER

`SWIER` cho phép tạo interrupt/event request bằng software.

Ví dụ khái niệm:

```c
EXTI->SWIER |= (1U << 0);
```

Nếu line được enable qua `IMR`, thao tác này có thể làm `PR` tương ứng được set và tạo interrupt request.

### EXTI_PR

`PR` cho biết line nào đã nhận trigger:

```text
PR bit = 0
→ không có pending trigger

PR bit = 1
→ trigger đã xảy ra
```

Điểm đặc biệt:

```text
EXTI_PR
→ Write 1 to Clear
```

---

<a id="muc-04-14"></a>
## 4.14. AFIO_EXTICR

STM32F1 dùng:

```text
AFIO_EXTICR1
AFIO_EXTICR2
AFIO_EXTICR3
AFIO_EXTICR4
```

để chọn port cho EXTI0–EXTI15.

Trước khi cấu hình các thanh ghi này:

```text
RCC_APB2ENR.AFIOEN = 1
```

### Phân chia EXTICR

```text
EXTICR1
→ EXTI0  → EXTI3

EXTICR2
→ EXTI4  → EXTI7

EXTICR3
→ EXTI8  → EXTI11

EXTICR4
→ EXTI12 → EXTI15
```

Mỗi EXTI line dùng 4 bit chọn port.

Mã port:

```text
0000 → PA
0001 → PB
0010 → PC
0011 → PD
0100 → PE
0101 → PF
0110 → PG
```

### Ví dụ PC13 → EXTI13

Pin number:

```text
13
→ EXTI13
```

Port:

```text
C
→ code 0010
```

EXTI13 nằm trong:

```text
AFIO_EXTICR4
```

Nếu dùng CMSIS struct:

```c
/* EXTICR[3] tương ứng EXTICR4.
   EXTI13 nằm ở field [7:4].
   Port C = 0x2. */
AFIO->EXTICR[3] &= ~(0xFU << 4);
AFIO->EXTICR[3] |=  (0x2U << 4);
```

Sau bước này:

```text
PC13
 ↓
EXTI13
```

---

<a id="muc-04-15"></a>
## 4.15. GPIO → AFIO → EXTI → NVIC

Đây là luồng quan trọng nhất khi dùng GPIO external interrupt:

```text
Tín hiệu bên ngoài
      ↓
GPIO Pin
      ↓
GPIO Input
      ↓
AFIO_EXTICR
chọn Port cho EXTI Line
      ↓
EXTI
├── IMR
├── RTSR
├── FTSR
└── PR
      ↓
IRQ request
      ↓
NVIC
├── Enable
└── Priority
      ↓
CPU
      ↓
IRQHandler
```

Phân biệt nhiệm vụ:

```text
GPIO
→ nhận tín hiệu điện tại chân

AFIO
→ chọn GPIO port nào nối vào EXTI line

EXTI
→ phát hiện cạnh
→ tạo interrupt/event request
→ giữ pending flag

NVIC
→ enable/disable IRQ
→ quản lý priority
→ đưa IRQ tới CPU

ISR
→ xử lý nguyên nhân interrupt
```

### Ví dụ PC13

```text
Button
 ↓
PC13
 ↓
AFIO_EXTICR4
Port C → EXTI13
 ↓
EXTI13 Falling Edge
 ↓
EXTI15_10_IRQn
 ↓
NVIC
 ↓
EXTI15_10_IRQHandler()
```

---

<a id="muc-04-16"></a>
## 4.16. Shared IRQ: EXTI5_9 và EXTI10_15

EXTI0–EXTI4 có IRQ riêng:

```text
EXTI0  → EXTI0_IRQn
EXTI1  → EXTI1_IRQn
EXTI2  → EXTI2_IRQn
EXTI3  → EXTI3_IRQn
EXTI4  → EXTI4_IRQn
```

Các line phía trên được nhóm:

```text
EXTI5 → EXTI9
→ EXTI9_5_IRQn
→ EXTI9_5_IRQHandler()

EXTI10 → EXTI15
→ EXTI15_10_IRQn
→ EXTI15_10_IRQHandler()
```

Vì nhiều line dùng chung một handler, ISR phải kiểm tra `EXTI_PR`.

Ví dụ:

```c
void EXTI15_10_IRQHandler(void)
{
    if (EXTI->PR & (1U << 13))
    {
        EXTI->PR = (1U << 13);

        /* xử lý EXTI13 */
    }

    if (EXTI->PR & (1U << 14))
    {
        EXTI->PR = (1U << 14);

        /* xử lý EXTI14 */
    }
}
```

Không nên giả định:

```text
Handler chạy
→ chắc chắn chỉ một line cụ thể gây ra
```

---

<a id="muc-04-17"></a>
## 4.17. Clear Pending Flag

`EXTI_PR` không phải thanh ghi read/write thông thường.

Cơ chế:

```text
PR bit = 1
→ line đang pending

ghi 1 vào bit đó
→ clear pending bit
```

Ví dụ clear EXTI13:

```c
EXTI->PR = (1U << 13);
```

Không nên viết:

```c
EXTI->PR &= ~(1U << 13);
```

vì đây là tư duy read-modify-write của thanh ghi thông thường, không phù hợp với semantics **write 1 to clear**.

### Vì sao phải Clear Flag?

Luồng:

```text
Edge xảy ra
 ↓
EXTI_PR bit được set
 ↓
IRQ request
 ↓
ISR chạy
 ↓
phải acknowledge / clear nguồn interrupt
```

Nếu flag của nguồn interrupt không được xử lý đúng, interrupt có thể tiếp tục giữ trạng thái request hoặc lại làm handler được kích hoạt.

### Peripheral Pending và NVIC Pending khác nhau

Hai tầng:

```text
EXTI / Peripheral
      ↓
interrupt request
      ↓
NVIC
```

Do đó:

```text
EXTI_PR
→ pending flag ở EXTI

NVIC Pending
→ pending state ở NVIC
```

Không nên coi hai khái niệm là cùng một thanh ghi hay cùng một trạng thái.

Khi ISR xử lý EXTI, thao tác bắt buộc thường là xử lý `EXTI_PR`, không phải chỉ clear NVIC pending.

---

<a id="muc-04-18"></a>
## 4.18. Quy trình cấu hình EXTI Interrupt

Ví dụ mục tiêu:

```text
PC13
→ Input Pull-Up
→ Falling Edge
→ EXTI13
→ EXTI15_10_IRQn
```

Quy trình:

```text
1. Bật GPIOC clock
        ↓
2. Bật AFIO clock
        ↓
3. Cấu hình PC13 Input Pull-Up
        ↓
4. AFIO_EXTICR4
   chọn Port C cho EXTI13
        ↓
5. EXTI_IMR
   unmask Line 13
        ↓
6. EXTI_FTSR
   enable Falling Edge
        ↓
7. EXTI_RTSR
   disable Rising Edge nếu không dùng
        ↓
8. Clear PR13 cũ
        ↓
9. Đặt NVIC priority
        ↓
10. NVIC Enable EXTI15_10_IRQn
        ↓
11. Viết EXTI15_10_IRQHandler()
        ↓
12. Trong ISR:
    kiểm tra PR13
        ↓
13. Clear PR13 bằng cách ghi 1
        ↓
14. Xử lý sự kiện
```

### Bước 1 — Bật Clock

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPCEN
               | RCC_APB2ENR_AFIOEN;
```

### Bước 2 — PC13 Input Pull-Up

PC13 nằm trong `GPIOC_CRH`.

Shift:

```text
(13 - 8) × 4
= 20
```

Cấu hình:

```text
MODE = 00
CNF  = 10
→ Input Pull-Up / Pull-Down
```

Code:

```c
GPIOC->CRH &= ~(0xFU << 20);
GPIOC->CRH |=  (0x8U << 20);

/* Chọn Pull-Up */
GPIOC->ODR |= (1U << 13);
```

### Bước 3 — Mapping PC13 vào EXTI13

```c
AFIO->EXTICR[3] &= ~(0xFU << 4);
AFIO->EXTICR[3] |=  (0x2U << 4);
```

### Bước 4 — Cấu hình EXTI13

Unmask interrupt:

```c
EXTI->IMR |= (1U << 13);
```

Falling Edge:

```c
EXTI->FTSR |= (1U << 13);
```

Không dùng Rising Edge:

```c
EXTI->RTSR &= ~(1U << 13);
```

Clear pending cũ:

```c
EXTI->PR = (1U << 13);
```

### Bước 5 — NVIC

Ví dụ:

```c
NVIC_SetPriority(EXTI15_10_IRQn, 5);
NVIC_EnableIRQ(EXTI15_10_IRQn);
```

Giá trị priority cụ thể phải được chọn theo thiết kế priority của toàn hệ thống.

### Bước 6 — ISR

```c
void EXTI15_10_IRQHandler(void)
{
    if (EXTI->PR & (1U << 13))
    {
        EXTI->PR = (1U << 13);

        /* xử lý sự kiện */
    }
}
```

---

<a id="muc-04-19"></a>
## 4.19. Quy ước thiết kế ISR

Các nguyên tắc sau là quy ước thiết kế phổ biến để giảm latency và làm hệ thống dễ phân tích hơn.

ISR nên:

```text
ISR
├── xác định đúng nguồn interrupt
├── clear/acknowledge flag đúng cách
├── lấy dữ liệu cần thiết
├── cập nhật trạng thái ngắn gọn
└── thoát sớm
```

Ví dụ:

```c
volatile uint8_t button_pressed = 0;

void EXTI15_10_IRQHandler(void)
{
    if (EXTI->PR & (1U << 13))
    {
        EXTI->PR = (1U << 13);
        button_pressed = 1;
    }
}
```

Main loop xử lý công việc dài:

```c
while (1)
{
    if (button_pressed)
    {
        button_pressed = 0;

        /* xử lý dài hơn */
    }
}
```

Nên tránh trong ISR:

```text
delay dài
busy-wait lâu
vòng lặp không có giới hạn
xử lý thuật toán nặng
I/O blocking kéo dài
```

Lý do:

```text
ISR chạy lâu
→ tăng interrupt latency
→ IRQ priority thấp hơn phải chờ lâu hơn
→ tăng Stack usage khi có nested interrupt
```

### Dữ liệu chia sẻ giữa ISR và Main

Nếu một biến được thay đổi trong ISR và đọc ở main:

```c
volatile uint8_t event_flag;
```

`volatile` giúp compiler thực hiện các truy cập bộ nhớ như đã viết, nhưng không tự biến mọi thao tác nhiều bước thành atomic và không thay thế cơ chế đồng bộ khi dữ liệu phức tạp.

---

<a id="muc-04-20"></a>
## 4.20. Ví dụ Button → EXTI → ISR

Giả sử:

```text
Button
→ PC13

PC13
→ Input Pull-Up

Button nhấn
→ kéo pin xuống GND

Trigger
→ Falling Edge
```

### Cấu hình

```c
static void Button_EXTI_Init(void)
{
    /* 1. Clock cho GPIOC và AFIO */
    RCC->APB2ENR |= RCC_APB2ENR_IOPCEN
                   | RCC_APB2ENR_AFIOEN;

    /* 2. PC13 Input Pull-Up
       MODE=00, CNF=10 */
    GPIOC->CRH &= ~(0xFU << 20);
    GPIOC->CRH |=  (0x8U << 20);
    GPIOC->ODR |=  (1U << 13);

    /* 3. PC13 → EXTI13
       EXTICR4, Port C = 0010 */
    AFIO->EXTICR[3] &= ~(0xFU << 4);
    AFIO->EXTICR[3] |=  (0x2U << 4);

    /* 4. EXTI13 */
    EXTI->IMR  |=  (1U << 13);
    EXTI->RTSR &= ~(1U << 13);
    EXTI->FTSR |=  (1U << 13);

    /* Clear pending cũ */
    EXTI->PR = (1U << 13);

    /* 5. NVIC */
    NVIC_SetPriority(EXTI15_10_IRQn, 5);
    NVIC_EnableIRQ(EXTI15_10_IRQn);
}
```

### Handler

```c
volatile uint8_t button_pressed = 0;

void EXTI15_10_IRQHandler(void)
{
    if (EXTI->PR & (1U << 13))
    {
        EXTI->PR = (1U << 13);
        button_pressed = 1;
    }
}
```

### Main

```c
int main(void)
{
    Button_EXTI_Init();

    while (1)
    {
        if (button_pressed)
        {
            button_pressed = 0;

            /* xử lý sự kiện button */
        }
    }
}
```

### Luồng hoàn chỉnh

```text
Button chưa nhấn
→ PC13 = 1

nhấn Button
→ PC13: 1 → 0
→ Falling Edge

AFIO
→ PC13 được nối vào EXTI13

EXTI13
→ FTSR phát hiện cạnh
→ PR13 = 1
→ IMR13 cho phép interrupt request

NVIC
→ EXTI15_10_IRQn Pending
→ CPU nhận IRQ

CPU
→ Handler mode
→ EXTI15_10_IRQHandler()

ISR
→ kiểm tra PR13
→ ghi 1 để clear PR13
→ set button_pressed

Exception Return
→ quay lại main
```

### Button Bounce

Button cơ khí có thể tạo nhiều chuyển mức trong một lần nhấn:

```text
1 ─────┐
       └─┐ ┌─┐
         └─┘ └──── 0
```

Do đó một lần nhấn có thể tạo nhiều EXTI trigger.

Có thể xử lý bằng:

```text
software debounce
Timer debounce
lọc phần cứng
```

Không nên giải quyết debounce bằng một delay dài ngay trong ISR nếu có thể tránh.

---

### Interrupt từ Peripheral khác

EXTI chỉ là một loại interrupt source.

Mẫu tổng quát cho peripheral interrupt:

```text
Peripheral
   ↓
enable interrupt source trong peripheral
   ↓
event xảy ra
   ↓
status flag được set
   ↓
IRQ request
   ↓
NVIC
   ↓
ISR
   ↓
kiểm tra flag
   ↓
clear / service flag đúng cơ chế
```

Ví dụ:

```text
Timer Update Event
→ TIM status flag
→ TIMx_IRQn
→ TIMx_IRQHandler()

USART RX event
→ USART status
→ USARTx_IRQn
→ USARTx_IRQHandler()

DMA Transfer Complete
→ DMA flag
→ DMAx_Channely_IRQn
→ handler tương ứng
```

Điểm chung:

```text
Peripheral flag
và
NVIC state
là hai tầng khác nhau
```

---

### Interrupt và Event trong Low-Power

EXTI có thể tạo:

```text
Interrupt
→ đi qua NVIC
→ handler

Event
→ event path
→ không nhất thiết chạy handler
```

Khi dùng low-power:

```text
WFI
→ thường thức dậy bởi interrupt

WFE
→ thức dậy bởi event
```

Chi tiết low-power không cần để cấu hình EXTI cơ bản, nhưng cần phân biệt:

```text
IMR
→ Interrupt path

EMR
→ Event path
```

---

<a id="muc-04-21"></a>
## 4.21. Câu hỏi tự kiểm tra

1. Polling là gì?
2. Interrupt khác Polling như thế nào?
3. Exception là gì trên Cortex-M?
4. External Interrupt có phải là Exception không?
5. Reset, NMI và HardFault thuộc nhóm nào?
6. Vector Table dùng để làm gì?
7. ISR là gì?
8. IRQ được nối tới ISR thông qua cơ chế nào?
9. Khi interrupt được chấp nhận, CPU chuyển sang mode nào?
10. Hardware stacking lưu những register cơ bản nào?
11. NVIC viết tắt của gì?
12. NVIC có những chức năng chính nào?
13. `Enabled` khác `Pending` như thế nào?
14. `Pending` khác `Active` như thế nào?
15. STM32F10xxx sử dụng bao nhiêu bit interrupt priority?
16. Có bao nhiêu mức priority lập trình được?
17. Priority số `2` và `5`, mức nào cao hơn?
18. Nested Interrupt là gì?
19. Preemption là gì?
20. Khi IRQ mới không thể preempt handler hiện tại thì điều gì xảy ra?
21. Preemption Priority khác Subpriority ở điểm nào?
22. Priority Grouping dùng để làm gì?
23. EXTI viết tắt của gì?
24. EXTI tạo ra hai loại request nào?
25. `EXTI_IMR` dùng để làm gì?
26. `EXTI_EMR` dùng để làm gì?
27. `EXTI_RTSR` dùng để làm gì?
28. `EXTI_FTSR` dùng để làm gì?
29. `EXTI_SWIER` dùng để làm gì?
30. `EXTI_PR` dùng để làm gì?
31. EXTI0 có thể nhận tín hiệu từ những pin nào về mặt số pin?
32. Vì sao PA0 và PB0 không phải hai EXTI line độc lập?
33. PC13 phải map vào EXTI line nào?
34. Port cho EXTI line được chọn bằng thanh ghi nào?
35. Trước khi cấu hình AFIO_EXTICR cần bật clock nào?
36. `AFIO_EXTICR1` điều khiển những EXTI line nào?
37. `AFIO_EXTICR4` điều khiển những EXTI line nào?
38. EXTI16, EXTI17, EXTI18 nối với những nguồn nào?
39. EXTI19 có ở nhóm STM32F10xxx nào?
40. Rising Edge là chuyển mức gì?
41. Falling Edge là chuyển mức gì?
42. Button Active-Low với Pull-Up thường dùng cạnh nào để phát hiện nhấn?
43. EXTI0–EXTI4 có IRQ như thế nào?
44. EXTI5–EXTI9 dùng chung IRQ nào?
45. EXTI10–EXTI15 dùng chung IRQ nào?
46. Vì sao shared IRQ handler phải kiểm tra `EXTI_PR`?
47. Clear pending bit của `EXTI_PR` bằng cách nào?
48. Vì sao không nên dùng `EXTI->PR &= ~bit` để clear?
49. EXTI pending flag và NVIC pending state khác nhau thế nào?
50. Hãy mô tả luồng `GPIO → AFIO → EXTI → NVIC → ISR`.
51. Hãy mô tả toàn bộ quy trình cấu hình PC13 Falling Edge Interrupt.
52. Vì sao nên clear pending cũ trước khi enable NVIC?
53. ISR nên thực hiện những công việc nào?
54. Vì sao không nên delay dài trong ISR?
55. `volatile` có vai trò gì với flag chia sẻ giữa ISR và main?
56. `volatile` có tự làm mọi thao tác trở thành atomic không?
57. Button bounce là gì?
58. EXTI Event khác EXTI Interrupt như thế nào?
59. WFI và WFE liên quan khác nhau thế nào với Interrupt/Event?
60. Với Timer/UART/DMA interrupt, vì sao vẫn phải xử lý flag ở peripheral?

---

## 4.22. Tóm tắt

Luồng interrupt tổng quát:

```text
Event
 ↓
Peripheral / EXTI
 ↓
Interrupt Request
 ↓
NVIC
 ↓
Vector Table
 ↓
ISR
 ↓
clear / service source flag
 ↓
Exception Return
```

NVIC:

```text
NVIC
├── Enable / Disable
├── Pending
├── Active
├── Priority
└── Nested / Preemption
```

Priority:

```text
Số nhỏ hơn
→ Priority cao hơn
```

EXTI:

```text
IMR
→ Interrupt Mask

EMR
→ Event Mask

RTSR
→ Rising Edge

FTSR
→ Falling Edge

SWIER
→ Software trigger

PR
→ Pending
→ Write 1 to Clear
```

GPIO external interrupt:

```text
GPIO Pin
   ↓
AFIO_EXTICR
   ↓
EXTI Line
   ↓
Trigger
   ↓
IMR
   ↓
NVIC
   ↓
IRQHandler
```

Shared IRQ:

```text
EXTI0 → EXTI0_IRQn
EXTI1 → EXTI1_IRQn
EXTI2 → EXTI2_IRQn
EXTI3 → EXTI3_IRQn
EXTI4 → EXTI4_IRQn

EXTI5–9
→ EXTI9_5_IRQn

EXTI10–15
→ EXTI15_10_IRQn
```

**Điểm cần nhớ:**

> **EXTI phát hiện cạnh và tạo interrupt request; AFIO chọn GPIO port được nối vào từng EXTI line; NVIC quản lý enable, pending và priority; ISR phải xác định đúng nguồn interrupt và clear flag theo cơ chế của peripheral. Với `EXTI_PR`, pending bit được clear bằng cách ghi `1` vào chính bit đó.**

[↑ Về mục lục](#muc-luc)


---

<a id="chuong-05"></a>
# 5. Timer + PWM

Timer là một khối phần cứng đếm theo clock. Từ bộ đếm này, STM32 có thể tạo time base, interrupt định kỳ, đo tín hiệu đầu vào, tạo Output Compare và phát PWM.

Luồng nền tảng:

```text
TIMxCLK
   ↓
Prescaler
   ↓
Counter
   ↓
Auto-Reload
   ↓
Update Event
   ↓
Capture / Compare
   ↓
PWM / Input Capture / Output Compare
```

Các khối Timer liên hệ trực tiếp với:

```text
RCC
→ cấp Timer Clock

GPIO Alternate Function
→ đưa tín hiệu Timer ra/vào chân

NVIC
→ xử lý Timer Interrupt

DMA
→ truyền dữ liệu theo Timer event
```

<a id="muc-05-01"></a>
## 5.1. Timer là gì?

Timer có thể xem như một bộ đếm phần cứng:

```text
Clock
  ↓
Counter
  ↓
0, 1, 2, 3, ...
```

Thay vì CPU tự tăng một biến bằng software:

```c
counter++;
```

Timer tự đếm bằng phần cứng khi được cấp clock và enable.

Timer thường được dùng cho:

```text
Time base
Delay / periodic event
Periodic interrupt
PWM generation
Output Compare
Input Capture
Frequency measurement
Pulse-width measurement
Encoder interface
Trigger cho ADC / DAC
DMA request
```

Ba khối cơ bản nhất:

```text
PSC
→ chia Timer Clock

CNT
→ giá trị bộ đếm hiện tại

ARR
→ giới hạn chu kỳ đếm
```

Các channel Capture/Compare thêm:

```text
CCR1
CCR2
CCR3
CCR4
```

---

<a id="muc-05-02"></a>
## 5.2. Các loại Timer trên STM32F1

STM32F1 có nhiều loại Timer với khả năng khác nhau. Peripheral thực tế phụ thuộc từng mã MCU và density.

### Advanced-Control Timer

Các Timer tiêu biểu:

```text
TIM1
TIM8
```

Khả năng:

```text
16-bit counter
Prescaler
Input Capture
Output Compare
PWM
Complementary PWM
Dead-Time
Break input
Repetition Counter
Encoder
Interrupt / DMA
```

TIM1/TIM8 phù hợp với các ứng dụng điều khiển công suất và motor cần nhiều cơ chế bảo vệ phần cứng.

### General-Purpose Timer

Nhóm chính:

```text
TIM2
TIM3
TIM4
TIM5
```

Các Timer này có:

```text
16-bit up/down/up-down counter
16-bit prescaler
tối đa 4 Capture/Compare channel
Input Capture
Output Compare
PWM Edge-Aligned
PWM Center-Aligned
One-Pulse
Encoder interface
Interrupt / DMA
Timer synchronization
```

Đây là nhóm phù hợp nhất để học các khái niệm Timer cơ bản.

Một số STM32F1 còn có:

```text
TIM9
TIM10
TIM11
TIM12
TIM13
TIM14
```

với số channel và tính năng phụ thuộc từng Timer.

### Basic Timer

```text
TIM6
TIM7
```

Basic Timer có cấu trúc đơn giản hơn:

```text
16-bit Upcounter
16-bit Prescaler
Auto-Reload
Update Event
Interrupt / DMA
Trigger cho DAC
```

Basic Timer không có các Capture/Compare channel như TIM2–TIM5.

TIM6/TIM7 chỉ có trên một số nhóm STM32F1 như high-density, XL-density và connectivity line.

### So sánh khái quát

| Nhóm | Counter | Capture/Compare | PWM | Complementary PWM / Dead-Time |
|---|---|---|---|---|
| TIM1/TIM8 | Có | Có | Có | Có |
| TIM2–TIM5 | Có | Có | Có | Không như Advanced Timer |
| TIM6/TIM7 | Có | Không | Không | Không |

---

<a id="muc-05-03"></a>
## 5.3. Timer Clock

Trước mọi phép tính Timer, phải xác định:

```text
Timer nằm trên APB nào?
        ↓
PCLK của bus đó là bao nhiêu?
        ↓
APB Prescaler = /1 hay khác /1?
        ↓
TIMxCLK bằng bao nhiêu?
```

Quy tắc Timer Clock của STM32F1:

```text
Nếu APB Prescaler = /1:
    TIMxCLK = PCLKx

Nếu APB Prescaler != /1:
    TIMxCLK = 2 × PCLKx
```

Ví dụ hệ thống:

```text
HCLK  = 72 MHz
PCLK1 = 36 MHz
PPRE1 = /2
```

TIM2 nằm trên APB1:

```text
PPRE1 != /1

TIM2CLK
= 2 × PCLK1
= 2 × 36 MHz
= 72 MHz
```

Nếu:

```text
PCLK2 = 72 MHz
PPRE2 = /1
```

thì TIM1:

```text
TIM1CLK
= PCLK2
= 72 MHz
```

### Sai lầm thường gặp

Không được mặc định:

```text
Timer Clock = PCLK
```

Mà phải kiểm tra APB Prescaler.

---

<a id="muc-05-04"></a>
## 5.4. Counter: CNT

Thanh ghi:

```text
TIMx_CNT
```

chứa giá trị counter hiện tại.

Ở Upcounting:

```text
0
1
2
3
...
ARR
0
1
...
```

Khi Timer được enable bằng:

```text
TIMx_CR1.CEN = 1
```

counter bắt đầu chạy theo `CK_CNT`.

Có thể đọc counter:

```c
uint16_t value = TIM2->CNT;
```

hoặc ghi giá trị mới:

```c
TIM2->CNT = 0;
```

`CNT` là nền tảng của:

```text
Time base
Input Capture
Output Compare
PWM
Encoder
```

---

<a id="muc-05-05"></a>
## 5.5. Prescaler: PSC

Thanh ghi:

```text
TIMx_PSC
```

chia Timer Clock trước khi clock đi vào counter.

Sơ đồ:

```text
TIMxCLK
   ↓
  PSC
   ↓
CK_CNT
   ↓
  CNT
```

Công thức:

```text
fCNT =
fTIM
─────────
PSC + 1
```

Điểm quan trọng:

```text
Hệ số chia thực tế = PSC + 1
```

Ví dụ:

```text
TIMxCLK = 72 MHz
PSC     = 71
```

Ta có:

```text
fCNT
= 72 MHz / (71 + 1)
= 1 MHz
```

Suy ra:

```text
1 Timer tick = 1 µs
```

### Khoảng chia

PSC là thanh ghi 16-bit:

```text
PSC = 0
→ chia 1

PSC = 1
→ chia 2

...

PSC = 65535
→ chia 65536
```

### PSC được Buffer

Giá trị `PSC` mới không nhất thiết tác động ngay lập tức.

Luồng:

```text
Software ghi PSC
      ↓
PSC preload / buffer
      ↓
Update Event
      ↓
prescaler mới có hiệu lực
```

Nếu muốn nạp cấu hình trước khi bắt đầu Timer, thường tạo Update Event bằng:

```text
TIMx_EGR.UG = 1
```

---

<a id="muc-05-06"></a>
## 5.6. Auto-Reload Register: ARR

Thanh ghi:

```text
TIMx_ARR
```

quy định giới hạn chu kỳ đếm.

Trong Upcounting:

```text
CNT = 0
  ↓
1
  ↓
2
  ↓
...
  ↓
ARR
  ↓
overflow
  ↓
CNT = 0
```

Counter đi qua:

```text
0 → ARR
```

nên số tick trong một chu kỳ Edge-Aligned Upcounting là:

```text
ARR + 1
```

Ví dụ:

```text
ARR = 999
```

counter chạy:

```text
0 → 999
```

tức:

```text
1000 Timer ticks
```

### ARR Preload

`ARR` có cơ chế preload/shadow register.

Bit:

```text
TIMx_CR1.ARPE
```

quyết định cách giá trị ARR mới được áp dụng.

Khi preload được sử dụng:

```text
Software ghi ARR
      ↓
ARR preload register
      ↓
Update Event
      ↓
ARR active/shadow register
```

Cơ chế này rất hữu ích khi thay đổi period trong lúc Timer đang chạy.

---

<a id="muc-05-07"></a>
## 5.7. Up / Down / Center-Aligned Counting

General-Purpose và Advanced Timer có thể hỗ trợ nhiều hướng đếm.

### Upcounting

```text
0 → 1 → 2 → ... → ARR
                      ↓
                      0
```

Đây là mode đơn giản nhất.

Trong `TIMx_CR1`:

```text
DIR = 0
```

khi dùng Edge-Aligned Upcounting.

### Downcounting

```text
ARR → ARR-1 → ... → 1 → 0
                         ↓
                        ARR
```

Trong Edge-Aligned Downcounting:

```text
DIR = 1
```

### Center-Aligned

Counter chạy lên rồi chạy xuống:

```text
0 → 1 → 2 → ... → ARR-1
                       ↓
                      ARR
                       ↓
ARR-1 ← ... ← 2 ← 1
                       ↓
                       0
```

Trong Center-Aligned mode:

```text
CMS != 00
```

Direction được phần cứng cập nhật để phản ánh hướng đếm hiện tại.

Center-Aligned hữu ích khi tạo PWM đối xứng.

### Edge-Aligned và Center-Aligned

```text
Edge-Aligned
→ chu kỳ bắt đầu tại cùng một biên counter

Center-Aligned
→ counter chạy lên rồi xuống
→ waveform được căn đối xứng quanh tâm chu kỳ
```

Với cùng `PSC` và `ARR`, Center-Aligned có chu kỳ đếm dài hơn Edge-Aligned.

---

<a id="muc-05-08"></a>
## 5.8. Update Event và Update Interrupt

`Update Event` thường được viết tắt:

```text
UEV
```

UEV có thể xảy ra từ:

```text
Counter overflow
Counter underflow
Software đặt UG
Một số cơ chế trigger/synchronization
```

### Khi UEV xảy ra

Tùy cấu hình, UEV có thể:

```text
Reload Prescaler
Transfer ARR preload → active
Transfer CCR preload → active
Set Update Interrupt Flag
Generate Interrupt
Generate DMA request
```

Các bit/register quan trọng:

```text
TIMx_SR.UIF
→ Update Interrupt Flag

TIMx_DIER.UIE
→ Update Interrupt Enable

TIMx_EGR.UG
→ tạo Update Event bằng software
```

### Update Interrupt

Luồng:

```text
CNT overflow / underflow
        ↓
Update Event
        ↓
UIF = 1
        ↓
UIE = 1?
        ↓
Timer IRQ
        ↓
NVIC
        ↓
TIMx_IRQHandler()
```

Ví dụ khái niệm:

```c
void TIM2_IRQHandler(void)
{
    if (TIM2->SR & TIM_SR_UIF)
    {
        TIM2->SR &= ~TIM_SR_UIF;

        /* xử lý Update Event */
    }
}
```

Phải phân biệt:

```text
TIMx_SR.UIF
→ flag của Timer

NVIC pending
→ trạng thái IRQ tại NVIC
```

---

<a id="muc-05-09"></a>
## 5.9. Công thức tính Timer Period / Frequency

Với Edge-Aligned Upcounting:

```text
fCNT =
fTIM
─────────
PSC + 1
```

Số tick một chu kỳ:

```text
ARR + 1
```

Do đó:

```text
fUPDATE =
fTIM
──────────────────────
(PSC + 1)(ARR + 1)
```

Period:

```text
TUPDATE =
(PSC + 1)(ARR + 1)
──────────────────────
fTIM
```

Nếu Timer được dùng để tạo PWM Edge-Aligned:

```text
fPWM = fUPDATE
```

### Ví dụ 1 ms

Giả sử:

```text
TIM2CLK = 72 MHz
PSC     = 71
ARR     = 999
```

Bước 1:

```text
fCNT
= 72 MHz / 72
= 1 MHz
```

Bước 2:

```text
Timer tick
= 1 / 1 MHz
= 1 µs
```

Bước 3:

```text
Period
= 1000 × 1 µs
= 1 ms
```

Tần số:

```text
f = 1 kHz
```

### Center-Aligned

Trong Center-Aligned mode, một chu kỳ đầy đủ gồm một lượt đếm lên và một lượt đếm xuống.

Với `ARR > 0`:

```text
số Timer tick / chu kỳ
= 2 × ARR
```

Nên:

```text
fCENTER =
fCNT
────────
2 × ARR
```

Công thức Edge-Aligned `(ARR + 1)` không được áp dụng nguyên xi cho Center-Aligned.

---

<a id="muc-05-10"></a>
## 5.10. Capture/Compare Channel

General-Purpose Timer có thể có tối đa bốn channel:

```text
CH1
CH2
CH3
CH4
```

Mỗi channel có một Capture/Compare Register:

```text
TIMx_CCR1
TIMx_CCR2
TIMx_CCR3
TIMx_CCR4
```

`CCR` có vai trò khác nhau tùy mode:

```text
Input Capture
→ CNT được chụp vào CCR

Output Compare
→ CNT được so sánh với CCR
```

Đây là quan hệ cốt lõi:

```text
Input Capture:
CNT → CCR

Output Compare:
CNT ↔ CCR
```

Các channel còn có thể dùng cho:

```text
PWM
One-Pulse
PWM Input
```

---

<a id="muc-05-11"></a>
## 5.11. Output Compare

Output Compare liên tục so sánh:

```text
TIMx_CNT
với
TIMx_CCRx
```

Khi:

```text
CNT == CCRx
```

xảy ra Compare Match.

Tại Compare Match, Timer có thể:

```text
giữ output
set output active
set output inactive
toggle output
set CCxIF
generate interrupt
generate DMA request
```

Các thanh ghi liên quan:

```text
TIMx_CCMR1 / CCMR2
→ chọn Output Compare mode

TIMx_CCER
→ enable output + polarity

TIMx_CCRx
→ compare value

TIMx_SR
→ Compare flag

TIMx_DIER
→ Compare interrupt / DMA enable
```

### Ví dụ Toggle

Giả sử:

```text
CCR1 = 500
```

Khi:

```text
CNT == 500
```

channel có thể được cấu hình để:

```text
OC1 toggle
```

Output Compare có thể dùng để:

```text
tạo waveform
tạo timestamp event
tạo interrupt tại thời điểm xác định
One-Pulse
```

PWM là một chế độ chuyên biệt dựa trên cơ chế Compare.

---

<a id="muc-05-12"></a>
## 5.12. Input Capture

Input Capture dùng cạnh tín hiệu đầu vào để chụp giá trị counter.

Luồng:

```text
External signal
      ↓
Rising / Falling Edge
      ↓
Timer Input
      ↓
Capture Event
      ↓
CNT hiện tại
      ↓
CCRx
```

Ví dụ:

```text
CNT = 1250
Rising Edge xảy ra
→ CCR1 = 1250
```

### Đo Period

Hai capture liên tiếp:

```text
Capture 1 = 1000
Capture 2 = 3000
```

Hiệu:

```text
ΔCNT = 3000 - 1000
     = 2000 ticks
```

Nếu:

```text
fCNT = 1 MHz
```

thì:

```text
1 tick = 1 µs
```

Period:

```text
T = 2000 µs
  = 2 ms
```

Frequency:

```text
f = 1 / 2 ms
  = 500 Hz
```

### Input Capture có thể cấu hình thêm

```text
Input polarity
Digital filter
Input prescaler
Capture interrupt
Capture DMA
```

Input Capture thường dùng để đo:

```text
Frequency
Period
Pulse width
PWM input
```

---

<a id="muc-05-13"></a>
## 5.13. PWM là gì?

`PWM`:

```text
Pulse Width Modulation
```

PWM là tín hiệu số tuần hoàn được mô tả bởi hai thông số chính:

```text
Frequency
Duty Cycle
```

Ví dụ Duty 25%:

```text
HIGH ─────┐
          │
LOW       └───────────────

      25%       75%
```

Duty 50%:

```text
HIGH ─────────┐
              │
LOW           └───────────

        50%        50%
```

### Frequency

Frequency cho biết:

```text
bao nhiêu chu kỳ PWM / giây
```

### Duty Cycle

Duty Cycle là tỷ lệ thời gian tín hiệu ở trạng thái active trong một chu kỳ.

```text
Duty =
Tactive
─────── × 100%
Tperiod
```

PWM được dùng trong:

```text
LED brightness
DC motor speed
Servo control
Power conversion
Audio đơn giản
Tạo tín hiệu điều khiển theo tỷ lệ
```

---

<a id="muc-05-14"></a>
## 5.14. PWM Frequency và Duty Cycle

Với PWM Edge-Aligned, Upcounting:

```text
fPWM =
fTIM
──────────────────────
(PSC + 1)(ARR + 1)
```

Ba tham số:

```text
PSC
→ chia Timer Clock

ARR
→ quyết định PWM period / frequency

CCR
→ quyết định thời điểm compare
→ từ đó quyết định duty
```

### PWM Mode 1, Active-High, Upcounting

Quan hệ cơ bản:

```text
CNT < CCRx
→ OCxREF active

CNT >= CCRx
→ OCxREF inactive
```

Với cấu hình này:

```text
Duty ≈
CCRx
─────── × 100%
ARR + 1
```

Ví dụ:

```text
ARR  = 999
CCR1 = 250
```

Ta có:

```text
Duty
= 250 / 1000 × 100%
= 25%
```

### Edge Cases

Nếu:

```text
CCR = 0
```

thì PWM Mode 1 Upcounting active-high cho:

```text
0% duty
```

Nếu compare value lớn hơn ARR:

```text
CCR > ARR
```

thì `OCxREF` được giữ active trong PWM Mode 1 Upcounting.

Cần lưu ý giới hạn độ rộng thanh ghi khi tạo trường hợp 100%.

---

<a id="muc-05-15"></a>
## 5.15. PWM Mode 1 / PWM Mode 2

PWM được chọn qua:

```text
OCxM
```

trong `TIMx_CCMRx`.

### PWM Mode 1

Mã mode:

```text
OCxM = 110
```

Với Upcounting:

```text
CNT < CCRx
→ OCxREF active

CNT >= CCRx
→ OCxREF inactive
```

### PWM Mode 2

Mã mode:

```text
OCxM = 111
```

PWM Mode 2 tạo quan hệ logic ngược với PWM Mode 1 đối với `OCxREF`.

Có thể nhớ:

```text
PWM Mode 1
→ active trước Compare Match

PWM Mode 2
→ active theo quan hệ ngược lại
```

### Output Polarity

Waveform thực tế ở chân còn phụ thuộc:

```text
CCxP
→ Output Polarity
```

Do đó:

```text
PWM Mode
+
Output Polarity
→ mức logic thực tế trên OCx pin
```

Không được suy luận duty ở chân chỉ từ `CCR` nếu polarity đã bị đảo.

---

<a id="muc-05-16"></a>
## 5.16. CCRx và Compare Match

`TIMx_CCRx` chứa Capture hoặc Compare value tùy channel mode.

Trong PWM:

```text
CNT
 ↓
so sánh
 ↓
CCRx
```

Compare point quyết định vị trí chuyển trạng thái của `OCxREF`.

Ví dụ:

```text
ARR = 999
```

Duty 10%:

```text
CCR = 100
```

Duty 50%:

```text
CCR = 500
```

Duty 90%:

```text
CCR = 900
```

Trong PWM Mode 1, Upcounting, Active-High:

```text
CCR tăng
→ thời gian active tăng
→ Duty tăng
```

### Thay Duty ở Runtime

Có thể cập nhật:

```c
TIM2->CCR1 = new_value;
```

Nếu `OCxPE` được enable:

```text
giá trị mới
→ preload
→ chờ Update Event
→ mới áp dụng đồng bộ
```

Điều này giúp tránh thay compare value giữa chu kỳ theo cách không mong muốn.

---

<a id="muc-05-17"></a>
## 5.17. Preload: ARPE / OCxPE

Timer sử dụng shadow/preload register để cập nhật period và duty tại thời điểm xác định.

### ARR Preload

Bit:

```text
TIMx_CR1.ARPE
```

Khi ARPE được bật:

```text
Software ghi ARR
      ↓
ARR preload
      ↓
Update Event
      ↓
ARR active
```

### CCR Preload

Bit:

```text
OCxPE
```

nằm trong `TIMx_CCMRx`.

Khi enable:

```text
Software ghi CCRx
      ↓
CCRx preload
      ↓
Update Event
      ↓
CCRx active
```

### Vì sao cần Preload?

Nếu PWM đang chạy:

```text
cycle hiện tại
      ↓
Software đổi ARR / CCR
      ↓
giá trị tác động ngay giữa cycle
```

có thể làm chu kỳ hiện tại không theo period/duty mong muốn.

Với preload:

```text
ghi giá trị mới
      ↓
chờ Update Event
      ↓
áp dụng ở boundary thích hợp
```

Waveform vì thế đồng bộ hơn.

### UG

Trước khi bắt đầu PWM, sau khi cấu hình các preload register:

```text
TIMx_EGR.UG = 1
```

được dùng để tạo Update Event bằng software.

Luồng:

```text
PSC / ARR / CCR đã cấu hình
        ↓
UG = 1
        ↓
preload → active
        ↓
bắt đầu counter
```

---

<a id="muc-05-18"></a>
## 5.18. GPIO Alternate Function cho PWM

Timer có thể tạo PWM bên trong nhưng muốn thấy waveform ở chân ngoài thì channel phải được nối tới GPIO qua Alternate Function.

Luồng:

```text
TIMx Counter
    ↓
Compare
    ↓
OCxREF
    ↓
Output Control
    ↓
Timer Channel
    ↓
GPIO Alternate Function
    ↓
Physical Pin
```

Trên STM32F1, Timer PWM output thường dùng:

```text
Alternate Function Push-Pull
```

Ví dụ:

```text
TIM2_CH1
→ PA0
→ Alternate Function Push-Pull
```

Pin chính xác phụ thuộc:

```text
Timer
Channel
Default mapping
AFIO Remap
MCU package
```

### Ví dụ PA0 cho TIM2_CH1

GPIO:

```text
PA0
→ Alternate Function Push-Pull
```

Timer:

```text
TIM2_CH1
→ PWM Mode
```

Hai phía đều phải cấu hình đúng:

```text
GPIO config
+
Timer config
```

Chỉ cấu hình Timer mà bỏ GPIO Alternate Function sẽ không tạo waveform đúng ở chân.

---

<a id="muc-05-19"></a>
## 5.19. Timer Interrupt và DMA

Timer có thể tạo interrupt hoặc DMA request từ nhiều loại event.

Ví dụ:

```text
Update
Input Capture
Output Compare
Trigger
```

### Update Interrupt

Các bit:

```text
TIMx_DIER.UIE
→ enable Update Interrupt

TIMx_SR.UIF
→ Update Interrupt Flag
```

Luồng:

```text
Counter overflow
      ↓
UEV
      ↓
UIF = 1
      ↓
UIE = 1
      ↓
Timer IRQ
      ↓
NVIC
      ↓
TIMx_IRQHandler()
```

### Capture/Compare Interrupt

Mỗi channel có thể tạo:

```text
CC1IF
CC2IF
CC3IF
CC4IF
```

và interrupt enable tương ứng:

```text
CC1IE
CC2IE
CC3IE
CC4IE
```

### DMA

Timer event cũng có thể tạo DMA request nếu bit enable tương ứng được bật.

Luồng:

```text
Timer Event
    ↓
DMA Request
    ↓
DMA Controller
    ↓
Memory / Peripheral transfer
```

Ứng dụng:

```text
cập nhật PWM duty tự động
capture chuỗi timestamp
tạo waveform không cần CPU ghi từng mẫu
```

Chi tiết DMA được xử lý ở chương DMA.

---

<a id="muc-05-20"></a>
## 5.20. Advanced Timer: Complementary PWM / Dead-Time / Break

TIM1 và TIM8 có các tính năng nâng cao cho điều khiển công suất.

### Complementary Output

Một channel có thể có cặp output:

```text
CH1
CH1N

CH2
CH2N

CH3
CH3N
```

Sơ đồ khái niệm:

```text
OC1REF
 ├──→ CH1
 └──→ CH1N
```

Hai output này có thể được dùng để điều khiển cặp transistor công suất.

### Dead-Time

Trong half-bridge hoặc full-bridge, không nên bật đồng thời transistor high-side và low-side.

Nếu xảy ra:

```text
High-side ON
+
Low-side ON
```

có thể tạo:

```text
Shoot-Through
```

Dead-Time chèn một khoảng trễ giữa hai lần chuyển trạng thái:

```text
High-side OFF
      ↓
  Dead-Time
      ↓
Low-side ON
```

Dead-Time được cấu hình trong:

```text
TIMx_BDTR
```

### Break Input

Break cung cấp cơ chế phần cứng để đưa output về trạng thái an toàn khi có fault.

Luồng:

```text
Fault
 ↓
BKIN / Break source
 ↓
Break logic
 ↓
Timer output protection
 ↓
PWM được disable / đưa về trạng thái đã cấu hình
```

Clock failure từ RCC CSS cũng có thể liên hệ với break của Advanced Timer trên STM32F1.

### Main Output Enable

Advanced Timer có:

```text
MOE
→ Main Output Enable
```

trong `TIMx_BDTR`.

Với TIM1/TIM8, chỉ cấu hình PWM channel chưa chắc đã đủ để pin xuất waveform nếu main output chưa được enable.

### Repetition Counter

`TIMx_RCR` cho phép Update Event xảy ra sau một số chu kỳ lặp nhất định.

Khái niệm:

```text
PWM cycle
PWM cycle
PWM cycle
...
      ↓
Repetition Counter hết
      ↓
Update Event
```

---

<a id="muc-05-21"></a>
## 5.21. Quy trình cấu hình Timer

Ví dụ tạo time base định kỳ bằng TIM2.

### Bước 1 — Bật Clock

TIM2 nằm trên APB1:

```c
RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
```

### Bước 2 — Xác định TIM2CLK

Ví dụ:

```text
PCLK1 = 36 MHz
PPRE1 = /2
```

thì:

```text
TIM2CLK = 72 MHz
```

### Bước 3 — Chọn PSC

Muốn counter tick:

```text
1 µs
```

cần:

```text
fCNT = 1 MHz
```

Với:

```text
TIM2CLK = 72 MHz
```

ta chọn:

```text
PSC = 71
```

### Bước 4 — Chọn ARR

Muốn period:

```text
1 ms
```

với tick 1 µs:

```text
1000 ticks
```

nên:

```text
ARR = 999
```

### Bước 5 — Cấu hình PSC / ARR

```c
TIM2->PSC = 71;
TIM2->ARR = 999;
```

### Bước 6 — Tạo Update Event

```c
TIM2->EGR = TIM_EGR_UG;
```

Điều này giúp nạp giá trị buffered cần thiết trước khi bắt đầu.

### Bước 7 — Nếu cần Interrupt

```c
TIM2->DIER |= TIM_DIER_UIE;
NVIC_SetPriority(TIM2_IRQn, 5);
NVIC_EnableIRQ(TIM2_IRQn);
```

### Bước 8 — Enable Counter

```c
TIM2->CR1 |= TIM_CR1_CEN;
```

### Luồng

```text
RCC Clock
   ↓
TIMxCLK
   ↓
PSC
   ↓
ARR
   ↓
UG
   ↓
Interrupt config nếu cần
   ↓
CEN = 1
```

---

<a id="muc-05-22"></a>
## 5.22. Quy trình cấu hình PWM

Ví dụ:

```text
TIM2_CH1
PWM = 1 kHz
Duty = 25%
TIM2CLK = 72 MHz
```

### Bước 1 — Bật Clock

```text
GPIO clock
+
TIM2 clock
+
AFIO clock nếu cần remap
```

Ví dụ PA0 / TIM2_CH1 mặc định:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
```

### Bước 2 — GPIO Alternate Function Push-Pull

PA0 nằm trong `GPIOA_CRL`.

Ví dụ chọn AF Push-Pull:

```text
CNF = 10
MODE != 00
```

Nếu chọn maximum output speed 2 MHz:

```text
MODE = 10
CNF  = 10
→ 0b1010
```

Ví dụ:

```c
GPIOA->CRL &= ~(0xFU << 0);
GPIOA->CRL |=  (0xAU << 0);
```

### Bước 3 — PSC

Chọn:

```text
PSC = 71
```

Ta có:

```text
fCNT = 1 MHz
```

### Bước 4 — ARR

Muốn PWM 1 kHz:

```text
ARR + 1
= 1 MHz / 1 kHz
= 1000
```

nên:

```text
ARR = 999
```

### Bước 5 — CCR1

Duty 25%:

```text
CCR1
= 25% × 1000
= 250
```

### Bước 6 — Chọn PWM Mode 1 + Preload

Channel 1 nằm trong `TIM2_CCMR1`.

Khái niệm:

```text
CC1S = 00
→ channel là Output

OC1M = 110
→ PWM Mode 1

OC1PE = 1
→ CCR1 preload
```

### Bước 7 — Enable Channel

```text
CC1E = 1
```

trong:

```text
TIM2_CCER
```

Nếu Active-High:

```text
CC1P = 0
```

### Bước 8 — ARR Preload

```text
ARPE = 1
```

### Bước 9 — Generate Update Event

```text
UG = 1
```

### Bước 10 — Enable Counter

```text
CEN = 1
```

### Luồng hoàn chỉnh

```text
RCC GPIO + Timer
       ↓
GPIO AF Push-Pull
       ↓
TIMxCLK
       ↓
PSC
       ↓
ARR
       ↓
CCRx
       ↓
PWM Mode
       ↓
OCxPE + ARPE
       ↓
CCxE
       ↓
UG
       ↓
CEN
       ↓
PWM xuất ra chân
```

---

<a id="muc-05-23"></a>
## 5.23. Ví dụ tính PSC / ARR / CCR

### Ví dụ 1 — PWM 1 kHz, Duty 25%

Cho:

```text
TIM2CLK = 72 MHz
```

Yêu cầu:

```text
fPWM = 1 kHz
Duty = 25%
```

Chọn counter clock dễ tính:

```text
fCNT = 1 MHz
```

Suy ra:

```text
PSC + 1
= 72 MHz / 1 MHz
= 72
```

nên:

```text
PSC = 71
```

Tiếp theo:

```text
ARR + 1
= 1 MHz / 1 kHz
= 1000
```

nên:

```text
ARR = 999
```

Duty:

```text
CCR1
= 25% × 1000
= 250
```

Kết quả:

```text
PSC  = 71
ARR  = 999
CCR1 = 250
```

Kiểm tra:

```text
fPWM
= 72 MHz
  ─────────────────
  72 × 1000

= 1 kHz
```

Duty:

```text
250 / 1000
= 25%
```

---

### Ví dụ 2 — PWM 20 kHz, Duty 60%

Cho:

```text
TIMxCLK = 72 MHz
```

Chọn:

```text
PSC = 0
```

nên:

```text
fCNT = 72 MHz
```

Muốn:

```text
fPWM = 20 kHz
```

ta cần:

```text
ARR + 1
= 72,000,000 / 20,000
= 3600
```

nên:

```text
ARR = 3599
```

Duty 60%:

```text
CCR
= 0.60 × 3600
= 2160
```

Kết quả:

```text
PSC = 0
ARR = 3599
CCR = 2160
```

---

### Ví dụ 3 — Timer Interrupt mỗi 10 ms

Cho:

```text
TIM2CLK = 72 MHz
```

Chọn:

```text
PSC = 71
```

Suy ra:

```text
fCNT = 1 MHz
1 tick = 1 µs
```

Muốn:

```text
10 ms
= 10,000 µs
```

nên:

```text
ARR + 1 = 10,000
```

suy ra:

```text
ARR = 9999
```

Kết quả:

```text
Update Event mỗi 10 ms
```

nếu Timer chạy Upcounting liên tục.

---

### Ví dụ 4 — Đo Frequency bằng Input Capture

Cho:

```text
fCNT = 1 MHz
```

Capture lần 1:

```text
CCR1 = 1000
```

Capture lần 2:

```text
CCR1 = 5000
```

Chênh lệch:

```text
ΔCNT = 4000
```

Period:

```text
T
= 4000 / 1 MHz
= 4 ms
```

Frequency:

```text
f
= 1 / 4 ms
= 250 Hz
```

Nếu counter overflow giữa hai capture, phép tính phải xử lý modulo/overflow phù hợp.

---

### Cách chọn PSC và ARR

Một tần số có thể đạt được bằng nhiều cặp:

```text
PSC
ARR
```

Ví dụ cùng một frequency có thể dùng:

```text
PSC nhỏ + ARR lớn
hoặc
PSC lớn + ARR nhỏ
```

Thông thường nên cân nhắc:

```text
Độ phân giải
Giới hạn 16-bit
Tần số mong muốn
Dải Duty Cycle
Khả năng thay đổi period/duty
```

Với PWM, `ARR` càng lớn thì số mức duty khả dụng càng nhiều.

Ví dụ:

```text
ARR = 99
→ khoảng 100 mức

ARR = 999
→ khoảng 1000 mức
```

với cùng cách biểu diễn duty.

---

<a id="muc-05-24"></a>
## 5.24. Câu hỏi tự kiểm tra

1. Timer là gì?
2. Timer phần cứng khác software counter ở điểm nào?
3. Ba thanh ghi nền tảng `PSC`, `CNT`, `ARR` có vai trò gì?
4. Advanced Timer gồm những Timer tiêu biểu nào?
5. General-Purpose Timer TIM2–TIM5 hỗ trợ những chức năng chính nào?
6. TIM6/TIM7 khác TIM2–TIM5 ở điểm quan trọng nào?
7. Trước khi tính Timer period phải xác định clock nào?
8. Khi APB prescaler bằng `/1`, `TIMxCLK` bằng gì?
9. Khi APB prescaler khác `/1`, `TIMxCLK` bằng gì?
10. Nếu `PCLK1 = 36 MHz` và `PPRE1 = /2`, Timer trên APB1 chạy bao nhiêu MHz?
11. `TIMx_CNT` chứa gì?
12. `TIMx_CR1.CEN` có tác dụng gì?
13. Công thức `fCNT` là gì?
14. Vì sao công thức Prescaler có `PSC + 1`?
15. `PSC = 0` tương ứng hệ số chia bao nhiêu?
16. `PSC = 71`, `TIMxCLK = 72 MHz` cho `fCNT` bao nhiêu?
17. `ARR` có vai trò gì?
18. Vì sao một chu kỳ Upcounting từ `0 → ARR` có `ARR + 1` tick?
19. `ARR = 999` tương ứng bao nhiêu count mỗi chu kỳ?
20. Upcounting hoạt động như thế nào?
21. Downcounting hoạt động như thế nào?
22. Center-Aligned Counting hoạt động như thế nào?
23. `DIR` dùng để làm gì trong Edge-Aligned mode?
24. `CMS` dùng để chọn gì?
25. Update Event là gì?
26. UEV có thể được tạo từ những nguồn nào?
27. `UIF` có ý nghĩa gì?
28. `UIE` có ý nghĩa gì?
29. `UG` dùng để làm gì?
30. Công thức frequency của Timer Edge-Aligned Upcounting là gì?
31. Công thức period tương ứng là gì?
32. `TIMx_CCRx` có hai vai trò chính nào?
33. Input Capture khác Output Compare như thế nào?
34. Compare Match xảy ra khi nào?
35. Output Compare có thể làm gì khi match?
36. Input Capture dùng để đo những đại lượng nào?
37. Làm sao tính period từ hai Capture value?
38. PWM là gì?
39. Hai thông số chính của PWM là gì?
40. PWM Frequency phụ thuộc chủ yếu vào những thanh ghi nào?
41. PWM Duty Cycle phụ thuộc chủ yếu vào thanh ghi nào?
42. Công thức PWM Edge-Aligned frequency là gì?
43. Công thức Duty cho PWM Mode 1 Active-High Upcounting là gì?
44. `ARR = 999`, `CCR = 250` cho duty khoảng bao nhiêu?
45. PWM Mode 1 và PWM Mode 2 khác nhau thế nào?
46. `OCxM = 110` là mode nào?
47. `OCxM = 111` là mode nào?
48. `CCxP` điều khiển gì?
49. `CCxE` điều khiển gì?
50. `ARPE` dùng để làm gì?
51. `OCxPE` dùng để làm gì?
52. Vì sao preload hữu ích khi PWM đang chạy?
53. Vì sao thường tạo `UG` trước khi enable counter?
54. GPIO cho Timer PWM output thường dùng mode nào trên STM32F1?
55. Nếu chỉ cấu hình Timer nhưng GPIO vẫn là Input Floating thì điều gì xảy ra ở chân?
56. Timer có thể tạo những loại interrupt nào?
57. Timer event có thể kích DMA không?
58. `TIMx_SR.UIF` và NVIC Pending có phải cùng một trạng thái không?
59. Complementary PWM là gì?
60. Dead-Time dùng để tránh vấn đề gì?
61. Break input có tác dụng gì?
62. `MOE` có vai trò gì trên TIM1/TIM8?
63. Repetition Counter dùng để làm gì?
64. Hãy tính `PSC`, `ARR`, `CCR` cho PWM 1 kHz, duty 25%, `TIMxCLK = 72 MHz`.
65. Hãy tính `ARR` cho Timer interrupt 10 ms nếu `fCNT = 1 MHz`.
66. Với `TIMxCLK = 72 MHz`, `PSC = 0`, `ARR = 3599`, PWM frequency là bao nhiêu?
67. Với `ARR = 3599`, `CCR = 2160`, duty xấp xỉ bao nhiêu?
68. Vì sao cùng một PWM frequency có thể có nhiều cặp `PSC/ARR`?
69. `ARR` lớn hơn có lợi gì cho PWM duty resolution?
70. Hãy mô tả đầy đủ luồng `RCC → TIMxCLK → PSC → CNT → ARR → CCR → PWM → GPIO`.

---

## 5.25. Tóm tắt

Luồng Timer:

```text
RCC
 ↓
TIMxCLK
 ↓
PSC
 ↓
CK_CNT
 ↓
CNT
 ↓
ARR
 ↓
Update Event
```

Công thức Edge-Aligned:

```text
fCNT =
fTIM / (PSC + 1)
```

```text
fUPDATE =
fTIM / [(PSC + 1)(ARR + 1)]
```

PWM:

```text
Timer Clock
    ↓
PSC
    ↓
CNT
    ↓
ARR
    ↓
Compare với CCRx
    ↓
OCxREF
    ↓
GPIO Alternate Function
    ↓
PWM
```

Với PWM Mode 1, Active-High, Upcounting:

```text
fPWM =
fTIM / [(PSC + 1)(ARR + 1)]
```

```text
Duty ≈
CCRx / (ARR + 1) × 100%
```

Capture/Compare:

```text
Input Capture
→ CNT → CCR

Output Compare
→ CNT ↔ CCR
```

Preload:

```text
Software ghi ARR / CCR
        ↓
Preload
        ↓
Update Event
        ↓
Active register
```

Advanced Timer:

```text
TIM1 / TIM8
├── Complementary PWM
├── Dead-Time
├── Break
├── MOE
└── Repetition Counter
```

**Điểm cần nhớ:**

> **Muốn tính hoặc cấu hình Timer đúng, trước hết phải xác định `TIMxCLK`. Sau đó `PSC` quyết định counter clock, `ARR` quyết định chu kỳ, còn `CCR` quyết định Capture/Compare point và trong PWM quyết định duty. Khi xuất PWM ra chân, Timer channel phải được nối với GPIO bằng Alternate Function phù hợp.**

[↑ Về mục lục](#muc-luc)


---

<a id="chuong-06"></a>
# 6. UART / USART

STM32F10xxx sử dụng peripheral `USART` cho truyền thông nối tiếp. Khi USART hoạt động ở chế độ bất đồng bộ với `TX/RX`, cách sử dụng tương ứng với UART.

Luồng tổng quát:

```text
Peripheral Clock
      ↓
Baud Rate Generator
      ↓
USART
├── Transmitter
│   └── TX
└── Receiver
    └── RX
```

Các khối liên quan:

```text
RCC
→ cấp clock cho USART

GPIO
→ TX / RX

USART_BRR
→ Baud Rate

USART_DR
→ dữ liệu truyền / nhận

USART_SR
→ trạng thái và lỗi

NVIC
→ USART Interrupt

DMA
→ truyền dữ liệu USART ↔ SRAM
```

<a id="muc-06-01"></a>
## 6.1. UART và USART là gì?

`UART`:

```text
Universal Asynchronous Receiver/Transmitter
```

`USART`:

```text
Universal Synchronous/Asynchronous Receiver/Transmitter
```

Quan hệ:

```text
USART
├── Asynchronous
│   └── truyền kiểu UART
│
└── Synchronous
    └── có thêm clock CK
```

STM32F10xxx USART còn hỗ trợ:

```text
Full-Duplex Asynchronous
Synchronous one-way communication
Single-Wire Half-Duplex
LIN
Smartcard
IrDA
Multiprocessor communication
CTS / RTS
DMA
```

Phần nền tảng tập trung vào:

```text
Asynchronous
Full-Duplex
TX + RX
```

---

<a id="muc-06-02"></a>
## 6.2. Truyền nối tiếp bất đồng bộ

Truyền nối tiếp gửi các bit lần lượt theo thời gian trên một đường tín hiệu.

Ví dụ một byte:

```text
D0 → D1 → D2 → D3 → D4 → D5 → D6 → D7
```

Trong USART bất đồng bộ:

```text
không có đường clock CK dùng để đồng bộ từng bit
```

Hai thiết bị phải thống nhất các tham số:

```text
Baud Rate
Word Length
Parity
Stop Bits
```

Ví dụ:

```text
115200 baud
8 data bits
No parity
1 stop bit
```

được viết:

```text
115200 8N1
```

### Full-Duplex

Full-Duplex cho phép:

```text
TX
→ truyền

RX
→ nhận

TX và RX có thể hoạt động đồng thời
```

Kết nối hai thiết bị:

```text
MCU A TX ─────────→ RX MCU B
MCU A RX ←───────── TX MCU B
GND      ────────── GND
```

---

<a id="muc-06-03"></a>
## 6.3. UART Frame: Start / Data / Parity / Stop

Một frame bất đồng bộ gồm:

```text
Idle
 ↓
Start Bit
 ↓
Data
 ↓
Parity nếu enable
 ↓
Stop Bit
```

USART truyền data:

```text
LSB first
```

### Idle

Khi không truyền:

```text
TX = logic 1
```

### Start Bit

Start Bit:

```text
logic 0
```

Dùng để báo receiver rằng một frame mới bắt đầu.

### Data Word

Word length được cấu hình bằng:

```text
USART_CR1.M
```

Hai lựa chọn:

```text
M = 0
→ 8-bit word

M = 1
→ 9-bit word
```

### Parity

Parity được điều khiển bởi:

```text
USART_CR1.PCE
USART_CR1.PS
```

Trong đó:

```text
PCE = 0
→ parity disable

PCE = 1
→ parity enable
```

Khi parity enable:

```text
PS = 0
→ Even parity

PS = 1
→ Odd parity
```

### Stop Bit

Stop Bit có mức:

```text
logic 1
```

`USART_CR2.STOP`:

```text
00 → 1 Stop Bit
01 → 0.5 Stop Bit
10 → 2 Stop Bits
11 → 1.5 Stop Bits
```

Trong truyền UART thông thường, các cấu hình quan trọng nhất là:

```text
1 Stop Bit
2 Stop Bits
```

`0.5` và `1.5` Stop Bit được dùng trong Smartcard mode.

---

<a id="muc-06-04"></a>
## 6.4. 8N1 và các cấu hình Frame

Ký hiệu:

```text
8N1
│││
││└── 1 Stop Bit
│└─── No Parity
└──── 8 Data Bits
```

Cấu hình 8N1:

```text
M = 0
PCE = 0
STOP = 00
```

Frame:

```text
Idle  Start   Data[7:0]                Stop
 1      0    D0 D1 D2 D3 D4 D5 D6 D7   1
```

Tổng số bit:

```text
1 Start
+
8 Data
+
1 Stop
=
10 bit / frame
```

Ở:

```text
115200 bit/s
```

throughput lý tưởng của 8N1:

```text
115200 / 10
≈ 11520 byte/s
```

nếu các frame được truyền liên tục.

### Quan hệ giữa M và Parity

Parity sử dụng vị trí MSB của word.

Các frame:

| `M` | `PCE` | Frame data |
|---|---|---|
| `0` | `0` | 8 data bits |
| `0` | `1` | 7 data bits + parity |
| `1` | `0` | 9 data bits |
| `1` | `1` | 8 data bits + parity |

Do đó muốn:

```text
8E1
```

cần:

```text
M   = 1
PCE = 1
PS  = 0
STOP = 00
```

Tương tự:

```text
8O1
```

cần:

```text
M   = 1
PCE = 1
PS  = 1
STOP = 00
```

### Khi ghi / đọc USART_DR với Parity

Khi parity được enable:

```text
Transmit:
MSB của data word
→ được thay bằng parity bit

Receive:
MSB đọc từ DR
→ chứa parity bit nhận được
```

---

<a id="muc-06-05"></a>
## 6.5. Baud Rate

Baud Rate biểu diễn tốc độ symbol của đường truyền.

Trong UART NRZ thông thường:

```text
1 symbol
→ 1 bit
```

nên thường có thể xem:

```text
Baud Rate ≈ bit/s
```

Các giá trị phổ biến:

```text
9600
19200
57600
115200
230400
...
```

Hai thiết bị phải dùng Baud Rate đủ tương thích để receiver lấy mẫu đúng.

Baud Rate của USART STM32F1 được tạo bằng:

```text
Peripheral Clock
      ↓
Fractional Baud Rate Generator
      ↓
Transmit / Receive Baud
```

Transmitter và Receiver sử dụng cùng Baud Rate Generator.

---

<a id="muc-06-06"></a>
## 6.6. USART Clock và USART_BRR

Clock cấp cho USART:

```text
USART1
→ PCLK2

USART2
USART3
UART4
UART5
→ PCLK1
```

Peripheral tồn tại hay không phụ thuộc từng mã STM32F10xxx.

### Công thức Baud Rate

```text
Baud =
fCK
────────────────
16 × USARTDIV
```

Trong đó:

```text
fCK
→ PCLK2 với USART1
→ PCLK1 với USART2/USART3/UART4/UART5
```

`USARTDIV`:

```text
USARTDIV
=
DIV_Mantissa
+
DIV_Fraction / 16
```

Thanh ghi:

```text
USART_BRR
```

có cấu trúc:

```text
BRR[15:4]
→ DIV_Mantissa

BRR[3:0]
→ DIV_Fraction
```

### Ví dụ USART1 115200 baud

Cho:

```text
PCLK2 = 72 MHz
Baud  = 115200
```

Ta có:

```text
USARTDIV
=
72,000,000
────────────────
16 × 115200

= 39.0625
```

Suy ra:

```text
DIV_Mantissa = 39
```

Phần thập phân:

```text
0.0625 × 16
= 1
```

Nên:

```text
DIV_Fraction = 1
```

BRR:

```text
BRR
= (39 << 4) | 1
= 0x271
```

### Ví dụ USART2 115200 baud với PCLK1 = 36 MHz

```text
USARTDIV
=
36,000,000
────────────────
16 × 115200

= 19.53125
```

Phần fraction:

```text
0.53125 × 16
= 8.5
```

Làm tròn gần nhất:

```text
DIV_Fraction ≈ 9
```

Nên giá trị thực tế sẽ có một sai số baud nhỏ.

### Điểm cần nhớ

```text
BRR giống nhau
+
Peripheral Clock khác nhau
→ Baud Rate khác nhau
```

Không được copy `USART_BRR` giữa USART1 và USART2 mà không kiểm tra `PCLK2/PCLK1`.

`USART_BRR` không nên được thay đổi trong khi đang communication.

---

<a id="muc-06-07"></a>
## 6.7. TX / RX và GPIO

Giao tiếp bất đồng bộ full-duplex tối thiểu cần:

```text
TX
→ Transmit Data Output

RX
→ Receive Data Input
```

Trên STM32F1:

```text
TX
→ Alternate Function Push-Pull

RX
→ Input Floating
  hoặc Input Pull-Up
```

### USART1 mặc định

```text
PA9  → USART1_TX
PA10 → USART1_RX
```

### USART1 Remap

```text
PB6 → USART1_TX
PB7 → USART1_RX
```

Remap được cấu hình bằng:

```text
AFIO_MAPR
```

và phải bật:

```text
AFIOEN
```

trước khi truy cập AFIO.

### Nối hai thiết bị

```text
Device A TX → Device B RX
Device A RX ← Device B TX
GND A       ↔ GND B
```

Không nối:

```text
TX → TX
RX → RX
```

trong kết nối UART thông thường giữa hai thiết bị.

---

<a id="muc-06-08"></a>
## 6.8. USART_DR và cơ chế truyền dữ liệu

Thanh ghi:

```text
USART_DR
```

có hai chức năng:

```text
Write
→ Transmit Data Register — TDR

Read
→ Receive Data Register — RDR
```

### Transmit path

```text
CPU / DMA
    ↓ write USART_DR
   TDR
    ↓
Transmit Shift Register
    ↓
   TX Pin
```

### Receive path

```text
RX Pin
  ↓
Receive Shift Register
  ↓
 RDR
  ↓ read USART_DR
CPU / DMA
```

### Transmit Shift Register

Shift Register gửi:

```text
Start
Data LSB first
Parity nếu có
Stop
```

ra TX theo Baud Rate.

### Receive Shift Register

Receiver lấy mẫu RX, phục hồi các bit và sau khi nhận xong character sẽ chuyển dữ liệu vào RDR.

---

<a id="muc-06-09"></a>
## 6.9. TXE và TC

Hai flag:

```text
USART_SR.TXE
→ Transmit Data Register Empty

USART_SR.TC
→ Transmission Complete
```

### TXE

`TXE = 1` khi:

```text
dữ liệu từ TDR
→ đã chuyển sang Transmit Shift Register
```

Điều này có nghĩa:

```text
TDR trống
→ có thể ghi data tiếp theo
```

TXE được clear khi:

```text
write USART_DR
```

Luồng truyền liên tục:

```text
TXE = 1
 ↓
write byte 1 vào DR
 ↓
TDR → Shift Register
 ↓
TXE = 1
 ↓
write byte 2
 ↓
...
```

### TC

`TC = 1` khi:

```text
frame chứa data đã truyền hoàn tất
+
TXE = 1
```

Tức:

```text
TXE
→ buffer truyền trống

TC
→ frame cuối đã đi hết ra đường truyền
```

### So sánh

| Flag | Ý nghĩa |
|---|---|
| `TXE` | Có thể nạp byte tiếp theo |
| `TC` | Frame cuối đã truyền hoàn tất |

Khi gửi nhiều byte:

```text
TXE
→ dùng để feed byte tiếp theo
```

Sau byte cuối:

```text
TC
→ dùng khi cần xác nhận transmission hoàn tất
```

Trước khi disable USART hoặc chuyển sang trạng thái có thể làm dừng transmission:

```text
phải chờ TC = 1
```

---

<a id="muc-06-10"></a>
## 6.10. RXNE và quá trình nhận dữ liệu

Flag:

```text
USART_SR.RXNE
```

là:

```text
Read Data Register Not Empty
```

Khi một character được nhận:

```text
RX
 ↓
Receive Shift Register
 ↓
RDR
 ↓
RXNE = 1
```

Ý nghĩa:

```text
RXNE = 1
→ dữ liệu đã sẵn sàng trong USART_DR
```

Trong single-buffer mode:

```text
read USART_DR
→ clear RXNE
```

Ví dụ:

```c
while (!(USART1->SR & USART_SR_RXNE))
{
}

uint8_t data = (uint8_t)USART1->DR;
```

### Yêu cầu thời gian

Software phải đọc dữ liệu trước khi character tiếp theo cần chuyển vào RDR.

Nếu:

```text
RXNE vẫn = 1
+
character mới nhận xong
```

thì có thể xảy ra:

```text
Overrun Error
```

---

<a id="muc-06-11"></a>
## 6.11. Error Flags: ORE / FE / NE / PE

Các lỗi quan trọng trong `USART_SR`:

```text
ORE
→ Overrun Error

NE
→ Noise Error

FE
→ Framing Error

PE
→ Parity Error
```

### ORE — Overrun Error

ORE xảy ra khi:

```text
RDR đang chứa data chưa đọc
RXNE = 1
      ↓
character mới nhận xong
      ↓
không thể chuyển character mới vào RDR
      ↓
ORE = 1
```

Khi ORE xảy ra:

```text
data cũ trong RDR vẫn còn
shift register sẽ bị ghi đè
ít nhất một data đã bị mất
```

Một nguyên nhân thường gặp:

```text
CPU hoặc DMA không lấy data đủ nhanh
```

### FE — Framing Error

FE xảy ra khi:

```text
Stop Bit không được nhận đúng tại thời điểm mong đợi
```

Có thể liên quan đến:

```text
mất đồng bộ
noise quá lớn
baud không phù hợp
Break character
```

### NE — Noise Error

Receiver dùng kỹ thuật oversampling để phân biệt dữ liệu hợp lệ và noise.

Nếu các mẫu tại điểm quyết định không一致:

```text
NE = 1
```

Data vẫn được chuyển vào `USART_DR`, nhưng phải được xem là không đảm bảo.

### PE — Parity Error

Khi parity enable:

```text
receiver tính parity
       ↓
so với parity nhận được
       ↓
không khớp
       ↓
PE = 1
```

### Clear ORE / NE / FE / PE

Các flag này dùng trình tự:

```text
read USART_SR
      ↓
read USART_DR
```

Đối với `PE`, software phải chờ `RXNE` được set trước khi hoàn tất trình tự clear bằng đọc `DR`.

---

<a id="muc-06-12"></a>
## 6.12. USART Interrupt

Các nguồn interrupt quan trọng:

```text
TXE
TC
RXNE
IDLE
PE
ORE
FE
NE
CTS
LIN Break
```

Các bit enable thường dùng:

```text
USART_CR1.TXEIE
→ TXE Interrupt Enable

USART_CR1.TCIE
→ TC Interrupt Enable

USART_CR1.RXNEIE
→ RXNE Interrupt Enable

USART_CR1.IDLEIE
→ IDLE Interrupt Enable

USART_CR1.PEIE
→ Parity Error Interrupt Enable
```

Trong multi-buffer/DMA reception:

```text
USART_CR3.EIE
```

liên quan đến interrupt khi:

```text
FE
ORE
NE
```

với `DMAR = 1`.

### Receive Interrupt

Luồng:

```text
Character received
      ↓
RXNE = 1
      ↓
RXNEIE = 1
      ↓
USARTx IRQ
      ↓
NVIC
      ↓
USARTx_IRQHandler()
      ↓
read USART_DR
```

Ví dụ:

```c
void USART1_IRQHandler(void)
{
    if (USART1->SR & USART_SR_RXNE)
    {
        uint8_t data = (uint8_t)USART1->DR;

        /* lưu data */
    }
}
```

### Transmit Interrupt

```text
TDR trống
  ↓
TXE = 1
  ↓
TXEIE = 1
  ↓
USART IRQ
```

ISR có thể ghi byte tiếp theo vào `USART_DR`.

Khi không còn byte cần gửi:

```text
disable TXEIE
```

để tránh interrupt liên tục do `TXE` vẫn ở trạng thái `1`.

---

<a id="muc-06-13"></a>
## 6.13. IDLE Line

Flag:

```text
USART_SR.IDLE
```

được set khi receiver phát hiện Idle Line.

Nếu:

```text
IDLEIE = 1
```

USART có thể tạo interrupt.

Clear IDLE:

```text
read USART_SR
      ↓
read USART_DR
```

IDLE chỉ được set lại sau khi `RXNE` đã từng được set, tức phải có hoạt động nhận mới trước một Idle Line mới.

### Ứng dụng

IDLE Line hữu ích khi:

```text
nhận dữ liệu có độ dài thay đổi
```

Luồng khái niệm:

```text
RX data
RX data
RX data
      ↓
đường RX rỗi
      ↓
IDLE
      ↓
xác định một đoạn dữ liệu đã kết thúc
```

IDLE thường được kết hợp với:

```text
DMA Receive
```

để nhận chuỗi dữ liệu mà không cần interrupt cho từng byte.

---

<a id="muc-06-14"></a>
## 6.14. UART bằng Polling

Polling trực tiếp kiểm tra các flag trong `USART_SR`.

### Transmit một byte

```c
static void USART1_WriteByte(uint8_t data)
{
    while (!(USART1->SR & USART_SR_TXE))
    {
    }

    USART1->DR = data;
}
```

### Transmit một buffer

```c
static void USART1_Write(const uint8_t *data, uint32_t length)
{
    for (uint32_t i = 0; i < length; ++i)
    {
        while (!(USART1->SR & USART_SR_TXE))
        {
        }

        USART1->DR = data[i];
    }

    while (!(USART1->SR & USART_SR_TC))
    {
    }
}
```

Luồng:

```text
TXE?
 ↓ yes
Write DR
 ↓
TXE?
 ↓
Write byte tiếp
 ↓
...
 ↓
byte cuối
 ↓
wait TC
```

### Receive một byte

```c
static uint8_t USART1_ReadByte(void)
{
    while (!(USART1->SR & USART_SR_RXNE))
    {
    }

    return (uint8_t)USART1->DR;
}
```

### Đặc điểm Polling

Ưu điểm:

```text
đơn giản
dễ kiểm tra
ít trạng thái
```

Hạn chế:

```text
CPU phải chờ flag
blocking
khó mở rộng khi nhiều công việc chạy đồng thời
```

---

<a id="muc-06-15"></a>
## 6.15. UART bằng Interrupt

Interrupt cho phép CPU làm việc khác cho tới khi USART có sự kiện.

### Receive

Enable:

```c
USART1->CR1 |= USART_CR1_RXNEIE;
NVIC_EnableIRQ(USART1_IRQn);
```

Handler:

```c
volatile uint8_t rx_data;
volatile uint8_t rx_ready;

void USART1_IRQHandler(void)
{
    if (USART1->SR & USART_SR_RXNE)
    {
        rx_data = (uint8_t)USART1->DR;
        rx_ready = 1;
    }
}
```

Main:

```c
while (1)
{
    if (rx_ready)
    {
        rx_ready = 0;

        /* xử lý rx_data */
    }
}
```

### Buffer nhiều byte

Thay vì giữ một byte:

```text
ISR
 ↓
read DR
 ↓
ghi vào RAM buffer
 ↓
cập nhật write index
 ↓
return
```

Có thể dùng:

```text
Ring Buffer
```

để nhận dữ liệu liên tục.

### Transmit

Với interrupt-driven TX:

```text
software có buffer TX
      ↓
enable TXEIE
      ↓
TXE interrupt
      ↓
ISR ghi byte tiếp theo
      ↓
hết buffer
      ↓
disable TXEIE
```

Nếu cần biết frame cuối đã rời TX:

```text
TC / TCIE
```

được dùng sau byte cuối.

---

<a id="muc-06-16"></a>
## 6.16. UART bằng DMA

USART hỗ trợ DMA cho transmit và receive.

Các bit:

```text
USART_CR3.DMAT
→ DMA Enable Transmitter

USART_CR3.DMAR
→ DMA Enable Receiver
```

### DMA Transmit

```text
SRAM Buffer
    ↓
DMA
    ↓
USART TDR
    ↓
Shift Register
    ↓
TX
```

### DMA Receive

```text
RX
 ↓
Receive Shift Register
 ↓
RDR
 ↓
DMA
 ↓
SRAM Buffer
```

Ưu điểm:

```text
CPU không cần xử lý từng byte
phù hợp dữ liệu liên tục
giảm số interrupt
```

Trong multibuffer reception:

```text
RXNE được set sau mỗi byte
→ DMA read DR
→ RXNE được clear
```

Nếu DMA không phục vụ kịp và data mới tới:

```text
ORE có thể xảy ra
```

### DMA + IDLE

Một mô hình thường dùng:

```text
USART RX
   ↓
DMA
   ↓
Buffer
   ↓
IDLE interrupt
   ↓
xử lý số byte đã nhận
```

Chi tiết channel, transfer count và DMA flags được triển khai ở chương DMA.

---

<a id="muc-06-17"></a>
## 6.17. Hardware Flow Control: CTS / RTS

USART1/2/3 hỗ trợ hardware flow control với:

```text
CTS
RTS
```

UART4/UART5 không có các chức năng CTS/RTS tương ứng.

### CTS — Clear To Send

Bit:

```text
USART_CR3.CTSE
```

Khi enable:

```text
CTS asserted
→ USART được phép truyền

CTS deasserted
→ transmission mới bị hoãn
```

Nếu CTS thay đổi khi một character đang được truyền:

```text
character hiện tại được truyền xong
→ sau đó transmitter mới dừng
```

### RTS — Request To Send

Bit:

```text
USART_CR3.RTSE
```

RTS cho biết receiver có khả năng nhận dữ liệu hay không.

Khái niệm:

```text
RTS asserted
→ có chỗ trong receive buffer
→ phía bên kia có thể gửi
```

### CTS Interrupt

```text
CTSIE
→ interrupt khi CTS thay đổi
```

Hardware Flow Control hữu ích khi hai thiết bị không thể luôn xử lý dữ liệu với cùng tốc độ.

---

<a id="muc-06-18"></a>
## 6.18. Half-Duplex và Synchronous Mode

### Single-Wire Half-Duplex

Bit:

```text
USART_CR3.HDSEL
```

Khi:

```text
HDSEL = 1
```

USART hoạt động ở single-wire half-duplex.

Khái niệm:

```text
một đường data
←→
vừa TX vừa RX
```

Do chỉ có một đường:

```text
không truyền và nhận đồng thời như Full-Duplex
```

### Synchronous Mode

USART có thể xuất clock:

```text
CK
```

Bit:

```text
USART_CR2.CLKEN
```

Các bit:

```text
CPOL
CPHA
LBCL
```

điều khiển clock synchronous.

Luồng:

```text
USART
├── TX
├── RX
└── CK
```

Trong synchronous mode:

```text
CK
→ transmitter clock output
```

UART4/UART5 không có synchronous clock output tương ứng.

### Các mode khác

USART còn hỗ trợ:

```text
LIN
Smartcard
IrDA
Multiprocessor communication
```

Các mode này sử dụng thêm các bit trong `CR1/CR2/CR3` và có quy tắc frame riêng.

---

<a id="muc-06-19"></a>
## 6.19. Quy trình cấu hình UART

Ví dụ mục tiêu:

```text
USART1
115200 baud
8N1
Full-Duplex
TX = PA9
RX = PA10
PCLK2 = 72 MHz
```

### Bước 1 — Bật Clock

USART1 và GPIOA nằm trên APB2:

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN
               | RCC_APB2ENR_USART1EN;
```

Nếu dùng remap:

```text
bật AFIOEN
+
cấu hình AFIO_MAPR
```

### Bước 2 — TX GPIO

PA9:

```text
Alternate Function Push-Pull
```

Ví dụ maximum output speed 50 MHz:

```text
MODE = 11
CNF  = 10
→ 0b1011
```

PA9 nằm trong `GPIOA_CRH`, shift:

```text
(9 - 8) × 4
= 4
```

```c
GPIOA->CRH &= ~(0xFU << 4);
GPIOA->CRH |=  (0xBU << 4);
```

### Bước 3 — RX GPIO

PA10:

```text
Input Floating
```

```text
MODE = 00
CNF  = 01
→ 0b0100
```

Shift:

```text
(10 - 8) × 4
= 8
```

```c
GPIOA->CRH &= ~(0xFU << 8);
GPIOA->CRH |=  (0x4U << 8);
```

### Bước 4 — Enable USART

```c
USART1->CR1 |= USART_CR1_UE;
```

### Bước 5 — Word Length

8N1:

```text
M = 0
```

```c
USART1->CR1 &= ~USART_CR1_M;
```

### Bước 6 — Parity

No Parity:

```text
PCE = 0
```

```c
USART1->CR1 &= ~USART_CR1_PCE;
```

### Bước 7 — Stop Bit

1 Stop Bit:

```text
STOP = 00
```

```c
USART1->CR2 &= ~USART_CR2_STOP;
```

### Bước 8 — Baud Rate

Với:

```text
PCLK2 = 72 MHz
115200 baud
```

```text
BRR = 0x271
```

```c
USART1->BRR = 0x271U;
```

### Bước 9 — Enable Transmitter / Receiver

```c
USART1->CR1 |= USART_CR1_TE
               | USART_CR1_RE;
```

Khi `TE` được set:

```text
transmitter gửi Idle Frame trước data đầu tiên
```

Receiver bắt đầu tìm Start Bit khi:

```text
RE = 1
```

### Bước 10 — Nếu dùng Interrupt

Ví dụ RXNE:

```c
USART1->CR1 |= USART_CR1_RXNEIE;

NVIC_SetPriority(USART1_IRQn, 5);
NVIC_EnableIRQ(USART1_IRQn);
```

### Luồng đầy đủ

```text
RCC Clock
   ↓
GPIO TX / RX
   ↓
UE
   ↓
M
   ↓
Parity
   ↓
STOP
   ↓
BRR
   ↓
TE / RE
   ↓
Interrupt / DMA nếu cần
   ↓
Transmit / Receive
```

---

<a id="muc-06-20"></a>
## 6.20. Ví dụ USART1 115200 8N1

Điều kiện:

```text
PCLK2 = 72 MHz
USART1_TX = PA9
USART1_RX = PA10
Baud = 115200
8N1
Polling
```

### Initialization

```c
static void USART1_Init_115200_8N1(void)
{
    /* GPIOA + USART1 clock */
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN
                   | RCC_APB2ENR_USART1EN;

    /* PA9: Alternate Function Push-Pull, 50 MHz */
    GPIOA->CRH &= ~(0xFU << 4);
    GPIOA->CRH |=  (0xBU << 4);

    /* PA10: Input Floating */
    GPIOA->CRH &= ~(0xFU << 8);
    GPIOA->CRH |=  (0x4U << 8);

    /* USART enable */
    USART1->CR1 |= USART_CR1_UE;

    /* 8-bit word */
    USART1->CR1 &= ~USART_CR1_M;

    /* No parity */
    USART1->CR1 &= ~USART_CR1_PCE;

    /* 1 stop bit */
    USART1->CR2 &= ~USART_CR2_STOP;

    /* 72 MHz / (16 × 39.0625) = 115200 */
    USART1->BRR = 0x271U;

    /* TX + RX enable */
    USART1->CR1 |= USART_CR1_TE
                   | USART_CR1_RE;
}
```

### Transmit Byte

```c
static void USART1_WriteByte(uint8_t data)
{
    while (!(USART1->SR & USART_SR_TXE))
    {
    }

    USART1->DR = data;
}
```

### Transmit Buffer

```c
static void USART1_Write(const uint8_t *data, uint32_t length)
{
    for (uint32_t i = 0; i < length; ++i)
    {
        USART1_WriteByte(data[i]);
    }

    while (!(USART1->SR & USART_SR_TC))
    {
    }
}
```

### Receive Byte

```c
static uint8_t USART1_ReadByte(void)
{
    while (!(USART1->SR & USART_SR_RXNE))
    {
    }

    return (uint8_t)USART1->DR;
}
```

### Echo

```c
int main(void)
{
    USART1_Init_115200_8N1();

    while (1)
    {
        uint8_t data = USART1_ReadByte();
        USART1_WriteByte(data);
    }
}
```

Luồng:

```text
PC / USB-UART
      ↓
     RX
      ↓
USART1
      ↓
ReadByte()
      ↓
WriteByte()
      ↓
     TX
      ↓
PC / USB-UART
```

---

### Quy trình xử lý lỗi khi nhận

Một handler có thể kiểm tra:

```text
PE
FE
NE
ORE
RXNE
```

Ví dụ khái niệm:

```c
void USART1_IRQHandler(void)
{
    uint32_t sr = USART1->SR;

    if (sr & (USART_SR_PE |
              USART_SR_FE |
              USART_SR_NE |
              USART_SR_ORE))
    {
        volatile uint32_t dummy = USART1->DR;
        (void)dummy;

        /* ghi nhận / xử lý lỗi */
        return;
    }

    if (sr & USART_SR_RXNE)
    {
        uint8_t data = (uint8_t)USART1->DR;

        /* lưu data */
    }
}
```

Việc đọc `SR` rồi đọc `DR` thực hiện trình tự clear các receive error flag tương ứng.

---

### Các lỗi cấu hình thường gặp

#### Dùng sai Peripheral Clock khi tính BRR

Sai:

```text
USART1 dùng PCLK1
```

Đúng:

```text
USART1
→ PCLK2

USART2/3, UART4/5
→ PCLK1
```

#### Nhầm TXE với TC

Sai:

```text
TXE = 1
→ byte cuối đã truyền xong
```

Đúng:

```text
TXE
→ TDR trống

TC
→ frame cuối truyền hoàn tất
```

#### Không đọc DR đủ nhanh

```text
RXNE = 1
+
data mới tới
→ ORE
```

#### Clear Error Flag sai cách

`ORE/NE/FE/PE` không được xử lý như một bit read/write thông thường.

Trình tự:

```text
read SR
→ read DR
```

#### Cấu hình 8E1 với M = 0

Nếu:

```text
M = 0
PCE = 1
```

frame thực tế là:

```text
7 data bits + parity
```

Muốn:

```text
8 data bits + parity
```

phải dùng:

```text
M = 1
PCE = 1
```

#### GPIO sai Mode

```text
TX
→ AF Push-Pull

RX
→ Input
```

#### Disable USART trước TC

Sau byte cuối:

```text
wait TC = 1
```

trước khi disable USART nếu muốn bảo đảm frame cuối không bị hỏng.

---

<a id="muc-06-21"></a>
## 6.21. Câu hỏi tự kiểm tra

1. UART viết tắt của gì?
2. USART viết tắt của gì?
3. USART khác UART ở khả năng nào?
4. Asynchronous communication có dùng đường CK để đồng bộ từng bit không?
5. Full-Duplex cần tối thiểu những đường dữ liệu nào?
6. TX của thiết bị A phải nối với chân nào của thiết bị B?
7. Một frame UART bất đồng bộ gồm những phần nào?
8. Start Bit có mức logic gì?
9. Stop Bit có mức logic gì?
10. USART STM32F1 truyền MSB first hay LSB first?
11. `USART_CR1.M = 0` tương ứng word length bao nhiêu?
12. `M = 1` tương ứng word length bao nhiêu?
13. `PCE` dùng để làm gì?
14. `PS = 0` là parity gì?
15. `PS = 1` là parity gì?
16. `STOP = 00` là bao nhiêu Stop Bit?
17. 8N1 nghĩa là gì?
18. Một frame 8N1 có bao nhiêu bit?
19. Vì sao 115200 8N1 chỉ đạt khoảng 11520 byte/s lý tưởng?
20. Nếu `M=0`, `PCE=1`, có bao nhiêu data bit thực tế?
21. Muốn 8E1 phải cấu hình `M/PCE/PS` thế nào?
22. Baud Rate của USART được tạo từ clock nào?
23. USART1 lấy clock từ bus nào?
24. USART2/USART3 lấy clock từ bus nào?
25. Công thức Baud Rate của STM32F1 là gì?
26. `USARTDIV` gồm hai phần nào?
27. `BRR[15:4]` lưu gì?
28. `BRR[3:0]` lưu gì?
29. Với USART1, PCLK2=72 MHz, 115200 baud, BRR bằng bao nhiêu?
30. Vì sao cùng BRR nhưng USART1 và USART2 có thể ra baud khác nhau?
31. TX GPIO thường dùng mode nào trên STM32F1?
32. RX GPIO thường dùng mode nào?
33. USART1 TX/RX mặc định ở chân nào?
34. USART1 remap sang chân nào?
35. `USART_DR` có hai chức năng nào?
36. TDR là gì?
37. RDR là gì?
38. TXE có nghĩa gì?
39. TXE được clear khi nào?
40. TC có nghĩa gì?
41. TXE khác TC thế nào?
42. Khi gửi nhiều byte, flag nào dùng để nạp byte tiếp?
43. Sau byte cuối, khi nào cần chờ TC?
44. RXNE có nghĩa gì?
45. RXNE được clear như thế nào trong single-buffer mode?
46. ORE xảy ra khi nào?
47. Khi ORE xảy ra, data nào bị mất?
48. FE biểu thị lỗi gì?
49. NE biểu thị lỗi gì?
50. PE biểu thị lỗi gì?
51. Trình tự clear ORE/NE/FE là gì?
52. `RXNEIE` dùng để làm gì?
53. `TXEIE` dùng để làm gì?
54. `TCIE` dùng để làm gì?
55. `IDLEIE` dùng để làm gì?
56. Tại sao phải disable TXEIE khi TX buffer đã hết dữ liệu?
57. IDLE Line được dùng để nhận biết điều gì?
58. IDLE được clear bằng trình tự nào?
59. Polling UART hoạt động như thế nào?
60. Interrupt UART khác Polling ở điểm nào?
61. DMA UART có lợi gì?
62. `DMAT` dùng để làm gì?
63. `DMAR` dùng để làm gì?
64. DMA Receive kết hợp IDLE hữu ích trong trường hợp nào?
65. CTS có vai trò gì?
66. RTS có vai trò gì?
67. UART4/UART5 có CTS/RTS như USART1/2/3 không?
68. `HDSEL` chọn chế độ gì?
69. Synchronous USART cần thêm chân nào?
70. `CLKEN` dùng để làm gì?
71. `CPOL` và `CPHA` liên quan đến gì?
72. Hãy mô tả luồng `CPU → DR → TDR → Shift Register → TX`.
73. Hãy mô tả luồng `RX → Shift Register → RDR → DR → CPU`.
74. Hãy mô tả toàn bộ quy trình cấu hình USART1 115200 8N1.
75. Vì sao phải dùng đúng GPIO Alternate Function trước khi truyền USART?
76. Vì sao receiver có thể bị ORE dù Baud Rate cấu hình đúng?
77. Vì sao error flag và NVIC pending state không phải cùng một thứ?
78. Khi parity enable, MSB ghi vào DR có được truyền nguyên vẹn không?
79. Khi parity enable ở reception, MSB đọc từ DR chứa gì?
80. Hãy phân biệt Polling, Interrupt và DMA trong USART.

---

## 6.22. Tóm tắt

Frame:

```text
Idle
 ↓
Start
 ↓
Data LSB first
 ↓
Parity nếu có
 ↓
Stop
```

8N1:

```text
8 Data
No Parity
1 Stop Bit
```

Clock:

```text
USART1
→ PCLK2

USART2/3
UART4/5
→ PCLK1
```

Baud:

```text
Baud =
fCK / (16 × USARTDIV)
```

```text
USARTDIV =
DIV_Mantissa + DIV_Fraction/16
```

Transmit:

```text
CPU / DMA
 ↓
DR / TDR
 ↓
Shift Register
 ↓
TX
```

Receive:

```text
RX
 ↓
Shift Register
 ↓
RDR / DR
 ↓
CPU / DMA
```

Flags:

```text
TXE
→ TDR trống

TC
→ frame cuối truyền xong

RXNE
→ RDR có data

IDLE
→ phát hiện Idle Line

ORE
→ Overrun

NE
→ Noise

FE
→ Framing

PE
→ Parity
```

Interrupt:

```text
USART Event
   ↓
Flag
   ↓
Interrupt Enable
   ↓
USART IRQ
   ↓
NVIC
   ↓
USARTx_IRQHandler()
```

DMA:

```text
USART
↔
DMA
↔
SRAM Buffer
```

**Điểm cần nhớ:**

> **Muốn cấu hình USART đúng phải xác định peripheral clock trước khi tính `USART_BRR`, cấu hình đúng frame và GPIO, phân biệt `TXE` với `TC`, đọc `DR` kịp thời khi `RXNE` được set, và xử lý đúng trình tự clear các receive error flag.**

[↑ Về mục lục](#muc-luc)


---

<a id="chuong-07"></a>
# 7. SPI + I2C

SPI và I2C đều là giao tiếp nối tiếp đồng bộ nhưng tổ chức bus theo hai mô hình khác nhau.

```text
SPI
├── SCK
├── MOSI
├── MISO
└── NSS / CS

I2C
├── SCL
└── SDA
```

Tư duy tổng quát:

```text
SPI
→ tốc độ cao
→ giao thức đơn giản hơn
→ chọn Slave bằng NSS/CS
→ Full-Duplex tự nhiên

I2C
→ 2 dây dùng chung
→ chọn Slave bằng Address
→ có ACK/NACK
→ hỗ trợ Arbitration và Clock Stretching
```

<a id="muc-07-01"></a>
## 7.1. Tổng quan SPI và I2C

### SPI

```text
Synchronous
Serial
Master / Slave
```

Các đặc điểm chính trên STM32F1:

```text
Full-Duplex
Half-Duplex
8-bit / 16-bit frame
MSB First / LSB First
CPOL / CPHA
Hardware / Software NSS
Interrupt
DMA
CRC
```

### I2C

```text
Synchronous
Serial
Address-based shared bus
```

Các đặc điểm chính:

```text
2-wire bus
7-bit / 10-bit address
Master / Slave
Multi-Master
ACK / NACK
Repeated START
Clock Stretching
Arbitration
Standard Mode
Fast Mode
Interrupt
DMA
```

### So sánh tư duy

```text
SPI:
Master chọn Slave bằng đường vật lý NSS/CS

I2C:
Master chọn Slave bằng địa chỉ được truyền trên SDA
```

---

# SPI

<a id="muc-07-02"></a>
## 7.2. SPI là gì?

`SPI`:

```text
Serial Peripheral Interface
```

SPI là giao tiếp nối tiếp đồng bộ.

Master cung cấp clock:

```text
Master
  ↓
 SCK
  ↓
Slave
```

Dữ liệu được shift theo từng cạnh clock.

Ứng dụng:

```text
Sensor
Display
Flash memory
ADC / DAC ngoài
RF module
Memory card
Peripheral tốc độ cao
```

STM32F1 SPI có thể hoạt động:

```text
Master
Slave
Multi-Master
```

và hỗ trợ:

```text
Full-Duplex
Simplex
Bidirectional Half-Duplex
```

---

<a id="muc-07-03"></a>
## 7.3. SCK / MOSI / MISO / NSS

Một SPI Full-Duplex thường có bốn tín hiệu.

### SCK

```text
Serial Clock
```

Master tạo SCK.

```text
Master SCK ─────────→ Slave SCK
```

### MOSI

```text
Master Out
Slave In
```

```text
Master MOSI ────────→ Slave MOSI
```

### MISO

```text
Master In
Slave Out
```

```text
Master MISO ←──────── Slave MISO
```

### NSS / CS

```text
NSS
→ Slave Select

CS
→ Chip Select
```

Dùng để chọn Slave cần giao tiếp.

```text
Master
 ├── SCK  ─────────────┬── Slave A
 ├── MOSI ─────────────┼── Slave B
 ├── MISO ←────────────┴── ...
 │
 ├── CS_A ───────────────→ Slave A
 └── CS_B ───────────────→ Slave B
```

Thông thường:

```text
CS Low
→ Slave được chọn

CS High
→ Slave không được chọn
```

mức active thực tế phải kiểm tra theo thiết bị ngoài.

---

<a id="muc-07-04"></a>
## 7.4. Master / Slave và Full-Duplex

### Master

Master:

```text
khởi tạo communication
tạo SCK
chọn Slave
```

### Slave

Slave:

```text
nhận SCK từ Master
truyền / nhận theo clock đó
```

### Full-Duplex

SPI sử dụng hai đường dữ liệu:

```text
MOSI
MISO
```

nên có thể truyền và nhận đồng thời.

Bên trong:

```text
Master Shift Register
       ↕
    MOSI / MISO
       ↕
Slave Shift Register
```

Mỗi xung clock:

```text
1 bit được shift ra
+
1 bit được shift vào
```

Do đó:

```text
Master gửi 1 byte
→ đồng thời nhận 1 byte
```

Ngay cả khi application chỉ quan tâm TX, Receive path vẫn có thể nhận dữ liệu.

Đây là lý do phải xử lý đúng:

```text
RXNE
OVR
```

trong Full-Duplex.

---

<a id="muc-07-05"></a>
## 7.5. SPI Clock và Baud Rate Prescaler

Trong Master mode:

```text
fSCK =
fPCLK
──────────
Prescaler
```

`SPI_CR1.BR[2:0]` chọn:

```text
/2
/4
/8
/16
/32
/64
/128
/256
```

Trên STM32F1:

```text
SPI1
→ APB2
→ PCLK2

SPI2
SPI3
→ APB1
→ PCLK1
```

SPI3 chỉ có trên các MCU hỗ trợ peripheral này.

### Ví dụ

```text
PCLK2 = 72 MHz
SPI1 Prescaler = /8
```

Suy ra:

```text
SCK
= 72 MHz / 8
= 9 MHz
```

### Trong Slave mode

SCK do Master bên ngoài cung cấp.

Do đó:

```text
BR[2:0]
→ không quyết định SCK khi STM32 là Slave
```

Master phải bảo đảm SCK nằm trong giới hạn của Slave và điều kiện điện/timing của hệ thống.

---

<a id="muc-07-06"></a>
## 7.6. CPOL / CPHA và 4 SPI Mode

Hai bit:

```text
CPOL
→ Clock Polarity

CPHA
→ Clock Phase
```

quyết định timing của SCK và thời điểm lấy mẫu data.

### CPOL

```text
CPOL = 0
→ SCK idle Low

CPOL = 1
→ SCK idle High
```

### CPHA

```text
CPHA = 0
→ lấy mẫu tại cạnh đầu tiên

CPHA = 1
→ lấy mẫu tại cạnh thứ hai
```

### Leading / Trailing Edge

Nếu:

```text
CPOL = 0
```

thì:

```text
Leading Edge  = Rising
Trailing Edge = Falling
```

Nếu:

```text
CPOL = 1
```

thì:

```text
Leading Edge  = Falling
Trailing Edge = Rising
```

### Bốn SPI Mode

| SPI Mode | CPOL | CPHA | SCK Idle | Sampling |
|---|---:|---:|---|---|
| Mode 0 | 0 | 0 | Low | Leading Edge |
| Mode 1 | 0 | 1 | Low | Trailing Edge |
| Mode 2 | 1 | 0 | High | Leading Edge |
| Mode 3 | 1 | 1 | High | Trailing Edge |

Nếu datasheet thiết bị ghi:

```text
SPI Mode 3
```

thì:

```text
CPOL = 1
CPHA = 1
```

Master và Slave phải sử dụng timing tương thích.

Trước khi thay đổi:

```text
CPOL
CPHA
```

nên disable SPI:

```text
SPE = 0
```

rồi mới cấu hình lại.

---

<a id="muc-07-07"></a>
## 7.7. Data Frame: 8/16-bit, MSB/LSB First

STM32F1 hỗ trợ:

```text
8-bit frame
16-bit frame
```

được chọn bởi:

```text
SPI_CR1.DFF
```

```text
DFF = 0
→ 8-bit

DFF = 1
→ 16-bit
```

Thứ tự bit được chọn bởi:

```text
SPI_CR1.LSBFIRST
```

```text
LSBFIRST = 0
→ MSB First

LSBFIRST = 1
→ LSB First
```

Cấu hình phổ biến:

```text
8-bit
MSB First
```

nhưng phải theo protocol của thiết bị ngoài.

### Ví dụ 8-bit MSB First

```text
Data = 0b10110010

truyền:
bit7
 ↓
bit6
 ↓
...
 ↓
bit0
```

---

<a id="muc-07-08"></a>
## 7.8. NSS Hardware / Software

Bit:

```text
SPI_CR1.SSM
```

chọn cách quản lý Slave Select.

### Software NSS

```text
SSM = 1
```

Trạng thái NSS nội bộ được điều khiển bởi:

```text
SSI
```

External NSS pin có thể được dùng cho mục đích khác.

Trong Master mode, một cách phổ biến là:

```text
SSM = 1
SSI = 1
```

và dùng một GPIO riêng để điều khiển Chip Select:

```text
GPIO CS = 0
→ bắt đầu transaction

GPIO CS = 1
→ kết thúc transaction
```

Ưu điểm:

```text
software kiểm soát chính xác transaction boundary
```

### Hardware NSS

```text
SSM = 0
```

Trong Master mode:

```text
SPI_CR2.SSOE = 1
```

cho phép peripheral điều khiển NSS output.

Khi SPI được enable ở Master mode:

```text
NSS được kéo Low
```

và giữ Low tới khi SPI bị disable.

Do đó Hardware NSS không phải lúc nào phù hợp với thiết bị yêu cầu:

```text
CS High
```

giữa từng command/transaction.

### Master Mode Fault

Nếu Master dùng Hardware NSS input:

```text
MSTR = 1
SSM = 0
SSOE = 0
```

và NSS bị kéo Low, SPI có thể phát hiện:

```text
MODF
→ Mode Fault
```

vì phần cứng hiểu rằng có xung đột Master trên bus.

---

<a id="muc-07-09"></a>
## 7.9. SPI_DR và cơ chế Shift Register

Thanh ghi:

```text
SPI_DR
```

được dùng cho cả Transmit và Receive.

### Transmit

```text
CPU / DMA
    ↓
Write SPI_DR
    ↓
Tx Buffer
    ↓
Shift Register
    ↓
MOSI
```

### Receive

```text
MISO
 ↓
Shift Register
 ↓
Rx Buffer
 ↓
Read SPI_DR
 ↓
CPU / DMA
```

Trong Full-Duplex:

```text
một frame được shift ra
đồng thời
một frame được shift vào
```

Do đó thao tác transfer thường là:

```text
Write DR
→ tạo / tiếp tục clock
→ chờ Receive
→ Read DR
```

Master không thể nhận dữ liệu SPI từ Slave mà không tạo clock.

Nếu muốn chỉ đọc:

```text
Master vẫn phải gửi dummy data
```

để tạo SCK.

Ví dụ:

```text
write 0xFF
→ tạo 8 xung SCK
→ đồng thời nhận 8 bit từ Slave
```

---

<a id="muc-07-10"></a>
## 7.10. TXE / RXNE / BSY

Ba flag quan trọng trong:

```text
SPI_SR
```

### TXE

```text
TXE
→ Transmit Buffer Empty
```

```text
TXE = 1
→ có thể ghi frame mới vào SPI_DR
```

### RXNE

```text
RXNE
→ Receive Buffer Not Empty
```

```text
RXNE = 1
→ có frame mới trong Rx Buffer
→ có thể đọc SPI_DR
```

### BSY

```text
BSY
→ Busy Flag
```

```text
BSY = 1
→ SPI đang thực hiện communication

BSY = 0
→ SPI không còn bận shift frame
```

### Một frame transfer

```text
wait TXE
   ↓
write SPI_DR
   ↓
Shift Register hoạt động
   ↓
wait RXNE
   ↓
read SPI_DR
```

Khi chuẩn bị:

```text
CS High
hoặc
disable SPI
```

phải bảo đảm frame cuối đã hoàn tất.

Thực tế thường kiểm tra:

```text
TXE = 1
BSY = 0
```

trước khi kết thúc transaction.

---

<a id="muc-07-11"></a>
## 7.11. OVR / MODF / CRCERR

Các lỗi SPI quan trọng:

```text
OVR
→ Overrun

MODF
→ Mode Fault

CRCERR
→ CRC Error
```

### OVR

Trong Receive / Full-Duplex:

```text
Rx Buffer còn data chưa đọc
        ↓
frame mới nhận xong
        ↓
data mới không thể chuyển bình thường
        ↓
OVR = 1
```

Nguyên nhân:

```text
software / DMA không đọc RX đủ nhanh
```

Clear OVR trên STM32F1:

```text
read SPI_DR
      ↓
read SPI_SR
```

Đây là thứ tự phải giữ đúng.

### MODF

Mode Fault liên quan đến NSS và Master mode.

Ví dụ:

```text
STM32 đang Master
      ↓
Hardware NSS input bị kéo Low
      ↓
MODF
```

Khi MODF xảy ra:

```text
MSTR có thể bị clear
SPE có thể bị clear
```

Trình tự xử lý có bước đọc `SPI_SR` rồi ghi lại `SPI_CR1` với cấu hình Master/SPI enable phù hợp.

### CRCERR

Khi SPI CRC được sử dụng:

```text
CRC nhận
≠
CRC tính toán
→ CRCERR = 1
```

CRC không bắt buộc cho mọi SPI protocol.

---

<a id="muc-07-12"></a>
## 7.12. SPI bằng Polling / Interrupt / DMA

### Polling

CPU trực tiếp kiểm tra:

```text
TXE
RXNE
BSY
```

Ví dụ một frame:

```c
static uint8_t SPI1_Transfer(uint8_t tx)
{
    while (!(SPI1->SR & SPI_SR_TXE))
    {
    }

    SPI1->DR = tx;

    while (!(SPI1->SR & SPI_SR_RXNE))
    {
    }

    return (uint8_t)SPI1->DR;
}
```

### Interrupt

Các bit:

```text
TXEIE
→ TX buffer empty interrupt

RXNEIE
→ RX buffer not empty interrupt

ERRIE
→ error interrupt
```

Luồng:

```text
SPI Event
 ↓
Flag
 ↓
Interrupt Enable
 ↓
SPI IRQ
 ↓
NVIC
 ↓
SPIx_IRQHandler()
```

### DMA

Các bit:

```text
TXDMAEN
RXDMAEN
```

Luồng Full-Duplex:

```text
TX Buffer in SRAM
      ↓ DMA
    SPI_DR
      ↓
     MOSI

     MISO
      ↓
    SPI_DR
      ↓ DMA
RX Buffer in SRAM
```

Với Full-Duplex DMA, thường cần cấu hình cả:

```text
TX DMA
RX DMA
```

để hai hướng được phục vụ đồng thời.

---

<a id="muc-07-13"></a>
## 7.13. GPIO cho SPI

### SPI Master

Cấu hình phổ biến:

```text
SCK
→ Alternate Function Push-Pull

MOSI
→ Alternate Function Push-Pull

MISO
→ Input Floating / Pull-Up

NSS
→ AF Push-Pull nếu Hardware NSS
  hoặc GPIO Output nếu Software Chip Select
```

### SPI1 mặc định

```text
PA4 → NSS
PA5 → SCK
PA6 → MISO
PA7 → MOSI
```

### SPI1 Remap

```text
PA15 → NSS
PB3  → SCK
PB4  → MISO
PB5  → MOSI
```

Các chân:

```text
PA15
PB3
PB4
```

liên quan JTAG sau reset.

Nếu dùng SPI1 remap, có thể cần:

```text
JTAG disabled
SWD retained
```

qua `AFIO_MAPR.SWJ_CFG`.

### SPI Slave

Ở Slave mode, hướng GPIO phụ thuộc tín hiệu:

```text
SCK
→ Input

MOSI
→ Input

MISO
→ Alternate Function Output

NSS
→ Input nếu Hardware NSS
```

---

<a id="muc-07-14"></a>
## 7.14. Quy trình cấu hình SPI Master

Ví dụ:

```text
SPI1
Master
Full-Duplex
8-bit
MSB First
Mode 0
SCK = PCLK2 / 8
Software NSS
```

### Bước 1 — Bật Clock

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN
               | RCC_APB2ENR_SPI1EN;
```

### Bước 2 — GPIO

```text
PA5 → SCK  → AF Push-Pull
PA6 → MISO → Input
PA7 → MOSI → AF Push-Pull
CS  → GPIO Output
```

### Bước 3 — Disable SPI trước khi cấu hình

```c
SPI1->CR1 &= ~SPI_CR1_SPE;
```

### Bước 4 — Master

```c
SPI1->CR1 |= SPI_CR1_MSTR;
```

### Bước 5 — Clock Prescaler

Ví dụ:

```text
BR = /8
```

### Bước 6 — CPOL / CPHA

Mode 0:

```text
CPOL = 0
CPHA = 0
```

### Bước 7 — Frame Format

```text
DFF = 0
→ 8-bit

LSBFIRST = 0
→ MSB First
```

### Bước 8 — Software NSS

```text
SSM = 1
SSI = 1
```

### Bước 9 — Full-Duplex

```text
BIDIMODE = 0
RXONLY   = 0
```

### Bước 10 — Enable SPI

```c
SPI1->CR1 |= SPI_CR1_SPE;
```

### Luồng

```text
RCC
 ↓
GPIO
 ↓
SPE = 0
 ↓
MSTR
 ↓
BR
 ↓
CPOL / CPHA
 ↓
DFF / LSBFIRST
 ↓
SSM / SSI
 ↓
SPE = 1
```

---

<a id="muc-07-15"></a>
## 7.15. Ví dụ một SPI Transaction

Giả sử thiết bị cần:

```text
CS Low
 ↓
Command
 ↓
Address
 ↓
Read Data
 ↓
CS High
```

Ví dụ khái niệm:

```c
static uint8_t SPI1_Transfer(uint8_t tx)
{
    while (!(SPI1->SR & SPI_SR_TXE))
    {
    }

    SPI1->DR = tx;

    while (!(SPI1->SR & SPI_SR_RXNE))
    {
    }

    return (uint8_t)SPI1->DR;
}
```

Transaction:

```c
CS_LOW();

SPI1_Transfer(command);
SPI1_Transfer(address);

uint8_t data = SPI1_Transfer(0xFFU);

while (SPI1->SR & SPI_SR_BSY)
{
}

CS_HIGH();
```

Điểm cần hiểu:

```text
SPI1_Transfer(0xFF)
```

không phải chỉ là gửi `0xFF`.

Nó đồng thời:

```text
gửi dummy byte
+
tạo SCK
+
nhận byte từ Slave
```

---

# I2C

<a id="muc-07-16"></a>
## 7.16. I2C là gì?

`I2C`:

```text
Inter-Integrated Circuit
```

I2C là bus nối tiếp đồng bộ hai dây.

```text
SCL
→ Serial Clock

SDA
→ Serial Data
```

Một bus có thể có:

```text
Master
Slave 1
Slave 2
Slave 3
...
```

Các thiết bị dùng chung:

```text
SCL
SDA
```

và được phân biệt bằng:

```text
Address
```

STM32F1 I2C hỗ trợ:

```text
Master
Slave
Multi-Master
7-bit Address
10-bit Address
General Call
Standard Mode
Fast Mode
Clock Stretching
Interrupt
DMA
```

---

<a id="muc-07-17"></a>
## 7.17. SDA / SCL và Open-Drain

Trên STM32F1:

```text
I2C_SCL
→ Alternate Function Open-Drain

I2C_SDA
→ Alternate Function Open-Drain
```

Bus cần pull-up.

```text
 VDD                  VDD
  │                    │
 RPU                  RPU
  │                    │
 SCL                  SDA
  │                    │
  ├── Master           ├── Master
  ├── Slave A          ├── Slave A
  └── Slave B          └── Slave B
```

Mỗi thiết bị chỉ:

```text
kéo line xuống Low
hoặc
nhả line về High-Z
```

Mức High được tạo bởi:

```text
Pull-Up
```

### Vì sao Open-Drain?

Open-Drain cho phép nhiều thiết bị dùng chung line mà không có tình huống một thiết bị chủ động kéo High trong khi thiết bị khác kéo Low.

Nó là cơ sở cho:

```text
ACK / NACK
Clock Stretching
Arbitration
Multi-Master
```

### Pull-Up thực tế

Giá trị điện trở Pull-Up phụ thuộc:

```text
Bus capacitance
SCL frequency
Supply voltage
Rise-time requirement
Số thiết bị
```

Không nên mặc định một giá trị duy nhất cho mọi bus.

---

<a id="muc-07-18"></a>
## 7.18. START / STOP / Address / R/W / ACK / NACK

Một I2C transaction thường có:

```text
START
  ↓
Address + R/W
  ↓
ACK
  ↓
Data
  ↓
ACK
  ↓
...
  ↓
NACK / ACK
  ↓
STOP
```

### START

Trong trạng thái bus rỗi:

```text
SCL = High
SDA = High
```

START được tạo khi:

```text
SDA: High → Low
trong lúc
SCL = High
```

### STOP

STOP được tạo khi:

```text
SDA: Low → High
trong lúc
SCL = High
```

### Byte

Data và Address truyền:

```text
MSB First
```

Mỗi byte gồm:

```text
8 data/address bits
+
1 ACK/NACK bit
```

Tổng:

```text
9 SCL pulses
```

### ACK

Receiver kéo SDA Low ở xung thứ 9:

```text
ACK
→ byte được nhận
```

### NACK

Receiver không kéo SDA Low:

```text
NACK
→ không acknowledge
```

Trong Master Receiver, NACK thường được dùng với byte cuối để báo:

```text
không nhận thêm data
```

---

<a id="muc-07-19"></a>
## 7.19. 7-bit / 10-bit Addressing

### 7-bit Address

Address phase:

```text
[A6 A5 A4 A3 A2 A1 A0 R/W]
```

Bit cuối:

```text
R/W = 0
→ Master Transmitter / Write

R/W = 1
→ Master Receiver / Read
```

Ví dụ Slave có:

```text
7-bit address = 0x50
```

Address byte trên bus:

```text
Write:
(0x50 << 1) | 0
= 0xA0

Read:
(0x50 << 1) | 1
= 0xA1
```

Phải phân biệt:

```text
7-bit Slave Address
≠
8-bit Address Byte
```

Đây là lỗi cấu hình phổ biến khi datasheet của thiết bị và API sử dụng cách biểu diễn address khác nhau.

### 10-bit Address

STM32F1 cũng hỗ trợ 10-bit addressing.

Luồng có thêm:

```text
10-bit Header
+
phần địa chỉ còn lại
```

10-bit mode cần các event như:

```text
ADD10
```

nhưng 7-bit addressing nên được nắm chắc trước.

---

<a id="muc-07-20"></a>
## 7.20. Master / Slave / Transmitter / Receiver

I2C STM32F1 có bốn trạng thái chính:

```text
Master Transmitter
Master Receiver
Slave Transmitter
Slave Receiver
```

### Master Transmitter

```text
STM32 tạo START
→ gửi Address + Write
→ gửi Data
→ tạo STOP
```

### Master Receiver

```text
STM32 tạo START
→ gửi Address + Read
→ nhận Data
→ ACK / NACK
→ tạo STOP
```

### Slave Transmitter

```text
Master Address + Read
→ STM32 được chọn
→ STM32 gửi Data
```

### Slave Receiver

```text
Master Address + Write
→ STM32 được chọn
→ STM32 nhận Data
```

Các bit trong `I2C_SR2` giúp nhận biết trạng thái:

```text
MSL
→ Master / Slave

TRA
→ Transmitter / Receiver

BUSY
→ Bus đang bận
```

---

<a id="muc-07-21"></a>
## 7.21. Standard Mode / Fast Mode

STM32F1 hỗ trợ:

```text
Standard Mode
→ tối đa 100 kHz

Fast Mode
→ tối đa 400 kHz
```

### Standard Mode

```text
Sm
≤ 100 kHz
```

Peripheral input clock tối thiểu:

```text
2 MHz
```

### Fast Mode

```text
Fm
≤ 400 kHz
```

Peripheral input clock tối thiểu:

```text
4 MHz
```

Fast Mode có hai tỷ lệ duty:

```text
DUTY = 0
→ tLOW / tHIGH = 2

DUTY = 1
→ tLOW / tHIGH = 16 / 9
```

Tốc độ thực tế còn phụ thuộc:

```text
Pull-Up
Bus capacitance
Rise time
Clock Stretching
```

---

<a id="muc-07-22"></a>
## 7.22. I2C Clock: CR2.FREQ / CCR / TRISE

I2C1/I2C2 nằm trên:

```text
APB1
```

nên timing dựa trên:

```text
PCLK1
```

Ba cấu hình quan trọng:

```text
I2C_CR2.FREQ
I2C_CCR
I2C_TRISE
```

### CR2.FREQ

`FREQ` chứa tần số peripheral input clock theo MHz.

Ví dụ:

```text
PCLK1 = 36 MHz
```

thì:

```text
FREQ = 36
```

Không phải:

```text
FREQ = 100 kHz
```

### CCR — Standard Mode

Standard Mode có:

```text
tHIGH = CCR × TPCLK1
tLOW  = CCR × TPCLK1
```

Suy ra:

```text
fSCL =
PCLK1
──────────
2 × CCR
```

Ví dụ:

```text
PCLK1 = 36 MHz
fSCL  = 100 kHz
```

Ta có:

```text
CCR
= 36 MHz / (2 × 100 kHz)
= 180
```

### CCR — Fast Mode DUTY = 0

Với:

```text
tLOW / tHIGH = 2
```

công thức:

```text
fSCL =
PCLK1
──────────
3 × CCR
```

### CCR — Fast Mode DUTY = 1

Với:

```text
tLOW / tHIGH = 16 / 9
```

công thức:

```text
fSCL =
PCLK1
───────────
25 × CCR
```

### TRISE

`TRISE` cho phép I2C timing tính tới maximum rise time của SCL.

Trong Standard Mode, với đơn vị MHz:

```text
TRISE = FREQ + 1
```

Ví dụ:

```text
PCLK1 = 36 MHz

TRISE = 37
```

Trong Fast Mode, giá trị dựa trên maximum rise time 300 ns và chu kỳ PCLK1.

### Ví dụ 100 kHz với PCLK1 = 36 MHz

```text
FREQ = 36
CCR  = 180
TRISE = 37
```

---

<a id="muc-07-23"></a>
## 7.23. Các I2C Status Flag quan trọng

I2C STM32F1 sử dụng hai Status Register:

```text
I2C_SR1
I2C_SR2
```

### I2C_SR1

Các event flag quan trọng:

```text
SB
→ Start Bit generated

ADDR
→ Address sent / matched

ADD10
→ 10-bit header sent

STOPF
→ Stop detected trong Slave mode

BTF
→ Byte Transfer Finished

RxNE
→ Receive Data Register Not Empty

TxE
→ Transmit Data Register Empty
```

Error:

```text
BERR
→ Bus Error

ARLO
→ Arbitration Lost

AF
→ Acknowledge Failure

OVR
→ Overrun / Underrun

PECERR
→ PEC Error
```

### I2C_SR2

Các trạng thái quan trọng:

```text
MSL
→ Master mode

BUSY
→ Bus đang bận

TRA
→ Transmitter mode

GENCALL
→ General Call Address nhận được

DUALF
→ xác định Own Address nào matched
```

### Flag Clear Sequence

I2C STM32F1 có nhiều flag cần một **trình tự đọc/ghi cụ thể**.

#### SB

```text
SB = 1
```

clear bằng:

```text
read SR1
    ↓
write DR với address
```

#### ADDR

```text
ADDR = 1
```

clear bằng:

```text
read SR1
    ↓
read SR2
```

#### STOPF

Trong Slave mode:

```text
read SR1
    ↓
write CR1
```

#### RxNE

```text
read DR
```

#### TxE

```text
write DR
```

Đây là lý do driver I2C bare-metal phải giữ đúng thứ tự thao tác register.

---

<a id="muc-07-24"></a>
## 7.24. Clock Stretching

Clock Stretching cho phép Slave kéo:

```text
SCL = Low
```

để trì hoãn Master.

Luồng:

```text
Master muốn tiếp tục clock
        ↓
Slave chưa sẵn sàng
        ↓
Slave giữ SCL Low
        ↓
Master chờ
        ↓
Slave nhả SCL
        ↓
communication tiếp tục
```

Trong STM32F1, một số event còn làm peripheral giữ SCL Low trong lúc chờ software xử lý.

Ví dụ:

```text
ADDR
BTF
```

có thể liên quan tới clock stretching tùy mode và trạng thái.

Clock Stretching là một cơ chế tự nhiên của I2C vì các line sử dụng Open-Drain.

---

<a id="muc-07-25"></a>
## 7.25. Arbitration và Multi-Master

I2C hỗ trợ nhiều Master trên cùng bus.

Ví dụ:

```text
Master A ─┐
          ├── SCL / SDA ── Slaves
Master B ─┘
```

Hai Master có thể cùng bắt đầu transaction.

Arbitration dựa trên SDA:

```text
Master gửi bit
      ↓
đồng thời đọc SDA
      ↓
so sánh mức thực tế
```

Nếu Master cố gửi:

```text
1
```

nhưng bus thực tế là:

```text
0
```

thì Master đó mất arbitration.

STM32 set:

```text
ARLO
→ Arbitration Lost
```

Sau khi mất arbitration:

```text
STM32 không tiếp tục làm Master
→ chuyển về trạng thái phù hợp để bus tiếp tục hoạt động
```

I2C dùng Open-Drain nên:

```text
Low
→ dominant

High
→ line được nhả
```

điều này cho phép arbitration mà không gây xung đột điện kiểu Push-Pull.

---

<a id="muc-07-26"></a>
## 7.26. Repeated START

Repeated START là START được tạo khi Master đã đang giữ bus.

Sơ đồ:

```text
START
 ↓
Address + W
 ↓
ACK
 ↓
Register Address
 ↓
ACK
 ↓
Repeated START
 ↓
Address + R
 ↓
ACK
 ↓
Data
 ↓
NACK
 ↓
STOP
```

Đây là mẫu đọc register rất phổ biến.

Ví dụ sensor có:

```text
Slave Address = 0x68
Register      = 0x75
```

Master cần:

```text
1. START
2. 0x68 + Write
3. gửi 0x75
4. Repeated START
5. 0x68 + Read
6. nhận data
7. NACK
8. STOP
```

Trên STM32F1:

```text
START bit được set khi đang Master
→ tạo Repeated START sau byte hiện tại
```

---

<a id="muc-07-27"></a>
## 7.27. Master Transmit / Master Receive

### Master Transmit

Luồng:

```text
BUSY = 0
  ↓
START = 1
  ↓
SB = 1
  ↓
read SR1
  ↓
write Address + W vào DR
  ↓
ADDR = 1
  ↓
read SR1
read SR2
  ↓
write Data vào DR
  ↓
TxE / BTF
  ↓
write byte tiếp theo
  ↓
...
  ↓
BTF
  ↓
STOP = 1
```

Pseudocode:

```c
/* START */
I2C1->CR1 |= I2C_CR1_START;
while (!(I2C1->SR1 & I2C_SR1_SB))
{
}

/* Address + Write */
(void)I2C1->SR1;
I2C1->DR = (slave_addr << 1);

while (!(I2C1->SR1 & I2C_SR1_ADDR))
{
}

/* Clear ADDR */
(void)I2C1->SR1;
(void)I2C1->SR2;

/* Data */
while (!(I2C1->SR1 & I2C_SR1_TXE))
{
}
I2C1->DR = data;

while (!(I2C1->SR1 & I2C_SR1_BTF))
{
}

/* STOP */
I2C1->CR1 |= I2C_CR1_STOP;
```

### Master Receive

Master Receive khó hơn vì ACK/NACK và STOP phải được cấu hình đúng thời điểm.

Nguyên tắc:

```text
byte chưa phải cuối
→ ACK

byte cuối
→ NACK

sau đó
→ STOP
```

### Receive 1 byte

Trình tự khái niệm trên STM32F1:

```text
START
 ↓
Address + R
 ↓
ADDR = 1
 ↓
ACK = 0
 ↓
clear ADDR
 ↓
STOP = 1
 ↓
wait RxNE
 ↓
read DR
```

Điểm quan trọng:

```text
ACK phải được clear trước khi clear ADDR
```

để byte duy nhất được NACK đúng lúc.

### Receive 2 byte

STM32F1 dùng sequence đặc biệt với:

```text
POS
ACK
BTF
```

Khái niệm:

```text
POS = 1
ACK = 0
clear ADDR
      ↓
wait BTF
      ↓
STOP = 1
      ↓
read DR
read DR
```

### Receive nhiều hơn 2 byte

Khi còn nhiều byte:

```text
ACK = 1
→ nhận liên tục
```

Khi tiến gần byte cuối:

```text
dùng BTF
clear ACK đúng thời điểm
generate STOP
đọc các byte cuối theo sequence
```

Do sequence 1 byte, 2 byte và `N > 2` khác nhau, không nên dùng một trình tự đơn giản duy nhất cho mọi độ dài Receive.

---

<a id="muc-07-28"></a>
## 7.28. I2C Interrupt / DMA

I2C có hai nhóm interrupt chính.

### Event Interrupt

Bit:

```text
ITEVFEN
```

Các event:

```text
SB
ADDR
BTF
STOPF
...
```

### Buffer Interrupt

Bit:

```text
ITBUFEN
```

liên quan:

```text
TxE
RxNE
```

### Error Interrupt

Bit:

```text
ITERREN
```

các lỗi:

```text
BERR
ARLO
AF
OVR
PECERR
...
```

I2C thường có vector:

```text
I2Cx_EV_IRQn
→ Event IRQ

I2Cx_ER_IRQn
→ Error IRQ
```

### DMA

I2C hỗ trợ DMA để truyền/nhận data qua `I2C_DR`.

Luồng:

```text
I2C_DR
  ↕
 DMA
  ↕
SRAM Buffer
```

DMA giảm số lần CPU phải xử lý:

```text
TxE
RxNE
```

nhưng software vẫn phải quản lý đúng:

```text
START
Address
ACK / NACK
STOP
Repeated START
Error state
```

---

<a id="muc-07-29"></a>
## 7.29. Quy trình cấu hình I2C

Ví dụ:

```text
I2C1
Master
Standard Mode
100 kHz
PCLK1 = 36 MHz
```

### Bước 1 — Bật Clock

```c
RCC->APB2ENR |= RCC_APB2ENR_IOPBEN;
RCC->APB1ENR |= RCC_APB1ENR_I2C1EN;
```

Nếu remap:

```text
AFIO clock
+
AFIO_MAPR
```

### Bước 2 — GPIO

Mặc định I2C1:

```text
PB6 → SCL
PB7 → SDA
```

Cả hai:

```text
Alternate Function Open-Drain
```

và bus cần pull-up phù hợp.

### Bước 3 — Disable Peripheral khi cấu hình timing

```c
I2C1->CR1 &= ~I2C_CR1_PE;
```

### Bước 4 — CR2.FREQ

```text
PCLK1 = 36 MHz

FREQ = 36
```

### Bước 5 — CCR

Standard Mode 100 kHz:

```text
CCR
= 36 MHz / (2 × 100 kHz)
= 180
```

### Bước 6 — TRISE

Standard Mode:

```text
TRISE
= FREQ + 1
= 37
```

### Bước 7 — Enable I2C

```c
I2C1->CR1 |= I2C_CR1_PE;
```

### Bước 8 — Communication

```text
check BUSY
 ↓
START
 ↓
SB
 ↓
Address
 ↓
ADDR
 ↓
Data / Receive
 ↓
STOP
```

### Luồng cấu hình

```text
RCC
 ↓
GPIO Open-Drain
 ↓
Pull-Up
 ↓
PE = 0
 ↓
CR2.FREQ
 ↓
CCR
 ↓
TRISE
 ↓
PE = 1
 ↓
START / Address / Data / STOP
```

---

<a id="muc-07-30"></a>
## 7.30. So sánh SPI và I2C

| Đặc điểm | SPI | I2C |
|---|---|---|
| Đồng bộ | Có | Có |
| Clock | SCK | SCL |
| Data | MOSI + MISO | SDA |
| Full-Duplex tự nhiên | Có | Không |
| Số dây cơ bản | 3 + NSS/CS | 2 |
| Chọn Slave | NSS/CS | Address |
| Output Driver | Push-Pull phổ biến | Open-Drain |
| Pull-Up bắt buộc theo bus | Không | Có |
| Multiple Slave | Mỗi Slave thường cần CS | Dùng chung bus, khác Address |
| Addressing | Không phải phần cốt lõi của SPI | 7-bit / 10-bit |
| ACK/NACK | Không | Có |
| Repeated START | Không | Có |
| Clock Stretching | Không | Có |
| Arbitration | Không theo cơ chế I2C | Có |
| Full-Duplex Shift | Có | Không |
| STM32F1 Speed | phụ thuộc PCLK/Prescaler | Sm 100 kHz, Fm 400 kHz |
| Protocol State Machine | Đơn giản hơn | Phức tạp hơn |

### Chọn SPI khi

```text
cần tốc độ cao
ít Slave
có đủ chân
giao thức thiết bị hỗ trợ SPI
```

### Chọn I2C khi

```text
muốn ít dây
nhiều peripheral dùng chung bus
thiết bị có I2C Address
tốc độ 100/400 kHz phù hợp
```

---

<a id="muc-07-31"></a>
## 7.31. Câu hỏi tự kiểm tra

### SPI

1. SPI viết tắt của gì?
2. SPI là synchronous hay asynchronous?
3. Master và Slave khác nhau ở vai trò tạo SCK thế nào?
4. MOSI có ý nghĩa gì?
5. MISO có ý nghĩa gì?
6. NSS/CS dùng để làm gì?
7. Vì sao SPI Full-Duplex có thể truyền và nhận đồng thời?
8. Khi Master gửi một byte, Receive path có hoạt động không?
9. SPI1 lấy clock từ bus nào?
10. SPI2/SPI3 lấy clock từ bus nào?
11. Các SPI Master prescaler của STM32F1 là gì?
12. Với PCLK2=72 MHz và `/8`, SCK bằng bao nhiêu?
13. CPOL điều khiển gì?
14. CPHA điều khiển gì?
15. SPI Mode 0 tương ứng CPOL/CPHA nào?
16. SPI Mode 3 tương ứng CPOL/CPHA nào?
17. `DFF=0` là frame bao nhiêu bit?
18. `DFF=1` là frame bao nhiêu bit?
19. `LSBFIRST=0` nghĩa là gì?
20. `SSM` dùng để làm gì?
21. `SSI` dùng để làm gì?
22. `SSOE` dùng để làm gì?
23. Hardware NSS có hạn chế gì khi thiết bị cần CS toggle cho từng transaction?
24. `SPI_DR` có vai trò gì?
25. `TXE` có nghĩa gì?
26. `RXNE` có nghĩa gì?
27. `BSY` có nghĩa gì?
28. Vì sao Master muốn Read vẫn phải transmit dummy data?
29. OVR xảy ra khi nào?
30. Clear OVR bằng trình tự nào?
31. MODF là lỗi gì?
32. CRCERR là lỗi gì?
33. SPI Interrupt có những nguồn cơ bản nào?
34. SPI DMA có thể dùng ở cả TX và RX không?
35. GPIO SCK/MOSI của Master thường dùng mode nào?
36. MISO Master thường dùng mode nào?
37. SPI1 mặc định dùng những chân nào?
38. SPI1 remap dùng những chân nào?
39. Vì sao SPI1 remap có thể liên quan JTAG?
40. Trước khi CS High sau frame cuối nên kiểm tra gì?

### I2C

41. I2C viết tắt của gì?
42. I2C dùng mấy dây tín hiệu chính?
43. SDA dùng để làm gì?
44. SCL dùng để làm gì?
45. Vì sao SDA/SCL dùng Open-Drain?
46. Vì sao bus I2C cần Pull-Up?
47. START Condition là gì?
48. STOP Condition là gì?
49. I2C truyền MSB First hay LSB First?
50. Mỗi byte I2C có bao nhiêu clock nếu tính ACK/NACK?
51. ACK là gì?
52. NACK là gì?
53. 7-bit Address và Address Byte khác nhau thế nào?
54. Address 0x50 khi Write tạo byte nào trên bus?
55. Address 0x50 khi Read tạo byte nào trên bus?
56. STM32F1 hỗ trợ 10-bit Address không?
57. Bốn trạng thái Master/Slave Transmit/Receive là gì?
58. Standard Mode tối đa bao nhiêu?
59. Fast Mode tối đa bao nhiêu?
60. I2C1/I2C2 nằm trên bus nào?
61. `CR2.FREQ` chứa giá trị gì?
62. Với PCLK1=36 MHz, `FREQ` bằng bao nhiêu?
63. Công thức CCR trong Standard Mode là gì?
64. Với PCLK1=36 MHz và 100 kHz, CCR bằng bao nhiêu?
65. TRISE Standard Mode bằng bao nhiêu khi PCLK1=36 MHz?
66. `SB` có nghĩa gì?
67. `ADDR` có nghĩa gì?
68. `TxE` có nghĩa gì?
69. `RxNE` có nghĩa gì?
70. `BTF` có nghĩa gì?
71. `BUSY` nằm trong register nào?
72. Clear `ADDR` bằng trình tự nào?
73. Clear `SB` bằng trình tự nào?
74. Clear `RxNE` bằng cách nào?
75. `BERR` là lỗi gì?
76. `ARLO` là lỗi gì?
77. `AF` là lỗi gì?
78. Clock Stretching là gì?
79. Arbitration hoạt động dựa trên nguyên tắc gì?
80. Vì sao Low là mức dominant trên bus I2C?
81. Repeated START dùng khi nào?
82. Hãy mô tả sequence đọc một register của sensor.
83. Master Transmit bắt đầu từ những event nào?
84. Vì sao Master Receive 1 byte và nhiều byte có sequence khác nhau?
85. Khi nhận byte cuối, Master dùng ACK hay NACK?
86. Event Interrupt và Error Interrupt khác nhau thế nào?
87. I2C DMA thay CPU xử lý phần data nhưng vẫn cần software quản lý những phần nào?
88. GPIO của I2C1 mặc định là chân nào?
89. I2C1 remap sang chân nào?
90. Hãy mô tả đầy đủ luồng `RCC → GPIO → FREQ → CCR → TRISE → START → Address → Data → STOP`.

### So sánh

91. SPI chọn Slave bằng gì?
92. I2C chọn Slave bằng gì?
93. Giao tiếp nào Full-Duplex tự nhiên?
94. Giao tiếp nào hỗ trợ Clock Stretching?
95. Giao tiếp nào hỗ trợ Arbitration theo protocol?
96. Vì sao I2C phù hợp khi cần nhiều thiết bị nhưng ít chân?
97. Vì sao SPI thường đạt tốc độ cao hơn và state machine đơn giản hơn?
98. Khi thiết kế driver SPI, ba flag nào cần nhớ nhất?
99. Khi thiết kế driver I2C STM32F1, vì sao thứ tự đọc/ghi `SR1/SR2/DR/CR1` quan trọng?
100. Hãy so sánh một transaction đọc register bằng SPI với I2C.

---

## 7.32. Tóm tắt

### SPI

```text
Master
├── SCK
├── MOSI
├── MISO
└── NSS / CS
```

Clock:

```text
fSCK =
fPCLK / Prescaler
```

Timing:

```text
CPOL + CPHA
→ SPI Mode 0 / 1 / 2 / 3
```

Data:

```text
DFF
→ 8 / 16 bit

LSBFIRST
→ MSB / LSB First
```

Flags:

```text
TXE
→ TX buffer trống

RXNE
→ RX buffer có data

BSY
→ SPI đang bận

OVR
→ Receive Overrun
```

Full-Duplex:

```text
Write DR
  ↓
shift TX
+
shift RX
  ↓
Read DR
```

### I2C

```text
SCL
SDA
→ Open-Drain
→ Pull-Up
```

Transaction:

```text
START
 ↓
Address + R/W
 ↓
ACK
 ↓
Data
 ↓
ACK / NACK
 ↓
STOP
```

Timing:

```text
PCLK1
 ↓
CR2.FREQ
 ↓
CCR
 ↓
TRISE
 ↓
SCL
```

Flags:

```text
SB
ADDR
TxE
RxNE
BTF
BUSY
BERR
ARLO
AF
```

Flag sequence:

```text
SB
→ read SR1
→ write DR

ADDR
→ read SR1
→ read SR2
```

Repeated START:

```text
Write Register Address
        ↓
Repeated START
        ↓
Read Data
```

**Điểm cần nhớ:**

> **SPI cần xác định đúng `SCK`, `CPOL/CPHA`, frame format và luôn nhớ Full-Duplex nghĩa là truyền đồng thời với nhận. I2C cần hiểu đúng Open-Drain, Address, START/STOP, ACK/NACK, Repeated START và đặc biệt phải giữ đúng trình tự xử lý các flag `SB`, `ADDR`, `BTF`, `RxNE`, `TxE` của state machine STM32F1.**

[↑ Về mục lục](#muc-luc)
