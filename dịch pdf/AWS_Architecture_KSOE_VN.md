# Quản lý CCTV Tàu HiCAMS
**Đề xuất kiến trúc To-Be dựa trên AWS IoT**
**Khách hàng:** KSOE

Đề xuất hai tùy chọn kiến trúc để liên kết cụm (cluster) K3s hiện tại với các dịch vụ AWS IoT và đưa ra lộ trình chuyển đổi được khuyến nghị.

---

## 1. Các Giai Đoạn (Phases)

### Phase 1: Cập nhật OTA (Over-The-Air)
**Tự động hóa phân phối phần mềm từ xa**
- Triển khai dựa trên các component của Greengrass
- Hỗ trợ triển khai đồng thời trên nhiều tàu
- Hỗ trợ rollback và quản lý phiên bản
*Công nghệ:* AWS IoT Greengrass

### Phase 2: Video streaming + Điện toán Biên (Edge)
**Suy luận & Cảnh báo phát hiện bất thường / Phân tích thời gian thực bằng YOLO**
- Xử lý lai (Hybrid) giữa Biên và Đám mây (Cloud)
- Đồng bộ kết quả suy luận tại biên theo thời gian thực
- Truyền tải các video clip
*Công nghệ:* AWS IoT Greengrass + Amazon S3

### Phase 3: Dịch vụ giám sát cho Chủ tàu
**Dashboard, Cảnh báo, Phân tích dữ liệu**
- Cung cấp dashboard quản trị tích hợp
- Tự động hóa cảnh báo phát hiện sự cố
- Phân tích dữ liệu vận hành của tàu
*Công nghệ:* Nền tảng tích hợp AWS Cloud

---

## 2. Bối cảnh dự án & Cấu hình AS-IS (Hiện tại)

### Cụm K3s trên tàu (선박 K3s 클러스터)
- **CI/CD Pipeline**
  - GitLab CI: Build & Tạo Image
  - GitLab Registry: Nơi lưu kho Image của Container
  - `kubectl apply`: Triển khai ứng dụng lên cụm K3s
- **Application Services (Dịch vụ Ứng dụng)**
  - `hicams-server`: Express REST API (~300MB)
  - `hicams-vision3`: YOLO v8 + Suy luận GPU (~2GB, bao gồm CUDA)
  - `hicams-vlm`: Phân tích VLM đa phương thức (~10GB+, trọng số LLM)
  - `cloud-sync-agent`: Node.js (~150MB) 
  - `react-frontend`: (~200MB)
  - `hicams-report`: (~200MB)
  - ▲ *Đưa vào sử dụng mới:* `hicams-mediamtx`: Truyền phát RTSP/HLS stream (~100MB)
- **Storage (Lưu trữ)**
  - MariaDB: Hệ cơ sở dữ liệu quan hệ
  - MinIO: Nơi lưu trữ object (video clip)
  - Redis: Lớp bộ nhớ đệm (Cache)
- **Monitoring (Giám sát)**
  - Prometheus: Thu thập & lưu trữ số liệu (Metrics)
  - Grafana: Dashboard trực quan hóa hiển thị

> ⚠ **Hạn chế của hệ thống hiện tại:** Không có kết nối mạng Đám mây (Cloud) — Mọi hệ thống đều vận hành nội bộ cục bộ trên tàu (Cập nhật bằng cách cắm USB).
> ⚠ **Giới hạn tường lửa (Firewall):** Môi trường cụm nằm dưới sự kiểm soát của tường lửa mạng tàu. Không thể kết nối từ bên ngoài vào (inbound) trực tiếp. Khách hàng chỉ cho phép cổng outbound: `443 (HTTPS)` và `8883 (MQTT over TLS)`.

---

## 3. Giải pháp Khắc phục Giới hạn Mạng

### Giải pháp 1: AWS IoT Secure Tunneling
- **Khởi tạo luồng ra Outbound :443 (HTTPS)**
- Tác nhân (agent) trên tàu sẽ mở kết nối ra ngoài tạo ra một đường hầm bảo mật 2 chiều (bidirectional secure tunnel).
- Từ trên Cloud có thể truy cập điều khiển từ xa vào tàu (Để tải OTA, sửa lỗi khắc phục sự cố).
- Không yêu cầu thay đổi tùy chỉnh trên tường lửa.

### Giải pháp 2: MQTT over TLS
- **Khởi tạo luồng ra Outbound :8883 → Duy trì kết nối thường xuyên với IoT Core**
- Phương tiện duy trì sẵn 1 kết nối đến IoT Core.
- Truyền tải thông số đo từ xa (Telemetry) và sự kiện (Events) Real-time.
- Không yêu cầu thay đổi chính sách tường lửa.

---

## 4. Nguyên tắc Thiết kế Cốt lõi — Chiến lược Cài đặt Greengrass

📌 **Vị trí cài đặt:** Greengrass được cài đặt ở vòng ngoài cụm K3s, dưới dạng thiết lập Service `systemd` của Host OS. Tuân thủ kiến trúc khuôn mẫu kiến trúc (Reference Architecture) từ AWS.

⚠️ **Phòng ngừa Bootstrap Deadlock:** Nếu Agent OTA (Greengrass) vận hành bên trong K3s thì lúc cụm K3s bị sụp thì sẽ không tự khởi động / khắc phục khôi phục lại được.

🔗 **Luồng truyền tải thông tin:**
- Greengrass → K3s API Server (`localhost:6443`) qua tệp cấu hình kubeconfig.
- `K3s NodePort` sẽ bộc lộ Endpoint IPC của Greengrass (Điều kiện cần thiết của ứng dụng tự động phục hồi - Self-recovery OTA).

