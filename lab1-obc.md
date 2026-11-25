# Lab 1: Máy tính trên Bo mạch (OBC) & Môi trường Lập trình Nhúng

---

## 📖 Doc 1.1: ESP32-WROOM-32 và giao tiếp ngoại vi

### 1. Máy tính trên bo mạch

Máy tính trên bo mạch (OBC) điều phối các phân hệ của vệ tinh CubeSat 1U. OBC thực hiện các nhiệm vụ sau:

1. **Khởi động:** Đọc cấu hình từ ROM/Flash, kiểm tra ngoại vi và bắt đầu thời gian im lặng vô tuyến 30 phút sau khi công tắc hành trình phóng nhả.
2. **Thu thập dữ liệu:** Định kỳ đọc thông số EPS (điện áp, dòng sạc/xả, dung lượng pin, nhiệt độ cell), cảm biến (áp suất khí quyển, gia tốc, vận tốc góc, từ trường Trái Đất, tọa độ GPS) và trạng thái kết cấu; đóng gói thành bản tin nhị phân Beacon.
3. **Xử lý lệnh:** Nhận gói tin từ trạm mặt đất qua phân hệ COM, giải mã `Opcode` và `Payload`, xác thực `Security Token`, rồi thực hiện lệnh như chụp ảnh, đổi chu kỳ phát Beacon, bật/tắt cảm biến hoặc nạp lại tham số quỹ đạo.
4. **Lưu trữ:** Ghi dữ liệu cảm biến, ảnh viễn thám và nhật ký bay vào microSD qua SPI, dùng định dạng FAT32.
5. **Giám sát và phục hồi:** Theo dõi nhịp tim, làm mới Watchdog Timer và khởi động lại phần mềm hoặc cấp nguồn lại khi phát hiện bus bị treo do bức xạ.

![Kiến trúc phần cứng OBC ESP32](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_obc_architecture.png)

---

### 2. Thông số ESP32-WROOM-32

Bộ kit PTITCube dùng vi điều khiển **ESP32-WROOM-32** của Espressif Systems. Chip có kiến trúc 32-bit, bộ nhớ tích hợp và hỗ trợ FreeRTOS.

| Khối | Thông số | Ứng dụng |
| :--- | :--- | :--- |
| **Kiến trúc lõi** | Hai lõi Tensilica Xtensa 32-bit LX6 | Có thể tách tác vụ mạng ở Core 0 và thuật toán bay ở Core 1. |
| **Tần số xung nhịp** | 80, 160 hoặc tối đa 240 MHz | Hỗ trợ xử lý ma trận tư thế 3D và ảnh JPEG. |
| **SRAM** | $520\text{ KB}$ ($512\text{ KB}$ khả dụng) | $192\text{ KB}$ IRAM cho mã; $328\text{ KB}$ DRAM cho dữ liệu, DMA và ngăn xếp. |
| **Flash ngoài** | 4 MB hoặc 16 MB, Quad-SPI | Lưu chương trình, bảng phân vùng và SPIFFS/LittleFS. |
| **Bộ đồng xử lý ULP** | FSM tiết kiệm điện | Đọc cảm biến và theo dõi pin khi hai lõi CPU ở chế độ Deep Sleep. |
| **ADC** | 2 bộ SAR 12-bit, tối đa 18 kênh | Đo điện áp và tín hiệu tương tự với độ phân giải $4096$ mức. |
| **Giao tiếp nối tiếp** | 3 UART, 2 I2C, 3 SPI, 2 I2S, 1 CAN | Hỗ trợ các giao thức công nghiệp. |
| **Không dây** | Wi-Fi 802.11 b/g/n 2.4 GHz; Bluetooth 4.2 BR/EDR/BLE | Truyền dữ liệu tầm gần và kết nối Web LMS. |
| **Điện áp hoạt động** | 3.0–3.6 V; khuyến nghị 3.3 V | Tương thích nguồn 3.3 V từ mạch Buck của pin Lithium EPS. |

---

### 3. Sơ đồ chân NodeMCU-32S (38 chân)

Module **ESP32-WROOM-32** thường được gắn trên bo mạch phát triển **NodeMCU-32S 38 chân** để thử nghiệm và tích hợp vào OBC. Ma trận GPIO cho phép gán linh hoạt nhiều tín hiệu ngoại vi; cần tuân thủ chức năng và giới hạn điện áp của từng chân.

