# Machine Learning Engineering for Production (MLOps) — Khóa học và thực hành


---
title: "Machine Learning Engineering for Production (MLOps) — Khóa học và thực hành"
date: 2025-12-22T09:20:00+07:00
tags: ["mlops", "khóa-học", "production"]
draft: false
summary: "Ghi chú và hướng dẫn thực hành từ khóa ML Engineering for Production (MLOps)."
---

## Giới thiệu về MLOps

Khóa học về ML Engineering for Production tập trung vào việc đưa mô hình machine learning vào môi trường thực tế: giám sát, CI/CD cho mô hình, quản lý dữ liệu và vòng đời của mô hình.

MLOps (Machine Learning Operations) là sự kết hợp giữa Machine Learning, Data Engineering và DevOps. Nó giải quyết vấn đề: "Làm thế nào để đặt một mô hình ML vào sản xuất và duy trì nó một cách hiệu quả?"

## Các thách thức chính trong Production ML

### 1. **Data Quality (Chất lượng dữ liệu)**

Trong thế giới thực, dữ liệu thường bị nhiễu, không đầy đủ, và không phân bổ đều:

- **Data Drift**: Phân phối dữ liệu trong sản xuất khác với dữ liệu huấn luyện.
- **Concept Drift**: Mối quan hệ giữa đầu vào và đầu ra thay đổi theo thời gian.
- **Missing Values**: Xử lý dữ liệu thiếu một cách thông minh.

### 2. **Model Performance Monitoring**

- Theo dõi độ chính xác, precision, recall liên tục.
- Phát hiện khi mô hình bắt đầu suy giảm hiệu suất.
- Kích hoạt retraining tự động nếu cần.

### 3. **Data Pipeline và Automation**

- **ETL Pipeline**: Extract, Transform, Load dữ liệu tự động.
- **Feature Store**: Kho trữ tập trung cho các tính năng.
- **Automated Training**: Huấn luyện lại mô hình khi cần.

## Các thành phần chính của MLOps

### 1. **Data Collection và Validation**

```python
import tensorflow_data_validation as tfdv

# Tạo thống kê về dữ liệu
stats = tfdv.generate_statistics_from_csv('data.csv')

# Kiểm tra schema
schema = tfdv.infer_schema(stats)
anomalies = tfdv.validate_statistics(stats, schema)
```

### 2. **Model Training và Versioning**

Sử dụng MLflow để theo dõi:

```bash
mlflow run . -P alpha=0.5 -P l1_ratio=0.5
mlflow ui  # Xem dashboard
```

### 3. **Model Deployment**

Đóng gói mô hình với Docker:

```dockerfile
FROM python:3.9
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY model.pkl .
COPY app.py .
CMD ["python", "app.py"]
```

Phục vụ mô hình qua Flask API:

```python
from flask import Flask, request, jsonify
import pickle

app = Flask(__name__)
model = pickle.load(open('model.pkl', 'rb'))

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    prediction = model.predict([data['features']])
    return jsonify({'prediction': prediction[0]})
```

### 4. **Monitoring và Alerting**

- Theo dõi latency, throughput, error rate.
- Cảnh báo nếu hiệu suất mô hình giảm.
- Log toàn bộ predictions.

### 5. **Retraining Pipeline**

Quy trình tự động:

```
Data Update → Validation → Training → Evaluation → Deploy (nếu tốt)
```

## Công cụ phổ biến

| Công cụ | Mục đích |
|---------|---------|
| **MLflow** | Tracking, packaging models |
| **Kubeflow** | ML workflows trên Kubernetes |
| **Apache Airflow** | Orchestration pipelines |
| **Docker** | Containerization |
| **Kubernetes** | Container orchestration |
| **TensorFlow Extended** | End-to-end ML platform |

## Kết luận

MLOps không chỉ là xây dựng mô hình tốt, mà còn về duy trì, giám sát, và liên tục cải thiện mô hình trong sản xuất. Đây là yếu tố quan trọng để trở thành một professional ML engineer.

---

Các chủ đề kỹ thuật quan trọng:

- CI/CD cho mô hình: tự động hóa huấn luyện, kiểm thử và deploy khi có dữ liệu mới.
- Giám sát mô hình: thiết lập metrics (latency, accuracy, data drift) và alerting.
- Quản lý phiên bản mô hình và dữ liệu: lưu metadata, checkpoints và schema cho feature store.

Bài tập thực tế:

- Xây pipeline đơn giản: từ dữ liệu thô → preprocessing → huấn luyện → đóng gói Docker + deploy trên một endpoint.
- Thử nghiệm rollback khi model mới có hiệu năng kém.

Tài liệu tham khảo:

- Tài liệu MLOps trên Google Cloud / AWS SageMaker cho các pattern triển khai.
- Công cụ: MLflow, Kubeflow, TFX.

