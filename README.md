# INT4211 - LẬP TRÌNH TRÊN CÁC THIẾT BỊ DI ĐỘNG
## BÁO CÁO THỰC HÀNH LAB A2: XỬ LÝ SỰ KIỆN VÒNG ĐỜI, LƯU TRẠNG THÁI & GIT/GITHUB

---

### THÔNG TIN SINH VIÊN
* **Họ và tên:** Đỗ Lê Tuấn Đạt
* **Mã số sinh viên (MSSV):** 231A290089
* **Lớp học phần:** Lập trình thiết bị di động (INT4211)
* **Môi trường thực thi:** Android Studio trên macOS (Apple Silicon)
* **Thiết bị chạy thử nghiệm:** Máy ảo Pixel 8 (Android 17, API 37.1 – aarch64)
* **Chế độ Repository:** Private (Đã mời giảng viên làm Collaborator)

---

### CÁC MỤC ĐÃ HOÀN THÀNH

#### 1. Logic Đồng hồ bấm giờ & Cập nhật UI bằng Handler (Phần 1, 2, 3)
* **Thuật toán mốc thời gian:** Sử dụng `SystemClock.elapsedRealtime()` đo khoảng cách mili-giây độc lập với đồng hồ hệ thống, không dùng vòng lặp `while/sleep` trên Main Thread để tránh lỗi ANR.
* **Cập nhật định kỳ 100ms:** Sử dụng `Handler(Looper.getMainLooper())` kết hợp `Runnable ticker` gọi đệ quy `handler.postDelayed(this, 100)`.
* **Phòng chống lỗi:** Gọi `handler.removeCallbacks(ticker)` trong `startTicking()` để tránh chạy chồng ticker làm sai lệch tốc độ đếm, và gọi trong `onDestroy()` để chống rò rỉ bộ nhớ (Memory Leak).
* **Giao diện chuẩn:** Áp dụng phông chữ `monospace` cho TextView hiển thị thời gian để các chữ số không bị nhảy rung khi thay đổi.

#### 2. Xử lý Vòng đời & Lưu/Khôi phục trạng thái (Phần 3, 4)
* **Tiết kiệm tài nguyên:** Ngắt cập nhật giao diện trong `onPause()` bằng `stopTicking()` để tiết kiệm CPU/pin khi ứng dụng bị che khuất; tự động bật lại cập nhật trong `onResume()`.
* **Lưu trạng thái UI:** Override phương thức `onSaveInstanceState(Bundle outState)` đóng gói 4 biến trạng thái cốt lõi:
  * `running` (boolean): Trạng thái đang chạy hay tạm dừng.
  * `accumulated` (long): Thời gian đã tích lũy từ các phiên trước.
  * `start` (long): Mốc `elapsedRealtime()` khi bắt đầu lượt chạy hiện tại.
  * `recreate` (int): Số lần Activity bị hủy và tạo lại.
* **Khôi phục hoàn hảo:** Đọc dữ liệu từ `savedInstanceState` trong `onCreate()` sau khi xoay màn hình (Configuration Change).

#### 3. Kiểm chứng 5 Kịch bản thí nghiệm vòng đời (Phần 4)
* **Kịch bản 1 (Xoay màn hình):** Đồng hồ chạy liên tục không về 0, số lần tái tạo tăng thêm 1.
* **Kịch bản 2 (Bấm Home 10s rồi quay lại):** Thời gian bù đủ ~10 giây đã trôi qua ở nền nhờ tính mốc `SystemClock.elapsedRealtime()`.
* **Kịch bản 3 (Tạm dừng rồi xoay màn hình):** Giữ nguyên trạng thái dừng và con số thời gian.
* **Kịch bản 4 (Don't keep activities):** Activity bị tiêu diệt hoàn toàn khi ra nền nhưng trạng thái được khôi phục trọn vẹn nhờ `onSaveInstanceState`.
* **Kịch bản 5 (Thoát bằng phím Back):** Hệ thống không gọi `onSaveInstanceState`, mở lại app đưa về `00:00.0` đúng chủ đích người dùng.

#### 4. Bài nâng cao NC1: Nút Vòng (Lap)
* Thêm nút Lap ghi nhận mốc thời gian tức thời vào danh sách `ArrayList<String> lapList`.
* Hiển thị danh sách cuộn mượt mà bằng `TextView` đặt trong `ScrollView`.
* Lưu danh sách qua `outState.putStringArrayList("laps", lapList)` giúp dữ liệu không bị mất khi xoay màn hình.

#### 5. Bài nâng cao NC2: CheckBox "Dừng khi ra nền"
* Thêm `CheckBox cbStopOnBackground` trên giao diện.
* Kiểm tra trong callback vòng đời `onStop()`: Nếu CheckBox được tích chọn và đồng hồ đang chạy thì tự động kích hoạt `pauseStopwatch()`.
* Lưu trạng thái bật/tắt của CheckBox vào Bundle để duy trì sau khi tái tạo.

---

### CẤU TRÚC THƯ MỤC CHÍNH
```text
A2/
├── app/
│   ├── src/main/
│   │   ├── AndroidManifest.xml
│   │   ├── java/vn/edu/vhu/ltdd/a2stopwatch/
│   │   │   └── MainActivity.java           # Toàn bộ logic vòng đời, Handler, NC1, NC2
│   │   └── res/
│   │       ├── layout/
│   │       │   └── activity_main.xml       # Giao diện đồng hồ + CheckBox NC2 + ScrollView NC1
│   │       └── values/
│   │           └── strings.xml             # Khai báo toàn bộ chuỗi đa ngôn ngữ
│   ├── .gitignore
│   └── build.gradle.kts
├── .gitignore
├── build.gradle.kts
└── README.md                               # Báo cáo tổng quan dự án