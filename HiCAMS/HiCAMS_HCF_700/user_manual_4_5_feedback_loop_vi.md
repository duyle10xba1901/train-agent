# 4.5 Phản hồi hệ thống (Feedback Loop)

Hệ thống HiCAMS sử dụng AI để tự động phát hiện các sự kiện an toàn từ camera CCTV. Tuy nhiên, việc nhận diện của AI không phải lúc nào cũng chính xác 100%. Vì vậy bạn cần xem lại các cảnh báo và đánh dấu xem mỗi lần nhận diện là đúng hay sai. Phản hồi của bạn giúp giữ cho nhật ký sự kiện và báo cáo luôn chính xác.

---

## Hiểu về Cảnh báo Đúng (True Positive) và Cảnh báo Sai (False Positive)

Trước khi xem lại các cảnh báo, bạn cần hiểu hai thuật ngữ chính sau:

| Thuật ngữ | Ý nghĩa | Ví dụ |
|-----------|---------|-------|
| **Cảnh báo Đúng** (True Positive) | AI đã phát hiện chính xác một sự kiện an toàn thực tế. | Một công nhân **không** đội mũ bảo hộ, và hệ thống báo động "Phát hiện không đội mũ". |
| **Cảnh báo Sai** (False Positive) | AI đã phát hiện sai một sự kiện không thực sự xảy ra. | Một công nhân **đang** đội mũ bảo hộ, nhưng hệ thống vẫn báo động "Phát hiện không đội mũ". |

> **Mẹo:** Khi xem lại một cảnh báo, hãy luôn nhìn kỹ hình ảnh chụp được và so sánh với mô tả sự kiện. Nếu những gì bạn thấy trong ảnh khớp với cảnh báo, đó là **Cảnh báo Đúng**. Nếu ảnh không khớp với cảnh báo, đó là **Cảnh báo Sai**.

---

## Cách đánh dấu một sự kiện là Cảnh báo Sai (False Positive)

Nếu bạn thấy AI nhận diện sai một sự kiện (Cảnh báo Sai), bạn có thể đánh dấu nó trong Nhật ký sự kiện (Event Log). Việc này sẽ loại bỏ sự kiện sai đó khỏi các báo cáo của bạn.

**1. Mở Nhật ký sự kiện (Event Log)**

Đi đến màn hình **Event Log Explorer** để xem danh sách các sự kiện đã được phát hiện.

**2. Nhấp vào sự kiện bạn muốn xem lại**

Nhấp vào bất kỳ dòng nhật ký nào. **Cửa sổ chi tiết nhật ký** (Event Log Detail popup) sẽ hiện ra, hiển thị:
- Hình ảnh chụp lại của sự kiện.
- Tên của đối tượng được phát hiện.
- Khung nhận diện trên hình ảnh.
- Thông tin cảnh báo: tên camera, loại sự kiện, tên sự kiện, phần trăm rủi ro, ngày và giờ.

**3. Đánh dấu là Cảnh báo Sai (False Positive)**

Ở phía dưới cùng của cửa sổ chi tiết, bạn sẽ thấy một ô tích có tên **"False positive check"**.
- Nhấp vào ô tích này để đánh dấu sự kiện này là nhận diện sai.
- Sự kiện sẽ bị **loại bỏ ngay lập tức** khỏi danh sách nhật ký sự kiện.
- Sự kiện cũng sẽ bị **loại trừ khỏi các báo cáo** được tạo ra.

**4. Đóng cửa sổ**

Nhấp vào nút **Close** (Đóng) để đóng cửa sổ chi tiết.

---

## Cách xem lại các sự kiện đã đánh dấu là Cảnh báo Sai

Theo mặc định, Nhật ký sự kiện chỉ hiển thị các sự kiện được nhận diện đúng. Nếu bạn muốn xem lại các sự kiện đã từng bị đánh dấu là Cảnh báo Sai:

**1. Đi đến màn hình Event Log Explorer**

**2. Tìm phần lọc "Detection error"**

**3. Tích vào ô "Including false positive"**
- Thao tác này sẽ hiển thị **cả** các nhật ký sự kiện đúng **và** các sự kiện đã bị đánh dấu là Cảnh báo Sai.

**4. Bỏ tích ô "Including false positive"** để quay lại chế độ xem mặc định (chỉ các sự kiện đúng).

> **Lưu ý:** Theo mặc định, ô "Including false positive" **không được tích**, nghĩa là các sự kiện Cảnh báo Sai sẽ bị ẩn khỏi danh sách.

---

## Ảnh hưởng của phản hồi Cảnh báo Sai đến Báo cáo

Khi bạn tạo báo cáo từ màn hình Event Log Explorer bằng nút **"Create Report"**:
- Báo cáo sẽ **chỉ bao gồm các sự kiện được nhận diện đúng**.
- Các sự kiện đã bị đánh dấu là **"False Positive" sẽ KHÔNG xuất hiện** trong báo cáo.
- Việc này đảm bảo báo cáo của bạn chỉ phản ánh các sự kiện an toàn thực tế.
