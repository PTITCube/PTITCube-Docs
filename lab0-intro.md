# Lab 0: Giới thiệu Tổng quan Kỹ thuật Không gian & Vệ tinh CubeSat

---

## 📖 Doc 0.1: Lịch sử CubeSat, Tiêu chuẩn Kỹ thuật & Môi trường Quỹ đạo LEO

### 1. Lịch sử Phát triển và Triết lý Tiêu chuẩn Hóa CubeSat
Vệ tinh siêu nhỏ tiêu chuẩn **CubeSat** được khởi xướng vào năm 1999 thông qua sự hợp tác lịch sử giữa Giáo sư **Jordi Puig-Suari** (Đại học Bách khoa Bang California - CalPoly, San Luis Obispo) và Giáo sư **Bob Twiggs** (Đại học Stanford). Mục tiêu nguyên bản của dự án là thiết lập một tiêu chuẩn kích thước và giao diện cơ khí đồng nhất, giúp sinh viên đại học có thể tự tay thiết kế, chế tạo, thử nghiệm và vận hành một vệ tinh nhân tạo thực thụ trong khoảng thời gian phù hợp với chu kỳ đào tạo (1 - 2 năm) với ngân sách chỉ bằng một phần nghìn so với các vệ tinh truyền thống.

Trước khi CubeSat ra đời, ngành công nghiệp vũ trụ bị chi phối bởi các vệ tinh lớn có khối lượng từ vài trăm kilogam đến hàng tấn, chi phí phát triển lên tới hàng trăm triệu USD và thời gian chuẩn bị kéo dài hàng thập kỷ. Bất kỳ một sai sót nhỏ nào cũng dẫn đến thiệt hại khổng lồ, khiến các kỹ sư có xu hướng cực kỳ bảo thủ, chỉ sử dụng các linh kiện điện tử chuyên dụng chống bức xạ (**Radiation-Hardened**) thế hệ cũ và đắt đỏ.

CubeSat đã tạo nên một cuộc cách mạng mang tính đột phá (**New Space Revolution**) dựa trên ba trụ cột triết lý cốt lõi:
1. **Tiêu chuẩn Hóa Cơ khí & Giao diện Phóng (Standardized Form Factor & Deployment):** Quy định kích thước khối lập phương cố định giúp các nhà cung cấp tên lửa đẩy dễ dàng tích hợp vệ tinh vào các khoang chứa tiêu chuẩn mà không cần thiết kế lại cơ cấu gá phóng cho từng sứ mệnh.
2. **Khai thác Linh kiện Thương mại Sẵn có (COTS - Commercial Off-The-Shelf):** Tận dụng bước nhảy vọt của ngành công nghiệp bán dẫn tiêu dùng (vi xử lý kiến trúc **ARM** / **ESP32**, cảm biến vi cơ điện tử **MEMS** - *Micro-Electro-Mechanical Systems*, pin Lithium-Ion điện thoại). Mặc dù linh kiện COTS không được bọc chắn bức xạ chuyên dụng, các kỹ sư vũ trụ hiện đại áp dụng các giải pháp kiến trúc hệ thống, thuật toán tự sửa lỗi và cơ chế dự phòng đa tầng để đạt độ tin cậy bay mong muốn với giá thành cực thấp.
3. **Đóng gói Kín Giảm Thiểu Rủi ro cho Tên lửa Mẹ (Containerized Launch Integration):** Vệ tinh được nhốt hoàn toàn bên trong ống phóng dạng hộp kín có nắp đậy (như **P-POD** - *Poly-Picosatellite Orbital Deployer*). Mọi nguy cơ bung vỡ chi tiết, đoản mạch hay phát sóng ngoài ý muốn đều bị cô lập hoàn toàn, bảo vệ an toàn tuyệt đối cho tên lửa đẩy và tải trọng thương mại chính.

![Lịch sử phát triển chuẩn CubeSat & New Space](../PTITCube-Figs/png/lab0_timeline.png)

---

### 2. Phân loại Vệ tinh Nhỏ (SmallSat) theo Khối lượng
Trong thang phân loại quốc tế của Liên đoàn Hàng không Vũ trụ Quốc tế (**IAF** - *International Astronautical Federation*) và Cơ quan Hàng không và Vũ trụ Hoa Kỳ (**NASA** - *National Aeronautics and Space Administration*), các vệ tinh nhỏ (**SmallSat** - *Small Satellite*) được định danh chính xác dựa trên khối lượng ướt khi phóng (**Wet Mass** — bao gồm cả nhiên liệu nếu có):

| Nhóm Vệ tinh | Khối lượng Phóng ($M$) | Ví dụ Điển hình & Cấu hình CubeSat |
| :--- | :--- | :--- |
| **Minisatellite (Vệ tinh nhỏ)** | $100\text{ kg} \le M < 500\text{ kg}$ | Vệ tinh viễn thám cao cấp, chòm vệ tinh Starlink v1 |
| **Microsatellite (Vệ tinh siêu nhỏ)** | $10\text{ kg} \le M < 100\text{ kg}$ | Vệ tinh VNREDSat-1 (115 kg), CubeSat 12U - 27U |
| **Nanosatellite (Vệ tinh nano)** | $1\text{ kg} \le M < 10\text{ kg}$ | **Chuẩn CubeSat tiêu chuẩn: 1U (1.33 kg), 2U, 3U, 6U** |
| **Picosatellite (Vệ tinh pico)** | $0.1\text{ kg} \le M < 1\text{ kg}$ | Vệ tinh dạng bản mạch PocketQube (5 × 5 × 5 cm), Chipsat |
| **Femtosatellite (Vệ tinh femto)** | $M < 0.1\text{ kg}$ ($< 100\text{ g}$) | Vệ tinh vi mạch đơn màng mỏng (Sprite, KickSat) |

---

### 3. Thông số Hình học Tiêu chuẩn 1U (1 Unit) theo Quy chuẩn CalPoly
Một vệ tinh CubeSat đơn vị cơ sở (**1U** - *1 Unit*) phải tuân thủ nghiêm ngặt các quy định kích thước trong tài liệu đặc tả **CubeSat Design Specification (CDS)** do Đại học CalPoly ban hành để đảm bảo vệ tinh có thể trượt vừa vặn trơn tru bên trong ống phóng mà không bao giờ bị kẹt:

![Thông số cơ học tiêu chuẩn CubeSat 1U](../PTITCube-Figs/png/lab0_1u_dimensions.png)

| Thông số Kỹ thuật | Tiêu chuẩn Quy định | Ý nghĩa Kỹ thuật & Dung sai Kiểm định |
| :--- | :--- | :--- |
| **Kích thước Thân cơ sở ($X \times Y$)** | 100.0 × 100.0 mm | Dung sai cực kỳ nghiêm ngặt ±0.1 mm; kiểm tra bằng dưỡng đo cơ khí (**Go/No-Go Gauge**). |
| **Chiều dài Dọc trục Ray ($Z$)** | 113.5 mm | Tính từ mặt phẳng đáy ray đến đỉnh ray. Các thanh ray nhô ra 6.5 mm ở mỗi đầu để ngăn các mặt bên chạm đáy. |
| **Bề rộng Thanh ray (Rails)** | Tối thiểu 8.5 mm | Tiếp xúc trực tiếp với rãnh dẫn hướng của ống phóng **POD** (*Picosatellite Orbital Deployer*) để chịu tải trọng rung động. |
| **Bán kính Bo viền Ray** | Tối thiểu 1.0 mm | Bán kính lượn góc nhẵn bóng nhằm loại bỏ hoàn toàn các gờ sắc nhọn gây cào xước ống phóng. |
| **Độ nhám Bề mặt Ray ($R_a$)** | $R_a \le 0.8\text{ }\mu\text{m}$ | Bề mặt ray trượt phải được mài bóng kỹ thuật cao để giảm thiểu lực ma sát trượt khi bung. |
| **Xử lý Bề mặt Ray (Hard Anodize)** | Anodize Cứng Type III | Tạo lớp gốm oxit nhôm ($\text{Al}_2\text{O}_3$) dày ($25 - 50\text{ }\mu\text{m}$) cách điện, chống mài mòn và chống hàn nguội trong chân không. |
| **Vùng Cấm Nhô (Keep-out Zone)** | Tối đa 6.5 mm | Không có bất kỳ linh kiện, đầu ốc vít hay thấu kính nào được nhô ra khỏi bề mặt ngoài vượt quá 6.5 mm. |
| **Vị trí Trọng tâm (CoM - Center of Mass)** | Cách tâm hình học ≤ 20 mm | Phân bố khối lượng lệch tâm vượt chuẩn sẽ tạo mô-men xoắn gây rung lắc ống phóng và kẹt khi đẩy. |

