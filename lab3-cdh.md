# Lab 3: Xử lý dữ liệu, lệnh và lưu trữ

---

## 📖 Doc 3.1: SPI và hệ thống tệp FAT32 trên microSD

### 1. Lưu trữ dữ liệu trên vệ tinh
Phân hệ xử lý dữ liệu và lệnh (C&DH) quản lý thông tin trên CubeSat. Trong mỗi vòng quỹ đạo 95 phút, vệ tinh chỉ liên lạc với trạm mặt đất khoảng 8–12 phút. Thời gian còn lại, phân hệ C&DH phải:
- **Lưu trữ dữ liệu viễn trắc liên tục:** Ghi lại toàn bộ hành trình điện áp, dòng điện, nhiệt độ, tư thế quán tính và tọa độ vào thẻ nhớ MicroSD.
- **Lưu trữ dữ liệu sứ mệnh:** Lưu ảnh và số liệu quan trắc để gửi về trạm mặt đất.
- **Ghi nhật ký sự kiện:** Lưu thời điểm cảnh báo, lỗi vi xử lý và các lần phục hồi hệ thống.

![Sơ đồ nối bus SPI hai chiều](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab3_spi_connection.png)

### 2. Giao thức SPI
Chuẩn **SPI** truyền dữ liệu đồng bộ qua bốn dây, cho phép gửi và nhận đồng thời:
- **SCLK / SCK:** Xung nhịp do thiết bị chủ tạo; tốc độ có thể đạt 20–40 MHz.
- **MOSI:** Dữ liệu từ thiết bị chủ đến thiết bị phụ.
- **MISO:** Dữ liệu từ thiết bị phụ về thiết bị chủ.
- **CS / SS:** Chọn thiết bị phụ, tích cực ở mức `LOW`. Mỗi thiết bị cần một chân CS riêng.
- **Bốn chế độ SPI:** Được xác định bởi CPOL (cực tính xung) và CPHA (pha xung):
  - **Mode 0 (CPOL=0, CPHA=0):** Xung nhịp ở mức `LOW` khi rảnh; lấy mẫu ở cạnh lên và đổi dữ liệu ở cạnh xuống. MicroSD dùng chế độ này.
  - **Mode 1 (CPOL=0, CPHA=1):** Xung nhịp rảnh ở mức `LOW`. Dữ liệu được dịch chuyển ở cạnh lên và lấy mẫu ở cạnh xuống.
  - **Mode 2 (CPOL=1, CPHA=0):** Xung nhịp rảnh ở mức `HIGH`. Dữ liệu được lấy mẫu ở cạnh xuống và dịch chuyển ở cạnh lên.
  - **Mode 3 (CPOL=1, CPHA=1):** Xung nhịp rảnh ở mức `HIGH`. Dữ liệu được dịch chuyển ở cạnh xuống và lấy mẫu ở cạnh lên.

### 3. Thẻ nhớ và hệ thống tệp FAT32
- **Ghi dữ liệu:** Bộ nhớ flash NAND của MicroSD ghi theo các cung từ 512 byte. Thư viện gom các cung từ thành cụm lưu trữ.
- **Đệm RAM:** Lệnh `file.print()` hoặc `file.write()` ghi dữ liệu vào bộ đệm 512 byte của ESP32. Khi bộ đệm đầy, dữ liệu được gửi qua SPI.
- **Tầm quan trọng Cốt tử của hàm `file.flush()` và `file.close()`:**
  - Nếu nguồn sụt áp hoặc Watchdog Timer khởi động lại hệ thống trước khi bộ đệm được ghi, dữ liệu chưa lưu sẽ mất.
  - Nếu bảng phân bổ tệp FAT chưa cập nhật kích thước mới, tệp có thể bị hỏng.
  - **Quy tắc:** Gọi `file.flush()` sau mỗi chu kỳ ghi để đẩy dữ liệu trong bộ đệm xuống thẻ.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 3.1)
