Được. Tôi đã đối chiếu trực tiếp **Section 5 - Doppler Centroid Estimation Algorithms, từ trang 5-1 đến 5-13** của file _Sentinel-1 Level 1 Detailed Algorithm Definition, Issue 2/5_. Nếu xét **đúng pipeline Stripmap**, thì gần như toàn bộ Section 5 được sử dụng, **ngoại trừ phương trình (5-17) và phần (5-31), vì hai phần này dành riêng cho TOPSAR/multi-swath**.

Điểm quan trọng nhất trước khi đi vào từng phương trình là Sentinel-1 **không đơn giản lấy Doppler centroid bằng một phép ước lượng duy nhất từ raw echo**. IPF kết hợp hai nguồn:

- **Geometry-based DC**: tính từ orbit + attitude, chủ yếu để xác định Doppler ambiguity.
    
- **Data-based Fine DC**: đo chính xác phần Doppler nằm trong baseband $[-PRF/2,+PRF/2]$ từ echo SAR.
    
- Sau đó **unwrap + ambiguity resolution + polynomial fitting** để thu được hàm $f_{DC}$ cuối cùng theo range.
    

Figure 5-1 và 5-2 của tài liệu thể hiện đúng tư tưởng này. Nguồn DC cuối cùng còn có thể cấu hình là geometry, data hoặc một polynomial định trước; nếu quá trình data-driven gặp lỗi/không đạt ngưỡng chất lượng thì geometry DC là fallback.

---

# 1. Toàn bộ pipeline Doppler Estimation cho Stripmap

Đối với **Sentinel-1 Stripmap**, tôi sẽ viết pipeline thành:

$$  
\boxed{  
\text{Orbit/Attitude}  
\rightarrow  
\text{Geometry DC}  
}  
$$

song song với

$$  
\boxed{  
L0  
\rightarrow  
\text{BAQ/FDBAQ Decode}  
\rightarrow  
\text{Raw Correction}  
\rightarrow  
\text{Drift Compensation}  
\rightarrow  
\text{Range Compression}  
\rightarrow  
\text{CDCE}  
}  
$$

sau đó hai nhánh gặp nhau:

$$  
\boxed{  
\text{Fine DC}  
\rightarrow  
\text{Unwrapping}  
\rightarrow  
\text{Ambiguity Resolution bằng Geometry DC}  
\rightarrow  
\text{Absolute DC}  
\rightarrow  
\text{Polynomial Fit}  
}  
$$

Kết quả cuối cùng cho mỗi **azimuth block** của Stripmap là một đa thức Doppler centroid theo range, về cơ bản:

$$  
f_{DC}(\tau)=c_0+c_1\tau+c_2\tau^2  
$$

Đây mới là thứ SLC processor sử dụng ở những bước xử lý sau.

---

# 2. Bước 1 - Absolute DC từ Orbit & Attitude

## 2.1. Tại sao phải tính Doppler từ geometry?

Sentinel-1 trước hết luôn tính Doppler từ:

- vị trí vệ tinh $\mathbf p$,
    
- vận tốc vệ tinh $\mathbf v$,
    
- attitude,
    
- slant range $r$.
    

Geometry DC có thể không đủ chính xác để focus trực tiếp do sai số attitude/beam pointing, nhưng tài liệu giả thiết rằng nó **đủ tin cậy để xác định Doppler ambiguity**. Đây là vai trò cực kỳ quan trọng của nó.

---

# 3. Phương trình (5-1) - Doppler của một target

Với platform có relative velocity $\mathbf v$, target có slant range $r$ và view vector $\mathbf r$:

$$  
\boxed{  
f=-\frac{2,\mathbf v\cdot\mathbf r}{\lambda r}  
}  
\tag{5-1}  
$$

Trong đó:

- $\lambda$: bước sóng radar;
    
- $\mathbf v$: relative velocity của vệ tinh;
    
- $\mathbf r$: vector từ vệ tinh đến target;
    
- $r=|\mathbf r|$.
    

Vì:

$$  
\mathbf r=r\mathbf r_0  
$$

với $\mathbf r_0$ là unit look vector, nên có thể viết:

$$  
f=-\frac{2}{\lambda}\mathbf v\cdot\mathbf r_0  
$$

Ý nghĩa vật lý rất rõ: Doppler được quyết định bởi **thành phần radial của vận tốc tương đối theo hướng satellite-target**.

---

