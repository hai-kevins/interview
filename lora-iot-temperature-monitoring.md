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

---

# STM32 Node

## Phần 1. Kiến trúc Firmware và luồng hoạt động chính

### 1.1. Vai trò của STM32 Node trong hệ thống

Một Node đảm nhiệm ba công việc gắn liền với nhau:

1. **Thu thập:** lấy dữ liệu analog từ sáu cảm biến NTC qua ADC nội (Node 01/02) hoặc MCP3208 (Node 03).
2. **Xử lý:** lọc các mẫu, đổi ADC → điện trở NTC → nhiệt độ, xác định lỗi và tính nhiệt độ trung bình của các kênh được chấp nhận.
3. **Phản hồi Gateway:** chỉ gửi gói `DATA` khi nhận được `POLL` hợp lệ, sau đó chờ gói `ACK` khớp giao dịch.

```text
ESP32 Gateway                         STM32 Node
      │                                    │
      ├──────────── POLL ─────────────────►│
      │                                    ├─ Lấy mẫu 6 kênh
      │                                    ├─ Lọc và tính nhiệt độ
      │                                    ├─ Phát hiện / ghi nhận lỗi
      │                                    └─ Đóng gói dữ liệu
      │◄─────────── DATA ──────────────────┤
      ├──────────── ACK ──────────────────►│
      │                                    └─ Kết thúc giao dịch
```

Đây là cơ chế **Gateway chủ động thăm dò (polling)**, không phải Node tự động phát liên tục theo một lịch truyền độc lập. Giao thức điều khiển giao dịch nằm trong `main.c`; các module bên dưới đảm nhận thao tác cụ thể.

### 1.3. Trình tự khởi tạo trong `main()`

Ở Node 01/02, thứ tự gọi là:

```text
SysTick_Init()
      ↓
SX1278_Init()
      ↓
ADC_GPIO_Init()
      ↓
ADC_DMA_Init()
      ↓
while (1) — chờ POLL
```

Ở Node 03, hai bước ADC nội được thay bởi `MCP3208_Init()`:

```text
SysTick_Init() → SX1278_Init() → MCP3208_Init() → while (1)
```

- **SysTick** được khởi tạo trước vì driver LoRa sử dụng delay và timeout mili giây.
- **SX1278** được chuẩn bị trước khi vào vòng nhận gói.
- **ADC/DMA hoặc MCP3208** được khởi tạo trước khi Gateway yêu cầu đo lần đầu.

Chưa có một scheduler/RTOS riêng trong phần firmware STM32 Node đang phân tích; chương trình được tổ chức theo vòng lặp và các hàm xử lý đồng bộ. FreeRTOS nằm ở **ESP32 Gateway**, không phải đây.

### 1.4. Luồng `while (1)` từ nhận POLL đến phản hồi DATA

Có thể hiểu logic trong `main()` bằng giả mã sau:

```text
LẶP VÔ HẠN:
    nhận gói LoRa, tối đa 2000 ms
    nếu không nhận gói hợp lệ → trở lại đầu vòng lặp

    nếu địa chỉ không phải của Node
       hoặc TYPE không phải POLL
       hoặc LEN khác 0:
           bỏ qua gói, chờ tiếp

    lấy nhiệt độ trung bình và mốc thời gian đo
    lấy status bitmask của 6 cảm biến
    lấy mã lỗi chi tiết của từng cảm biến
    đóng payload dài 13 byte
    gửi DATA với SEQ sao chép từ POLL
    đợi ACK đúng ADDR + TYPE + SEQ + LEN
    nếu thất bại → truyền lại, tối đa 3 lần
```

**Vì sao phải kiểm tra `ADDR`, `TYPE`, `LEN` trước khi đo?** Một Node có thể nghe được bản tin dành cho Node khác; kiểm tra này tránh lấy mẫu/gửi nhầm. `POLL` không mang payload nên `LEN` phải bằng `0`. Còn `SEQ` từ POLL được giữ nguyên trong DATA và ACK để gắn ba bản tin vào **cùng một giao dịch**.

---

## Phần 2. Thu thập dữ liệu từ cảm biến NTC

### 2.1. Node 01 và Node 02: ADC1 + DMA để lấy sáu kênh

Các cảm biến được ánh xạ như sau:

| Cảm biến | ADC1 channel | Chân STM32 |
|---|---:|---|
| S1 | 0 | PA0 |
| S2 | 1 | PA1 |
| S3 | 2 | PA2 |
| S4 | 3 | PA3 |
| S5 | 8 | PB0 |
| S6 | 9 | PB1 |

Driver khai báo:

```c
volatile uint16_t g_adcRawBuffer[NUMBER_CHANNEL];
```

Với `NUMBER_CHANNEL = 6`, vùng nhớ có sáu phần tử 16-bit, tương ứng sáu vị trí trong chuỗi chuyển đổi. Trong `ADC_DMA_Init()`, ADC1 được cấu hình **scan sáu kênh, chuyển đổi liên tục**, còn DMA1 Channel 1 **ghi vòng (circular)** vào bộ đệm trên. Các thông số liên quan trong code gồm `DMA_BufferSize = 6`, dữ liệu half-word và tự tăng địa chỉ bộ nhớ.

