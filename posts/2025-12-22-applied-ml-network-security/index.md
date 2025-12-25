# Ứng dụng Học Máy trong Phát hiện Xâm nhập Mạng (IDS)


---
title: "Ứng dụng Học Máy trong Phát hiện Xâm nhập Mạng (IDS)"
date: 2025-12-22T09:30:00+07:00
tags: ["machine-learning", "network-security", "application"]
draft: false
summary: "Cách sử dụng học máy để phát hiện xâm nhập và bất thường trong lưu lượng mạng." 
---

## Giới thiệu

Phát hiện xâm nhập (Intrusion Detection Systems - IDS) là một trong những ứng dụng quan trọng nhất của machine learning trong an ninh mạng. Thay vì chỉ dựa vào các quy tắc cứng định sẵn, IDS dựa trên ML có thể tự học các mẫu của các cuộc tấn công mới.

## Các loại tấn công mạng

### 1. **Denial of Service (DoS)**
Tấn công làm quá tải máy chủ, làm cho nó không thể phục vụ người dùng hợp pháp.

### 2. **Port Scanning**
Quét các cổng mở để tìm các dịch vụ dễ bị tấn công.

### 3. **Brute Force Attack**
Thử nhiều lần để đoán mật khẩu.

### 4. **Data Exfiltration**
Lấy cắp dữ liệu nhạy cảm khỏi mạng.

### 5. **Man-in-the-Middle (MITM)**
Chèn mình vào giữa hai bên để nghe lén hoặc thay đổi thông điệp.

## Quy trình phát hiện xâm nhập với ML

### Bước 1: Thu thập dữ liệu mạng

Có nhiều công cụ để thu thập traffic mạng:

- **tcpdump**: Capture raw packets
- **Wireshark**: GUI tool để phân tích packets
- **NetFlow**: Flow-based data collection (nhanh hơn, ít chi phí)
- **zeek**: Network IDS với scripting language riêng

```bash
# Capture traffic với tcpdump
tcpdump -i eth0 -w capture.pcap

# Đọc file PCAP
from scapy.all import rdpcap
packets = rdpcap('capture.pcap')
```

### Bước 2: Trích xuất đặc trưng (Feature Extraction)

Từ raw packets, ta trích xuất các tính năng quan trọng:

**Tính năng cấp packet:**
- Protocol (TCP, UDP, ICMP, ...)
- Packet size
- TTL (Time To Live)

**Tính năng cấp flow:**
- Source IP, Destination IP, Port
- Duration (thời lượng kết nối)
- Total bytes sent/received
- Packet count
- Packet rate (packets/second)
- Byte rate (bytes/second)

**Tính năng cấp connection pattern:**
- Number of flows per source IP
- Diversity of destination ports
- Frequency of protocol usage
- Ratio of inbound to outbound traffic

```python
from scapy.all import rdpcap, IP, TCP, UDP

def extract_flow_features(pcap_file):
    packets = rdpcap(pcap_file)
    flows = {}
    
    for pkt in packets:
        if IP in pkt:
            src_ip = pkt[IP].src
            dst_ip = pkt[IP].dst
            
            # Tạo flow key
            flow_key = f"{src_ip}-{dst_ip}"
            
            if flow_key not in flows:
                flows[flow_key] = {
                    'packet_count': 0,
                    'byte_count': 0,
                    'protocols': [],
                    'packet_sizes': []
                }
            
            flows[flow_key]['packet_count'] += 1
            flows[flow_key]['byte_count'] += len(pkt)
            flows[flow_key]['packet_sizes'].append(len(pkt))
            
            if TCP in pkt:
                flows[flow_key]['protocols'].append('TCP')
            elif UDP in pkt:
                flows[flow_key]['protocols'].append('UDP')
    
    return flows
```

### Bước 3: Chuẩn bị dữ liệu huấn luyện

Sử dụng các bộ dữ liệu công khai như:

- **KDD Cup 99**: Dataset kinh điển (nhưng lỗi thời)
- **NSL-KDD**: Cải tiến của KDD Cup 99
- **CICIDS2017**: Dataset hiện đại, có các cuộc tấn công mới
- **UNSW-NB15**: Bộ dữ liệu toàn diện từ đại học UNSW

