# Lab 4: Cảm biến và tải trọng quang học

---

## 📖 Doc 4.1: Giao thức I2C và mạng cảm biến

### 1. Vai trò của cảm biến và tải trọng
Phân hệ Cảm biến đóng vai trò là "Giác quan" của vệ tinh CubeSat, cho phép vệ tinh cảm nhận được các thông số vật lý của bản thân và môi trường quỹ đạo:
- **Theo dõi môi trường:** Đo áp suất và nhiệt độ bên trong khung để phát hiện rò khí hoặc quá nhiệt.
- **Xác định tư thế:** Đo vận tốc góc và gia tốc bằng IMU 6 trục để hỗ trợ giảm tốc độ quay.
- **Định vị và thời gian:** Cung cấp tọa độ, độ cao và thời gian UTC cho các phép đo.
- **Chụp ảnh:** Ghi ảnh bề mặt Trái Đất phục vụ nghiên cứu.

### 2. Nguyên lý I2C
Bus **I2C** do hãng Philips (nay là NXP) phát triển là một bus truyền thông nối tiếp đồng bộ 2 dây hai chiều, cực kỳ phổ biến trong các hệ thống vệ tinh nhỏ nhờ khả năng kết nối nhiều IC cảm biến chỉ với hai chân vi điều khiển duy nhất:
- **Hai Đường Tín hiệu:**
  - **SDA:** Đường dữ liệu hai chiều.
  - **SCL:** Xung nhịp đồng bộ do thiết bị chủ phát.
- **Ngõ ra cực thu hở và điện trở kéo lên:**
  - Các thiết bị chỉ kéo đường tín hiệu xuống mức `LOW` hoặc thả nổi.
  - Một cặp điện trở kéo lên ngoài ($R_p$, thông thường từ 2.2 kΩ đến 4.7 kΩ) bắt buộc phải được mắc từ đường SDA và SCL lên nguồn 3.3V.
  - Khi không có thiết bị nào truyền tin, điện trở kéo lên giữ cho đường bus luôn ở mức logic `HIGH`.
  - Cách mắc này cho phép nhiều thiết bị dùng chung bus mà không gây chập khi truyền đồng thời.

![Sơ đồ kết nối bus I2C đa cảm biến và điện trở kéo lên](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_i2c_connection.png)

![Trình tự giao thức đọc thanh ghi I2C](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_i2c_protocol_sequence.png)

### 3. Khung truyền và trạng thái điều khiển
1. **Điều kiện START và STOP:**
   - Trong quá trình truyền dữ liệu bình thường, đường SDA chỉ được phép thay đổi trạng thái khi SCL đang ở mức `LOW`. Khi SCL ở mức `HIGH`, đường SDA bắt buộc phải giữ ổn định.
   - **START Condition:** SDA chuyển trạng thái từ `HIGH` xuống `LOW` trong khi SCL đang ở mức `HIGH`. Báo hiệu đường bus bắt đầu bận.
   - **STOP Condition:** SDA chuyển trạng thái từ `LOW` lên `HIGH` trong khi SCL đang ở mức `HIGH`. Báo hiệu kết thúc phiên truyền tin, trả bus về trạng thái rảnh rỗi.
2. **Địa chỉ Thiết bị 7-bit & Bit R/W:**
   - Byte đầu tiên sau điều kiện START bao gồm 7 bit địa chỉ thiết bị Slave (2⁷ = 128 địa chỉ khả dĩ) và bit thứ 8 là bit chọn chiều dữ liệu:
     - Bit 8 = `0`: Thao tác Ghi dữ liệu từ Master sang Slave.
     - Bit 8 = `1`: Thao tác Đọc dữ liệu từ Slave về Master.
3. **Bit xác nhận ACK và phản hồi NACK:**
   - Trong nhịp xung SCL thứ 9, thiết bị nhận dữ liệu phải phản hồi một bit:
     - **ACK:** Kéo SDA xuống `LOW` để báo đã nhận dữ liệu.
     - **NACK:** Để SDA ở `HIGH` để báo không nhận hoặc kết thúc đọc.
4. **Kéo dãn xung nhịp:**
   - Khi một cảm biến Slave chậm chưa kịp chuẩn bị xong dữ liệu hoặc đang bận chuyển đổi ADC nội bộ, nó có quyền kéo đường SCL xuống mức `LOW`. Master phát hiện đường SCL bị giữ ở mức thấp sẽ tự động tạm dừng đồng hồ xung nhịp cho tới khi Slave nhả SCL lên mức cao mới truyền tiếp.