![Sơ đồ chân đầy đủ của bo mạch ESP32 NodeMCU-32S](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab1.1_esp32.png)

#### Chức năng các chân NodeMCU-32S

| Nhóm chân | Ký hiệu | Chức năng | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Nguồn & Điều khiển** | **3V3**, **5V0**, **GND** | `3V3`: Đường nguồn ổn áp 3.3V; `5V0`: Nguồn vào/ra 5V từ cổng USB Type-C; `GND`: Mass chung hệ thống. | Nguồn cấp logic cho ESP32 tối đa 3.3V (không cấp 5V vào chân GPIO). |
| **Khởi động lại** | **EN** | Tích cực mức `HIGH`; kéo xuống `LOW` để khởi động lại chip. | Kết hợp với `GPIO0` (`IO0`) qua mạch DTR/RTS để nạp firmware tự động. |
| **Chỉ nhận tín hiệu** | **GPIO36** (`S_VP`), **GPIO39** (`S_VN`), **GPIO34** (`VDET_1`), **GPIO35** (`VDET_2`) | Các kênh ADC 12-bit `ADC1_0`, `ADC1_3`, `ADC1_6`, `ADC1_7`; thuộc vùng RTC. | Không xuất tín hiệu và không có điện trở kéo nội. Dùng đọc cảm biến analog, điện áp hoặc tín hiệu `AUX`. |
| **Cảm ứng, RTC, DAC và ADC** | **GPIO32** (`TOUCH9 / ADC1_4`), **GPIO33** (`TOUCH8 / ADC1_5`), **GPIO25** (`DAC_1`), **GPIO26** (`DAC_2`), **GPIO27** (`TOUCH7 / ADC2_7`) | Hỗ trợ I/O hai chiều, PWM, ngắt ngoài, DAC 8-bit và dao động 32 kHz. | Dùng điều khiển ngoại vi, UART1 qua ma trận GPIO hoặc PWM. `ADC2` không dùng đồng thời với Wi-Fi. |
| **Bus I2C Mặc định (Wire)** | **GPIO21** (`WIRE_SDA`), **GPIO22** (`WIRE_SCL`) | Đường dữ liệu `SDA` và xung nhịp `SCL` mặc định của thư viện `Wire.h`. | Giao tiếp chuẩn I2C với **MAX17048** (`0x36`), **BMP280** (`0x76/0x77`), **MPU6500** (`0x68`) và **GY-271** (`0x0D`). |
| **Cổng UART0** | **GPIO1** (`U0TXD / SERIAL_TX`), **GPIO3** (`U0RXD / SERIAL_RX`) | Nối với chip USB-UART CP2102 hoặc CH340. | Nạp firmware qua Web IDE và xem dữ liệu ở Serial Monitor (115200 bps). Tránh gắn tải gây xung đột mức logic. |
| **Cổng UART2 (Phần cứng)** | **GPIO17** (`TX2`), **GPIO16** (`RX2`) | Kênh truyền/nhận nối tiếp phần cứng độc lập `Serial2`. | Giao tiếp tốc độ cao với module định vị **GNSS L76X** và **Camera OV2640** thông qua IC đa hóa **74HC4066**. |
| **Bus VSPI (SPI mặc định)** | **GPIO23** (`VSPI_MOSI`), **GPIO19** (`VSPI_MISO`), **GPIO18** (`VSPI_SCK`), **GPIO5** (`VSPI_SS`) | Giao tiếp SPI tốc độ cao (tối đa 40 MHz); `GPIO5` kiêm chân strapping `SDIO`. | Kết nối khe microSD để lưu nhật ký bay và ảnh JPEG. |
| **Bus HSPI & JTAG** | **GPIO13** (`HSPI_MOSI / MTCK`), **GPIO12** (`HSPI_MISO / MTDI`), **GPIO14** (`HSPI_SCK / MTMS`), **GPIO15** (`HSPI_SS / MTDO`) | Kênh SPI thứ hai (`HSPI`) đồng thời phục vụ giao diện gỡ lỗi JTAG và ADC2 / Touch. | Trong **Lab 1** (trên bo rời): `GPIO13, 14, 15` dùng điều khiển IC dịch chốt `74HC595`. **Lưu ý:** `GPIO12` và `GPIO15` là chân Strapping nhạy cảm khi khởi động. |
| **Chân Điều khiển & Strapping** | **GPIO0** (`BOOT`), **GPIO2** (`LED`), **GPIO4** (`ADC2_0 / TOUCH0`) | I/O đa năng hỗ trợ PWM, ADC2, Touch. `GPIO2` thường gắn sẵn LED xanh trên bo; `GPIO0` nối nút `IO0/BOOT`. | Trong **Lab 1**: `GPIO2` điều khiển LED đơn, `GPIO4` đọc nút nhấn `INPUT_PULLUP`. Khi tích hợp kit: dùng điều khiển chân chọn `M0/M1`, nút nhấn hoặc tín hiệu chuyển mạch. |
| **Chân Flash Nội (! CẤM DÙNG)** | **GPIO6** (`SCK`), **GPIO7** (`D0`), **GPIO8** (`D1`), **GPIO9** (`D2`), **GPIO10** (`D3`), **GPIO11** (`CMD`) | Đường kết nối nội bộ giữa lõi ESP32 và chip nhớ **SPI Flash 4 MB** bên trong vỏ bọc kim loại. | **TUYỆT ĐỐI KHÔNG SỬ DỤNG** làm GPIO thường (cũng không dùng trực tiếp chân UART1 mặc định `GPIO9/10` mà phải remap qua GPIO Matrix). |

