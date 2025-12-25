# Machine Learning (Andrew Ng) — Tóm tắt và ứng dụng


---
title: "Machine Learning (Andrew Ng) — Tóm tắt và ứng dụng"
date: 2025-12-22T09:00:00+07:00
tags: ["machine-learning", "khóa-học", "Coursera"]
draft: false
summary: "Tóm tắt nội dung chính và ứng dụng thực tiễn từ khóa Machine Learning của Andrew Ng (Coursera)."
---

## Giới thiệu

Khóa học "Machine Learning" của Andrew Ng trên Coursera là một trong những khóa nền tảng quan trọng nhất cho người mới bắt đầu tiếp cận lĩnh vực học máy. Với phong cách giảng dạy rõ ràng và dễ hiểu, Andrew Ng đã giúp hàng triệu người thế giới hiểu được các nguyên lý cơ bản của machine learning.

## Nội dung chính của khóa học

### 1. **Các thuật toán cơ bản**

Khóa học bắt đầu với những thuật toán nền tảng:

- **Linear Regression (Hồi quy tuyến tính)**: Dùng để dự đoán các giá trị liên tục. Ví dụ: dự đoán giá nhà dựa trên diện tích, vị trí.
- **Logistic Regression (Hồi quy logistic)**: Dù có tên là "regression", nhưng được sử dụng cho bài toán phân loại nhị phân. Ứng dụng: phân loại email spam/không spam.
- **Support Vector Machine (SVM)**: Một thuật toán mạnh mẽ cho bài toán phân loại với ranh giới quyết định phi tuyến.
- **K-means Clustering**: Thuật toán học không giám sát để phân nhóm dữ liệu thành K cụm dữ liệu.
- **Neural Networks (Mạng nơ-ron)**: Giới thiệu khái niệm về các lớp ẩn (hidden layers) và forward/backward propagation.

### 2. **Tiền xử lý dữ liệu và đặc trưng**

Một phần quan trọng của machine learning là làm sạch và chuẩn bị dữ liệu:

- Xử lý dữ liệu bị thiếu (missing values).
- Loại bỏ những dữ liệu ngoài ý muốn (outliers).
- **Feature Scaling (Chuẩn hóa tính năng)**: Đưa các tính năng về cùng một khoảng giá trị (ví dụ 0-1) để thuật toán học tập hiệu quả hơn.
- **Feature Engineering**: Tạo ra những tính năng mới từ các tính năng có sẵn để cải thiện hiệu suất mô hình.

### 3. **Regularization (Chính quy hóa)**

Để tránh overfitting (mô hình học quá tốt trên dữ liệu huấn luyện nhưng không tổng quát hóa tốt):

- **L1 Regularization (Lasso)**: Khuyến khích mô hình sử dụng ít tính năng hơn.
- **L2 Regularization (Ridge)**: Giữ cho các hệ số của mô hình nhỏ hơn.
- **Dropout**: Trong mạng nơ-ron, tắt ngẫu nhiên một số neuron trong quá trình huấn luyện.

### 4. **Các kỹ thuật tối ưu hóa**

- **Gradient Descent**: Thuật toán cơ bản để tối ưu hóa hàm mất mát (loss function).
- **Batch Gradient Descent**: Cập nhật trọng số sau khi xử lý toàn bộ dữ liệu.
- **Mini-batch Gradient Descent**: Cập nhật sau khi xử lý một phần nhỏ dữ liệu (nhanh hơn và ít yêu cầu bộ nhớ).
- **Stochastic Gradient Descent (SGD)**: Cập nhật sau mỗi mẫu dữ liệu.

## Ứng dụng thực tế

### Dự đoán (Regression)

- **Dự báo kinh doanh**: Dự đoán doanh thu, lượng tiêu thụ, giá sản phẩm.
- **Y tế**: Dự đoán chiều cao, cân nặng, hoặc các chỉ số sức khỏe.
- **Giao thông**: Dự đoán thời gian tới nơi, mức tiêu thụ năng lượng.

### Phân loại (Classification)

- **Email Filtering**: Phân loại email là spam hay không spam.
- **Phân tích cảm xúc (Sentiment Analysis)**: Xác định liệu một bình luận/đánh giá là tích cực hay tiêu cực.
- **Phát hiện gian lận (Fraud Detection)**: Xác định các giao dịch bất thường trong ngành ngân hàng.
- **Chẩn đoán bệnh**: Dự đoán liệu một bệnh nhân có bị mắc bệnh hay không dựa trên các dấu hiệu y tế.

## Gợi ý học tập hiệu quả

1. **Thực hành từng bài tập**: Khóa học cung cấp các bài tập lập trình. Ban đầu được yêu cầu sử dụng Octave/MATLAB, nhưng bạn cũng có thể sử dụng Python.

2. **Chuyển từ Octave sang Python**: Python với các thư viện như NumPy, SciPy, Scikit-learn sẽ là công cụ chính sau khóa học.

3. **Áp dụng trên dữ liệu nhỏ trước**: 
   - Bắt đầu với các bộ dữ liệu đơn giản (ví dụ: Iris, Boston Housing).
   - Sau đó nâng cao độ phức tạp.

4. **Ghi chép và ôn tập**: Ghi lại những khái niệm quan trọng, công thức toán học để dễ ôn lại.

5. **Tham gia cộng đồng**: Tham gia diễn đàn Coursera, Stack Overflow, hoặc các cộng đồng machine learning để hỏi đáp.

## Kết luận

Khóa học Machine Learning của Andrew Ng là bước đệm hoàn hảo cho bất kỳ ai muốn bước vào lĩnh vực machine learning. Nó không chỉ dạy các thuật toán mà còn dạy cách tư duy về các vấn đề dưới góc độ machine learning.

Sau khi hoàn thành khóa này, bạn sẽ sẵn sàng cho các khóa chuyên sâu hơn như Deep Learning Specialization, Applied Machine Learning, hoặc MLOps Engineering.

**Tài liệu tham khảo**: https://www.coursera.org/learn/machine-learning

---

Chi tiết khoá học (gợi ý để ôn tập):

- Tuần 1-2: Hồi quy tuyến tính, Mean Squared Error, gradient descent.
- Tuần 3: Hồi quy logistic, phân loại nhị phân, hàm mất mát log-loss.
- Tuần 4: Regularization (L1/L2), bias-variance tradeoff.
- Tuần 5: Support Vector Machines cơ bản.
- Tuần 6: K-means & học không giám sát, PCA khái quát.

Bài tập & dự án nhỏ:

- Triển khai hồi quy tuyến tính để dự đoán giá nhà (bộ dữ liệu giả định).
- Làm một bài phân loại email spam với logistic regression.

Tài nguyên thêm:

- Lecture notes và bài tập trên Coursera.
- Sách tham khảo: "Pattern Recognition and Machine Learning" (Christopher Bishop) để đọc sâu hơn.

Lời khuyên:

- Không chỉ đọc lý thuyết: triển khai từng thuật toán bằng code để hiểu sâu.
- Khi chuyển sang sản phẩm thực tế, học kỹ phần tiền xử lý dữ liệu và đánh giá mô hình (precision/recall/F1).