5. **Đọc khối dữ liệu:**
   - Hầu hết các cảm biến hiện đại (BMP280, MPU6500) đều hỗ trợ con trỏ địa chỉ tự tăng. Master chỉ cần gửi địa chỉ thanh ghi khởi đầu một lần duy nhất, sau đó phát liên tục các xung nhịp đọc; cảm biến sẽ tự động đẩy tuần tự toàn bộ mảng dữ liệu (ví dụ 14 byte từ 0x3B đến 0x48 của MPU6500) mà không cần lặp lại điều kiện START/STOP, giúp tăng tốc độ đọc dữ liệu lên gấp 5 lần.

---

### 4. Cảm biến BMP280
- **Nhà sản xuất & Địa chỉ I2C:** Bosch Sensortec. Địa chỉ I2C cấu hình bằng chân SDO: `0x76` (SDO nối GND) hoặc `0x77` (SDO nối VDD).
- **Dải Đo & Độ phân giải:** Đo áp suất khí quyển từ 300 hPa đến 1100 hPa (tương đương độ cao từ $-500\text{m}$ dưới mực nước biển đến $+9000\text{m}$ trên đỉnh Everest) với độ phân giải siêu cao 0.16 Pa và độ phân giải nhiệt độ 0.01°C.
- **Hệ số hiệu chuẩn nội bộ:**
  - Màng cảm biến áp suất gốm áp điện (Piezoresistive) có đặc tính phi tuyến tính cao theo nhiệt độ. Trong quá trình sản xuất tại nhà máy, mỗi chip BMP280 được đo kiểm riêng lẻ và nạp sẵn 24 byte hằng số hiệu chuẩn vào bộ nhớ ROM không khả biến (từ thanh ghi `0x88` đến `0xA1`: `dig_T1` đến `dig_T3` cho nhiệt độ, `dig_P1` đến `dig_P9` cho áp suất).
  - Thuật toán giải mã bắt buộc phải đọc toàn bộ mảng 24 byte hằng số này và áp dụng công thức bù trừ nhiệt độ bậc cao theo tài liệu chuẩn của Bosch để quy đổi giá trị ADC thô thành giá trị áp suất thực tế.
- **Công thức Tính Độ cao Khí quyển (Barometric Hypsometric Formula):**

$$h = 44330 \times \left[ 1 - \left(\frac{P}{P_0}\right)^{\frac{1}{5.255}} \right]$$

  Trong đó: $h$ là độ cao tính bằng mét, $P$ là áp suất khí quyển đo được, $P_0$ là áp suất chuẩn tại mực nước biển (1013.25 hPa).

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 4.1)
1. **Bus I2C chuẩn công nghiệp yêu cầu cấu hình phần cứng đặc thù nào trên hai đường tín hiệu SDA và SCL?**
   - A. Cấu hình ngõ ra Push-Pull trực tiếp
   - B. Nối đất trực tiếp cả hai chân tín hiệu
   - C. Cấu hình cực máng hở kết hợp điện trở kéo lên
   - D. Nối tiếp tụ điện cách ly một chiều

2. **Cảm biến khí quyển BMP280 đo đạc và cung cấp các thông số vật lý nào thông qua giao thức I2C?**
   - A. Áp suất khí quyển, Nhiệt độ môi trường và Độ cao khí áp
   - B. Vận tốc góc và từ trường Trái Đất
   - C. Nồng độ khí oxy và độ ẩm tương đối
   - D. Tọa độ kinh độ và vĩ độ vị trí

---

---

## 📖 Doc 4.2: MPU6500, GY-271 và bộ lọc bù

### 1. Cảm biến MPU6500
- **Nhà sản xuất & Địa chỉ I2C:** InvenSense / TDK. Địa chỉ I2C là `0x68` (chân `ADO = GND`) hoặc `0x69` (chân `ADO = VCC`).
- **Cấu trúc Phần cứng:** Tích hợp một vi cơ điện tử gia tốc kế 3 trục (MEMS 3-axis Accelerometer) và con quay hồi chuyển 3 trục (MEMS 3-axis Gyroscope) trên cùng một đế silicon theo hệ trục tọa độ $X, Y, Z$ in trực tiếp trên bo mạch.
- **Thanh ghi Dữ liệu Thô:** 14 thanh ghi liên tiếp từ `0x3B` đến `0x48` chứa các giá trị số nguyên có dấu 16-bit (`int16_t`):
  - `0x3B - 0x40`: Gia tốc 3 trục $A_x, A_y, A_z$ (độ nhạy cấu hình ±2g, ±4g, ±8g, ±16g).
  - `0x41 - 0x42`: Nhiệt độ nội bộ của chip silicon.
  - `0x43 - 0x48`: Vận tốc góc 3 trục $G_x, G_y, G_z$ (độ nhạy cấu hình ±250, ±500, ±1000, ±2000°/s).

![Sơ đồ chân và trục tọa độ mô-đun IMU MPU6500](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab4.2_MPU6500.png)