# 4. Phương trình (5-2) - Doppler Frequency Function

Tài liệu định nghĩa **Doppler Frequency Function - DFF**:

# $$  
\boxed{  
f(r,\boldsymbol{\varphi})

-\frac{2}{\lambda}  
\mathbf v\cdot  
\mathbf r_0(r,\boldsymbol{\varphi})  
}  
\tag{5-2}  
$$

Trong đó $\boldsymbol{\varphi}$ biểu diễn attitude.

Điểm cần chú ý là:

$$  
f_{DC}=f(r,\boldsymbol{\varphi})  
$$

không nhất thiết là hằng số theo range.

Do look direction thay đổi từ near range tới far range:

$$  
\mathbf r_0=\mathbf r_0(r,\boldsymbol{\varphi})  
$$

nên:

$$  
f_{DC}=f_{DC}(r)  
$$

Đây chính là lý do sau này Sentinel-1 phải fit Doppler centroid thành polynomial theo range.

---

# 5. Tìm unit view vector $\mathbf r_0$

Đây là phần từ (5-3) đến (5-8). Figure 5-3 của tài liệu mô tả hình học với:

- $\mathbf p$: vector từ tâm Trái Đất tới vệ tinh;
    
- $\mathbf g$: vector từ tâm Trái Đất tới ground target;
    
- $\mathbf r$: satellite → target;
    
- $\mathbf v$: satellite relative velocity;
    
- $\mathbf u_0$: vector định hướng bởi attitude.
    

## Phương trình (5-3)

Quan hệ hình học:

$$  
\boxed{  
\mathbf g(r,\boldsymbol{\varphi})

\mathbf p-r\mathbf r_0(r,\boldsymbol{\varphi})  
}  
\tag{5-3}  
$$

Tức:

# $$  
\text{ground target}

## \text{satellite position}

\text{slant-range vector}  
$$

---

# 6. Phương trình (5-4) - Beam pointing constraint

$$  
\boxed{  
\mathbf u_0(\boldsymbol{\varphi})  
\cdot  
\mathbf r_0(r,\boldsymbol{\varphi})=0  
}  
\tag{5-4}  
$$

Đây là một constraint liên quan tới orientation của spacecraft/antenna.

---

# 7. Phương trình (5-5) - $\mathbf r_0$ phải là unit vector

$$  
\boxed{  
\mathbf r_0(r,\boldsymbol{\varphi})  
\cdot  
\mathbf r_0(r,\boldsymbol{\varphi})  
=1  
}  
\tag{5-5}  
$$

hay:

$$  
|\mathbf r_0|=1  
$$

---

# 8. Phương trình (5-6) - Target phải nằm trên ellipsoid Trái Đất

Tài liệu định nghĩa:

$$  
\boxed{  
\mathbf g_s(r,\boldsymbol{\varphi})  
\cdot  
\mathbf g_s(r,\boldsymbol{\varphi})  
=1  
}  
\tag{5-6}  
$$

với vector normalized ellipsoid:

# $$  
\boxed{  
\mathbf g_s(r,\boldsymbol{\varphi})

\begin{bmatrix}  
g_x/a\  
g_y/a\  
g_z/b  
\end{bmatrix}  
}  
\tag{5-7}  
$$

trong đó:

- $a$: Earth major semi-axis;
    
- $b$: Earth minor semi-axis.
    

Thế (5-7) vào (5-6) cho ta:

$$  
\frac{g_x^2}{a^2}  
+  
\frac{g_y^2}{a^2}  
+  
\frac{g_z^2}{b^2}  
=1  
$$

chính là phương trình ellipsoid.

Target vector được viết:

# $$  
\boxed{  
\mathbf g(r,\boldsymbol{\varphi})

\begin{bmatrix}  
g_x&g_y&g_z  
\end{bmatrix}^{T}  
}  
\tag{5-8}  
$$

Như vậy (5-3)...(5-8) tạo thành một **hệ phương trình phi tuyến** để tìm $\mathbf r_0$ ứng với một $r$ cụ thể.

---

# 9. Phương trình (5-9) - Xây dựng vector định hướng $\mathbf u_0$

Attitude của spacecraft được đưa vào qua:

# $$  
\boxed{  
\mathbf u_0(\boldsymbol{\varphi})

\mathbf L_0(\mathbf p,\mathbf v_a)  
\mathbf L(\boldsymbol{\varphi})  
\begin{bmatrix}  
1\  
0\  
0  
\end{bmatrix}  
}  
\tag{5-9}  
$$

