# 6. Xem lại & Lưu trữ Video (Video Playback & Archiving)

Hệ thống HiCAMS tự động lưu trữ các đoạn video từ camera CCTV khi có sự kiện an toàn được AI phát hiện. Bạn có thể tìm kiếm, xem lại và tải về các đoạn video này bất cứ lúc nào thông qua màn hình **Video Storage**.

> **Lưu ý về quyền truy cập:**
>
> - **Admin** và **Manager**: Xem được video từ tất cả các camera.
> - **User** (người dùng thường): Chỉ xem được video từ các camera đã được gán cho mình.

---

## 6.1 Tìm kiếm Video (Video Search)

Màn hình **Video Storage** cho phép bạn tìm kiếm và lọc các bản ghi video đã lưu. Dưới đây là hướng dẫn chi tiết từng bước.

### Bước 1: Chọn loại Video

Ở đầu bảng lọc bên trái, chọn một trong hai loại:

| Loại video                    | Mô tả                                                  |
| ----------------------------- | ------------------------------------------------------ |
| **Auto** (Tự động lưu)        | Video được hệ thống tự động lưu khi có sự kiện xảy ra. |
| **Manual** (Tải lên thủ công) | Video do người dùng tự tải lên hệ thống.               |

### Bước 2: Chọn khoảng thời gian

oảng thời g
Sử dụng bộ chọn ngày giờ (**Date Picker**) để xác định khian cần tìm:

- **Start time** (Thời gian bắt đầu): Nhấp vào biểu tượng lịch để chọn ngày và giờ bắt đầu.
- **End time** (Thời gian kết thúc): Nhấp vào biểu tượng lịch để chọn ngày và giờ kết thúc.

> **Mặc định:** Hệ thống hiển thị dữ liệu từ **1 tháng trước** cho đến **thời điểm hiện tại**.

### Bước 3: Chọn Camera (Channel)

Lọc video theo camera cụ thể:

- Nhập tên camera vào ô tìm kiếm, hoặc
- Nhấp vào menu thả xuống để chọn camera từ danh sách.

> **Mặc định:** Tất cả các camera đều được chọn.

### Bước 4: Lọc theo Sự kiện (Tùy chọn)

Bạn có thể thu hẹp kết quả tìm kiếm bằng các bộ lọc sự kiện sau:

**a) Checkbox "Show only records include events"** (Chỉ hiển thị bản ghi có sự kiện)

- Tích vào ô này để chỉ hiển thị các video có chứa sự kiện.
- Mặc định: Không tích (hiển thị tất cả video).
- _Lưu ý: Chức năng này bị vô hiệu hóa khi chọn loại video "Manual"._

**b) Lọc theo Loại sự kiện (Event Type)**

Các sự kiện được chia thành 3 nhóm chính:

| Nhóm sự kiện                              | Ví dụ                          |
| ----------------------------------------- | ------------------------------ |
| **Human** (Con người)                     | Không đội mũ bảo hộ, Ngã, v.v. |
| **Disaster/Accident** (Thiên tai/Tai nạn) | Khói, Cháy, v.v.               |
| **Traffic** (Giao thông)                  | Các sự kiện giao thông         |

- Nhấp vào **ô checkbox của nhóm** để chọn/bỏ chọn tất cả sự kiện trong nhóm đó.
- Nhấp vào **mũi tên mở rộng** để xem danh sách chi tiết các loại sự kiện.
- Nhấp vào **biểu tượng sự kiện cụ thể** để chọn/bỏ chọn từng loại.
- Di chuột qua biểu tượng sự kiện để xem mô tả (tooltip).

> _Lưu ý: Bộ lọc loại sự kiện bị vô hiệu hóa khi chọn loại video "Manual"._

### Bước 5: Nhấp "Apply" (Áp dụng)

Sau khi đã thiết lập xong các bộ lọc, nhấp nút **"Apply"** để thực hiện tìm kiếm. Danh sách video sẽ được cập nhật theo kết quả.

### Đọc hiểu kết quả tìm kiếm

Kết quả tìm kiếm hiển thị dưới dạng lưới các thẻ video (**Video capture cards**). Mỗi thẻ bao gồm:

- **Hình thu nhỏ (Thumbnail):** Đối với video có sự kiện, hình thu nhỏ sẽ là khung hình tại thời điểm sự kiện xảy ra lần đầu.
- **Biểu tượng sự kiện:** Hiển thị ở góc trên bên trái của hình thu nhỏ, cho biết loại sự kiện đã xảy ra (chỉ áp dụng cho video có sự kiện).
- **Thời lượng video:** Hiển thị ở góc dưới bên phải của hình thu nhỏ.
- **Thông tin video:** Hiển thị bên dưới hình thu nhỏ gồm: Tên camera, mô tả camera, ngày giờ ghi hình.

**Sắp xếp kết quả:**

- Nhấp vào menu **"Sort by"** ở góc trên bên phải danh sách.
- Chọn **"Latest"** (Mới nhất trước) hoặc **"Oldest"** (Cũ nhất trước).

**Tổng số video:**

- Số lượng video tìm được hiển thị dưới dạng **"Total: [n] videos"** ở góc trên bên phải.

**Cuộn để tải thêm:**

- Hệ thống tải 20 video đầu tiên. Cuộn xuống cuối danh sách để tự động tải thêm 20 video tiếp theo (20 → 40 → 60, v.v.).

---

## 6.2 Xem và Tải Video (Video Play and Download)

### Cách xem lại Video

**1. Nhấp vào thẻ video**

Từ danh sách kết quả tìm kiếm tại màn hình Video Storage, nhấp vào bất kỳ thẻ video nào. Cửa sổ **Video Details** (Chi tiết video) sẽ hiện ra và video sẽ **tự động bắt đầu phát**.