---

### 4. Cơ chế Phóng P-POD & J-SSOD
CubeSat không được gắn trực tiếp lên tên lửa đẩy mà bắt buộc phải nằm trong một hệ thống phóng dạng hộp kín chuyên dụng (**POD** - *Picosatellite Orbital Deployer*):
1. **P-POD (Poly-Picosatellite Orbital Deployer):** Phát triển bởi CalPoly, là ống phóng kinh điển bằng hợp kim nhôm **Al 7075-T6**. Một ống P-POD tiêu chuẩn có thể chứa một vệ tinh 3U hoặc ba vệ tinh 1U xếp chồng nối tiếp nhau.
2. **J-SSOD (JEM Small Satellite Orbital Deployer):** Hệ thống phóng vệ tinh lắp đặt trên Module Thí nghiệm Nhật Bản **JEM** (*Japanese Experiment Module*, còn gọi là Module Kibo) thuộc Trạm Vũ trụ Quốc tế (**ISS** - *International Space Station*). Các phi hành gia đưa bộ phóng ra ngoài không gian qua cửa điều áp (**Air Lock**) và cánh tay robot Kibo sẽ định hướng ống phóng về phía sau quỹ đạo ISS để đẩy CubeSat ra ngoài.

![Trình tự phóng và triển khai CubeSat lên quỹ đạo](../PTITCube-Figs/png/lab0_deployment_sequence.png)

- **Lò xo đẩy chính (Ejection Spring):** Đặt tại đáy ống phóng, cung cấp lực đẩy tuyến tính ổn định để phóng vệ tinh với vận tốc tương đối từ $1.0\text{ m/s}$ đến $1.5\text{ m/s}$, tránh nguy cơ vệ tinh bay lơ lửng ngay cửa ống gây va chạm với tên lửa.
- **Lò xo tách tầng (Deployment Plungers):** Lắp ở đỉnh các thanh ray trượt. Khi hai vệ tinh CubeSat xếp chồng trong cùng một ống phóng rời khỏi cửa, các lò xo plunger nhỏ sẽ lập tức đẩy hai vệ tinh dạt xa nhau để ngăn ngừa va chạm thứ cấp.
- **Công tắc Hành trình Phóng (Deployment Switch / Inhibit Switch):** Lắp ở chân thanh ray trượt. Khi vệ tinh nằm trong ống phóng, nắp hoặc ray đè chặt công tắc ở trạng thái ngắt mạch điện (**Open Circuit**). Khi vệ tinh vừa trượt ra ngoài không gian, công tắc bung ra, đóng mạch điện kết nối pin sạc với hệ thống phân phối nguồn điện **EPS** (*Electrical Power System*).
- **Quy tắc Im lặng Vô tuyến 30 Phút (30-Minute Radio Silence Rule):** Theo yêu cầu an toàn phóng quốc tế của CalPoly, NASA và Cơ quan Thám hiểm Hàng không Vũ trụ Nhật Bản (**JAXA** - *Japan Aerospace Exploration Agency*), CubeSat **bắt buộc phải giữ im lặng vô tuyến (RF Silence) hoàn toàn trong ít nhất 30 phút đầu tiên** sau khi rời ống phóng. Trong khoảng thời gian này, vi điều khiển không được phát sóng radio (**RF** - *Radio Frequency*), không được kích hoạt đốt nhiệt bung anten hay mở các tấm pin để tránh gây nhiễu cho các thiết bị điều khiển của tên lửa đẩy và các vệ tinh khác phóng cùng cụm.

---

### 5. Môi trường Vật lý Khắc nghiệt tại Quỹ đạo Trái Đất Tầm thấp (LEO)
Quỹ đạo **LEO** (*Low Earth Orbit* - Quỹ đạo Trái Đất Tầm thấp) nằm ở độ cao từ 160 km đến 2000 km so với mực nước biển (phổ biến nhất cho CubeSat là 400 - 600 km, tương đương quỹ đạo trạm ISS $\approx 410\text{ km}$). Môi trường không gian tại đây đặt ra những thử thách kỹ thuật cực đoan:

#### A. Môi trường Chân không Cực cao (Ultra-High Vacuum)
Áp suất khí quyển tại độ cao 500 km suy giảm xuống chỉ còn khoảng $10^{-6}\text{ Pa}$ đến $10^{-8}\text{ Pa}$ ($10^{-8} - 10^{-10}\text{ Torr}$), so với áp suất khí quyển tiêu chuẩn tại mặt đất là $101.325\text{ kPa}$ ($760\text{ Torr}$):
- **Triệt tiêu Hoàn toàn Truyền nhiệt Đối lưu:** Định luật làm mát đối lưu Fourier/Newton $\dot{Q} = h A (T_s - T_\infty)$ hoàn toàn biến mất do không có phân tử khí chuyển động tự do. Toàn bộ nhiệt lượng sinh ra từ bộ xử lý trung tâm (**CPU** - *Central Processing Unit*), mạch sạc pin và máy phát sóng chỉ có thể truyền qua hai cơ chế:
  1. Dẫn nhiệt (**Conduction**) qua lớp đồng của bo mạch in (**PCB** - *Printed Circuit Board*) và khung kim loại nhôm.
  2. Bức xạ nhiệt (**Radiation**) từ bề mặt ngoài của thân vệ tinh vào không gian sâu.
- **Hiện tượng Thoát khí Vật liệu (Outgassing):** Dưới áp suất chân không cực thấp, các hợp chất hữu cơ dễ bay hơi, dung môi tồn dư, chất hóa dẻo trong polyme, băng keo và chất cách điện sẽ thoát ra khỏi bề mặt vật liệu. Các hạt khí này bay lơ lửng và nhanh chóng ngưng tụ lạnh lên bề mặt quang học của ống kính camera làm mờ đục ảnh chụp, hoặc phủ lên tế bào quang điện làm sụt giảm công suất sạc. Tiêu chuẩn **ASTM E595** (tiêu chuẩn thử nghiệm vật liệu không gian của tổ chức tiêu chuẩn quốc tế *ASTM International*) do NASA áp dụng quy định vật liệu dùng trong không gian phải đạt:

$$\text{TML} \le 1.0\%, \quad \text{CVCM} \le 0.1\%$$

  *(Trong đó: **TML** – Total Mass Loss là tổng khối lượng hao hụt; **CVCM** – Collected Volatile Condensable Material là vật liệu bay hơi ngưng tụ thu được).*

- **Hiện tượng Hàn nguội Chân không (Cold Welding):** Trong không khí mặt đất, bề mặt kim loại luôn được bao phủ bởi một màng oxit mỏng tự nhiên và lớp phân tử ẩm. Trong môi trường chân không sâu, lớp oxit này bị triệt tiêu hoặc mài mòn. Khi hai bề mặt kim loại trần cùng loại (nhất là nhôm hoặc thép không gỉ) tiếp xúc trực tiếp dưới lực siết mạnh, các mạng tinh thể kim loại sẽ liên kết tự do, dính chặt vào nhau như được hàn nóng chảy. Đây là lý do các thanh ray CubeSat bắt buộc phải mạ **Anodize cứng Type III** (tạo lớp gốm $\text{Al}_2\text{O}_3$ siêu cứng ngăn cách tiếp xúc kim loại trực tiếp).

#### B. Môi trường Bức xạ Vũ trụ & Hạt Mang điện Năng lượng Cao
Không gian LEO không được tầng khí quyển dày bảo vệ hoàn toàn, vệ tinh liên tục bị bắn phá bởi các chùm hạt proton từ gió Mặt Trời, tia vũ trụ ngân hà (**GCR** - *Galactic Cosmic Rays*) và các điện tử năng lượng cao bị bẫy trong hai vành đai bức xạ Van Allen. Tác động của **bức xạ ion hóa (Ionizing Radiation)** lên linh kiện bán dẫn được chia thành hai nhóm chính:

![Tác động bức xạ không gian và giải pháp giảm thiểu](../PTITCube-Figs/png/lab0_radiation_effects.png)

1. **Tác động Tích lũy — TID (Total Ionizing Dose - Tổng liều Ion hóa Tích lũy):**
   - **Bản chất:** Lượng bức xạ ion hóa tích tụ dần trong lớp oxit cách điện của vi mạch bán dẫn theo suốt thời gian hoạt động trên quỹ đạo.
   - **Hệ quả (Threshold Drift):** Điện tích bị bẫy trong lớp oxit làm trôi điện áp ngưỡng đóng/mở của transistor MOSFET, tăng dòng rò và khiến linh kiện suy giảm hiệu năng hoặc hỏng dần theo thời gian.
   - **Giải pháp giảm nhẹ:** Bọc chắn cơ học bằng vỏ hợp kim nhôm (**Al Shielding** - *Aluminum Shielding*) để hấp thụ và suy giảm cường độ bức xạ chiếu vào bo mạch bên trong.

2. **Hiệu ứng Tức thời — SEE (Single Event Effects - Hiệu ứng Sự kiện Đơn lẻ):**
   - Xảy ra tức thời khi một hạt mang điện năng lượng cao đơn lẻ (proton hoặc ion nặng) xuyên qua vùng nhạy cảm của chip bán dẫn, gây ra ba dạng lỗi phổ biến sau trên CubeSat:
   - **SEU (Single Event Upset - Lật trạng thái bit đơn lẻ):**
     - *Hiện tượng:* Hạt mang điện làm thay đổi lượng điện tích tại một nút nhớ, gây đảo trạng thái bit (`0` ↔ `1`, gọi là **Bit Flip**) trong bộ nhớ truy cập ngẫu nhiên (**RAM** - *Random Access Memory*) hoặc thanh ghi của vi xử lý. Đây là **lỗi mềm** (không làm hỏng vĩnh viễn phần cứng, nhưng làm sai lệch dữ liệu hoặc biến điều khiển).
     - *Giải pháp giảm nhẹ:* Sử dụng mã phát hiện và sửa sai **ECC / EDAC** (*Error-Correcting Code / Error Detection and Correction*, như mã Hamming) và thuật toán lưu trữ dự phòng ba lần (**TMR** - *Triple Modular Redundancy*) trong bộ nhớ để tự động phát hiện và khôi phục bit bị lật.
   - **SEL (Single Event Latchup - Chốt mạch / Ngắn mạch đơn sự kiện):**
     - *Hiện tượng:* Hạt năng lượng cao kích hoạt mở các cấu trúc transistor ký sinh (thyristor ký sinh) bên trong mạch CMOS, tạo thành đường dẫn dòng ngắn mạch nội bộ (**Latchup Short**) giữa nguồn và đất. Dòng điện tăng vọt đột ngột có thể làm nóng chảy và phá hủy chip vĩnh viễn nếu không ngắt kịp thời.
     - *Giải pháp giảm nhẹ:* Bố trí mạch giám sát và giới hạn dòng điện tức thời (**Current Limiter**) trên phân hệ nguồn **EPS** (*Electrical Power System*) để phát hiện quá dòng SEL và thực hiện ngắt rồi cấp lại nguồn cưỡng bức (**Power Cycling**) trong vài mili-giây trước khi chip bị phá hủy do nhiệt.
   - **SEFI (Single Event Functional Interrupt - Gián đoạn Chức năng Đơn sự kiện):**
     - *Hiện tượng:* Hạt năng lượng cao đánh trúng các thanh ghi trạng thái, mạch định thời hoặc bộ điều khiển giao tiếp của vi mạch, khiến máy tính trên bo (**OBC** - *On-Board Computer*) hoặc bus truyền thông bị treo, mất phản hồi hoặc chạy sai trình tự (**CPU / Bus Hang**).
     - *Giải pháp giảm nhẹ:* Sử dụng bộ định thời giám sát phần cứng độc lập (**Hardware Watchdog Timer / Watchdog Reset**) để tự động phát hiện trạng thái treo và khởi động lại OBC an toàn.

#### C. Khí Oxy Nguyên tử (Atomic Oxygen - AO)
Ở độ cao 200 - 600 km, bức xạ cực tím (**UV** - *Ultraviolet*) năng lượng cao từ Mặt Trời bẻ gãy các liên kết phân tử oxy hai nguyên tử ($\text{O}_2$) trong tầng nhiệt khí quyển thành các gốc **oxy nguyên tử tự do ($\text{O}$)** — gọi tắt là **AO** (*Atomic Oxygen*). Khi vệ tinh bay với vận tốc quỹ đạo siêu thanh $\approx 7.7\text{ km/s}$, các nguyên tử oxy này va chạm trực diện với bề mặt đón gió của vệ tinh với động năng khoảng $5\text{ eV}$. Khí oxy nguyên tử có tính oxy hóa cực mạnh, ăn mòn nhanh chóng các bề mặt mạ bạc, ăn mòn polyme Kapton và phá hủy các lớp phủ quang học.

#### D. Cân bằng Nhiệt Quỹ đạo (Thermal Equilibrium in Space)
Nhiệt độ của CubeSat trong không gian là kết quả của sự cân bằng động học giữa tổng công suất nhiệt hấp thụ ($\dot{Q}_{\text{in}}$), nhiệt nội sinh ($Q_{\text{internal}}$) và tổng công suất nhiệt bức xạ ra vũ trụ ($\dot{Q}_{\text{out}}$):

$$\dot{Q}_{\text{in}} + Q_{\text{internal}} = \dot{Q}_{\text{out}}$$

Trong đó:
1. **Nhiệt Hấp thụ ($\dot{Q}_{\text{in}}$):**
   - **Bức xạ Mặt Trời Trực tiếp ($Q_{\text{solar}}$):** Hằng số Mặt Trời tại quỹ đạo Trái Đất $G_{sc} \approx 1361\text{ W/m}^2$. Công suất hấp thụ:

$$Q_{\text{solar}} = \alpha \cdot A_{\text{proj}} \cdot G_{sc}$$

     (với $\alpha$ là hệ số hấp thụ quang học của bề mặt vỏ, $A_{\text{proj}}$ là diện tích hình chiếu vuông góc với tia nắng).
   - **Bức xạ Phản xạ từ Trái Đất (Earth Albedo - $Q_{\text{albedo}}$):** Trái Đất phản xạ lại không gian trung bình khoảng 30% (hệ số phản xạ suất $a \approx 0.30$) thông lượng ánh sáng Mặt Trời:

$$Q_{\text{albedo}} = \alpha \cdot A_{\text{earth}} \cdot G_{sc} \cdot a \cdot F_{\text{sat-earth}}$$

   - **Bức xạ Hồng ngoại Phát ra từ Trái Đất (Earth Infrared - $Q_{\text{IR}}$):** Bản thân Trái Đất phát xạ nhiệt hồng ngoại (**IR** - *Infrared*) như một vật đen ở nhiệt độ $\approx 255\text{ K}$, với thông lượng trung bình $q_{\text{IR}} \approx 237\text{ W/m}^2$ (và $F_{\text{sat-earth}}$ là hệ số góc nhìn giữa vệ tinh và Trái Đất):

$$Q_{\text{IR}} = \varepsilon \cdot A_{\text{earth}} \cdot q_{\text{IR}} \cdot F_{\text{sat-earth}}$$

2. **Nhiệt Nội sinh ($Q_{\text{internal}}$):** Công suất tiêu tán nhiệt từ các linh kiện điện tử đang hoạt động (CPU của máy tính trung tâm **OBC**, mạch sạc pin **EPS**, mạch khuếch đại công suất **PA** - *Power Amplifier* của module truyền thông tầm xa **LoRa** - *Long Range*).
3. **Nhiệt Bức xạ ra Không gian Sâu ($\dot{Q}_{\text{out}}$):** Theo định luật bức xạ vật đen **Stefan-Boltzmann**:

$$\dot{Q}_{\text{out}} = \varepsilon \cdot \sigma \cdot A_{\text{total}} \cdot (T_{\text{sat}}^4 - T_{\text{space}}^4)$$

   (với $\varepsilon$ là hệ số phát xạ hồng ngoại của vật liệu vỏ, hằng số Stefan-Boltzmann $\sigma = 5.67 \times 10^{-8}\text{ W/(m}^2\text{K}^4)$, $A_{\text{total}}$ là tổng diện tích bức xạ $6$ mặt của CubeSat, và nhiệt độ nền vũ trụ $T_{\text{space}} \approx 2.7\text{ K} \approx -270.45^\circ\text{C}$).