Có hai phép biến đổi:

$$  
\mathbf L(\boldsymbol{\varphi})  
$$

chuyển theo attitude, còn

$$  
\mathbf L_0(\mathbf p,\mathbf v_a)  
$$

liên quan tới hệ tọa độ orbit.

---

# 10. Phương trình (5-10) - Absolute satellite velocity

Tài liệu phân biệt relative velocity $\mathbf v$ với absolute velocity $\mathbf v_a$:

# $$  
\boxed{  
\mathbf v_a

\mathbf v+\mathbf w\times\mathbf p  
}  
\tag{5-10}  
$$

Trong đó $\mathbf w$ là Earth sidereal rotational velocity.

Tài liệu cho thành phần $z$:

$$  
w_z=0.00007292115833\ \mathrm{rad/s}  
$$

Điều này bổ sung chuyển động do Trái Đất quay vào vận tốc tuyệt đối.

---

# 11. Xây dựng orbit matrix $\mathbf L_0$

Orbit matrix được định nghĩa:

# $$  
\boxed{  
\mathbf L_0(\mathbf p,\mathbf v_a)

\begin{bmatrix}  
\mathbf a&\mathbf b&\mathbf c  
\end{bmatrix}  
}  
\tag{5-11}  
$$

Lưu ý $\mathbf a,\mathbf b,\mathbf c$ ở đây là **các vector basis**, không phải $a,b$ semi-axis của ellipsoid.

Vector $\mathbf c$:

$$  
\boxed{  
\mathbf c=-\frac{\mathbf q}{|\mathbf q|}  
}  
\tag{5-12}  
$$

với:

$$  
\mathbf q=  
\begin{bmatrix}  
p_x\  
p_y\  
\dfrac{p_z}{1-e^2}  
\end{bmatrix}  
$$

trong đó $e$ là eccentricity của Earth spheroid.

Sau đó:

# $$  
\boxed{  
\mathbf b

\frac{\mathbf c\times\mathbf v_a}  
{|\mathbf c\times\mathbf v_a|}  
}  
\tag{5-13}  
$$

và:

$$  
\boxed{  
\mathbf a=\mathbf b\times\mathbf c  
}  
\tag{5-14}  
$$

Kết quả là một hệ basis định nghĩa orientation liên quan tới quỹ đạo.

---

# 12. Phương trình (5-15) - Attitude matrix

Sentinel-1 sử dụng roll $\varphi$, pitch $\theta$, yaw $\Psi$:

# $$  
\boxed{  
\mathbf L(\boldsymbol{\varphi})

\begin{bmatrix}  
\cos\Psi&-\sin\Psi&0\  
\sin\Psi&\cos\Psi&0\  
0&0&1  
\end{bmatrix}  
\begin{bmatrix}  
\cos\theta&0&\sin\theta\  
0&1&0\  
-\sin\theta&0&\cos\theta  
\end{bmatrix}  
\begin{bmatrix}  
1&0&0\  
0&\cos\varphi&-\sin\varphi\  
0&\sin\varphi&\cos\varphi  
\end{bmatrix}  
}  
\tag{5-15}  
$$

Sentinel-1 sử dụng rotation order **3-2-1**.

Sau khi có $\mathbf L$, ta có $\mathbf u_0$, từ đó giải hệ (5-3)...(5-8) để lấy $\mathbf r_0$, rồi đưa $\mathbf r_0$ vào (5-2) để tính Doppler.

---

# 13. Phương trình (5-16) - Geometry DC tại các range block

Sentinel-1 không tính geometry DC cho từng raw range sample. IPF tính trên một **sub-sampled grid**.

Với $n$ range blocks:

$$  
\boxed{  
(r_i,\tilde f_i),  
\qquad  
i=0,1,\ldots,n-1  
}  
\tag{5-16}  
$$

Trong đó:

- $r_i$: slant range tại **tâm range block thứ $i$**;
    
- $\tilde f_i$: absolute DC tính từ geometry.
    

Sau đó polynomial được fit qua tập các điểm này.

Đây là nhánh:

$$  
\boxed{  
Orbit+Attitude  
\rightarrow  
{(r_i,\tilde f_i)}  
}  
$$

---

# 14. Bước 2 - Chuẩn bị dữ liệu cho Fine DC Estimation

