# Trình bày dự án LoRa IoT Temperature Monitoring

## Phần 1. Giới thiệu tổng quan

Dự án xây dựng hệ thống IoT giám sát nhiệt độ từ xa qua LoRa, sử dụng **3 nút cảm biến STM32F103** (tổng cộng **18 cảm biến nhiệt độ NTC**) kết hợp với **ESP32 Gateway**. Dữ liệu được truyền về máy chủ để lưu trữ và hiển thị trên Web Dashboard.

## Phần 2. Kiến trúc và cách hoạt động

Hệ thống có luồng dữ liệu chính như sau:

**STM32 Nodes → LoRa SX1278 → ESP32 Gateway → MQTT Broker → Backend ↔ InfluxDB**  
**Backend → REST API → Web Dashboard**

- **STM32 Nodes:** Hệ thống gồm 3 nút; mỗi nút sử dụng 1 STM32F103, 1 module LoRa SX1278 và 6 cảm biến nhiệt độ NTC. Trong đó, một nút sử dụng thêm 3 IC khuếch đại LM318 và ADC ngoài MCP3208. Các nút thu thập, xử lý dữ liệu nhiệt độ và truyền đến Gateway qua LoRa.
- **ESP32 Gateway:** Sử dụng ESP32 và module LoRa SX1278 để điều phối giao tiếp với các nút, lưu đệm dữ liệu bằng LittleFS và chuyển tiếp dữ liệu đến MQTT Broker qua Wi-Fi.
- **Backend:** Xây dựng bằng Node.js và Express, có nhiệm vụ tiếp nhận, kiểm tra và xử lý dữ liệu từ MQTT Broker; lưu đệm dữ liệu khi cần và cung cấp REST API cho Web Dashboard.
- **InfluxDB:** Cơ sở dữ liệu chuỗi thời gian, lưu trữ dữ liệu nhiệt độ, thời điểm đo và trạng thái cảm biến để phục vụ truy vấn dữ liệu hiện tại và lịch sử.
- **Web Dashboard:** Hiển thị nhiệt độ, biểu đồ lịch sử, trạng thái các nút và tình trạng của từng cảm biến.

## Phần 3. Công việc thực hiện

- **STM32:** Phát triển firmware thu thập dữ liệu từ 18 cảm biến NTC qua ADC nội và ADC ngoài MCP3208; thực hiện lọc tín hiệu, tính nhiệt độ và phát hiện lỗi cảm biến.
- **LoRa:** Xây dựng định dạng bản tin gồm `START`, `ADDR`, `TYPE`, `SEQ`, `LEN`, `PAYLOAD`, `CRC8`; triển khai ACK, Timeout và truyền lại bản tin để tăng độ tin cậy.
- **ESP32 Gateway:** Sử dụng FreeRTOS để quản lý giao tiếp LoRa/MQTT và lưu đệm dữ liệu bằng LittleFS khi mất kết nối.
- **Tích hợp hệ thống:** Kết nối MQTT Broker, Node.js Backend, InfluxDB và React Dashboard để quản lý, lưu trữ và hiển thị dữ liệu.

## Phần 4. Kết quả đạt được

- Xây dựng hệ thống giám sát từ xa **18 kênh nhiệt độ trên 3 nút STM32** qua LoRa.
- Hỗ trợ xác nhận, truyền lại và nhận diện bản tin trùng lặp nhằm hạn chế mất dữ liệu trong quá trình truyền.
- Có khả năng lưu đệm và khôi phục dữ liệu khi kết nối Wi-Fi/MQTT gián đoạn hoặc Gateway khởi động lại.
- Hoàn thiện Web Dashboard hiển thị nhiệt độ, biểu đồ lịch sử, RSSI và cảnh báo lỗi cảm biến.

---

# STM32 Node

> **Phạm vi:** Phân tích firmware của ba STM32 Sensor Node, tập trung vào những gì cần để hiểu luồng chương trình và mã nguồn. Node 01/02 dùng ADC1 + DMA; Node 03 dùng ADC ngoài MCP3208. Phần xử lý nhiệt độ, phát hiện lỗi và giao thức LoRa có nhiều logic tương đồng, vì vậy chỉ phân tích điểm chung một lần.

## Phần 1. Kiến trúc Firmware và luồng hoạt động chính

### 1.1. Các module và trách nhiệm

| Mã nguồn | Vai trò trong chương trình |
|---|---|
| `main.c` | Khởi tạo hệ thống, nhận `POLL`, gọi xử lý cảm biến, đóng gói `DATA` và chờ `ACK`. |
| `adc_driver.c/.h` (Node 01/02) | Lấy mẫu ADC1 + DMA, lọc, tính nhiệt độ và quản lý lỗi cảm biến. |
| `mcp3208.c/.h` (Node 03) | Đọc ADC ngoài qua SPI; xử lý nhiệt độ và lỗi theo luồng tương tự. |
| `sx1278.c/.h`, `spi_driver.c/.h` | Điều khiển SX1278, tạo/nhận frame LoRa, truyền byte qua SPI. |
| `systick_driver.c/.h`, `systick_utils.c/.h` | Cung cấp mốc thời gian, delay và timeout để phối hợp các thao tác. |

Firmware STM32 Node chạy bằng **vòng lặp chính**, không sử dụng FreeRTOS; FreeRTOS thuộc ESP32 Gateway.

### 1.2. Luồng xử lý từ `main()` đến khi gửi dữ liệu

Thứ tự khởi tạo được triển khai như sau:

- **Node 01/02:** `SysTick_Init()` → `SX1278_Init()` → `ADC_GPIO_Init()` → `ADC_DMA_Init()`.
- **Node 03:** `SysTick_Init()` → `SX1278_Init()` → `MCP3208_Init()`.

Sau khởi tạo, chương trình chờ bản tin `POLL` từ Gateway và xử lý theo luồng:

```text
Nhận frame LoRa
      │
      ├─ Không hợp lệ / không đúng ADDR, TYPE=POLL, LEN=0 → Bỏ qua
      ▼
Yêu cầu dữ liệu cảm biến (cập nhật nếu đã đủ chu kỳ)
      ▼
Lọc ADC → Tính nhiệt độ → Xác định lỗi → Tính trung bình
      ▼
Đóng gói DATA (giữ SEQ của POLL)
      ▼
Gửi DATA → Chờ ACK đúng ADDR, TYPE, SEQ, LEN
      ├─ ACK hợp lệ → Hoàn tất giao dịch
      └─ Thất bại / timeout → Thử gửi lại (tối đa 3 lần tổng cộng)
```

