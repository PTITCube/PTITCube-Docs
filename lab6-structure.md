# Lab 6: Cấu trúc cơ khí và tích hợp vật lý

---

## 📖 Doc 6.1: Vai trò khung cơ khí

### 1. Vai trò của phân hệ cấu trúc
Khung CubeSat không chỉ bảo vệ linh kiện mà còn:
- **Chịu tải khi phóng:** Chịu gia tốc tĩnh 10–15 g, rung âm học và rung ngẫu nhiên đến 14 g RMS mà không biến dạng hoặc nứt.
- **Định vị vệ tinh trong ống P-POD:** Giữ đúng kích thước bao ngoài để vệ tinh trượt ra khỏi ống mà không bị kẹt.
- **Tản nhiệt:** Dẫn nhiệt từ linh kiện công suất ra bề mặt khung để bức xạ vào không gian.

![Mô hình khung CubeSat 1U với các vách và thanh ray dẫn hướng ở góc](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/Lab6.1_structure.png)

*Mô hình khung CubeSat 1U.*

### 2. Tác động của khung cơ khí lên các phân hệ
Một thay đổi nhỏ trong thiết kế cơ khí sẽ ngay lập tức gây ra hiệu ứng domino ảnh hưởng trực tiếp tới hoạt động của tất cả các phân hệ còn lại:

![Giao diện liên kết cơ khí cấu trúc 1U](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab6_structure_interfaces.png)

1. **Tác động lên Phân hệ Năng lượng (EPS):**
   - *Tản nhiệt pin:* Pin Li-Po hoạt động an toàn trong khoảng 0–45°C. Cầu nhiệt kim loại dẫn nhiệt từ pin ra khung.
   - *Giữ pin:* Pin Li-Po dạng túi có thể giãn nở khi sạc và xả. Khung cần tạo lực nén đồng đều 10–20 kPa để hạn chế tách lớp điện cực.
   - *Công tắc hành trình:* Vị trí và hành trình công tắc ở chân ray phụ thuộc dung sai gia công khung.
2. **Tác động lên Phân hệ Xử lý Trung tâm (OBC):**
   - *Che chắn nhiễu và bức xạ:* Vách nhôm dày 1.5–2.0 mm giúp giảm nhiễu điện từ và bức xạ ion hóa tác động lên vi xử lý.
   - *Nối mass khung:* Nối các tấm nhôm với mass OBC, bảo đảm điện trở dưới 0.1 Ω để tiêu tán điện tích tĩnh.
3. **Tác động lên Phân hệ Truyền thông (COM):**
   - *Mặt phẳng tiếp địa:* Ăng-ten LoRa đơn cực λ/4 cần mặt phẳng kim loại dẫn điện để phối hợp trở kháng 50 Ω. Khung nhôm có thể đóng vai trò này; tiếp xúc điện kém làm tăng sóng phản xạ và có thể gây hỏng tầng khuếch đại công suất.
4. **Tác động lên cảm biến và tải trọng:**
   - *Độ cứng và căn chỉnh:* Thấu kính camera và IMU cần một mặt phẳng tham chiếu ổn định. Nếu khung cong hoặc xoắn do thay đổi nhiệt độ, trục quang học có thể lệch và cảm biến đo sai tư thế.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 6.1)
1. **Ngoài chịu tải khi phóng, khung vệ tinh còn có vai trò nào?**
   - A. Tạo lực đẩy phản lực giúp vệ tinh giữ quỹ đạo
   - B. Tự động thu thập năng lượng mặt trời không cần quang điện
   - C. Cung cấp đường dẫn tản nhiệt ra không gian và mặt phẳng tiếp địa chung chống nhiễu
   - D. Tăng độ nhạy thu của anten vô tuyến lên gấp đôi

2. **Trọng tâm của CubeSat 1U cần đáp ứng yêu cầu nào?**
   - A. Trọng tâm phải nằm trong bán kính 2 cm tính từ tâm hình học khối 1U
   - B. Trọng tâm phải nằm lệch hẳn về một góc đáy
   - C. Toàn bộ trọng lượng phải dồn hết về thanh ray trượt
   - D. Khối lượng vệ tinh không được vượt quá 100 gram

---

---

## 📖 Doc 6.2: Vật liệu và xử lý bề mặt