Geometry DC chưa đủ chính xác, nên Sentinel-1 ước lượng **fine/baseband Doppler** trực tiếp từ SAR data.

Do sampling theo azimuth có sampling frequency bằng PRF, Doppler quan sát được chỉ nằm trong:

$$  
\boxed{  
-\frac{PRF}{2}  
\leq  
f  
\leq  
+\frac{PRF}{2}  
}  
$$

Do đó fine DC từ dữ liệu chỉ cung cấp **fractional/baseband part** của Doppler.

Trước CDCE, đối với từng range line IPF thực hiện:

1. Raw data decoding.
    
2. Raw data correction.
    
3. Drift compensation.
    
4. Range compression.
    

Đặc biệt, **FDBAQ decoding được thực hiện lại inline trong DCE processor** trước fine DC estimation; tài liệu nói rõ DCE giải mã L0 range-line-by-range-line.

---

# 15. Phương trình (5-17) - KHÔNG áp dụng cho Stripmap

Tài liệu có:

# $$  
d(\eta)

\exp  
\left[  
j\pi k_s  
\left(  
\eta-\frac{T_b}{2}  
\right)^2  
\right]  
\tag{5-17}  
$$

Đây là **DCE pre-conditioning dành riêng cho TOPSAR**.

Trong IW/EW, dữ liệu burst phải được de-ramp trước CDCE do azimuth beam steering.

### Với Stripmap:

$$  
\boxed{  
\text{Không thực hiện (5-17)}  
}  
$$

Stripmap đi trực tiếp:

$$  
\text{Range-compressed data}  
\rightarrow  
\text{CDCE}  
$$

---

# 16. Bước 3 - Correlation Doppler Centroid Estimator

Đây là phần quan trọng nhất của **data-driven Doppler estimation**.

CDCE là một estimator dựa vào **phase difference giữa các echo azimuth liên tiếp**.

Nếu Doppler centroid là $f_{DC}$, một cách trực giác, tín hiệu azimuth giữa hai pulse cách nhau:

$$  
\Delta\eta=\frac{1}{PRF}  
$$

sẽ có phase increment liên quan tới:

$$  
2\pi f_{DC}\Delta\eta  
$$

Sentinel-1 khai thác chính phase increment đó.

---

# 17. Phương trình (5-18) - ACCC lag-one

Tài liệu tính Average Cross Correlation Coefficient ở lag 1:

# $$  
\boxed{  
c(\eta)

\sum_{\eta}  
s^*(\eta)  
s(\eta+\Delta\eta)  
}  
\tag{5-18}  
$$

với:

$$  
\boxed{  
\Delta\eta=\frac{1}{PRF}  
}  
$$

và:

- $s(\eta)$: radar sample;
    
- $s(\eta+\Delta\eta)$: sample azimuth kế tiếp;
    
- $*$: complex conjugate.
    

Ý nghĩa của tích:

$$  
s^*(\eta)s(\eta+\Delta\eta)  
$$

nằm ở phase của nó.

Nếu:

$$  
s(\eta)=A(\eta)e^{j\phi(\eta)}  
$$

thì:

# $$  
s^*(\eta)s(\eta+\Delta\eta)

A(\eta)A(\eta+\Delta\eta)  
e^{j[\phi(\eta+\Delta\eta)-\phi(\eta)]}  
$$

Do đó phase của correlation chứa thông tin về:

# $$  
\Delta\phi

\phi(\eta+\Delta\eta)-\phi(\eta)  
$$

và từ đó suy ra Doppler centroid.

Phần giải thích đại số ở đoạn này là diễn giải trực tiếp ý nghĩa của estimator; công thức triển khai chính thức của IPF vẫn là (5-18).

---

# 18. Chia ACCC thành các range block

Sau khi có ACCC vector, IPF:

- chia nó thành $n$ sub-vectors tương ứng với $n$ range blocks;
    
- lấy average trong từng block;
    
- mỗi range block thu được một complex ACCC;
    
- lấy phase của complex ACCC:
    

# $$  
\phi_{\mathrm{accc}}

\angle C_{\mathrm{ACCC}}  
$$

Đây là cách giảm ảnh hưởng noise bằng cách average nhiều range samples trong cùng range block.

---

# 19. Phương trình (5-19) - Fine Doppler Centroid

Từ phase $\phi_{\mathrm{accc}}$:

# $$  
\boxed{  
f_{\eta c}

-\frac{PRF}{2\pi}  
\phi_{\mathrm{accc}}  
}  
\tag{5-19}  
$$

