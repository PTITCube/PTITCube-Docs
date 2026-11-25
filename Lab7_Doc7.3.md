# Lab 7: Doc 7.3 - Vận hành bay và quỹ đạo LEO

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