Ba kiểm tra đầu vào quan trọng là: `ADDR` phải là địa chỉ Node, `TYPE` phải là `POLL` và `LEN` phải bằng `0`. `SEQ` gắn `POLL`, `DATA` và `ACK` với **cùng một giao dịch**. Node không tự phát dữ liệu theo lịch độc lập mà phản hồi yêu cầu của Gateway.

## Phần 2. Thu thập dữ liệu từ cảm biến NTC

> **Cách đọc sơ đồ:** Tài liệu phân biệt (1) **kết nối được firmware/README xác nhận**, (2) **cách mắc suy ra từ công thức đo**, và (3) **mạch ví dụ để hiểu nguyên lý**. Code cho biết chân MCU và cách tính ADC nhưng không thay thế được sơ đồ nguyên lý của bo mạch thực tế. Các sơ đồ ASCII bên dưới thể hiện **mạng điện**, không thể hiện số chân vật lý của linh kiện NTC hoặc đầu nối bo mạch.

### 2.1. Node 01 và Node 02 – Nối từng NTC, điện trở 10 kΩ và chân ADC

**Một kênh đo (S1) theo chiều phân áp tương ứng công thức firmware:**

```text
  Nguồn VBIAS  ──────┐
                     │
                [R1 = 10 kΩ]    ← điện trở cố định
                     │
                     ●────────────── PA0 (STM32F103, ADC1_IN0)
                     │
                [NTC1 = 10 kΩ @ 25 °C]
                     │
  GND ───────────────┘
```

**Cách nối cụ thể cho S1:** một đầu **R1 = 10 kΩ** nối nguồn phân áp `VBIAS`; đầu còn lại của R1 nối **đồng thời** với một đầu NTC1 và chân **PA0**. Đầu NTC1 còn lại nối **GND**. Điểm nối giữa R1 và NTC1 chính là điện áp đo `V_NTC1`. Những kênh còn lại lặp lại đúng cấu trúc này, mỗi kênh có **một điện trở 10 kΩ riêng**.

**Sơ đồ đủ 6 kênh của một Node 01/02:**

```text
  VBIAS ── R1 10k ──●── NTC1 ── GND       ● ── PA0 / ADC1_IN0
  VBIAS ── R2 10k ──●── NTC2 ── GND       ● ── PA1 / ADC1_IN1
  VBIAS ── R3 10k ──●── NTC3 ── GND       ● ── PA2 / ADC1_IN2
  VBIAS ── R4 10k ──●── NTC4 ── GND       ● ── PA3 / ADC1_IN3
  VBIAS ── R5 10k ──●── NTC5 ── GND       ● ── PB0 / ADC1_IN8
  VBIAS ── R6 10k ──●── NTC6 ── GND       ● ── PB1 / ADC1_IN9

  Lưu ý: ở mỗi dòng, hai dấu ● biểu diễn CÙNG MỘT điểm nối.
  Sáu kênh dùng chung nguồn VBIAS và mass GND, nhưng không nối
  chung các điểm giữa phân áp của những cảm biến khác nhau.
```

| Kênh | Đầu 1 điện trở 10 kΩ | Đầu 2 điện trở 10 kΩ (điểm giữa) | Đầu NTC còn lại | STM32 đọc điểm giữa |
|---|---|---|---|---|
| S1 / R1 / NTC1 | `VBIAS` | `V_NTC1` | GND | PA0 / ADC1_IN0 |
| S2 / R2 / NTC2 | `VBIAS` | `V_NTC2` | GND | PA1 / ADC1_IN1 |
| S3 / R3 / NTC3 | `VBIAS` | `V_NTC3` | GND | PA2 / ADC1_IN2 |
| S4 / R4 / NTC4 | `VBIAS` | `V_NTC4` | GND | PA3 / ADC1_IN3 |
| S5 / R5 / NTC5 | `VBIAS` | `V_NTC5` | GND | PB0 / ADC1_IN8 |
| S6 / R6 / NTC6 | `VBIAS` | `V_NTC6` | GND | PB1 / ADC1_IN9 |

**Vì sao mắc như vậy?** Firmware Node 01/02 dùng:

```text
R_NTC = 10000 × ADC_raw / (4095 − ADC_raw)
```

Với điện trở cố định **phía nguồn** và NTC **phía GND**, ta có `V_NTC / VBIAS = R_NTC / (10000 + R_NTC)`. Công thức code tương ứng với trường hợp điện áp nguồn phân áp `VBIAS` bằng điện áp tham chiếu ADC `VREF_ADC`. **Nếu bo mạch sử dụng nguồn phân áp 3,3 V và ADC tham chiếu 3,3 V**, có thể đặt `VBIAS = 3,3 V`; không được mặc định hai điện áp này bằng nhau khi chưa đo/đối chiếu sơ đồ thật. Giá trị ADC khoảng 2048 (nửa thang 12-bit) tương ứng gần 10 kΩ và 25 °C trong điều kiện trên.

**Điểm đã xác nhận:** các chân PA0, PA1, PA2, PA3, PB0, PB1; điện trở cố định 10 kΩ; NTC 10 kΩ, Beta 3950. **Điểm suy ra:** chiều lắp R cố định ở trên và NTC ở dưới. File README/source chưa xác nhận đầu nối và sơ đồ cấp nguồn thực tế của từng cảm biến.

### 2.2. Node 01/02 – Nối STM32 với SX1278 và thu thập bằng DMA

**Kết nối STM32F103 ↔ SX1278 theo firmware:**

| Chân/tín hiệu trên module SX1278 | Nối STM32F103 | Chức năng |
|---|---|---|
| SCK / SCLK | PA5 (SPI1_SCK) | Xung SPI |
| MISO / DOUT | PA6 (SPI1_MISO) | SX1278 → STM32 |
| MOSI / DIN | PA7 (SPI1_MOSI) | STM32 → SX1278 |
| NSS / CS | PA4 (GPIO) | Chọn SX1278 |
| RESET / RST | PB10 (GPIO) | Reset SX1278 |
| VCC | Nguồn **3,3 V phù hợp với module** | Cấp nguồn vô tuyến; cần kiểm tra module đang dùng |
| GND | GND chung STM32 | Mass tham chiếu |

Firmware đang điều khiển SX1278 bằng cách đọc/ghi thanh ghi qua SPI và **thăm dò cờ trạng thái**; tài liệu này không tự thêm kết nối DIO0/DIO1 khi code chưa yêu cầu. Vị trí chân vật lý trên module SX1278 phụ thuộc loại module (Ra-01, Ra-02 hoặc breakout), nên **đấu theo ký hiệu chân in trên module**, không suy đoán số thứ tự chân từ ảnh một biến thể khác.