```text
ADC1 quét CH0 → CH1 → CH2 → CH3 → CH8 → CH9
       │
       ▼
DMA1 Channel 1 (Circular)
       │
       ▼
[g_adcRawBuffer[0], ..., g_adcRawBuffer[5]]
       │
       ▼
Firmware đọc giá trị mới nhất của sáu kênh
```

**Ý nghĩa trong dự án:** CPU không phải thực hiện sáu lần truy vấn ADC đồng bộ mỗi khi tính nhiệt độ. DMA liên tục cập nhật các giá trị mới nhất, còn code xử lý đọc từ bộ đệm khi có yêu cầu tạo kết quả đo.

**Lưu ý khi đọc code:** `g_adcRawBuffer` là **bộ đệm mới nhất của mỗi kênh**, không phải bộ đệm chứa sẵn cả 16 mẫu mỗi kênh. Bộ `16 × 6` mẫu được dựng riêng trong `AcquireFilteredRawSamples()`, bằng cách đọc lặp bộ đệm DMA.

### 2.2. Một đợt đo (acquisition burst) được tạo ra thế nào?

Với cả ba Node, các hằng số chính là:

| Hằng số | Giá trị | Vai trò |
|---|---:|---|
| `RAW_SAMPLE_COUNT` | 16 | Số mẫu thu thập cho mỗi kênh trong một đợt đo |
| `RAW_SAMPLE_PERIOD_MS` | 1 | Khoảng delay được chèn giữa hai lượt đọc mẫu liên tiếp |
| `RAW_TRIM_COUNT` | 4 | Số mẫu lớn nhất/nhỏ nhất sẽ bị loại khi lọc |
| `SENSOR_UPDATE_MS` | 1000 | Khoảng thời gian tối thiểu giữa hai lần xử lý lại dữ liệu cảm biến |

Trên Node 01/02, một đợt đo được thực hiện gần như sau:

```text
Lặp 16 lượt:
    chép 6 giá trị mới nhất trong g_adcRawBuffer
    vào ma trận samples[6][16]
    delay 1 ms trước lượt kế (trừ lượt cuối)

Với mỗi kênh:
    lọc 16 mẫu → filteredRaw
    tính spread = max(raw) − min(raw)
```

Cần phân biệt **1 ms delay giữa hai lần sao chép bộ đệm** với thời gian ADC thực sự chuyển đổi. Code không khẳng định mỗi mẫu lấy đúng cách nhau 1 ms ở mức phần cứng; thời gian thực còn phụ thuộc quá trình thực thi. Trên Node 03, mỗi lượt thực sự thực hiện lệnh SPI để đọc từng kênh MCP3208.

### 2.3. Node 03: dùng ADC MCP3208 thay cho ADC1 nội

MCP3208 là ADC ngoài 12-bit. Trong `MCP3208_ReadChannel(channel)`, firmware:

1. Kéo chân CS xuống để chọn MCP3208.
2. Gửi các byte lệnh qua `SPI1_Transfer()` để chọn chế độ đọc và số kênh.
3. Nhận hai byte kết quả; ghép **4 bit thấp của byte cao** với **8 bit của byte thấp** thành kết quả 12-bit.
4. Nhả CS để kết thúc giao dịch SPI.

Có thể viết phép ghép giá trị thành:

```text
raw12 = ((highByte & 0x0F) << 8) | lowByte
```

Đọc cả sáu kênh theo thứ tự `CH0 → CH5`, lặp 16 lần rồi lọc. Các hàm cần chú ý là `MCP3208_ReadChannel()`, `AcquireFilteredRawSamples()` và `MCP3208_Init()`. Node 03 có chân CS của MCP3208 trên `PB0`; chip SX1278 và MCP3208 dùng chung các tín hiệu SPI nhưng có **chân CS riêng**.

### 2.4. Khác biệt cần nhớ giữa hai phương án

| Tiêu chí | Node 01/02 | Node 03 |
|---|---|---|
| Nguồn dữ liệu ADC | ADC1 nội STM32F103 | ADC ngoài MCP3208 |
| Cách nhận mẫu | DMA ghi vòng bộ đệm 6 phần tử | Từng giao dịch SPI đọc ADC |
| Số kênh đang sử dụng | 6 | 6 |
| Độ phân giải được xử lý | 12-bit, `0..4095` | 12-bit, `0..4095` |
| Khối xử lý tiếp theo | Trimmed Mean, NTC, chẩn đoán | Trimmed Mean, bù khuếch đại, NTC, chẩn đoán |
| Điểm đặc thù | `g_adcRawBuffer` và `ADC_DMA_Init()` | `MCP3208_ReadChannel()` và `OPAMP_GAIN` |

**Tránh nhầm lẫn:** Cả hai đều có 16 mẫu/lần xử lý và thuật toán lọc tương tự. Sự khác biệt nằm ở **cách lấy mẫu** và bước **bù khuếch đại analog** trên Node 03, không phải số lượng cảm biến hay định dạng bản tin LoRa.

**Tự kiểm tra:** Tìm được vòng lặp 16 lượt, chứng minh vì sao bộ đệm DMA 6 phần tử vẫn đủ dùng để dựng 16 mẫu/kênh; giải thích ba byte truyền SPI được dùng để đọc một kênh MCP3208.

