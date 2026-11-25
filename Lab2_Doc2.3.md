# Lab 2: Doc 2.3 - Hướng dẫn lắp ráp hệ thống nguồn

Tài liệu này tổng hợp các kết nối của mô-đun sạc TP4056A và mạch đo pin MAX17048. Đối chiếu nhãn chân trên bo mạch trước khi đấu nối.

## 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí bo EPS, pin, mô-đun sạc và MAX17048.

## 2. Kết nối TP4056A

| Chân | Kết nối |
| :--- | :--- |
| `IN+` / `IN-` | Nguồn sạc DC 4.5–8.0 V |
| `B+` / `B-` | Cực dương / cực âm của pin Li-Po |
| `OUT+` / `OUT-` | Đầu ra cấp tải |

> **Vị trí chèn hình:** Đấu nối nguồn sạc, pin và tải vào TP4056A.

Không sạc pin vượt 4.25 V. Ngừng xả khi pin còn 3.0 V; xả dưới 2.7 V có thể làm hỏng pin.

## 3. Kết nối MAX17048 với ESP32

| MAX17048 | ESP32 |
| :--- | :--- |
| `VIN` | 3.3 V |
| `GND` | `GND` |
| `SCL` | `GPIO22` |
| `SDA` | `GPIO21` |

> **Vị trí chèn hình:** Kết nối MAX17048 với ESP32 và bo EPS.

## 4. Kiểm tra trước khi thực hành

- Đối chiếu cực tính pin và các đầu nối với sơ đồ lắp ráp.
- Giữ nguyên địa chỉ I2C của MAX17048 là `0x36`.
