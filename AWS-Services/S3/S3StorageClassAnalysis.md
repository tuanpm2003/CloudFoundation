# Amazon S3 Storage Class Analysis — Tổng hợp kiến thức cốt lõi

## 1. Vấn đề bài học muốn giải quyết

Trong thực tế vận hành:

- Rất khó biết object nào đang được truy cập nhiều (Hot data).
- Rất khó biết object nào ít dùng (Warm/Cold data).
- Không thể tối ưu chi phí (cost) nếu không hiểu hành vi truy cập (access pattern).

> **Giải pháp:** AWS cung cấp công cụ **Amazon S3 Storage Class Analysis**.

---

## 2. Amazon S3 Storage Class Analysis là gì?

Amazon S3 Storage Class Analysis là một công cụ phân tích giúp:

- Monitor (theo dõi) access patterns của các object.
- Phân tích xem dữ liệu đang được sử dụng như thế nào.
- Xác định thời điểm thích hợp để chuyển object sang storage class rẻ hơn.
- Hỗ trợ tối đa cho việc tối ưu chi phí lưu trữ trên S3.

---

## 3. Mục tiêu chính của Storage Class Analysis

### 3.1. Hiểu Access Pattern

Giúp bạn biết chính xác:

- Object nào được dùng nhiều.
- Object nào ít dùng.
- Object nào gần như không bao giờ dùng.

### 3.2. Xác định phân loại dữ liệu

| Loại data     | Ý nghĩa                          |
| ------------- | -------------------------------- |
| **Hot Data**  | Truy cập thường xuyên, liên tục  |
| **Warm Data** | Thỉnh thoảng mới truy cập        |
| **Cold Data** | Hiếm khi hoặc không còn truy cập |

### 3.3. Tối ưu Storage Class

Dựa vào data để map với chuẩn lưu trữ phù hợp:

| Access Pattern                      | Storage Class phù hợp             |
| ----------------------------------- | --------------------------------- |
| Frequent access (Truy cập nhiều)    | **S3 Standard**                   |
| Infrequent access (Truy cập ít)     | **S3 Standard-IA**                |
| Rarely accessed (Hiếm truy cập)     | **S3 Glacier Flexible Retrieval** |
| Archive lâu dài (Lưu trữ vĩnh viễn) | **S3 Glacier Deep Archive**       |

---

## 4. Giá trị cốt lõi (Bắt buộc phải nhớ)

> ⚠️ **LƯU Ý QUAN TRỌNG:** Storage Class Analysis **KHÔNG** tự động di chuyển (move) data!

Nó chỉ làm nhiệm vụ: **Quan sát → Phân tích → Tạo report.**
Để thực sự di chuyển data, bạn **phải dùng Lifecycle Policy**.

### Flow thực tế:

```text
[1] Observe Access Pattern (Quan sát)
        ↓
[2] Analyze Data (Phân tích)
        ↓
[3] Generate Reports (Tạo báo cáo)
        ↓
[4] Create Lifecycle Policy (Tạo Rule)
        ↓
[5] Move To Cheaper Storage Class (Chuyển Data)
```

---

## 5. Storage Class Analysis hoạt động như thế nào?

Hệ thống AWS sẽ tự động:

1. Monitor object access (theo dõi lượt truy cập).
2. Phân tích dữ liệu theo thời gian.
3. Tạo visualization (biểu đồ trực quan).
4. Export report (xuất báo cáo) hàng ngày.

---

## 6. Các kiểu Configuration (Cấu hình phân tích)

### 6.1 Analyze Entire Bucket

- **Phân tích:** Toàn bộ object có trong bucket.
- **Use case:** Khi muốn có cái nhìn tổng quan (overview) cho toàn bộ bucket.

### 6.2 Analyze By Prefix

- **Ví dụ:** Lọc theo các thư mục `logs/`, `images/`, `videos/`, `backup/`
- **Use case:** Cho phép phân tích từng nhóm dữ liệu riêng biệt.

### 6.3 Analyze By Object Tags

- **Ví dụ:** Lọc theo tag `Environment=Production`, `Team=AI`, `Project=Media`
- **Use case:** Giúp phân tích chi phí/truy cập theo application, team hoặc project.

### 6.4 Combine Prefix + Tags

- **Use case:** Kết hợp cả 2 cách trên để phân tích chi tiết và sâu (granular) hơn.

---

## 7. Giới hạn quan trọng

> 📌 **Maximum:** AWS chỉ cho phép tạo tối đa **1000 filter configurations** trên mỗi bucket.

---

## 8. Export Reports (Xuất báo cáo)

Storage Class Analysis có thể:

- Export file **CSV** hàng ngày vào một S3 Bucket.
- Lưu lại lịch sử access pattern.
- Build **historical trend** (xu hướng lịch sử).

**Ý nghĩa cực kỳ quan trọng:**
Công cụ này không chỉ cho biết object hiện tại đang được access ra sao, mà còn cho biết **trend theo thời gian**.
_(Ví dụ: Tháng này ít dùng, nhưng tháng sau lại tăng đột biến)._

---

## 9. Báo cáo hoạt động như thế nào?