1. **Bus giao tiếp SPI 4 dây chuẩn kết nối thẻ nhớ MicroSD trên kit OBC bao gồm các chân tín hiệu nào?**
   - A. SDA, SCL, INT, RST
   - B. TX, RX, CTS, RTS
   - C. CS, SCK, MISO, MOSI
   - D. D0, D1, D2, CLK

2. **Khi ghi dữ liệu viễn trắc vào thẻ MicroSD, tại sao sau khi gọi `dataFile.println()` bắt buộc phải gọi `dataFile.flush()`?**
   - A. Để đẩy dữ liệu từ đệm RAM xuống thẻ nhớ vật lý ngay lập tức
   - B. Để xóa sạch toàn bộ nội dung vừa ghi
   - C. Để đóng tệp tin và giải phóng con trỏ bộ nhớ
   - D. Để chuyển đổi định dạng tệp tin từ nhị phân sang văn bản

---

---

## 📖 Doc 3.2: Giao diện dòng lệnh và hệ điều hành

### 1. Giao diện dòng lệnh
Để phục vụ việc kiểm thử, vận hành và trút dữ liệu từ trạm mặt đất, phân hệ C&DH cài đặt một bộ phân tích cú pháp dòng lệnh tương tự hệ điều hành Linux thông qua cổng nối tiếp UART (`PTIT_Storage::handleCommand`):
- **`pwd`:** In đường dẫn thư mục làm việc hiện tại (ví dụ: `/`, `/photos`, `/hieu`).
- **`cd <path>`:** Đổi thư mục làm việc hiện tại (hỗ trợ thư mục cha `..`, thư mục gốc `/`, thư mục home `~`, đường dẫn tương đối và tuyệt đối).
- **`ls [path]`:** Duyệt cây thư mục trên thẻ MicroSD, phân loại `DIR:` và `FILE:` kèm dung lượng byte.
- **`cat <path>`:** Mở tệp tin nhật ký văn bản và in toàn bộ nội dung ra màn hình.
- **`touch <path>`:** Tạo một tệp tin mới rỗng nếu tệp chưa tồn tại.
- **`echo <text> [> | >> <file>]`:** In chuỗi văn bản hoặc chuyển hướng ghi đè (`>`) / ghi nối tiếp (`>>`) vào tệp tin.
- **`mkdir <path>`:** Tạo thư mục mới trên thẻ nhớ.
- **`rmdir <path>`:** Xóa thư mục rỗng (bảo vệ không xóa thư mục gốc `/`).
- **`rm <path>`:** Xóa bỏ tệp tin dữ liệu cũ trên thẻ nhớ (từ chối xóa nếu đường dẫn là thư mục).
- **`df`:** In tổng dung lượng, dung lượng đã dùng và dung lượng còn trống (MB).
- **`help`:** Hiển thị danh sách các lệnh được hệ thống hỗ trợ.

**Nguyên tắc hoạt động:**
- Các lệnh thay đổi trạng thái (`cd`, `mkdir`, `rmdir`, `rm`, `touch`, `echo >`) không in thông báo khi thành công. Lệnh có đầu ra hoặc gặp lỗi mới hiển thị thông tin.
- Trước khi mở tệp hoặc thư mục, gọi `SD.exists()` để kiểm tra sự tồn tại và tránh lỗi VFS.

![Kiến trúc Shell CLI và bộ điều phối FreeRTOS](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab3_cli_rtos_overview.png)

### 2. Vòng lặp tuần tự và FreeRTOS
Chương trình đơn giản có thể dùng vòng lặp tuần tự. Hệ thống nhiều tác vụ cần hệ điều hành thời gian thực như FreeRTOS:

| Tiêu chí | Vòng lặp tuần tự | FreeRTOS |
| :--- | :--- | :--- |
| **Thực thi** | Chạy lần lượt các hàm trong `loop()`. | Chia thời gian CPU cho nhiều tác vụ. |
| **Đáp ứng sự kiện** | Phụ thuộc thời gian chạy các hàm trước đó. | Tác vụ ưu tiên cao có thể chiếm CPU. |
| **Bộ nhớ** | Các hàm dùng chung một ngăn xếp. | Mỗi tác vụ có ngăn xếp riêng. |
| **Độ phức tạp** | Dễ viết, khó mở rộng khi hệ thống lớn. | Cần đồng bộ tác vụ bằng Mutex, Queue hoặc Semaphore. |

### 3. Tác vụ và bộ điều phối ưu tiên
- **Tác vụ (Task):** Mỗi chức năng độc lập của vệ tinh được đóng gói thành một luồng thực thi vô tận (ví dụ: `TaskTelemetry`, `TaskSensors`, `TaskCommand`, `TaskStorage`).
- **Mức Ưu tiên (Priority):** Mỗi tác vụ được gán một con số ưu tiên (từ 0 đến $N-1$). Con số càng lớn, độ ưu tiên càng cao.
- **Bộ điều phối ưu tiên:** Mỗi nhịp hệ thống (thường là $1\text{ ms}$), bộ điều phối kiểm tra các tác vụ sẵn sàng và cấp CPU cho tác vụ có mức ưu tiên cao nhất.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 3.2)
1. **Bộ phân tích cú pháp dòng lệnh Shell CLI trên vệ tinh đóng vai trò then chốt nào trong quản trị hệ thống?**
   - A. Thay thế hoàn toàn hệ điều hành thời gian thực
   - B. Tăng tốc độ ghi dữ liệu nhị phân lên thẻ MicroSD
   - C. Tự động điều chỉnh góc quay hướng tấm pin mặt trời
   - D. Cung cấp giao diện tương tác thực thi lệnh chẩn đoán và quản lý tệp tin

2. **Kiến trúc phần mềm bay Command-Centric Architecture (C2A) vận hành theo nguyên lý nào?**
   - A. Mọi cảm biến đều tự ghi thẳng vào thẻ nhớ không qua CPU
   - B. Toàn bộ tiến trình và hành vi của vệ tinh đều quy về xử lý lệnh
   - C. Chỉ duy nhất một tác vụ chạy liên tục từ lúc phóng
   - D. Cấm sửa đổi cấu hình logic sau khi phóng lên quỹ đạo

---

---

## 📖 Doc 3.3: Tranh chấp tài nguyên và đồng bộ tác vụ

### 1. Tranh chấp tài nguyên
Khi hệ thống chạy đa tác vụ, các tài nguyên phần cứng duy nhất (như bus SPI của thẻ nhớ, cổng Serial Monitor, bus I2C) trở thành **Tài nguyên Dùng chung (Shared Resources)**:
- Nếu `TaskTelemetry` đang mở tệp và ghi dở 50 byte đầu tiên thì bị `TaskCommand` có độ ưu tiên cao hơn chiếm quyền CPU và cũng nhảy vào ghi tệp trên cùng thẻ nhớ qua bus SPI, dữ liệu của hai tác vụ sẽ bị trộn lẫn vào nhau gây lỗi vỡ khung tệp tin nghiêm trọng.

### 2. Khóa Mutex
Để bảo vệ tài nguyên dùng chung, FreeRTOS cung cấp cơ chế khóa **Mutex** (`SemaphoreHandle_t`):
- Quy tắc: "Ai giữ chìa khóa thì mới được vào phòng".
- Trước khi truy cập thẻ nhớ hoặc bus SPI, tác vụ bắt buộc phải gọi hàm xin khóa: `xSemaphoreTake(xSPIMutex, portMAX_DELAY)`.
- Sau khi hoàn thành việc ghi dữ liệu, tác vụ bắt buộc phải nhả khóa: `xSemaphoreGive(xSPIMutex)`.
- Các tác vụ khác muốn truy cập tài nguyên trong thời gian này sẽ tự động chuyển sang trạng thái chờ (Blocked), giải phóng chu kỳ CPU cho các công việc khác.