![Cân bằng nhiệt vệ tinh CubeSat trên quỹ đạo LEO](../PTITCube-Figs/png/lab0_thermal_balance.png)

- **Biên độ Biến thiên Nhiệt:**
  - **Pha Ban ngày (Sunlight):** Vệ tinh hấp thụ nhiệt mặt trời tối đa, nhiệt độ bề mặt ngoài có thể tăng vọt lên $+80^\circ\text{C}$ đến $+100^\circ\text{C}$.
  - **Pha Vùng tối (Eclipse):** Khi vệ tinh đi vào vùng khuất bóng của Trái Đất ($\approx 35\text{ phút}$ mỗi vòng bay), $Q_{\text{solar}} = 0$ và $Q_{\text{albedo}} = 0$, vệ tinh liên tục mất nhiệt vào vũ trụ, nhiệt độ vỏ tụt dốc xuống $-40^\circ\text{C}$ đến $-50^\circ\text{C}$.
  - Trung bình mỗi ngày vệ tinh trải qua $15 - 16$ chu kỳ sốc nhiệt nóng - lạnh cực đoan, đòi hỏi thiết kế nhiệt thụ động (**Passive Thermal Control**) bằng cách tính toán tỷ số $\alpha/\varepsilon$ của các lớp phủ bề mặt (như băng keo Kapton, mạ niken hoặc sơn anodize đen).

#### E. Động học Quỹ đạo & Thời gian Suy thoái (Orbital Decay)

##### 1. Các Công thức Động học Quỹ đạo Tổng quát (Quỹ đạo tròn)
- **Vận tốc Quỹ đạo Cân bằng (Orbital Velocity):**
  Vệ tinh duy trì chuyển động trên quỹ đạo tròn ở độ cao $h$ khi lực hấp dẫn cân bằng hoàn toàn với lực hướng tâm:

$$\frac{G M_E m}{(R_E + h)^2} = \frac{m v^2}{R_E + h} \implies v = \sqrt{\frac{G M_E}{R_E + h}} = \sqrt{\frac{\mu}{r}}$$

- **Chu kỳ Quỹ đạo (Orbital Period):**
  Thời gian để vệ tinh hoàn thành đúng 1 vòng quay quanh Trái Đất:

$$T = \frac{2\pi r}{v} = 2\pi \sqrt{\frac{r^3}{\mu}} = 2\pi \sqrt{\frac{(R_E + h)^3}{\mu}}$$

- **Số vòng Quỹ đạo trong 1 Ngày Đêm Trái Đất (24 giờ = 86400 giây):**

$$N = \frac{86400}{T_{\text{(s)}}} = \frac{24 \times 60}{T_{\text{(min)}}}$$

  *(Trong đó: $r = R_E + h$ là bán kính quỹ đạo tính từ tâm Trái Đất; $h$ là độ cao quỹ đạo so với mực nước biển; $R_E \approx 6378.14\text{ km}$ là bán kính trung bình Trái Đất; $\mu = G M_E \approx 398600.44\text{ km}^3/\text{s}^2$ là hằng số hấp dẫn địa tâm, với $G$ là hằng số vạn vật hấp dẫn và $M_E$ là khối lượng Trái Đất).*

##### 2. Ví dụ Minh họa Tính toán ở Độ cao $h = 500\text{ km}$
Xét một vệ tinh CubeSat hoạt động trên quỹ đạo tròn LEO ở độ cao phổ biến $h = 500\text{ km}$:
- **Bán kính quỹ đạo:**

$$r = R_E + h = 6378.14 + 500 = 6878.14\text{ km}$$

- **Vận tốc quỹ đạo:**

$$v = \sqrt{\frac{398600.44}{6878.14}} \approx 7.613\text{ km/s} \approx 27400\text{ km/h}$$

- **Chu kỳ quỹ đạo:**

$$T = 2\pi \sqrt{\frac{(6878.14)^3}{398600.44}} \approx 5677\text{ s} \approx 94.6\text{ min}$$

- **Số vòng bay mỗi ngày (24 giờ):**

$$N = \frac{24 \times 60}{94.6} \approx 15.22\text{ orbits/day}$$

##### 3. Lực Cản Khí quyển, Độ cao Quỹ đạo & Quy tắc 25 Năm (IADC 25-Year Rule)
- **Công thức Lực Cản Khí quyển (Atmospheric Drag):**
  Mặc dù rất loãng ở tầng LEO, phần khí quyển tàn dư vẫn tạo ra lực cản khí động học tác dụng ngược chiều bay của vệ tinh:

$$F_{\text{drag}} = \frac{1}{2} \rho v^2 C_d A$$

  *(Trong đó: $\rho$ là mật độ không khí tại độ cao quỹ đạo; $v$ là vận tốc bay của vệ tinh; $C_d \approx 2.2$ là hệ số cản khí động học của khối lập phương; $A$ là diện tích mặt cắt ngang đón gió).*

- **Ảnh hưởng của Độ cao Quỹ đạo đến Thời gian Suy thoái (Orbital Decay):**
  Lực cản $F_{\text{drag}}$ liên tục tiêu hao cơ năng của vệ tinh, làm độ cao quỹ đạo giảm dần cho đến khi vệ tinh rơi vào tầng khí quyển dày và bốc cháy hoàn toàn. Tốc độ suy thoái phụ thuộc rất mạnh vào độ cao ban đầu (do mật độ khí $\rho$ giảm theo hàm mũ khi lên cao):
  - Độ cao $< 200\text{ km}$: Vệ tinh rơi và bốc cháy chỉ trong vài ngày.
  - Độ cao $300 - 400\text{ km}$: Tuổi thọ quỹ đạo kéo dài vài tháng đến $1 - 2$ năm (quỹ đạo lý tưởng cho CubeSat giáo dục).
  - Độ cao $500 - 600\text{ km}$: Tuổi thọ quỹ đạo từ $5$ đến $25$ năm.
  - Độ cao $> 800\text{ km}$: Tồn tại hàng trăm năm, trở thành rác vũ trụ nguy hiểm nếu không có động cơ khử quỹ đạo.

- **Quy chuẩn IADC / Liên Hợp Quốc:** Ủy ban Điều phối Rác Vũ trụ Quốc tế (**IADC** - *Inter-Agency Space Debris Coordination Committee*) quy định mọi vệ tinh LEO sau khi kết thúc sứ mệnh phải tự động khử quỹ đạo và bốc cháy trong vòng không quá **25 năm**. Vệ tinh cũng phải thực hiện quy trình **Thụ động hóa (Passivation)**: xả kiệt dung lượng pin sạc và giải phóng toàn bộ năng lượng dư thừa để tránh nguy cơ phát nổ tạo thành các mảnh vụn không gian.

---

---

## 📖 Doc 0.2: Kiến trúc Hệ thống Vệ tinh 1U & 6 Phân hệ Cốt lõi

### 1. Kiến trúc Hệ thống Phân rã theo Khối Chức năng
Một vệ tinh CubeSat là một hệ sinh thái cơ điện tử thu nhỏ, trong đó các thành phần được tổ chức thành các phân hệ chức năng phối hợp chặt chẽ với nhau. Xét theo mục tiêu nhiệm vụ, kiến trúc vệ tinh được chia thành hai nhóm lớn:
1. **Hệ thống Nền tảng (Bus System):** Tập hợp các phân hệ kỹ thuật cơ bản đóng vai trò hạ tầng dịch vụ hỗ trợ — bảo đảm độ bền cơ khí, cung cấp năng lượng, điều khiển nhiệt, xử lý lệnh/dữ liệu, ổn định hướng và duy trì liên lạc với Trái Đất.
2. **Tải trọng Sứ mệnh (Payload):** Bộ phận trực tiếp thực hiện mục tiêu khoa học, quan sát hoặc thử nghiệm công nghệ của chuyến bay — đây chính là lý do tồn tại của toàn bộ hệ thống vệ tinh trên quỹ đạo (ví dụ: camera chụp ảnh bề mặt Trái Đất, máy thu tín hiệu hàng hải AIS/ADS-B, cảm biến đo bức xạ hoặc mô-đun quan sát khí tượng).


