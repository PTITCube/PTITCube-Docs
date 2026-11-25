# Lab 2: Phân hệ năng lượng và quản lý nguồn

---

## 📖 Doc 2.1: EPS, pin Li-Po và MAX17048

### 1. Vai trò của hệ thống nguồn
EPS lưu trữ và phân phối điện cho các hệ thống trên CubeSat. Nhiệm vụ chính:
- **Lưu trữ:** Cấp điện cho vệ tinh khi đi qua vùng tối của Trái Đất.
- **Sạc pin:** Điều phối dòng sạc từ các tấm pin quang điện và bảo vệ pin khỏi quá nạp, quá nhiệt.
- **Chuyển đổi điện áp:** Cấp các đường nguồn 3.3 V và 5.0 V cho tải.
- **Giám sát:** Đo điện áp, dòng điện và dung lượng còn lại (SOC), để OBC quản lý tải khi pin yếu.
- **Ngắt an toàn:** Giữ vệ tinh mất nguồn trong quá trình phóng cho đến khi được đẩy khỏi ống phóng.

![Kiến trúc hệ thống nguồn điện EPS](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab2_eps_architecture.png)

### 2. Đặc tính pin Li-Po
Kit PTITCube dùng pin **Li-Po** dạng túi:
- Điện áp danh định: 3.7 V mỗi cell.
- Điện áp sạc đầy: 4.20 V ± 0.05 V. Không sạc quá 4.25 V để tránh làm pin phồng hoặc hỏng.
- Ngưỡng ngắt xả: 3.00 V. Xả dưới 2.7 V có thể làm hỏng pin vĩnh viễn.
- Điện áp giảm nhanh ở 10% đầu và cuối dung lượng, nhưng ít đổi trong khoảng 80% giữa. Vì vậy, không thể ước lượng chính xác dung lượng chỉ từ điện áp tĩnh.

### 3. Sạc pin bằng CC/CV
Khối nạp nguồn sử dụng vi mạch điều khiển sạc tuyến tính chuyên dụng **TP4056A** kết hợp mạch bảo vệ pin **DW01A + MOSFET kép 8205A** thực hiện chu trình sạc hai giai đoạn hoàn hảo:
1. **Dòng không đổi (CC):**
   - Khi pin có điện áp từ 3.0V đến dưới 4.2V, mạch bơm một dòng điện sạc không đổi tối đa $I_\text{chg}$ vào pin.
   - Dòng sạc được thiết lập thông qua điện trở gắn ở chân `PROG` ($R_\text{prog}$) theo công thức của nhà sản xuất:

$$I_\text{chg} = \frac{1200\text{ V}}{R_\text{prog}}$$

     Ví dụ: Với điện trở $R_\text{prog} = 1.2\text{ k}\Omega$, dòng nạp tối đa là $I_\text{chg} = 1000\text{ mA} = 1.0\text{ A}$.
2. **Áp không đổi (CV):**
   - Khi điện áp pin đạt chính xác ngưỡng 4.20V, mạch tự động chuyển sang chế độ áp không đổi, giữ điện áp ở mức 4.20V và để dòng điện giảm dần theo hàm mũ.
   - Khi dòng điện giảm xuống dưới $1/10$ dòng sạc định mức ($C/10 \approx 100\text{ mA}$), TP4056 ngắt sạc; đèn sạc tắt và đèn báo đầy bật sáng.

![Sơ đồ mô-đun sạc và bảo vệ pin Li-Po TP4056A](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab2.1_Tp4056.png)

| Chân / thành phần | Kết nối hoặc chức năng |
| :--- | :--- |
| `IN+` / `IN-` | Nguồn sạc DC 4.5–8.0 V, tối đa 1.2 A. |
| `B+` / `B-` | Cực pin Li-Po 3.7 V; sạc đầy 4.2 V, dòng sạc tối đa 1 A. |
| `OUT+` / `OUT-` | Cấp nguồn cho tải, tối đa 3 A; tự ngắt khi pin cạn hoặc ngắn mạch. |
| TP4056A | Điều khiển sạc CC/CV và hai đèn trạng thái. |
| DW01A, 8205A | Bảo vệ pin khỏi quá nạp, quá xả, quá dòng và ngắn mạch. |

### 4. Mạch Buck và Boost
Để cấp nguồn cho các phân hệ, bo mạch EPS tích hợp hai mạch ổn áp chuyển mạch (Switching Regulators) với hiệu suất vượt trội so với ổn áp tuyến tính LDO:
- **Mạch hạ áp:** Chuyển $V_\text{bat}$ xuống 3.3 V, cấp tối đa 2 A cho OBC, bộ nhớ flash và cảm biến; hiệu suất khoảng 92%.
- **Mạch nâng áp:** Duy trì 5.0 V khi pin còn 3.4 V, cấp cho tầng khuếch đại LoRa và camera.

