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
- Sau khi truyền một xung từ một vị trí $u$, radar lấy mẫu tín hiện truyền về từ mặt đất với fasst time rate $T_S$, trong khoảng thời 