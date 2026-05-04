# 8.1 Danh mục Kiểm tra Vận hành Hàng ngày (Daily Operational Checklist)

## Mô tả

Danh mục Kiểm tra Vận hành Hàng ngày giúp các thành viên thủy thủ đoàn đảm bảo hệ thống HiCAMS hoạt động bình thường trước và trong mỗi ca trực. Bằng cách thực hiện các bước kiểm tra nhanh này, bạn có thể phát hiện và xử lý sự cố sớm — giúp hệ thống giám sát CCTV và nhận diện an toàn bằng AI luôn hoạt động ổn định.

> **Thời điểm thực hiện:** Hoàn thành danh mục kiểm tra này vào **đầu mỗi ca trực** và thêm một lần giữa ca nếu có thể.

---

## Danh mục Kiểm tra

### 1. Trạng thái Camera

Mở màn hình **Viewer** (màn hình mặc định sau khi đăng nhập). Quan sát toàn bộ lưới camera và kiểm tra các mục sau:

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 1.1 | Xác nhận tất cả camera đang **hoạt động** | Mỗi khung camera phải hiển thị hình ảnh video trực tiếp. | ☐ Đạt |
| 1.2 | Kiểm tra camera **"Disconnected"** (Mất kết nối) | Tìm bất kỳ khung camera nào hiển thị màn hình tối với dòng chữ **"Disconnected"**. | ☐ Đạt |
| 1.3 | Xác định **nguyên nhân mất kết nối** (nếu có) | Di chuột qua **biểu tượng thông tin ("i")** trên khung camera bị ngắt kết nối. Đọc tooltip — sẽ hiển thị loại lỗi và thời gian xảy ra. | ☐ Đạt |
| 1.4 | Xác nhận video trực tiếp **không bị đóng băng** | Xác nhận mỗi luồng camera đang phát trực tiếp (quan sát chuyển động hoặc thời gian đang thay đổi). | ☐ Đạt |
| 1.5 | Kiểm tra camera đang **"Loading"** hoặc **"Synchronizing"** | Nếu khung camera nào hiển thị biểu tượng xoay với dòng chữ "Loading" hoặc "Synchronizing", hãy chờ một lát. Nếu kéo dài hơn 1 phút, báo cáo sự cố. | ☐ Đạt |

**Các loại lỗi ngắt kết nối và ý nghĩa:**

| Loại lỗi | Ý nghĩa | Hành động |
|-----------|---------|-----------|
| IP error | Camera không thể truy cập qua mạng | Báo cho IT / Admin |
| Account info error | Thông tin đăng nhập camera (ID/Mật khẩu) không đúng | Báo cho Admin |
| Profile path error | Cấu hình luồng camera chưa được thiết lập đúng | Báo cho Admin |

> **Kết quả:** Tất cả camera phải hiển thị video trực tiếp. Nếu camera nào bị ngắt kết nối, ghi lại tên camera, loại lỗi và thời gian, sau đó báo cáo cho Admin hoặc Manager.

---

### 2. Trạng thái Giám sát AI

Trên màn hình **Viewer**, kiểm tra bảng điều khiển bên trái:

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 2.1 | Xác nhận công tắc **"AI Surveillance"** đang **BẬT** | Nút bật/tắt trên bảng điều khiển bên trái phải ở trạng thái BẬT. | ☐ Đạt |
| 2.2 | Xác nhận điểm **General Severity (GS)** đang **hiển thị** | Mỗi khung camera phải hiển thị mức độ nghiêm trọng (Low / Medium / High). Nếu điểm số bị ẩn, có thể AI Surveillance đã bị tắt. | ☐ Đạt |
| 2.3 | Xác nhận **khung nhận diện** đang hoạt động | Quan sát các luồng camera trong vài phút. Khi có người hoặc đối tượng xuất hiện, bạn sẽ thấy các khung viền màu (detection frames) bao quanh đối tượng được phát hiện. | ☐ Đạt |

> **Kết quả:** Công tắc AI Surveillance phải ở trạng thái BẬT. Nếu đang tắt, nhấp vào công tắc để bật. Nếu khung nhận diện không xuất hiện dù có đối tượng trước camera, báo cáo cho Admin.

---

### 3. Phát hiện Sự kiện

Chuyển đến màn hình **Event Log Explorer** bằng cách nhấp vào **"Event Log Explorer"** trên thanh điều hướng phía trên.

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 3.1 | Xác nhận **sự kiện gần đây đang được ghi nhận** | Kiểm tra danh sách Event Log. Xác nhận các sự kiện gần đây xuất hiện với ngày hôm nay (hoặc trong vài giờ qua). | ☐ Đạt |
| 3.2 | Xem lại **5–10 sự kiện mới nhất** | Nhấp vào từng sự kiện gần đây trong danh sách. Trong bảng chi tiết, xác nhận tab **Capture** hiển thị hình ảnh rõ ràng với khung nhận diện. | ☐ Đạt |
| 3.3 | Kiểm tra **các mẫu bất thường** | Tìm xem có sự gia tăng đột biến về sự kiện, hoặc không có sự kiện nào trong nhiều giờ liên tục khi camera vẫn đang hoạt động. | ☐ Đạt |
| 3.4 | Xác nhận **loại sự kiện chính xác** | Xác nhận biểu tượng và mô tả loại sự kiện khớp với nội dung hiển thị trong ảnh chụp. | ☐ Đạt |

> **Kết quả:** Sự kiện phải được ghi nhận đều đặn trong giờ hoạt động. Nếu không có sự kiện nào được ghi nhận trong thời gian dài dù camera vẫn hoạt động, báo cáo cho Admin.

---

