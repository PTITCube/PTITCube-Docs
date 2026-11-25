# Lab 2: Doc 2.2 - Sụt áp, ngắt an toàn và điều khiển nguồn tải

![Sụt áp và điều khiển nguồn tải](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab2_brownout_power_gating.png)

### 1. Ngắt an toàn khi phóng
Vệ tinh cần có ít nhất hai cơ chế ngắt nguồn độc lập trong quá trình phóng:
1. **Chốt RBF:**
   - Một chốt cơ khí cách điện được cắm xuyên qua vỏ vệ tinh, ngắt đường nguồn chính của pin.
   - Trước khi đưa vệ tinh vào ống phóng, kỹ sư rút chốt. Dải cờ đỏ giúp xác nhận chốt đã được tháo.
2. **Công tắc hành trình:**
   - Bố trí ở đầu mút của thanh ray dẫn hướng.
   - Khi vệ tinh được trượt vào trong lòng ống phóng P-POD, cần gạt công tắc bị nén chặt xuống bởi thành ống phóng, giữ tiếp điểm ở trạng thái ngắt mạch điện.
   - Khi cửa P-POD mở và lò xo đẩy vệ tinh bay ra ngoài không gian, cần gạt bật bung ra, đóng tiếp điểm cấp điện toàn bộ hệ thống EPS.

### 2. Sụt áp và bảo vệ ESP32
Khi tải đột ngột cần dòng lớn, nội trở của pin và điện cảm đường mạch có thể làm sụt áp:
- Nếu điện áp cấp cho vi điều khiển sụt xuống dưới ngưỡng an toàn tối thiểu (thường là 2.7V), CPU sẽ thực thi sai tập lệnh, bộ nhớ Flash có thể bị ghi đè dữ liệu rác làm hỏng hệ điều hành.
- **Bộ phát hiện sụt áp (BOD):** ESP32 so sánh điện áp nguồn $V_{\text{dd}}$ với ngưỡng đặt trước và kích hoạt reset khi điện áp xuống thấp.
- **Biện pháp Khắc phục Phần cứng:** Bố trí các tụ điện hóa Tantalum dung lượng lớn ($100\mu\text{F} - 470\mu\text{F}$) có nội trở ESR cực thấp ngay sát chân nguồn của module phát sóng và camera để xả năng lượng bù đắp cho các đỉnh dòng tức thời.

### 3. Điều khiển mạch Buck và tiết kiệm điện
Cấp nguồn liên tục cho tải ngoại vi làm tăng mức tiêu thụ do dòng tĩnh và dòng rò.
- **Đầu ra mạch hạ áp:** Trên bo EPS, mạch Buck cấp nguồn 3.3 V cho tải ngoại vi.
- **Cắt nguồn tải:** Trên bo EPS, tín hiệu điều khiển mạch hạ áp có logic:
  - **Mức 1 (`HIGH`):** Ngắt nguồn vào mạch hạ áp và tắt tải.
  - **Mức 0 (`LOW`):** Bật mạch hạ áp, cấp nguồn 3.3 V cho tải.
  - Thư viện `PTIT_EPS` điều khiển nguồn tải qua `eps.setBuckState(true)` (bật) và `eps.setBuckState(false)` (tắt), không cần biết chân GPIO cụ thể.
  - **Ngưỡng bật chế độ tiết kiệm điện:**
    Hệ thống giám sát điện áp pin $V_\text{cell}$ và dung lượng $\text{SOC}$. Chế độ tiết kiệm điện bật khi:

$$V_\text{cell} < 3.70\text{ V} \quad \text{hoặc} \quad \text{SOC} < 25.0\%$$

    - *Khi pin yếu:* OBC lập tức gọi `eps.setBuckState(false)` để ngắt mạch hạ áp, sa thải toàn bộ phụ tải tiêu thụ và kéo dài chu kỳ ngủ ($5000\text{ ms}$) nhằm bảo toàn năng lượng cho máy tính sinh tồn và nạp lại pin.
    - *Khi pin khỏe ($V_\text{cell} \ge 3.70\text{ V}$ và $\text{SOC} \ge 25.0\%$):* OBC gọi `eps.setBuckState(true)` với chu kỳ $1500\text{ ms}$.
- **Tiết kiệm năng lượng theo chu kỳ bật/tắt:**
  - Giả sử tổng dòng tiêu thụ của khối tải ngoại vi khi hoạt động là 25 mA.
  - Nếu bật liên tục trong 1 chu kỳ quỹ đạo 95 phút, năng lượng tiêu thụ là:

$$E_{\text{continuous}} = 3.3\text{ V} \times 25\text{ mA} \times 1.583\text{ h} \approx 130.6\text{ mWh}$$

  - Nếu chỉ bật 1.5 giây mỗi 6.5 giây (tỉ lệ bật $\approx 23\%$):