#### Chân kết nối MPU6500

| Chân | Kết nối hoặc chức năng |
| :--- | :--- |
| `VCC` | Cấp 3.3 V; một số bo hỗ trợ 3.3–5 V qua IC ổn áp. |
| `GND` | Nối mass chung với OBC. |
| `SCL` | I2C: `GPIO22`; SPI: SCK. |
| `SDA` | I2C: `GPIO21`; SPI: MOSI. |
| `EDA`, `ECL` | Bus I2C phụ của MPU6500. |
| `ADO` / `AD0` | Nối GND để chọn `0x68`, VCC để chọn `0x69`; dùng làm MISO ở chế độ SPI. |
| `INT` | Báo dữ liệu mới hoặc sự kiện vượt ngưỡng. |
| `NCS` | Chọn SPI, tích cực `LOW`; kéo lên `HIGH` để dùng I2C. |
| `FSYNC` | Đồng bộ khung hình; nối GND nếu không dùng. |

### 2. Cảm biến từ trường GY-271
- **Nhà sản xuất & Địa chỉ I2C:** QST / Honeywell. Địa chỉ I2C chuẩn là `0x0D` (QMC5883L / QMC5883P) hoặc `0x1E` (HMC5883L).
- **Cấu trúc Phần cứng:** Tích hợp 3 cảm biến từ điện trở dị hướng (Anisotropic Magnetoresistive - AMR) độ nhạy cao đo cường độ từ trường Trái Đất theo 3 trục trực giao $M_x, M_y, M_z$ (đơn vị Gauss hoặc $\mu\text{T}$) tương ứng với ký hiệu mũi tên trục $X, Y, Z$ trên bo mạch.

![Sơ đồ chân và mặt linh kiện mô-đun từ kế GY-271](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab4.2_GY271.png)

#### Chân kết nối GY-271

| Chân | Kết nối hoặc chức năng |
| :--- | :--- |
| `VCC` | Cấp nguồn 3.3 V; một số bo có IC ổn áp khi dùng nguồn 5 V. |
| `GND` | Nối mass chung. |
| `SCL` | Nối chung bus I2C với MPU6500 và `GPIO22`. |
| `SDA` | Nối chung bus I2C với MPU6500 và `GPIO21`. |
| `DRDY` | Báo khi dữ liệu từ trường mới sẵn sàng. |

- **Thuật toán Tính Góc Hướng La bàn (Heading):**
  Góc hướng la bàn so với cực Bắc từ được tính bằng hàm lượng giác:

$$\text{Heading} = \text{atan2}(M_y, M_x) \cdot \frac{180}{\pi}$$

  Nếu $\text{Heading} < 0^\circ$, hiệu chỉnh: $\text{Heading} = \text{Heading} + 360^\circ$.
- **Hợp nhất Dữ liệu Quán tính 9 Bậc Tự do (9-DOF):** 
  Kết hợp gia tốc kế 3 trục (đo véc-tơ trọng trường), con quay hồi chuyển 3 trục (đo vận tốc góc) từ MPU6500 và từ kế 3 trục (đo véc-tơ từ trường Trái Đất) từ GY-271 cho phép phân hệ ADCS của vệ tinh xác định đầy đủ và ổn định cả 3 góc Euler trong không gian 3 chiều: Roll $\phi$, Pitch $\theta$ và Yaw / Heading $\psi$.
- **Chuẩn hóa Định dạng Log Serial:** Dữ liệu cảm biến được in theo từng cụm 3 trục trên một dòng (`ACCEL: ...`, `GYRO: ...`, `MAG: ...`), loại bỏ hoàn toàn các tiền tố ngoặc vuông `[...]` rườm rà.

### 3. Ước lượng tư thế bằng bộ lọc bù
Để tính toán góc nghiêng (Roll $\phi$, Pitch $\theta$) của vệ tinh, mỗi cảm biến đơn lẻ đều có nhược điểm chí mạng:
- **Gia tốc kế:** Đo véc-tơ gia tốc trọng trường $g$. Cho góc nghiêng chính xác tuyệt vời khi ở trạng thái tĩnh dài hạn, nhưng lại cực kỳ nhạy cảm với các rung động cơ học ngắn hạn và lực quán tính lúc chuyển động (Nhiễu tần số cao).
- **Con quay hồi chuyển:** Đo vận tốc góc $\omega$. Tích phân góc quay $\theta = \int \omega \, dt$ cho phản hồi nhanh nhưng sai số trôi dạt điểm không tích lũy theo thời gian.
- **Bộ lọc bù:** Kết hợp lọc thông cao cho con quay và lọc thông thấp cho gia tốc kế:

$$\theta_t = \alpha \cdot (\theta_{t-1} + \omega \cdot \Delta t) + (1 - \alpha) \cdot \theta_\text{accel}$$

  Trong đó: $\Delta t$ là chu kỳ lấy mẫu, $\alpha$ là trọng số lọc (thường chọn $\alpha = 0.96 - 0.98$). Thuật toán chạy siêu nhanh trên vi điều khiển, triệt tiêu hoàn toàn độ trôi của con quay và lọc sạch nhiễu rung lắc của gia tốc kế.

![Bộ lọc bù hợp nhất dữ liệu IMU](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_complementary_filter.png)

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 4.2)
1. **Cảm biến quán tính IMU được tích hợp trên bộ kit PTITCube thuộc dòng cảm biến nào và có địa chỉ I2C mặc định là gì?**
   - A. ADXL345 địa chỉ 0x53
   - B. LSM9DS1 địa chỉ 0x1E
   - C. BNO055 địa chỉ 0x28
   - D. MPU6500 địa chỉ 0x68

2. **Bộ lọc bù tính góc nghiêng (Roll/Pitch) theo nguyên lý nào?**
   - A. Tắt hoàn toàn tín hiệu đo của gia tốc kế
   - B. Kết hợp lọc thông cao cho con quay hồi chuyển và thông thấp cho gia tốc kế
   - C. Lấy trung bình cộng số học của hai giá trị thô
   - D. Chỉ dùng tích phân con quay hồi chuyển theo thời gian

3. **Cảm biến từ trường QMC5883L (địa chỉ 0x0D) cung cấp dữ liệu nào phục vụ phân hệ xác định tư thế vệ tinh?**
   - A. Cường độ từ trường 3 trục dùng để tính góc hướng La bàn (Heading)
   - B. Vận tốc quay quanh trục Z
   - C. Độ ẩm môi trường xung quanh vỏ vệ tinh
   - D. Gia tốc trọng trường dọc theo trục thẳng đứng

---

---

## 📖 Doc 4.3: GNSS L76X và đọc dữ liệu NMEA qua UART

![Luồng thu nhận tín hiệu vệ tinh định vị GNSS L76X qua UART2](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_gnss_l76x_architecture.png)

### 1. Module định vị GNSS L76X
Module Quectel **L76X** là khí tài định vị không gian tích hợp đồng thời hai hệ thống vệ tinh: **GPS** (Hoa Kỳ - tần số L1 1575.42 MHz) và **GLONASS** (Nga - tần số L1 1602 MHz):
- **Ăng-ten Gốm Tích hợp (Ceramic Patch Antenna):** Thu sóng vô tuyến phân cực tròn thuận (RHCP) từ các chòm vệ tinh trên quỹ đạo.
- **Tầng Tiền Khuếch đại Nhiễu Thấp (LNA):** Khuếch đại tín hiệu vệ tinh cực kỳ yếu ớt (mức thu chỉ từ -130 dBm đến -165 dBm) trước khi đưa vào bộ giải mã RF.
- **Các Chế độ Khởi động Định vị:**
  - *Cold Start (~35 giây):* Khi module vừa cấp nguồn và chưa có dữ liệu lịch thiên văn (Almanac/Ephemeris), nó phải quét toàn bộ các kênh tần số để tìm vệ tinh.
  - *Hot Start (~1 giây):* Khi module đã có sẵn dữ liệu quỹ đạo và đồng hồ UTC chính xác trong bộ nhớ tạm RTC, thời gian chốt vị trí diễn ra gần như tức thời.
- **Giao tiếp Nối tiếp UART Phần cứng:**
  - Kết nối với cổng nối tiếp phần cứng `UART2` (`Serial2`) của OBC.
  - Tốc độ baud tiêu chuẩn: 9600 bps, cấu hình khung truyền `8-N-1`.
  - **Đặc tính phát dữ liệu tự động:** Ngay khi được cấp nguồn 3.3V, module tự động phát ra luồng ký tự ASCII chứa các dòng bản tin NMEA định kỳ mỗi giây (1 Hz) qua chân TX mà không cần OBC phải gửi bất kỳ lệnh kích hoạt nào.

![Hai mặt module Waveshare L76X, đầu nối 5 chân và anten gốm](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab4.3_L76X.png)



| Chân | Tín hiệu | Chức năng và cách nối trong Lab |
|---:|---|---|
| 1 | VCC | Cấp nguồn cho module. Dùng đường 3,3 V theo cấu hình Lab. |
| 2 | GND | Nối mass chung với ESP32. |
| 3 | TX | Ngõ ra UART gửi bản tin NMEA. Nối với RX2 (GPIO16). |
| 4 | RX | Ngõ vào UART nhận lệnh cấu hình. Nối với TX2 (GPIO17) nếu cần gửi lệnh. |
| 5 | PPS | Ngõ ra xung 1 PPS dùng làm mốc thời gian. Có thể để hở khi chỉ đọc NMEA. |



