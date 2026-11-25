# Lab 7: Tích hợp hệ thống và vận hành sứ mệnh

---

## 📖 Doc 7.1: Thách thức khi tích hợp CubeSat 1U

### 1. Tổng quan tích hợp hệ thống
Giai đoạn tích hợp kết nối các phân hệ phần cứng và phần mềm (OBC, EPS, C&DH, cảm biến, COM, khung cơ khí) thành một **hệ thống vệ tinh tự chủ và đồng bộ** để vận hành trong môi trường không gian.

Khi nhồi nhét tất cả các phân hệ vào không gian chật hẹp của khối lập phương 1U (10 × 10 × 11.35 cm), kỹ sư tích hợp phải đối mặt và giải quyết đồng thời 5 thách thức kỹ thuật lớn:

![5 thách thức tích hợp hệ thống CubeSat](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab7_integration_challenges.png)

### 2. Giải pháp cho năm thách thức
1. **Giải quyết Xung đột Bus I2C/SPI:**
   - *Vấn đề:* Bus I2C là tài nguyên dùng chung nối tiếp; nếu hai tác vụ phần mềm cùng truy cập đồng thời sẽ làm vỡ khung truyền tin và treo cứng cảm biến.
   - *Giải pháp:* Dùng khóa **Mutex (`xI2CMutex`)** trong FreeRTOS với cơ chế thừa kế ưu tiên. Tác vụ phải lấy khóa bằng `xSemaphoreTake` trước khi đọc cảm biến và nhả bằng `xSemaphoreGive` sau khi hoàn tất.
2. **Khắc phục Nhiễu Tương thích Điện từ (EMI Nội bộ):**
   - *Vấn đề:* Bộ phát sóng LoRa công suất cao (100 mW) và xung nhịp camera 20 MHz phát xạ điện từ làm giảm độ nhạy máy thu GPS.
   - *Giải pháp:* Nối mass hình sao, bọc các đường RF bằng màng đồng và dùng IC 74HC4066 để ngắt camera khi định vị GPS.
3. **Chống Sụt áp Nguồn (Brownout Mitigation):**
   - *Vấn đề:* Nếu camera nạp điện (250 mA), thẻ nhớ ghi khối (100 mA) và LoRa phát sóng (120 mA) cùng xảy ra trong 1 mili-giây, tổng dòng đỉnh vượt quá 500 mA sẽ làm sụt áp pin xuống dưới 2.7V gây sập nguồn OBC.
   - *Giải pháp:* Giãn cách thời điểm bật các tải công suất cao và dùng tụ lọc Tantalum dung lượng lớn.
4. **Quản lý Cân bằng Nhiệt Toàn diện:**
   - *Vấn đề:* Trong chân không, nhiệt lượng sinh ra từ OBC và mạch phát sóng bị giữ lại bên trong khung.
   - *Giải pháp:* Dùng bề mặt Anodize đen và cầu nhiệt kim loại để truyền nhiệt ra khung, giữ nhiệt độ pin trong khoảng 0–45°C.
5. **Tính Tất định Thời gian thực & Ngăn ngừa Deadlock:**
   - *Vấn đề:* Phần mềm xử lý đồng thời nhiều luồng tín hiệu: đọc cảm biến 10 Hz, ghi thẻ nhớ 1 Hz, phát beacon mỗi 30 giây và nhận lệnh vô tuyến. Thiết kế khóa không hợp lý có thể gây bế tắc.
   - *Giải pháp:* Thiết kế cấu trúc phân cấp khóa tài nguyên cố định và áp dụng kỹ thuật FreeRTOS Queue truyền dữ liệu bất đồng bộ.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 7.1)
1. **Một trong những thách thức kỹ thuật lớn nhất khi tích hợp toàn bộ các phân hệ vào không gian 1U là gì?**
   - A. Khó tìm kiếm cổng kết nối màn hình ngoài
   - B. Không thể gắn được nút nhấn khởi động
   - C. Can nhiễu điện từ (EMI) giữa module phát sóng LoRa và các cảm biến nhạy
   - D. Bộ nhớ vi điều khiển quá nhỏ không chứa đủ tên sinh viên

