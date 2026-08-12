Trong pipeline Sentinel-1 mà bạn đang đọc, **Correlation Doppler Centroid Estimator (CDCE)** có một nhiệm vụ rất cụ thể:

> **Ước lượng phần Fine Doppler Centroid trực tiếp từ tín hiệu echo SAR**, bằng cách đo **độ thay đổi pha trung bình giữa các mẫu azimuth liên tiếp**.

Nói cách khác, CDCE trả lời câu hỏi:

> **“Từ chính dữ liệu radar thu được, tâm phổ Doppler thực tế đang nằm ở đâu trong khoảng baseband ([-PRF/2,+PRF/2])?”**

### Vì sao correlation lại cho ta Doppler?

Xét một range bin cố định. Theo azimuth, radar có chuỗi complex samples:

$(s_0,s_1,s_2,s_3,\ldots)$

Mỗi sample đến từ một pulse radar liên tiếp, cách nhau một PRI:

$(\Delta t=PRI=1/PRF).$

Nếu có Doppler (f_D), pha của tín hiệu thay đổi theo azimuth. Đơn giản hóa:

$(s_n=A_ne^{j\phi_n})$

và giữa hai pulse:

$(\Delta\phi=\phi_{n+1}-\phi_n).$

CDCE tạo **lag-one correlation**:

$(s_n^*s_{n+1}).$

Thay biểu thức complex vào:

$(s_n^*s_{n+1}=A_nA_{n+1}e^{j(\phi_{n+1}-\phi_n)}).$

Vì vậy:

$(\angle(s_n^*s_{n+1})=\phi_{n+1}-\phi_n=\Delta\phi).$

Đây chính là điểm mấu chốt của CDCE:

**correlation giữa hai pulse liên tiếp biến thông tin Doppler thành một phase difference có thể đo được.**

Sau khi average trên nhiều samples, tài liệu lấy phase của ACCC rồi chuyển thành Doppler centroid theo:

$(f_c=-\frac{PRF}{2\pi}\phi_{ACCC}).$

### Tại sao phải "average correlation"?

Nếu chỉ lấy:

$(s_0^*s_1)$

thì kết quả phụ thuộc rất mạnh vào một cặp samples cụ thể.

Thay vào đó CDCE thực hiện trên nhiều pulse:

```text
Azimuth →

s0 ─ s1 ─ s2 ─ s3 ─ s4 ─ ... ─ sN
│     │
└─────┘  s0* s1
      │     │
      └─────┘  s1* s2
            │     │
            └─────┘  s2* s3
                  ...
```

rồi cộng/average các correlation này. Vì vậy nó được gọi là **Average Cross Correlation Coefficient – ACCC**.

Sau đó Sentinel-1 còn average ACCC theo **range block**. Kết quả là:

```text
                 RANGE →

          Block 1    Block 2    Block 3    Block 4
             │          │          │          │
            ACCC₁      ACCC₂      ACCC₃      ACCC₄
             │          │          │          │
           angle      angle      angle      angle
             │          │          │          │
            fDC₁       fDC₂       fDC₃       fDC₄
```

Tức là ta có **Fine DC theo range**, chứ không nhất thiết một giá trị DC duy nhất cho toàn bộ swath.

### Vậy tại sao không chỉ dùng DC tính từ orbit?

Đây là ý nghĩa lớn nhất của CDCE.

Sentinel-1 trước đó có thể tính Doppler từ **geometry** dựa trên orbit, attitude, slant range... Nhưng tài liệu nói rằng geometry estimate thường **không đủ chính xác** để dùng trực tiếp cho focusing.

Do đó có hai nguồn thông tin:

**Geometry DC**

$$
(Orbit + Attitude + Geometry \rightarrow DC_{\text{geometry}})
$$

cho ta dự đoán Doppler dựa trên mô hình hình học.

Còn:

**Fine DC – CDCE**

$$(Raw\ echo \rightarrow phase\ correlation \rightarrow DC_{\text{fine}})$$

cho ta Doppler **đo trực tiếp từ dữ liệu radar**.

Hai phần sau đó được kết hợp để giải quyết Doppler ambiguity và tạo **Absolute Doppler Centroid**.

### Tại sao chỉ gọi là “Fine DC”?

Bởi vì dữ liệu radar được sampling theo azimuth với sampling frequency bằng PRF. Do đó CDCE chỉ quan sát được Doppler trong baseband:

$$
(-PRF/2 \le f_{\text{fine}} < +PRF/2).
$$

Ví dụ Doppler thật có thể là:

$$
(f_{DC}=2430\text{ Hz})
$$

với:

$$
(PRF=2000\text{ Hz}).
$$

CDCE không thể phân biệt trực tiếp 2430 Hz với:

$$
(2430-2000=430\text{ Hz}).
$$

Nó chỉ nhìn thấy:

**Fine DC = 430 Hz.**

Geometry estimate giúp xác định Doppler ambiguity number (M), rồi:

$$
f_{DC}^{absolute}=f_{DC}^{fine}+M\cdot PRF.
$$

Đây là lý do **CDCE một mình chưa đủ để có Absolute DC**.

### Liên hệ với raw data correction mà bạn vừa hỏi

Đây cũng là lý do ESA yêu cầu:

**BAQ decode → Raw data correction → PG/drift compensation → Range compression → CDCE**

trước khi CDCE chạy.

[Inference] Vì CDCE đo **phase difference**, nếu instrument tạo ra phase drift:

$$
\Delta\phi_{measured}=\Delta\phi_{Doppler}+\Delta\phi_{instrument},
$$

thì CDCE có thể coi phase drift của instrument là Doppler. Do đó drift compensation đặc biệt quan trọng trước phase-based estimator.

---

Nếu rút gọn toàn bộ CDCE thành **một câu để nhớ**:

> **CDCE đo tốc độ quay pha trung bình của echo SAR từ pulse này sang pulse kế tiếp; tốc độ quay pha đó được chuyển thành Fine Doppler Centroid.**

Và về mặt xử lý tín hiệu:

**Doppler → phase progression → lag-1 correlation → ACCC phase → Fine DC.**

Đây là lý do thuật toán correlation xuất hiện ở đây: **correlation không phải mục tiêu cuối cùng; nó là công cụ để lấy phase derivative một cách thống kê và từ đó suy ra Doppler centroid.**