### 2. Đọc dữ liệu NMEA qua UART
Trong bài thực hành đọc dữ liệu thô, mục tiêu là kiểm tra thông mạch phần cứng và quan sát trực tiếp luồng ký tự thô đổ về từ không gian:
```cpp
void setup() {
    Serial.begin(115200);
    // Khởi tạo UART2 nối với GPS L76X ở tốc độ 9600 baud
    Serial2.begin(9600);
    Serial.println("[RAW] Dang doc luong byte thô NMEA truc tiep tu GPS...");
}

void loop() {
    // Đọc từng byte thô từ GPS và in thẳng ra màn hình máy tính
    while (Serial2.available() > 0) {
        char c = Serial2.read();
        Serial.print(c);
    }
}
```

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 4.3)
1. **Module định vị vệ tinh GNSS L76X kết nối với vi điều khiển OBC thông qua cổng giao tiếp và tốc độ baud mặc định nào?**
   - A. Bus SPI ở tần số 10 MHz
   - B. Bus I2C ở tốc độ 400 kHz
   - C. Cổng UART2 ở tốc độ 9600 baud
   - D. Giao tiếp 1-Wire ở tốc độ 115200 baud

2. **Bản tin NMEA nào sau đây của module GPS chứa thông tin kinh độ, vĩ độ và số lượng vệ tinh đang khóa?**
   - A. `$GPVTG`
   - B. `$GPGGA`
   - C. `$GPGSV`
   - D. `$GPGLL`

---

---

## 📖 Doc 4.4: Giải mã dữ liệu NMEA 0183

### 1. Vì sao cần giải mã dữ liệu NMEA?
Luồng dữ liệu thô thu được từ cổng UART là các dòng chuỗi văn bản ASCII thuần túy. Vi điều khiển OBC không thể sử dụng trực tiếp chuỗi ký tự này để tính toán quỹ đạo, kiểm tra hàng rào địa lý (Geofencing) hay đóng gói vào bản tin Telemetry nếu không chuyển đổi thành các kiểu dữ liệu số thực (`float`, `double`) và số nguyên (`int`).

![Quy trình kiểm tra Checksum XOR và giải mã bản tin NMEA 0183](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_nmea_parsing_flow.png)

### 2. Cấu trúc bản tin NMEA
Mỗi câu tin NMEA bắt đầu bằng ký tự `$`, theo sau là mã định danh nguồn phát (`GP` cho GPS, `GL` cho GLONASS, `GN` cho đa hệ thống), tên câu lệnh 3 ký tự, các trường dữ liệu ngăn cách bằng dấu phẩy `,` và kết thúc bằng mã Checksum sau dấu `*`:

#### A. Câu tin `$GPRMC`
Đây là câu tin cốt lõi cung cấp dữ liệu chuyển động và tọa độ vị trí tối thiểu:
```
$GPRMC,083559.00,A,2102.1234,N,10547.5678,E,0.05,84.4,121126,,,A*76
```
- **Trường 1 (`083559.00`):** Thời gian chuẩn quốc tế UTC theo định dạng `hhmmss.ss` (08 giờ 35 phút 59.00 giây).
- **Trường 2 (`A`):** Trạng thái định vị (`Status`):
  - `A` = Active: Tín hiệu vệ tinh hợp lệ, tọa độ tin cậy.
  - `V` = Void: Chưa khóa được vệ tinh, tọa độ không hợp lệ.
- **Trường 3 & 4 (`2102.1234,N`):** Vĩ độ định dạng Độ - Phút thập phân `ddmm.mmmm` và bán cầu (`N` = Bắc, `S` = Nam).
- **Trường 5 & 6 (`10547.5678,E`):** Kinh độ định dạng Độ - Phút thập phân `dddmm.mmmm` và bán cầu (`E` = Đông, `W` = Tây).
- **Trường 7 (`0.05`):** Vận tốc di chuyển mặt đất tính bằng Hải lý trên giờ (Knots).
- **Trường 8 (`84.4`):** Góc hướng di chuyển (Course over ground) tính bằng độ (0° - 360°).
- **Trường 9 (`121126`):** Ngày tháng năm UTC định dạng `ddmmyy` (Ngày 12 tháng 11 năm 2026).

#### B. Chuyển tọa độ sang độ thập phân
Định dạng NMEA biểu diễn theo Độ và Phút (`ddmm.mmmm`). Để đưa lên bản đồ số (Google Maps, Leaflet, Dashboard), bắt buộc phải đổi sang Độ Thập Phân:

$$\text{Decimal Degrees} = \text{Degrees} + \frac{\text{Minutes}}{60.0}$$

