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