### 1. Hợp kim nhôm Al 6061-T6 và Al 7075-T6
Vật liệu sử dụng cho khung CubeSat phải đáp ứng đồng thời 4 tiêu chí khắt khe: Tỷ số độ bền trên khối lượng cao, tính dẫn nhiệt tốt, khả năng gia công cắt gọt CNC chính xác và không có từ tính:
- **Nhôm Hợp kim Al 6061-T6:**
  - Thành phần hợp kim chính: Nhôm, Magie (1.0%) và Silic (0.6%).
  - Trạng thái nhiệt luyện `-T6`: Tôi dung dịch và hóa già nhân tạo.
  - Độ bền: Giới hạn chảy 276 MPa; giới hạn kéo đứt 310 MPa.
  - Đặc tính: Dễ gia công phay CNC, hàn được, chống ăn mòn xuất sắc trong môi trường biển, là vật liệu kinh điển chiếm hơn 80% khối lượng khung CubeSat.
- **Nhôm Hợp kim Al 7075-T6:**
  - Thành phần hợp kim chính: Nhôm và Kẽm (5.6%).
  - Độ bền: Giới hạn chảy 503 MPa; giới hạn kéo đứt 572 MPa.
  - Ứng dụng: Dành riêng cho 4 thanh ray dẫn hướng góc hoặc các chi tiết chịu lực cắt và tải trọng va đập lớn nhất.

### 2. Anodize Type III và lớp phủ Alodine
Khung nhôm CubeSat được xử lý bề mặt theo hai quy trình chọn lọc, theo tiêu chuẩn quân sự Hoa Kỳ:

![Quy trình xử lý bề mặt Anodize và Alodine](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab6_surface_treatment.png)

1. **Anodize Type III (MIL-A-8625):**
   - Áp dụng trên: **4 thanh ray dẫn hướng ở 4 góc** tiếp xúc trực tiếp với lòng ống phóng P-POD.
   - Nguyên lý: Điện phân trong dung dịch axit sulfuric ở nhiệt độ thấp (0°C), tạo ra lớp màng gốm nhôm oxit Al₂O₃ dày 25 - 50 µm ăn sâu vào bề mặt kim loại.
   - Đặc tính: Độ cứng 60–70 HRC, chống xước và cách điện, hạn chế nguy cơ hàn nguội giữa thanh ray và ống phóng.
2. **Lớp phủ Alodine (MIL-DTL-5541 Class 3):**
   - Áp dụng trên: **Toàn bộ lòng trong của khung và các bề mặt lắp ghép bo mạch**.
   - Nguyên lý: Nhúng trong dung dịch muối cromat tạo màng thụ động hóa cực mỏng (< 1 µm).
   - Đặc tính cốt lõi: Khác với Anodize là chất cách điện, **lớp phủ Alodine Class 3 vẫn duy trì tính dẫn điện tiếp xúc tuyệt hảo**. Cho phép bu lông và các cọc tiếp địa nối thông điện các tấm khung về mass chung của hệ thống với điện trở tiếp xúc < 0.1 Ω.

### 3. Tiêu chuẩn thoát khí ASTM E595
Trong môi trường chân không sâu (< 10⁻⁵ Torr) và nhiệt độ cao, các vật liệu polyme hữu cơ (keo dán, băng dính, lớp phủ dây điện) bị bốc hơi giải phóng các phân tử khí (hiện tượng Outgassing). Tiêu chuẩn **ASTM E595** của NASA/ESA quy định 2 chỉ số kiểm định nghiêm ngặt:
- **TML:** Không quá 1.0% sau 24 giờ ở 125°C trong buồng chân không.
- **CVCM:** Không quá 0.10% khi ngưng tụ ở 25°C, giúp hạn chế lớp bám trên thấu kính và pin mặt trời.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 6.2)
1. **Vì sao bốn thanh ray của CubeSat cần được xử lý bằng Anodize Type III?**
   - A. Để khung có màu sắc thẩm mỹ bắt mắt
   - B. Để giảm trọng lượng của vật liệu nhôm xuống
   - C. Để tăng tính dẫn điện giữa vệ tinh và tên lửa
   - D. Hạn chế hàn nguội và ma sát kẹt trong ống phóng

2. **Vật liệu hợp kim nhôm nào sau đây thường được ưu tiên lựa chọn để gia công khung cấu trúc vệ tinh nhỏ?**
   - A. Nhôm tạp chất tái chế thông thường
   - B. Hợp kim nhôm hàng không Al 6061-T6 hoặc Al 7075
   - C. Gang đúc xám công nghiệp
   - D. Thép carbon cường độ thấp

---

---

## 📖 Doc 6.3: PC/104, khóa ren và cân bằng trọng tâm

