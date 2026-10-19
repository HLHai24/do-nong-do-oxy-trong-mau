# Hệ thống đo nồng độ oxy trong máu (SpO2) với ESP32 + MAX30102 + Blynk

Dự án đo SpO2 bằng cảm biến **MAX30102**, hiển thị kết quả trên màn hình **OLED SSD1306** và gửi dữ liệu lên **Blynk IoT** để theo dõi từ xa. Hệ thống có thể bật/tắt bằng **nút nhấn vật lý** hoặc **công tắc trên Blynk**, hai bên luôn đồng bộ trạng thái với nhau.

> ⚠️ **Lưu ý:** Đây là dự án học tập/nghiên cứu, **không phải thiết bị y tế**. Không dùng kết quả để chẩn đoán hay đưa ra quyết định điều trị.

## Tính năng

- Đo SpO2 bằng cảm biến MAX30102 (thuật toán `spo2_algorithm` của SparkFun/Maxim).
- Cửa sổ trượt: dùng 100 mẫu (~4 giây), cập nhật kết quả mới mỗi 25 mẫu (~1 giây).
- Tự phát hiện có/không có ngón tay (ngưỡng IR `50000`).
- Chỉ chấp nhận kết quả hợp lệ trong khoảng 70–100 %.
- Hiển thị trên OLED 128x64 và in ra Serial Monitor.
- Gửi SpO2 lên Blynk (Virtual Pin `V0`).
- Bật/tắt hệ thống từ Blynk (`V8`) hoặc nút nhấn vật lý (có chống dội phím 50 ms).
- Khi tắt: cảm biến vào chế độ shutdown để tiết kiệm điện, ESP32 vẫn giữ kết nối Blynk.

## Phần cứng

| Linh kiện | Số lượng |
|---|---|
| ESP32 DevKit | 1 |
| Cảm biến MAX30102 | 1 |
| Màn hình OLED SSD1306 128x64 (I2C) | 1 |
| Nút nhấn | 1 |
| Dây nối, breadboard | - |

## Sơ đồ nối dây

MAX30102 và OLED dùng chung bus I2C.

| Thiết bị | Chân thiết bị | Chân ESP32 |
|---|---|---|
| MAX30102 | VIN / VCC | 3.3V |
| MAX30102 | GND | GND |
| MAX30102 | SDA | GPIO 21 |
| MAX30102 | SCL | GPIO 22 |
| OLED SSD1306 | VCC | 3.3V |
| OLED SSD1306 | GND | GND |
| OLED SSD1306 | SDA | GPIO 21 |
| OLED SSD1306 | SCL | GPIO 22 |
| Nút nhấn | Một chân | GPIO 14 |
| Nút nhấn | Chân còn lại | GND |

- Địa chỉ I2C của OLED: `0x3C`.
- Nút nhấn dùng `INPUT_PULLUP` nên **không cần điện trở ngoài**, nhấn xuống là mức `LOW`.

## Thư viện cần cài

Cài qua **Arduino IDE → Library Manager**:

- `Blynk` (by Volodymyr Shymanskyy)
- `SparkFun MAX3010x Pulse and Proximity Sensor Library`
- `Adafruit GFX Library`
- `Adafruit SSD1306`

Board: **ESP32** (cài *esp32 by Espressif Systems* trong Boards Manager).

## Cấu hình Blynk

1. Tạo **Template** trên [Blynk Console](https://blynk.cloud).
2. Tạo các Datastream:

   | Virtual Pin | Kiểu | Mục đích |
   |---|---|---|
   | `V0` | Integer (0–100) | Giá trị SpO2 |
   | `V8` | Integer (0/1) | Công tắc bật/tắt hệ thống |

3. Trong Web Dashboard, thêm widget **Gauge/Chart** cho `V0` và **Switch** cho `V8`.
4. Tạo Device từ template để lấy `BLYNK_TEMPLATE_ID`, `BLYNK_TEMPLATE_NAME`, `BLYNK_AUTH_TOKEN`.

## Cấu hình thông tin kết nối

Thay các giá trị ở đầu file `.ino`:

```cpp
#define BLYNK_TEMPLATE_ID   "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "YOUR_TEMPLATE_NAME"
#define BLYNK_AUTH_TOKEN    "YOUR_AUTH_TOKEN"

char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";
```

> 🔒 **Không commit token và mật khẩu WiFi lên GitHub.** Nên tách chúng ra file `secrets.h`, thêm file này vào `.gitignore`, và chỉ commit file mẫu `secrets.example.h`.

## Cách sử dụng

1. Nối dây theo sơ đồ, nạp code cho ESP32.
2. Mở Serial Monitor (baud `115200`).
3. Chờ ESP32 kết nối WiFi và Blynk. OLED hiện `Dat ngon tay len MAX30102`.
4. Đặt ngón tay lên cảm biến và giữ yên. OLED hiện `Dang do...`.
5. Sau khoảng 4 giây, SpO2 hiển thị trên OLED, Serial Monitor và Blynk, rồi cập nhật mỗi giây.
6. Nhấn nút (hoặc gạt Switch `V8` trên Blynk) để tắt/bật hệ thống.

### Các trạng thái hiển thị trên OLED

| Trạng thái | Nội dung |
|---|---|
| Chưa có ngón tay | `DO NONG DO OXY – Dat ngon tay len MAX30102` |
| Đang đo | `HE THONG DO SpO2 – Dang do...` |
| Có kết quả | `SpO2: 98%` (hiện `--` nếu kết quả không hợp lệ) |
| Hệ thống tắt | `HE THONG DA TAT` |
| Lỗi cảm biến | `LOI MAX30102!` kèm hướng dẫn kiểm tra dây |

## Thông số kỹ thuật chính

| Tham số | Giá trị |
|---|---|
| Cấu hình cảm biến | `setup(60, 4, 2, 100, 411, 4096)` |
| Tốc độ lấy mẫu thực tế | 25 sps (sampleRate 100 / sampleAverage 4) |
| Kích thước bộ đệm | 100 mẫu (~4 s) |
| Bước cập nhật | 25 mẫu (~1 s) |
| Ngưỡng phát hiện ngón tay | IR ≥ 50000 |
| Dòng LED Red / IR | `0x2A` |
| Khoảng SpO2 hợp lệ | 70–100 % |

## Xử lý sự cố

- **`LOI: Khong tim thay OLED!`**: kiểm tra dây SDA/SCL, nguồn 3.3V, địa chỉ I2C (thử `0x3D` nếu `0x3C` không nhận).
- **`LOI MAX30102!`**: kiểm tra lại dây SDA (GPIO21), SCL (GPIO22), VCC (3.3V), GND.
- **Luôn hiện `--`**: giữ ngón tay yên, không ấn quá mạnh, tránh ánh sáng mạnh chiếu trực tiếp; có thể thử chỉnh `FINGER_THRESHOLD` hoặc cường độ LED.
- **Không lên Blynk**: kiểm tra tên/mật khẩu WiFi (ESP32 chỉ hỗ trợ 2.4 GHz) và thông tin Template/Auth Token.

## Cấu trúc dự án

```
.
├── oxy_spo2.ino        # Mã nguồn chính
├── secrets.example.h   # File mẫu cấu hình (tùy chọn)
├── .gitignore
└── README.md
```

## Giấy phép

Phát hành theo giấy phép [MIT](LICENSE) (thay đổi nếu bạn dùng giấy phép khác).

## Tác giả

**LeHai**