Đây là một trong những phương trình quan trọng nhất của toàn bộ DCE.

Do:

$$  
-\pi\leq\phi_{\mathrm{accc}}\leq\pi  
$$

nên:

$$  
-\frac{PRF}{2}  
\leq  
f_{\eta c}  
\leq  
+\frac{PRF}{2}  
$$

Vì vậy estimator này **không biết Doppler absolute nằm ở PRF ambiguity thứ mấy**.

Ví dụ về mặt cấu trúc:

# $$  
f_{DC}^{absolute}

M\cdot PRF  
+  
f_{DC}^{fine}  
$$

Trong đó CDCE chỉ tìm được phần:

$$  
f_{DC}^{fine}  
$$

còn geometry giúp xác định $M$.

---

# 20. Phương trình (5-20) - Fine DC samples

Sau CDCE, đối với một azimuth block:

$$  
\boxed{  
(\tau_i,f_i),  
\qquad  
i=0,1,\ldots,n-1  
}  
\tag{5-20}  
$$

Trong đó:

- $\tau_i$: fast time tại tâm range block;
    
- $f_i$: fine Doppler centroid của block đó;
    
- $n$: số range blocks.
    

Do đó output của CDCE không phải:

$$  
f_{DC}=123\ \mathrm{Hz}  
$$

mà là một vector:

$$  
\begin{bmatrix}  
f_0\  
f_1\  
\vdots\  
f_{n-1}  
\end{bmatrix}  
$$

theo range.

---

# 21. Bước 4 - Fine DC unwrapping

Đây là bước rất quan trọng.

Do CDCE chỉ quan sát phase modulo $2\pi$, fine DC có thể xuất hiện các jump cỡ PRF giữa những range block kế tiếp.

Ví dụ về bản chất:

$$  
490,\quad 510,\quad -480,\quad -460  
$$

có thể thực ra đại diện cho một trend liên tục.

IPF không chỉ dùng một unwrap đơn giản bằng cách kiểm tra chênh lệch $PRF/2$, vì tài liệu nói cách đó dễ gây unwrap error. Thay vào đó Sentinel-1 dùng **FFT-based estimator**.

---

# 22. Phương trình (5-21) - Mô hình linear trend

Fine DC normalized được coi có một linear component:

$$  
\boxed{  
f_{\mathrm{linear}}(\tau)=a\tau+b  
}  
\tag{5-21}  
$$

Trong đó:

- $a$: slope theo range;
    
- $b$: intercept.
    

Lưu ý đây là mô hình dùng trong **unwrapping**, chưa phải quadratic DCE polynomial cuối cùng.

---

# 23. Phương trình (5-22) - Chuyển frequency thành phase phasor

Fine DC được normalize theo PRF:

# $$  
\boxed{  
F(\nu)

FFT  
\left[  
\exp  
\left(  
j2\pi  
\frac{  
f_{\eta c}(kT_S)  
}{PRF}  
\right)  
\right]  
}  
\tag{5-22}  
$$

Ý tưởng ở đây là thay vì unwrap trực tiếp một frequency sequence bị modulo PRF, ta ánh nó lên unit circle:

$$  
f  
\rightarrow  
e^{j2\pi f/PRF}  
$$

Khi $f$ lệch đi $PRF$:

# $$  
e^{j2\pi(f+PRF)/PRF}

e^{j2\pi f/PRF}  
$$

nên ambiguity do integer multiples of PRF biến mất trong representation này.

---

# 24. Phương trình (5-23) - Weighted FFT

Nếu mỗi estimate có weighting $w(\tau)$:

# $$  
\boxed{  
F(\nu)

FFT  
\left[  
w(kT_S)  
\exp  
\left(  
j2\pi  
\frac{  
f_{\eta c}(kT_S)  
}{PRF}  
\right)  
\right]  
}  
\tag{5-23}  
$$

Estimate kém chất lượng có thể được down-weight.

---

# 25. Tìm peak của FFT

Tài liệu định nghĩa normalized frequency tại peak:

# $$  
\boxed{  
\hat\nu

\arg\max_{\nu}|F(\nu)|^2  
}  
$$

Phương trình này không được đánh số riêng trong PDF nhưng nằm ngay trước (5-24).

Nếu các phasor theo range có linear phase ramp, FFT sẽ tập trung năng lượng tại một frequency $\hat\nu$.