![Đảo ngược ưu tiên và cơ chế khóa Mutex](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab3_priority_inversion_mutex.png)

### 3. Nghịch đảo và kế thừa độ ưu tiên
- **Sự cố Lịch sử Mars Pathfinder (1997):** Tàu đổ bộ Sao Hỏa liên tục bị reset phần mềm bí ẩn giữa sa mạc Sao Hỏa do một lỗi thiết kế RTOS kinh điển:
  - Tác vụ ưu tiên thấp giữ khóa tài nguyên chung.
  - Tác vụ ưu tiên cao cần tài nguyên nên bị chặn lại chờ đợi.
  - Một tác vụ ưu tiên trung bình (không cần tài nguyên đó) bất ngờ thức dậy và chiếm quyền CPU suốt một thời gian dài, gián tiếp ngăn không cho tác vụ ưu tiên thấp nhả khóa, khiến tác vụ ưu tiên cao bị "bỏ đói" (Starvation) dẫn tới vi phạm deadline khẩn cấp.
- **Giải pháp Thừa kế Ưu tiên (Priority Inheritance):** Khi tác vụ ưu tiên cao bị chặn bởi một Mutex do tác vụ ưu tiên thấp nắm giữ, hệ điều hành tạm thời **nâng mức ưu tiên của tác vụ thấp lên ngang bằng với tác vụ cao**, giúp nó nhanh chóng hoàn thành công việc và nhả khóa, sau đó trả lại mức ưu tiên ban đầu.

### 4. Bế tắc
Xảy ra khi hai tác vụ cùng chờ tài nguyên mà tác vụ kia đang nắm giữ:
- `Task 1` đang giữ khóa Mutex A và muốn lấy khóa Mutex B.
- `Task 2` đang giữ khóa Mutex B và muốn lấy khóa Mutex A.
- Hậu quả: Cả hai tác vụ bị treo vĩnh viễn, kéo sập hệ điều hành.
- **Quy tắc Phòng ngừa:** Luôn luôn chiếm giữ các khóa theo một thứ tự phân cấp cố định trong toàn bộ dự án phần mềm.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 3.3)
1. **Trong hệ điều hành FreeRTOS, cơ chế Khóa Mutex (Mutual Exclusion) được áp dụng nhằm mục đích gì?**
   - A. Ngăn chặn hiện tượng tranh chấp tài nguyên dùng chung giữa các tác vụ
   - B. Tăng xung nhịp xử lý của lõi vi điều khiển
   - C. Tự động xóa các tác vụ bị lỗi khỏi danh sách chờ
   - D. Hạ điện áp cấp cho các cảm biến ngoại vi

2. **Hiện tượng nghịch đảo độ ưu tiên (Priority Inversion) trong hệ thống thời gian thực được khắc phục bằng kỹ thuật nào?**
   - A. Chuyển sang mô hình vòng lặp đơn tuần tự
   - B. Xóa bỏ hoàn toàn các tác vụ có mức ưu tiên thấp
   - C. Kế thừa độ ưu tiên (Priority Inheritance)
   - D. Giảm tần số xung nhịp của CPU

---

---

## 📖 Doc 3.4: Watchdog và cơ chế tự phục hồi

### 1. Nguyên lý Watchdog
Trong môi trường bức xạ quỹ đạo LEO, các hạt ion mang điện tích năng lượng cao có thể gây hiện tượng lật bit dữ liệu ngẫu nhiên (Single Event Upset - SEU), làm con trỏ chương trình (Program Counter) nhảy vào vùng nhớ rác khiến vi điều khiển bị treo cứng hoàn toàn:
- **Watchdog Timer (WDT)** là một mạch đếm lùi phần cứng độc lập với CPU, hoạt động nhờ nguồn xung nhịp xung nội riêng biệt.
- Khi khởi tạo, WDT được nạp một khoảng thời gian giới hạn (ví dụ: $5000\text{ ms}$). Mạch đếm liên tục giảm giá trị về $0$.
- Trong điều kiện hoạt động bình thường, phần mềm bay định kỳ thực hiện thao tác **Làm tươi Watchdog Timer (Feed Watchdog / Reset WDT)** để nạp lại giá trị ban đầu cho bộ đếm.
- Nếu phần mềm bị treo hoặc rơi vào vòng lặp vô tận, thao tác làm tươi Watchdog Timer không được thực hiện. Bộ đếm chạm mức $0$, ngay lập tức kéo chân Reset phần cứng của vi điều khiển xuống mức thấp, ép buộc toàn bộ hệ thống khởi động lại sạch sẽ.

