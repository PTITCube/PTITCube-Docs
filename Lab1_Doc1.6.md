# Lab 1: Doc 1.6 - Định thời bằng `millis()` và xử lý tràn số

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

Xem [hướng dẫn lắp ráp bo thí nghiệm Lab 1](./Lab1_Doc1.7.md) trước khi thực hành.

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
