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

### 2.1. Node 01/02: ADC1 + DMA

ADC1 liên tục quét **6 kênh** (`CH0`, `CH1`, `CH2`, `CH3`, `CH8`, `CH9`). DMA1 Channel 1 ghi vòng vào:

```c
volatile uint16_t g_adcRawBuffer[NUMBER_CHANNEL]; // NUMBER_CHANNEL = 6
```

Bộ đệm trên chứa **giá trị mới nhất của mỗi kênh**, **không phải** 16 mẫu của từng kênh. Khi cần cập nhật dữ liệu cảm biến, `AcquireFilteredRawSamples()` lặp **16 lượt**, mỗi lượt sao chép sáu giá trị từ bộ đệm DMA sang nhóm mẫu xử lý; giữa các lượt đọc có delay cấu hình **1 ms**.

```text
ADC1 quét 6 kênh → DMA cập nhật g_adcRawBuffer[6]
                              │
             Đọc lặp 16 lượt (6 kênh/lượt)
                              ▼
                    samples[6][16]
                              ▼
                      Lọc từng kênh
```

DMA giảm việc CPU phải tự đọc ADC theo từng kênh. Cần phân biệt **thời gian chờ giữa hai lần đọc bộ đệm** với thời gian chuyển đổi ADC thực tế.

### 2.2. Node 03: ADC ngoài MCP3208

`MCP3208_ReadChannel(channel)` giao tiếp với MCP3208 qua SPI:

1. Kéo chân CS của MCP3208 xuống và gửi lệnh chọn kênh.
2. Nhận các byte kết quả và ghép giá trị **12-bit**:

   ```c
   raw12 = ((highByte & 0x0F) << 8) | lowByte;
   ```
3. Nhả CS, chuyển sang kênh tiếp theo; đọc **6 kênh, lặp 16 lượt** rồi lọc giống Node 01/02.

SX1278 và MCP3208 sử dụng chung các đường SPI nhưng **chân CS riêng**. Khác biệt cần nhớ: Node 01/02 lấy dữ liệu mới nhất từ **bộ đệm DMA**, còn Node 03 **chủ động đọc qua SPI**. Node 03 cũng phải bù hệ số khuếch đại analog khi tính điện trở NTC (trình bày ở Phần 3).

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

