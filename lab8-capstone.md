# Lab 8: Thiết kế hệ thống, bảng giám sát và đồ án cuối khóa

---

## 📖 Doc 8.1: Kỹ thuật hệ thống và vòng đời dự án

### 1. Kỹ thuật hệ thống không gian
Kỹ thuật Hệ thống là phương pháp luận kỹ thuật và quản trị đa ngành được NASA, ESA và các cơ quan hàng không vũ trụ hàng đầu thế giới sử dụng để hiện thực hóa các sứ mệnh không gian phức tạp:
- Khác với kỹ thuật truyền thống tập trung vào từng phân hệ riêng lẻ, Kỹ thuật Hệ thống nhìn nhận vệ tinh như một thực thể thống nhất.
- Đảm bảo sự cân bằng tối ưu giữa các ràng buộc cạnh tranh khốc liệt: Khối lượng, Kích thước, Điện năng tiêu thụ, Chi phí và Thời gian triển khai.

### 2. Các giai đoạn của dự án không gian
Chu trình phát triển sứ mệnh vệ tinh gồm nhiều giai đoạn và cột mốc đánh giá kỹ thuật:

![Vòng đời dự án vũ trụ chuẩn NASA/ESA](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab8_project_lifecycle.png)

- **Pre-Phase A:** Xác định mục tiêu khoa học, đánh giá tính khả thi và đề xuất kiến trúc.
- **Phase A:** Hoàn thiện yêu cầu hệ thống; kết thúc bằng các mốc **MDR / SRR**.
- **Phase B:** Phân rã yêu cầu, xác định giao diện (ICD) và ngân sách; kết thúc bằng **PDR**.
- **Phase C:** Hoàn thiện bản vẽ CAD, sơ đồ mạch, phần mềm và mẫu kỹ thuật; kết thúc bằng **CDR**.
- **Phase D:** Lắp ráp mẫu bay, thử nghiệm môi trường, nghiệm thu **AR** và đánh giá sẵn sàng phóng **FRR**.
- **Phase E:** Vận hành vệ tinh, thu thập dữ liệu và xử lý sự cố.
- **Phase F:** Kết thúc sứ mệnh, hạ quỹ đạo và xử lý vệ tinh theo quy định về rác thải không gian.

### 3. Ma trận kiểm định MECE
Nguyên tắc: "Không trùng lặp, Không bỏ sót" - Mọi thông số kỹ thuật của vệ tinh đều phải được phân bổ phương pháp kiểm chứng tương ứng trong 4 phương pháp chuẩn mực:
1. **Thử nghiệm:** Đo phản hồi của hệ thống trong môi trường như buồng rung hoặc buồng TVAC.
2. **Phân tích:** Dùng mô hình và phần mềm để tính ngân sách tuyến, truyền nhiệt hoặc ứng suất.
3. **Kiểm tra:** Quan sát và đo kích thước, độ sạch, chất lượng lắp ráp.
4. **Trình diễn:** Xác nhận chức năng hoạt động như yêu cầu, chẳng hạn cơ cấu bung ăng-ten hoặc đèn LED.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 8.1)
1. **Cột mốc PDR trong kỹ thuật hệ thống có mục tiêu gì?**
   - A. Bàn giao vệ tinh ra bãi phóng tên lửa
   - B. Thay thế toàn bộ linh kiện phần cứng sau khi thử nghiệm
   - C. Đánh giá tính khả thi kiến trúc sơ bộ và xác nhận thiết kế đáp ứng yêu cầu sứ mệnh
   - D. Công bố dữ liệu khoa học sau khi kết thúc sứ mệnh

2. **Ma trận truy xuất yêu cầu (RTM) dùng để làm gì?**
   - A. Liên kết từng yêu cầu nhiệm vụ với khối mã nguồn phần mềm và kịch bản thử nghiệm
   - B. Theo dõi vị trí quỹ đạo không gian của vệ tinh
   - C. Quản lý chi phí tài chính mua sắm linh kiện
   - D. Tự động sinh mã nguồn nhúng cho vi điều khiển

---

## 📖 Doc 8.2: Chọn và thiết kế khối hiển thị trên LMS

### 1. Chọn khối hiển thị phù hợp
Bảng điều khiển giúp kíp trực mặt đất theo dõi tình trạng vệ tinh. Chọn thành phần hiển thị phù hợp giúp phát hiện bất thường và vận hành an toàn:

| Thành phần | Dữ liệu | Mục đích |
| :--- | :--- | :--- |
| **Đồng hồ chỉ thị** | Điện áp pin, SOC, nhiệt độ | Hiển thị giá trị tức thời và ngưỡng cảnh báo. |
| **Đồ thị theo thời gian** | Điện áp, công suất, RSSI | Theo dõi xu hướng và chu kỳ nạp xả. |
| **Mô hình tư thế 3D** | Góc Roll, Pitch, Yaw từ IMU | Hiển thị tư thế vệ tinh. |
| **Bản đồ quỹ đạo mặt đất** | Vĩ độ, kinh độ, độ cao | Hiển thị đường bay và vùng phủ sóng. |
| **Cửa sổ dữ liệu thô** | Byte nhị phân, chuỗi ASCII | Kiểm tra dữ liệu, CRC-16 và lỗi giao thức. |
| **Bảng phát lệnh** | Lệnh và tham số | Phát lệnh với bước xác nhận an toàn. |
| **Thư viện ảnh** | Các mảnh dữ liệu ảnh | Ghép và hiển thị ảnh khi nhận đủ dữ liệu. |