---

### 4. Chân Strapping và lưu ý phần cứng

ESP32 lấy mẫu mức logic trên một số chân GPIO khi tín hiệu reset chuyển từ $0$ lên $1$. Mức này quyết định chế độ khởi động:

![Luồng khởi động và chân Strapping Pin ESP32](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_strapping_flow.png)

| Tên Chân GPIO | Chức năng Strapping | Mức Logic Boot Bình thường | Mức Logic Nạp Firmware | Cảnh báo An toàn Thiết kế |
| :--- | :--- | :--- | :--- | :--- |
| **GPIO 0** | Chọn chế độ khởi động | **HIGH (kéo lên 3.3 V)** | **LOW (nối GND)** | Có điện trở kéo lên nội. Nếu bị kéo xuống khi khởi động, ESP32 vào chế độ nạp. |
| **GPIO 2** | Chọn chế độ khởi động và nạp Flash | **LOW hoặc không nối** | **LOW (nối GND)** | Thường nối với LED trên bo. Không kéo chân lên mức cao khi khởi động. |
| **GPIO 12 (MTDI)** | Chọn điện áp cho Flash nội (`VDD_SDIO`) | **LOW (3.3 V)** | **LOW (3.3 V)** | Nếu GPIO 12 bị kéo lên HIGH khi khởi động, chip đặt nguồn Flash ở 1.8 V khiến ESP32 không đọc được bộ nhớ. |
| **GPIO 15 (MTDO)** | Bật/tắt thông báo gỡ lỗi trên UART0 | **HIGH (kéo lên 3.3 V)** | **HIGH (kéo lên 3.3 V)** | Quyết định ROM Bootloader có gửi thông báo qua cổng nối tiếp ở 115200 bps hay không. |

> [!WARNING]
> **Lưu ý khi thiết kế OBC:**
>
> - Không dùng GPIO 6, 7, 8, 9, 10, 11 làm chân GPIO thông thường vì chúng nối trực tiếp với bộ nhớ SPI Flash bên trong module.
> - GPIO 34, 35, 36 (VP) và 39 (VN) chỉ nhận tín hiệu, không có điện trở kéo nội và không thể cấu hình làm ngõ ra `OUTPUT`.

---

---

## 📖 Doc 1.2: Web IDE và nạp firmware qua Web Serial API

### 1. Soạn và biên dịch trên PTITCube Web IDE

PTITCube Web IDE cho phép viết mã, biên dịch và nạp chương trình cho ESP32 ngay trên trình duyệt, không cần cài Python, GCC, ESP-IDF hoặc trình điều khiển USB-UART.

![Quy trình biên dịch đám mây và nạp Web Serial](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_cloud_compile_flow.png)

- **Monaco Editor:** Tô màu cú pháp C++, gợi ý mã bằng IntelliSense, tự động thụt lề và phát hiện lỗi cú pháp.
- **Biên dịch:** Máy chủ tạo môi trường chứa `xtensa-esp32-elf-gcc`, biên dịch mã, liên kết thư viện PTITCube và tạo tệp nhị phân `firmware.bin` trong vòng dưới 5 giây.

### 2. Nạp firmware qua Web Serial API

Web Serial API là tiêu chuẩn web của W3C, cho phép trang HTTPS giao tiếp hai chiều với thiết bị nối qua USB.