2. **Cân đối năng lượng khi tích hợp hệ thống cần bảo đảm điều gì?**
   - A. Tổng năng lượng tấm pin tạo ra trong chu kỳ bay phải lớn hơn tổng tiêu thụ của các tải
   - B. Pin phải luôn được nạp đầy 100% trong suốt thời gian bay
   - C. Vệ tinh phải tắt hoàn toàn bộ vi xử lý khi đi vào vùng tối
   - D. Chỉ cấp điện cho phân hệ truyền thông còn tắt toàn bộ cảm biến

---

## 📖 Doc 7.2: Kiến trúc phần mềm bay và máy trạng thái

### 1. Các lớp phần mềm bay
Để hệ thống phần mềm có tính module hóa cao, dễ kiểm thử và độc lập phần cứng, phần mềm bay được tổ chức theo 5 tầng phân cấp:
1. **Tầng trừu tượng phần cứng (HAL):** Cung cấp giao diện cho GPIO, ADC, PWM, UART, I2C và SPI.
2. **Tầng trình điều khiển:** Điều khiển các IC như BMP280, MPU6500, MAX17048, LoRa E32 và MicroSD.
3. **Tầng hệ điều hành:** FreeRTOS quản lý tác vụ, hàng đợi, bộ định thời và khóa Mutex.
4. **Tầng dịch vụ:** Xử lý lệnh, đóng khung dữ liệu, ghi thẻ nhớ và giám sát hệ thống.
5. **Tầng ứng dụng:** Máy trạng thái sứ mệnh (FSM) điều khiển chế độ hoạt động của vệ tinh.

### 2. Máy trạng thái sứ mệnh
Phần mềm bay được điều khiển bởi máy trạng thái hữu hạn (FSM):

![Máy trạng thái nhiệm vụ bay CubeSat FSM](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab7_mission_fsm.png)

### 3. Sáu trạng thái bay
- **`STATE_BOOT`:**
  - Khởi động hệ thống, kiểm tra ngoại vi, khôi phục tham số từ NVS.
  - Chờ **30 phút im lặng vô tuyến** theo quy định của ITU trước khi bung ăng-ten và phát sóng.
- **`STATE_DETUMBLE`:**
  - Giảm tốc độ xoay hỗn loạn sinh ra khi rời ống phóng xuống dưới 5°/s bằng thuật toán điều khiển từ tính B-dot và thanh giảm chấn trễ từ.
- **`STATE_NORMAL_OPS`:**
  - Vận hành quỹ đạo tiêu chuẩn: Định kỳ thu thập dữ liệu cảm biến (10Hz), ghi nhật ký thẻ nhớ SD (1Hz), phát gói tin Beacon (mỗi 30-60s) và lắng nghe lệnh viễn lệnh từ trạm mặt đất.
- **`STATE_PAYLOAD_EXEC`:**
  - Kích hoạt tải trọng camera chụp ảnh viễn thám Trái Đất, nén JPEG và phân mảnh tệp dữ liệu. Chỉ cho phép kích hoạt khi dung lượng pin $\text{SOC} > 60\%$ và nhiệt độ nằm trong dải an toàn.
- **`STATE_POWER_SAVE`:**
  - Khi pin yếu ($V_\text{bat} < 3.55\text{ V}$, $\text{SOC} < 25\%$), hệ thống tắt camera và GPS, giãn chu kỳ phát beacon lên 120 giây, giảm xung CPU xuống 80 MHz.
- **`STATE_SAFE_MODE`:**
  - Chế độ phòng thủ sống còn khi xảy ra sự cố nghiêm trọng (pin chạm đáy $< 3.4\text{V}$ hoặc cảm biến báo lỗi liên tục), chỉ duy trì nguồn tối thiểu cho bộ thu lệnh vô tuyến để chờ lệnh giải cứu từ mặt đất.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 7.2)
