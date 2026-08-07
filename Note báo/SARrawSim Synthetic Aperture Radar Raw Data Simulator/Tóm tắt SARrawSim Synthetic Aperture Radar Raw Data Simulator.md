# 2. Mô tả phần mềm 
## 2.1 Brief presentation of SAR basics
![[Pasted image 20260807161919.png]]
Khi radar di chuyển, nó phát ra các chuỗi xung có độ rộng xung $\tau$ sau mỗi $t_R$ giây, $\tau < \tau_R$.
- Chuỗi các xung được biểu diễn bởi: 
$$
\sum_{n=0}^N p_{RF}(t-nt_R), \tag{1}
$$
với $p_{RF}(t)$ là tín hiệu RF chirp ở dạng số phức, có thể biểu diễn bằng: 
$$
p_{RF}(t) = rect\left(\frac{t}{\tau}\right)exp(j2\pi f_0t+j\pi k t^2), \tag{2}
$$
với $f_0$ là tần số thấp nhất của xung, $k = B/\tau$ là chirp rate, $B$ là độ rộng băng thông.
- Sau khi truyền một xung từ một vị trí $u$, radar lấy mẫu tín hiện truyền về từ mặt đất với fast time rate $T_S$, trong khoảng thời gian từ $\tau$ đến $\tau_R - \tau$. Các samples đó được xếp thành 1 hàng trong ma trận, được gọi là range line. 
- Các hàng này được thêm mới mỗi $\tau_R$.
- Theo góc độ xử lý tín hiệu, tín hiệu phản xạ thu được dọc theo một đường range là kết quả của phép tích chập giữa tín hiệu truyền đi và đáp ứng xung của vùng mặt đất được chiếu sáng bởi antenna pattern. 
- Vùng được chiếu sáng này được xác định bởi độ rộng búp sóng tại mức half-power beamwidth theo cả hai phương azimuth và phương elevation. 
- Đáp ứng xung này, còn được gọi là hàm mục tiêu lý tưởng của địa hình (terrain's ideal target function [42]), biểu diễn một phân phối đặc tính phản xạ địa hình 2D liên tục. 
- Tuy nhiên, với một địa hình lý tưởng, hàm này có thể được mô hình thành một bộ rời rạc các điểm tán xạ theo dạng sau:
$$
f_0(x,y) = \sum_n\sigma_n\delta(x-x_n)\delta(y-y_n), \tag{3}
$$
với $\sigma_n$ là radar cross section (RCS) của điểm tán xạ tại vị trí $(x_n, y_n)$ trên mặt đất.
- Để đơn giản, ta chỉ xét một mục tiêu điểm trên tọa độ $(u_0, r_0)$ trên slant-plane:
$$
f_0(u,r) = \sigma_0 \delta(u-u_0)\delta(r - r_0), \tag{4}
$$
với $\sigma_0$ là RCS của mục tiêu được chọn. 
- Tín hiệu base-band analytical được tạo bằng cách trộn tín hiệu phả xạ và tín hiệu mang, được biểu diễn dưới dạng: 
$$
e(t) = A_0exp\left[-j2\pi f_0 \Delta + j\pi k (t-\Delta)^2 \right] + n(t), \tag{5}
$$
với $A_0$ là hằng số phụ thuộc vào độ phản xạ của mục tiêu $\sigma_0$, $\Delta  = \frac{2R}{c}$ là delay 2 chiều của tín hiệu tới mục tiêu và phản xạ về, với $R = \sqrt{(u-u_0)^2+r_0^2}$ , $n(t)$ là white noise, dùng để mô phỏng nhiễu nhiệt. 
- Phương trình tổng quát từ (5) cho trường hợp N mục tiêu điểm là:
$$
e(t) = \sum_m^N A_mexp\left[-j2\pi f_0 \Delta_m + j\pi k (t-\Delta_m)^2 \right] + n(t), \tag{6}
$$
>[!note]
>Furthermore, for non-ideal terrain, given the coherent nature of SAR radiation and its interaction with the terrain’s roughness, the echo is a linear coherent superposition of the individual scattered waves from a set of sub-scatterers within a pixel. Consequently, from pulse to pulse, this superposition results in signal-dependent noise known as speckle, which is multiplicative in nature.
>For man-made targets, speckle is less pronounced, and the scattering
process can be considered deterministic. For simulation purposes, manmade targets in the optical region can be effectively modeled using asymptotic methods such as the ray tracing algorithm [43]. ==However, a simpler model based on a set of point scatterers often suffices for precise simulations with reduced computational cost==