ADC1 liên tục quét 6 kênh `CH0, CH1, CH2, CH3, CH8, CH9`. DMA1 Channel 1 ghi vòng vào:

```c
volatile uint16_t g_adcRawBuffer[NUMBER_CHANNEL]; // NUMBER_CHANNEL = 6
```

Mỗi phần tử là **mẫu ADC mới nhất của một kênh**, không phải bộ 16 mẫu. Khi xử lý dữ liệu, `AcquireFilteredRawSamples()` đọc lặp 16 lượt (mỗi lượt sao chép đủ 6 giá trị) để tạo `samples[6][16]`, sau đó lọc Trimmed Mean.

### 2.3. Node 03 – Nối 6 NTC với các chân MCP3208

Node 03 dùng 6 NTC 10 kΩ và **MCP3208 12-bit** thay ADC nội. Trong code, cảm biến logic S1–S6 tương ứng các kênh MCP3208 CH0–CH5:

| Cảm biến logic | Tín hiệu sau mạch analog đưa tới | Chân vật lý MCP3208 |
|---|---|---:|
| NTC1 / S1 | CH0 | **1** |
| NTC2 / S2 | CH1 | **2** |
| NTC3 / S3 | CH2 | **3** |
| NTC4 / S4 | CH3 | **4** |
| NTC5 / S5 | CH4 | **5** |
| NTC6 / S6 | CH5 | **6** |

**Điểm quan trọng:** Bảng trên là **ánh xạ kênh mà firmware đọc**. Nếu một cảm biến đi qua LM318, điện áp tới chân MCP3208 là **đầu ra tầng analog**, không phải nối trực tiếp dây NTC vào MCP3208. Repo chưa chỉ rõ kênh vật lý nào qua từng LM318.

**Mỗi NTC vẫn cần mạch biến đổi điện trở thành điện áp.** Mô hình phần mềm của Node 03 tiếp tục sử dụng R cố định 10 kΩ và giả thiết tỷ số phân áp:

```text
  VBIAS ── [R_FIXED 10 kΩ] ──●── [NTC 10 kΩ] ── GND
                             │
                             └── V_NTC ──► [Tầng analog nếu có]
                                             │
                                             └──► MCP3208 CHx
```

Firmware có `OPAMP_GAIN = 1.15` và tính `ratio_at_ntc = (ADC_raw / 4095) / 1.15`, tức **giả định biên độ ADC đã được khuếch đại 1,15 lần**. Repo **không xác nhận** chiều mắc các điện trở của bo Node 03, số kênh có LM318 và các điện trở hồi tiếp. Vì vậy, sơ đồ này mô tả **cách firmware dự kiến đọc tín hiệu**, không phải bản vẽ đấu nối 3 LM318 đã kiểm chứng.

### 2.4. Node 03 – Nối MCP3208 với STM32 và SX1278

**Bảng đầy đủ MCP3208 16 chân (theo pinout datasheet):**

| Số chân MCP3208 | Tên chân | Nối đến / vai trò |
|---:|---|---|
| **1** | CH0 | Tín hiệu đo S1 sau khối analog |
| **2** | CH1 | Tín hiệu đo S2 sau khối analog |
| **3** | CH2 | Tín hiệu đo S3 sau khối analog |
| **4** | CH3 | Tín hiệu đo S4 sau khối analog |
| **5** | CH4 | Tín hiệu đo S5 sau khối analog |
| **6** | CH5 | Tín hiệu đo S6 sau khối analog |
| **7** | CH6 | Không được firmware sử dụng |
| **8** | CH7 | Không được firmware sử dụng |
| **9** | DGND | Mass số, tham chiếu chung với STM32 |
| **10** | CS/SHDN | **PB0** của STM32 (CS riêng MCP3208) |
| **11** | DIN | **PA7** của STM32 (SPI1_MOSI) |
| **12** | DOUT | **PA6** của STM32 (SPI1_MISO) |
| **13** | CLK | **PA5** của STM32 (SPI1_SCK) |
| **14** | AGND | Mass analog, kết nối tham chiếu đất đúng thiết kế ADC |
| **15** | VREF | Điện áp tham chiếu ADC; kiểm tra thực tế trên bo |
| **16** | VDD | Nguồn MCP3208; kiểm tra thực tế trên bo |

**Đường SPI dùng chung với SX1278 ở Node 03:**

```text
 STM32 PA5 (SPI1_SCK)  ─────────┬── MCP3208 chân 13 (CLK)
                                └── SX1278 SCK
 STM32 PA6 (SPI1_MISO) ─────────┬── MCP3208 chân 12 (DOUT)
                                └── SX1278 MISO
 STM32 PA7 (SPI1_MOSI) ─────────┬── MCP3208 chân 11 (DIN)
                                └── SX1278 MOSI

 STM32 PB0 (GPIO) ───────────────── MCP3208 chân 10 (CS/SHDN)
 STM32 PA4 (GPIO) ───────────────── SX1278 NSS/CS
 STM32 PB10 (GPIO) ──────────────── SX1278 RESET

 MCP3208 chân 9 (DGND) / 14 (AGND) ─ GND thiết kế của Node
 SX1278 GND ───────────────────────── GND chung Node
```

Khi truy cập MCP3208, firmware kéo **PB0 xuống** để chọn ADC ngoài và giữ SX1278 không được chọn. Khi truy cập SX1278, firmware điều khiển **PA4** và bỏ chọn MCP3208. MCP3208 hỗ trợ cấp nguồn 2,7–5,5 V theo datasheet, nhưng phải kiểm tra **mức logic phù hợp với STM32 3,3 V** và giới hạn điện áp các đầu vào analog. Việc ghi `VDD = 3,3 V`, `VREF = 3,3 V` chỉ được xác nhận sau khi đo hoặc xem sơ đồ bo thực tế, không suy ra từ tên chip.

### 2.5. Node 03 – IC LM318: chân nào nối điện trở nào?

Bạn cung cấp pinout **LM318 loại 8 chân**. Với một IC LM318, các chân có ý nghĩa:

| Chân LM318 | Tên | Vai trò |
|---:|---|---|
| **1** | COMP-1 / BAL | Chân bù/chỉnh theo cấu hình datasheet |
| **2** | IN− | Ngõ vào đảo; thường nhận mạng hồi tiếp ở cấu hình không đảo |
| **3** | IN+ | Ngõ vào không đảo; có thể nhận tín hiệu `V_NTC` |
| **4** | VCC− | Nguồn âm/nguồn dưới của op-amp |
| **5** | COMP-3 / BAL | Chân bù/chỉnh theo datasheet |
| **6** | OUTPUT | Đầu ra analog của IC |
| **7** | VCC+ | Nguồn dương của op-amp |
| **8** | COMP-2 | Chân bù/chỉnh theo datasheet |