### 1. Ngăn xếp bo mạch PC/104
Bên trong lòng khối lập phương 1U, các tầng bo mạch điện tử (OBC, EPS, Cảm biến, COM) được xếp chồng lên nhau theo chuẩn kết cấu công nghiệp **PC/104**:
- **Kích thước Bo mạch Chuẩn:** 90.17 × 95.89 mm với 4 lỗ bắt bu lông ở 4 góc đối xứng.
- **Trụ cách tầng:** Dùng trụ lục giác bằng đồng hoặc nhôm, cao 15 mm.
- **Thanh ren:** Luồn bốn thanh M3 bằng thép không gỉ từ đáy lên nóc để cố định cụm bo và phân tán rung động.

#### 1.1. Các phương thức tích hợp cơ khí và điện của các bo mạch trên CubeSat

Tích hợp cơ khí xác định vị trí và cách cố định từng bo mạch vào khung. Tích hợp điện xác định đường cấp nguồn và truyền tín hiệu giữa các bo mạch. Mục 2.1.3.4 của báo cáo *17– SV – 2026 – VT1, NV.Hưng (1).docx* trình bày ba phương thức sau.

![Minh họa ba phương thức PC/104, Backplane và Point-to-Point](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab6_tich_hop_bo_mach.png)

*Hình bổ sung. Các phương thức tích hợp bo mạch CubeSat, trích Hình 2.12 của báo cáo Word (nguồn [18] trong báo cáo).*

| Phương thức | Tích hợp cơ khí | Tích hợp điện |
|---|---|---|
| **PC/104 (ngăn xếp)** | Các bo mạch xếp chồng, cách tầng bằng trụ đỡ và cố định qua lỗ lắp. | Đầu nối xuyên tầng mang nguồn và tín hiệu, giúp giảm dây rời. Các bo phải thống nhất vị trí đầu nối và sơ đồ chân. |
| **Backplane (bo mạch nền)** | Các bo chức năng cắm vào đầu nối trên một bo nền chung và được định vị, giữ chắc bằng cơ cấu của khung. | Bo nền phân phối nguồn và đi dây tín hiệu đến các đầu nối. Có thể tháo hoặc thay từng bo thuận tiện khi giao diện tương thích. Bo nền phải được thiết kế theo số mô-đun và giao diện cần dùng. |
| **Point-to-Point (nối trực tiếp)** | Các bo hoặc thiết bị được cố định ở vị trí phù hợp trong khung. Dây và đầu nối cần được giữ chắc để tránh kéo căng. | Từng cặp thiết bị nối với nhau bằng dây và đầu nối riêng. Cách này linh hoạt với các giao diện khác nhau, nhưng dây chiếm thêm không gian và khó quản lý hơn khi số thiết bị tăng. |

**Liên hệ với bộ kit:** Có thể xếp các bo bằng trụ ren và thanh M3, đồng thời nối riêng GNSS hoặc LoRa với OBC qua dây UART. Cùng một bộ kit có thể dùng ngăn xếp để cố định cơ khí và dùng Point-to-Point cho một số đường điện. Khi báo cáo, hãy ghi rõ cả cách giữ bo và cách truyền nguồn, tín hiệu.

### 2. Bu lông, keo khóa ren và bu lông thông hơi
- **Cấp bền Bu lông Inox A2-70:** Chế tạo bằng thép không gỉ austenit AISI 304, giới hạn bền kéo tối thiểu 700 MPa.
- **Keo Khóa Ren Chuyên dụng Không gian Loctite 222 (Tím / Low Strength):** Rung động tần số cao của tên lửa dễ làm nới lỏng các vòng ren cơ khí. Tất cả các mối ghép ren bắt buộc phải được tra keo khóa kỵ khí Loctite 222. Keo tự đông cứng khi không có không khí giữa các khe ren kim loại, khóa chặt bu lông không bị xoay lỏng nhưng vẫn cho phép kỹ sư tháo mở bằng dụng cụ cầm tay khi cần sửa chữa trong phòng lab.
- **Bu lông Thông Hơi (Drilled-core Vented Screws):**
  - Khi vặn bu lông vào lỗ ren kín đáy, không khí bị giữ lại trong lỗ.
  - Khi thử nghiệm trong buồng chân không nhiệt (TVAC), khí thoát chậm qua khe ren, gây rò rỉ ảo và làm sai lệch kết quả đo.
  - Bu lông thông hơi được khoan một lỗ rỗng siêu nhỏ xuyên suốt tâm dọc trục bu lông, cho phép toàn bộ không khí ở đáy lỗ ren thoát ra ngay lập tức khi buồng hút chân không.