1. **Quy định an toàn quốc tế yêu cầu trạng thái khởi động (`STATE_BOOT`) của phần mềm bay phải duy trì thời gian trễ im lặng bao lâu sau khi rời ống phóng?**
   - A. Trễ chính xác 1 giây rồi phát sóng ngay
   - B. Phát sóng liên tục ngay khi công tắc RBF nhả
   - C. Chờ 24 giờ liên tục không thực hiện lệnh nào
   - D. Trễ im lặng tối thiểu 30 đến 45 phút trước khi kích hoạt thu phát RF

2. **Phần mềm bay chuyển từ `DETUMBLE` sang `MISSION` khi nào?**
   - A. Khi dung lượng pin sụt giảm dưới 20%
   - B. Khi vận tốc góc quay của vệ tinh trên các trục giảm xuống dưới ngưỡng an toàn
   - C. Khi có tín hiệu ngắt nguồn khẩn cấp từ trạm mặt đất
   - D. Khi thẻ nhớ MicroSD đã ghi đầy một nửa dung lượng

---

## 📖 Doc 7.3: Vận hành bay và quỹ đạo LEO

![Chu kỳ quỹ đạo LEO 95 phút và cửa sổ liên lạc Trạm Mặt đất](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab7_leo_orbit_pass.png)

### 1. Sổ tay vận hành bay
Sổ tay FOM là tài liệu quy chuẩn tác chiến dành cho kíp trực điều hành Trạm mặt đất:
- **Trước phiên liên lạc:** Kiểm tra hướng ăng-ten, hiệu chuẩn tần số SDR và tải phần tử quỹ đạo TLE mới nhất từ Space-Track.org để tính thời điểm bắt đầu (AOS) và kết thúc (LOS) liên lạc.
- **Quy trình liên lạc:**
  1. *Bắt sóng (AOS):* Khi vệ tinh xuất hiện ở chân trời, trạm nhận beacon đầu tiên và gửi lệnh xác thực `CMD_PING`.
  2. *Trút dữ liệu:* Gửi lệnh `CMD_DUMP_LOG`; vệ tinh truyền dữ liệu cảm biến đã lưu trong vòng quỹ đạo trước.
  3. *Giai đoạn Thực thi Nhiệm vụ:* Gửi các lệnh cấu hình hoặc yêu cầu chụp ảnh.
  4. *Mất sóng (LOS):* Khi vệ tinh khuất dưới chân trời, trạm đóng phiên và lưu nhật ký liên lạc.

### 2. Mô phỏng chu kỳ quỹ đạo 95 phút
Bộ kit PTITCube hỗ trợ chế độ mô phỏng quỹ đạo thực tế để rèn luyện kỹ năng vận hành:
- **Pha có nắng (60 phút):** Tấm pin cấp dòng sạc đến 500 mA; điện áp pin tăng từ 3.7 V lên 4.15 V. Vệ tinh có thể chụp ảnh và truyền dữ liệu.
- **Pha vùng tối (35 phút):** Dòng sạc bằng 0 mA; vệ tinh dùng pin Li-Po. Quản lý năng lượng để pin không cạn trước pha có nắng tiếp theo.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 7.3)
1. **AOS và LOS trong sổ tay vận hành bay là gì?**
   - A. Mốc thời gian bắt đầu và kết thúc cửa sổ liên lạc vô tuyến khi vệ tinh bay qua trạm
   - B. Thời điểm bật và tắt nguồn khối pin mặt trời
   - C. Tín hiệu báo động lỗi bộ nhớ RAM và ROM
   - D. Lệnh chuyển đổi chế độ ngủ và chế độ thức