Từ vị trí peak đó có thể suy ra slope $a$.

---

# 26. Phương trình (5-24) - Tìm $a$ và $b$

# $$  
\boxed{  
\hat a

\frac{\hat\nu}{T_S}  
}  
$$

và:

# $$  
\boxed{  
b

\frac{\angle F(\hat\nu)}{2\pi}  
}  
\tag{5-24}  
$$

Trong đó $T_S$ là range sampling interval **của các fine DC estimates**, tức spacing giữa các DCE range-block samples, không phải ADC raw sample interval.

---

# 27. Phương trình (5-25) - Tìm residual

Sau khi loại linear trend:

# $$  
\boxed{  
res(kT_S)

\frac{1}{2\pi}  
\angle  
\left[  
\exp  
\left(  
j2\pi  
\frac{f_{\eta c}(kT_S)}{PRF}  
\right)  
\exp  
\left(  
-j(akT_S+b)  
\right)  
\right]  
}  
\tag{5-25}  
$$

Ý nghĩa:

$$  
\text{measured phasor}  
\times  
\text{conjugate linear trend phasor}  
$$

để còn lại residual phase.

---

# 28. Phương trình (5-26) - Reconstruct unwrapped DC

# $$  
\boxed{  
\hat f_{\eta c}(kT_S)

\left[  
akT_S+b+res(kT_S)  
\right]PRF  
}  
\tag{5-26}  
$$

Như vậy:

# $$  
\text{Unwrapped DC}

\text{linear trend}  
+  
\text{residual}  
$$

rồi scale trở lại Hz bằng PRF.

---

# 29. Implementation thực tế của unwrapping

Section 5.3.2 viết lại thuật toán dưới dạng discrete implementation.

## Phương trình (5-27)

Tạo vector:

# $$  
\boxed{  
u_i

w_i  
\exp  
\left(  
j2\pi\frac{f_i}{PRF}  
\right),  
\qquad  
i=0,\ldots,n-1  
}  
\tag{5-27}  
$$

Sau đó IPF:

1. zero-pad $\mathbf u$ tới FFT length cấu hình;
    
2. FFT vector đã zero-pad;
    
3. tìm peak;
    
4. tính $a,b$ bằng (5-24).
    

---

# 30. Phương trình (5-28) - Residual discrete

# $$  
\boxed{  
res_i

\frac{1}{2\pi}  
\angle  
\left[  
u_i  
\exp  
\left(  
-j(a\tau_i+b)  
\right)  
\right],  
\qquad i=0,\ldots,n-1  
}  
\tag{5-28}  
$$

---

# 31. Phương trình (5-29) - Fine DC unwrapped vector

# $$  
\boxed{  
\hat f_i

(a\tau_i+b+res_i)PRF,  
\qquad  
i=0,\ldots,n-1  
}  
\tag{5-29}  
$$

Đây là output của fine-DC unwrapping.

---

# 32. Bước 5 - Resolve Doppler ambiguity bằng geometry

Đây là chỗ **data DC và geometry DC được ghép lại**.

Data estimator biết chính xác fine DC nhưng bị modulo PRF:

$$  
f_{\text{data}}  
\pmod{PRF}  
$$

Geometry DC thì không chính xác bằng nhưng biết absolute Doppler đang ở ambiguity nào.

---

# 33. Phương trình (5-30) - Ambiguity number

Từ geometry DC của **range block đầu tiên**:

$$  
\boxed{  
A=  
\left[  
\frac{\tilde f_0}{PRF}  
\right]  
}  
\tag{5-30}  
$$

Trong đó:

- $\tilde f_0$: absolute DC từ orbit/attitude của range block đầu tiên;
    
- $A$: Doppler ambiguity.
    

Tài liệu ở đoạn này viết toán tử bằng dấu ngoặc vuông như trên; đoạn văn của Section 5.4 không định nghĩa thêm loại phép làm tròn của ký hiệu ngoặc vuông, nên tôi giữ nguyên ký hiệu của tài liệu thay vì tự gán nó thành `floor` hay `round`.

---

# 34. Sáu bước tạo Absolute Data DC

Đây là các bước **nguyên văn về mặt thuật toán** mà Section 5.4 yêu cầu thực hiện cho mỗi azimuth block:

1. Unwrap FDC từ data trong baseband bằng Sections 5.2 và 5.3.
    
