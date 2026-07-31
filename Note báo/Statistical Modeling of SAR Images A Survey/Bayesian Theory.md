

**Bayesian theory (Lý thuyết Bayes)** là một khuôn khổ của xác suất và thống kê dùng để **cập nhật mức độ tin tưởng (belief)** vào một giả thuyết khi có thêm dữ liệu mới.

Ý tưởng cốt lõi rất đơn giản:

> **Niềm tin ban đầu + Dữ liệu mới → Niềm tin được cập nhật**

Thay vì chỉ nhìn vào dữ liệu hiện tại, Bayesian kết hợp:
- kiến thức đã có trước đó (prior),
- và bằng chứng mới (evidence),
để đưa ra kết luận.

---

# 1. Định lý Bayes

Toàn bộ lý thuyết dựa trên công thức:

$$
P(H|D)=\frac{P(D|H)P(H)}{P(D)}
$$

Trong đó:

- $H$ : Hypothesis (giả thuyết)
- $D$ : Data (dữ liệu quan sát)

Các thành phần:

### Prior

$$
P(H)
$$

Xác suất ta tin vào giả thuyết **trước khi có dữ liệu**.

Ví dụ:

> Trước khi xét ảnh SAR,
> ta tin xác suất một pixel là tàu chỉ khoảng 1%.

Đó là prior.

---

### Likelihood

$$
P(D|H)
$$

Nếu giả thuyết đúng,
khả năng sinh ra dữ liệu quan sát là bao nhiêu.

Ví dụ:

Nếu đúng là tàu,

thì xác suất đo được RCS lớn như hiện tại là 95%.

---

### Evidence

$$
P(D)
$$

Xác suất xuất hiện dữ liệu đó bất kể giả thuyết nào.

Đây chỉ đóng vai trò chuẩn hóa để tổng xác suất bằng 1.

---

### Posterior

$$
P(H|D)
$$

Đây là thứ ta cần.

Là xác suất của giả thuyết sau khi đã quan sát dữ liệu.

---

# 2. Ví dụ đơn giản

Giả sử có bệnh A.

Trong dân số:

- 1% người mắc bệnh

Prior:

$$
P(\text{Disease})=0.01
$$

Máy xét nghiệm:

Nếu thật sự mắc bệnh

- Test dương tính 99%

$$
P(+|\text{Disease})=0.99
$$

Nếu không mắc bệnh

- vẫn có 5% dương tính giả

$$
P(+|\text{Healthy})=0.05
$$

Giả sử một người test dương tính.

Người đó có thật sự mắc bệnh không?

Áp dụng Bayes.

Evidence:

$$
P(+)=0.99\times0.01+0.05\times0.99
$$

$$
=0.0594
$$

Posterior:

$$
P(\text{Disease}|+)
=\frac{0.99\times0.01}{0.0594}
\approx16.7\%
$$

Kết quả khá bất ngờ:

Mặc dù test chính xác 99%,

nhưng người đó chỉ có khoảng **16.7%** khả năng mắc bệnh vì bệnh rất hiếm trong dân số (prior thấp).

Đây là một ví dụ kinh điển cho thấy tầm quan trọng của prior.

---

# 3. Ví dụ với SAR

Giả sử đang phát hiện tàu.

Một pixel có biên độ rất lớn.

Nếu chỉ nhìn biên độ:

> Có vẻ là tàu.

Nhưng Bayesian sẽ hỏi:

"Tàu xuất hiện ở khu vực này có phổ biến không?"

Ví dụ:

Prior:

Trong vùng ảnh

- 99.9% pixel là biển
- 0.1% là tàu

Sau đó thêm dữ liệu:

- RCS
- kích thước
- texture
- Doppler
- AIS

Mỗi dữ liệu sẽ cập nhật posterior.

Ví dụ:

Ban đầu:

```
Ship
1%
```

Sau khi thấy RCS lớn:

```
Ship
40%
```

Sau khi thấy Doppler phù hợp:

```
Ship
75%
```

Sau khi thấy AIS xác nhận:

```
Ship
99.8%
```

Đó chính là Bayesian updating.

---

# 4. Vì sao Bayesian mạnh?

Giả sử robot theo dõi mục tiêu.

Lần đầu radar đo:

```
1000 m
```

Sai số lớn.

Lần sau:

```
1002 m
```

Bayesian không bỏ toàn bộ thông tin cũ.

Nó sẽ kết hợp:

- kết quả cũ
- phép đo mới

để cho kết quả tốt hơn.

Chính vì vậy:

- Kalman Filter
- Particle Filter
- SLAM
- Sensor Fusion

đều dựa trên tư tưởng Bayesian.

---

# 5. Bayesian và Frequentist khác nhau

| Frequentist | Bayesian |
|------------|-----------|
| Xác suất là tần suất xảy ra khi lặp lại thí nghiệm nhiều lần | Xác suất biểu diễn mức độ tin tưởng vào một giả thuyết |
| Tham số mô hình là giá trị cố định nhưng chưa biết | Tham số mô hình được xem là biến ngẫu nhiên có phân bố xác suất |
| Không dùng prior | Có prior |
| Chỉ dựa trên dữ liệu hiện tại | Kết hợp kiến thức trước đó và dữ liệu mới |
| Kết quả thường là một giá trị ước lượng hoặc khoảng tin cậy | Kết quả là một phân bố xác suất (posterior) |

Ví dụ:

Frequentist:

> "Tốc độ mục tiêu được ước lượng là 153 m/s."

Bayesian:

> "Tốc độ mục tiêu có phân bố xác suất; giá trị có khả năng cao nhất là 153 m/s, nhưng vẫn tồn tại xác suất cho các giá trị lân cận."

---

# 6. Ứng dụng trong xử lý tín hiệu và SAR

Với các chủ đề bạn đang quan tâm về radar và SAR, Bayesian xuất hiện rất nhiều:

| Bài toán | Vai trò của Bayesian |
|----------|----------------------|
| Kalman Filter | Cập nhật trạng thái mục tiêu từ các phép đo nhiễu |
| Particle Filter | Theo dõi mục tiêu phi tuyến, phi Gaussian |
| Doppler estimation | Ước lượng tần số Doppler kết hợp thông tin tiên nghiệm |
| SAR Autofocus | Ước lượng sai số pha dựa trên mô hình xác suất |
| Target Detection (CFAR mở rộng) | Kết hợp xác suất xuất hiện mục tiêu với dữ liệu đo |
| Image Segmentation | Phân loại pixel bằng xác suất hậu nghiệm |
| Ship Detection | Kết hợp cường độ, kết cấu, Doppler, AIS và mô hình nền |
| Sensor Fusion | Hợp nhất dữ liệu từ radar, GPS, INS, IMU và các cảm biến khác |
| AI/Deep Learning | Suy luận Bayesian để ước lượng độ không chắc chắn của mô hình |

---

# 7. Tóm tắt trực quan

Có thể hình dung quy trình Bayesian như sau:

```
Kiến thức ban đầu
      │
      ▼
    Prior
      │
      │ + Dữ liệu mới
      ▼
Likelihood
      │
      ▼
Định lý Bayes
      │
      ▼
 Posterior
      │
      ├── Có dữ liệu mới
      ▼
Posterior trở thành Prior mới
      │
      ▼
Lặp lại
```

Đây là điểm đặc trưng của tư duy Bayesian: **mỗi khi có thêm quan sát, kết quả hiện tại (posterior) sẽ trở thành kiến thức ban đầu (prior) cho lần cập nhật tiếp theo**, giúp mô hình cải thiện dần theo thời gian. Với các bộ lọc như Kalman hay Particle Filter, quá trình này diễn ra ở mỗi chu kỳ đo.