---

## Phần 3. Xử lý tín hiệu và tính toán nhiệt độ

### 3.1. Chuỗi xử lý dữ liệu của một kênh NTC

```text
16 mẫu ADC thô
       │
       ▼
Trimmed Mean + tính độ phân tán (spread)
       │
       ▼
Giá trị ADC sau lọc
       │
       ├─ Node 03: bù khuếch đại analog
       ▼
Điện trở NTC
       │
       ▼
Nhiệt độ theo mô hình Beta 3950
       │
       ▼
Calibration riêng cho kênh
       │
       ▼
Kiểm tra hợp lệ / trạng thái lỗi
       │
       ▼
Bộ lọc EMA (chỉ cập nhật bằng mẫu tốt)
       │
       ▼
Nhiệt độ đã lọc của kênh
```

Các thuật toán được gom trong `ADC_UpdateSensorData()` hoặc `MCP3208_UpdateSensorData()`. Phần này chỉ giải thích **dữ liệu được tính như thế nào**; quy tắc xác nhận lỗi chi tiết được trình bày ở Phần 4.

### 3.2. Trimmed Mean: lọc 16 mẫu ADC

Với mỗi kênh, firmware:

1. Sắp xếp 16 giá trị ADC từ nhỏ đến lớn.
2. Tính `spread = lớn nhất − nhỏ nhất` trên **toàn bộ 16 mẫu ban đầu**; chỉ số này dùng để phát hiện nhiễu.
3. Loại bỏ 4 mẫu nhỏ nhất và 4 mẫu lớn nhất.
4. Lấy trung bình 8 mẫu ở giữa, làm tròn về số nguyên ADC.

Công thức ứng với các mẫu đã sắp xếp `x[0] ≤ ... ≤ x[15]`:

```text
filteredRaw ≈ (x[4] + x[5] + ... + x[11]) / 8
rawSpread = x[15] - x[0]
```

Ví dụ minh họa một dãy 16 mẫu đã sắp xếp:

```text
1900 1990 2000 2004 | 2007 2008 2009 2010 2010 2011 2012 2013 | 2015 2016 2020 2200
                      └────────── 8 giá trị được giữ ──────────┘
```

Tám giá trị giữa cho kết quả trung bình khoảng **2010 ADC counts** (được làm tròn), trong khi hai giá trị 1900 và 2200 có thể tạo ra sai lệch mạnh nếu lấy trung bình đơn giản. `rawSpread = 300`, lớn hơn ngưỡng nhiễu `100` đang cấu hình: **mẫu sau lọc có thể ổn định nhưng đợt lấy mẫu vẫn bị đánh dấu nhiễu**. Hai thao tác lọc và chẩn đoán phục vụ hai mục đích khác nhau.

### 3.3. Chuyển giá trị ADC thành điện trở NTC

Các tham số trong code Node 01/02:

| Tham số | Giá trị |
|---|---:|
| `ADC_FULL_SCALE` | `4095` |
| `R_FIXED` | `10000 Ω` |
| `NTC_R25` | `10000 Ω` tại `25 °C` |
| `NTC_BETA` | `3950` |
| `NTC_T25_K` | `298.15 K` |

Với mạch phân áp theo chiều đã được giả định trong code:

```text
R_NTC = R_FIXED × rawAdc / (4095 − rawAdc)
```

**Ví dụ minh họa:** `rawAdc ≈ 2048`, `R_FIXED = 10 kΩ` → `R_NTC ≈ 10 kΩ`, tương ứng gần nhiệt độ danh định 25 °C của NTC 10 kΩ. Công thức phải khớp với hướng mắc NTC và điện trở cố định trên phần cứng; đảo vị trí hai điện trở sẽ làm thay đổi công thức.

Trước phép chia, code kiểm tra `rawAdc == 0` hoặc `rawAdc >= 4095` để tránh đầu vào không hợp lệ. Nếu điện trở tính ra nằm ngoài giới hạn cấu hình, driver gắn lỗi điện trở.

### 3.4. Mô hình Beta 3950 đổi điện trở thành nhiệt độ

Sau khi tìm được điện trở, firmware áp dụng:

```text
1 / T(K) = 1 / 298.15 + ln(R_NTC / 10000) / 3950
T(°C) = T(K) − 273.15
```

Đây chính là cách `ConvertRawToTemperature()` sử dụng `logf()` trong mã nguồn. Nó tạo ra **nhiệt độ theo mô hình của cảm biến**, chưa phải giá trị cuối cùng được dùng ngay để phát LoRa.

**Ví dụ:** Nếu `R_NTC = 10000 Ω` thì `ln(1) = 0`; kết quả là `298.15 K = 25 °C`.

### 3.5. Bù độ khuếch đại trên Node 03

Node 03 có mạch khuếch đại analog; driver đặt `OPAMP_GAIN = 1.15`. Vì ADC đọc tín hiệu **sau khuếch đại**, chương trình loại hệ số khuếch đại trước khi tính điện trở:

```text
ratio_at_adc = rawAdc / 4095
ratio_at_ntc = ratio_at_adc / 1.15
R_NTC = R_FIXED × ratio_at_ntc / (1 − ratio_at_ntc)
```

