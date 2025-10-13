# 📄 GUIDELINE ĐỒ ÁN MẠNG MÁY TÍNH: CẢNH BÁO SỚM TẤN CÔNG TỔNG HỢP (DL-IDS)

---

## 🎯 1. MỤC TIÊU VÀ PHẠM VI DỰ ÁN

* **Mục tiêu:** Xây dựng mô hình Học sâu Đa đầu vào (Multi-Input Deep Learning) để phân loại và cảnh báo sớm các loại tấn công mạng (DDoS, Quét cổng, Rò rỉ dữ liệu) từ các bộ dữ liệu NIDS có sẵn.
* **Chiến lược Dữ liệu:** Sử dụng các datasets NIDS công khai (ví dụ: CICIDS, UNSW-NB15) thay vì thu thập log thủ công để tập trung tối đa vào **Mô hình và Đánh giá rủi ro**.

---

## ⚙️ 2. PHÂN CÔNG NHIỆM VỤ CHI TIẾT

| Giai đoạn | Nhiệm vụ Chính | Thành viên Phụ trách(s) |
| :--- | :--- | :--- |
| **GĐ 1** | **Thu thập & Phân tích Dữ liệu** | [Tên TV] |
| **GĐ 2** | **Tiền xử lý & Kỹ thuật Đặc trưng** | [Tên TV] |
| **GĐ 3** | **Xây dựng & Huấn luyện Mô hình DL** | [Tên TV] |
| **GĐ 4** | **Tính toán & Phân loại Rủi ro (Risk Scoring)** | [Tên TV] |
| **GĐ 5** | **Triển khai & Vận hành Thử nghiệm** | [Tên TV] |


---

## 🚀 3. HƯỚNG DẪN THỰC HIỆN CỤ THỂ

### 3.1. GIAI ĐOẠN 1: Thu thập & Phân tích Dữ liệu

* **1.1. Lựa chọn Dataset:** Chọn ít nhất **hai (2)** bộ dữ liệu NIDS lớn (ví dụ: CICIDS 2018 và UNSW-NB15) để tăng tính tổng hợp của mô hình. Có thể sử dụng các bộ dữ liệu khác nếu đảm bảo tính khách quan và tránh sai lệch mẫu.
* **1.2. Phân tích Sơ bộ:**
    * Xác định tổng số **Features** (cột) và **Labels** (nhãn tấn công).
    * Phân tích **Class Imbalance** (sự mất cân bằng lớp) để chuẩn bị cho GĐ 2.
    * Đảm bảo dữ liệu đã được **gộp lại** thành một cấu trúc bảng.

### 3.2. GIAI ĐOẠN 2: Tiền xử lý & Kỹ thuật Đặc trưng

* **2.1. Phân nhóm Đầu vào (Multi-Input Creation):**
    * Chia toàn bộ Features thành ít nhất **3 nhóm logic** dựa trên ý nghĩa (ví dụ: **Statistical**, **Protocol/Port**, **Time/Sequential**).
    * **Mục tiêu:** Mỗi nhóm sẽ là một đầu vào riêng biệt cho mô hình DL.
* **2.2. Chuẩn hóa/Mã hóa:**
    * Áp dụng **MinMaxScaler** cho các đặc trưng số.
    * Xử lý các giá trị bị thiếu (Missing Values) bằng cách điền **Mean/Median** hoặc loại bỏ.
    * Mã hóa các đặc trưng phân loại (Categorical Features) như Giao thức (`Protocol`) bằng **One-Hot Encoding**.
* **2.3. Xử lý Mất cân bằng:** Áp dụng kỹ thuật cân bằng lớp như **SMOTE** (Oversampling) hoặc **Undersampling** để cải thiện hiệu suất phát hiện các lớp tấn công nhỏ.
* **2.4. Phân chia Tập:** Chia dữ liệu thành **Training, Validation, Testing** (Ví dụ: 70/15/15).

### 3.3. GIAI ĐOẠN 3: Xây dựng & Huấn luyện Mô hình Deep Learning

* **3.1. Thiết kế Mô hình:**
    * Xây dựng kiến trúc **3 nhánh độc lập** tương ứng với 3 nhóm đầu vào từ GĐ 2 (sử dụng thư viện **Keras/TensorFlow**).
    * Sử dụng lớp **LSTM/GRU** cho nhánh **Time/Sequential** (nếu có thể sắp xếp dữ liệu thành chuỗi).
    * Sử dụng lớp **Dense (MLP)** cho các nhánh còn lại.
* **3.2. Nối (Concatenation):** Hợp nhất đầu ra của 3 nhánh lại.
* **3.3. Huấn luyện:**
    * Sử dụng hàm mất mát **Binary/Categorical Cross-entropy**.
    * Sử dụng tập Validation để **tinh chỉnh Hyperparameters** (Batch Size, Learning Rate).
* **3.4. Đánh giá Ban đầu:** Sử dụng tập Testing để tính **Accuracy, Precision, Recall, F1-score**.

### 3.4. GIAI ĐOẠN 4: Tính toán & Phân loại Rủi ro

* **4.1. Công thức Risk Score:** Thiết lập công thức chính:
  
$$
\text{Risk Score} = (W_{\text{Severity}} \times P_{\text{Attack}})
$$

* **4.2. Trọng số (Severity Weights):** Gán trọng số $W_{\text{Severity}}$ cho từng loại tấn công (VD: DDoS=0.9, Rò rỉ=0.7, Quét cổng=0.4).
* **4.3. Phân loại Ưu tiên:** Thiết lập các ngưỡng cụ thể dựa trên Risk Score (ví dụ: >0.8 = CRITICAL, 0.6-0.79 = HIGH).
* **4.4. Module Cảnh báo:** Xây dựng hàm Python mô phỏng việc đưa ra cảnh báo bao gồm: **Loại Tấn công, Confidence Score ($P_{\text{Attack}}$), Risk Score và Mức Ưu tiên**.

### 3.5. GIAI ĐOẠN 5: Triển khai & Vận hành Thử nghiệm

* **5.1. Phục vụ Mô hình (Model Serving):** Xây dựng một **API Inference** (sử dụng Flask/FastAPI) để nhận 3 đầu vào và trả về $P_{\text{Attack}}$.
* **5.2. Đóng gói Container:** Sử dụng **Docker** để đóng gói API và các dependencies liên quan.
* **5.3. Giả lập Data Stream:** Sử dụng tập **Testing** để tạo một luồng dữ liệu giả lập (mock stream). Gửi từng mẫu dữ liệu đến API đã triển khai để nhận kết quả.
* **5.4. Tích hợp & Ghi nhận Cảnh báo:** Tích hợp logic **Risk Score** (GĐ 4) vào quy trình API. Ghi lại các cảnh báo cuối cùng (bao gồm Risk Score và Mức Ưu tiên) vào file log/DB.
* **5.5. Đánh giá Cuối cùng:** Đánh giá hiệu suất của toàn bộ pipeline (End-to-End) trên luồng dữ liệu giả lập.

---

## 📅 4. HẠN CHÓT (DEADLINE)

* **GĐ 1 & 2 (Dữ liệu):** [Ngày cụ thể]
* **GĐ 3 (Mô hình DL):** [Ngày cụ thể]
* **GĐ 4 (Risk Scoring):** [Ngày cụ thể]
* **GĐ 5 (Triển khai Thử nghiệm):** [Ngày cụ thể]
