# Tóm tắt

```text
Chunk 13
num_quads = 9975
```

có nghĩa là:

- mỗi echo line trong chunk này có **9975 mẫu I/Q** theo phương range.
    

Tức là:

```text
Pulse k

Range
-------------------------------------------------->

I0,Q0
I1,Q1
I2,Q2
...
I9974,Q9974
```
---
# Đầy đủ
## Tại sao gọi là "quad"?

Trong Sentinel-1, một mẫu radar bao gồm:

- I (In-phase)
    
- Q (Quadrature)
    

Sau khi BAQ/FDBAQ giải nén, mỗi mẫu sẽ có:

```text
(I, Q)
```

Đây là **một complex sample**.

Trong tài liệu và code của ESA, người ta thường gọi đơn vị này là một **quad**.

Vì vậy:

```text
1 quad
    ↓
1 complex sample
    ↓
(I,Q)
```

---

## Trong ví dụ của bạn

```text
Chunk 13
num_quads = 9975
```

có nghĩa là:

- mỗi echo line trong chunk này có **9975 mẫu I/Q** theo phương range.
    

Tức là:

```text
Pulse k

Range
-------------------------------------------------->

I0,Q0
I1,Q1
I2,Q2
...
I9974,Q9974
```

---

Chunk tiếp theo

```text
Chunk 14

num_quads = 9993
```

lại có

```text
9993
```

mẫu.

---

## Tại sao Chunk 13 và Chunk 14 khác nhau?

Đây là điểm rất thú vị.

Ta thấy:

```text
Chunk 13

SWST = 83.336 µs
SWL  = 426.698 µs
num_quads = 9975
```

và

```text
Chunk 14

SWST = 81.604 µs
SWL  = 427.444 µs
num_quads = 9993
```

Ta nhận thấy:

- SWST giảm khoảng **1.73 µs**
    
- SWL tăng khoảng **0.75 µs**
    

=> cửa sổ lấy mẫu bắt đầu sớm hơn và kéo dài hơn.

Điều đó đồng nghĩa:

- thu được nhiều echo hơn,
    
- nên số sample cũng tăng.
    

Nếu tần số lấy mẫu gần:

$$  
f_s \approx 23.5\ \text{MHz}  
$$

thì

$$  
9993-9975=18  
$$

mẫu tương ứng khoảng

$$  
\frac{18}{23.5,\text{MHz}}  
\approx0.77,\mu s  
$$

rất gần với mức tăng của SWL (~0.746 µs).

Điều này cho thấy **num_quads thay đổi chủ yếu do độ dài cửa sổ lấy mẫu (SWL)**.

---

## Quan hệ giữa SWL và num_quads

Về nguyên tắc:

$$  
\boxed{  
\text{num_quads}  
\approx  
SWL \times f_s  
}  
$$

trong đó

- SWL là thời gian lấy mẫu,
    
- $f_s$ là tần số lấy mẫu ADC.
    

Do đó:

- SWL tăng → num_quads tăng.
    
- SWL giảm → num_quads giảm.
    

---

## Trong ma trận SAR

Nếu một acquisition chunk gồm 5000 pulse:

```text
Pulse 0     9975 quads
Pulse 1     9975 quads
...
Pulse 4999  9975 quads
```

thì ma trận I/Q sẽ có kích thước:

```text
5000 × 9975
```

Trong đó:

- **5000** là số echo line (azimuth),
    
- **9975** là `num_quads` (range samples).
    

---

### Kết luận

Trong ngữ cảnh Sentinel-1 Level-0:

- **`num_quads` là số lượng mẫu I/Q phức (complex samples) theo phương range của mỗi echo line trong acquisition chunk.**
    
- Nó quyết định **chiều rộng của ma trận dữ liệu SAR** (số cột theo range).
    
- Sự thay đổi từ **9975 → 9993** trong ví dụ của bạn phù hợp với việc **SWL tăng**, dẫn đến cửa sổ lấy mẫu dài hơn và thu được nhiều mẫu hơn.