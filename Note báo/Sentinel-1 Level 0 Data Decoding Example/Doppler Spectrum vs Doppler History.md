Không giống nhau, nhưng chúng liên hệ trực tiếp với nhau.

**Doppler history** là sự biến thiên của Doppler theo slow-time $\eta$ đối với một scatterer:

$f_D(\eta)$

Ví dụ gần closest approach, nó thường có dạng gần tuyến tính:

$f_D(\eta)\approx f_{DC}+K_a(\eta-\eta_c)$

Nghĩa là bạn đang nhìn “tần số Doppler thay đổi theo thời gian azimuth như thế nào”.

Còn **Doppler spectrum** là biểu diễn trong miền tần số của tín hiệu theo azimuth, thu được bằng FFT theo slow-time:

$S(f_a)=\mathcal{F}_{\eta}{s(\eta)}$

Nó cho biết năng lượng của tín hiệu phân bố trên các tần số Doppler nào.

Có thể hình dung:

```text
Doppler history
f_D
 ^
 |\
 | \
 |  \
 |   \
 +----------> η
```

còn:

```text
Doppler spectrum
|S(f_a)|
 ^
 |        /\
 |      /    \
 |_____/______\______> f_a
          ↑
         f_DC
```

Vì vậy:

- Doppler history: **tần số tức thời thay đổi theo thời gian**
    
- Doppler spectrum: **phân bố năng lượng theo tần số Doppler**
    

[Inference] Với SAR chirp azimuth, Doppler history càng quét qua một dải tần rộng thì Doppler spectrum thường càng có bandwidth lớn tương ứng.

Điểm dễ nhầm là cả hai đều dùng “Doppler”, nhưng một cái mô tả theo **time**, một cái mô tả theo **frequency domain**.

Nếu bạn muốn, tôi có thể giải thích tiếp mối quan hệ chính xác giữa **Doppler history → azimuth chirp → Doppler spectrum → Doppler centroid → Doppler bandwidth** trong cùng một sơ đồ.