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
- Vùng được chiếu sáng này được xác định bởi độ rộng búp sóng tại 