1. Nhấn **Kết nối OBC**. Lệnh `navigator.serial.requestPort()` mở hộp thoại chọn cổng COM của chip USB-UART (CP2102 hoặc CH340).
2. Cổng nối tiếp dùng tốc độ 115200 bps và định dạng **SERIAL_8N1**: 8 bit dữ liệu, không bit chẵn lẻ, 1 bit dừng.
3. Mạch hai transistor NPN trên bo OBC dùng tín hiệu **DTR** và **RTS** để tự khởi động lại ESP32 và đưa chip vào chế độ nạp:
   - DTR ở mức HIGH, RTS ở mức LOW: kéo `EN` xuống 0 V để khởi động lại.
   - DTR ở mức LOW, RTS ở mức HIGH: kéo `GPIO0` xuống 0 V để vào chế độ nạp.
   - Nhả `EN` lên 3.3 V khi `GPIO0` vẫn ở mức 0 V để mở ROM Bootloader.
4. Dữ liệu trong `firmware.bin` được đóng khung theo SLIP rồi ghi vào bộ nhớ Flash SPI. Khi hoàn tất, ESP32 khởi động lại và chạy chương trình mới.

---

---

## 📖 Doc 1.3: IC 74HC595 và LED 7 đoạn

### 1. Cách hoạt động của IC 74HC595

IC **74HC595** chuyển dữ liệu nối tiếp thành 8 ngõ ra song song. Bên trong có thanh ghi dịch và thanh ghi chốt:

![Cấu trúc vi mạch ghi dịch 74HC595 và LED 7 đoạn](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_74hc595_structure.png)

- **Thanh ghi dịch:** Mỗi cạnh lên trên `SH_CP` ghi bit tại `DS` vào thanh ghi và đẩy các bit trước đó sang vị trí kế tiếp.
- **Thanh ghi chốt:** Giữ nguyên trạng thái các ngõ ra `Q0`–`Q7` trong lúc nạp dữ liệu. Cạnh lên tại `ST_CP` chuyển đồng thời 8 bit sang ngõ ra, tránh nhấp nháy LED khi dịch dữ liệu.

### 2. Điều khiển LED 7 đoạn

LED 7 đoạn gồm các đoạn `a`–`g` và chấm thập phân `dp`. Có hai kiểu đấu: anode chung và cathode chung.

- Với loại **cathode chung**, xuất mức **HIGH (1)** để bật một đoạn và **LOW (0)** để tắt.
- Bảng mã dưới đây hiển thị các chữ số 0–9. Thứ tự bit `Q7`–`Q0` tương ứng `dp, g, f, e, d, c, b, a`.

| Số | Các đoạn bật | Mã nhị phân (`dp-g-f-e-d-c-b-a`) | Mã hexa |
| :---------: | :---------------------: | :-----------------------------------------: | :-------------: |
| **0** |  $a, b, c, d, e, f$  |                `00111111`                |    `0x3F`    |
| **1** |        $b, c$        |                `00000110`                |    `0x06`    |
| **2** |    $a, b, d, e, g$    |                `01011011`                |    `0x5B`    |
| **3** |    $a, b, c, d, g$    |                `01001111`                |    `0x4F`    |
| **4** |     $b, c, f, g$     |                `01100110`                |    `0x66`    |
| **5** |    $a, c, d, f, g$    |                `01101101`                |    `0x6D`    |
| **6** |  $a, c, d, e, f, g$  |                `01111101`                |    `0x7D`    |
| **7** |       $a, b, c$       |                `00000111`                |    `0x07`    |
| **8** | $a, b, c, d, e, f, g$ |                `01111111`                |    `0x7F`    |
| **9** |  $a, b, c, d, f, g$  |                `01101111`                |    `0x6F`    |

---

---

## 📖 Doc 1.4: Giao tiếp UART

### 1. Nguyên lý truyền dữ liệu

**UART** truyền dữ liệu nối tiếp giữa hai thiết bị mà không cần chung đường xung nhịp. Hai bên cần dùng cùng tốc độ truyền và căn thời điểm theo cạnh tín hiệu.

- Dùng hai đường tín hiệu:
  - `TX`: truyền dữ liệu.
  - `RX`: nhận dữ liệu. Nối chéo hai thiết bị: `TX_A` → `RX_B`, `RX_A` → `TX_B`.

![Sơ đồ kết nối UART hai chiều](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_uart_connection.png)

![Sơ đồ khung truyền UART 8-N-1](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_uart_timing_sequence.png)

