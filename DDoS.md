# HỆ THỐNG CẢNH BÁO SỚM TẤN CÔNG TỪ CHỐI DỊCH VỤ PHÂN TÁN (DDoS)

## 🚀 TÓM TẮT DỰ ÁN

Dự án nhỏ nhằm xây dựng một **mini-pipeline** phát hiện sớm các hoạt động **quét cổng** (port scanning) và các **hành vi mạng bất thường** tiềm ẩn. Đây là các dấu hiệu tiền thân phổ biến của tấn công DDoS quy mô lớn, sử dụng dữ liệu luồng mạng (network flow data).

* **Bộ dữ liệu**: NF-UNSW-NB15-v3 (NetFlow-based representation of UNSW-NB15) — định dạng CSV.
* **Công nghệ**: Python, tập trung vào việc tối ưu bộ nhớ và quy trình xử lý từng bước.

---

## 🧬 CẤU TRÚC PIPELINE

Dự án này trình bày một quy trình hoàn chỉnh từ dữ liệu luồng mạng thô đến các cảnh báo bảo mật, bao gồm 3 giai đoạn chính:

1.  **TIỀN XỬ LÝ DỮ LIỆU**: Chuẩn bị và làm sạch bộ dữ liệu NF-UNSW-NB15-v3.
2.  **PHÁT HIỆN SỚM**: Triển khai các thuật toán phân tích lưu lượng, xác định hành vi quét cổng và các hoạt động bất thường.
3.  **ĐÁNH GIÁ & KẾT LUẬN**: Phân tích kết quả, thảo luận về các hạn chế và đề xuất cải tiến.

---

## 🧹 BƯỚC 1: TIỀN XỬ LÝ & LÀM SẠCH DỮ LIỆU

### 1.1. Nguồn Dữ Liệu & Mục Tiêu

* **Bộ dữ liệu**: NF-UNSW-NB15-v3 (Các tệp CSV chứa dữ liệu luồng mạng).
* **Nguồn tham khảo**: [Link NIDS Datasets](https://staff.itee.uq.edu.au/marius/NIDS_datasets/)
* **Mục tiêu**: Chuyển đổi dữ liệu luồng mạng thô thành một tập dữ liệu **sẵn sàng cho phân tích** (*analysis-ready*). Tập trung vào các trường cần thiết để phát hiện quét cổng (ví dụ: IP nguồn/đích, cổng đích, số lượng luồng, v.v.).

### 1.2. Công Cụ & Thư Viện

* **Yêu cầu**: Python 3.x
* **Thư viện**: `pandas`, `numpy`, `tqdm` (để theo dõi tiến độ xử lý tệp lớn).

### 1.3. Cấu Trúc Đặc Trưng (Features) NF-UNSW-NB15-v3

Bộ dữ liệu chứa các đặc trưng của luồng mạng được trích xuất (ví dụ):

* `IPV4_SRC_ADDR`, `IPV4_DST_ADDR` (Địa chỉ IP nguồn/đích)
* `L4_DST_PORT` (Cổng đích lớp 4)
* `IN_BYTES`, `OUT_BYTES` (Lượng byte vào/ra)
* `FLOW_DURATION_MILLISECONDS` (Thời lượng luồng)
* `Attack` (Nhãn: 0 cho Bình thường, 1 cho Tấn công)
* ... và các đặc trưng liên quan đến tần suất, sự đa dạng của luồng.

> *Lưu ý: Với kích thước tệp lớn, cần sử dụng **kiểu dữ liệu tối ưu** trong Pandas hoặc **xử lý theo khối (chunking)** để tiết kiệm bộ nhớ.*

### 1.4. Task 1: Tải và Làm Sạch Dữ Liệu

**Tóm tắt các bước thực hiện:**
1.  Tải (Load) các tệp dataset CSV vào DataFrame (sử dụng tài nguyên Google Colab/Drive).
2.  Kiểm tra thông tin DataFrame (`.info()`) và xử lý các **giá trị bị thiếu** (missing values).
3.  **Mã hóa** các đặc trưng phân loại (Categorical Encoding).
4.  **Chuẩn hóa** các đặc trưng số (Feature Scaling).
5.  Kết quả cuối cùng là một tập dữ liệu đã được tiền xử lý.

### 1.5. Task 2: Chia Mẫu Dữ Liệu

**Tóm tắt các bước thực hiện:**
1.  **Tách nhãn**: Phân chia tập dữ liệu thành các đặc trưng ($X$) và nhãn ($y$).
2.  **Cân bằng lớp**: Tiến hành **undersampling** hoặc **oversampling** (nếu cần) do tính mất cân bằng nghiêm trọng giữa lớp tấn công và lớp bình thường.
3.  **Phân chia**: Tách $X, y$ thành tập **Huấn luyện** (Training), **Kiểm tra** (Testing), và **Thẩm định** (Validation) theo tỷ lệ hợp lý (ví dụ: 70/15/15).

---

## 🧠 BƯỚC 2: HUẤN LUYỆN MÔ HÌNH HỌC SÂU ĐƠN GIẢN

### 2.1. Mục Tiêu

Xây dựng và huấn luyện một mô hình học sâu (ví dụ: **Mạng nơ-ron truyền thẳng - FNN/MLP**) để phân loại luồng mạng thành *bình thường* (0) hoặc *tấn công/bất thường* (1), dựa trên các đặc trưng đã tiền xử lý.

### 2.2. Công Cụ và Thư Viện

* **Thư viện**: `TensorFlow` / `Keras`, `scikit-learn` (cho các metric đánh giá).

### 2.3. Nội Dung Tóm Tắt

1.  **Thiết kế mô hình**: Xây dựng cấu trúc mạng nơ-ron (số lớp, số nơ-ron, hàm kích hoạt).
2.  **Huấn luyện**: Sử dụng tập huấn luyện để tối ưu hóa trọng số.
3.  **Đánh giá**: Sử dụng tập kiểm tra để đánh giá hiệu suất mô hình (Accuracy, Precision, Recall, F1-score).

### 2.4. Thực Hiện
* *Phần này sẽ chứa mã code Python/Keras chi tiết cho việc xây dựng và huấn luyện mô hình.*