2. Offset unwrapped data FDC về ambiguity của geometry FDC.
    
3. Evaluate geometry FDC để tạo một array các FDC points trên toàn range.
    
4. Tính delta giữa unwrapped data FDC và geometry FDC:
    

# $$  
\Delta f_{DC}(\tau)

## f_{DC,data}(\tau)

f_{DC,geom}(\tau)  
$$

5. Polynomial-fit $\Delta f_{DC}$.
    
6. Cộng các coefficient của $\Delta f_{DC}$ vào geometry-FDC coefficients để tạo **data FDC polynomial cho azimuth block hiện tại**.
    

Tài liệu không đánh số phương trình riêng cho sáu phép này; chúng nằm ngay sau (5-30).

Điểm này rất đáng chú ý: Sentinel-1 không đơn thuần làm:

# $$  
f_{\text{absolute}}

A\cdot PRF+f_{\text{fine}}  
$$

rồi dừng lại. IPF còn so sánh data với geometry trên **toàn range**, fit phần delta và ghép các polynomial coefficients.

---

# 35. Bước 6 - Polynomial fitting cho Stripmap

Polynomial fitting được thực hiện **hai lần**:

- một polynomial cho DC từ orbit/attitude;
    
- một polynomial cho DC ước lượng từ data.
    

Đối với **single-swath Stripmap**, Sentinel-1 dùng quadratic polynomial:

$$  
\boxed{  
p(x)=c_0+c_1x+c_2x^2  
}  
$$

Đây là biểu thức trong Section 5.5 nhưng không mang số phương trình riêng.

Các coefficients được tìm bằng **least-squares fitting**.

Với data DC, quá trình fitting cũng phát hiện và loại bỏ outliers.

Vì vậy output cuối cho một azimuth block Stripmap có dạng:

# $$  
\boxed{  
f_{DC}(\tau)

c_0+c_1\tau+c_2\tau^2  
}  
$$

hoặc cùng dạng nhưng sử dụng range variable được định nghĩa trong processor.

---

# 36. Phương trình (5-31) - KHÔNG áp dụng cho Stripmap

Tài liệu có:

# $$  
\boxed{  
p_s(x)

c_{0s}+c_1x+c_2x^2,  
\qquad  
s=0,\ldots,S-1  
}  
\tag{5-31}  
$$

Đây là **TOPSAR multi-swath fitting**.

Các sub-swath dùng chung $c_1,c_2$, nhưng có thể có $c_{0s}$ khác nhau để xử lý step jump do electronic beam switching.

### Với Stripmap:

$$  
\boxed{\text{Không dùng (5-31)}}  
$$

vì Stripmap chỉ có một swath.

---

# 37. DC Quality Measurement

Sentinel-1 lưu ý rằng data-derived DC có thể bị bias bởi scene content.

Do đó least-squares fitting:

- phát hiện outlier;
    
- loại outlier;
    
- tính RMS difference giữa các DC estimate được chọn và fitted polynomial.
    

**DC quality indicator của IPF chính là RMS error này.**

Tài liệu **không đưa ra một phương trình đánh số riêng cho RMS error ở Section 5.5.1**; vì vậy tôi không tự bổ sung một công thức ngoài tài liệu.

Nếu quality threshold bị vi phạm, geometry-derived DC có thể được dùng làm fallback.

---

# 38. DCE được tính theo azimuth block như thế nào?

Section 5.6 rất quan trọng để hiểu cấu trúc dữ liệu.

Giả sử ảnh Stripmap có:

$$  
N_{\eta}  
$$

range lines theo azimuth.

IPF không tính một $f_{DC}$ duy nhất cho cả ảnh mà chia thành nhiều:

$$  
\boxed{\text{Azimuth blocks}}  
$$

Mỗi azimuth block lại được chia theo range thành:

$$  
\boxed{\text{Range blocks}}  
$$

Ví dụ về mặt cấu trúc:

$$  
\begin{array}{c|cccc}  
& RB_0&RB_1&RB_2&RB_3\  
\hline  
AB_0&f_{00}&f_{01}&f_{02}&f_{03}\  
AB_1&f_{10}&f_{11}&f_{12}&f_{13}\  
AB_2&f_{20}&f_{21}&f_{22}&f_{23}  
\end{array}  
$$

Sau đó với mỗi azimuth block:

$$  
{f_{i0},f_{i1},...,f_{in}}  
\rightarrow  
\text{quadratic fit}  
$$