### 2. Khung truyền SERIAL_8N1

Khung **SERIAL_8N1** gồm:

- 8 bit dữ liệu, tương ứng 1 byte.
- Không có bit kiểm tra chẵn lẻ.
- 1 bit bắt đầu và 1 bit dừng.

Mỗi byte cần truyền 10 bit. Ở tốc độ 115200 bps, thời gian truyền là:

$$T_{\text{bit}} = \frac{1}{115200} \approx 8.68\,\mu\text{s} \implies T_{\text{byte}} = 10 \times 8.68\,\mu\text{s} \approx 86.8\,\mu\text{s}$$

Tốc độ dữ liệu tối đa xấp xỉ 11520 byte/s, tương đương 11.25 KB/s.

### 3. Ba cổng UART trên ESP32

ESP32 có ba cổng UART: `UART0`, `UART1` và `UART2`. Ma trận GPIO cho phép gán tín hiệu TX/RX cho các chân phù hợp trên NodeMCU-32S:

![Vị trí ba cổng UART trên ESP32 NodeMCU-32S](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab1.1_esp32.png)

1. **UART0 (`Serial`, TX: `GPIO1`, RX: `GPIO3`):** Nối với chip USB-UART CP2102 hoặc CH340, dùng để nạp mã và xem nhật ký ở 115200 bps.
2. **UART1 (`Serial1`):** Chân mặc định `GPIO9` và `GPIO10` dùng chung với bộ nhớ Flash. Khi khởi tạo, gán UART1 sang chân GPIO khác bằng `Serial1.begin(baud, SERIAL_8N1, RX_PIN, TX_PIN)`. Ví dụ: kết nối LoRa E32.
3. **UART2 (`Serial2`, TX: `GPIO17`, RX: `GPIO16`):** Dùng với GNSS L76X ở 9600 bps và camera OV2640 ở 115200 bps thông qua IC 74HC4066.

| Cổng | Đối tượng | Chân | Kết nối và ứng dụng |
| :--- | :--- | :--- | :--- |
| **UART0** | `Serial` | TX `GPIO1`<br>RX `GPIO3` | USB-UART; nạp chương trình, xem nhật ký và nhận lệnh CLI. |
| **UART1** | `Serial1` / `HardwareSerial(1)` | Gán qua ma trận GPIO; không dùng `GPIO9/10`. | Kết nối LoRa E32 (433 MHz). |
| **UART2** | `Serial2` / `HardwareSerial(2)` | TX `GPIO17`<br>RX `GPIO16` | Nhận NMEA từ GNSS L76X và ảnh JPEG từ OV2640 qua `74HC4066`; độc lập với nạp USB. |

---

---

## 📖 Doc 1.5: Wi-Fi, máy chủ HTTP và SoftAP

### 1. Wi-Fi trên ESP32

Wi-Fi trên ESP32 hỗ trợ chuẩn 802.11 b/g/n ở băng tần 2.4 GHz. Có thể dùng theo hai chế độ:

- **STA:** ESP32 kết nối mạng Wi-Fi có sẵn và nhận địa chỉ IP từ máy chủ DHCP.
- **SoftAP:** ESP32 tạo mạng Wi-Fi riêng, cho phép thiết bị kết nối trực tiếp mà không cần mạng ngoài. Địa chỉ mặc định là `192.168.4.1`.

![Kiến trúc SoftAP và máy chủ HTTP trên ESP32](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_softap_webserver.png)

### 2. Máy chủ HTTP nhúng

OBC dùng thư viện `WebServer.h` trên cổng 80:

- Dùng `server.on("/endpoint", HTTP_METHOD, handler_function)` để gán hàm xử lý cho từng địa chỉ và phương thức HTTP.
- Gọi `server.handleClient()` liên tục trong `loop()` để xử lý yêu cầu từ trình duyệt mà không chặn các tác vụ khác.

---

---

## 📖 Doc 1.6: Định thời bằng `millis()` và xử lý tràn số

![Định thời không chặn và xử lý tràn số](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab1_millis_nonblocking.png)

### 1. Vì sao cần tránh `delay()`?

`delay(ms)` chặn chương trình trong khoảng thời gian chỉ định.

- Với `delay(1000)`, CPU chờ trong $1000\text{ ms}$.
- Trong thời gian đó, chương trình không đọc cảm biến, xử lý dữ liệu Serial, phục vụ yêu cầu từ trạm mặt đất hay phát hiện sự cố.
- Nếu thời gian chờ vượt chu kỳ Watchdog Timer, WDT phát hiện CPU bị treo và khởi động lại hệ thống.

