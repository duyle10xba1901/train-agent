# 8.2 Xử lý Lỗi (Error Handling): Phản hồi cơ bản khi Camera gặp sự cố

## Mô tả

Hướng dẫn này trình bày các bước đầu tiên bạn nên thực hiện khi một luồng camera bị mất kết nối hoặc báo lỗi. Việc làm theo các bước này giúp xác định xem đây là sự cố mạng tạm thời hay lỗi phần cứng, cho phép bạn nhanh chóng khôi phục hệ thống giám sát hoặc báo cáo sự cố một cách chính xác.

---

## Các Bước Xử lý Cơ bản

Nếu bạn phát hiện sự cố về camera (ví dụ: khung video chuyển sang màu đen và hiển thị chữ "Disconnected"), vui lòng thực hiện theo các bước sau:

### 1. Kiểm tra Trạng thái Camera
Xác minh thông báo lỗi cụ thể trên khung camera để hiểu nguyên nhân gốc rễ.
- **Xác định camera mất kết nối**: Tìm khung màn hình tối hiển thị dòng chữ **"Disconnected"** (Mất kết nối) cùng với tên camera.
- **Kiểm tra thông tin lỗi**: Di chuột qua **biểu tượng thông tin ("i")** ở góc của khung camera bị mất kết nối. Đọc nội dung tooltip để xác định chính xác lỗi:
  - `IP error`: Không thể ping (truy cập) được IP của camera trên mạng.
  - `Account info error`: Tài khoản (ID) hoặc mật khẩu của camera bị sai.
  - `Profile path error`: Cấu hình profile luồng video của camera chưa chính xác.
- **Kiểm tra các camera xung quanh**: Quan sát các luồng camera khác trên lưới. Nếu chỉ có một camera bị mất kết nối, đây là sự cố riêng lẻ. Nếu nhiều hoặc tất cả camera đều mất kết nối cùng lúc, đây rất có thể là sự cố mạng chung.

### 2. Xác minh Kết nối Mạng
Đảm bảo hệ thống và camera vẫn đang giao tiếp bình thường.
- **Kiểm tra phản hồi của hệ thống**: Xác nhận rằng bạn vẫn có thể chuyển đổi giữa các màn hình khác nhau (ví dụ: Viewer và Event Log Explorer). Nếu toàn bộ giao diện HiCAMS không phản hồi, hãy kiểm tra kết nối mạng của máy tính làm việc.
- **Kiểm tra kết nối (Dành cho Admin/Manager)**: Truy cập **Set up > Channel management**. Chọn camera đang gặp sự cố và nhấp vào nút **"Check"** để buộc hệ thống chủ động kiểm tra lại nguồn cấp, IP và giao thức.

### 3. Tải lại Màn hình (Refresh)
Đôi khi sự cố rớt mạng tạm thời khiến luồng video bị đóng băng hoặc ngắt kết nối.
- **Tải lại màn hình giám sát**: Nhấp lại vào tab **Viewer** hoặc làm mới (refresh) trình duyệt web của bạn để buộc hệ thống thử kết nối lại.
- **Quan sát trạng thái kết nối lại**: Sau khi tải lại, hãy theo dõi khung camera đó.
  - Tìm biểu tượng đang tải **"Loading"**.
  - Tìm dòng trạng thái đồng bộ **"Synchronizing"**.
- **Xác nhận khôi phục**: Chờ xem video trực tiếp có hiển thị trở lại bình thường và thời gian có đồng bộ đúng với thời gian thực hay không.

### 4. Khởi động lại Camera (Nếu áp dụng)
Nếu camera vẫn bị ngắt kết nối và báo lỗi `IP error` (sự cố mạng riêng lẻ), thiết bị phần cứng có thể cần được khởi động lại.
- **Khởi động lại thiết bị**: Nếu bạn có quyền truy cập vật lý hoặc quyền truy cập mạng vào bộ cấp nguồn / switch PoE của camera, hãy khởi động lại thiết bị camera.
- **Đợi camera kết nối lại**: Đợi 2-3 phút để camera khởi động xong.
- **Giám sát hệ thống**: Quét lại màn hình HiCAMS Viewer để xem hệ thống có vượt qua các giai đoạn "Loading" và "Synchronizing" thành công hay không.

---

## Báo cáo Các Sự cố Kéo dài

Nếu bạn đã hoàn thành tất cả các bước trên mà camera vẫn báo **"Disconnected"**, hãy báo cáo sự cố này cho bộ phận IT hoặc Quản trị hệ thống (Admin) ngay lập tức.

Cung cấp cho họ các thông tin bạn đã thu thập được từ Bước 1:
- **Tên Camera**
- Loại **Lỗi chính xác** (IP error, Account info error, hoặc Profile path error)
- **Thời điểm** camera bị mất kết nối (hiển thị trong tooltip)
- Cung cấp thông tin xem có các camera khác cũng bị ảnh hưởng hay không.