**Ví dụ lý thuyết cách mắc hai điện trở hồi tiếp để đạt hệ số khuếch đại 1,15** (cấu hình *không đảo*; **không phải sơ đồ đã xác nhận của Node 03**):

```text
  NTC / điểm giữa phân áp ─────────────► LM318 chân 3 (IN+)

                          LM318 chân 6 (OUT) ───► V_OUT
                                   │
                               [Rf = 3 kΩ]
                                   │
  LM318 chân 2 (IN−) ◄──────────────●
                                   │
                              [Rg = 20 kΩ]
                                   │
                                  GND*   (*mốc tham chiếu phù hợp mạch nguồn)

  LM318 chân 7 = VCC+  → nguồn dương đúng datasheet
  LM318 chân 4 = VCC−  → nguồn âm/nguồn dưới đúng datasheet
  LM318 chân 1,5,8     → tùy mạng bù theo thiết kế và datasheet
```

Khi hệ số khuếch đại vòng kín hoạt động đúng miền tuyến tính:

`Gain = 1 + Rf / Rg = 1 + 3 kΩ / 20 kΩ = 1,15`.

**Không dùng ví dụ này để đấu thẳng lên bo khi chưa kiểm tra:** LM318 **không phải op-amp nguồn đơn 3,3 V**: datasheet TI ghi dải nguồn **±5 V đến ±20 V**. Cần thiết kế nguồn phù hợp, xem dải common-mode/đầu ra và giới hạn đầu vào MCP3208. Nếu `V_OUT` vượt `VREF`/giới hạn của MCP3208, có thể sai số, bão hòa hoặc gây hỏng IC. Ngoài ra, **ba LM318 là ba kênh op-amp đơn**, trong khi Node có sáu NTC: không thể suy ra 3 IC đang khuếch đại kênh S1–S3 hay mắc kiểu khác chỉ từ pinout và mã nguồn.

**Kết luận kỹ thuật:** Có thể ghi chính xác **tên/số chân LM318 và nguyên tắc mắc điện trở hồi tiếp**, nhưng để ghi **“LM318 số 1 chân 6 nối MCP3208 chân mấy; Rf/Rg thực tế bao nhiêu; chân 7/4 dùng nguồn bao nhiêu”** thì cần **sơ đồ nguyên lý hoặc ảnh mạch thực tế đủ rõ**. Không được trình bày sơ đồ ví dụ ở trên thành cấu hình đã có của dự án.

### 2.6. Node 03 – Đọc dữ liệu từ MCP3208

Hàm `MCP3208_ReadChannel(channel)` kéo CS (PB0) xuống, gửi lệnh chọn CH0–CH5 qua SPI, lấy 12 bit ADC từ byte nhận rồi nhả CS. Driver đọc đủ sáu kênh, lặp 16 lượt để lọc giống Node 01/02. Khác biệt chính: Node 01/02 nhận giá trị ADC mới qua **DMA**, còn Node 03 **chủ động đọc từng kênh MCP3208 qua SPI**.

