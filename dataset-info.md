# METADATA: INTRUSION DETECTION EVALUATION DATASET (CIC-IDS2017)

---

## 1. Thông tin Cơ bản (General Information)

| Thuộc tính | Mô tả |
| :--- | :--- |
| **Tên Dataset** | Intrusion Detection Evaluation Dataset (CIC-IDS2017) |
| **Nguồn Gốc** | Canadian Institute for Cybersecurity (CIC), University of New Brunswick (UNB) |
| **Năm Phát hành** | 2017 |
| **Mục đích** | Cung cấp dữ liệu chuẩn mực để đánh giá các Hệ thống Phát hiện Xâm nhập (IDS/IPS), đặc biệt cho các ứng dụng Machine Learning. |
| **Giấy phép** | Miễn phí cho mục đích nghiên cứu và giáo dục. Yêu cầu trích dẫn nguồn gốc. |

---

## 2. Cấu trúc Dữ liệu và Định dạng (Data Structure and Format)

| Thuộc tính | Mô tả chi tiết |
| :--- | :--- |
| **Tổng kích thước** | Khoảng 3.2 triệu bản ghi (network flows). |
| **Định dạng File** | **CSV** (cho Machine Learning) và **PCAP** (dữ liệu gói tin thô). |
| **Phân chia** | Chia thành **5 file lớn**, tương ứng với 5 ngày thu thập dữ liệu (Thứ Hai đến Thứ Sáu). |
| **Đơn vị Bản ghi** | Mỗi hàng trong CSV đại diện cho một **Lưu lượng mạng hai chiều (Bidirectional Flow)**. |
| **Số lượng Features** | **80 Features** được trích xuất bằng **CICFlowMeter**. |

---

## 3. Các Cột Tính năng và Nhãn Chính (Key Features and Labels)

| Tên Cột | Loại | Mục đích |
| :--- | :--- | :--- |
| **Flow ID** | Định danh | Định danh duy nhất cho flow. |
| **Source IP, Destination IP** | Định danh | Địa chỉ IP của nguồn và đích. |
| **Flow Duration** | Định lượng | Thời gian tồn tại của flow. |
| **Total Fwd Packets, Total Backward Packets** | Định lượng | Tổng số gói tin gửi đi và nhận về. |
| **FIN/SYN/ACK Flag Count** | Định lượng | Đếm các cờ TCP (quan trọng cho các kiểu Scan và DDoS). |
| **Label** | Phân loại | **Chứa nhãn phân loại:** `BENIGN` (Bình thường) hoặc tên cụ thể của cuộc tấn công. |

---

## 4. Các Loại Tấn công (Attack Taxonomy)

Dataset bao gồm các loại tấn công mạng hiện đại được mô phỏng:
|STT | Tên loại | Số lượng |
| :--- | :--- | :--- |
|1 |DoS Hulk             |          231073|
|2 |PortScan              |         158930|
|3 |DDoS                 |          128027|
|4 |DoS GoldenEye         |          10293|
|5 |FTP-Patator           |           7938|
|6 |SSH-Patator           |           5897|
|7 |DoS slowloris          |          5796|
|8 |DoS Slowhttptest        |         5499|
|9 |Bot                      |         1966|
|10 |Web Attack – Brute Force  |       1507|
|11 |Web Attack – XSS         |         652|
|12 |Infiltration              |         36|
|13 |Web Attack – Sql Injection  |       21|
|14 |Heartbleed                |         11|

---

## 5. Trích dẫn
> **Iman Sharafaldin, Arash Habibi Lashkari, and Ali A. Ghorbani, “Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization”, 4th International Conference on Information Systems Security and Privacy (ICISSP), Portugal, January 2018.**