$$E_{\text{gating}} \approx 130.6\text{ mWh} \times 0.23 \approx 30.0\text{ mWh}$$

  - Cách này giảm khoảng **77%** điện năng tiêu thụ của tải ngoại vi trong vùng tối quỹ đạo.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 2.2)
1. **Chức năng an toàn của chốt RBF hoặc công tắc hành trình trên EPS là gì?**
   - A. Tự động xả cạn năng lượng pin trước khi đưa lên quỹ đạo
   - B. Kích hoạt chế độ truyền dữ liệu công suất cực đại
   - C. Chuyển đổi qua lại giữa pin chính và pin dự phòng
   - D. Ngắt hoàn toàn nguồn điện khi vệ tinh nằm trong ống phóng

2. **Mạch hạ áp Buck trên bo EPS có chức năng gì?**
   - A. Chuyển điện áp một chiều thành xoay chiều
   - B. Ổn định điện áp 3.3V cấp cho vi xử lý với hiệu suất cao
   - C. Tăng điện áp pin từ 3.7V lên 12V để nạp nhanh
   - D. Đo đạc dòng điện rò rỉ của khung sườn cơ học

---

## 📖 Doc 2.3: Hướng dẫn lắp ráp hệ thống nguồn

Xem [hướng dẫn lắp ráp hệ thống nguồn Lab 2](./Lab2_Doc2.3.md) trước khi thực hành.

---

## 💻 Hướng dẫn thực hành Lab 2: Giám sát hệ thống nguồn

### 1. Kỹ thuật Đọc Thanh Ghi I2C Thuần trên Chip MAX17048
Chương trình C++ giao tiếp trực tiếp với các thanh ghi nội của MAX17048 thông qua thư viện `Wire.h`:

```cpp
#include <Wire.h>

#define MAX17048_ADDR 0x36
#define REG_VCELL      0x02
#define REG_SOC        0x04

float readBatteryVoltage() {
    Wire.beginTransmission(MAX17048_ADDR);
    Wire.write(REG_VCELL); // Trỏ tới thanh ghi VCELL
    if (Wire.endTransmission(false) != 0) return -1.0f; // Báo lỗi truyền tin

    Wire.requestFrom(MAX17048_ADDR, 2); // Yêu cầu 2 byte dữ liệu
    if (Wire.available() == 2) {
        uint8_t msb = Wire.read();
        uint8_t lsb = Wire.read();
        uint16_t raw = (msb << 8) | lsb;
        // Mỗi LSB tương ứng với 78.125 uV (1.25mV / 16)
        return (float)raw * 0.000078125f;
    }
    return -1.0f;
}

float readBatterySOC() {
    Wire.beginTransmission(MAX17048_ADDR);
    Wire.write(REG_SOC); // Trỏ tới thanh ghi SOC
    if (Wire.endTransmission(false) != 0) return -1.0f;

    Wire.requestFrom(MAX17048_ADDR, 2);
    if (Wire.available() == 2) {
        uint8_t msb = Wire.read();
        uint8_t lsb = Wire.read();
        // Byte cao là phần nguyên %, byte thấp là phần thập phân chia 256
        return (float)msb + ((float)lsb / 256.0f);
    }
    return -1.0f;
}
```

### 2. Danh mục Bài Thực hành Lab 2
- `Lab2-01a_Battery_I2C`: Thực hành lập trình đọc trực tiếp thanh ghi điện áp `VCELL` và dung lượng `SOC` từ IC MAX17048 qua giao tiếp I2C thuần.
- `Lab2-01b_Battery_Library`: Khởi tạo và quản lý phân hệ nguồn EPS thông qua lớp đối tượng đóng gói cấp cao của thư viện PTITCube, in báo cáo định kỳ ra Serial Monitor.
- `Lab2-02_Power_Saving`: Bật chế độ tiết kiệm điện khi điện áp pin dưới $3.70\text{ V}$ hoặc SOC dưới $25\%$. Dùng `eps.setBuckState()` của `PTIT_EPS` để ngắt tải an toàn.

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 2)
1. **Công thức chuyển đổi giá trị thô 16-bit đọc từ thanh ghi VCELL (0x02) của MAX17048 sang điện áp thực tế (Volt) là:**
   - A. $V = \text{raw} \times 78.125\ \mu\text{V} / 1000000$
   - B. $V = \text{raw} \times 1.25\text{ mV}$
   - C. $V = \text{raw} / 1024.0$
   - D. $V = \text{raw} \times 3.3 / 4096.0$

2. **Khi điện áp pin sụt giảm dưới ngưỡng an toàn (ví dụ 3.4V), phần mềm bay trên OBC cần thực hiện hành động khẩn cấp nào?**
   - A. Tăng công suất phát sóng vô tuyến để gửi cầu cứu
   - B. Ghi đè toàn bộ dữ liệu thẻ nhớ để giải phóng bộ nhớ
   - C. Ngắt tải ngoại vi và chuyển hệ thống sang chế độ Sleep tiết kiệm điện
   - D. Khởi động lại liên tục để phục hồi dung lượng pin
