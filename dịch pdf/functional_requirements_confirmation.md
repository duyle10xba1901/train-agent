# 📋 Bản Yêu Cầu Chức Năng — Fleet Dashboard & OTA/AWS
## (Dùng để xác nhận với khách hàng)

> **Nguồn tham chiếu:**
> - Fleet_dashboard_v0 (1).pdf — Đề xuất tính năng Fleet Dashboard
> - 선박용_OTA_AWS_요구기능_정리.pdf — Yêu cầu chức năng OTA/AWS cho tàu biển
>
> **Hệ thống:** SMTAI HMS Fleet Dashboard
> **Ngày tạo:** 2026-04-23
> **Người tạo:** BA Team

---

## 📌 Mục đích tài liệu

Tài liệu này tổng hợp **toàn bộ yêu cầu chức năng** từ 2 tài liệu của khách hàng, được phân nhóm theo module. Mỗi yêu cầu kèm theo **câu hỏi cần confirm** trước khi tiến hành thiết kế & phát triển.

---

## I. TỔNG QUAN HỆ THỐNG

**Mô tả:** Nền tảng web giám sát an toàn hàng hải từ xa, quản lý đội tàu, xử lý sự kiện, và triển khai phần mềm OTA cho thiết bị edge trên tàu.

**Tiền đề kỹ thuật (từ tài liệu gốc):**
- Không có realtime video streaming → Hệ thống hoạt động theo mô hình **event-driven** (xem clip sau sự kiện)
- Hạ tầng cloud: AWS (IoT Greengrass, MQTT, IoT Jobs)
- Edge device trên tàu chạy container-based services

**2 nhóm người dùng chính:**

| Nhóm | Đối tượng | Quyền hạn |
|------|-----------|-----------|
| **Admin** | Nhà phát triển (Dev) + Nhân viên vận hành (Ops) | Toàn quyền — Dev có quyền cao nhất |
| **End-User** | Chủ tàu / Nhân viên an toàn (Safety Officer) | Giới hạn — chỉ giám sát & xem báo cáo |

### ❓ Câu hỏi xác nhận — Tổng quan

| # | Câu hỏi | Lý do cần hỏi |
|---|---------|----------------|
| Q1 | Dev và Ops **chia sẻ cùng 1 dashboard** nhưng phân quyền khác nhau — đúng không? Cụ thể Ops bị giới hạn những gì so với Dev? | Để thiết kế RBAC matrix |
| Q2 | End-user (chủ tàu) có cần **tài khoản riêng** để đăng nhập hay chỉ được cấp link xem báo cáo? | Quyết định có cần user management cho end-user không |
| Q3 | Hệ thống có cần hỗ trợ **multi-tenant** không? (1 instance phục vụ nhiều công ty chủ tàu khác nhau) | Ảnh hưởng đến kiến trúc data isolation |

---

## II. YÊU CẦU CHỨC NĂNG CHI TIẾT

### Module 1: Fleet Overview — Tổng quan đội tàu

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-1.1 | Bản đồ AIS | Cả 2 file | Hiển thị vị trí tàu real-time trên bản đồ dựa theo dữ liệu AIS |
| FR-1.2 | Hiển thị hải trình (Route) | OTA/AWS | Hiển thị hải trình tàu trên bản đồ |
| FR-1.3 | Trạng thái kết nối tàu | Cả 2 file | Online/Offline, thời điểm đồng bộ cuối, latency |
| FR-1.4 | Chọn đội tàu (Fleet Selection) | Fleet Dashboard | Màn hình chọn/chuyển đổi giữa các đội tàu |
| FR-1.5 | Tóm tắt theo tàu | Fleet Dashboard | Số camera, số event, trạng thái alarm mỗi tàu |
| FR-1.6 | Quản lý theo cấu trúc phân cấp | OTA/AWS | Khách hàng → Đội tàu → Tàu (multi-level hierarchy) |

