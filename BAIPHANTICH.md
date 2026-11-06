# K26CNTT2_IT108_Session19_Ex03
BTVN
# 🛡️ Báo Cáo Phân Tích: Quản Lý Rủi Ro Khi Mất Thiết Bị Xác Thực (AWS MFA Recovery)

---

## 📌 Thông Tin Bài Nộp
* **Họ và tên:** Lê Hồ Thanh Thảo
* **MSSV:** N26DTCN030

---

## 1. Phân Tích Hệ Thống Theo Mô Hình IPO (Input - Process - Output)

### 1.1. Bối cảnh sự cố
Lead Developer bị mất điện thoại di động – thiết bị duy nhất cài đặt ứng dụng Google Authenticator chứa mã xác thực 2 bước (MFA) để truy cập vào tài khoản quản trị AWS. Hệ thống đang cần bảo trì gấp nhưng tài khoản bị khóa bởi lớp bảo mật MFA.

### 1.2. Mô hình IPO cho quy trình phục hồi truy cập
* **INPUT (Đầu vào):**
  * Một mã dự phòng (**Backup Code**) trong tập mã đã được lưu trữ an toàn trước đó.
  * Mật khẩu đăng nhập chính (Master/Account Password) của tài khoản AWS.

* **PROCESS (Tiến trình xử lý khẩn cấp):**
  1. **Đăng nhập khẩn cấp:** Truy cập trang đăng nhập AWS, nhập Mật khẩu $\rightarrow$ Chọn *Troubleshoot MFA* $\rightarrow$ Chọn *Use Backup Code* và nhập 1 mã dự phòng khả dụng.
  2. **Vô hiệu hóa MFA cũ (Revoke Old MFA):** Ngay sau khi vào được giao diện AWS Console, truy cập mục **IAM / Security Credentials** để gỡ bỏ (Deactivate/Remove) thiết bị Google Authenticator trên chiếc điện thoại đã mất.
  3. **Đăng xuất tất cả các phiên:** Thực hiện hủy bỏ toàn bộ Session Tokens, API Access Keys cũ nhằm ngăn ngừa kẻ nhặt điện thoại tìm cách bẻ khóa thiết bị và thâm nhập vào hệ thống.
  4. **Tái thiết lập MFA & Cấp mã mới:** Liên kết Google Authenticator trên điện thoại mới (hoặc thiết bị hardware token như YubiKey), đồng thời sinh tập mã **Backup Codes mới** và hủy hoàn toàn bộ mã cũ.

* **OUTPUT (Kết quả đầu ra):**
  * Quyền truy cập quản trị hệ thống AWS được khôi phục thành công.
  * Thiết bị bị mất bị vô hiệu hóa hoàn toàn khả năng truy cập vào tài khoản.
  * Hệ thống quay trở lại trạng thái an toàn tuyệt đối với thiết bị MFA mới.

---

## 2. Phân Tích Kỹ Thuật: Tại Sao KHÔNG Được Lưu Backup Codes Dạng Ảnh Trong Điện Thoại?

Lưu mã dự phòng dưới dạng hình ảnh (chụp màn hình/ảnh tải về) trong thư viện ảnh (Gallery/Camera Roll) điện thoại tạo ra các lỗ hổng bảo mật nghiêm trọng:

### 2.1. Rủi ro khi mất thiết bị vật lý
Nếu kẻ nhặt được điện thoại bẻ khóa được màn hình (hoặc truy xuất dữ liệu qua cổng USB/Recovery), họ chỉ cần mở ứng dụng **Photos**, dùng tính năng nhận diện chữ (**OCR**) tìm kiếm với từ khóa *"backup"*, *"AWS"*, *"code"* là sẽ lấy được ngay mã dự phòng. Lúc này kẻ xấu nắm trong tay cả app OTP lẫn mã dự phòng, hoàn toàn chiếm đoạt được tài khoản.

### 2.2. Rủi ro đồng bộ Đám mây (Cloud Syncing Vulnerability)
Thư viện ảnh trên điện thoại thường tự động đồng bộ lên các dịch vụ lưu trữ đám mây (Google Photos, iCloud, OneDrive). Nếu tài khoản Cloud bị rò rỉ mật khẩu hoặc bị tấn công **Credential Stuffing**, toàn bộ kho ảnh chứa mã dự phòng sẽ bị tải về máy chủ kẻ tấn công.

### 2.3. Rủi ro từ phần mềm độc hại (Malware & App Permissions)
Nhiều ứng dụng độc hại hoặc ứng dụng chứa mã độc **Infostealer** được cài trên điện thoại thường yêu cầu quyền truy cập thư viện ảnh (`READ_EXTERNAL_STORAGE`). Mã độc có thể âm thầm quét ảnh trong máy, trộm mã Backup Code và gửi về máy chủ điều khiển (C2 Server) của hacker.

---

## 3. Khuyến Nghị Phương Pháp Lưu Trữ Backup Codes An Toàn

1. **Trình quản lý mật khẩu (Password Manager):** Lưu mã dự phòng dưới dạng ghi chú mã hóa trong Vault của Bitwarden, 1Password hoặc KeePass (được bảo vệ bằng thuật toán mã hóa cấp quân sự **AES-256**).
2. **Lưu trữ Offline vật lý (Physical Paper Storage):** In tập mã dự phòng ra giấy và cất giữ trong két an toàn hoặc hộp khóa cá nhân tại nhà/văn phòng.
3. **Mã hóa file tập tin (Encrypted Container):** Lưu mã trong file văn bản được nén mã hóa mật khẩu mạnh bằng 7-Zip (AES-256) hoặc lưu trong ổ đĩa mã hóa VeraCrypt.
