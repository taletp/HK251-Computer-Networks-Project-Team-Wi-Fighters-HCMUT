# ⚡ Hệ thống cảnh báo sớm Port Scan

> 🚀 Một dự án nhỏ nhằm **phát hiện hoạt động quét cổng (port scan)** từ dữ liệu mạng quy mô lớn.  
> 🧩 Dự án gồm 2 phần chính: **Làm sạch dữ liệu (Clean Data)** và **Phát hiện tấn công (Detection Implementation)**.

---

## 🗂️ Cấu trúc dự án

```bash
PortScan-Early-Warning/
│
├── Clean_Data/
│   ├── json_array_to_csv_sample.py         # Script chuyển JSON → CSV (xử lý dữ liệu lớn bằng streaming)
│   ├── sample_clean_portscan.csv           # Dữ liệu mẫu đã làm sạch (1k–10k dòng)
│   └── extract_100k_sample.py (tùy chọn)   # Script trích mẫu nhỏ hơn từ file JSON gốc
│
├── Implementation/
│   └── detect_portscan_test.py             # Chương trình phát hiện port scan theo cửa sổ trượt
│
├── logs/
│   └── alerts.log                          # Log cảnh báo được tạo trong quá trình chạy
│
├── out/
│   └── alerts.csv                          # File kết quả cảnh báo (CSV)
│
├── README.md                               # File mô tả dự án
├── requirements.txt                        # Danh sách thư viện cần cài
└── raw_dataset.json                        # Dataset gốc tải từ IEEE DataPort hoặc Kaggle
```

📌 **Giải thích nhanh:**

- `Clean_Data/` chứa toàn bộ bước xử lý đầu vào.
- `Implementation/` là phần lõi của hệ thống phát hiện.
- `logs/` và `out/` được tạo tự động khi chạy.
- `raw_dataset.json` là dữ liệu gốc (dung lượng lớn, ~9GB).

---

## 🧭 Tổng quan

Hệ thống này mô phỏng quy trình phát hiện sớm (early warning) trong an ninh mạng gồm 3 giai đoạn:

1. 🧹 **Làm sạch dữ liệu (Data Cleaning)** — chuyển đổi file JSON lớn sang CSV phẳng dễ phân tích.
2. ⚙️ **Phát hiện Port Scan (Implementation)** — thống kê số lượng port được truy cập theo IP trong một cửa sổ thời gian.
3. 📊 **Phân tích & kết luận (Result)** — tổng hợp các cảnh báo và đánh giá kết quả.

---

## 🧩 Giai đoạn 1 — Làm sạch dữ liệu

### Mục tiêu:
Chuyển dữ liệu quét mạng định dạng JSON sang định dạng CSV có cấu trúc.

### 🔧 Script chính
`Clean_Data/json_array_to_csv_sample.py`

```python
import ijson, csv
from tqdm import tqdm

INPUT = "raw_dataset.json"
OUTPUT = "Clean_Data/sample_clean_portscan.csv"
MAX_ITEMS = 1000000  # số lượng host cần đọc (1 triệu mẫu)

FIELDS = ["ip", "timestamp", "port", "proto", "status", "reason", "ttl"]
...
```

### 💡 Cách hoạt động
- Sử dụng thư viện `ijson` để đọc file JSON cực lớn mà không cần tải hết vào RAM.
- Mỗi phần tử JSON chứa danh sách các port mở của một IP.
- Mỗi dòng CSV tương ứng một bản ghi `(ip, timestamp, port, proto, status, reason, ttl)`.

### 🧾 Kết quả đầu ra
```csv
ip,timestamp,port,proto,status,reason,ttl
165.221.32.138,1619562631,21,tcp,open,syn-ack,245
165.2.102.63,1619562631,8443,tcp,open,syn-ack,55
...
```

### 🧰 Công cụ sử dụng

| Thư viện | Chức năng |
|----------|-----------|
| `ijson` | Đọc file JSON lớn bằng streaming |
| `tqdm` | Hiển thị tiến độ xử lý |
| `csv` | Ghi dữ liệu ra file CSV |

---

## ⚙️ Giai đoạn 2 — Phát hiện Port Scan

