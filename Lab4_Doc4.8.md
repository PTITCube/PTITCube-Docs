# Lab 4: Doc 4.8 - Hướng dẫn kết nối cảm biến và tải trọng

Tài liệu này tổng hợp các kết nối cảm biến, module GNSS và camera dùng trong Lab 4. Dùng các vị trí chờ hình để bổ sung sơ đồ lắp thực tế.

## 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí ESP32, các cảm biến, module GNSS, camera và IC 74HC4066.

## 2. Kết nối cảm biến I2C

Các cảm biến BMP280, MPU6500 và GY-271 dùng chung bus I2C:

| Tín hiệu | ESP32 |
| :--- | :--- |
| `SDA` | `GPIO21` |
| `SCL` | `GPIO22` |

| Cảm biến | Địa chỉ I2C |
| :--- | :--- |
| BMP280 | `0x76` hoặc `0x77` |
| MPU6500 | `0x68` hoặc `0x69` |
| GY-271 | `0x0D` hoặc `0x1E` |

> **Vị trí chèn hình:** Đấu nối ba cảm biến trên cùng bus I2C.

## 3. Kết nối GNSS và camera

Module GNSS L76X dùng UART2:

| L76X | ESP32 |
| :--- | :--- |
| `VCC` | 3.3 V |
| `GND` | `GND` |
| `TX` | `GPIO16` (`RX2`) |
| `RX` | `GPIO17` (`TX2`), khi gửi lệnh |
| `PPS` | Để hở nếu chỉ đọc NMEA |

Camera ESP32-CAM dùng chung UART2 với GNSS qua IC chuyển mạch `74HC4066`. Chân điều khiển IC cần được đấu theo sơ đồ phần cứng của bộ kit.

> **Vị trí chèn hình:** Đấu nối GNSS, camera và 74HC4066; thể hiện cả các chân điều khiển chuyển mạch.

## 4. Kiểm tra trước khi thực hành

- Đối chiếu điện áp cấp với yêu cầu của từng mô-đun.
- Kiểm tra địa chỉ I2C và đường UART trước khi chạy chương trình.