![Kiến trúc hệ thống CubeSat tổng thể](../PTITCube-Figs/png/system.png)

> **Lưu ý về phân chia phân hệ:** Theo tài liệu kỹ thuật hệ thống CubeSat tiêu chuẩn, vệ tinh thường được phân tách thành 7 nhóm chức năng (Kết cấu cơ khí, Nguồn điện EPS, Xử lý dữ liệu & điều khiển C&DH, Thông tin liên lạc COM, Xác định & điều khiển hướng ADCS, Điều khiển nhiệt TCS và Tải hữu ích Payload). Trên bộ kit giáo dục **PTITCube 1U**, hệ thống được tổ chức gọn thành **6 Phân hệ Cốt lõi**: các cảm biến giám sát trạng thái và tải trọng nhiệm vụ được gộp chung thành phân hệ **Cảm biến & Tải trọng Sứ mệnh**, còn chức năng **Điều khiển Nhiệt (TCS)** được tích hợp trực tiếp ở dạng thụ động thông qua khung nhôm dẫn nhiệt và giám sát nhiệt độ trên các cảm biến/mạch sạc.

---

### 2. Chi tiết Chức năng & Thông số 6 Phân hệ Cốt lõi

#### 1. Phân hệ Xử lý Dữ liệu & Điều khiển (OBC / C&DH - On-Board Computer / Command & Data Handling)
Trong kiến trúc vệ tinh, **C&DH** là phân hệ đóng vai trò trung tâm điều phối toàn bộ hoạt động trên khoang, còn **OBC** (*On-Board Computer*) là máy tính phần cứng hạt nhân thực thi các tác vụ đó.

- **Chức năng Cốt lõi (3 nhóm nhiệm vụ chính):**
  1. **Xử lý Lệnh Điều khiển (Telecommand Processing):** Tiếp nhận các gói tin lệnh từ phân hệ truyền thông **COM** gửi lên qua tuyến đường lên (*Uplink*), thực hiện giải mã, kiểm tra cấu trúc, tính hợp lệ và xác thực an toàn trước khi thực thi ngay hoặc lập lịch thực hiện, sau đó phân phối lệnh tới các phân hệ liên quan.
  2. **Thu thập, Xử lý & Quản lý Dữ liệu (Data Handling):** Thu thập định kỳ hai luồng dữ liệu chính là **Dữ liệu giám sát trạng thái/sinh tồn (HK - *Housekeeping Telemetry*)** (điện áp, dòng điện, nhiệt độ, trạng thái thiết bị) và **Dữ liệu nhiệm vụ (Mission Data)** từ tải hữu ích; tiến hành định dạng, gắn nhãn thời gian (*timestamp*), lưu đệm vào bộ nhớ và đóng gói trước khi chuyển cho COM phát xuống mặt đất (*Downlink*).
  3. **Tự chủ Hệ thống & Phục hồi Lỗi (Autonomy & FDIR - *Fault Detection, Isolation, and Recovery*):** Tự động giám sát sức khỏe vệ tinh khi không có liên lạc trực tiếp với trạm mặt đất. Khi phát hiện treo phần mềm, điện áp thấp hoặc sự cố ngoại vi, OBC kích hoạt bộ định thời giám sát phần cứng (**Watchdog Timer**), chuyển sang chế độ an toàn (*Safe Mode*) hoặc khởi động/ngắt mở nguồn thiết bị bị lỗi.

- **Phần cứng & Thông số trên PTITCube 1U:**
  - **Bộ xử lý trung tâm:** Vi điều khiển 32-bit lõi kép **ESP32-WROOM-32** (kiến trúc Xtensa LX6), xung nhịp tối đa 240 MHz, cho phép tách luồng xử lý tác vụ thu phát và tác vụ điều khiển trên hệ điều hành thời gian thực.
  - **Bộ nhớ & Lưu trữ:** 520 KB SRAM nội bộ (dữ liệu làm việc), 4 MB SPI Flash (lưu chương trình bay), kết hợp khe cắm thẻ nhớ **microSD** định dạng FAT32 qua giao tiếp **SPI** tốc độ cao để lưu nhật ký bay (*Flight Log*) và ảnh chụp.
  - **Nguồn thời gian & Bus giao tiếp nội bộ:** Duy trì mốc thời gian bằng bộ đếm thời gian thực (**RTC** - *Real-Time Clock*) kết hợp đồng bộ thời gian chuẩn UTC từ module định vị **GNSS**; kết nối với các phân hệ khác qua các chuẩn bus nối tiếp phổ biến trên CubeSat gồm **I2C** (cảm biến, đo pin), **SPI** (thẻ nhớ microSD) và **UART** (module LoRa, GNSS, camera). *(Trên các CubeSat quy mô lớn hoặc yêu cầu chống nhiễu cao hơn, bus nội bộ còn có thể dùng chuẩn vi sai RS-422/485 hoặc mạng CAN).*

---

#### 2. Phân hệ Năng lượng Điện (EPS - Electrical Power System)
**EPS** đảm nhiệm việc thu nhận (phát điện), lưu trữ, điều hòa điện áp, phân phối và bảo vệ nguồn điện cho toàn bộ vệ tinh. Khả năng cấp nguồn quyết định trực tiếp thời lượng vận hành tải hữu ích, công suất phát sóng và sự sống còn của vệ tinh.

- **Nguyên lý Vận hành Theo Chu kỳ Quỹ đạo:**
  - **Khi được chiếu sáng (Sunlight):** Các tấm pin mặt trời chuyển đổi bức xạ Mặt Trời thành dòng điện một chiều (**DC**) để vừa cấp cho tải đang hoạt động, vừa nạp năng lượng dự trữ vào khối pin.
  - **Khi đi vào vùng bóng tối (Eclipse) hoặc tải đỉnh:** Khi nguồn quang điện không đủ (lúc bị Trái Đất che khuất hoặc khi máy phát RF và tải nặng cùng hoạt động), khối pin lưu trữ sẽ cung cấp phần điện năng còn thiếu.

- **Phương thức Khai thác Năng lượng & Quản lý Nguồn:**
  - Trên CubeSat có hai phương thức khai thác mảng pin mặt trời phổ biến là **Truyền năng lượng trực tiếp (DET - *Direct Energy Transfer*)** (cấu trúc đơn giản, nối mảng pin tới đường nguồn qua mạch điều chỉnh song song) và **Bám điểm công suất cực đại (MPPT - *Maximum Power Point Tracking*)** (dùng bộ biến đổi DC–DC điều chỉnh điểm làm việc của mảng pin để đạt công suất tối đa khi góc chiếu và nhiệt độ thay đổi).
  - Trong thiết kế ngân sách năng lượng (*Power Budget*), cần kiểm tra đồng thời **công suất trung bình** (bảo đảm cân bằng năng lượng trong một vòng quỹ đạo và nạp lại kịp trước pha tối tiếp theo) và **công suất đỉnh** (bảo đảm pin và mạch biến đổi chịu được thời điểm nhiều tải cùng bật). Khối phân phối nguồn phải có chức năng bảo vệ quá dòng/ngắn mạch để cô lập nhánh lỗi, tránh sập nguồn toàn hệ thống.

- **Phần cứng & Thông số trên PTITCube 1U:**
  - **Nguồn Thu Quang điện:** 4 mặt thân ngoài gắn các cell pin mặt trời đơn tinh thể (*Monocrystalline Solar Cells*) có hiệu suất chuyển đổi khoảng 19 - 22%.
  - **Nguồn Lưu trữ:** 1 pack pin sạc **Lithium-Polymer (Li-Po)** đơn cell, dung lượng 1200 - 2000 mAh, điện áp danh định 3.7 V (ngưỡng sạc đầy 4.2 V, ngưỡng ngắt xả bảo vệ 3.0 V). *Lưu ý:* Khi thiết kế thực tế cần kiểm soát chặt độ sâu xả cho phép (**DoD** - *Depth of Discharge*), dòng sạc/xả tối đa và giới hạn nhiệt độ cell pin theo tài liệu nhà sản xuất.
  - **Mạch Quản lý Sạc:** IC sạc **TP4056** thực hiện chu trình sạc hai giai đoạn chuẩn **CC/CV** (*Constant Current / Constant Voltage* — dòng không đổi rồi áp không đổi) với dòng sạc danh định 1000 mA, tích hợp cảm biến nhiệt điện trở **NTC** để ngắt sạc khi pin quá nhiệt.
  - **Mạch Chuyển đổi DC-DC:** Mạch hạ áp **Buck** hiệu suất cao (> 90%) tạo đường nguồn ổn định 3.3 V (cho MCU, cảm biến I2C) và mạch nâng áp **Boost** tạo đường nguồn 5.0 V (cho module LoRa, GPS) từ điện áp biến thiên của pin (3.0 - 4.2 V).
  - **Mạch Giám sát Nhiên liệu (Fuel Gauge):** Chip **MAX17048** đo điện áp cell và ước lượng phần trăm dung lượng còn lại (**SOC** - *State of Charge*) bằng thuật toán *ModelGauge* qua giao tiếp **I2C** mà không cần điện trở *shunt* gây tổn hao công suất.