### ❓ Câu hỏi xác nhận — Fleet Overview

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q4 | AIS data lấy từ đâu? API bên thứ 3 hay tự thu thập từ thiết bị trên tàu? | Quyết định kiến trúc map component |
| Q5 | Tính năng AIS Map và Route Display đã **xác nhận khả thi** chưa? (Tài liệu OTA/AWS ghi "cần xác nhận") | Quyết định có đưa vào MVP hay không |
| Q6 | Số lượng tàu dự kiến quản lý đồng thời? (5? 50? 500?) | Ảnh hưởng performance & UI pagination |
| Q7 | 1 khách hàng có thể sở hữu **bao nhiêu đội tàu**? | Quyết định navigation hierarchy |

---

### Module 2: Event Management — Quản lý sự kiện

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-2.1 | Danh sách sự kiện | Cả 2 file | Event list với filter/search, phân loại theo severity |
| FR-2.2 | Chi tiết sự kiện | Cả 2 file | Xem thông tin chi tiết: thời gian, loại, camera, tàu |
| FR-2.3 | Video clip sự kiện | Cả 2 file | Phát lại video trước/sau thời điểm sự kiện |
| FR-2.4 | Hình ảnh sự kiện | Cả 2 file | Xem ảnh chụp liên quan |
| FR-2.5 | Trạng thái sự kiện | Fleet Dashboard | Lifecycle: New → In Progress → Resolved |
| FR-2.6 | Xử lý hậu sự kiện | Fleet Dashboard | Ghi nhận biện pháp xử lý, lưu bằng chứng |
| FR-2.7 | Xu hướng sự kiện (Trend) | Fleet Dashboard | Biểu đồ trend event theo thời gian (24h/7d) |
| FR-2.8 | Truyền dữ liệu theo cấp sự kiện | OTA/AWS | Cháy/khói → gửi video+ảnh **tức thì**; Event khác → gửi ảnh+metadata tức thì |
| FR-2.9 | Tìm kiếm & lọc sự kiện | Fleet Dashboard | Filter theo tàu, thời gian, loại event; Bookmark/Favorites |

### ❓ Câu hỏi xác nhận — Event Management

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q8 | Vòng đời sự kiện gồm những trạng thái nào? (New → Acknowledged → In Progress → Resolved → Closed?) Ai chuyển trạng thái? | Thiết kế event state machine |
| Q9 | "Xử lý hậu sự kiện" — cần quy trình approval không? Hay chỉ ghi nhận text? | Quyết định workflow complexity |
| Q10 | Sự kiện **cháy/khói** gửi video tức thì — video bao nhiêu giây? Trước và sau sự kiện bao lâu? | Spec cho video clip extraction |
| Q11 | HiCAMS lưu trữ video/ảnh cho data collection được ghi là "phát triển sau" — có cần đặt placeholder trong UI không? | Quyết định có thiết kế sẵn hay không |

---

### Module 3: Monitoring & Edge Health — Giám sát hệ thống

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-3.1 | Dashboard trạng thái hệ thống | Cả 2 file | Trạng thái server, mạng, dịch vụ theo từng tàu |
| FR-3.2 | Giám sát Edge Device | Fleet Dashboard | CPU/GPU/Memory/Storage trên thiết bị edge |
| FR-3.3 | Trạng thái kết nối camera | Cả 2 file | Online/Offline cho từng camera |
| FR-3.4 | Bảng cảnh báo (Alarm) | Cả 2 file | Alarm khi phát hiện bất thường (deploy/vận hành) |
| FR-3.5 | Bandwidth & Latency | Cả 2 file | Giám sát băng thông mạng theo tàu |
| FR-3.6 | Error Message Viewer | OTA/AWS | Hiển thị error messages từ các tàu — cho nhân viên non-dev |

### ❓ Câu hỏi xác nhận — Monitoring

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q12 | Edge Health data (CPU/GPU/Memory) polling bao lâu 1 lần? Realtime hay mỗi 5 phút? | Spec cho monitoring interval |
| Q13 | Ngưỡng cảnh báo (threshold) cho CPU/Memory/Storage — khách có giá trị cụ thể không? Hay để user tự cấu hình? | Thiết kế alert rule engine |
| Q14 | "Nhân viên non-dev có thể deploy/debug đơn giản" — cụ thể non-dev là ai? Operator tại trung tâm trên bờ? Hay nhân viên IT của chủ tàu? | Quyết định mức đơn giản hóa UI |

