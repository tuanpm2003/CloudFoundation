# Amazon S3 Cost Optimization — Tổng hợp kiến thức cốt lõi

## 1. Mục tiêu của tối ưu chi phí trên Amazon S3

Tối ưu chi phí không chỉ là giảm tiền lưu trữ mà còn nhằm:

- Chọn đúng storage class cho đúng loại dữ liệu.
- Đảm bảo hiệu năng phù hợp với ứng dụng.
- Tự động hóa việc quản lý dữ liệu.
- Theo dõi và tối ưu liên tục.
- Tăng khả năng quản trị và quan sát hệ thống.

---

## 2. Four Pillars of S3 Cost Optimization (4 Trụ cột tối ưu)

### 2.1 Application Requirements

Cần hiểu rõ yêu cầu của ứng dụng:

- Tần suất truy cập dữ liệu
- Độ trễ (Latency)
- Hiệu năng cần thiết
- Thời gian lưu trữ dữ liệu

**Ví dụ:**

| Use Case               | Yêu cầu                     |
| ---------------------- | --------------------------- |
| **Website production** | Truy cập nhanh, độ trễ thấp |
| **Backup lâu dài**     | Chi phí thấp                |
| **Archive compliance** | Lưu trữ rất lâu, an toàn    |

### 2.2 Data Organization

Tổ chức dữ liệu hợp lý giúp:

- Dễ quản lý chi phí.
- Theo dõi usage theo team/project.
- Quan sát access pattern.
- Tăng visibility (Khả năng quan sát).

**Các công cụ hỗ trợ:**

- Object Tags
- Cost Allocation Tags

**Ví dụ về Tags:**

```text
Environment = Production
Team = Backend
Project = MediaSystem
```

### 2.3 Understand, Analyze and Optimize

Cần thực hiện liên tục các công việc:

- Quan sát access pattern.
- Phân tích usage.
- Điều chỉnh storage class.
- Tối ưu lifecycle policy.

**AWS cung cấp các công cụ hỗ trợ:**

- S3 Storage Class Analysis
- S3 Storage Lens
- Amazon CloudWatch
- AWS Budgets
- AWS Cost and Usage Reports

### 2.4 Continuous Right Sizing

**Right sizing** = Chọn đúng storage class cho dữ liệu. Không phải mọi dữ liệu đều nên dùng S3 Standard.

| Loại dữ liệu              | Storage Class phù hợp             |
| ------------------------- | --------------------------------- |
| Truy cập thường xuyên     | **S3 Standard**                   |
| Ít truy cập               | **S3 Standard-IA**                |
| Lưu trữ lâu dài (Archive) | **S3 Glacier Flexible Retrieval** |
| Pattern khó đoán          | **S3 Intelligent-Tiering**        |

---

## 3. Amazon S3 Storage Classes

**Ý nghĩa:** AWS tạo nhiều storage class để khách hàng có thể cân bằng giữa các yếu tố:

- **Cost:** Chi phí lưu trữ.
- **Performance:** Hiệu năng truy xuất.
- **Availability:** Độ sẵn sàng.
- **Retrieval Time:** Thời gian lấy lại dữ liệu.

---

## 4. Amazon S3 Intelligent-Tiering

### Mục đích

Tự động tối ưu chi phí khi _access pattern_ thay đổi. AWS sẽ tự động:

- Monitor (theo dõi) access pattern.
- Move object (chuyển đổi) giữa các tier.
- Giảm chi phí lưu trữ tối đa.

### Các tier trong Intelligent-Tiering

| Tier                       | Mục đích                           |
| -------------------------- | ---------------------------------- |
| **Frequent Access**        | Truy cập thường xuyên              |
| **Infrequent Access**      | Truy cập ít                        |
| **Archive Instant Access** | Hiếm truy cập (nhưng cần lấy ngay) |
| **Archive Access**         | Không truy cập > 90 ngày           |
| **Deep Archive Access**    | Không truy cập > 180 ngày          |

### Ưu điểm

- Không cần cấu hình Lifecycle Policy phức tạp.
- Tự động optimize.
- **Không có phí truy xuất** (No retrieval charge).
- Không yêu cầu thời gian lưu trữ tối thiểu (No minimum storage duration).
- Hiệu năng tương tự S3 Standard.

### Khi nào nên dùng?

- Access pattern không thể đoán trước (Unpredictable).
- Workload thay đổi liên tục.
- Team không có thời gian/nhân lực để optimize thủ công.

---

## 5. S3 Storage Class Analysis

### Mục đích

Dùng để theo dõi:

- Object nào đang được access.
- Tần suất access.
- Access pattern thay đổi theo thời gian.

**Mục tiêu đầu ra:**

- Tạo Lifecycle policy phù hợp.
- Chuyển dữ liệu sang storage class rẻ hơn.

### Flow hoạt động

```text
[1] Observe access pattern
      ↓
[2] Analyze
      ↓
[3] Create Lifecycle Policy
      ↓
[4] Move to cheaper storage class
```

