# Triển khai Mô hình ML với Flask và Docker — Một hướng dẫn ngắn


---
title: "Triển khai Mô hình ML với Flask và Docker — Một hướng dẫn ngắn"
date: 2025-12-22T09:40:00+07:00
tags: ["machine-learning", "deployment", "docker"]
draft: false
summary: "Các bước nhanh để đóng gói và triển khai mô hình học máy đơn giản bằng Flask và Docker."
---

## Giới thiệu

Xây dựng một mô hình ML tốt chỉ là nửa công việc. Nửa còn lại là triển khai (deployment) nó sao cho các ứng dụng khác có thể sử dụng nó. Bài viết này hướng dẫn cách triển khai mô hình bằng **Flask** (web framework) và **Docker** (containerization).

## Tại sao cần Flask + Docker?

### Flask
- Lightweight web framework dễ sử dụng
- Có thể expose model thành REST API
- Hỗ trợ request/response trong JSON format

### Docker
- Packaging ứng dụng với tất cả dependencies
- Chạy được trên bất kỳ máy nào (Windows, Linux, Mac)
- Dễ scale với Kubernetes hoặc Docker Compose

## Quy trình từng bước

### Bước 1: Chuẩn bị mô hình

Đầu tiên, huấn luyện và lưu mô hình:

```python
# train.py
import pickle
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier

# Load data
iris = load_iris()
X, y = iris.data, iris.target

# Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X, y)

# Save model
with open('iris_model.pkl', 'wb') as f:
    pickle.dump(model, f)

print("Model saved as iris_model.pkl")
```

Hoặc với TensorFlow:

```python
import tensorflow as tf

model = tf.keras.Sequential([...])
model.compile(...)
model.fit(...)

# Save model
model.save('iris_model.h5')
```

### Bước 2: Tạo Flask API

Viết API để load model và trả kết quả dự đoán:

```python
# app.py
from flask import Flask, request, jsonify
import pickle
import numpy as np
from sklearn.datasets import load_iris

app = Flask(__name__)

# Load model
with open('iris_model.pkl', 'rb') as f:
    model = pickle.load(f)

# Load class names
iris = load_iris()
class_names = iris.target_names

@app.route('/', methods=['GET'])
def home():
    """Health check endpoint"""
    return jsonify({
        'status': 'Model API is running',
        'model': 'Iris Classifier',
        'version': '1.0'
    })

@app.route('/predict', methods=['POST'])
def predict():
    """Endpoint để dự đoán"""
    try:
        # Lấy dữ liệu từ request
        data = request.get_json()
        
        # Validate
        if 'features' not in data:
            return jsonify({'error': 'Missing "features" field'}), 400
        
        features = np.array(data['features']).reshape(1, -1)
        
        # Dự đoán
        prediction = model.predict(features)[0]
        probability = model.predict_proba(features)[0].max()
        
        return jsonify({
            'prediction': class_names[prediction],
            'confidence': float(probability),
            'class_index': int(prediction)
        })
    
    except Exception as e:
        return jsonify({'error': str(e)}), 500

@app.route('/predict/batch', methods=['POST'])
def predict_batch():
    """Dự đoán nhiều samples một lúc"""
    try:
        data = request.get_json()
        features = np.array(data['features'])
        
        predictions = model.predict(features)
        probabilities = model.predict_proba(features).max(axis=1)
        
        results = []
        for pred, prob in zip(predictions, probabilities):
            results.append({
                'prediction': class_names[pred],
                'confidence': float(prob)
            })
        
        return jsonify({'predictions': results})
    
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### Bước 3: Test Flask API locally

```bash
# Install dependencies
pip install flask scikit-learn

# Run server
python app.py
# Output: Running on http://127.0.0.1:5000

# Test trong terminal khác
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [5.1, 3.5, 1.4, 0.2]}'

# Hoặc với Python
import requests

response = requests.post('http://localhost:5000/predict', json={
    'features': [5.1, 3.5, 1.4, 0.2]
})
print(response.json())
```

### Bước 4: Viết Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY app.py .
COPY iris_model.pkl .

# Install dependencies
RUN pip install --no-cache-dir flask scikit-learn numpy

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:5000/')"

# Run application
CMD ["python", "app.py"]
```