*Ví dụ cụ thể:*
- Vĩ độ thô: `2102.1234, N` ⇒ Số độ nguyên là 21°, số phút là 02.1234'.

$$\text{Latitude} = 21 + \frac{2.1234}{60.0} = \mathbf{21.03539^\circ\text{ N}}$$

- Kinh độ thô: `10547.5678, E` ⇒ Số độ nguyên là 105°, số phút là 47.5678'.

$$\text{Longitude} = 105 + \frac{47.5678}{60.0} = \mathbf{105.79279^\circ\text{ E}}$$

*(Quy ước: Nếu là bán cầu Nam `S` hoặc bán cầu Tây `W`, giá trị tọa độ mang dấu âm)*.

#### C. Câu tin `$GPGGA`
Cung cấp thông tin độ cao và chất lượng hình học của tín hiệu vệ tinh:
```
$GPGGA,083559.00,2102.1234,N,10547.5678,E,1,08,1.05,45.2,M,-28.9,M,,*5A
```
- **Trường 6 (`1`):** Chỉ số phẩm chất Fix (`Fix Quality`):
  - `0`: Chưa chốt được vị trí (Invalid).
  - `1`: Định vị GPS tiêu chuẩn (SPS Fix).
  - `2`: Định vị vi sai độ chính xác cao (DGPS Fix).
- **Trường 7 (`08`):** Số lượng vệ tinh đang bám khóa thành công (khuyến nghị $\ge 4$ vệ tinh để tính toán tọa độ không gian 3 chiều).
- **Trường 8 (`1.05`):** Hệ số suy giảm độ chính xác hình học mặt phẳng (**HDOP - Horizontal Dilution of Precision**):
  - $\text{HDOP} < 1.0$: Chất lượng vị trí lý tưởng.
  - $1.0 \le \text{HDOP} \le 2.0$: Rất tốt (đủ tiêu chuẩn điều hướng).
  - $2.0 < \text{HDOP} \le 5.0$: Mức chấp nhận được.
  - $\text{HDOP} > 5.0$: Vệ tinh phân bố góc hẹp, sai số vị trí lớn, phần mềm bay nên tạm thời loại bỏ mẫu đo này.
- **Trường 9 (`45.2,M`):** Cao độ so với mực nước biển trung bình (MSL Altitude) tính bằng mét.

### 3. Kiểm tra Checksum XOR
Để bảo đảm đường truyền UART không bị suy hao biến dạng bit làm sai lệch tọa độ:
- Hai ký tự Hex sau dấu `*` ở cuối câu tin chính là mã Checksum 8-bit.
- **Giải thuật:** Duyệt qua tất cả các byte nằm giữa ký tự `$` và ký tự `*`, thực hiện phép toán logic `XOR (^)` liên tục:

$$\text{Checksum} = \bigoplus_{i=1}^{N} \text{byte}[i]$$

- Nếu mã tính toán khớp với 2 ký tự Hex cuối dòng, câu tin được xác nhận hợp lệ.

### 4. Giải mã luồng dữ liệu bằng PTIT_GPS
Thay vì dùng biến chuỗi `String` gây phân mảnh bộ nhớ Heap của ESP32, bộ giải mã chuyên dụng sử dụng máy trạng thái hữu hạn xử lý từng ký tự đến trong thời gian thực:
```cpp
#include <PTITCube.h>

PTIT_GPS myGPS;

void setup() {
    Serial.begin(115200);
    myGPS.init(); // Khởi tạo UART2 và cấu hình parser
}

void loop() {
    // Đẩy từng byte từ UART vào máy trạng thái giải mã NMEA
    myGPS.update();

    static unsigned long lastPrint = 0;
    if (millis() - lastPrint > 2000) {
        lastPrint = millis();
        if (myGPS.getSatellites() > 0 && myGPS.getLat() != 0.0) {
            Serial.printf("Toa do: %.6f, %.6f | Cao do: %.1f m | Ve tinh: %d | HDOP: %.2f\n",
                          myGPS.getLat(), myGPS.getLng(), 
                          myGPS.getAltitude(), myGPS.getSatellites(), myGPS.getHDOP());
        } else {
            Serial.println("GPS: Dang tim tin hieu ve tinh ngoai troi...");
        }
    }
}
```

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 4.4)
1. **Công thức toán học chuyển đổi tọa độ từ định dạng NMEA độ-phút (`ddmm.mmmm`) sang số thập phân chuẩn WGS-84 (`dd.dddddd`) là:**
   - A. $\text{Decimal} = \text{Degrees} \times 60 + \text{Minutes}$
   - B. $\text{Decimal} = \frac{\text{Degrees} + \text{Minutes}}{100.0}$
   - C. $\text{Decimal} = \text{Degrees} \times \text{Minutes}$
   - D. $\text{Decimal} = \text{Degrees} + \frac{\text{Minutes}}{60.0}$