---

## 6. Lifecycle Policy

### Mục đích

Tự động chuyển đổi các object giữa các storage classes dựa trên thời gian tạo. Là công cụ tối ưu chi phí cực kỳ quan trọng trên S3.

**Ví dụ luồng chuyển đổi:**

```text
Ngày 0   : Lưu ở S3 Standard
Sau 30 ngày  → Chuyển xuống S3 Standard-IA
Sau 90 ngày  → Chuyển xuống S3 Glacier
Sau 180 ngày → Chuyển xuống S3 Glacier Deep Archive
```

---

## 7. Công cụ quản lý và Monitoring

### 7.1 Amazon S3 Inventory

Dùng để:

- Liệt kê các object có trong bucket.
- Audit encryption (Kiểm tra mã hóa).
- Audit replication (Kiểm tra nhân bản).

### 7.2 Amazon S3 Server Access Logging

Theo dõi:

- Ai truy cập object?
- Request nào được gửi?
- Tần suất truy cập?

_Dùng cho: Security audit, Usage analysis, Billing analysis._

### 7.3 AWS CloudTrail

Theo dõi:

- API calls.
- Console actions.
- SDK/CLI activity.

_Mục đích: Governance, Compliance, Security auditing._

---

## 8. Phân biệt các công cụ Logging/Monitoring

| Tool               | Vai trò cốt lõi                            |
| ------------------ | ------------------------------------------ |
| **S3 Inventory**   | Biết object nào đang tồn tại trong bucket  |
| **Access Logging** | Biết object nào đang được truy cập         |
| **CloudTrail**     | Biết **AI** (who) thực hiện action/API nào |

---

## 9. Monitoring và Cost Management

- **Amazon CloudWatch:** Theo dõi metrics, performance, thiết lập alarms, theo dõi operational health.
- **AWS Budgets:** Set giới hạn ngân sách (budget limit), cảnh báo (alert) khi chi phí gần vượt ngân sách.
- **AWS Cost and Usage Reports (CUR):** Cung cấp billing reports, usage reports và phân tích chi phí chi tiết nhất.
- **Amazon S3 Storage Lens:** Quan sát usage trên toàn organization, xem xu hướng hoạt động (activity trends), nhận recommendations để tối ưu chi phí.

---

## 10. Tư duy cốt lõi cần nhớ

AWS muốn bạn nắm vững 4 bước tư duy:

1. **Hiểu dữ liệu:** Data đang "nóng" (truy cập nhiều) hay "lạnh" (ít truy cập)?
2. **Quan sát access pattern liên tục:** Usage luôn thay đổi theo thời gian.
3. **Tự động hóa:** Sử dụng `Lifecycle Policy` hoặc `Intelligent-Tiering`.
4. **Chọn đúng storage class:** Không phải mọi data đều cần hiệu năng truy xuất cao nhất.

---

## 11. Các câu hỏi phỏng vấn thường gặp

**Q1: Vì sao dùng Intelligent-Tiering?**

> Khi access pattern không thể đoán trước (unpredictable) và muốn hệ thống tự động tối ưu chi phí mà không tốn công quản lý.

**Q2: Storage Class Analysis dùng để làm gì?**

> Phân tích access pattern của dữ liệu. Dựa vào đó để xây dựng Lifecycle policy và chọn storage class rẻ hơn, phù hợp hơn.

**Q3: Sự khác biệt giữa S3 Inventory và Access Logging?**

| S3 Inventory                                       | Access Logging                                              |
| -------------------------------------------------- | ----------------------------------------------------------- |
| Cho biết **những gì đang có** (object nào tồn tại) | Cho biết **những gì đang diễn ra** (object nào được access) |

**Q4: Lifecycle Policy dùng để làm gì?**

> Tự động di chuyển object sang các storage class rẻ hơn (hoặc xóa chúng) dựa trên độ tuổi (thời gian) của object.

---

## 12. Tổng kết bài học (Workflow)

```text
[1] Observe Data
      ↓
[2] Analyze Access Pattern
      ↓
[3] Choose Correct Storage Class
      ↓
[4] Automate Lifecycle
      ↓
[5] Continuously Optimize Cost
```

---

## 13. Các Keyword quan trọng cần nhớ

- **Right Sizing:** Chọn đúng kích cỡ/chuẩn lưu trữ.
- **Access Pattern:** Hành vi truy cập dữ liệu.
- **Storage Class:** Phân loại lưu trữ trên S3.
- **Lifecycle Policy:** Chính sách vòng đời dữ liệu.
- **Intelligent-Tiering:** Phân tầng lưu trữ thông minh.
- **Cost Optimization:** Tối ưu hóa chi phí.
- **S3 Storage Lens:** Thấu kính quan sát không gian lưu trữ S3.
- **S3 Inventory:** Kiểm kê S3.
- **CloudTrail:** Theo dõi lịch sử API.
- **Access Logging:** Ghi log truy cập S3.
- **Monitoring & Automation:** Giám sát và tự động hóa.

```

```
