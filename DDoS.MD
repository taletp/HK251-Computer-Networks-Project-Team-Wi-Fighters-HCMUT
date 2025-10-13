# 🔍 Hệ thống cảnh báo sớm Tấn công từ chối dịch vụ phân tán (DDoS)
> 🚀 Dự án nhỏ nhằm phát hiện các hoạt động quét cổng và các hành vi bất thường tiềm ẩn, thường là dấu hiệu tiền thân của tấn công DDoS, sử dụng dữ liệu luồng mạng quy mô lớn.
> 💡 Bộ dữ liệu được sử dụng: NF-UNSW-NB15-v3 (NetFlow-based representation of UNSW-NB15) — định dạng CSV.
> 🧪 Toàn bộ quá trình xử lý và triển khai được thực hiện bằng Python (tối ưu bộ nhớ và từng bước).

# 🧬 Tổng quan
Dự án này trình bày một mini-pipeline hoàn chỉnh từ dữ liệu luồng mạng thô đến các cảnh báo bảo mật:

1. 🧹 Tiền xử lý dữ liệu — Chuẩn bị và làm sạch bộ dữ liệu NF-UNSW-NB15-v3 (CSV).

2. ⚙️ Triển khai / Phát hiện — Phân tích các mô hình lưu lượng, xác định hành vi quét cổng và các hoạt động bất thường.

3. 📊 Kết quả & Kết luận — Đánh giá phát hiện, thảo luận các hạn chế và cải tiến trong tương lai.

## 🧹 Bước 1 — Làm sạch dữ liệu (Tiền xử lý)
Bộ dữ liệu: NF-UNSW-NB15-v3 (Các tệp CSV chứa dữ liệu luồng mạng)
Nguồn dữ liệu: [](https://staff.itee.uq.edu.au/marius/NIDS_datasets/)


### 🎯 Mục tiêu
Chuyển đổi dữ liệu luồng mạng NF-UNSW-NB15-v3 thành một tập dữ liệu sẵn sàng cho phân tích (analysis-ready), tập trung vào các trường cần thiết để phát hiện hành vi quét cổng (ví dụ: IP nguồn/đích, cổng đích, số lượng luồng, v.v.).