![Kiến trúc bảng điều khiển viễn trắc thời gian thực](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab8_telemetry_dashboard.png)

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 8.2)
1. **Mô hình tư thế 3D trên LMS dùng loại dữ liệu nào để hiển thị tư thế vệ tinh?**
   - A. Nhiệt độ và áp suất môi trường
   - B. Tần số vô tuyến và mức công suất phát
   - C. Tỷ lệ phần trăm pin và dòng tiêu thụ
   - D. Ba góc Euler (Roll, Pitch, Yaw) hoặc bộ bốn quaternion (q0, q1, q2, q3)

2. **Bản đồ GPS hiển thị thông tin nào của vệ tinh hoặc trạm di động?**
   - A. Bản đồ nhiệt độ bên trong lõi vi xử lý
   - B. Đường bay trên mặt đất và vị trí kinh độ, vĩ độ theo thời gian thực
   - C. Sơ đồ các đường mạch đồng trên bo mạch OBC
   - D. Danh sách các bài thực hành đã hoàn thành

---

## 📖 Doc 8.3: Đề tài đồ án cuối khóa

### Đề tài 1: Vệ tinh giám sát và cảnh báo cháy rừng
- **Mục tiêu:** Ứng dụng CubeSat 1U tích hợp Camera OV2640 và cảm biến môi trường để quan sát bề mặt Trái Đất, phát hiện sớm các đám cháy rừng, gắn nhãn tọa độ GPS và truyền cảnh báo khẩn cấp về trạm mặt đất.
- **Kỹ thuật Cốt lõi:**
  - Tự động chụp ảnh khi vệ tinh đi vào vùng địa lý đã định.
  - Phân tích không gian màu (HSV/YCbCr) hoặc mô hình học máy siêu nhẹ TFLite Micro nhận diện vùng khói lửa.
  - Nén ảnh JPEG, chia thành các gói nhỏ và truyền qua LoRa.

### Đề tài 2: Trạm vệ tinh LEO thu thập và chuyển tiếp dữ liệu nông nghiệp
- **Mục tiêu:** Xây dựng vệ tinh lưu trữ rồi chuyển tiếp dữ liệu, thu thập vi khí hậu và độ ẩm đất từ cảm biến nông nghiệp ở vùng xa, sau đó gửi về trung tâm điều hành.
- **Kỹ thuật Cốt lõi:**
  - Beacon thăm dò giúp các cảm biến mặt đất ngủ để tiết kiệm pin.
  - Thuật toán đa truy cập ngẫu nhiên ALOHA giải quyết xung đột kênh vô tuyến.
  - Cơ sở dữ liệu nhật ký trên thẻ nhớ MicroSD với cờ trạng thái chuyển tiếp (`isForwarded`) và lệnh trút dữ liệu lớn (`CMD_DUMP_AGTECH_DATA`).

### Đề tài 3: Mạng vệ tinh tìm kiếm và cứu nạn
- **Mục tiêu:** Mô phỏng mạng lưới cứu hộ không gian Cospas-Sarsat: Vệ tinh lắng nghe tín hiệu cầu cứu khẩn cấp từ các phao cứu nạn ngoài biển khơi, giải mã tức thời tọa độ định vị nạn nhân và phát cảnh báo ưu tiên cao nhất về trạm cứu hộ mặt đất.
- **Kỹ thuật Cốt lõi:**
  - Xử lý ưu tiên ngắt trước đối với gói tin có cờ `EMERGENCY_SOS_FLAG`.
  - Cấu hình độ nhạy thu cực đại (LoRa SF12, BW 125kHz) bắt tín hiệu phao cứu sinh yếu ớt từ cự ly hàng trăm kilomet.
  - Hiển thị cảnh báo SOS, phát còi và chỉ đường cứu nạn trên bảng điều khiển web.

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 8)
1. **Khi dung lượng pin giảm dưới ngưỡng báo động (ví dụ < 15%), phần mềm cần thực hiện hành động an toàn nào?**
   - A. Chuyển sang chế độ an toàn: tắt camera và tải phụ, chỉ phát beacon cấp cứu theo chu kỳ dài
   - B. Bật toàn bộ đèn LED và camera chụp ảnh liên tục
   - C. Tự động xóa sạch toàn bộ mã nguồn chương trình trong Flash
   - D. Tăng tần số xung nhịp CPU lên mức tối đa

2. **Quy trình nghiệm thu và bảo vệ đồ án Capstone trên hệ thống CubeSat-LMS bao gồm những nội dung chính nào?**
   - A. Chỉ nộp bản vẽ 3D mà không cần chạy code thực tế
   - B. Thi công lắp ráp tên lửa đẩy mô hình tại phòng lab
   - C. Trình diễn hoạt động firmware tích hợp, truyền nhận viễn trắc lên Dashboard và báo cáo kỹ thuật
   - D. Chờ vệ tinh thật trên quỹ đạo gửi tín hiệu về máy bay