![Máy trạng thái bộ định thời Watchdog Timer](https://raw.githubusercontent.com/PTITCube/PTITCube-Figs/main/png/lab3_watchdog_fsm.png)

### 2. Các watchdog trên ESP32
1. **Hardware Interrupt Watchdog (IWDT):** Giám sát ở cấp độ vi xử lý phần cứng cấp thấp. Nếu một ngắt (Interrupt Handler) chạy quá lâu hoặc CPU bị khóa hoàn toàn, IWDT kích hoạt.
2. **Task Watchdog Timer (TWDT):** Giám sát ở cấp độ hệ điều hành FreeRTOS. Từng tác vụ riêng lẻ phải đăng ký với TWDT (`esp_task_wdt_add(NULL)`). Nếu bất kỳ tác vụ quan trọng nào bị kẹt và không gọi hàm `esp_task_wdt_reset()` trong khoảng thời gian cấu hình, TWDT sẽ in báo cáo lỗi ngăn xếp và reset hệ thống.

---

### ❓ Câu Hỏi Củng Cố Kiến Thức (Doc 3.4)
1. **Bộ định thời giám sát Task Watchdog Timer (TWDT) trên ESP32 sẽ tự động thực hiện hành động gì nếu một tác vụ bị treo?**
   - A. Tự động ngắt kết nối WiFi và Bluetooth
   - B. Kích hoạt Reset phần cứng toàn bộ vi điều khiển
   - C. Xóa sạch dữ liệu trên thẻ nhớ ngoài
   - D. Chuyển vi điều khiển về chế độ nạp Bootloader

2. **Quy trình FDIR (Fault Detection, Isolation, and Recovery) trong thiết kế phần mềm vệ tinh là:**
   - A. Quy trình kiểm tra lỗi cơ học trước khi phóng
   - B. Thuật toán định vị quỹ đạo vệ tinh dựa trên GNSS
   - C. Giao thức nén ảnh viễn thám không gian
   - D. Cơ chế tự động phát hiện, cô lập lỗi và tái khởi động phục hồi hệ thống

---

## 📖 Doc 3.5: Hướng dẫn kết nối thẻ microSD

Tài liệu này hướng dẫn kết nối mô-đun microSD với ESP32 qua bus SPI để thực hành lưu trữ dữ liệu.

### 1. Bố trí linh kiện

> **Vị trí chèn hình:** Bố trí ESP32, mô-đun microSD và dây kết nối.

### 2. Kết nối SPI

| Mô-đun microSD | ESP32 |
| :--- | :--- |
| `MOSI` | `GPIO23` |
| `MISO` | `GPIO19` |
| `SCK` | `GPIO18` |
| `CS` | `GPIO13` |
| `VCC`, `GND` | Theo nhãn nguồn trên mô-đun và sơ đồ lắp ráp |

Ví dụ trong Lab 3 đặt chân chọn thẻ nhớ là `SD_CS_PIN 13`.

> **Vị trí chèn hình:** Sơ đồ nối từng chân SPI giữa mô-đun microSD và ESP32.

### 3. Kiểm tra trước khi thực hành

- Kiểm tra thẻ microSD đã được lắp đúng chiều.
- Đối chiếu chân `CS` với cấu hình `SD_CS_PIN` trong chương trình.

---

## 💻 Hướng dẫn thực hành Lab 3: Lưu trữ dữ liệu

### 1. Kỹ thuật Đọc/Ghi Thẻ Nhớ An Toàn với `flush()`
Đoạn mã C++ minh họa chu trình ghi dữ liệu cảm biến kèm đóng dấu thời gian và xả bộ đệm RAM an toàn:

```cpp
#include <SPI.h>
#include <SD.h>

#define SD_CS_PIN 13

void logTelemetryData(float vbat, float temp, float pressure) {
    File dataFile = SD.open("/telemetry.csv", FILE_APPEND);
    if (dataFile) {
        dataFile.printf("%lu,%.2f,%.2f,%.2f\n", millis(), vbat, temp, pressure);
        // Ép buộc ghi dữ liệu từ bộ đệm RAM 512-byte xuống bộ nhớ Flash vật lý
        dataFile.flush();
        dataFile.close();
    } else {
        Serial.println("Loi: Khong the mo tep telemetry.csv de ghi!");
    }
}
```

### 2. Danh mục Bài Thực hành Lab 3
- `Lab3-01a_MicroSD_SPI`: Thực hành khởi tạo bus SPI phần cứng ở tốc độ cao, nhận diện thẻ nhớ MicroSD và đọc thông tin dung lượng.
- `Lab3-01b_MicroSD_Library`: Ghi log dữ liệu tự động định dạng CSV thông qua đối tượng thư viện `PTIT_Storage` (`storage.init()`).
- `Lab3-02_Linux_FS_Commands`: Lập trình bộ phân tích cú pháp dòng lệnh CLI nhận 11 lệnh Linux chuẩn (`pwd`, `cd`, `ls`, `cat`, `touch`, `echo`, `mkdir`, `rmdir`, `rm`, `df`, `help`) tuân thủ nguyên tắc Silent Success, không in tiền tố `OK:` rườm rà.
- `Lab3-03_Multitasking_Scheduler`: Thiết lập 3 tác vụ FreeRTOS chạy song song (Nháy LED nhịp tim chỉ thị, Sinh dữ liệu viễn trắc giả lập có Mutex bảo vệ, Tự chẩn đoán bộ nhớ Heap và xung nhịp CPU) được đồng bộ an toàn qua `PTIT_Scheduler`.
- `Lab3-04_Watchdog_Timer`: Kích hoạt Task Watchdog Timer (TWDT), nạp lại định kỳ, mô phỏng lỗi treo vi xử lý qua Nút nhấn phần cứng hoặc chuỗi lệnh Serial (`fault`/`freeze`), ghi nhận nguyên nhân khởi động lại bằng `esp_reset_reason()`.
- `Lab3-05_Event_Driven_Queue`: Xây dựng kiến trúc hướng sự kiện (Event-Driven) với hàng đợi FreeRTOS Queue không khóa, tiếp nhận sự kiện ngắt nút nhấn phần cứng, cảnh báo viễn trắc nhiệt độ vượt ngưỡng và lệnh điều khiển chuỗi Serial mà không cần vòng lặp thăm dò (Polling).

### ❓ Câu Hỏi Đánh Giá Kỹ Năng Thực Hành (Lab 3)
1. **Kể từ Lab 3, mọi thao tác xuất log debug ra màn hình Console được chuẩn hóa qua đối tượng nào?**
   - A. Đối tượng `Serial` mặc định của Arduino core
   - B. Đối tượng `Wire` qua bus giao tiếp I2C
   - C. Đối tượng `cdh` (`PTIT_CDH cdh;`) kế thừa từ lớp Stream/Print
   - D. Đối tượng `File` ghi trực tiếp vào thẻ nhớ

2. **Chế độ mở tệp nào sau đây trong C++ được sử dụng để ghi nối tiếp dữ liệu viễn trắc mà không làm mất nội dung cũ?**
   - A. `FILE_APPEND`
   - B. `FILE_WRITE`
   - C. `FILE_READ`
   - D. `FILE_TRUNC`