**Nguồn kiểm chứng:** [README Node 01](https://github.com/haikevins/lora-iot-temperature-monitoring/blob/main/node01/README.md), [README Node 03](https://github.com/haikevins/lora-iot-temperature-monitoring/blob/main/node03/README.md), `node01/USER/adc_driver.c`, `node03/USER/mcp3208.c/.h`; [datasheet Microchip MCP3208](https://ww1.microchip.com/downloads/en/DeviceDoc/21298e.pdf), [datasheet TI LM318](https://www.ti.com/lit/ds/symlink/lm318.pdf). Các kết nối còn thiếu sơ đồ PCB được đánh dấu riêng, không tự suy đoán.

## Phần 3. Xử lý tín hiệu và tính toán nhiệt độ

### 3.1. Lọc Trimmed Mean và phát hiện độ phân tán

Mỗi kênh sử dụng **16 mẫu ADC**. Firmware sắp xếp mẫu từ nhỏ đến lớn, bỏ **4 giá trị nhỏ nhất** và **4 giá trị lớn nhất**, sau đó lấy trung bình **8 giá trị còn lại**:

```text
filteredRaw ≈ (x[4] + x[5] + ... + x[11]) / 8
rawSpread    = x[15] - x[0]      (x đã sắp xếp tăng dần)
```

- `filteredRaw` giảm tác động của những mẫu nhiễu đột biến.
- `rawSpread` tính từ **đủ 16 mẫu ban đầu**, dùng để phát hiện nhiễu; nếu `rawSpread > 100` ADC counts thì đánh dấu `SIGNAL_NOISY`.

Hai phép tính phục vụ hai mục đích khác nhau: **lọc kết quả** và **đánh giá chất lượng đợt đo**.

### 3.2. Chuyển đổi ADC → điện trở → nhiệt độ

Đối với Node 01/02, firmware tính điện trở NTC theo mạch phân áp được sử dụng trong dự án:

```text
R_NTC = 10000 × rawAdc / (4095 − rawAdc)
```

Sau đó dùng mô hình **Beta 3950** (`R25 = 10 kΩ`, `T25 = 298,15 K`):

```text
1/T(K) = 1/298,15 + ln(R_NTC/10000)/3950
T(°C) = T(K) − 273,15
```

Ví dụ, khi `R_NTC = 10 kΩ` thì `T ≈ 25 °C`. Firmware phải loại các giá trị đầu vào không hợp lệ như ADC ở biên, tránh phép chia không xác định hoặc phép tính logarit sai.

**Riêng Node 03:** Do dữ liệu ADC được đọc sau mạch khuếch đại, code khai báo `OPAMP_GAIN = 1.15` và bù theo tỷ số:

```text
ratio_at_ntc = (rawAdc / 4095) / 1,15
R_NTC = 10000 × ratio_at_ntc / (1 − ratio_at_ntc)
```

Sau bước này, cả ba Node tiếp tục dùng mô hình Beta và các bước xử lý nhiệt độ tương tự nhau.

### 3.3. Calibration, EMA và nhiệt độ trung bình

**Hiệu chuẩn từng kênh:**

```text
T_calibrated[i] = T_model[i] × scale[i] + offset[i]
```

Trong mã nguồn hiện tại, `scale = 1` và `offset = 0` ở cả sáu kênh: **đã có cơ chế hiệu chuẩn trong code nhưng chưa có hệ số hiệu chuẩn thực nghiệm riêng được áp dụng**.

**Lọc EMA** (`alpha = 0.25`), chỉ cập nhật khi mẫu hiện tại không có lỗi quan sát:

```text
T_filtered(new) = T_filtered(old) + 0,25 × [T_calibrated − T_filtered(old)]
```

Lần đầu có mẫu tốt, lấy mẫu đó làm giá trị khởi tạo. Nếu mẫu mới có lỗi, không đưa nó vào EMA mà giữ giá trị lọc trước đó.

**Tính nhiệt độ trung bình:** `ADC_GetStableAverageTemp()` hoặc `MCP3208_GetStableAverageTemp()` lấy trung bình các kênh **đã khởi tạo** và **không có lỗi đã xác nhận** (`latchedFaults == 0`). Nếu không còn kênh hợp lệ, driver trả `TEMP_ERROR_VALUE = -1000.0f`; khi đóng gói LoRa, giá trị không hợp lệ được biểu diễn bằng marker **`0x8000`**, không được hiểu là nhiệt độ thực.

## Phần 4. Phát hiện và quản lý lỗi cảm biến

### 4.1. Các nhóm lỗi quan trọng

Firmware sử dụng **8 nhóm lỗi**, nhưng có thể nhớ theo mục đích kiểm tra:

| Nhóm | Mã lỗi | Điều kiện chính trong code |
|---|---|---|
| Đầu vào ADC | `SHORT`, `OPEN` / `HIGH_SAT` | ADC quá thấp (`≤ 20`) hoặc quá cao (`≥ 4075`). |
| Chất lượng tín hiệu | `SIGNAL_NOISY` | Độ phân tán 16 mẫu `> 100` counts. |
| Miền giá trị / mô hình | `RESISTANCE`, `TEMP_RANGE`, `MODEL` | Điện trở/nhiệt độ ngoài phạm vi; phép chuyển đổi không hợp lệ. |
| Tính hợp lý theo thời gian | `RATE` | Nhiệt độ thay đổi `> 10 °C/s` trong khoảng thời gian so sánh hợp lệ. |
| So sánh giữa cảm biến | `CROSS_SENSOR` | Khi đủ tối thiểu 4 kênh ứng viên, một kênh lệch trung vị `> 12 °C`. |

Phạm vi nhiệt độ và điện trở ở Node 03 khác Node 01/02; khi cần debug ngưỡng cụ thể, xem `adc_driver.h` và `mcp3208.h`. `CROSS_SENSOR` dùng **trung vị** thay vì trung bình để hạn chế tác động của kênh lệch lớn.

### 4.2. Lỗi tức thời, lỗi xác nhận và ảnh hưởng đến kết quả

Mỗi cảm biến quản lý hai trạng thái:

- `observedFaults`: lỗi phát hiện ở **lượt cập nhật hiện tại**.
- `latchedFaults`: lỗi đã được **xác nhận qua nhiều lượt**; ảnh hưởng đến trạng thái chính thức và phép tính trung bình.

Firmware duy trì bộ đếm **riêng cho từng loại lỗi**:

```text
Phát hiện cùng lỗi 3 lượt liên tiếp → Chốt lỗi (FAULT_ASSERT_COUNT = 3)
Hết lỗi 5 lượt liên tiếp       → Xóa lỗi chốt (FAULT_CLEAR_COUNT = 5)
```

Nhờ vậy, một giá trị nhiễu nhất thời không lập tức loại cảm biến. Khi một kênh có lỗi vừa được quan sát, mẫu lỗi không cập nhật EMA; khi lỗi đã chốt, kênh bị loại khỏi nhiệt độ trung bình.

**Hai lớp bitmask được gửi về Gateway:**

- `STATUS` (1 byte): bit 0–5 ứng với **S1–S6**; bit bằng `1` khi kênh chưa có mẫu tốt hoặc có lỗi đã chốt.
- `FAULT_S1`…`FAULT_S6` (6 byte): mỗi byte là bitmask cho **loại lỗi cụ thể** đã chốt của cảm biến tương ứng.

Ví dụ `STATUS = 0x02` nghĩa là cảm biến **S2** đang không hợp lệ. Cần phân biệt trạng thái *kênh nào lỗi* (`STATUS`) với *lỗi thuộc loại gì* (`FAULT_Sn`).

## Phần 5. Giao tiếp LoRa và giao thức truyền dữ liệu

### 5.1. SX1278 và cấu trúc frame

`sx1278.c` đảm nhận điều khiển radio (thanh ghi, FIFO, chế độ TX/RX); `spi_driver.c` truyền byte SPI; `main.c` điều khiển trình tự trao đổi bản tin. Frame ứng dụng được dùng trong dự án:

```text
START | ADDR | TYPE | SEQ | LEN | PAYLOAD | CRC8
  1B     1B     1B    1B    1B    LEN B      1B
```

| Trường | Ý nghĩa |
|---|---|
| `START` | Byte bắt đầu cố định `0xAA`. |
| `ADDR` | Địa chỉ Node: `0x01`, `0x02`, `0x03`. |
| `TYPE` | `POLL=0x23`, `DATA=0x01`, `ACK=0x10`. |
| `SEQ` | Số thứ tự giao dịch do Gateway đưa ra trong POLL. |
| `LEN`, `PAYLOAD` | Độ dài và nội dung dữ liệu; POLL/ACK có `LEN=0`. |
| `CRC8` | Kiểm tra tính toàn vẹn của frame ứng dụng. |

**CRC8** dùng polynomial `0x07`, giá trị khởi tạo `0x00` và tính trên `ADDR | TYPE | SEQ | LEN | PAYLOAD` (**không** gồm `START`). Ngoài CRC8 do firmware tự tính, SX1278 còn có **CRC payload tại lớp radio**. CRC phát hiện lỗi truyền ngẫu nhiên, không phải cơ chế bảo mật/xác thực.

### 5.2. Nội dung DATA Payload (13 byte)

Điểm cần nhớ: Node xử lý **6 cảm biến nhưng chỉ gửi 1 nhiệt độ trung bình**, kèm thông tin chẩn đoán.

| Byte | Dữ liệu | Cách hiểu |
|---|---|---|
| `0–1` | `TEMP_L`, `TEMP_H` | `int16_t` little-endian, nhiệt độ trung bình × 100; `0x8000` là không hợp lệ. |
| `2` | `STATUS` | Bitmask trạng thái 6 cảm biến. |
| `3–8` | `FAULT_S1`…`FAULT_S6` | Mỗi cảm biến có 1 byte mã lỗi. |
| `9–12` | `sampleAgeMs` | Tuổi mẫu đo (ms), `uint32_t` little-endian. |

Ví dụ `27,35 °C` được mã hóa thành `2735 = 0x0AAF`, hai byte đầu là `0xAF 0x0A`.

`sampleAgeMs = millis_now − measurementTimeMs` cho biết dữ liệu đã được lấy cách đây bao lâu. Nếu phải truyền lại, firmware **giữ nguyên dữ liệu đo/SEQ nhưng cập nhật tuổi mẫu** trước lần gửi mới để Gateway có thể ước lượng thời điểm đo.

### 5.3. POLL → DATA → ACK, timeout và truyền lại

```text
Gateway ── POLL(ADDR, SEQ=N) ──► STM32 Node
Gateway ◄─ DATA(ADDR, SEQ=N) ─── STM32 Node
Gateway ── ACK(ADDR, SEQ=N) ──► STM32 Node
```

Một giao dịch được coi là thành công chỉ khi ACK có **đúng `ADDR`, `TYPE=ACK`, `SEQ` và `LEN=0`**. Cần phân biệt `TX_DONE` (radio đã phát xong) với `ACK` (Gateway xác nhận đã nhận/xử lý giao dịch).

| Cấu hình | Giá trị |
|---|---:|
| Chờ nhận POLL | `2000 ms` |
| Timeout phát DATA | `1000 ms` |
| Chờ ACK | `500 ms` |
| Số lần thử gửi tối đa | **3 lần, tính cả lần đầu** |

Nếu phát thất bại hoặc không nhận đúng ACK, Node thử gửi lại cùng giao dịch. Khi ACK bị mất, Gateway có thể nhận cùng DATA nhiều lần; **Gateway**, không phải STM32 Node, chịu trách nhiệm nhận diện và tránh lưu trùng bản ghi. Sau khi hết ba lần gửi mà vẫn không có ACK hợp lệ, Node quay lại chờ POLL mới.

---

# ESP32 Gateway

> **Phạm vi:** Phân tích những cơ chế cần thiết để hiểu firmware Gateway trong `gateway/src/main.cpp`: FreeRTOS, điều phối LoRa, lưu dữ liệu an toàn vào LittleFS và chuyển tiếp bằng MQTT QoS 1. Không lặp lại lý thuyết về SPI, Wi-Fi, FreeRTOS hoặc cấu trúc frame đã giải thích ở phần STM32 Node.

## Phần 1. Kiến trúc Firmware và FreeRTOS

### 1.1. Trình tự khởi tạo và hai Task chính

Trong `setup()`, firmware khởi tạo SPI/SX1278, cấu hình LoRa, gọi `InitializePersistentOutbox()` để khôi phục hàng đợi LittleFS, sau đó cấu hình Wi-Fi và tạo hai Task bằng `xTaskCreatePinnedToCore()`:

| Task | Core | Nhiệm vụ |
|---|---:|---|
| `LoraTask` | 0 | Lần lượt gửi `POLL` đến các STM32 Node, nhận/kiểm tra `DATA`, lưu bền vững và gửi `ACK`. |
| `MqttTask` | 1 | Duy trì Wi-Fi/MQTT, đọc dữ liệu đã lưu và gửi đến Broker bằng MQTT QoS 1. |

Cả hai Task có cùng mức ưu tiên `1`; hàm Arduino `loop()` chỉ gọi `vTaskDelay(portMAX_DELAY)` vì công việc ứng dụng nằm trong hai Task trên.

**Điểm thiết kế quan trọng:** `setup()` **không đợi Wi-Fi/MQTT kết nối** rồi mới chạy LoRa. Vì vậy, khi mạng gián đoạn, `LoraTask` vẫn có thể nhận dữ liệu và ghi vào LittleFS. Tuy nhiên, nếu không khởi tạo được LoRa hoặc hàng đợi LittleFS, `setup()` dừng trong vòng lặp lỗi, không tạo các Task.

### 1.2. Đồng bộ truy cập LittleFS bằng Mutex

Hai Task cùng sử dụng hàng đợi trên Flash: `LoraTask` **thêm** bản ghi, còn `MqttTask` **đọc và xóa** bản ghi sau khi gửi thành công. Firmware dùng `g_outboxMutex` với `xSemaphoreTake()` / `xSemaphoreGive()` để bảo vệ các thao tác và biến quản lý hàng đợi.

```text
LoraTask ── PersistentOutboxPush() ──┐
                                     ├── g_outboxMutex ── LittleFS /outbox
MqttTask ── Peek() / Pop() ──────────┘
```

Mutex ngăn hai Task đồng thời thay đổi trạng thái hàng đợi gây sai lệch thứ tự, bộ đếm hoặc file dữ liệu. **Mutex không thay thế LittleFS**: Mutex bảo vệ truy cập đồng thời, còn LittleFS giữ dữ liệu sau mất điện/khởi động lại.

## Phần 2. Điều phối LoRa và xử lý giao dịch với STM32

### 2.1. Polling tuần tự và kiểm tra DATA

`LoraTask()` duyệt ba địa chỉ `0x01 → 0x02 → 0x03` theo **round-robin**, mỗi Node có biến sequence riêng trong `s_nextSequence[]`. Với mỗi lượt, Gateway:

1. Gửi `POLL` kèm `ADDR` và `SEQ` của Node hiện tại (`SendPoll()`).
2. Chờ `DATA` trong tối đa `DATA_WAIT_TIMEOUT_MS = 1000 ms`.
3. `ReceiveProtocolPacket()` kiểm tra cấu trúc frame và CRC8; lớp ứng dụng đối chiếu `ADDR`, `SEQ`, `TYPE=DATA`, độ dài payload, cùng tính nhất quán giữa `STATUS` và mã lỗi chi tiết.
4. Giải mã nhiệt độ (`int16_t`, đơn vị 0,01 °C), bitmask cảm biến, mã lỗi và `sampleAgeMs`; lấy RSSI ngay lúc nhận bằng `LoRa.packetRssi()`.
5. **Chỉ khi ghi LittleFS thành công** mới phát `ACK` và kết thúc giao dịch.

```text
POLL Node 01 → DATA → Lưu LittleFS → ACK
       ↓
POLL Node 02 → DATA → Lưu LittleFS → ACK
       ↓
POLL Node 03 → DATA → Lưu LittleFS → ACK
       └───────────── Lặp lại ─────────────┘
```

Sau mỗi lượt, Task trì hoãn `NODE_CYCLE_DELAY_MS = 1000 ms` trước khi chọn Node tiếp theo. Nếu không nhận được DATA hợp lệ hoặc ghi dữ liệu thất bại, Gateway không coi lượt đó là một giao dịch nhận thành công.

### 2.2. ACK bị mất và nhận diện bản tin trùng lặp

Giả sử Gateway đã ghi DATA vào LittleFS và gửi ACK nhưng STM32 **không nhận được ACK**. Node sẽ truyền lại DATA với cùng `ADDR`/`SEQ`. Để không lưu cùng một đợt đo hai lần, sau khi xử lý DATA thành công, `AcknowledgeDuplicateRetries()` tiếp tục nghe trong **700 ms**:

```text
DATA lần đầu → Lưu 1 bản ghi → ACK bị mất
DATA gửi lại (cùng ADDR, SEQ)
             → Nhận diện bản tin lặp
             → Gửi lại ACK, KHÔNG thêm bản ghi mới
```

**Giới hạn cần hiểu:** Đây là cơ chế nhận diện retry **trong cửa sổ 700 ms của giao dịch hiện tại**, không phải cơ chế loại trùng toàn cục trong mọi lần khởi động. Việc tránh xử lý trùng ở các tầng sau còn phụ thuộc `Record ID` và Backend.

## Phần 3. LittleFS và cơ chế Persist-Before-ACK

### 3.1. LittleFS là gì?

**LittleFS (Little File System)** là **hệ thống tệp (file system) dành cho vi điều khiển**, giúp firmware tổ chức dữ liệu thành tệp và thư mục trên **bộ nhớ Flash**. Thay vì phải tự quản lý địa chỉ Flash và từng lần xóa/ghi, ứng dụng sử dụng các thao tác quen thuộc như `open()`, `write()`, `read()`, `rename()` và `remove()`.

LittleFS phù hợp với Gateway vì có ba đặc điểm:

- **Lưu trữ bền vững:** dữ liệu đã ghi thành công nằm trong Flash, không biến mất chỉ vì ESP32 reset hoặc mất Wi-Fi (khác với biến và hàng đợi chỉ nằm trong RAM).
- **Chống mất điện khi đang cập nhật:** sử dụng cơ chế **Copy-on-Write** và cập nhật metadata có khả năng phục hồi, nhằm tránh làm hỏng toàn bộ hệ thống tệp nếu mất nguồn giữa thao tác.
- **Wear Leveling:** phân bổ hoạt động ghi/xóa qua các khối Flash để giảm hao mòn tập trung; việc sử dụng Flash vẫn có giới hạn về dung lượng và tuổi thọ.

**Cần phân biệt:** LittleFS **không phải** một bộ nhớ Flash riêng, cũng không phải giao thức mạng hay hàng đợi FIFO có sẵn. Nó là lớp quản lý tệp trên phân vùng Flash; **firmware Gateway tự xây dựng hàng đợi** dựa trên các tệp được lưu trong LittleFS.

### 3.2. LittleFS hoạt động thế nào trong Gateway?

**Ở mức hệ thống tệp**, khi ứng dụng ghi/cập nhật một tệp, LittleFS quản lý các khối Flash, metadata và cơ chế ghi theo kiểu Copy-on-Write; chỉ sau khi một thay đổi được xác nhận hoàn tất, trạng thái mới trở thành trạng thái hợp lệ. Nếu mất nguồn giữa chừng, nó được thiết kế để phục hồi về trạng thái nhất quán đã xác nhận trước đó. Đây là cơ chế của **LittleFS**, khác với các bước kiểm tra bản ghi do code ứng dụng tự thực hiện.

**Ở mức mã nguồn Gateway** (`gateway/src/main.cpp`), quy trình sử dụng như sau:

```text
Khởi động ESP32
    ↓
Mount LittleFS trên phân vùng Flash → mở/tạo thư mục /outbox
    ↓
Nhận DATA hợp lệ từ STM32
    ↓
Ghi bản ghi vào file tạm .tmp → flush/close
    ↓
Đổi tên thành file .rec → đọc lại, kiểm tra CRC32
    ↓
Bản ghi đã lưu: có thể gửi ACK cho STM32
    ↓
MQTT kết nối: đọc file .rec cũ nhất → Publish QoS 1
    ↓
Có PUBACK hợp lệ → xóa file .rec tương ứng
```

Các hàm cần nhận biết: `MountTelemetryFilesystem()` (mount LittleFS), `WritePersistentRecordFile()` (ghi và xác minh tệp), `ScanPersistentOutbox()` (khôi phục sau reboot) và `PersistentOutboxPush/Peek/Pop()` (quản lý hàng đợi). Khi mount thất bại, code **không tự format phân vùng đang có dữ liệu**; chỉ cho phép format ở lần đầu khi xác định phân vùng còn trống.

**Ví dụ:** Gateway nhận một bản đo lúc Wi-Fi mất. Bản ghi vẫn được lưu dưới `/outbox` trên Flash. Sau khi Wi-Fi phục hồi hoặc ESP32 khởi động lại, `MqttTask` đọc lại bản ghi còn tồn và gửi MQTT. Nếu mất điện **trước khi tệp được ghi/xác minh thành công**, bản ghi mới có thể chưa được lưu; vì vậy Gateway không gửi ACK cho STM32.

### 3.3. Vì sao phải lưu trước khi gửi ACK?

Đây là **nguyên tắc cốt lõi** của Gateway:

```text
STM32 gửi DATA
      ↓
Gateway kiểm tra dữ liệu
      ↓
Tạo TelemetryMessage
      ↓
PersistentOutboxPush() → Ghi và xác minh LittleFS
      │
      ├─ Thành công → Gửi ACK cho STM32
      └─ Thất bại / đầy bộ nhớ → KHÔNG gửi ACK
```

Nếu Gateway ACK ngay rồi mới thử lưu, khi mất điện ở giữa hai bước, STM32 sẽ tưởng dữ liệu đã được tiếp nhận trong khi Gateway chưa giữ bản sao. Vì thế `LoraTask()` chỉ gọi `SendAcknowledgement()` **sau khi** `PersistTelemetry()` trả về thành công.

### 3.4. Cấu trúc bản ghi và cách ghi an toàn

Dữ liệu được lưu dưới dạng `PersistentTelemetryRecord`, gồm các thông tin chính: **Magic/Version**, `queueSequence` (thứ tự FIFO), `recordId` (định danh bản ghi), `TelemetryMessage` và **CRC32** kiểm tra tính toàn vẹn của bản ghi trên Flash.

`WritePersistentRecordFile()` thực hiện:

1. Ghi toàn bộ bản ghi vào file tạm `.tmp`, `flush()` và đóng file.
2. Đổi tên sang file chính thức `.rec`.
3. Mở file `.rec` để đọc lại, kiểm tra cấu trúc, CRC32 và `recordId`.
4. Chỉ trả về **thành công** khi bản ghi cuối cùng đã được xác minh.

Cách ghi **file tạm → rename → đọc lại** giảm nguy cơ coi một bản ghi đang ghi dở là dữ liệu hợp lệ sau khi khởi động lại. CRC32 ở đây bảo vệ **bản ghi lưu trên Flash**, khác CRC8 dùng cho frame LoRa.

### 3.5. Hàng đợi FIFO, khôi phục và giới hạn lưu trữ

Gateway quản lý hàng đợi bằng ba thao tác:

- `PersistentOutboxPush()`: ghi thêm bản ghi mới.
- `PersistentOutboxPeek()`: đọc **bản ghi cũ nhất** nhưng chưa xóa.
- `PersistentOutboxPop()`: xóa bản ghi cũ nhất **khi đã được MQTT Broker xác nhận**.

Khi khởi động lại, `ScanPersistentOutbox()` quét các file `.rec`, kiểm tra CRC32 và dựng lại thứ tự FIFO theo `queueSequence`; các file `.tmp` dang dở bị loại. Dữ liệu đã ghi thành công vẫn có thể được gửi tiếp sau khi Gateway phục hồi.

Hàng đợi không vô hạn: firmware đặt `FILE_OUTBOX_MAX_RECORDS = 2048` và giữ lại tối thiểu **128 KiB** dung lượng trống. Khi không đủ chỗ để ghi DATA mới, Gateway **không gửi ACK** thay vì nhận dữ liệu mà không lưu được.

## Phần 4. Wi-Fi, MQTT QoS 1 và chuyển dữ liệu từ LittleFS

### 4.1. `MqttTask()` làm gì?

`MqttTask()` gọi `MaintainWiFiConnection()` và khởi tạo MQTT Client khi có Wi-Fi. Client được triển khai bằng **`esp_mqtt_client` của ESP-IDF**, không phải thao tác publish trực tiếp không theo dõi kết quả.

Khi kết nối sẵn sàng, Task xử lý **mỗi lần một bản ghi đầu hàng đợi**:

```text
LittleFS: Peek bản ghi cũ nhất
             ↓
Đóng gói JSON → MQTT PUBLISH QoS 1
             ↓
Chờ sự kiện MQTT_EVENT_PUBLISHED / PUBACK
             ↓
Đối chiếu MQTT message ID
             ↓
Đúng PUBACK → Pop / Xóa bản ghi khỏi LittleFS
Không có PUBACK → Giữ bản ghi để gửi lại
```

Trong `MqttEventHandler()`, sự kiện `MQTT_EVENT_PUBLISHED` gọi `xTaskNotify()` để báo `msg_id` về `MqttTask`; Task dùng `xTaskNotifyWait()` để nhận thông báo và chỉ `PersistentOutboxPop()` khi ID khớp với lần gửi đang chờ.

**Phân biệt hai ACK:** `ACK` của LoRa xác nhận Gateway đã **lưu DATA vào Flash**; `PUBACK` của MQTT QoS 1 xác nhận **Broker đã tiếp nhận PUBLISH**. Nhận `PUBACK` **không đồng nghĩa** dữ liệu đã được ghi thành công vào InfluxDB; khâu đó do Backend đảm nhiệm.

### 4.2. Mất Wi-Fi/MQTT, tự kết nối lại và tránh mất bản ghi

Nếu Wi-Fi/MQTT tạm mất:

- `LoraTask` tiếp tục nhận và lưu dữ liệu **nếu LittleFS còn dung lượng**.
- `MqttTask` duy trì logic kết nối lại; các bản ghi chưa được Broker xác nhận vẫn nằm trong LittleFS.
- Khi mạng hồi phục, Task đọc lại FIFO và gửi tiếp các bản ghi theo thứ tự.

**Lưu ý:** MQTT QoS 1 hướng đến **at-least-once delivery**: Broker có thể nhận bản tin lặp nếu ACK bị mất. Vì vậy, Gateway gửi `recordId` ổn định trong JSON để Backend có cơ sở nhận diện bản ghi trùng, không khẳng định toàn hệ thống bảo đảm exactly-once tuyệt đối.

## Phần 5. Dữ liệu Telemetry, Record ID và Timestamp

### 5.1. Từ DATA Payload LoRa sang bản ghi MQTT

Sau khi giải mã DATA 13 byte, Gateway tạo `TelemetryMessage` bổ sung các thông tin mà STM32 không gửi: **RSSI tại Gateway**, thời điểm Gateway tiếp nhận và các thông tin cần để truy vết bản ghi. Bản ghi được giữ trong LittleFS rồi mới chuyển sang JSON để publish vào topic theo Node:

```text
iot/node01/telemetry
iot/node02/telemetry
iot/node03/telemetry
```

JSON có các trường quan trọng như `id`, `seq`, `temp`, `tempValid`, `status`, `faults`, `rssiDbm`, `sampledAtMs`, `ageMs`, `timestampValid` và `recovered`.

`recordId` được `GenerateRecordId()` tạo theo **Gateway Device ID + Boot Nonce + queueSequence**, nhằm phân biệt bản ghi qua các lần khởi động và giữ một ID ổn định khi gửi lại MQTT. **Không nhầm** `recordId` với `SEQ` LoRa (gắn một giao dịch Node–Gateway) hoặc MQTT `msg_id` (gắn một lần publish đang chờ PUBACK).

### 5.2. Khôi phục thời điểm đo thực tế

STM32 gửi `sampleAgeMs` — tuổi của mẫu tính từ giữa đợt lấy ADC đến lúc bắt đầu truyền. Gateway có thể sử dụng đồng hồ Unix đã đồng bộ NTP và thời gian truyền LoRa ước tính **52 ms** để khôi phục thời điểm đo:

```text
sampledAtUnixMs ≈ receivedAtUnixMs − sampleAgeMs − 52 ms
```

Nhờ đó, dữ liệu lưu đệm và được MQTT gửi lên muộn vẫn có thể mang **thời điểm đo**, thay vì mặc định lấy thời gian publish. Nếu lúc nhận chưa có giờ NTP, firmware giữ thông tin tuổi mẫu trong LittleFS để tìm cách tính lại khi thời gian hệ thống hợp lệ. Nếu Gateway khởi động lại mà trước đó chưa lưu được timestamp tuyệt đối, **không thể luôn khôi phục chính xác thời điểm đo**; code biểu diễn trường không khả dụng khi cần, thay vì tự tạo một timestamp giả.

**Điểm cần nhớ:** `sampleAgeMs` và `sampledAtMs` giúp bảo toàn ý nghĩa thời gian của dữ liệu; `recordId` giúp xử lý trùng lặp xuyên suốt quá trình truyền MQTT. Chúng bổ sung cho nhau nhưng giải quyết **hai bài toán khác nhau**.

