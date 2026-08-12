Từ [[Tóm tắt Sentinel-1 Level 0 Data Decoding Example]], tác giả có đề cập rằng đoạn demo đang bỏ qua một số bước so với thuật toán được sử lý bởi IPF ESA, có thể kể đến như: 
- Tác giả đang mặc định Doppler Centroid = 0 Hz. 
- Tác giả chưa áp dụng bước Secondary Range Compression (SRC)
Bản demo này sẽ thực hiện bổ sung 2 bước thành phần vào luồng xử lý của thuật toán, bao gồm: 
1. Doppler Centroid Estimation 
2. Secondary Range Compression
từ đó so sánh độ cải thiện của chất lượng ảnh.

# 1. Cơ sở lý thuyết
## 1.1 Doppler Centroid Estimation
- Nhìn chung, quá trình xử lý DCE  có thể biểu diễn như ở [Hình 5-1]
![[Pasted image 20260812152509.png]]
- Các bước thực hiện thuật toán DCE được thực hiện bởi ESA nhìn chung có thể được chia làm 6 bước chính như sau: 
	1. Tính giá trị Doppler Centroid tuyệt đối (từ thông tin quỹ đạo và tư thế của vệ tinh) [Section 5.1]
	2. Tính ước lượng Fine Doppler Centroid từ dữ liệu thô [Section 5.2]. 
	3. Đo lường chất lượng ước lượng DC [Section 5.5.1]
	4. Giải quấn ước lượng Fine DC [Section 5.3]
	5. Polynomial Fitting [Section 5.5] 
	6. Absolute DC estimation [Section 5.4]
 ![[Pasted image 20260812151042.png|528]]
 
==Câu hỏi==: Có được DC polynomial từ bước này rồi, vậy sử dụng polynomial này ở đâu?
==Trả lời:== Trong bước tạo azimuth filter:
$$
H_az(f_\eta) = W(f_\eta-f\eta_c)exp\left(j\frac{4\pi R_0D(f_\eta,V_r)f_0}c{} \right) = W(f_\eta-f\eta_c)exp\left(j\frac{4\pi RD^2(f_\eta,V_r)f_0}c{} \right) 
$$
==Câu hỏi==: Thử kiểm tra 8.2.1 Stripmap Case xem có liên quan gì đến cách chia range/azimuth block không?


### 1.1.1 Absolute DC Calculation (từ thông tin quỹ đạo và tư thế)
- Bước đầu là xác định absolute DC từ hình học của vệ tinh-trái đất sử dụng dữ liệu Orbit và Attitude. 
- Các thông tin về vị trí và vận tốc vệ tinh (satellite ephemeris), được biểu diễn bằng các vector trạng thái, thường được ước lượng bởi hệ thống GPS on-board.
>[!note]
>Đọc [R-9] Section 2 để biết về định nghĩa attitude angles
- Với một vị trí quỹ đạo xác định (fixed satellite ephemeris), tần số Doppler là một hàm của $r$ và $\mathbf{\varphi}$ 