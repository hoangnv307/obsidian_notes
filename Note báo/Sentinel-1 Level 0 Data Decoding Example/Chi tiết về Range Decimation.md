`Range Decimation (RGDEC)` là một tham số rất quan trọng của Sentinel-1 nhưng thường bị hiểu nhầm. **RGDEC không phải là hệ số decimation trực tiếp**, mà là **một mã (code)** dùng để chỉ cấu hình decimation của bộ thu.

Do đó:

> **RGDEC = 9** **không có nghĩa là** "giảm lấy mẫu đi 9 lần".

---

## Trước hết, Range Decimation là gì?

Sau khi echo được trộn xuống baseband và lọc, ADC sẽ lấy mẫu ở một tần số khá cao.

Ví dụ:

```text
ADC
245.76 MHz
```

Nhưng không phải lúc nào radar cũng cần băng thông lớn như vậy.

Ví dụ:

|Chế độ|Chirp Bandwidth|
|---|--:|
|Spotlight|100 MHz|
|Stripmap|50 MHz|
|EW|15 MHz|

Nếu chirp chỉ rộng 15 MHz thì không cần giữ tốc độ lấy mẫu 245.76 MHz.

Vì vậy phần cứng sẽ:

1. lọc số (digital filter),
    
2. giảm tốc độ lấy mẫu (decimation).
    

Đó chính là **Range Decimation**.

---

## Minh họa

Ban đầu

```text
245.76 MHz

x x x x x x x x x x x x x
```

Sau decimation

```text
61.44 MHz

x . . . x . . . x . . .
```

Tất nhiên trong thực tế, trước khi bỏ mẫu luôn có **anti-aliasing filter**.

---

# RGDEC = 9 nghĩa là gì?

Đối với Sentinel-1, trường:

```text
Range Decimation
```

không lưu:

```text
61.44 MHz
```

mà lưu

```text
9
```

Trong tài liệu Sentinel-1, giá trị **9 là một mã cấu hình**.

Ví dụ (minh họa):

|RGDEC|Fs sau decimation|
|---|---|
|0|...|
|4|...|
|7|...|
|**9**|...|

Nghĩa là decoder phải tra bảng để biết:

```text
RGDEC = 9
```

tương ứng với tần số lấy mẫu nào.

---

## Tại sao ESA không lưu luôn Fs?

Bởi vì trên phần cứng chỉ tồn tại một số cấu hình hợp lệ.

Ví dụ firmware FPGA có:

```text
Mode 0
Mode 1
Mode 2
...
Mode 9
```

Chỉ cần truyền số:

```text
9
```

thay vì truyền:

```text
61 440 000
```

là đủ.

---

# RGDEC ảnh hưởng điều gì?

Nó quyết định:

## 1. Tần số lấy mẫu sau decimation

$$  
f_s  
$$

---

## 2. Khoảng cách giữa hai sample

# $$  
\Delta R

\frac{c}{2f_s}  
$$

Nếu

```text
Fs lớn
```

↓

sample dày

↓

range resolution tốt.

---

Nếu

```text
Fs nhỏ
```

↓

sample thưa

↓

ít dữ liệu hơn.

---

## 3. num_quads

Ta vừa nói:

$$  
num_quads  
\approx  
SWL\times f_s  
$$

Nếu

```text
RGDEC tăng
```

↓

Fs giảm

↓

num_quads giảm.

---

# Trong Sentinel-1 decoder

Thông thường bạn sẽ thấy

```python
metadata["range_decimation"]
```

trả về

```text
9
```

Đây **chưa phải** tần số lấy mẫu.

Decoder phải dùng bảng nội bộ để suy ra

```text
Fs
```

rồi mới tính

- slant range
    
- range spacing
    
- sample time
    

---

# Điều mình muốn lưu ý

[Unverified] Mình **không thể khẳng định** rằng:

> **RGDEC = 9 tương ứng với chính xác tần số lấy mẫu bao nhiêu MHz**

vì điều này phụ thuộc vào **bảng ánh xạ (lookup table)** được ESA định nghĩa trong tài liệu Instrument Processing Facility (IPF) hoặc Instrument Source Packet Protocol.

Đó **không phải** là phép tính đơn giản từ số 9.

---

## Nếu bạn đang nghiên cứu Sentinel-1 Level-0

Mình có thể giúp bạn tra và giải thích **bảng ánh xạ đầy đủ**:

```text
RGDEC code
        ↓
Digital filter
        ↓
Output sampling frequency
        ↓
Range sample spacing
        ↓
Ground range spacing
```

bao gồm **ý nghĩa của tất cả các giá trị RGDEC (0–15 hoặc các mã được Sentinel-1 sử dụng)** theo đúng tài liệu kỹ thuật của ESA, thay vì chỉ giải thích khái niệm chung. Đây cũng là phần mà hầu hết các tài liệu giới thiệu về Sentinel-1 không trình bày chi tiết.