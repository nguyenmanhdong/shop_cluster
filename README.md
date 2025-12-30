# README – Phân khúc khách hàng dựa trên Luật kết hợp & RFM
## 👥 Thông tin Nhóm

- **Nhóm:** Nhóm 5
- **Thành viên:**
  - Phan Việt Hùng
  - Nguyễn Mạnh Đông
  - Trần Minh Thành
## 🎯 Mục tiêu

Mini Project này nhằm xây dựng một **pipeline phân khúc khách hàng hoàn chỉnh** theo hướng:

**Luật kết hợp → Đặc trưng hành vi mua kèm → Phân cụm khách hàng → Diễn giải → Đề xuất chiến lược marketing**.

Cụ thể, nhóm hướng tới các mục tiêu sau:

* Khai phá các **mối quan hệ mua kèm (association rules)** có ý nghĩa từ dữ liệu giao dịch.
* Biến các luật kết hợp thành **đặc trưng hành vi** cho từng khách hàng.
* So sánh **nhiều biến thể feature engineering** (baseline vs nâng cao).
* Đánh giá và trực quan hóa kết quả phân cụm bằng các chỉ số và PCA 2D.
* Thực hiện **profiling và diễn giải cụm** gắn với giá trị kinh doanh.
* Đề xuất **chiến lược marketing cụ thể** cho từng nhóm khách hàng.

---

## 1. Khai phá luật kết hợp (Association Rule Mining)

### 1.1 Phương pháp

Nhóm sử dụng thuật toán **FP-Growth** để khai phá luật kết hợp từ bộ dữ liệu **Online Retail**. Dữ liệu giao dịch được tiền xử lý và chuyển sang dạng **basket format** trước khi sinh luật.

Tổng số luật sinh ra ban đầu: **3,856 luật**.

### 1.2 Quy trình lọc luật

Để đảm bảo chất lượng và khả năng diễn giải, nhóm áp dụng các tiêu chí lọc:

* **min_support**: loại bỏ các luật xuất hiện quá ít.
* **min_confidence**: đảm bảo xác suất xảy ra consequent đủ lớn khi antecedent xuất hiện.
* **min_lift > 1**: chỉ giữ các luật có mối quan hệ mua kèm có ý nghĩa.
* Giới hạn độ dài antecedent/consequent để tránh luật quá phức tạp.

Sau lọc, số luật giảm từ **3,856 → 1,794 luật**, giữ lại phần lớn các quan hệ mua kèm quan trọng.

### 1.3 Tiêu chí lựa chọn luật cho phân cụm

Từ tập luật đã lọc, nhóm:

* Sắp xếp theo **lift giảm dần**.
* Chọn **Top-100 luật** làm đầu vào cho phân cụm.

**Lý do lựa chọn lift & Top-100:**

* Lift phản ánh độ mạnh thực sự của mối quan hệ mua kèm.
* Tránh đưa quá nhiều luật gây nhiễu và tăng số chiều không cần thiết.
* Đảm bảo cân bằng giữa **độ phong phú hành vi** và **khả năng diễn giải**.

### 1.4 Các luật tiêu biểu

| Antecedents                               | Consequents          | Support | Confidence | Lift  |
| ----------------------------------------- | -------------------- | ------- | ---------- | ----- |
| HERB MARKER PARSLEY, HERB MARKER ROSEMARY | HERB MARKER THYME    | 0.0109  | 0.9517     | 74.57 |
| HERB MARKER MINT, HERB MARKER THYME       | HERB MARKER ROSEMARY | 0.0106  | 0.9550     | 74.50 |
| HERB MARKER BASIL, HERB MARKER THYME      | HERB MARKER ROSEMARY | 0.0107  | 0.9507     | 74.17 |

👉 Các luật đều có **lift rất cao**, cho thấy mối liên kết mua kèm mạnh mẽ giữa các sản phẩm cùng dòng.

---



## 2. Feature Engineering cho phân cụm

Nhóm xây dựng **hai biến thể đặc trưng** để so sánh.

### 2.1 Biến thể 1 – Baseline (Rule-based Binary Features)

**Cấu hình:**

```bash
RULE_FEATURE_TYPE=binary   # chỉ đánh dấu có / không thỏa luật
TOP_K_RULES=200
USE_RFM=false
RFM_SCALE=false
RULE_SCALE=false
MIN_ANTECEDENT_LEN=2
```

**Không gian đặc trưng:**

* Shape X: **(3921 × 175)**
* Chỉ sử dụng rule-features dạng nhị phân

**Kết quả phân cụm:**

* Silhouette cao nhất tại **k = 2**, score ≈ **0.56**
* Phân tách được nhóm mua nhiều và mua ít, nhưng mức độ chưa rõ ràng

---

### 2.2 Biến thể 2 – Rule + RFM (Weighted Features)

Đây là **biến thể được lựa chọn chính thức** cho các bước phân tích tiếp theo.

**Cấu hình:**

```bash
RULE_FEATURE_TYPE=weighted  # lift × confidence
TOP_K_RULES=200
USE_RFM=true
RFM_SCALE=true
RULE_SCALE=false
MIN_ANTECEDENT_LEN=2
```

**Không gian đặc trưng:**

* Shape X: **(3921 × 203)**
* Rule-features có trọng số + RFM chuẩn hóa
  
**Kết quả phân cụm:**

* Silhouette cao nhất tại **k = 2**, score ≈ **0.96**
* Phân tách được nhóm mua nhiều và mua ít, phân cụm rõ ràng
  
**Ưu điểm:**

* Giữ được cường độ hành vi mua kèm (thông qua lift & confidence)
* Kết hợp giá trị khách hàng (RFM) → tăng khả năng diễn giải
* Phù hợp cho profiling & marketing action

---
