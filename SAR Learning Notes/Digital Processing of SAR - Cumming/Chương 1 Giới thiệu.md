## 1.1 Brief background of SAR

Nói qua về lịch sử hình thành của công nghệ radar
## 1.2 Radar in Remote sensing

Công nghệ SAR được sử dụng phổ biến nhờ 3 yếu tố chính:
- Có radar làm nguồn phát, hoạt động kể cả khi trời tối
- Sóng điện từ của các tần số radar thông dụng xuyên qua mây, mưa, tuyết,etc với suy hao không đáng kể
- Năng lượng phát xạ lại mang tính chất về bề mặt, vật liệu. 
## 1.3 SAR fundamentals

### Different modes of SAR operation

Có thể kể đến một vài chế độ: 
- Stripmap: giữ antenna nguyên 1 góc.
- ScanSAR: Quét búp sóng theo chiều range, đổi lại là giảm độ phân dải theo chiều azimuth
- SpotSAR: Chỉ soi một vùng quan tâm (giữ búp sóng trên vùng đó).
- InSAR: Vật được quan sát đứng im, radar di chuyển, thường thấy ở ground-based SAR
- BistaticSAR: Phát và thu ở 2 vị trí khác nhau.
- InSAR: Kĩ thuật hậu xử lý dữ liệu SAR, nhân liên hợp 2 ảnh phức được chụp từ cùng 1 vị trí không gian (differential InSAR) hoặc lệch 1 chút (terrain height InSAR), kết quả thu được 1 giao thoa đồ (interferogram) với các contoures biểu diễn sự dịch chuyển hoặc độ cao địa hình.
Sách tập chung vào chế độ Stripmap và ScanSAR, hoạt động ở chế độ monostatic.

### SAR Resolution

- Độ phân dải theo chiều range: $\Delta R_{range} = \frac{c}{2B}$. Để tăng độ phân giải, dùng kĩ thuật nén xung theo chiều range (chương 3).
- Độ phân dải theo chiều azimuth, nếu là radar thông thường, bằng độ rộng búp sóng x khoảng cách tới điểm phát xạ. Đối với SAR, dựa vào cơ chế mỗi điểm phát xạ có 1 độ dịch Doppler riêng, nhờ cơ chế đó, độ phân giải azimuth được giảm xuống: $\Delta R_{azimuth} = \frac{L_a}{2}$, bằng một nửa độ rộng ăng ten, độc lập với khoảng cách. Vậy nên thiết kế ăng ten của SAR nhỏ là tốt, nhưng nếu ăng ten quá nhỏ hoặc độ dài synthetic quá lớn thì sẽ giảm SNR (chương 4).
### SNR
- SNR trong final image là 1 tham số quan trọng trong hệ thống SAR.
![[Pasted image 20260719210415.png]]
- Công suất phát trung bình $P_{ave}$:![[Pasted image 20260719212551.png]]
với $P_T$ là công suất truyền đỉnh, $F_a$ là PRF, $T_r$ là độ dài xung phát.
- Với một mục tiêu rời rạc và nằm trong 1 pixel của ảnh (để năng lượng nằm hoàn toàn trong 1 pixel), ta có SNR của target là:![[Pasted image 20260719213015.png]]
với $p_r, p_a$ là độ phân dải theo chiều range và chiều azimuth, $\sigma_t$  là RCS thực (không chuẩn hóa) của target. 
- Một lưu ý quan trọng là SNR ở SAR khác với radar bình thường ở chỗ tỉ lệ với $1/R^3$  thay vì $1/R^4$.
### Range cell migration 
- Radar di chuyển khi trong quá trình chụp làm range thay đổi, xảy ra hiện tượng RCM.
- Nếu sự thay đổi theo range, lớn hơn một sample (1 cell) thì được tính là significant và cần phải được xử lý.
### Spaceborne SAR Sensor
- Sách tập trung vào spaceborne SAR. Sensor tập trung là SEASAT.
- SEASAT: L-Band(1.27 GHz), góc incidence là 23$\degree$  