2. **Một chu kỳ bay quỹ đạo thấp LEO ở độ cao ~400 km mất khoảng bao lâu và vệ tinh trải qua bao nhiêu vòng bay mỗi ngày?**
   - A. Khoảng 24 giờ mỗi chu kỳ, 1 vòng mỗi ngày
   - B. Khoảng 12 giờ mỗi chu kỳ, 2 vòng mỗi ngày
   - C. Khoảng 90 - 95 phút mỗi chu kỳ, xấp xỉ 15 - 16 vòng mỗi ngày
   - D. Khoảng 45 phút mỗi chu kỳ, 32 vòng mỗi ngày

---

## 💻 Hướng dẫn thực hành: Vòng lặp tuần tự và FreeRTOS

### 1. Mô phỏng vận hành bay bằng FreeRTOS
Đoạn mã C++ minh họa cài đặt máy trạng thái FSM kết hợp hàng đợi sự kiện:

```cpp
enum SatelliteState {
    STATE_BOOT,
    STATE_DETUMBLE,
    STATE_NORMAL_OPS,
    STATE_PAYLOAD_EXEC,
    STATE_POWER_SAVE,
    STATE_SAFE_MODE
};

SatelliteState currentState = STATE_BOOT;

void handleFSM() {
    float vbat = readBatteryVoltage();
    float soc = readBatterySOC();

    switch (currentState) {
        case STATE_BOOT:
            Serial.println("[FSM] STATE_BOOT: Khoi tao va giu im lang vo tuyen...");
            currentState = STATE_DETUMBLE;
            break;

        case STATE_DETUMBLE:
            Serial.println("[FSM] STATE_DETUMBLE: On dinh goc quay...");
            currentState = STATE_NORMAL_OPS;
            break;

        case STATE_NORMAL_OPS:
            if (vbat < 3.55f || soc < 25.0f) {
                Serial.println("[FSM] Pin yeu -> Chuyen sang STATE_POWER_SAVE!");
                currentState = STATE_POWER_SAVE;
            }
            break;

        case STATE_POWER_SAVE:
            if (vbat > 3.90f && soc > 60.0f) {
                Serial.println("[FSM] Pin da hoi phuc -> Tro ve STATE_NORMAL_OPS!");
                currentState = STATE_NORMAL_OPS;
            } else if (vbat < 3.40f) {
                Serial.println("[FSM] Pin nguy cap -> Vao STATE_SAFE_MODE phong thu!");
                currentState = STATE_SAFE_MODE;
            }
            break;

        case STATE_SAFE_MODE:
            // Tat toan bo phu tai, chi duy tri bo thu vo tuyen
            break;
    }
}
```

### 2. Bài thực hành Lab 7
- `Lab7-01_Full_Satellite_Program`: Phiên bản phần mềm bay tích hợp chạy theo vòng lặp tuần tự Non-blocking (Super-Loop) kết hợp `millis()`.
- `Lab7-02_RTOS_Satellite_System`: Phiên bản phần mềm bay đa nhiệm thời gian thực FreeRTOS phân chia hai lõi vi xử lý, tích hợp mô phỏng chu kỳ quỹ đạo 95 phút (Sunlight/Eclipse).

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 7)
1. **Trong chương trình vệ tinh đa nhiệm FreeRTOS (`Lab7-02_RTOS_Satellite_System`), ưu thế vượt trội so với vòng lặp tuần tự là:**
   - A. Giúp vệ tinh tiết kiệm được 100% dung lượng pin
   - B. Đảm bảo các tác vụ khẩn cấp (như nghe lệnh LoRa) được đáp ứng kịp thời không bị nghẽn
   - C. Không cần khai báo cấu hình phần cứng chân GPIO
   - D. Tự động sửa chữa phần cứng linh kiện bị hư hỏng

2. **Chuẩn hóa mã nguồn tích hợp trong Lab 7 yêu cầu đối tượng nào quản lý luồng in thông tin viễn trắc ra terminal?**
   - A. `Serial.println()` trực tiếp
   - B. `Wire.write()` qua bus truyền cảm biến
   - C. `printf()` qua cổng song song máy in
   - D. Đối tượng `cdh.println()` thuộc lớp PTIT_CDH của thư viện PTITCube