### 2. Định thời không chặn bằng `millis()`

Thay `delay()` bằng bộ đếm thời gian `millis()` để chương trình tiếp tục xử lý các tác vụ khác.

- `millis()` trả về số mili-giây từ lúc ESP32 khởi động. Trên ESP32, giá trị dùng kiểu số nguyên không dấu 32-bit (`unsigned long`).
- Lưu thời điểm chạy gần nhất vào `previousMillis`. Trong `loop()`, chỉ chạy tác vụ khi `currentMillis - previousMillis` lớn hơn hoặc bằng `INTERVAL`, sau đó cập nhật mốc thời gian:

```cpp
unsigned long previousMillis = 0;
const unsigned long INTERVAL = 1000; // Chu kỳ 1000 ms (1 giây)

void loop() {
    unsigned long currentMillis = millis();
  
    // Chỉ chạy khi đã đủ chu kỳ
    if (currentMillis - previousMillis >= INTERVAL) {
        previousMillis = currentMillis;
      
        // Đọc cảm biến và gửi dữ liệu
        readSensorsAndSendTelemetry();
    }
  
    // Tiếp tục xử lý các tác vụ khác
    handleSerialCommands();
    server.handleClient();
}
```

### 3. Xử lý tràn số

Trên ESP32, `unsigned long` là số nguyên không dấu 32-bit, có giá trị từ $0$ đến $2^{32} - 1 = 4.294.967.295$:

- Sau khoảng thời gian:

$$T_{\text{overflow}} = \frac{4294967295\text{ ms}}{1000 \times 60 \times 60 \times 24} \approx 49.71\text{ days}$$

  giá trị `millis()` quay vòng về $0$.
- Dùng phép trừ thời điểm hiện tại cho thời điểm trước đó:

$$\text{currentMillis} - \text{previousMillis} \ge \text{INTERVAL}$$

  Phép trừ vẫn cho khoảng thời gian đã trôi qua khi bộ đếm quay vòng. Ví dụ, nếu `previousMillis = 4,294,967,200` và `currentMillis = 50`:

$$50 - 4294967200 = 146\text{ ms}$$

  Kết quả là 146 ms.

---

## 📖 Doc 1.7: Hướng dẫn lắp ráp bo thí nghiệm

Tài liệu này hướng dẫn đấu nối các linh kiện dùng trong Lab 1 trên bo thí nghiệm rời. Không kết nối mạch thực hành với các bo EPS, Sensor hoặc COM của vệ tinh.

### 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí tổng thể linh kiện trên bo thí nghiệm Lab 1.

### 2. Kết nối linh kiện

| Linh kiện | Kết nối |
| :--- | :--- |
| LED đơn | Chân Anode nối qua điện trở hạn dòng đến `GPIO 2`; chân Cathode nối GND. |
| Nút nhấn | Một chân nối `GPIO 4`; chân còn lại nối GND. Dùng `INPUT_PULLUP`. |
| 74HC595 | `DS` nối `GPIO 13`; `ST_CP` nối `GPIO 14`; `SH_CP` nối `GPIO 15`; `VCC` nối 3.3V; `GND` nối GND. |

> **Vị trí chèn hình:** Đấu nối LED đơn và nút nhấn.

> **Vị trí chèn hình:** Đấu nối 74HC595 và LED 7 đoạn.

### 3. Đối chiếu trước khi thực hành

- Dùng bo thí nghiệm rời và kết nối ESP32 bằng dây jumper.
- Kiểm tra từng dây theo bảng kết nối và các hình sẽ được bổ sung.
- Giữ nguyên cấu hình chân trong bài thực hành tương ứng.

---

## 💻 Hướng dẫn thực hành Lab 1

> **Thiết bị thực hành:** Dùng bo thí nghiệm rời nối với ESP32 bằng dây jumper. Không nối bo này với các bo chính EPS, Sensor hoặc COM. Xem [hướng dẫn lắp ráp](./Lab1_Doc1.7.md) để biết các chân kết nối.

Thực hiện lần lượt 11 bài thực hành trên Web IDE:

### Bài 1.1: Nháy LED

- **Mục tiêu:** Cấu hình chân GPIO làm `OUTPUT` và điều khiển bật/tắt LED theo chu kỳ.