nên ta có:

$$  
f_{DC}^{(0)}(\tau)  
$$

$$  
f_{DC}^{(1)}(\tau)  
$$

$$  
f_{DC}^{(2)}(\tau)  
$$

...

Các range block và azimuth block **có thể overlap** để Doppler thay đổi mượt hơn. Kích thước block và spacing là configurable parameters. Với azimuth block, tài liệu nói block quá nhỏ có thể gây impulse-response truncation; lý tưởng có thể khoảng $10\times$ antenna footprint, nhưng thực tế quá lớn, và tài liệu coi tối thiểu khoảng $2\times$ footprint là phù hợp.

---

# 39. Một chi tiết quan trọng nếu Stripmap là dual-pol

Ngoài Section 5, Section 3.2 còn quy định một điều rất quan trọng đối với Doppler estimation:

**Hai polarisation phải dùng cùng estimated Doppler centroid** để hai ảnh được co-registered chính xác.

Sentinel-1 lấy Doppler estimate từ **co-polarisation channel**, vì cross-pol có khả năng noisy hơn và kém tin cậy hơn.

Ví dụ HH/HV:

$$  
f_{DC}^{HH}=f_{DC}^{HV}=f_{DC,\ estimated\ from\ HH}  
$$

chứ không estimate độc lập HH và HV rồi dùng hai Doppler khác nhau.

---

# 40. Tóm gọn toàn bộ Stripmap DCE bằng các phương trình

Nếu chỉ lấy **những phương trình thực sự nằm trên đường xử lý Stripmap**, chúng là:

$$  
\boxed{  
(5!-!1)\rightarrow(5!-!16)  
}  
$$

Geometry:

$$  
(\mathbf p,\mathbf v,\boldsymbol{\varphi},r)  
\rightarrow  
\mathbf r_0  
\rightarrow  
\tilde f_{DC}(r)  
$$

sau đó **bỏ (5-17)** và thực hiện:

$$  
\boxed{  
(5!-!18)\rightarrow(5!-!20)  
}  
$$

Data-based CDCE:

$$  
s(\tau,\eta)  
\rightarrow  
C_{\mathrm{ACCC}}  
\rightarrow  
\phi_{\mathrm{ACCC}}  
\rightarrow  
f_{DC}^{fine}  
$$

tiếp theo:

$$  
\boxed{  
(5!-!21)\rightarrow(5!-!29)  
}  
$$

Unwrapping:

$$  
f_{DC}^{fine}  
\rightarrow  
\hat f_{DC}^{fine}  
$$

rồi:

$$  
\boxed{  
(5!-!30)  
}  
$$

Ambiguity resolution:

$$  
\tilde f_{DC}^{geometry}  
+  
\hat f_{DC}^{fine}  
\rightarrow  
f_{DC}^{absolute}  
$$

và cuối cùng:

$$  
\boxed{  
f_{DC}(\tau)=c_0+c_1\tau+c_2\tau^2  
}  
$$

Polynomial fitting.

**(5-31) cũng bỏ đối với Stripmap**, vì đó là multi-swath TOPSAR.

Vì vậy, nếu bạn đang tự xây dựng **L0 → L1 Stripmap processor**, tôi sẽ tách module Doppler thành đúng các module phần mềm sau:

$$  
\boxed{  
\begin{aligned}  
&\texttt{GeometryDopplerEstimator}\  
&\downarrow\  
&\texttt{CorrelationDopplerEstimator}\  
&\downarrow\  
&\texttt{DopplerUnwrapper}\  
&\downarrow\  
&\texttt{DopplerAmbiguityResolver}\  
&\downarrow\  
&\texttt{DopplerPolynomialFitter}\  
&\downarrow\  
&\texttt{DopplerQualityEvaluator}  
\end{aligned}  
}  
$$

Đây là cách tách rất sát với chính cấu trúc thuật toán của ESA trong Section 5.

Nếu đi tiếp vào mức **implementation**, phần đáng phân tích nhất tiếp theo là **CDCE (5-18 → 5-20)**: cụ thể ma trận raw I/Q 2D được chia thành azimuth/range blocks thế nào, correlation thực sự chạy trên axis nào, một range block cho ra một $f_{DC}$ như thế nào, rồi từ nhiều range blocks tạo ra polynomial ra sao. Đây chính là đoạn nối trực tiếp với `radar_data` mà bạn đang decode từ Sentinel-1 L0.