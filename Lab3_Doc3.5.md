# Lab 3: Doc 3.5 - Hướng dẫn kết nối thẻ microSD

Tài liệu này hướng dẫn kết nối mô-đun microSD với ESP32 qua bus SPI để thực hành lưu trữ dữ liệu.

## 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí ESP32, mô-đun microSD và dây kết nối.

## 2. Kết nối SPI

| Mô-đun microSD | ESP32 |
| :--- | :--- |
| `MOSI` | `GPIO23` |
| `MISO` | `GPIO19` |
| `SCK` | `GPIO18` |
| `CS` | `GPIO13` |
| `VCC`, `GND` | Theo nhãn nguồn trên mô-đun và sơ đồ lắp ráp |

Ví dụ trong Lab 3 đặt chân chọn thẻ nhớ là `SD_CS_PIN 13`.

> **Vị trí chèn hình:** Sơ đồ nối từng chân SPI giữa mô-đun microSD và ESP32.

## 3. Kiểm tra trước khi thực hành

- Kiểm tra thẻ microSD đã được lắp đúng chiều.
- Đối chiếu chân `CS` với cấu hình `SD_CS_PIN` trong chương trình.