```cpp
const int LED_PIN = 2;

void setup() {
    pinMode(LED_PIN, OUTPUT);
}

void loop() {
    digitalWrite(LED_PIN, HIGH);
    delay(500);
    digitalWrite(LED_PIN, LOW);
    delay(500);
}
```

### Bài 1.2: Định thời bằng `millis()`

- **Mục tiêu:** Thay thế hoàn toàn hàm `delay()` bằng thuật toán định thời non-blocking.

```cpp
const int LED_PIN = 2;
unsigned long prevTime = 0;
const unsigned long INTERVAL = 500;
bool ledState = false;

void setup() {
    pinMode(LED_PIN, OUTPUT);
}

void loop() {
    unsigned long curTime = millis();
    if (curTime - prevTime >= INTERVAL) {
        prevTime = curTime;
        ledState = !ledState;
        digitalWrite(LED_PIN, ledState ? HIGH : LOW);
    }
}
```

### Bài 1.3: Đọc công tắc hành trình phóng

- **Mục tiêu:** Cấu hình chân GPIO làm `INPUT_PULLUP` để đọc trạng thái đóng/ngắt của công tắc hành trình phóng mà không cần gắn thêm điện trở ngoài.

```cpp
const int SW_PIN = 4;
const int LED_PIN = 2;

void setup() {
    pinMode(SW_PIN, INPUT_PULLUP);
    pinMode(LED_PIN, OUTPUT);
}

void loop() {
    // Khi công tắc mở (vệ tinh ra khỏi ống phóng): chân đọc mức HIGH
    int state = digitalRead(SW_PIN);
    digitalWrite(LED_PIN, (state == HIGH) ? HIGH : LOW);
}
```

### Bài 1.4: Điều chỉnh độ sáng LED bằng PWM

- **Mục tiêu:** Cấu hình khối phần cứng LEDC PWM của ESP32 để điều chỉnh mức điện áp hiệu dụng ngõ ra.

```cpp
const int LED_PIN = 2;
const int PWM_CH = 0;
const int PWM_FREQ = 5000; // Tần số 5 kHz
const int PWM_RES = 8;     // Độ phân giải 8-bit (0 - 255)

void setup() {
    ledcSetup(PWM_CH, PWM_FREQ, PWM_RES);
    ledcAttachPin(LED_PIN, PWM_CH);
}

void loop() {
    // Tăng độ sáng từ từ
    for (int duty = 0; duty <= 255; duty++) {
        ledcWrite(PWM_CH, duty);
        delay(5);
    }
    // Giảm độ sáng từ từ
    for (int duty = 255; duty >= 0; duty--) {
        ledcWrite(PWM_CH, duty);
        delay(5);
    }
}
```

### Bài 1.5: Gửi dữ liệu qua cổng nối tiếp

- **Mục tiêu:** Xuất dữ liệu bản tin viễn trắc ra cổng nối tiếp ở tốc độ 115200 bps.

```cpp
unsigned long packetId = 0;

void setup() {
    Serial.begin(115200);
    while (!Serial) { ; } // Chờ kết nối cổng
    Serial.println("=== OBC BOOT COMPLETED ===");
}

void loop() {
    packetId++;
    Serial.print("[BEACON] ID: ");
    Serial.print(packetId);
    Serial.print(" | Uptime: ");
    Serial.print(millis() / 1000);
    Serial.println("s | Status: OK");
    delay(1000);
}
```

### Bài 1.6: Nhận lệnh qua cổng nối tiếp

- **Mục tiêu:** Lắng nghe luồng ký tự từ bàn phím và thực thi lệnh tương ứng.

```cpp
const int LED_PIN = 2;

void setup() {
    pinMode(LED_PIN, OUTPUT);
    Serial.begin(115200);
    Serial.println("Send '1' to ON, '0' to OFF");
}

void loop() {
    if (Serial.available() > 0) {
        char cmd = Serial.read();
        if (cmd == '1') {
            digitalWrite(LED_PIN, HIGH);
            Serial.println("ACK: LED TURNED ON");
        } else if (cmd == '0') {
            digitalWrite(LED_PIN, LOW);
            Serial.println("ACK: LED TURNED OFF");
        }
    }
}
```

### Bài 1.7: Điều khiển 74HC595 và LED đơn

- **Mục tiêu:** Lập trình dịch 8 bit nối tiếp qua hàm `shiftOut()`.