2. **Mã kiểm tra toàn vẹn Checksum trong bản tin NMEA 0183 được tính toán bằng phép toán logic nào trên các ký tự giữa `$` và `*`?**
   - A. Phép cộng dồn modulo 256
   - B. Phép toán XOR từng byte ký tự ASCII
   - C. Mã đa thức CRC-32 tiêu chuẩn mạng
   - D. Phép nhân băm chuỗi ký tự MD5

---

---

## 📖 Doc 4.5: Camera OV2640 và truyền ảnh qua UART

![Quy trình chụp ảnh JPEG từ OV2640 lưu MicroSD và truyền LoRa](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_ov2640_capture_pipeline.png)

### 1. Camera OV2640
- **Đặc tính Tải trọng:** Cảm biến ảnh màu CMOS Omnivision **OV2640** độ phân giải 2 Megapixel (1600 × 1200 pixel), tích hợp bộ mã hóa nén phần cứng JPEG giúp giảm dung lượng tệp ảnh từ hàng Megabytes xuống còn $30 - 60\text{ KB}$, rất phù hợp với băng thông hạn chế của kênh truyền sóng không gian.
- **Yêu cầu Xung nhịp Ngoài XCLK:** Để mạch xử lý tín hiệu hình ảnh (ISP) bên trong OV2640 hoạt động, nó đòi hỏi vi điều khiển OBC phải phát ra một đường xung nhịp tần số cao ổn định từ 10 MHz đến 20 MHz tại chân `XCLK`.

### 2. Gửi lệnh chụp và nhận ảnh qua UART
Khác với giao thức ký tự đơn sơ khởi (`'p'`), giao thức chuẩn của hệ thống CubeSat sử dụng chuỗi ký tự rõ ràng để tăng tính an toàn và khả năng tự kiểm lỗi:
1. **Gửi lệnh chụp:** OBC gửi lệnh chuỗi text `"capture\n"` đến ESP32-CAM qua cổng nối tiếp UART2 (tốc độ baud 115200, chân RX 16, TX 17).
2. **Phản hồi kích thước:** ESP32-CAM chụp ảnh, nén phần cứng JPEG và phản hồi chuỗi `SIZE:<bytes>\n` thông báo kích thước tệp ảnh (ví dụ: `SIZE:38420`).
3. **Bắt tay đồng bộ:** OBC kiểm tra tính sẵn sàng của thẻ nhớ MicroSD (khởi tạo qua thư viện `PTIT_Storage`), mở tệp tin `/photos/uart_img_X.jpg` để ghi, sau đó gửi lại chuỗi `"OK\n"` báo hiệu sẵn sàng nhận luồng dữ liệu.
4. **Truyền nhận dữ liệu nhị phân:** ESP32-CAM truyền luồng byte nhị phân của ảnh JPEG (bắt đầu bằng byte `0xFFD8` và kết thúc bằng `0xFFD9`). OBC đọc theo khối buffer 256 byte và ghi trực tiếp vào thẻ nhớ.
5. **Hoàn tất phiên chụp:** Sau khi nhận đủ số byte đã cam kết, OBC gửi chuỗi `"DONE\n"` để ESP32-CAM giải phóng bộ nhớ đệm, sau đó đóng tệp thẻ nhớ và tự động trả quyền ưu tiên bus UART2 cho module định vị GPS thông qua IC chuyển mạch 74HC4066.

---

---

## 📖 Doc 4.6: Chuyển mạch UART bằng 74HC4066

![Sơ đồ đa hóa cổng UART2 qua IC 74HC4066](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab4_uart_mux_74hc4066.png)

### 1. IC chuyển mạch 74HC4066
- **Xung đột Phần cứng khi Dùng chung Cổng UART:**
  - Cả module Camera OV2640 và module GNSS L76X đều có nhu cầu giao tiếp nối tiếp tốc độ cao với vi điều khiển OBC, nhưng số lượng chân UART phần cứng trên vi điều khiển bị giới hạn do phải ưu tiên chân cho kết nối máy tính và module thu phát sóng LoRa.
- **Giải pháp Kỹ thuật bằng IC Chuyển mạch 74HC4066:**
  - Kit tích hợp vi mạch chuyển mạch tương tự 4 kênh **74HC4066**, dùng để chọn đường tín hiệu:
    - *Chế độ Định vị (GNSS Mode):* OBC điều khiển chân chọn đóng khóa kênh GPS vào đường UART của OBC. Bộ thu GNSS thu thập chuỗi bản tin NMEA và chốt tọa độ vị trí.
    - *Chế độ Chụp ảnh (Camera Mode):* Sau khi đã chốt xong tọa độ, OBC ngắt kết nối UART của GPS và đóng khóa kênh Camera vào đường UART để nhận luồng byte ảnh JPEG.