Nhờ đó Node 03 không áp trực tiếp công thức Node 01 lên tín hiệu đã khuếch đại. Code cũng kiểm tra `0 < ratio_at_ntc < 1` để loại trường hợp không có nghiệm phân áp hợp lệ.

**Lưu ý:** `1.15` là hệ số được *khai báo trong firmware*. Không nên trình bày nó như kết quả đo sai số khuếch đại thực tế nếu chưa có dữ liệu hiệu chuẩn phần cứng.

### 3.6. Calibration theo từng kênh

Trong mỗi module, chương trình có hai mảng riêng cho sáu kênh:

```text
scale[6]  = {1, 1, 1, 1, 1, 1}
offset[6] = {0, 0, 0, 0, 0, 0}
```

Công thức ứng dụng:

```text
T_calibrated[i] = T_model[i] × scale[i] + offset[i]
```

Ví dụ minh họa: nếu kênh S2 cần bù cộng `0.4 °C`, có thể đặt `offset[1] = +0.4`. **Nhưng trong code hiện tại**, mọi hệ số đang là giá trị trung tính, nên không nên khẳng định hệ thống đã hoàn tất hiệu chuẩn thực nghiệm sáu cảm biến.

### 3.7. Bộ lọc EMA làm mượt nhiệt độ

Khi có nhiệt độ **không phát hiện lỗi tức thời**, chương trình cập nhật:

```text
T_filtered(new) = T_filtered(old)
                + 0.25 × [T_candidate − T_filtered(old)]
```

- Ở lần đầu có dữ liệu tốt, firmware gán `T_filtered = T_candidate` để khởi tạo.
- Ở các lần sau, EMA giữ lại 75% giá trị lọc trước đó và nhận thêm 25% thay đổi từ mẫu mới.
- Khi đợt hiện tại có lỗi tức thời, EMA **không cập nhật bằng mẫu lỗi**; giá trị lọc cũ được giữ nguyên.

**Ví dụ minh họa:** Giá trị lọc cũ `25 °C`, nhiệt độ hợp lệ mới `29 °C` → giá trị mới `26 °C`. Điều này giúp giảm biến động hiển thị, đổi lại nhiệt độ phản ứng chậm hơn với thay đổi đột ngột.

### 3.8. Tính nhiệt độ trung bình của Node

Hàm `ADC_GetStableAverageTemp()` (hoặc `MCP3208_GetStableAverageTemp()`) tính trung bình các kênh:

- đã được khởi tạo bằng ít nhất một lần đo tốt (`isInitialized != 0`);
- **không có lỗi đã chốt** (`latchedFaults == SENSOR_FAULT_NONE`).

```text
T_average = tổng T_filtered của các kênh được chấp nhận / số kênh được chấp nhận
```

Ví dụ minh họa: 5 kênh tốt có nhiệt độ `25, 26, 27, 28, 29 °C`, một kênh có lỗi đã xác nhận → trung bình truyền đi là `(25+26+27+28+29)/5 = 27 °C`.

Nếu **không có kênh nào được chấp nhận**, hàm trả về `TEMP_ERROR_VALUE = -1000.0f`. Ở phần đóng gói LoRa, giá trị này vượt miền mã hóa và được biến thành **marker nhiệt độ không hợp lệ `0x8000`**, không phải gửi `-1000 °C` như một giá trị đo thật.

**Sắc thái quan trọng trong code:** Khi lỗi mới xuất hiện nhưng **chưa đạt ngưỡng chốt lỗi**, dữ liệu lọc cũ của kênh chưa bị thay bởi mẫu lỗi; kênh đó vẫn có thể được tính vào trung bình đến khi `latchedFaults` được kích hoạt. Đó là một lựa chọn thiết kế nhằm tránh phản ứng quá nhạy với lỗi thoáng qua, không phải bằng chứng mọi giá trị trung bình đều vừa được đo mới tại cùng thời điểm.

**Tự kiểm tra:** Giải thích kết quả khi một kênh lỗi, khi cả sáu kênh lỗi, và tại sao phải tính `spread` trước khi loại mẫu cực trị.

---

## Phần 4. Phát hiện và quản lý lỗi cảm biến

### 4.1. Hai lớp phát hiện lỗi trong firmware

Firmware không chỉ kiểm tra một ngưỡng ADC đơn lẻ. Nó phối hợp:

- **Phát hiện lỗi của đợt đo hiện tại** (`observedFaults`): cờ lỗi của dữ liệu vừa xử lý.
- **Trạng thái lỗi đã xác nhận** (`latchedFaults`): cờ lỗi tồn tại dai dẳng, được phép ảnh hưởng đến trạng thái chính thức và việc đưa kênh vào trung bình.

Mỗi kênh có cấu trúc trạng thái gần như sau:

```text
SensorState_t
├─ filteredTemperature   : nhiệt độ sau EMA
├─ resistanceOhm         : điện trở đã tính
├─ filteredRaw           : ADC sau Trimmed Mean
├─ rawSpread             : độ phân tán trong burst
├─ lastCandidateTemperature / TimeMs
├─ observedFaults        : lỗi thấy ở lần đo hiện tại
├─ latchedFaults         : lỗi đã được xác nhận
├─ faultAssertCount[8]   : bộ đếm xác nhận riêng từng loại lỗi
├─ faultClearCount[8]    : bộ đếm phục hồi riêng từng loại lỗi
└─ isInitialized / hasLastCandidate
```