- Sau khi bật Analysis: AWS cần khoảng **24 giờ** để tạo ra report đầu tiên.
- Sau đó: Report sẽ được update **mỗi ngày**.

---

## 10. Nội dung có trong Report

Một file report CSV sẽ chứa các thông tin:

- **Object age:** Tuổi đời của object.
- **Access frequency:** Tần suất truy cập.
- **Storage usage:** Dung lượng đang sử dụng.
- **Historical trend:** Xu hướng truy cập trong quá khứ.

_Dữ liệu trong report được sắp xếp theo ngày và theo nhóm tuổi của object (object age group)._

---

## 11. Cần phân biệt rõ

Storage Class Analysis là một **Tool phân tích (Analytics Tool)**. Nó **KHÔNG PHẢI LÀ**:
❌ Monitoring realtime (Giám sát thời gian thực).
❌ Logging system (Hệ thống ghi log request).
❌ Automation tool (Công cụ tự động hóa di chuyển file).

---

## 12. Kết hợp với Lifecycle Policy

Storage Class Analysis sinh ra là để đi cặp với **S3 Lifecycle Policy**.

**Ví dụ thực tế:**

1. _Storage Class Analysis_ chỉ ra rằng: Các file trong thư mục `logs/` chỉ được truy cập nhiều trong 30 ngày đầu, sau đó không ai đụng tới.
2. _Hành động:_ Bạn tạo một **Lifecycle Policy**:
   - Sau 30 days → Chuyển xuống `Standard-IA`.
   - Sau 90 days → Chuyển xuống `Glacier`.
   - Sau 180 days → Chuyển xuống `Deep Archive`.

---

## 13. Kết hợp với Business Intelligence (BI) Tools

AWS cho phép export file CSV report để bạn đưa vào các công cụ BI phân tích chuyên sâu.
**Các công cụ phổ biến:**

- Amazon QuickSight (của AWS)
- Microsoft Power BI
- Tableau

### 14. Amazon QuickSight là gì?

Là dịch vụ BI (Business Intelligence) managed của AWS. Giúp:

- Visualize data (Trực quan hóa dữ liệu bằng biểu đồ).
- Build dashboards.
- Phân tích (Analytics) & Machine learning insights.

---

## 15. Tư duy AWS muốn truyền đạt

> **"Không tối ưu chi phí bằng cảm tính, hãy dùng Data."**

- ❌ **Không nên nghĩ:** _"Chắc file này ít dùng, move nó đi."_
- ✅ **Nên tư duy:** _"Storage Class Analysis cho thấy file này chỉ được access 1 lần trong 90 ngày qua. Nó đủ điều kiện để dùng Lifecycle policy chuyển sang Standard-IA."_

---

## 16. Tổng hợp Chức năng - Vai trò

| Chức năng                  | Vai trò                       |
| -------------------------- | ----------------------------- |
| **Monitor access pattern** | Quan sát hành vi (Usage)      |
| **Generate reports**       | Phân tích dữ liệu (Analytics) |
| **Optimize lifecycle**     | Giảm chi phí (Cost reduction) |
| **Improve visibility**     | Hiểu rõ Workload hệ thống     |

---

## 17. Phân biệt với các Service S3 khác

| Service                    | Vai trò cốt lõi                                     |
| -------------------------- | --------------------------------------------------- |
| **Storage Class Analysis** | Phân tích access pattern để tối ưu chi phí          |
| **S3 Inventory**           | Liệt kê danh sách object đang tồn tại               |
| **S3 Access Logging**      | Ghi log chi tiết từng request (Ai, IP nào, giờ nào) |
| **CloudTrail**             | Audit các API actions trên toàn AWS                 |
| **Lifecycle Policy**       | Tool thực thi việc tự động move/xóa data            |

---

## 18. Các câu hỏi phỏng vấn dễ gặp

**Q1: Storage Class Analysis dùng để làm gì?**

> Dùng để monitor object access pattern, từ đó xác định chính xác thời điểm nên move object sang các storage class rẻ hơn.

**Q2: Storage Class Analysis có tự động move object không?**

> **Không.** Nó chỉ analyze và generate reports. Việc move object là nhiệm vụ của **Lifecycle Policy**.

**Q3: Sau khi bật cấu hình, bao lâu thì có report đầu tiên?**

> Mất khoảng **24 hours**.

**Q4: Có thể filter (lọc) analysis theo các tiêu chí nào?**

> Có 4 cách: Entire bucket, Prefix, Object Tags, và kết hợp (Prefix + Tags).

---

## 19. Một câu nhớ nhanh toàn bài (Mindmap)

```text
Analyze Access Pattern  (Phân tích hành vi)
        ↓
Understand Data Usage   (Hiểu dữ liệu)
        ↓
Choose Storage Class    (Chọn chuẩn lưu trữ phù hợp)
        ↓
Reduce Storage Cost     (Giảm chi phí)
```

---

## 20. Keyword quan trọng cần nhớ

- `Access Pattern`
- `Hot Data` / `Warm Data` / `Cold Data`
- `Storage Class Analysis`
- `Lifecycle Policy`
- `Object Tags` & `Prefix Filters`
- `CSV Export` & `Historical Trend`
- `Business Intelligence (BI)`
- `Amazon QuickSight`
- `Cost Optimization`