---

#### 3. Phân hệ Cảm biến & Tải trọng Sứ mệnh (Sensors & Payload)
Trong hệ thống CubeSat, cần phân biệt rõ hai nhóm thành phần: **các cảm biến nền tảng** (cung cấp dữ liệu vật lý cho OBC và ADCS để giám sát sức khỏe, định vị, xác định hướng) và **tải trọng sứ mệnh (Payload)** (thiết bị trực tiếp thực hiện nhiệm vụ khoa học hoặc quan sát của vệ tinh). Trên kit PTITCube, cả hai nhóm được tích hợp đồng bộ trên cùng phân hệ:

- **Nhóm Cảm biến Môi trường & Giám sát Nội bộ:**
  - **Cảm biến Khí quyển BMP280:** Cảm biến áp suất và nhiệt độ vi cơ điện tử (**MEMS**) của Bosch giao tiếp qua **I2C**, dải đo áp suất 300 - 1100 hPa, dải nhiệt độ -40°C đến +85°C. Trong thực hành trên mặt đất hoặc tầng khí quyển thấp, OBC sử dụng công thức khí áp chuẩn quốc tế (ICAO) để ước lượng độ cao khí áp $h$ (m):

$$h = 44330 \cdot \left[1 - \left(\frac{P}{P_0}\right)^{0.1903}\right]$$

  *(Trong đó: $P$ là áp suất đo được tại vị trí vệ tinh; $P_0 = 1013.25\text{ hPa}$ là áp suất chuẩn tại mực nước biển).*

- **Nhóm Cảm biến Xác định Tư thế & Định vị (phục vụ ADCS và C&DH):**
  - **Cảm biến Quán tính 6-DoF MPU6500:** Kết hợp gia tốc kế 3 trục (*Accelerometer*) và con quay hồi chuyển 3 trục (*Gyroscope*) đo vận tốc góc quay quanh 3 trục $X, Y, Z$. Đặt gần trọng tâm vệ tinh để giảm sai số gia tốc ký sinh. *Đặc tính kỹ thuật:* Con quay hồi chuyển cập nhật biến thiên tư thế rất nhanh nhưng có sai số trôi (*bias drift*) tích lũy theo thời gian, nên cần kết hợp với cảm biến tham chiếu tuyệt đối để hiệu chỉnh.
  - **Cảm biến Từ trường QMC5883L:** Từ kế 3 trục đo vectơ cảm ứng từ cục bộ $(B_x, B_y, B_z)$ để tính góc phương vị la bàn (*Heading*) và làm vectơ tham chiếu xác định tư thế:

$$\text{Heading} = \text{atan2}(B_y, B_x) \cdot \frac{180}{\pi}$$

  *Lưu ý lắp đặt:* Từ kế không tự phân biệt được từ trường Trái Đất với từ trường nhiễu do dòng điện trên vệ tinh sinh ra, nên cần đặt xa dây nguồn công suất lớn và động cơ.
  - **Module Định vị Toàn cầu GNSS L76X:** Thu đồng thời tín hiệu từ hệ thống **GPS** (Mỹ) và **GLONASS** (Nga), xuất các bản tin chuẩn **NMEA 0183** (như `GPRMC` chứa tọa độ phẳng/thời gian UTC, `GPGGA` chứa độ cao *geoid*) qua **UART** ở tốc độ 9600 bps. Ngoài vị trí quỹ đạo, tín hiệu từ GNSS còn giúp đồng bộ thời gian chuẩn cho toàn hệ thống OBC.

- **Tải trọng Sứ mệnh Viễn thám (Payload):**
  - **Camera OV2640:** Cảm biến ảnh CMOS độ phân giải 2 Megapixel (1600 × 1200 pixel), tích hợp bộ nén phần cứng **JPEG** giúp giảm dung lượng mỗi bức ảnh xuống còn khoảng 20 - 50 KB. Việc nén nhỏ dung lượng ngay tại tải trọng là yêu cầu bắt buộc để phù hợp với giới hạn bộ nhớ và tốc độ truyền dữ liệu băng thông hẹp của CubeSat 1U.

---

#### 4. Phân hệ Thông tin Liên lạc (COM - Communication Subsystem)
**COM** là cầu nối vô tuyến hai chiều duy nhất giữa CubeSat trên quỹ đạo và trạm mặt đất (**GS** - *Ground Station*). Nếu phân hệ COM mất kết nối không thể phục hồi, trạm mặt đất sẽ mất hoàn toàn khả năng điều khiển và thu nhận dữ liệu sứ mệnh.

- **Chức năng Theo Hai Chiều Liên kết:**
  - **Tuyến Đường lên (Uplink):** Thu tín hiệu RF mang lệnh điều khiển từ trạm mặt đất qua khối thu (**RX** - *Receiver*: gồm bộ khuếch đại tạp âm thấp **LNA**, bộ lọc thông dải loại nhiễu ngoài băng, bộ hạ tần và giải điều chế) rồi chuyển dữ liệu lệnh cho C&DH.
  - **Tuyến Đường xuống (Downlink):** Nhận dữ liệu viễn trắc trạng thái (HK), tín hiệu định vị (*Beacon*) và dữ liệu nhiệm vụ từ C&DH, đưa qua khối phát (**TX** - *Transmitter*: bộ điều chế, nâng tần và bộ khuếch đại công suất cao tần **PA** - *Power Amplifier*) để bức xạ qua anten về trạm mặt đất.
  - *Lưu ý về công suất:* Công suất phát vô tuyến ($P_{\text{RF}}$) đưa vào tính toán ngân sách đường truyền (*Link Budget*) **khác với** công suất điện một chiều ($P_{\text{DC}}$) tiêu thụ từ pin, vì hiệu suất chuyển đổi DC–RF của bộ khuếch đại công suất PA thường chỉ đạt khoảng 30 - 50% và phần năng lượng dư tỏa ra dưới dạng nhiệt.

- **Thách thức Truyền thông trên Quỹ đạo Thấp (LEO):**
  - **Cửa sổ liên lạc ngắn:** Vệ tinh ở độ cao 400 - 600 km bay với vận tốc xấp xỉ 7.6 km/s, nên mỗi lần bay qua vùng nhìn của một trạm mặt đất thường chỉ kéo dài khoảng 5 - 12 phút (khoảng 4 - 6 lượt mỗi ngày).
  - **Suy hao & Dịch tần Doppler:** Tín hiệu chịu suy hao không gian tự do (**FSPL** - *Free-Space Path Loss*) rất lớn trên cự ly hàng trăm đến hơn 2000 km, cộng thêm hiện tượng dịch tần **Doppler** (có thể đạt tới ±10 kHz ở băng UHF) do chuyển động tương đối tốc độ cao giữa vệ tinh và trạm.

- **Phần cứng & Thông số trên PTITCube 1U:**
  - **Module Thu phát Vô tuyến:** **LoRa (Long Range) Ebyte E32** hoạt động ở băng tần **UHF 433 MHz** (công suất phát RF tối đa 20 dBm ≈ 100 mW, hoặc 30 dBm ≈ 1 W ở phiên bản công suất cao).
  - **Kỹ thuật Điều chế:** Công nghệ trải phổ **CSS** (*Chirp Spread Spectrum*) giúp đạt độ nhạy thu cực cao (xuống tới -138 dBm), cho phép giải mã tín hiệu yếu ngay cả khi chìm dưới mức nhiễu nền, phù hợp với tuyến truyền tốc độ thấp yêu cầu độ tin cậy cao trên CubeSat 1U.
  - **Hệ thống Anten:** Sử dụng anten đơn cực (*Monopole*) dài 1/4 bước sóng ($\lambda/4$). Ở tần số $f = 433\text{ MHz}$:

$$\lambda = \frac{c}{f} = \frac{3 \times 10^8\text{ m/s}}{433 \times 10^6\text{ Hz}} \approx 0.6928\text{ m} = 69.28\text{ cm} \implies L_{\text{anten}} = \frac{\lambda}{4} \approx 17.32\text{ cm}$$

  Anten được làm từ lá thép lò xo đàn hồi (*Spring steel tape measure*) thuộc nhóm **anten triển khai**: cuộn gọn trong thân khi nằm trong ống phóng và tự bung thẳng khi ra quỹ đạo, lấy khung nhôm vệ tinh làm mặt phẳng đất phản xạ (*Ground Plane*).

---

#### 5. Phân hệ Kết cấu Cơ khí (Structure Subsystem) & Kiểm soát Nhiệt Thụ động
Phân hệ kết cấu cơ khí tạo bộ khung chịu lực, định hình bao hình tiêu chuẩn, liên kết và bảo vệ toàn bộ bo mạch, cảm biến và tải trọng trong suốt quá trình lắp ráp, vận chuyển, phóng tên lửa và vận hành trên quỹ đạo.

- **Chức năng Cơ khí & Tải trọng Khi Phóng:**
  - Trong giai đoạn tên lửa rời bệ phóng, kết cấu phải chịu đồng thời nhiều dạng tải trọng khắc nghiệt: gia tốc gần tĩnh (lên tới ~ 10 - 15 g), rung động hình sin tần số thấp (dưới 100 Hz), rung động ngẫu nhiên & âm học dải rộng (20 - 2000 Hz) và tải sốc khi tách tầng.
  - Do đó, độ cứng ($k$) và khối lượng ($m$) của khung phải được thiết kế sao cho **tần số dao động riêng** $f_n = \frac{1}{2\pi}\sqrt{k/m}$ đủ cao (thường hướng tới ≥ 100 Hz với vệ tinh nhỏ) nhằm tách khỏi vùng tần số kích thích chính của tên lửa đẩy và tránh cộng hưởng phá hủy kết cấu.
  - Kết cấu gồm các thanh khung, 4 thanh ray dẫn hướng (tiếp xúc ít nhất 75% chiều dài với rãnh trượt của bộ triển khai **P-POD**), tấm mặt ngoài và giá đỡ bo mạch bên trong.

- **Tích hợp Kiểm soát Nhiệt Thụ động (TCS - *Thermal Control Subsystem*):**
  - Trong chân không không có truyền nhiệt đối lưu, khung nhôm đóng vai trò cầu dẫn nhiệt chủ đạo đưa nhiệt lượng sinh ra từ CPU, mạch sạc pin và bộ khuếch đại phát sóng ra các tấm vỏ ngoài để bức xạ vào không gian, đồng thời bảo vệ khối pin Li-Po nhạy cảm nhiệt độ.

- **Thông số Chế tạo & Chuẩn Lắp ghép trên PTITCube 1U:**
  - **Vật liệu & Xử lý Bề mặt:** Phay CNC từ hợp kim nhôm hàng không **Al 6061-T6** (hoặc **Al 7075-T6**), vừa nhẹ, bền cơ học, vừa dẫn nhiệt tốt ($k \approx 167\text{ W/(m}\cdot\text{K)}$). *Lưu ý:* Bề mặt thanh ray tiếp xúc với bộ phóng bắt buộc phải **anốt hóa cứng (Hard Anodize)** để cách điện, chống mài mòn và ngăn hiện tượng hàn nguội trong chân không; trong khi các mặt tiếp xúc nối đất (*Chassis Ground*) dùng lớp phủ chuyển hóa dẫn điện như *Alodine*.
  - **Chuẩn Ngăn xếp Bo mạch (PC/104):** Các bo mạch phân hệ có kích thước tiêu chuẩn 90 × 96 mm, xếp chồng song song qua 4 cọc đồng lục giác (*Standoffs*) cao 16 mm và cố định bằng bu-lông Inox **A2-70**.
  - **Keo Khóa Ren (Threadlocker):** Toàn bộ mối ghép bu-lông sử dụng keo khóa ren kỵ khí độ thoát khí thấp (như *Loctite 222 / 242*) để chống tự tháo dưới rung động phóng và tuân thủ chuẩn thoát khí trong chân không.

---

#### 6. Phân hệ Xác định & Điều khiển Hướng (ADCS - Attitude Determination and Control Subsystem)
**ADCS** chịu trách nhiệm xác định tư thế (định hướng của thân vệ tinh so với hệ quy chiếu quán tính hoặc hệ quỹ đạo) và vận tốc góc, sau đó điều khiển vệ tinh xoay về hướng mong muốn và giữ ổn định. *(Lưu ý: ADCS điều khiển chuyển động quay quanh trọng tâm của thân vệ tinh, khác với việc điều chỉnh vị trí hay quỹ đạo bay).*

- **Thách thức Môi trường & 4 Mô-men Nhiễu Quỹ đạo LEO (Environmental Disturbance Torques):**
  - *Mô-men Gradient Trọng trường ($T_{gg}$):* Do chênh lệch lực hấp dẫn lên các phần thân vệ tinh; kéo trục quán tính nhỏ nhất về hướng xuyên tâm Trái Đất (*Nadir*).
  - *Mô-men Khí động ($T_a$):* Va chạm với khí quyển tầng cao (400 - 600 km) ở vận tốc ~ 7.6 km/s; sinh mô-men lật khi tâm cản lệch trọng tâm (**CoM** - *Center of Mass*).
  - *Mô-men Áp suất Bức xạ Mặt Trời ($T_{sp}$):* Va chạm của hạt photon Mặt Trời lên bề mặt vỏ và tấm pin; sinh mô-men khi tâm diện tích phản xạ không trùng CoM.
  - *Mô-men Từ dư Thân Vệ tinh ($T_m$):* Tương tác giữa từ trường nội bộ (dòng điện trên bo mạch, dây dẫn, pin) với từ trường Trái Đất.

- **Chu trình Vòng kín & Các Chế độ Điều khiển Theo Nhiệm vụ:**
  Tiến trình ADCS vận hành theo vòng lặp khép kín: *Cảm biến đo đạc $\rightarrow$ Ước lượng tư thế $\rightarrow$ So sánh hướng đặt $\rightarrow$ Thuật toán điều khiển $\rightarrow$ Cơ cấu chấp hành $\rightarrow$ Đo phản hồi*. Tùy giai đoạn, ADCS thực hiện 3 chế độ nhiệm vụ chính:
  1. *Chế độ Dập quay (Detumbling Mode):* Kích hoạt ngay sau khi tách ống phóng POD để triệt tiêu vận tốc góc quay hỗn loạn ban đầu bằng thuật toán $B$-dot (đo $\dot{B}$ và kích hoạt từ trường ngược chiều).
  2. *Chế độ Bám Mặt Trời (Sun-Pointing Mode):* Hướng các bề mặt gắn pin quang điện đối diện Mặt Trời để tối ưu hóa công suất sạc cho phân hệ **EPS**.
  3. *Chế độ Bám Mục tiêu / Bám Trạm Mặt Đất (Target / Ground Station Pointing Mode):* Điều khiển trục camera (**Payload**) hướng thẳng xuống Trái Đất (*Nadir*) hoặc hướng búp sóng anten (**COM**) bám theo trạm mặt đất trong cửa sổ bay qua (5 - 12 phút).