---

## 5. Dòng Hoạt động Dữ liệu và Thay đổi Hệ thống K3S

**Nguyên tắc thiết kế:** Bảo tồn tối đa hệ thống K3s cũ · Giảm thiểu vai trò của Greengrass
- Không sử dụng KVS (Kinesis Video Streams) · Áp dụng phương thức tải theo lô (Batch) vào S3 · Tiết kiệm tối đa băng thông mạng vệ tinh · Kết hợp GitLab CR + Đường ống Amazon S3.
- `CloudSync` chuyển kiện hệ thống Node.js cũ. Phương pháp tải File tĩnh trực tiếp lên S3 (Nút dữ liệu MinIO).
- **S3 và CloudFront:** Xử lý và phân phối phát vi phát lại (Clip Playback).

---

## 6. Lộ trình Triển khai Cập nhật OTA (Kịch bản phân bổ)

- Môi trường liên lạc tàu ngoài biển trải qua các hạn chế bao gồm: Băng thông hẹp (Vài chục đến Vài trăm Kbps), Độ trễ cấu hình giao tiếp cao (>600ms), đôi lúc rớt mạng ngắt quãng định kỳ.

**Thiết kế nguyên tắc OTA cực kỳ ổn định:**
1. **Lưu trữ chờ ngoại tuyến (Offline Queueing):** Trong trường hợp tắt mạng thì IoT Jobs sẽ được giữ lại cục bộ. Đợi mạng liên kết lại thì kích hoạt trở lại tự động.
2. **Cập nhật đệm Image trước (Pre-staging):** Tải phần trọng số ~10GB và Image ~2GB về khối lượng cứng MinIO của tàu một cách ngầm trước rồi mới update phân phối. Rủi ro lỗi hỏng trong môi trường sóng yếu sẽ không làm gãy dịch vụ.
3. **Tối thiểu hóa kích thước truy vấn:** Ứng dụng truyền thông điệp của bộ đệm lớp layer cache. Thay vì 200Mb thì gửi 20 Mb để tinh giảm sử dụng Internet.
4. **Triển khai tuần tự từng bước:** 1 chiếc ➜ Nhóm (5 tàu) ➜ Toàn bộ hạm đội tàu kiểm duyệt trong 24 giờ.

---

## 7. Các Kế hoạch Xử Lộ trình Theo Kịch Bản:

- **Kịch bản số 1 - Chạy triển khai OTA bình thường:**  
  Nhận OTA ➜ Pull Data (S3/ECR) ➜ Thực thi lệnh `kubectl apply` ➜ Trạng thái tốt ➜ Update lên AWS Device Shadow: SUCCESS.

- **Kịch bản số 2 – Ngắt tín hiệu Mạng:**  
  Vệ tinh ngắt → Mở lại kết nối từ Greengrass queue giữ lại → Cập nhật thành công.

- **Kịch bản số 3 – Backup lúc Cảng Neo:**  
  Tận dụng môi trường sóng ở cảng (thay vì đang ngoài khơi sóng đắt) để tải image lớn xong. Lỗi có thể roll back và trả thông báo Failed qua SNS.
  
- **Kịch bản số 4 - Tàu bị off mạng dài ngày:**  
  Nhiều lệnh IoT lỡ chừng xếp nhiều nhưng agent sẽ thông minh chỉ Update bản mới nhất khi tàu phát wifi online kết nối lại.

---

## 8. Trọng tâm Xử lý Image Models Lớn

**So sánh sự ổn định:** Greengrass OTA tối ưu hơn hẳn tính năng tự tải từ K3S Containerd (dễ trầy trật tải lỗi mất nguyên cục 7GB bắt tải lại). Nguyên tắc cốt lõi: Phân tách Mã lập trình và AI Models nguyên bản. 
- AI Models / Trọng Số = Sẽ chèn bằng **PV Mounts**
- Tải Image = Chứa Script Code (Nhẹ vài chục 100mb)

**Tối ưu Đường Truyền Vệ tinh Thủy hải (Sử dụng pre-staging):**
1. **Kiểm soát bóp băng thông (Throttling)**: Phải chừa ưu tiên để coi video streaming từ các cam trước. Lập giới hạn tải OTA ngầm tốc độ thấp để không chiếm sóng tàu.
2. **Lưu trú Ban Đêm:** Canh giờ lúc chập tối nhân công nghỉ làm, băng thông rảnh, lên lịch tự động chép các Image tệp khủng.
3. **S3 Transfer Acceleration:** Chống hạn chế độ trễ TCP thông qua mạng CDN edge cục bộ giảm ping.

---

## 9. Kế hoạch Những bước kế tiếp (Next Steps)

Cần thảo luận với phía Khách hàng / Tàu:
- Trích xuất chi tiết chính sách Cổng chặn Firewall 443 và 8883.
- Số lượng camera thực tế, Thông số CPU/GPU máy chủ chạy Node K3S tàu, Số Mbps cấu hình băng thông vệ tinh...

**Hoạt động 1:** Tổ chức Workshop lập định phương án cuối cùng của Kiến trúc trong vòng 2 tuần với đối tác R&R.
**Hoạt động 2:** Proof Of Concept (PoC) kênh kết nối mạng ống đồng Tunnel IoT AWS.
**Hoạt động 3:** Xây dựng khung MVP cấu phần Cập nhật OTA bằng Greengrass.
