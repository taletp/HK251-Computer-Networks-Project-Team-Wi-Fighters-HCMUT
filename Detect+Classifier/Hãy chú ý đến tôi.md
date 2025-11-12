# Đây là project nhỏ về việc triển khai hệ thống detect tấn công mạng tổng hợp đa đầu vào. 
Chúng tôi đã tham khảo về bài viết A Novel Two-Stage Deep Learning Model for Network Intrusion Detection: LSTM-AE để tham khảo cách thiết kế 1 hệ thống Classifier giúp phân loại các loại tấn công. 

## Lý do chọn

| Tiêu chí                            | Hệ thống của nhóm                                                    | Mô hình LSTM-AE đề xuất                                                                                            | Nhận xét mức độ phù hợp                                                                           |
| ----------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| **Mục tiêu chung**                  | Phát hiện sớm & cảnh báo tấn công mạng (DDoS, Port Scan, …)                  | Phát hiện bất thường (Intrusion / Anomaly Detection) bằng deep learning                                            | ✅ **Rất phù hợp** — cùng hướng “early warning & detection”                                        |
| **Nguồn dữ liệu đầu vào**           | Dữ liệu thu từ Sensor (PCAP, Netflow, Log, metrics từ router, server)        | Dữ liệu flow hoặc traffic metrics (CICIDS2017, CSE-CIC-IDS2018, tương đương dữ liệu nhóm có thể thu từ Sensor) | ✅ Hoàn toàn tương thích (cùng dạng numeric, time-based features)                                  |
| **Bước tiền xử lý (Preprocessing)** | Window Aggregation, tính thống kê (packets/sec, SYN rate, byte/sec)          | Missing Value Handling, Normalization, Feature Dropping, Sequence Reshape                                          | ✅ Rất tương đồng — “Window Aggregation” chính là phần tạo chuỗi đầu vào cho LSTM          |
| **Bộ phát hiện (Detection Engine)** | Rule-based detection (ngưỡng), và hướng mở rộng sang AI/ML                   | LSTM-AE gồm Encoder–Decoder học pattern bình thường, sau đó phân loại qua Softmax                                  | ✅ LSTM-AE có thể được tích hợp trực tiếp làm nhánh “Deep Learning Detection” song song Rule-based |
| **Kiến trúc tổng thể**              | Multi-stage pipeline: Sensor → Aggregation → Detection → Correlation → Alert | Two-stage deep learning: Stage 1 Preprocessing → Stage 2 Detection                                                 | ✅ Có thể **chèn trực tiếp Stage 2 của LSTM-AE vào khối Detection Engine**                         |
| **Đầu ra (Output)**                 | Alert message (attack type, severity, timestamp) → gửi Alert Manager         | Attack Label hoặc Anomaly Score (xác suất tấn công)                                                                | ✅ Có thể ánh xạ trực tiếp anomaly score → severity level                                          |
| **Cơ chế đánh giá (Evaluation)**    | Có thể dùng Accuracy, Precision, Recall, F1, False Alarm Rate                | Sử dụng chính xác các chỉ số đó                                                                                    | ✅ Giống hệ thống nhóm đang dùng                                                           |
| **Mục tiêu phát hiện**              | DDoS, Port Scan, Brute-force, …                                              | DDoS, PortScan, Infiltration, WebAttack, Botnet (phủ rộng hơn)                                                     | ✅ Có thể mở rộng để bao trùm các loại nhóm bạn yêu cầu                                            |
| **Tính thời gian thực (Real-time)** | Hướng đến cảnh báo sớm (real-time detection)                                 | Mô hình gốc offline, nhưng có thể chuyển sang streaming inference                                                  | ⚠️ **Cần tinh chỉnh** để dùng online (VD: sliding window LSTM, Kafka streaming input)             |
| **Độ phức tạp triển khai**          | Trung bình (đa module nhưng chủ yếu rule-based)                              | Cao (deep learning, cần GPU và nhiều dữ liệu)                                                                      | ⚠️ Cần training trước, nhưng inference nhanh nên phù hợp nếu kết hợp với hệ thống sẵn có          |
| **Khả năng mở rộng & tích hợp**     | Có thể tích hợp thêm mô hình AI trong Detection Engine                       | Dễ đóng gói mô hình LSTM-AE thành module (Python Flask API hoặc TensorFlow Serving)                                | ✅ Dễ tích hợp dưới dạng microservice gọi REST API                                                 |

Dưới đây là phần mã giả về các step mà chúng tôi sẽ hiện thực:

Stage 1 – Data Preprocessing:


FUNCTION preprocess_data(input_file):

    # 1. Load Dataset
    data ← read_csv(input_file)

    # 2. Handle Missing Values
    FOR each column in data:
        IF column has missing values:
            replace missing with mean(column) OR drop row

    # 3. Encode Categorical Features
    FOR each categorical column in data:
        data[column] ← one_hot_encode(data[column])

    # 4. Normalize Numeric Values
    scaler ← MinMaxScaler(0, 1)
    normalized_data ← scaler.fit_transform(data[numeric_features])

    # 5. Drop Irrelevant Features
    data ← remove_columns(data, [‘Timestamp’, ‘IP’, ‘Port’])

    # 6. Split Features and Labels
    X ← data[all_features_except_label]
    y ← data[label_column]

    # 7. Train/Test Split
    X_train, X_test, y_train, y_test ← train_test_split(X, y, test_size=0.2, shuffle=True)

    RETURN X_train, X_test, y_train, y_test

-----------------------------------------------------------------------------------------

   Mục đích của stage này là dùng để xử lý dữ liệu ( làm sạch và chuẩn hóa)
<img width="1260" height="752" alt="image" src="https://github.com/user-attachments/assets/45359a4a-b3ec-4e9d-b457-af320ff9ae5c" />


   
