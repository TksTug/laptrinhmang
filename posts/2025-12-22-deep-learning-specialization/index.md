# Deep Learning Specialization — Những điểm chính và lộ trình học


---
title: "Deep Learning Specialization — Những điểm chính và lộ trình học"
date: 2025-12-22T09:10:00+07:00
tags: ["deep-learning", "khóa-học", "Coursera"]
draft: false
summary: "Tổng quan về Deep Learning Specialization (Andrew Ng): mạng nơ-ron, CNN, RNN, và ứng dụng." 
---

## Giới thiệu

Deep Learning Specialization là một chuỗi 5 khóa học cung cấp lộ trình từ khái niệm cơ bản về mạng nơ-ron đến xây dựng những mô hình phức tạp như Convolutional Neural Networks (CNN) và Recurrent Neural Networks (RNN).

## Nội dung chính

### Khóa 1: Neural Networks and Deep Learning

**Cấu trúc mạng nơ-ron cơ bản:**

- **Neurons (Nơ-ron)**: Các đơn vị tính toán nhỏ nhất.
- **Layers (Lớp)**: Các nơ-ron được tổ chức thành các lớp.
  - Input layer: Nhận đầu vào
  - Hidden layers: Xử lý thông tin
  - Output layer: Tạo ra dự đoán

**Forward Propagation (Lan truyền xuôi):**

```
z = w·x + b  (tính tổng có trọng số)
a = activation(z)  (áp dụng hàm kích hoạt)
```

**Hàm kích hoạt (Activation Functions):**

- **ReLU (Rectified Linear Unit)**: a = max(0, z) - phổ biến nhất
- **Sigmoid**: a = 1/(1+e^-z) - thường dùng cho output nhị phân
- **Tanh**: a = (e^z - e^-z)/(e^z + e^-z) - phạm vi [-1, 1]
- **Softmax**: Dùng cho multi-class classification

**Backpropagation (Lan truyền ngược):**

Tính gradient để cập nhật trọng số:

```
∂L/∂w = (∂L/∂a) × (∂a/∂z) × (∂z/∂w)
w = w - α × (∂L/∂w)  (cập nhật trọng số)
```

### Khóa 2: Improving Deep Neural Networks

**Optimization techniques:**

- **Momentum**: Giữ lại "động lực" từ các bước trước để tối ưu hóa nhanh hơn.
- **RMSprop**: Điều chỉnh tốc độ học cho mỗi parameter riêng.
- **Adam**: Kết hợp Momentum và RMSprop - công cụ chuẩn hiện nay.

**Batch Normalization:**

Chuẩn hóa các lớp ẩn để huấn luyện ổn định hơn.

**Regularization để tránh overfitting:**

- L1/L2 Regularization: Thêm penalty vào loss function
- Dropout: Tắt ngẫu nhiên một số neuron

### Khóa 3: Structuring Machine Learning Projects

**Cách tổ chức một dự án ML hiệu quả:**

- Chia dữ liệu: Training set (60%), Dev set (20%), Test set (20%)
- Phân tích lỗi (error analysis): Xác định lỗi từ đâu
- Bias-Variance tradeoff: Cân bằng underfitting và overfitting

### Khóa 4: Convolutional Neural Networks (CNN)

**Cấu trúc CNN:**

```
Input → Conv → ReLU → Pool → ... → Flatten → FC → Output
```

**Convolution Layer:**

Áp dụng kernel (bộ lọc) nhỏ trên toàn bộ ảnh để phát hiện các đặc trưng (edges, textures, shapes).

```
Output = sum(Input × Kernel) + bias
```

**Pooling Layer:**

Giảm kích thước spatial, giữ lại thông tin quan trọng.

- **Max Pooling**: Lấy giá trị lớn nhất
- **Average Pooling**: Lấy trung bình

**Ví dụ kiến trúc:** LeNet, AlexNet, VGG, ResNet

**Transfer Learning:**