**Giải thích:**
- `FROM`: Image cơ sở (Python 3.11)
- `WORKDIR`: Thư mục làm việc trong container
- `COPY`: Copy file từ local vào container
- `RUN`: Chạy lệnh (install packages)
- `EXPOSE`: Khai báo port (documentation, không thực sự mở port)
- `HEALTHCHECK`: Kiểm tra sức khỏe container
- `CMD`: Lệnh chạy khi container start

### Bước 5: Build Docker Image

```bash
# Build image
docker build -t iris-api:1.0 .

# Verify image
docker images | grep iris-api
```

### Bước 6: Run Container

```bash
# Run container
docker run -p 5000:5000 iris-api:1.0

# Run in background
docker run -d -p 5000:5000 --name iris-api iris-api:1.0

# Check logs
docker logs iris-api

# Stop container
docker stop iris-api
```

### Bước 7: Test API từ container

```bash
# Từ local machine
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [5.1, 3.5, 1.4, 0.2]}'
```

## Cấu trúc Project

```
iris-ml-api/
├── app.py                  # Flask application
├── train.py               # Script huấn luyện model
├── iris_model.pkl         # Model đã lưu
├── Dockerfile             # Docker configuration
├── requirements.txt       # Dependencies
├── .dockerignore          # Files to ignore in Docker
└── README.md
```

### requirements.txt

```
Flask==2.3.0
scikit-learn==1.2.0
numpy==1.24.0
requests==2.31.0
```

```bash
# Build lại dựa trên requirements.txt
docker build -t iris-api:1.0 -f Dockerfile .
```

## Advanced: Docker Compose

Nếu cần nhiều services (API + Database), dùng Docker Compose:

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=production
    depends_on:
      - database
    restart: always

  database:
    image: postgres:13
    environment:
      - POSTGRES_DB=ml_db
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

```bash
# Run với docker-compose
docker-compose up -d
docker-compose logs -f
docker-compose down
```

## Deployment options

### 1. **Local Development**
```bash
python app.py
```

### 2. **Docker locally**
```bash
docker run -p 5000:5000 iris-api:1.0
```

### 3. **Docker Hub**
```bash
docker login
docker tag iris-api:1.0 username/iris-api:1.0
docker push username/iris-api:1.0
```

### 4. **Cloud Services**
- **AWS**: ECS, AppRunner
- **Google Cloud**: Cloud Run, GKE
- **Azure**: Container Instances, AKS
- **Heroku**: Hỗ trợ Docker

## Best Practices

1. **Use specific Python version** (không dùng `latest`)
2. **Multi-stage builds** để giảm image size:
   ```dockerfile
   FROM python:3.11 AS builder
   # ... build stages ...
   
   FROM python:3.11-slim
   # ... copy từ builder ...
   ```

3. **Environment variables** cho config:
   ```python
   import os
   MODEL_PATH = os.getenv('MODEL_PATH', 'iris_model.pkl')
   ```

4. **Logging** tốt để debug:
   ```python
   import logging
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   ```

5. **API versioning**:
   ```python
   @app.route('/v1/predict', methods=['POST'])
   ```

## Troubleshooting

| Vấn đề | Giải pháp |
|--------|----------|
| Container not starting | `docker logs container_id` |
| Port already in use | `docker run -p 5001:5000` |
| Model not found | Kiểm tra COPY trong Dockerfile |
| Out of memory | `docker run -m 512m` |

## Kết luận

Triển khai ML model bằng Flask + Docker là cách nhanh và hiệu quả để:

✅ Tạo API cho model  
✅ Packaging ứng dụng  
✅ Deploy trên bất kỳ máy nào  
✅ Scale với container orchestration  

Sau khi nắm vững cơ bản này, bạn có thể tiến tới các công cụ nâng cao như Kubernetes, Seldon Core, hoặc KServe!
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:8080"]
```

Lưu ý: đảm bảo `requirements.txt` chỉ chứa phiên bản cần thiết (Flask, scikit-learn, gunicorn...).