### 4. Hệ thống Cảnh báo & Báo động

Quay lại màn hình **Viewer** và xác nhận hệ thống cảnh báo đang hoạt động:

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 4.1 | Kiểm tra **chuông báo động đang hoạt động** | Tìm biểu tượng **Chuông báo động (Alarm bell)** màu đỏ trên các khung camera. Biểu tượng này cho biết có sự kiện chưa được xử lý. | ☐ Đạt |
| 4.2 | Xác nhận **âm thanh cảnh báo** có thể nghe được | Đảm bảo âm lượng máy tính hoặc trạm làm việc đang bật. Khi có sự kiện cảnh báo, âm thanh cảnh báo phải phát ra. | ☐ Đạt |
| 4.3 | Xác nhận các **cảnh báo còn tồn đọng** | Nếu có chuông báo động từ ca trước, di chuột qua biểu tượng chuông cho đến khi hiện nút "Turn off". Nhấp để xác nhận và xử lý sự kiện. | ☐ Đạt |
| 4.4 | Xác nhận **Event Popup** đang hoạt động | Nếu đã được cấu hình, cửa sổ popup thông tin kênh sẽ tự động xuất hiện khi phát hiện sự kiện rủi ro cao. Xác nhận tính năng này đang hoạt động. | ☐ Đạt |

> **Kết quả:** Cảnh báo phải duy trì hoạt động cho đến khi được xác nhận thủ công. Xử lý tất cả cảnh báo tồn đọng từ ca trước. Nếu không nghe thấy âm thanh cảnh báo khi có sự kiện, kiểm tra cài đặt âm thanh và báo cáo cho Admin.

---

### 5. Lưu trữ Video

Chuyển đến màn hình **Video Storage** bằng cách nhấp vào **"Video Storage"** trên thanh điều hướng phía trên.

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 5.1 | Xác nhận **video đang được ghi hình** | Chọn loại video "Auto" và nhấp "Apply". Xác nhận các bản ghi video gần đây xuất hiện trong lưới với ngày hôm nay. | ☐ Đạt |
| 5.2 | Xác nhận **video sự kiện có biểu tượng** | Kiểm tra hình thu nhỏ video — video ghi nhận sự kiện phải hiển thị biểu tượng sự kiện tương ứng ở góc trên bên trái. | ☐ Đạt |
| 5.3 | Kiểm tra nhanh **phát lại video** | Nhấp vào một thẻ video gần đây. Xác nhận video phát lại bình thường với hình ảnh và âm thanh (nếu có). | ☐ Đạt |
| 5.4 | Xác nhận **tải xuống hoạt động** | Từ màn hình chi tiết video, nhấp nút **Download**. Xác nhận file MP4 được tải về thành công. | ☐ Đạt |

> **Kết quả:** Các bản ghi gần đây phải xuất hiện. Nếu không có video nào hoặc phát lại bị lỗi, báo cáo cho Admin.

---

### 6. Mạng / Kết nối

Thực hiện kiểm tra kết nối chung:

| # | Kiểm tra | Cần quan sát | Trạng thái |
|---|----------|--------------|------------|
| 6.1 | Xác nhận **hệ thống phản hồi tốt** | Chuyển đổi giữa các màn hình (Viewer → Event Log Explorer → Video Storage). Xác nhận các trang tải mà không bị chậm quá mức. | ☐ Đạt |
| 6.2 | Kiểm tra **độ trễ** của luồng video trực tiếp | Trên màn hình Viewer, quan sát mốc thời gian trên camera. Chúng phải gần với thời gian thực (chênh lệch trong vài giây). | ☐ Đạt |
| 6.3 | Xác nhận **không có mất kết nối hàng loạt** | Nếu nhiều camera đồng thời hiển thị "Disconnected", đây có thể là sự cố mạng toàn hệ thống chứ không phải lỗi từng camera riêng lẻ. | ☐ Đạt |

> **Kết quả:** Hệ thống phải phản hồi nhanh với độ trễ tối thiểu. Nếu nhiều camera mất kết nối cùng lúc hoặc hệ thống không phản hồi, báo cáo sự cố mạng cho bộ phận IT ngay lập tức.

---

## Tổng kết

| Hạng mục kiểm tra | Màn hình sử dụng | Hành động chính |
|--------------------|------------------|-----------------|
| Trạng thái Camera | Viewer | Quét lưới camera, tìm camera "Disconnected" |
| Giám sát AI | Viewer (bảng bên trái) | Xác nhận công tắc AI Surveillance đang BẬT |
| Phát hiện Sự kiện | Event Log Explorer | Xem lại nhật ký sự kiện gần đây |
| Cảnh báo & Báo động | Viewer | Xác nhận và xử lý chuông báo động tồn đọng |
| Lưu trữ Video | Video Storage | Xác nhận có bản ghi video gần đây |
| Mạng / Kết nối | Tất cả các màn hình | Chuyển đổi giữa các màn hình, kiểm tra phản hồi |

---

## Báo cáo Sự cố

Nếu bất kỳ mục kiểm tra nào không đạt, ghi lại các thông tin sau và báo cáo cho **Admin** hoặc **Manager**:

1. **Ngày và giờ** xảy ra sự cố
2. **Tên màn hình** nơi phát hiện sự cố (ví dụ: Viewer, Event Log Explorer)
3. **Tên camera** (nếu có)
4. **Thông báo lỗi hoặc mô tả** (ví dụ: "Disconnected — IP error — 2026-04-07 08:15")
5. **Tên tài khoản của bạn**

> **Mẹo:** Chụp ảnh màn hình sự cố bất cứ khi nào có thể để giúp Admin chẩn đoán vấn đề nhanh hơn.