**Điểm quan trọng:** Hai bộ đếm được quản lý **theo từng loại lỗi** chứ không chỉ có một biến đếm chung cho toàn bộ cảm biến.

### 4.2. Tám loại lỗi đang được kiểm tra

| Mã lỗi | Khi nào xuất hiện | Giá trị cấu hình liên quan |
|---|---|---|
| `SHORT` | ADC quá thấp, nghi ngờ chập/đầu vào sát 0 | `rawAdc ≤ 20` |
| `OPEN` (Node 01/02) hoặc `HIGH_SAT` (Node 03) | ADC quá cao, nghi ngờ hở mạch/bão hòa | `rawAdc ≥ 4075` |
| `SIGNAL_NOISY` | Các giá trị ADC trong burst dao động quá lớn | `rawSpread > 100` counts |
| `RESISTANCE` | Điện trở NTC ngoài miền chấp nhận | Node 01/02: `250..500000 Ω`; Node 03: `250..60000 Ω` |
| `TEMP_RANGE` | Nhiệt độ theo mô hình/hiệu chuẩn nằm ngoài miền cấu hình | Node 01/02: `−40..125 °C`; Node 03: `−10..125 °C` |
| `RATE` | Nhiệt độ thay đổi nhanh bất thường giữa hai mẫu tham chiếu | `> 10 °C/s`, khi khoảng cách mẫu `≤ 3000 ms` |
| `CROSS_SENSOR` | Một kênh lệch nhiều so với trung vị của các kênh hợp lệ | Cần ít nhất 4 kênh ứng viên, lệch `> 12 °C` |
| `MODEL` | Không tính được điện trở/nhiệt độ hợp lệ theo mô hình | Ví dụ ADC ở biên hoặc tỷ số phân áp không hợp lệ |

Các ngưỡng trên lấy từ `adc_driver.h` và `mcp3208.h`. Riêng tên cờ `OPEN` ở Node 01/02 và `HIGH_SAT` ở Node 03 phản ánh cách hai driver đặt tên hiện tượng ADC cao, không nên coi chúng là hai vị trí bit khác nhau.

### 4.3. Cơ chế kiểm tra tốc độ biến thiên và đối chiếu cảm biến

**Rate check.** Driver có bộ nhớ giá trị ứng viên tốt trước đó và mốc thời gian tương ứng. Khi thời gian giữa hai lần so sánh nằm trong phạm vi hợp lệ, chương trình tính:

```text
rate = |T_current − T_previous| / Δt_seconds
```

Nếu lớn hơn `10 °C/s`, đánh dấu lỗi `RATE`. Ví dụ minh họa: tăng `25 → 40 °C` trong `1 s` tương đương `15 °C/s` nên vượt ngưỡng. Đây là biện pháp lọc biến động bất thường theo giả định ứng dụng; không có nghĩa mọi thay đổi nhanh đều chắc chắn do cảm biến hỏng.

**Cross-sensor check.** Khi có ít nhất bốn kênh ứng viên không mắc lỗi trước đó, firmware tính **trung vị** của nhóm này. Mỗi kênh được so với trung vị; nếu lệch hơn `12 °C` thì gắn `CROSS_SENSOR`. Trung vị được dùng thay vì trung bình để tránh một số kênh lệch lớn kéo mốc so sánh đi quá xa.

Ví dụ minh họa: `[25, 25, 26, 26, 27, 55] °C` → trung vị `26 °C`; kênh `55 °C` lệch `29 °C` nên bị đánh dấu. Nhưng nếu thật sự có một điểm nóng cục bộ, đây vẫn có thể là nhiệt độ thực: cần hiểu rằng thuật toán đang giả định sáu đầu đo có quan hệ nhiệt độ đủ gần nhau.

### 4.4. `Observed Fault` khác `Latched Fault` thế nào?

Cơ chế xác nhận lỗi dùng hai ngưỡng:

```text
FAULT_ASSERT_COUNT = 3
FAULT_CLEAR_COUNT  = 5
```

**Khi một loại lỗi xuất hiện:** tăng `faultAssertCount` của loại đó; nếu đủ **3 lần cập nhật có lỗi liên tiếp**, bật bit tương ứng trong `latchedFaults`. Trong lúc lỗi tiếp diễn, bộ đếm phục hồi của loại đó được đặt lại.

**Khi loại lỗi đó không xuất hiện:** đặt lại bộ đếm xác nhận. Nếu bit đã được chốt, tăng `faultClearCount`; sau **5 lần cập nhật không còn lỗi đó liên tiếp**, xóa bit lỗi đã chốt.

Ví dụ minh họa:

```text
Các lần cập nhật:     1   2   3   4   5   6   7   8
Lỗi SHORT quan sát:  Có Có Có  Không Không Không Không Không
Latched SHORT:       -   -   Bật  Bật   Bật   Bật   Bật   Xóa
```