```cpp
const int PIN_DATA  = 13; // DS
const int PIN_LATCH = 14; // ST_CP
const int PIN_CLOCK = 15; // SH_CP

void setup() {
    pinMode(PIN_DATA, OUTPUT);
    pinMode(PIN_LATCH, OUTPUT);
    pinMode(PIN_CLOCK, OUTPUT);
}

void loop() {
    for (byte b = 1; b != 0; b <<= 1) {
        digitalWrite(PIN_LATCH, LOW);
        shiftOut(PIN_DATA, PIN_CLOCK, MSBFIRST, b);
        digitalWrite(PIN_LATCH, HIGH);
        delay(200);
    }
}
```

### Bài 1.8: Đếm từ 0 đến 9 trên LED 7 đoạn

- **Mục tiêu:** Sử dụng mảng mã Hex điều khiển hiển thị các chữ số đếm trên LED 7 đoạn.

```cpp
const int PIN_DATA  = 13;
const int PIN_LATCH = 14;
const int PIN_CLOCK = 15;

const byte digitCodes[] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66, // 0, 1, 2, 3, 4
    0x6D, 0x7D, 0x07, 0x7F, 0x6F  // 5, 6, 7, 8, 9
};

void setup() {
    pinMode(PIN_DATA, OUTPUT);
    pinMode(PIN_LATCH, OUTPUT);
    pinMode(PIN_CLOCK, OUTPUT);
}

void loop() {
    for (int i = 0; i < 10; i++) {
        digitalWrite(PIN_LATCH, LOW);
        shiftOut(PIN_DATA, PIN_CLOCK, MSBFIRST, digitCodes[i]);
        digitalWrite(PIN_LATCH, HIGH);
        delay(1000);
    }
}
```

### Bài 1.9: Tạo mạng Wi-Fi bằng SoftAP

- **Mục tiêu:** Phát sóng mạng Wi-Fi mang tên vệ tinh với mật khẩu bảo mật WPA2.

```cpp
#include <WiFi.h>

const char* ssid = "PTITCube-OBC";
const char* pass = "12345678";

void setup() {
    Serial.begin(115200);
    WiFi.softAP(ssid, pass);
    Serial.print("Access Point Created. IP: ");
    Serial.println(WiFi.softAPIP());
}

void loop() {
    // Duy trì hoạt động mạng
}
```

### Bài 1.10: Tạo máy chủ HTTP giám sát dữ liệu

- **Mục tiêu:** Phục vụ trang HTML hiển thị trạng thái sinh tồn của vệ tinh qua trình duyệt.

```cpp
#include <WiFi.h>
#include <WebServer.h>

WebServer server(80);

void handleRoot() {
    String html = "<h1>PTITCube OBC Live Telemetry</h1>";
    html += "<p>Uptime: " + String(millis() / 1000) + " seconds</p>";
    html += "<p>Status: NOMINAL (Core 1 Active)</p>";
    server.send(200, "text/html", html);
}

void setup() {
    Serial.begin(115200);
    WiFi.softAP("PTITCube-OBC", "12345678");
    server.on("/", handleRoot);
    server.begin();
    Serial.println("HTTP Server Started");
}

void loop() {
    server.handleClient();
}
```

### Bài 1.11: Điều khiển LED qua API và hiển thị đồng thời

- **Mục tiêu:** Tổng hợp toàn bộ kỹ năng lập trình nhúng của Lab 1: kết hợp máy chủ Web API, điều khiển GPIO, dịch dữ liệu 74HC595 và định thời non-blocking trong cùng một chương trình bay duy nhất.

```cpp
#include <WiFi.h>
#include <WebServer.h>

WebServer server(80);
const int LED_PIN = 2;
unsigned long prevBlink = 0;
bool ledState = false;

void handleApiToggle() {
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState ? HIGH : LOW);
    server.send(200, "application/json", "{\"status\":\"success\",\"led\":" + String(ledState) + "}");
}

void setup() {
    pinMode(LED_PIN, OUTPUT);
    Serial.begin(115200);
    WiFi.softAP("PTITCube-OBC", "12345678");
  
    server.on("/api/toggle", HTTP_POST, handleApiToggle);
    server.begin();
}

void loop() {
    server.handleClient();
  
    // Tác vụ nhịp tim hệ thống non-blocking
    if (millis() - prevBlink >= 1000) {
        prevBlink = millis();
        Serial.printf("[HEARTBEAT] Free Heap: %d bytes\n", ESP.getFreeHeap());
    }
}
```