---

### Module 4: OTA Deployment — Triển khai phần mềm từ xa

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-4.1 | Deploy Controller | Cả 2 file | Deploy theo 3 cấp: hàng loạt / đội tàu / từng tàu |
| FR-4.2 | Trạng thái deploy | Cả 2 file | Progress, success, failed theo từng tàu |
| FR-4.3 | Lịch sử deploy | Cả 2 file | History với filter theo tàu/ngày/kết quả |
| FR-4.4 | Rollback | Cả 2 file | Quay lại phiên bản trước khi deploy lỗi |
| FR-4.5 | Rollout theo giai đoạn | Fleet Dashboard | Deploy dần: pilot → batch → full fleet |
| FR-4.6 | Ma trận phiên bản | Fleet Dashboard | Hiển thị version matrix theo tàu |
| FR-4.7 | Tải xuống thích ứng băng thông | OTA/AWS | Phát hiện bandwidth → quyết định download/dừng |
| FR-4.8 | Tách download / deploy | OTA/AWS | Download image xong → user confirm → mới deploy |
| FR-4.9 | Đăng ký tàu mới | OTA/AWS | Onboarding: đăng ký & cài đặt tàu mới vào hệ thống |
| FR-4.10 | Triển khai cấu hình | Fleet Dashboard | Deploy config changes (không chỉ software) |

### ❓ Câu hỏi xác nhận — OTA

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q15 | Deploy cần bao nhiêu bước xác nhận? (Download → Confirm → Deploy → Verify?) | Thiết kế wizard flow |
| Q16 | Rollback tự động khi health check fail? Hay manual rollback? | Spec cho rollback strategy |
| Q17 | "Rollout theo giai đoạn" — tỷ lệ như thế nào? (10% → 50% → 100%?) Ai quyết định chuyển giai đoạn? | Spec cho staged rollout |
| Q18 | Secure tunneling cho trường hợp khẩn cấp — scope cụ thể? SSH vào edge device? Hay chỉ gửi command? | Quyết định security boundary |

---

### Module 5: Quản lý từ xa (Remote Management)

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-5.1 | Cài đặt camera từ xa | Fleet Dashboard | Cấu hình camera parameters |
| FR-5.2 | Cấu hình model AI từ xa | OTA/AWS | Thay đổi model params (vd: decision_threshold) |
| FR-5.3 | Cấu hình biến DB từ xa | OTA/AWS | Thay đổi DB variables từ cloud |
| FR-5.4 | Chẩn đoán từ xa (Remote Diagnostics) | Fleet Dashboard | Log chi tiết, Log Explorer, lệnh từ xa |
| FR-5.5 | Kết nối camera portable | Fleet Dashboard | Liên kết camera di động vào hệ thống |

### ❓ Câu hỏi xác nhận — Remote Management

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q19 | Thay đổi config từ xa (model, DB) có cần **audit log** không? Cần approval từ ai? | Thiết kế remote config + audit trail |
| Q20 | Remote Terminal (tài liệu ghi "not usual") — có đưa vào scope hay loại bỏ? | Quyết định scope |
| Q21 | Camera portable — loại camera nào? Protocol kết nối? (RTSP? ONVIF? USB?) | Spec cho camera integration |

---

### Module 6: Thống kê & Báo cáo (Statistics & Reports)

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-6.1 | Safety Score | Fleet Dashboard | Chấm điểm an toàn cho từng tàu |
| FR-6.2 | Thống kê sự kiện | Fleet Dashboard | Event stats theo ngày/tuần (24h/7d trend) |
| FR-6.3 | Tỷ lệ xử lý sự kiện | Fleet Dashboard | % event đã xử lý vs chưa xử lý |
| FR-6.4 | Báo cáo tháng (Monthly Report) | Fleet Dashboard | Báo cáo tổng hợp định kỳ |
| FR-6.5 | Thống kê đội tàu | Fleet Dashboard | So sánh performance giữa các tàu |