Sử dụng mô hình đã được huấn luyện trên ImageNet, chỉ thay đổi lớp cuối cùng:

```python
import tensorflow as tf

base_model = tf.keras.applications.VGG16(weights='imagenet')
base_model.trainable = False  # Đóng băng các lớp cũ

model = tf.keras.Sequential([
    base_model,
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(256, activation='relu'),
    tf.keras.layers.Dense(num_classes, activation='softmax')
])
```

### Khóa 5: Sequence Models (RNN, LSTM)

**Recurrent Neural Networks (RNN):**

Xử lý dữ liệu chuỗi (time series, text):

```
h_t = activation(W_h × h_{t-1} + W_x × x_t + b)
y_t = W_y × h_t + b
```

**Vấn đề Vanishing Gradient:**

Trong RNN dài, gradient biến mất trong quá trình backpropagation.

**Long Short-Term Memory (LSTM):**

Giải quyết vanishing gradient bằng cấu trúc cell đặc biệt:

```
Forget Gate: f_t = σ(W_f × [h_{t-1}, x_t] + b_f)
Input Gate: i_t = σ(W_i × [h_{t-1}, x_t] + b_i)
Candidate: C̃_t = tanh(W_c × [h_{t-1}, x_t] + b_c)
Cell State: C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t
Output Gate: o_t = σ(W_o × [h_{t-1}, x_t] + b_o)
```

**Ứng dụng:**

- Machine Translation (dịch máy): seq2seq models
- Speech Recognition (nhận diện giọng nói)
- Sentiment Analysis (phân tích cảm xúc)
- Time Series Forecasting (dự báo chuỗi thời gian)

## Thực hành đề xuất

### 1. Mạng nơ-ron cơ bản

```python
import tensorflow as tf

# Xây dựng mạng 2-3 lớp
model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=10, batch_size=32, validation_data=(X_val, y_val))
```

### 2. CNN trên MNIST/CIFAR-10

```python
model = tf.keras.Sequential([
    tf.keras.layers.Conv2D(32, (3,3), activation='relu', input_shape=(28, 28, 1)),
    tf.keras.layers.MaxPooling2D((2,2)),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D((2,2)),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dense(10, activation='softmax')
])
```

### 3. LSTM cho Time Series

```python
model = tf.keras.Sequential([
    tf.keras.layers.LSTM(50, activation='relu', input_shape=(lookback, 1)),
    tf.keras.layers.Dense(1)
])

model.compile(optimizer='adam', loss='mse')
model.fit(X_train, y_train, epochs=50, batch_size=16)
```

## Công cụ và Framework

- **TensorFlow/Keras**: High-level, dễ sử dụng
- **PyTorch**: Linh hoạt, yêu thích của researchers
- **OpenCV**: Xử lý ảnh
- **Librosa**: Xử lý audio

## Kết luận

Deep Learning Specialization cung cấp nền tảng toàn diện về deep learning. Sau khóa này, bạn sẽ có thể:

- Hiểu cơ chế hoạt động của mạng nơ-ron sâu
- Xây dựng các mô hình CNN cho computer vision
- Xây dựng các mô hình RNN/LSTM cho NLP và time series
- Áp dụng transfer learning để làm việc với dữ liệu nhỏ
- Tổ chức các dự án ML một cách hiệu quả

Đây là bước ngoặt để từ machine learning cơ bản sang deep learning chuyên sâu!
- Sequence models: xây RNN/LSTM để dự đoán chuỗi số hoặc phân loại văn bản ngắn.

Bài tập dự án:

- Dự án 1: Nhận diện chữ số (MNIST) — thử các kiến trúc và so sánh accuracy.
- Dự án 2: Transfer learning cho phân loại ảnh (tận dụng ResNet hoặc MobileNet).

Tài nguyên thêm:

- Thư viện: TensorFlow (Keras) hoặc PyTorch.
- Các khóa thực hành trên Coursera / Udacity; các tutorial trên TensorFlow.org / PyTorch.org.