- **Cơ cấu Chấp hành Thực tế vs Bão hòa Động lượng (Actuators & Desaturation):**
  - **Trên CubeSat quỹ đạo thực tế:** Kết hợp **bánh đà phản ứng (*Reaction Wheel*)** tạo mô-men phản lực 3 trục chính xác bằng cách biến thiên tốc độ quay của đĩa quán tính ($\tau = -\dot{h}$). Dưới tác động tích lũy của mô-men nhiễu môi trường, bánh đà sẽ quay nhanh dần đến ngưỡng cực đại (**Bão hòa Động lượng - Momentum Saturation**). Khi đó, vệ tinh kích hoạt **bộ tạo mô-men từ (*Magnetorquer*)** tương tác với từ trường Trái Đất ($\tau_m = m \times B$) sinh ngoại lực để **xả động lượng (Desaturation / Momentum Dumping)**, đưa tốc độ bánh đà về mức an toàn mà không tốn nhiên liệu đẩy.
  - **Trên Bộ Kit Giáo dục PTITCube 1U:**
    - *Xác định tư thế:* Tích hợp cảm biến quán tính 6 trục **MPU6500** và từ kế 3 trục **QMC5883L** qua thuật toán lọc bù / Mahony / Madgwick để ước lượng góc **Roll, Pitch, Yaw** thời gian thực.
    - *Cơ cấu chấp hành mô phỏng:* Dùng **động cơ rung lệch tâm (ERM - *Eccentric Rotating Mass*)** điều khiển bằng xung **PWM** kết hợp còi chip âm báo, giúp sinh viên quan sát trực quan nguyên lý tạo mô-men phản lực cơ học và vòng phản hồi điều khiển tư thế trên bàn thí nghiệm.

---

#### Bảng Tổng hợp Nhanh 6 Phân hệ Cốt lõi trên Kit PTITCube 1U

| Phân hệ | Vai trò Chính | Thành phần / Thông số Cốt lõi trên PTITCube | Giao tiếp / Đường Nguồn |
| :--- | :--- | :--- | :--- |
| **1. OBC / C&DH** | Bộ não điều phối, xử lý lệnh, đóng gói dữ liệu HK & nhiệm vụ, FDIR | **ESP32-WROOM-32** (2 nhân 240 MHz, 520 KB SRAM, 4 MB Flash), microSD FAT32 | Bus chủ: I2C, SPI, UART; Nguồn 3.3 V |
| **2. EPS** | Thu nhận, lưu trữ, biến đổi, phân phối & giám sát năng lượng | Pin mặt trời 4 mặt (19 - 22%), Li-Po 3.7 V (1200 - 2000 mAh), **TP4056** (CC/CV), Buck 3.3 V / Boost 5 V, **MAX17048** | I2C (`0x36` cho MAX17048); cấp rail 3.3 V & 5.0 V |
| **3. Sensors & Payload** | Đo thông số môi trường, vị trí/tư thế & chụp ảnh viễn thám | **BMP280** (áp suất/nhiệt độ), **MPU6500** (IMU 6 trục), **QMC5883L** (từ kế), **GNSS L76X** (GPS/GLONASS), **OV2640** (Camera 2 MP JPEG) | I2C (BMP280, MPU6500, QMC5883L), UART (L76X, OV2640 qua 74HC4066) |
| **4. COM** | Cầu nối vô tuyến 2 chiều Uplink/Downlink với trạm mặt đất | **LoRa Ebyte E32** (UHF 433 MHz, CSS, độ nhạy -138 dBm, 20/30 dBm), anten đơn cực λ/4 ≈ 17.32 cm tự bung | UART; Nguồn 5.0 V |
| **5. Structure (& TCS)** | Chịu tải phóng, bảo vệ cơ khí, chuẩn hóa kết nối & dẫn nhiệt thụ động | Khung CNC **Al 6061-T6** / 7075-T6, ray Hard Anodize, chuẩn ngăn xếp **PC/104** (90 × 96 mm, standoff 16 mm), keo khóa ren | Liên kết cơ khí, tản nhiệt dẫn |
| **6. ADCS** | Ước lượng tư thế & mô phỏng điều khiển ổn định hướng | Thuật toán lọc bù / Mahony / Madgwick (từ **MPU6500** + **QMC5883L**); chấp hành thực nghiệm: động cơ **ERM** (PWM) & còi báo | Dữ liệu I2C; Điều khiển PWM từ OBC |

> **Tổng kết Kiến trúc Hệ thống:** Sáu phân hệ kỹ thuật trên kết hợp chặt chẽ tạo thành một hệ sinh thái CubeSat 1U hoàn chỉnh, tuân thủ đúng chuẩn công nghiệp vũ trụ CalPoly CDS và triết lý COTS hiện đại. Toàn bộ các luồng dữ liệu điều khiển, viễn trắc sinh tồn và công suất nguồn được liên kết xuyên suốt qua ngăn xếp PC/104, tạo nền tảng vững chắc để người học tiếp tục triển khai các bài thực hành cấu hình và lập trình chuyên sâu từ Lab 1 đến Lab 8.

---

## 💻 Hướng Dẫn Thực Hành: Khám Phá Mô Hình 3D & Thông Số Phân Hệ

### 3.1. Mục tiêu Bài Thực hành Lab 0
- Làm quen với giao diện học tập tương tác trực quan 3D trên nền tảng **CubeSat-LMS**.
- Nắm vững vị trí không gian, cách thức bố trí ngăn xếp PC/104 và mối liên kết cơ điện tử giữa các linh kiện thuộc 6 phân hệ CubeSat 1U.
- Tra cứu bảng đặc tả thông số kỹ thuật (Datasheet / Spec) chi tiết của từng chip IC, cảm biến và module phần cứng.
- Rèn luyện kỹ năng quan sát hình học không gian, phân biệt rõ ràng giữa ranh giới cơ khí của thân vệ tinh và thanh ray trượt tiêu chuẩn.

### 3.2. Hướng dẫn Tương tác Không gian 3D trên Web LMS
Trình mô phỏng 3D của CubeSat-LMS được xây dựng trên nền tảng WebGL mượt mà, cho phép người học thao tác không gian đa chiều trực tiếp trên trình duyệt mà không cần cài đặt phần mềm ngoài:

![Thao tác điều khiển mô phỏng 3D WebGL](../PTITCube-Figs/png/lab0_ui_controls.png)

### 3.3. Quy trình Thực hiện Bài Thực hành Từng bước
1. **Bước 1 - Khởi động Trình duyệt & Mở Trình Mô phỏng 3D:**
   - Truy cập vào hệ thống CubeSat-LMS, đăng nhập tài khoản học viên và mở **Lab 0: Tổng quan Kỹ thuật Không gian**. Chờ mô hình 3D của vệ tinh tải hoàn tất vào bộ nhớ đồ họa WebGL.
2. **Bước 2 - Khám phá Khung sườn Cơ khí & Thanh ray Trượt:**
   - Sử dụng thao tác xoay (Orbit) quan sát 4 thanh ray nhôm anodize cứng ở 4 góc dọc theo trục Z. Quan sát công tắc hành trình phóng (Deployment Switch) ở mặt đáy thanh ray và xác định vùng cấm nhô linh kiện (6.5 mm).
3. **Bước 3 - Khám phá Tầng Bo mạch Đáy (Tầng 1: Nguồn & Máy tính Trung tâm):**
   - Phóng to vào tầng bo mạch dưới cùng. Nhấp chuột vào vi xử lý **ESP32-WROOM-32** (2 nhân 240 MHz, 520 KB SRAM, 4 MB Flash), mạch sạc **TP4056**, pack pin sạc **Li-Po 3.7V** và chip đo dung lượng pin **MAX17048**.
4. **Bước 4 - Khám phá Tầng Bo mạch Giữa (Tầng 2: Cảm biến & Viễn thông LoRa):**
   - Nhấp chọn cảm biến quán tính 6 trục **MPU6500**, từ kế **QMC5883L**, cảm biến khí quyển áp suất/nhiệt độ **BMP280**, module truyền thông vô tuyến **LoRa 433 MHz** và module định vị **GNSS L76X**.
5. **Bước 5 - Khám phá Tầng Đỉnh & Tải trọng Viễn thám (Tầng 3: Camera & Anten):**
   - Quan sát cụm camera viễn thám **OV2640 2 Megapixel** hướng qua lỗ khoét tròn trên tấm nắp đỉnh nhôm, cơ cấu cuộn lá thép anten thước cuộn 433 MHz và dây cước giữ nhiệt bung anten (Burn-wire).
6. **Bước 6 - Báo cáo Kết quả & Đối chiếu Bảng Thông số:**
   - Điền đầy đủ các thông số điện áp hoạt động, dòng điện tiêu thụ và giao thức truyền thông của từng linh kiện vào Báo cáo Thực hành Lab 0 theo mẫu quy định.