### ❓ Câu hỏi xác nhận — Statistics

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q22 | **Safety Score** tính theo công thức nào? Dựa trên metrics nào? Ngưỡng Red/Yellow/Green? | Spec chính xác cho scoring algorithm |
| Q23 | Báo cáo tháng — format nào? (PDF export? Dashboard view? Email tự động?) | Quyết định report output format |
| Q24 | Có cần **so sánh** Safety Score giữa các tàu / giữa các tháng không? | Quyết định chart types cần thiết |

---

### Module 7: MLOps (Chỉ dành cho Admin)

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-7.1 | Thu thập dữ liệu false-positive | Fleet Dashboard | Gom data overkill/miss cho retrain model |
| FR-7.2 | Trạng thái update model | Fleet Dashboard | Theo dõi model training/deployment status |
| FR-7.3 | Lịch sử phiên bản model | Fleet Dashboard | Model version history & changelog |
| FR-7.4 | Thu thập dữ liệu định kỳ | Fleet Dashboard | Scheduled data collection pipeline |

### ❓ Câu hỏi xác nhận — MLOps

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q25 | MLOps có nằm trong scope MVP hay Phase sau? | Phân chia priority |
| Q26 | Data collection pipeline — dữ liệu gửi về AWS S3? Hay storage riêng? | Spec cho data architecture |

---

### Module 8: Khách hàng & Quản lý tàu (Admin only)

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-8.1 | CRUD Khách hàng | OTA/AWS | Đăng ký, chỉnh sửa, xóa khách hàng |
| FR-8.2 | CRUD Đội tàu / Tàu | OTA/AWS | Quản lý fleet & ship per customer |
| FR-8.3 | Phân quyền | Cả 2 file | RBAC: Admin/Operator/Viewer |
| FR-8.4 | Tiến độ cài đặt | Fleet Dashboard | Tracking installation progress cho tàu mới |
| FR-8.5 | Audit Log | Fleet Dashboard | Lịch sử thay đổi cài đặt, xem video |

### ❓ Câu hỏi xác nhận — Admin

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q27 | Audit log lưu trữ bao lâu? Có yêu cầu compliance nào không? (IMO regulations?) | Data retention policy |
| Q28 | Phân quyền có bao nhiêu role? Chỉ 3 (Admin/Operator/Viewer) hay cần custom role? | RBAC complexity |

---

### Module 9: Hỗ trợ & Giao tiếp

| REQ-ID | Yêu cầu | Nguồn | Mô tả |
|--------|---------|-------|-------|
| FR-9.1 | Chat tàu ↔ bờ | Fleet Dashboard | Giao tiếp giữa thuyền viên và giám sát viên trên bờ |
| FR-9.2 | Mobile notification | Fleet Dashboard | Push notification cho end-user khi có event |

### ❓ Câu hỏi xác nhận — Support

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q29 | Chat — chỉ text hay cần gửi file/ảnh? Có cần lưu lịch sử chat không? | Scope cho chat feature |
| Q30 | Mobile notification — app riêng hay tích hợp vào app có sẵn? Trigger sự kiện nào? | Platform decision |

---

## III. ITEMS CÓ MÂU THUẪN / CẦN LÀM RÕ

> [!WARNING]
> Các điểm sau có sự **không nhất quán** giữa 2 tài liệu hoặc được đánh dấu nghi vấn:

| # | Vấn đề | Chi tiết | Câu hỏi |
|---|--------|----------|---------|
| ⚠️1 | **Video feed real-time** | Tài liệu OTA/AWS đánh dấu **(x)** — có thể đã bị loại. Nhưng Fleet Dashboard vẫn có module Video & Data | **Q31:** Video realtime streaming **chính thức bị loại** hay chỉ **hoãn lại**? |
| ⚠️2 | **Video truyền về bờ** | OTA/AWS yêu cầu "truyền video về trung tâm trên bờ" nhưng Fleet Dashboard ghi "không có realtime streaming" | **Q32:** Cần phân biệt rõ: video truyền tức thì khi cháy/khói vs. video streaming liên tục — scope chính xác là gì? |
| ⚠️3 | **AIS Map khả thi?** | OTA/AWS ghi "cần xác nhận khả thi" cho AIS Map & Route Display | **Q33:** Đã có kết quả kiểm tra tính khả thi cho AIS integration chưa? |