Với `SENSOR_UPDATE_MS = 1000`, ba lần xác nhận không nhất thiết bằng chính xác ba giây đo liên tục ngoài thực tế: Node thường chỉ kích hoạt cập nhật khi nhận POLL và đã qua thời hạn tối thiểu. Không nên quy đổi ngưỡng **số lần cập nhật** thành một thời lượng cố định nếu không biết nhịp POLL.

### 4.5. Hai lớp bitmask: trạng thái kênh và loại lỗi

**Bitmask thứ nhất — trạng thái tổng hợp 6 cảm biến** do `ADC_GetStatusBitmask()`/`MCP3208_GetStatusBitmask()` trả về:

```text
bit 0 → S1
bit 1 → S2
bit 2 → S3
bit 3 → S4
bit 4 → S5
bit 5 → S6
```

Bit `1` nghĩa là kênh **chưa khởi tạo bằng mẫu tốt** hoặc **đang có lỗi đã chốt**; bit `0` nghĩa là kênh có thể được sử dụng theo tiêu chí này. Sáu bit cao hơn không dùng trong trạng thái tổng hợp.

Ví dụ minh họa: `STATUS = 0b00001010 = 0x0A` → S2 và S4 đang có trạng thái không hợp lệ.

**Bitmask thứ hai — mã lỗi chi tiết riêng từng cảm biến**, ví dụ:

| Bit | Giá trị | Ý nghĩa |
|---|---|---|
| 0 | `0x01` | SHORT |
| 1 | `0x02` | OPEN / HIGH_SAT (tùy Node) |
| 2 | `0x04` | SIGNAL_NOISY |
| 3 | `0x08` | RESISTANCE |
| 4 | `0x10` | TEMP_RANGE |
| 5 | `0x20` | RATE |
| 6 | `0x40` | CROSS_SENSOR |
| 7 | `0x80` | MODEL |

Ví dụ minh họa: `fault_S3 = 0x05` nghĩa là S3 đang có **SHORT và SIGNAL_NOISY** đã được chốt. Việc dùng bitmask cho phép một kênh mang đồng thời nhiều mã lỗi, thay vì chỉ chứa một chuỗi mô tả.

**Quan hệ giữa hai lớp:** `STATUS` dùng cho câu hỏi *"kênh nào không hợp lệ?"*; `FAULT_S1..S6` dùng cho *"kênh đó lỗi kiểu gì?"*. Cảm biến **chưa khởi tạo** có thể làm `STATUS` bật ngay cả khi mã lỗi chi tiết chưa có bit nào được chốt.

### 4.6. Ảnh hưởng của lỗi đến dữ liệu truyền đi

- **Mẫu có lỗi quan sát:** không được đưa vào cập nhật EMA ở lượt xử lý đó.
- **Kênh đã chốt lỗi:** bị loại khỏi phép tính nhiệt độ trung bình.
- **Kênh chưa chốt lỗi:** có thể tiếp tục dùng nhiệt độ EMA tốt trước đó nếu đã được khởi tạo.
- **Không còn kênh hợp lệ:** nhiệt độ trung bình được đánh dấu không hợp lệ khi đóng gói.
- **Thông tin chẩn đoán:** trạng thái tổng hợp và sáu mã lỗi chi tiết được gửi cùng gói DATA cho Gateway.

**Tự kiểm tra:** Giải thích tại sao phải có `observedFaults` và `latchedFaults` riêng; với một kênh lỗi SHORT chỉ một lần rồi phục hồi, liệu `STATUS` có nhất thiết bật không?

---

## Phần 5. Giao tiếp LoRa và giao thức truyền dữ liệu

### 5.1. Vai trò của SX1278 và cách tổ chức driver

Trong firmware, SX1278 là chip vô tuyến LoRa kết nối STM32 qua SPI. `sx1278.c` điều khiển chân NSS/RESET, ghi đọc thanh ghi và FIFO; `spi_driver.c` truyền các byte SPI; `main.c` quyết định **khi nào** gửi gói gì.

Cấu hình LoRa trong driver:

| Thông số | Giá trị đang cấu hình |
|---|---|
| Tần số | 433 MHz |
| Spreading Factor | SF7 |
| Bandwidth | 125 kHz |
| Coding Rate | 4/5 |
| Preamble | 8 symbols |
| Header | Explicit |
| LoRa payload CRC phần cứng | Bật |

Luồng thao tác chính:

```text
Ứng dụng muốn gửi DATA
      ↓
SX1278_SendPacket()
      ↓
Tạo frame ứng dụng + CRC8
      ↓
Ghi frame vào SX1278 FIFO qua SPI
      ↓
Chuyển radio sang TX, chờ TX_DONE
      ↓
Chuyển lại chế độ RX, chờ ACK
```

Lưu ý: chế độ **TX_DONE** chỉ xác nhận module radio đã hoàn tất phát, **không chứng minh Gateway đã nhận/lưu gói**. Đó là lý do lớp ứng dụng cần ACK riêng.

### 5.2. Cấu trúc frame ứng dụng

```text
+-------+------+------+-----+-----+----------------+------+
| START | ADDR | TYPE | SEQ | LEN | PAYLOAD        | CRC8 |
+-------+------+------+-----+-----+----------------+------+
   1B      1B     1B    1B    1B     LEN bytes      1B
```