### Mục tiêu:
Phát hiện các địa chỉ IP có dấu hiệu bị quét nhiều port trong một khoảng thời gian ngắn.

### 🔧 Script chính
`Implementation/detect_portscan_test.py`

### ⚙️ Cơ chế hoạt động
1. Đọc từng dòng trong file CSV → lấy `ip`, `timestamp`, `port`.
2. Nhóm dữ liệu theo IP và theo dõi bằng **cửa sổ thời gian trượt** (sliding window).
3. Đếm số lượng port khác nhau (`unique_ports`) xuất hiện trong khoảng thời gian `window`.
4. Nếu `unique_ports >= threshold` → sinh cảnh báo (alert).

### 🧠 Luồng xử lý
```
IP, Port, Timestamp, Proto
     │
     ▼
Nhóm theo IP
     │
     ▼
Cửa sổ thời gian 60 giây
     │
     ▼
Đếm unique_ports
     │
     ├─ < Threshold → Bỏ qua
     └─ ≥ Threshold → ⚠️ Cảnh báo
```

### 🧩 Tham số chạy

| Tham số | Ý nghĩa | Mặc định |
|---------|---------|----------|
| `--input` | Đường dẫn đến file CSV đã làm sạch | Bắt buộc |
| `--window` | Độ dài cửa sổ thời gian (giây) | 60 |
| `--threshold` | Số lượng port khác nhau để kích hoạt cảnh báo | 3 |
| `--max-states` | Số IP tối đa được theo dõi cùng lúc trong RAM | 200000 |
| `--out` / `--log` | Đường dẫn xuất kết quả và log | Tự động |

### 💻 Ví dụ chạy
```bash
python Implementation/detect_portscan_test.py \
  --input Clean_Data/sample_clean_portscan.csv \
  --window 60 --threshold 3 --max-states 200000 \
  --out out/alerts.csv \
  --log logs/alerts.log
```

---

## 📊 Kết quả đầu ra

### 🧾 File log cảnh báo
```
2025-10-23 17:06:05,929 WARNING [ALERT] ip=163.191.206.218 start=1619562643 end=1619562662 unique_ports=2
```

### 🧮 File CSV kết quả
```csv
alert_ts,ip,window_start,window_end,unique_ports,sample_rows
1729728365,163.191.206.218,1619562643,1619562662,2,400000
```

| Cột | Ý nghĩa |
|-----|---------|
| `alert_ts` | Thời điểm sinh cảnh báo |
| `ip` | IP bị nghi ngờ bị quét |
| `window_start` / `window_end` | Khoảng thời gian ghi nhận |
| `unique_ports` | Số port khác nhau trong khoảng 60s |
| `sample_rows` | Số dòng đã đọc khi phát hiện |

---

## 📦 requirements.txt

Lưu cùng cấp với `README.md`

```
ijson
tqdm
pandas
```

**Cài đặt thư viện:**

```bash
pip install -r requirements.txt
```

---

## 💡 Giải thích ngắn gọn

Hệ thống này mô phỏng cách hoạt động của **IDS/IPS** (Intrusion Detection System):

- **Dữ liệu đầu vào:** kết quả quét mạng quy mô lớn (Masscan, ZMap...).
- **Thuật toán:** đếm số lượng cổng khác nhau trong khoảng thời gian ngắn.
- **Đầu ra:** danh sách IP có dấu hiệu bị quét port.

---

## 👤 Tác giả

**Nguyễn Nhất Duy**  
Trường Đại học Bách Khoa TP.HCM – Khoa Khoa học và Kỹ thuật Máy tính  
📘 Đồ án Mạng Máy Tính – Hệ thống cảnh báo sớm Port Scan  
🕐 Tháng 10, 2025

---

## 🧭 Mục lục nhanh

| Mục | Nội dung |
|-----|----------|
| 🗂️ Cấu trúc dự án | Cấu trúc thư mục và mô tả các file |
| 🧩 Làm sạch dữ liệu | Tiền xử lý file JSON |
| ⚙️ Phát hiện Port Scan | Thuật toán phát hiện |
| 📊 Kết quả đầu ra | File log và CSV kết quả |
| 📦 requirements.txt | Các thư viện cần cài |
| 💡 Giải thích ngắn gọn | Ý nghĩa và ứng dụng |