---

## IV. ĐỀ XUẤT PHÂN CHIA PHASE

| Phase | Scope đề xuất | Ghi chú |
|-------|---------------|---------|
| **MVP** | Monitoring Dashboard, OTA Deployment, Event Management (cơ bản), Edge Health, RBAC | Core value — vận hành được |
| **Phase 2** | Fleet Overview (AIS Map), Remote Config, Customer/Fleet/Ship CRUD, Statistics | Mở rộng quản lý |
| **Phase 3** | Safety Score, Monthly Report, MLOps, Chat, Mobile Notification, Portable Camera | Nâng cao trải nghiệm |

### ❓ Câu hỏi xác nhận — Phân phase

| # | Câu hỏi | Lý do |
|---|---------|-------|
| Q34 | Khách có đồng ý với phân chia phase trên không? Có tính năng nào cần **đẩy lên MVP** hoặc **đẩy xuống Phase sau**? | Align priority |
| Q35 | Timeline mong muốn cho MVP là khi nào? | Ảnh hưởng resource planning |

---

## V. TỔNG HỢP CÂU HỎI (Checklist để mang đi confirm)

### 🔴 Câu hỏi CRITICAL (Cần trả lời trước khi bắt đầu)

| # | Câu hỏi | Module |
|---|---------|--------|
| Q1 | Dev vs Ops — khác nhau quyền gì cụ thể? | Tổng quan |
| Q5 | AIS Map & Route Display đã xác nhận khả thi chưa? | Fleet Overview |
| Q6 | Số lượng tàu quản lý đồng thời? | Fleet Overview |
| Q8 | Vòng đời sự kiện gồm những trạng thái nào? Ai chuyển? | Event |
| Q14 | "Nhân viên non-dev" cụ thể là ai? | Monitoring |
| Q22 | Safety Score tính theo công thức gì? | Statistics |
| Q31 | Video realtime — loại bỏ hay hoãn? | Mâu thuẫn |
| Q34 | Đồng ý phân chia phase? | Phase |
| Q35 | Timeline MVP? | Phase |

### 🟡 Câu hỏi IMPORTANT (Cần trả lời trước khi thiết kế UI)

| # | Câu hỏi | Module |
|---|---------|--------|
| Q3 | Multi-tenant? | Tổng quan |
| Q4 | AIS data source? | Fleet Overview |
| Q10 | Video clip sự kiện — bao nhiêu giây? | Event |
| Q15 | OTA deploy cần bao nhiêu bước? | OTA |
| Q16 | Rollback tự động hay manual? | OTA |
| Q19 | Remote config cần audit log? | Remote |
| Q28 | Bao nhiêu role phân quyền? | Admin |
| Q32 | Scope video truyền về bờ? | Mâu thuẫn |

### 🟢 Câu hỏi NICE-TO-HAVE (Có thể trả lời sau)

| # | Câu hỏi | Module |
|---|---------|--------|
| Q11 | HiCAMS placeholder? | Event |
| Q20 | Remote Terminal — in scope? | Remote |
| Q21 | Portable camera protocol? | Remote |
| Q23 | Report format? | Statistics |
| Q25 | MLOps trong MVP? | MLOps |
| Q29 | Chat scope? | Support |
| Q30 | Mobile app riêng? | Support |

---

> [!TIP]
> **Hướng dẫn sử dụng:** In tài liệu này → Mang vào buổi họp với khách → Đi qua từng module → Tick vào câu hỏi đã được trả lời → Ghi nhận câu trả lời → Cập nhật lại document sau meeting.