![Thiết kế bu-lông thông hơi chống rò rỉ ảo](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab6_vented_screw.png)

### 3. Khối lượng và trọng tâm
- **Khối lượng:** Tổng khối lượng CubeSat 1U không được vượt quá 1.330 kg (hoặc 2.000 kg). Dự phòng ít nhất 15–20% cho dây dẫn, màng bọc và keo dán phát sinh khi lắp ráp.
- **Quy chuẩn Trọng tâm CoM Khắt khe:**
  - Theo tiêu chuẩn CalPoly, trọng tâm CubeSat 1U không được lệch quá **±20 mm** so với tâm hình học trên ba trục $X, Y, Z$:

$$|X_\text{cm}| \le 20\text{ mm}, \quad |Y_\text{cm}| \le 20\text{ mm}, \quad |Z_\text{cm}| \le 20\text{ mm}$$

  - Nếu trọng tâm bị lệch quá giới hạn, khi lò xo P-POD đẩy vào mặt đáy vệ tinh, lực đẩy không đi qua trọng tâm sẽ tạo ra một mô-men ngẫu lực (Tip-off Torque) cực lớn, khiến vệ tinh bị quay lộn vòng hỗn loạn với vận tốc góc hàng trăm độ trên giây ngay khi vừa rời ống phóng, gây tê liệt hệ thống ăng-ten và pin mặt trời.

---

## 📖 Doc 6.4: Hướng dẫn lắp ráp khung bộ kit

Tài liệu này tổng hợp quy trình lắp các tầng bo mạch và khung cơ khí của bộ kit. Bổ sung hình vào các vị trí được đánh dấu.

Tham khảo Video 6.2 về quy trình lắp ráp trước khi bắt đầu.

### 1. Linh kiện lắp ráp

- Các tầng bo mạch PC/104.
- Trụ ren cách tầng cao 15 mm.
- Bốn thanh ren inox M3.
- Các tấm vách nhôm và bốn thanh ray dẫn hướng.
- Bu lông inox A2-70 và keo khóa ren Loctite 222.

> **Vị trí chèn hình:** Các linh kiện và thứ tự lắp ráp.

### 2. Lắp các tầng bo mạch

1. Xếp các tầng bo mạch theo đúng thứ tự.
2. Dùng trụ ren cách tầng để giữ khoảng cách giữa các bo.
3. Luồn bốn thanh ren M3 qua các lỗ ở góc và cố định cụm bo.

> **Vị trí chèn hình:** Thứ tự xếp bo mạch, trụ ren và thanh ren.

### 3. Lắp khung ngoài

1. Ghép các tấm vách nhôm quanh cụm bo.
2. Lắp bốn thanh ray dẫn hướng.
3. Siết đều bu lông bằng tua-vít cầm tay, tránh làm khung bị vênh.

> **Vị trí chèn hình:** Lắp vách nhôm, thanh ray và vị trí siết bu lông.

### 4. Kiểm tra sau khi lắp

- Kiểm tra độ vững chắc của khung và đầu nối dây.
- Kiểm tra độ đồng phẳng của bốn thanh ray.

---

## 💻 Báo cáo thực hành Lab 6

Chụp ảnh khung đã lắp hoàn chỉnh và nộp lên LMS.

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 6)
1. **Trong quy trình lắp ráp các tầng bo mạch chuẩn PC/104 của kit vệ tinh, nguyên tắc siết ốc cơ học nào phải được tuân thủ?**
   - A. Siết đều đối xứng theo hình chữ X và kiểm soát lực siết vừa đủ
   - B. Siết chặt tối đa từng góc một trước khi sang góc khác
   - C. Dùng keo dán sắt cố định thay cho việc vặn ốc
   - D. Không cần sử dụng vòng đệm chống rung lắc

2. **Mục đích của việc kiểm tra thông mạch và đo điện trở cách điện giữa khung nhôm và các đường nguồn (3.3V, 5V, VBAT) trước khi cấp nguồn là gì?**
   - A. Kiểm tra độ bóng bề mặt của khung kim loại
   - B. Đo nhiệt độ hoạt động của các cọc đỡ cơ học
   - C. Phát hiện sớm sự cố chạm chập ngắn mạch gây cháy linh kiện khi bật nguồn
   - D. Xác định trọng lượng tổng thể của khung máy
