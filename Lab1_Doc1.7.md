# Lab 1: Doc 1.7 - Hướng dẫn lắp ráp bo thí nghiệm

Tài liệu này hướng dẫn đấu nối các linh kiện dùng trong Lab 1 trên bo thí nghiệm rời. Không kết nối mạch thực hành với các bo EPS, Sensor hoặc COM của vệ tinh.

## 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí tổng thể linh kiện trên bo thí nghiệm Lab 1.

## 2. Kết nối linh kiện

| Linh kiện | Kết nối |
| :--- | :--- |
| LED đơn | Chân Anode nối qua điện trở hạn dòng đến `GPIO 2`; chân Cathode nối GND. |
| Nút nhấn | Một chân nối `GPIO 4`; chân còn lại nối GND. Dùng `INPUT_PULLUP`. |
| 74HC595 | `DS` nối `GPIO 13`; `ST_CP` nối `GPIO 14`; `SH_CP` nối `GPIO 15`; `VCC` nối 3.3V; `GND` nối GND. |

> **Vị trí chèn hình:** Đấu nối LED đơn và nút nhấn.

> **Vị trí chèn hình:** Đấu nối 74HC595 và LED 7 đoạn.

## 3. Đối chiếu trước khi thực hành

- Dùng bo thí nghiệm rời và kết nối ESP32 bằng dây jumper.
- Kiểm tra từng dây theo bảng kết nối và các hình sẽ được bổ sung.
- Giữ nguyên cấu hình chân trong bài thực hành tương ứng.