### 5. Giám sát pin bằng MAX17048
Trên bo EPS, chip **MAX17048** của hãng Maxim Integrated đo đạc liên tục dung lượng pin với độ chính xác cao:
- **Địa chỉ I2C Cố định:** `0x36` (dạng 7-bit).
- **ModelGauge:** Ước lượng dung lượng pin (SOC) theo nhiệt độ và tốc độ xả mà không cần điện trở đo dòng mắc nối tiếp.
- **Các Thanh ghi Trọng yếu:**
  - Thanh ghi Điện áp Cell (`VCELL` - Địa chỉ `0x02` và `0x03`): Giá trị 16-bit. Độ phân giải là $78.125\text{ }\mu\text{V}$ trên mỗi bit (tương đương $1.25\text{ mV} / 16\text{ LSB}$).

$$V_\text{cell}\text{ (V)} = \text{RegValue} \times 0.000078125\text{ V}$$

  - Thanh ghi Tỷ lệ Dung lượng (`SOC` - Địa chỉ `0x04` và `0x05`): Giá trị 16-bit. Byte cao thể hiện phần nguyên phần trăm (0 - 100%), byte thấp thể hiện phần thập phân với độ phân giải $1/256\% \approx 0.0039\%$.

![Sơ đồ chân và cổng kết nối bo mạch giám sát pin MAX17048](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab2.1_MAX17048.png)

| Chân | Kết nối hoặc chức năng |
| :--- | :--- |
| `VIN` | Cấp nguồn 3–5 V; mức logic tương thích ESP32 3.3 V. |
| `GND` | Nối chung mass với pin và OBC. |
| `SCL`, `SDA` | Nối lần lượt `GPIO22` và `GPIO21` của ESP32. |
| `INT` | Ngõ ra tích cực `LOW`, báo khi điện áp hoặc SOC xuống dưới ngưỡng. |
| `QStart` | Khởi động lại quá trình ước lượng dung lượng. |
| Hai cổng JST-PH | Nối pin, tải hoặc bộ sạc; hai cổng đấu song song. |
| Hai cổng STEMMA QT | Mở rộng bus I2C; có đèn báo nguồn. |

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 2.1)
1. **IC sạc chuyên dụng TP4056 thực hiện thuật toán nạp cho cell pin Li-Po 3.7V theo quy trình nào?**
   - A. Nạp xung gián đoạn tần số cao liên tục
   - B. Nạp xả luân phiên để chống chai điện cực
   - C. Nạp hai giai đoạn: Dòng không đổi (CC) rồi Áp không đổi (CV)
   - D. Tăng áp tuyến tính không giới hạn dòng nạp

2. **IC giám sát pin MAX17048 giao tiếp với vi điều khiển OBC qua giao thức nào và tại địa chỉ I2C mặc định nào?**
   - A. Giao thức I2C tại địa chỉ 0x36
   - B. Giao tiếp SPI tại địa chỉ 0x68
   - C. Cổng UART2 ở tốc độ 9600 baud
   - D. Giao tiếp 1-Wire tại địa chỉ 0x1A

---

---

## 📖 Doc 2.2: Sụt áp, ngắt an toàn và điều khiển nguồn tải

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

Tài liệu này tổng hợp các kết nối của mô-đun sạc TP4056A và mạch đo pin MAX17048. Đối chiếu nhãn chân trên bo mạch trước khi đấu nối.

### 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí bo EPS, pin, mô-đun sạc và MAX17048.

### 2. Kết nối TP4056A

| Chân | Kết nối |
| :--- | :--- |
| `IN+` / `IN-` | Nguồn sạc DC 4.5–8.0 V |
| `B+` / `B-` | Cực dương / cực âm của pin Li-Po |
| `OUT+` / `OUT-` | Đầu ra cấp tải |

> **Vị trí chèn hình:** Đấu nối nguồn sạc, pin và tải vào TP4056A.

Không sạc pin vượt 4.25 V. Ngừng xả khi pin còn 3.0 V; xả dưới 2.7 V có thể làm hỏng pin.

### 3. Kết nối MAX17048 với ESP32

| MAX17048 | ESP32 |
| :--- | :--- |
| `VIN` | 3.3 V |
| `GND` | `GND` |
| `SCL` | `GPIO22` |
| `SDA` | `GPIO21` |

> **Vị trí chèn hình:** Kết nối MAX17048 với ESP32 và bo EPS.

### 4. Kiểm tra trước khi thực hành

- Đối chiếu cực tính pin và các đầu nối với sơ đồ lắp ráp.
- Giữ nguyên địa chỉ I2C của MAX17048 là `0x36`.

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