---

---

## 📖 Doc 4.7: Thu thập và xử lý dữ liệu cảm biến

---

## 📖 Doc 4.8: Hướng dẫn kết nối cảm biến và tải trọng

Tài liệu này tổng hợp các kết nối cảm biến, module GNSS và camera dùng trong Lab 4. Dùng các vị trí chờ hình để bổ sung sơ đồ lắp thực tế.

### 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí ESP32, các cảm biến, module GNSS, camera và IC 74HC4066.

### 2. Kết nối cảm biến I2C

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

### 3. Kết nối GNSS và camera

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

### 4. Kiểm tra trước khi thực hành

- Đối chiếu điện áp cấp với yêu cầu của từng mô-đun.
- Kiểm tra địa chỉ I2C và đường UART trước khi chạy chương trình.

---

## 💻 Hướng dẫn thực hành Lab 4
- `Lab4-01a_Environment_I2C`: Khởi tạo bus I2C, đọc trực tiếp 24 byte hằng số hiệu chuẩn và các thanh ghi áp suất thô của BMP280.
- `Lab4-01b_Environment_Library`: Đọc và tính toán áp suất khí quyển, nhiệt độ môi trường và độ cao ước lượng thông qua thư viện PTITCube.
- `Lab4-02a_IMU_I2C`: Đọc dữ liệu gia tốc (ACCEL), vận tốc góc (GYRO) từ MPU6500 và từ trường (MAG), góc la bàn (Heading) từ GY-271 qua I2C; xuất log Serial gọn gàng 3 trục mỗi dòng, không chứa tiền tố ngoặc vuông `[...]`.
- `Lab4-02b_IMU_Library`: Kết hợp dữ liệu 9 trục bằng bộ lọc bù để tính góc Roll, Pitch và hướng la bàn.
- `Lab4-03a_GPS_UART`: Đọc trực tiếp luồng ký tự ASCII bản tin NMEA thô từ module GNSS L76X qua cổng UART2 thuần.
- `Lab4-03b_GPS_Library`: Thu nhận, phân loại câu tin NMEA thô và kiểm tra mã Checksum XOR (*hh) bằng bộ đệm dòng.
- `Lab4-03c_GPS_PTITCube`: Sử dụng thư viện PTITCube chuyển đổi luồng dữ liệu NMEA thô thành các trường số thực có ý nghĩa mà người dùng có thể hiểu được (Kinh độ, Vĩ độ, Cao độ m, Vận tốc km/h, Hướng đi, Số vệ tinh, HDOP, Thời gian UTC).
- `Lab4-04a_Camera_UART`: Giao tiếp UART thuần bắt tay bằng chuỗi văn bản (`"capture"` -> `SIZE` -> `OK` -> stream -> `DONE`) với module ESP32-CAM và ghi trực tiếp luồng byte ảnh JPEG vào thẻ nhớ MicroSD qua thư viện `PTIT_Storage`.
- `Lab4-04b_Camera_Library`: Sử dụng thư viện `PTITCube` (`PTIT_Camera`) để chụp ảnh và tự động quản lý lưu trữ vào thẻ nhớ MicroSD qua lệnh chuỗi văn bản (`"capture"`, `"photo"`).
- `Lab4-05_Read_All_Sensors`: Tổng hợp đọc đồng thời tất cả các cảm biến môi trường (BMP280), quán tính (MPU6500, QMC5883) và định vị (GPS L76X) trong một chu trình khép kín.
- `Lab4-06_Camera_Multiplexer`: Lập trình điều khiển IC chuyển mạch 74HC4066 luân phiên giữa Camera và GNSS theo nguyên tắc Break-Before-Make, giải quyết triệt để xung đột cổng UART.

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 4)
1. **Cặp byte đánh dấu bắt đầu (SOI) và kết thúc (EOI) của một tệp ảnh nén JPEG nhận được từ Camera OV2640 là:**
   - A. `0xFFD8` và `0xFFD9`
   - B. `0x0000` và `0xFFFF`
   - C. `0x1234` và `0x5678`
   - D. `0xAA55` và `0x55AA`

2. **Trong bài thực hành Camera UART (Lab4-04a), điều kiện tiên quyết về phần cứng để lưu trữ ảnh chụp là gì?**
   - A. Phải có kết nối Internet WiFi tốc độ cao
   - B. Phải tắt toàn bộ các cảm biến I2C còn lại
   - C. Phải cấp nguồn 12V riêng cho module Camera
   - D. Bắt buộc phải có thẻ nhớ MicroSD cắm trên bo mạch
