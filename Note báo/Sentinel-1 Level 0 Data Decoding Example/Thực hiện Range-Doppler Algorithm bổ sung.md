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
- Với một vị trí quỹ đạo xác định (fixed satellite ephemeris), tần số Doppler là một hàm của $r$ và $\vec{\varphi}$, được gọi là Doppler Frequency Function (DFF):
![[Pasted image 20260812162349.png|415]]
với $\vec{r_0}(r,\vec{\varphi})$ là unit view vector
![[Pasted image 20260812162448.png|391]]
- Vector đơn vị $\vec{r_0}(r,\vec{\varphi})$ là nghiệm được tính từ hệ phi tuyến sau:
![[Pasted image 20260812162647.png|455]]
![[Pasted image 20260812162730.png|485]]
- $\vec{u_0}(\vec{\varphi})$  là vector đơn vị, vuông góc với $\vec{r_0}(r,\vec{\varphi})$, mô tả hướng di chuyển của vệ tinh: 
![[Pasted image 20260812162935.png|444]]
1. Ma trận $\vec{L_0}(\vec{p},\vec{v_a})$ là ma trận quỹ đạo và có thể được tính từ vector vị trí $\vec{p}$  và vận tốc tuyệt đối của vệ tinh $\vec{v_a}$:
![[Pasted image 20260812163600.png|379]]
2. Ma trận $\vec{L}(\vec{\varphi})$ trong phương trình (5-9) là ma trận tư thế và có thể được tính từ góc tư thế roll, pitch, yaw ($\varphi, \theta, \psi$):
![[Pasted image 20260812163853.png|449]]
- ==Các phép tính trên sẽ được thực hiện trên slant range tương ứng với tâm của mỗi n range blocks.== 
- Kết quả là một bộ các cặp giá trị slant range và tần số Doppler Centroid tuyệt đối:
![[Pasted image 20260812164138.png|527]]

### 1.1.2 Fine DC Estimation 
- Doppler sampling rate = PRF, nên sẽ giới hạn tần số Doppler đo được. Vì vậy tín hiệu nhận được chỉ quan sát được trong dải -PRF/2 tới +PRF/2. 
- Để giải điều chế, dữ liệu phải được thực hiện các thao tác sau, các thao tác được thực hiện trên từng range line một trong azimuth block: 
![[Pasted image 20260812164853.png|420]]
- Với Mode Stripmap, tính FDC cần thực hiện Correlation DC Estimator (CDCE) [Section 5.2.2]
Lưu ý rằng FDC estimation được ước lượng trên dữ liệu range compressed, vì thế được tính trên miền range-time-azimuth-time.
#### 1.1.2.1 Correlation DC Estimator (CDCE)
- Thuật toán này tính được FDC từ Average Cross Correlation Coefficient (ACCC)
- Algorithm Overview: Đọc [R-2], [R-5 Section 12.4.2], [R-10]
- Các bước thực thi thuật toán: 
	1. Tính ACCC trên mỗi azimuth line: 
![[Pasted image 20260812165648.png|502]]
Lưu ý rằng bước trên có thể được thực hiện song song cho toàn azimuth lines.
2. Chia ACCC vector thành các vector con cho mỗi range blocks, sau đó tính trung bình với mỗi vector con đó. Sau bước này sẽ thu được giá trị ACCC được trung bình theo range theo mỗi range block. 
3. Tính giá trị góc của mỗi ACCC được cho từ bước 2, $\phi_{accc}$.
4. Tính tần số FDC cho mỗi  $\phi_{accc}$ lấy từ bước 3 (hay tính tần số DC tương ứng với mỗi range block):
![[Pasted image 20260812170443.png|435]] 
- Đầu ra tương ứng với mỗi azimuth block đầu vào, sẽ là một tập các giá trị ước lượng FDC, mỗi giá trị tương ứng với slant range ở tâm của một range block. 
- Tập hợp này có thể biểu diễn bằng một bộ các cặp giá trị fast time $\tau$ và tần số FDC:
![[Pasted image 20260812170944.png|449]]
### 1.1.3 Giải quấn các ước lượng FDC
- Với mỗi azimuth block, các ước lượng FDC phải được giải quấn theo chiều range, bằng cách loại bỏ các bước nhảy lớn hơn PRF/2 trên các giá trị ước lượng liên tục. 
- Để cho ra kết quả tốt nhất, các ước lượng từ các khối nếu quá nhiễu hoặc biased (dựa trên kết quả từ bước trước) phải được loại bỏ hoặc gán trọng số thấp hơn trong quá trình giải quấn. 
- IPF sử dụng một thuật toán cải tiến được đề xuất và mô tả trong [R-11]
- Algorithm overview: Đọc tài liệu hãng.
- Triển khai thuật toán: Đầu vào thuật toán sẽ là kết quả của (5-20) của azimuth block hiện tại: 
	1. Tính vector $\vec{u}$: ![[Pasted image 20260812172515.png]]
	2. Zero-pad vector $\vec{u}$ tới độ dài của FFT
	3. Ước lượng thành phần tuyến tính của tần số DC (là một hàm của fasst time) bằng cách tính hệ số $a$ và $b$ dựa trên phương trình (5-24):![[Pasted image 20260812173016.png]]
		![[Pasted image 20260812173104.png|469]]
	