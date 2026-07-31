# 1. Introduction 
- Bayesian Theory là gì? [[Giải thích Bayesian Theory]]
- Mô hình thống kê giúp cho các sản phẩm SAR có nhiều ứng dụng:
	1. Giúp tạo sự hiểu biết sâu sắc về cơ chế phân tán địa hình.
	2. Giúp phục vụ nghiên cứu các phương pháp speckle suppression, edge detection, segmentation, classification, target detection, recognition cho ảnh SAR. 
	3. Kết hợp mô hình thống kê với ISAR target database có thể mô phỏng ảnh SAR với các tham số như góc nhìn, loại định hình, vị trí vùng, SCR (signal to clutter ratio).
# 2. Model Classification and Research Contents
- Mô hình thống kê ảnh SAR được chia làm 2 loại: 
	- Parametric models: Mô hình tham số 
	- Nonparametric models: Mô hình phi tham số
>[!note]
>Nonparametric models làm cho quá trình mô hình thống kê linh hoạt hơn và làm cho mô hình khớp với dữ liệu thực tế hơn, nhưng đổi lại yêu cầu tính toán phức tạp, cũng như nhiều dữ liệu. 
- Quá trình mô hình tham số bao gồm các bước sau: 
	1. Phân tích các mô hình phân bố thống kê đã biết
	2. Ước lượng tham số: ước lượng các bộ tham số của các distribution 
	3. Goodness-of-fit tests: đánh giá độ chính xác của các mô hình đưa ra với dữ liệu thực tế.
![[Pasted image 20260731112144.png|637]]
## 2.1 Parameter estimation
- Hai phương pháp ước lượng tham số được sử dụng nhiều nhất là: 
	- Method of Moments (MoM)
	- Maximum-likelihood (ML) 
- Ở thời gian gần đây, phương pháp ==method of log-cummulants  (MoLC== cũng là một phương pháp được sử dụng phổ biến. 
## 2.2 Goodneses-of-Fit Tests
- [[Phương pháp test mô hình fit chung]]
- Nhìn chung, các rules chính để kiểm tra độ chính xác của mô hình bao gồm: 
	- $\chi^2$ matching test
	- AIC (Akaike information criteria) rule
	- K-S (Kolmogorov-Smirnov) test
	- K-L distance measurement
	- D' Agostino-Pearson test
	- Kuiper tests
# 3. Statistical Models
- Mô hình thống kê của ảnh SAR đơn cực có thể mở rộng để mô tả ảnh SAR đa phân cực (==các tính chất thống kê của ảnh phân cực không khác biệt so với ảnh đơn cực==).
## 3.1 Nonparametric Models
- Nonparametric models là những loại mô hình có thể ước lượng một cách hiệu quả hàm mật độ xác xuất (PDF) của ảnh SAR dựa trên các phương pháp không dùng tham số. 
- Ý tưởng cơ bản là sử dụng tổng có trọng số ($\sum_{i=1}^n w_i x_i$) của các hàm kernel khác nhau để tính ước lượng của mô hình thống kê. 
- Các phương pháp thông dụng bao gồm: 
	- Parzen window technique
	- Artificial neural network (ANN)
	- Support vector machine (SVM) method, etc.
- Các mô hình phi tham số có tính chất là mô