| Trường | Ý nghĩa |
|---|---|
| `START` | Giá trị cố định `0xAA`, nhận diện đầu frame ứng dụng |
| `ADDR` | Địa chỉ Node (`0x01`, `0x02`, `0x03`) |
| `TYPE` | Loại gói: `DATA=0x01`, `ACK=0x10`, `POLL=0x23` |
| `SEQ` | Số thứ tự giao dịch do Gateway đưa trong POLL |
| `LEN` | Số byte payload; với POLL/ACK đang dùng bằng `0` |
| `PAYLOAD` | Dữ liệu theo từng TYPE |
| `CRC8` | Giá trị kiểm tra lỗi lớp ứng dụng |

`LORA_FRAME_OVERHEAD = 6` vì có 5 byte trước payload và 1 byte CRC8 cuối. `MAX_LORA_PAYLOAD = 64`, nên độ dài frame ứng dụng tối đa theo hằng số là **70 byte**.

**Cần phân biệt hai tầng kiểm tra lỗi:** chip SX1278 đã bật **payload CRC của LoRa** ở tầng radio, đồng thời `sx1278.c` còn tự tính **CRC8 cho frame ứng dụng**. Hai cơ chế bổ sung cho nhau, không phải cùng một trường CRC.

### 5.3. CRC8 được tính trên phần nào?

Driver dùng CRC-8 với:

- **Polynomial:** `0x07`;
- **Giá trị khởi tạo:** `0x00`;
- **Xử lý:** XOR dữ liệu từng byte, dịch từng bit; nếu bit cao đang bằng 1 thì XOR với đa thức sau khi dịch.

Vùng dữ liệu đưa vào CRC8 là:

```text
ADDR | TYPE | SEQ | LEN | PAYLOAD
```

**Không** đưa `START` vào phép tính và **không** đưa chính byte CRC8 vào CRC đầu vào. Khi nhận, driver tính lại trên cùng vùng rồi so với CRC8 cuối frame. Nếu không khớp, bản tin bị loại.

Ví dụ ý nghĩa: nếu nhiễu làm một byte `TYPE` biến từ `DATA` thành `ACK`, CRC8 có thể phát hiện sự thay đổi đó trước khi `main()` xử lý loại gói. CRC8 phục vụ **phát hiện lỗi vô tình**, không phải xác thực nguồn gửi hoặc chống giả mạo bằng mật mã.

### 5.4. DATA Payload 13 byte: đọc đúng từng trường

Payload của gói nhiệt độ được tạo trong `main.c`:

```text
Offset      0        1       2        3  4  5  6  7  8         9  10 11 12
         +--------+--------+--------+------------------+---------------------+
         | TEMP_L | TEMP_H | STATUS | FAULT_S1 ... S6  | SAMPLE_AGE_MS_LE32  |
         +--------+--------+--------+------------------+---------------------+
Kích thước   1B       1B      1B             6B                 4B
```

**Trường nhiệt độ:** chỉ có **một nhiệt độ trung bình của Node**, không phải nhiệt độ riêng cả sáu kênh. Trước khi gửi, giá trị `float` được đổi thành số nguyên có dấu `int16_t` với tỷ lệ `100` (đơn vị **0,01 °C**) và đặt **little-endian** vào byte 0–1.

Ví dụ minh họa: `27.35 °C × 100 = 2735 = 0x0AAF` → `TEMP_L = 0xAF`, `TEMP_H = 0x0A`. Nếu không có cảm biến hợp lệ, marker gửi là `0x8000`, khác với nhiệt độ thật. Code cũng làm tròn giá trị trước khi ép kiểu số nguyên và dành riêng marker không hợp lệ.

**Trường trạng thái:** byte 2 chứa sáu bit trạng thái S1..S6; chỉ dùng `0x3F` để giữ 6 bit thấp.

**Trường mã lỗi:** byte 3–8 chứa 6 byte bitmask lỗi đã chốt của từng kênh. Thông tin này cho biết **loại lỗi**, nhưng không mang sáu nhiệt độ riêng lẻ.

**Trường `sampleAgeMs`:** byte 9–12 là số nguyên không dấu 32-bit little-endian, mô tả thời gian từ mốc giữa burst lấy mẫu tới thời điểm Node sắp phát DATA.

### 5.5. Vì sao cần `sampleAgeMs`, và tại sao cập nhật khi retry?

Trong module cảm biến, timestamp được gán bằng **trung điểm đợt lấy 16 mẫu**, không phải lúc truyền gói. Khi tạo DATA, hàm gửi lấy:

```text
sampleAgeMs = millis_now − measurementTimeMs
```

Giá trị này được ghi lại vào 4 byte cuối payload **trước từng lần thử gửi**. Nếu mất ACK và phải thử lại, mẫu đo không trở nên mới hơn: tuổi mẫu sẽ **tăng lên**. Gateway có thể sử dụng tuổi mẫu kết hợp với thời gian hệ thống để ước lượng thời điểm đo ban đầu, giúp dữ liệu lịch sử ít bị lệch bởi thời gian chờ hoặc truyền lại.

Ví dụ minh họa: đo ở `t=1000 ms`, thử gửi đầu ở `t=1040 ms` → `sampleAgeMs=40`; retry ở `t=1600 ms` → `sampleAgeMs=600`. **SEQ giữ nguyên**, tuổi mẫu được cập nhật.

