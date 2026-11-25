# Lab 5: Doc 5.4 - Hướng dẫn kết nối module LoRa

Tài liệu này hướng dẫn đấu nối module Ebyte E32 với ESP32. Các chân điều khiển chưa có số GPIO được để theo nhãn trên bo và chờ sơ đồ lắp thực tế.

## 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí module E32, ESP32 và anten.

## 2. Kết nối E32 với ESP32

| E32 | ESP32 / nguồn |
| :--- | :--- |
| `TXD` | `RXD` của UART1 |
| `RXD` | `TXD` của UART1 |
| `AUX` | Chân GPIO dùng đọc trạng thái |
| `M0`, `M1` | Chân GPIO dùng chọn chế độ |
| `VCC`, `GND` | 3.3 V, `GND` |

Ở chế độ truyền nhận thông thường, đặt `M0` và `M1` ở mức `LOW`. Chờ sơ đồ lắp để điền số GPIO cho `AUX`, `M0` và `M1`.

> **Vị trí chèn hình:** Đấu nối các chân UART, AUX, M0 và M1 của E32.

## 3. Kết nối khi cấu hình module

Khi cấu hình qua USB-UART, đặt `M0` và `M1` ở mức `HIGH`; nối chéo `TXD` và `RXD`, đồng thời nối chung `GND`.

> **Vị trí chèn hình:** Kết nối E32 với bộ chuyển USB-UART để cấu hình.
