Từ [[Tóm tắt Sentinel-1 Level 0 Data Decoding Example]], tác giả có đề cập rằng đoạn demo đang bỏ qua một số bước so với thuật toán được sử lý bởi IPF ESA, có thể kể đến như: 
- Tác giả đang mặc định Doppler Centroid = 0 Hz. 
- Tác giả chưa áp dụng bước Secondary Range Compression (SRC)
Bản demo này sẽ thực hiện bổ sung 2 bước thành phần vào luồng xử lý của thuật toán, bao gồm: 
1. Doppler Centroid Estimation 
2. Secondary Range Compression
từ đó so sánh độ cải thiện của chất lượng ảnh.

# 1. Cơ sở lý thuyết
## 1.1 Doppler Centroid Estimation
Các bước thực hiện thuật toán DCE được thực hiện bởi ESA nhìn chung có thể được chia làm 4 bước chính như sau: 
1. Tính giá trị Doppler Centroid tuyệt đối (từ thông tin quỹ đạo và tư thế của vệ tinh)
2. Tính ước lượng Fine Doppler Centroid từ dữ liệu thô. 
3. Tìm Absolute Doppler Centroid từ FDC + DC từ geometry. 
4. Fitting để tìm Doppler Centroid polynomial cho azimuth block.
==Câu hỏi: Có được DC polynomial từ bước này rồi, vậy sử dụng polynomial này ở đâu?==

Câu hỏi: Thử kiểm tra 8.2.1 Stripmap Case xem