### 5.6. POLL → DATA → ACK và xác nhận cùng giao dịch

Khi Node nhận POLL có địa chỉ và kiểu dữ liệu hợp lệ, nó lấy **chính `SEQ` của POLL** để đưa vào DATA. Sau khi phát DATA, `WaitForAcknowledgement()` chỉ chấp nhận ACK đáp ứng đồng thời:

```text
ACK.ADDR == địa chỉ Node hiện tại
ACK.TYPE == 0x10
ACK.SEQ  == SEQ của gói DATA vừa gửi
ACK.LEN  == 0
```

Nếu nhận một gói khác (ví dụ ACK cũ mang `SEQ` khác), giao dịch **không** được coi là thành công. Đây là ý nghĩa thực tế của Sequence Number trong dự án.

### 5.7. Timeout và Retransmission trong mã nguồn

Các ngưỡng `main.c`:

| Cấu hình | Giá trị | Tác dụng |
|---|---:|---|
| `NODE_RECEIVE_TIMEOUT_MS` | 2000 ms | Thời gian mỗi lần Node chờ nhận POLL |
| `DATA_TRANSMIT_TIMEOUT_MS` | 1000 ms | Giới hạn chờ radio hoàn tất phát DATA |
| `ACK_WAIT_TIMEOUT_MS` | 500 ms | Thời gian chờ ACK sau khi phát thành công |
| `DATA_TRANSMIT_MAX_ATTEMPTS` | 3 | Số **lần thử gửi tối đa, gồm cả lần đầu** |

Luồng retry:

```text
Nhận POLL (SEQ=N)
       │
       ▼
Tạo DATA từ một đợt đo
       │
       ├─ Thử gửi #1 ──► TX lỗi / ACK timeout ──┐
       ├─ Thử gửi #2 ◄───────────────────────────┤
       ├─ Thử gửi #3 ◄───────────────────────────┤
       │                                       │
       ├─ Có ACK hợp lệ → Thành công           │
       └─ Hết 3 lần, chưa có ACK → Thất bại ◄─┘
```

- Nếu **radio gửi thất bại**, tăng `g_transmitFailureCount` và chuyển sang lần thử tiếp theo, không vào bước đợi ACK của lần đó.
- Nếu **radio phát xong nhưng thiếu ACK hợp lệ**, tăng `g_ackTimeoutCount` rồi thử lại.
- Nếu **nhận đúng ACK**, tăng `g_successfulTransactionCount` và kết thúc ngay.
- Nếu **hết số lần thử**, tăng `g_retryExhaustedCount`; chương trình quay lại đợi POLL mới.

**Ba điểm dễ nhầm:** (1) tối đa **3 lần gửi tổng cộng**, không phải 3 lần retry thêm ngoài lần đầu; (2) `TX_DONE` khác ACK; (3) Node không tự động tạo một burst đo mới trong mỗi lần retry — dữ liệu gốc của giao dịch được giữ, chỉ cập nhật tuổi mẫu.

### 5.8. Tại sao truyền lại có thể tạo bản tin trùng lặp?

Giả sử Gateway đã nhận được DATA, nhưng ACK trả về bị mất. Node sẽ gửi lại **cùng SEQ**; Gateway có thể nhận hai bản tin biểu diễn cùng một đợt đo. Do đó, cần cơ chế nhận diện trùng lặp ở phía Gateway (theo địa chỉ Node và thông tin giao dịch) để không lưu đúp.

**Ranh giới trách nhiệm:** STM32 Node thực hiện **truyền lại và xác nhận ACK**. Cơ chế **chống lưu trùng dữ liệu** thuộc phần xử lý ở **ESP32 Gateway**, không phải một bảng lưu những SEQ đã nhận trong STM32 Node.

### 5.9. Đọc đường truyền và đường nhận trong `sx1278.c`

Khi **gửi**:

1. Kiểm tra độ dài payload tối đa và con trỏ payload.
2. Tạo dữ liệu CRC8 và byte CRC8.
3. Đưa `START`, header, payload, CRC8 vào FIFO.
4. Chuyển chip sang TX; chờ cờ `TX_DONE` hoặc timeout.
5. Trở lại chế độ RX để chuẩn bị nhận ACK.

Khi **nhận**:

1. Đưa chip vào RX và theo dõi cờ `RX_DONE`.
2. Loại gói có lỗi payload CRC ở tầng radio.
3. Kiểm tra kích thước frame, byte `START` và độ dài `LEN`.
4. Đọc header/payload từ FIFO; bảo đảm không vượt buffer 64 byte.
5. Tính lại CRC8 lớp ứng dụng, chỉ trả về gói hợp lệ.

Sau khi hàm driver xác nhận frame hợp lệ, `main()` vẫn kiểm tra **ngữ nghĩa ứng dụng** như `ADDR`, `TYPE`, `LEN`, `SEQ` tùy ngữ cảnh. Hai tầng kiểm tra này là khác nhau.

**Tự kiểm tra:** Giải mã một DATA có `TEMP_L=0xAF`, `TEMP_H=0x0A`, `STATUS=0x02`; nêu điều kiện chính xác để Node nhận ACK và giải thích vì sao `sampleAgeMs` phải cập nhật khi retry.

---

