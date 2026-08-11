**Doppler Centroid** ($f_{DC}$) trong SAR có thể hiểu ngắn gọn là:

> **Tần số Doppler nằm tại tâm của phổ Doppler của tín hiệu phản xạ thu được theo phương azimuth.**

Nó cho biết **phổ Doppler của mục tiêu/scene đang bị lệch bao nhiêu so với Doppler bằng 0**.

### 1. Doppler xuất hiện từ đâu?

Vệ tinh SAR đang chuyển động với vận tốc $\mathbf{v}$, nên khoảng cách từ radar đến một điểm trên mặt đất thay đổi theo slow-time $\eta$:

$R = R(\eta)$

Pha của tín hiệu phản xạ có dạng:

$\phi(\eta)=-\frac{4\pi}{\lambda}R(\eta)$

Do đó tần số Doppler tức thời là:

$f_D(\eta)=\frac{1}{2\pi}\frac{d\phi}{d\eta}=-\frac{2}{\lambda}\frac{dR(\eta)}{d\eta}$

Nếu xét vận tốc tương đối theo phương Line-of-Sight:

$f_D=-\frac{2v_{LOS}}{\lambda}$

Đây chính là nguồn gốc của Doppler trong SAR.

### 2. Vậy Doppler Centroid là gì?

Trong thời gian radar bay qua một mục tiêu, mục tiêu không chỉ có **một** tần số Doppler. Doppler thay đổi liên tục:

$f_D(\eta)$

Ví dụ phổ Doppler thu được có thể nằm quanh:

```text
Amplitude
   ^
   |                 /\
   |               /    \
   |             /        \
   |___________/____________\________> f_D
              250          750 Hz
                    ↑
                 500 Hz
                  f_DC
```

Trong ví dụ này:

$f_{DC}=500\ \text{Hz}$

Nghĩa là **tâm năng lượng của phổ Doppler nằm ở khoảng 500 Hz**, chứ không nằm tại 0 Hz.

Vì vậy cần phân biệt:

- $f_D(\eta)$: Doppler tức thời của một scatterer.
    
- $f_{DC}$: **tâm của phổ Doppler** quan sát được.
    

### 3. Trường hợp lý tưởng broadside

Giả sử antenna nhìn chính xác vuông góc với hướng bay.

Khi radar tiến đến mục tiêu:

```text
                 hướng bay
        ---------------------------->

             SAR
              ●
              |
              | broadside
              |
              ↓
              X target
```

Tại thời điểm closest approach:

$\frac{dR}{d\eta}=0$

nên:

$f_D=0$

Trong trường hợp broadside lý tưởng, phổ Doppler gần đối xứng quanh 0:

```text
               /\
             /    \
-----------/--------\----------> f
         -B/2       +B/2
               ↑
             f_DC=0
```

Do đó:

$f_{DC}\approx0$

### 4. Khi antenna bị squint

Nếu antenna nhìn hơi về phía trước hoặc phía sau:

```text
                  hướng bay
        ---------------------------->

             SAR ●
                  \
                   \ LOS
                    \
                     X target
```

Lúc target nằm ở **tâm antenna beam**, radar và target vẫn có vận tốc tương đối theo LOS.

Do đó:

$v_{LOS}\neq0$

và:

$f_{DC}\neq0$

Ví dụ:

```text
                         /\
                       /    \
---------------------/--------\--------> f
                   300        700 Hz
                         ↑
                       500 Hz
                        f_DC
```

Toàn bộ phổ Doppler đã dịch khỏi zero.

### 5. Ý nghĩa hình học của $f_{DC}$

[Inference] Với mô hình hình học đơn giản, nếu $\theta_{sq}$ là squint angle theo một quy ước góc xác định, Doppler centroid thường có quan hệ dạng:

$f_{DC}\approx\frac{2v}{\lambda}\sin\theta_{sq}$

Dấu $+$/$-$ phụ thuộc vào quy ước hướng vận tốc, LOS và định nghĩa squint angle.

Điểm quan trọng là:

$\boxed{\text{squint angle}\longleftrightarrow v_{LOS}\longleftrightarrow f_{DC}}$

Do đó Doppler Centroid mang thông tin về **hình học quan sát của antenna so với hướng chuyển động**.

### 6. Tại sao SAR processor phải biết Doppler Centroid?

Đây là phần rất quan trọng khi xử lý **L0 → L1 SLC**.

Azimuth compression về bản chất thực hiện matched filtering với lịch sử pha/Doppler của target. Có thể hình dung Doppler history gần closest approach dưới dạng:

$f_D(\eta)\approx f_{DC}+K_a(\eta-\eta_c)$

trong đó:

- $f_{DC}$: Doppler centroid,
    
- $K_a$: Doppler rate,
    
- $\eta_c$: thời điểm beam/target đạt vị trí tham chiếu.
    

Ví dụ:

```text
f_D
 ^
 |                   /
 |                 /
 |---------------●------------ f_DC
 |             /
 |           /
 |         /
 +-----------------------------> η
                 ηc
```

Nếu processor giả sử:

$f_{DC}=0$

trong khi dữ liệu thực tế:

$f_{DC}=500\ \text{Hz}$

thì bộ lọc azimuth không được đặt đúng vị trí phổ Doppler của dữ liệu. Điều này ảnh hưởng đến các bước như **azimuth compression, Doppler ambiguity handling, RCMC và định vị mục tiêu**, tùy thuật toán tạo ảnh được sử dụng.

### 7. Một điểm rất quan trọng với Sentinel-1

Doppler Centroid **không nhất thiết là một hằng số duy nhất cho toàn bộ ảnh**.

Nó có thể thay đổi theo:

$f_{DC}=f_{DC}(\eta,\tau)$

với:

- $\eta$: azimuth/slow time,
    
- $\tau$: range/fast time.
    

Vì vậy khi bạn thấy metadata Sentinel-1 chứa **nhiều Doppler Centroid estimates**, không nên hiểu rằng Sentinel-1 đang chọn ngẫu nhiên nhiều giá trị cho cùng một ảnh. Chúng đại diện cho sự biến thiên của Doppler centroid theo vị trí/thời gian và được mô hình hóa để processor sử dụng.

---

Có thể chốt lại bằng một chuỗi rất dễ nhớ:

**Vệ tinh chuyển động**  
$\downarrow$  
**Slant range thay đổi**  
$\downarrow$  
**Pha echo thay đổi theo azimuth**  
$\downarrow$  
**Xuất hiện Doppler**  
$\downarrow$  
**Echo có một Doppler spectrum**  
$\downarrow$  
$\boxed{f_{DC}=\text{tần số tại tâm Doppler spectrum}}$

Và điểm dễ nhầm nhất là:

$\boxed{f_{DC}\neq\text{Doppler bandwidth}}$

**Doppler bandwidth** nói phổ **rộng bao nhiêu**, còn **Doppler Centroid** nói phổ **nằm ở đâu trên trục tần số**.