**2. Sử dụng các nút điều khiển phát lại**

Cửa sổ Video Details cung cấp đầy đủ các công cụ sau:

| Nút điều khiển                                       | Chức năng                                                                                      |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Play / Pause** (Phát / Tạm dừng)                   | Nhấp để phát hoặc tạm dừng video.                                                              |
| **Rewind / Forward 10s** (Tua lại / Tua tới 10 giây) | Nhấp để tua lùi hoặc tua tiến 10 giây.                                                         |
| **Time bar** (Thanh tiến trình)                      | Hiển thị tiến trình phát của video. Nhấp hoặc kéo thanh trượt để nhảy đến thời điểm mong muốn. |
| **Volume** (Âm lượng)                                | Nhấp để bật/tắt âm thanh. Mặc định: **Tắt**.                                                   |
| **Play speed** (Tốc độ phát)                         | Nhấp để chọn tốc độ: 0.25x, 0.5x, 0.75x, 1x (mặc định), 1.25x, 1.5x, 1.75x, 2x.                |
| **Full screen** (Toàn màn hình)                      | Nhấp để mở chế độ xem toàn màn hình.                                                           |

**Thanh tiến trình (Time bar) cho video có sự kiện:**

- **Đánh dấu đỏ (Red marker):** Hiển thị vị trí thời gian chính xác nơi sự kiện xảy ra trên thanh tiến trình.
- **Đoạn trắng (White segment):** Hiển thị vị trí phát lại hiện tại.
- Di chuột qua thanh tiến trình sẽ hiển thị nhãn thời gian chính xác.

**3. Xem danh sách sự kiện (Event List)**

Đối với video được tự động lưu (Autosave) có chứa sự kiện, bảng **"Event list"** hiển thị ở bên phải cửa sổ, bao gồm:

- **Event total:** Tổng số sự kiện xảy ra trong video.
- **Chi tiết sự kiện:** Tên sự kiện, hình ảnh sự kiện, thời gian xảy ra.
- **False positive check:** Ô tích để đánh dấu nếu AI nhận diện sai (xem thêm phần 4.5 Phản hồi hệ thống).

**4. Chuyển giữa các video**

Sử dụng nút **Previous** (Trước) và **Next** (Tiếp) để chuyển sang video trước hoặc sau trong danh sách kết quả.

- Nút Previous bị vô hiệu hóa khi đang ở video đầu tiên.
- Nút Next bị vô hiệu hóa khi đang ở video cuối cùng.

---

### Cách tải Video về máy

**1. Mở video cần tải**

Nhấp vào thẻ video trong danh sách Video Storage để mở cửa sổ Video Details.

**2. Nhấp nút "Download"**

Nhấp vào nút **Download** (biểu tượng tải xuống) trên cửa sổ Video Details.

**3. Lưu file**

Video sẽ được tải về máy tính của bạn ở định dạng **MP4**.

> **Lưu ý:** Chỉ có định dạng MP4 được hỗ trợ khi tải xuống.

---

### Tải lên Video thủ công (Manual Upload)

Nếu bạn cần tải video lên hệ thống theo cách thủ công:

**1. Chuyển sang chế độ "Manual"**

Tại màn hình Video Storage, chọn loại video **"Manual"**.

**2. Nhấp nút "Upload video"**

Nút này nằm ở phía trên danh sách video. Cửa sổ **Upload video** sẽ hiện ra.

**3. Chọn file video**

- Kéo và thả file video vào vùng tải lên, hoặc
- Nhấp nút **"Select file"** để duyệt và chọn file từ máy tính.

> **Định dạng được chấp nhận:** Chỉ hỗ trợ file **MP4** và **MOV (H264)**. Nếu file không đúng định dạng, hệ thống sẽ thông báo lỗi _"Invalid file format"_.

**4. Xem trước video**

Sau khi chọn file, video sẽ hiển thị bản xem trước với đầy đủ nút điều khiển phát lại.

**5. Chọn Camera (Select channel)**

Sử dụng menu thả xuống **"Select channel"** để gán video cho camera cụ thể.

**6. Nhập mô tả (Description) – Tùy chọn**

Nhập nội dung mô tả cho video vào ô **Description**.

**7. Xác nhận tải lên**

- Nhấp **"Confirm"** để hoàn tất tải lên.
- Nhấp **"Cancel"** hoặc **"Close"** để hủy bỏ.

---

### Quy tắc Lưu trữ & Xóa tự động

Hệ thống HiCAMS áp dụng các quy tắc lưu trữ sau để quản lý dung lượng:

| Quy tắc                        | Chi tiết                                                                                                               |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| **Tự động lưu**                | Hệ thống tự động lưu các đoạn video có chứa sự kiện được AI phát hiện.                                                 |
| **Xóa video không có sự kiện** | Video không chứa sự kiện sẽ bị **tự động xóa sau 5 phút** kể từ khi được lưu.                                          |
| **Giới hạn dung lượng**        | Admin và Manager có thể thiết lập giới hạn dung lượng tối đa (Total recording limit) trong phần cài đặt Video Storage. |
| **Xóa theo nguyên tắc FIFO**   | Khi dung lượng đạt giới hạn, hệ thống sẽ **tự động xóa video cũ nhất** trước để giải phóng dung lượng cho video mới.   |

> **Lưu ý quan trọng:** Nếu một video liên kết với sự kiện trong nhật ký đã bị xóa do chính sách lưu trữ, khi mở sự kiện đó, hệ thống sẽ hiển thị thông báo **"Video has been removed"** (Video đã bị xóa) và các nút điều khiển phát sẽ bị vô hiệu hóa. Tuy nhiên, thông tin sự kiện và hình ảnh chụp (tab Capture) vẫn hoạt động bình thường.