### Bước 4: Chọn và huấn luyện mô hình

**Các thuật toán phổ biến:**

#### a) **Supervised Learning** (cần nhãn attack/normal)

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

# Chuẩn hóa dữ liệu
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Chia dữ liệu
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.3, random_state=42
)

# Huấn luyện Random Forest
model = RandomForestClassifier(n_estimators=100, max_depth=15)
model.fit(X_train, y_train)

# Đánh giá
accuracy = model.score(X_test, y_test)
print(f"Accuracy: {accuracy}")

# Dự đoán trên dữ liệu mới
predictions = model.predict(new_traffic)
```

#### b) **Unsupervised Learning** (không cần nhãn)

Phát hiện dị thường mà không cần biết loại tấn công:

```python
from sklearn.ensemble import IsolationForest

# Isolation Forest cho phát hiện dị thường
iso_forest = IsolationForest(contamination=0.1, random_state=42)
anomaly_predictions = iso_forest.fit_predict(X_train)

# -1 = anomaly, 1 = normal
print(f"Detected anomalies: {sum(anomaly_predictions == -1)}")
```

#### c) **Deep Learning** (Neural Networks)

```python
import tensorflow as tf

model = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu', input_shape=(num_features,)),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(32, activation='relu'),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(1, activation='sigmoid')
])

model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=20, batch_size=32, validation_data=(X_test, y_test))
```

### Bước 5: Đánh giá mô hình

Với unbalanced data (attacks hiếm hơn normal traffic), accuracy không phải metric tốt:

```python
from sklearn.metrics import precision_recall_fscore_support, confusion_matrix

precision, recall, f1, support = precision_recall_fscore_support(y_true, y_pred)

print(f"Precision: {precision}")
print(f"Recall: {recall}")
print(f"F1-score: {f1}")

# Confusion matrix
cm = confusion_matrix(y_true, y_pred)
print(cm)
```

**Giải thích các metric:**
- **Precision**: Trong những dự đoán "attack", bao nhiêu % đúng?
- **Recall**: Trong các attacks thực tế, phát hiện được bao nhiêu %?
- **F1-score**: Cân bằng giữa precision và recall

## Thách thức thực tế

### 1. **Imbalanced Data**
Attacks thường chiếm < 5% traffic, dẫn đến mô hình bias.

**Giải pháp:**
- SMOTE (Synthetic Minority Over-sampling)
- Adjusted class weights
- Threshold tuning

### 2. **False Positives**
Dự đoán normal traffic là attack gây quá tải SOC team.

**Giải pháp:**
- Tune threshold để giảm false positives
- Implement rule-based filtering đầu tiên
- Phân loại alerts theo mức độ nghiêm trọng

### 3. **Data Drift**
Kiểu tấn công thay đổi theo thời gian, mô hình cần cập nhật.

**Giải pháp:**
- Retrain mô hình hàng tuần/tháng
- Monitor model performance
- Implement online learning

### 4. **Privacy**
Không thể chia sẻ raw traffic data do chứa thông tin nhạy cảm.

**Giải pháp:**
- Chỉ sử dụng aggregated features
- Anonymize IP addresses
- Implement federated learning

## Ví dụ hoàn chỉnh

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, roc_auc_score

# 1. Load data
df = pd.read_csv('network_traffic.csv')

# 2. Feature engineering
X = df.drop('label', axis=1)
y = df['label']

# 3. Normalize
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 4. Train-test split
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2)

# 5. Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# 6. Evaluate
y_pred = model.predict(X_test)
print(classification_report(y_test, y_pred))
print(f"ROC-AUC: {roc_auc_score(y_test, model.predict_proba(X_test)[:, 1])}")
```

## Kết luận

Machine Learning cho IDS mang lại những lợi ích:

✅ Phát hiện attacks mới (không cần quy tắc cứng)  
✅ Giảm false positives so với rule-based systems  
✅ Tự động học từ dữ liệu mới  

Tuy nhiên, cũng cần:

⚠️ Dữ liệu huấn luyện chất lượng cao  
⚠️ Liên tục monitor và retrain  
⚠️ Kết hợp với các kỹ thuật security khác (firewalls, encryption, ...)  

Đây là lĩnh vực rất thú vị và cần thiết trong ngành an ninh mạng hiện nay!

