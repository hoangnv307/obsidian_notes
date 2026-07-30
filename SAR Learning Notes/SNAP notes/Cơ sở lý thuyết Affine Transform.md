
**Affine Transform (biến đổi affine)** là một phép biến đổi hình học rất quan trọng trong xử lý ảnh, đồ họa máy tính, GIS và đặc biệt là ảnh SAR. Nó cho phép biến đổi vị trí của các điểm trong không gian nhưng **vẫn bảo toàn tính thẳng hàng và tính song song**.

---

# 1. Định nghĩa

Một phép biến đổi affine được mô tả bởi:

$$
\mathbf{x}' = \mathbf{A}\mathbf{x} + \mathbf{t}
$$

Trong đó:

- $\mathbf{x}$: tọa độ điểm gốc
- $\mathbf{x}'$: tọa độ sau biến đổi
- $\mathbf{A}$: ma trận tuyến tính
- $\mathbf{t}$: vector tịnh tiến (translation)

Trong 2D:

$$
\begin{bmatrix}
x'\\
y'
\end{bmatrix}
=
\begin{bmatrix}
a & b\\
c & d
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix}
+
\begin{bmatrix}
t_x\\
t_y
\end{bmatrix}
$$

Viết đầy đủ:

$$
x' = ax + by + t_x
$$

$$
y' = cx + dy + t_y
$$

---

# 2. Ý nghĩa của ma trận A

Ma trận

$$
A=
\begin{bmatrix}
a&b\\
c&d
\end{bmatrix}
$$

có thể kết hợp rất nhiều phép biến đổi.

Ví dụ:

## Translation

Không tác động vào A

chỉ có

$$
t=
\begin{bmatrix}
10\\
5
\end{bmatrix}
$$

thì

```
(3,2)

↓

(13,7)
```

---

## Scale

$$
A=
\begin{bmatrix}
2&0\\
0&3
\end{bmatrix}
$$

Nghĩa là

- x nhân 2
- y nhân 3

---

## Rotation

Góc quay θ

$$
A=
\begin{bmatrix}
\cos\theta &-\sin\theta\\
\sin\theta & \cos\theta
\end{bmatrix}
$$

Ví dụ quay 90°

$$
A=
\begin{bmatrix}
0&-1\\
1&0
\end{bmatrix}
$$

Điểm

```
(1,0)

↓

(0,1)
```

---

## Shear (nghiêng)

Ví dụ

$$
A=
\begin{bmatrix}
1&0.5\\
0&1
\end{bmatrix}
$$

Kết quả

```
□

↓

▱
```

Hình vuông thành hình bình hành.

---

## Reflection

Ví dụ đối xứng qua trục y

$$
A=
\begin{bmatrix}
-1&0\\
0&1
\end{bmatrix}
$$

---

# 3. Những gì được bảo toàn

Affine transform **không bảo toàn khoảng cách hay góc**, nhưng bảo toàn các tính chất sau:

✅ Đường thẳng vẫn là đường thẳng

✅ Các điểm thẳng hàng vẫn thẳng hàng.

✅ Hai đường song song vẫn song song.


✅ Tỉ lệ trên cùng một đoạn thẳng được giữ nguyên.
Nếu
```
AB = 2
BC = 4
```
thì sau affine
```
A'B' : B'C' = 2 : 4
```
vẫn không đổi.

---
# 4. Những gì không được bảo toàn

Ví dụ một hình vuông

```
□□□□
□□□□
□□□□
```

sau affine có thể thành

```
▱▱▱▱
▱▱▱▱
```

Nghĩa là

- góc vuông mất
- chiều dài cạnh thay đổi
- diện tích có thể thay đổi

---

# 5. Dùng tọa độ thuần nhất (Homogeneous Coordinates)

Trong đồ họa máy tính người ta thường viết affine dưới dạng ma trận 3×3.

Thay vì

$$
x' = Ax+t
$$

ta viết

$$
\begin{bmatrix}
x'\\
y'\\
1
\end{bmatrix}
=
\begin{bmatrix}
a&b&t_x\\
c&d&t_y\\
0&0&1
\end{bmatrix}
\begin{bmatrix}
x\\
y\\
1
\end{bmatrix}
$$

Ưu điểm là:

- Translation cũng trở thành phép nhân ma trận.
- Có thể ghép nhiều phép biến đổi bằng cách nhân các ma trận.

Ví dụ

```
Rotate

↓

Scale

↓

Translate
```

chỉ cần

$$
M = T\,S\,R
$$

rồi

$$
x' = Mx
$$

---

# 6. Ví dụ tính toán

Cho điểm

$$
P=(3,2)
$$
Scale x2

Rotate 90°

Translate (5,1)
### Bước 1
Scale
$$
(3,2)
\rightarrow
(6,4)
$$
### Bước 2
Rotate
$$
(6,4)
\rightarrow
(-4,6)
$$
### Bước 3
Translate
$$
(-4,6)
\rightarrow
(1,7)
$$
---

# 7. Ứng dụng trong xử lý ảnh

Affine transform được dùng để:

- Zoom ảnh
- Pan ảnh
- Rotate ảnh
- Deskew (chỉnh nghiêng tài liệu)
- Image registration
- Orthorectification (ở mức cục bộ hoặc xấp xỉ)
- Resampling
- Chuyển đổi hệ tọa độ giữa pixel và không gian bản đồ (khi phép biến đổi có thể coi là affine)

Ví dụ OpenCV:

```cpp
cv::warpAffine(src, dst, M, dstSize);
```

Trong đó

```
M

[a b tx]
[c d ty]
```

chính là ma trận affine 2×3.

---

# 8. Ứng dụng trong GIS và ảnh SAR

Affine transform xuất hiện rất thường xuyên trong GIS và xử lý SAR:

- Chuyển đổi giữa tọa độ pixel `(row, col)` và tọa độ bản đồ `(X, Y)` đối với nhiều raster đã được georeference bằng mô hình affine.
- Pan, zoom, rotate trong các trình xem ảnh như QGIS hoặc SNAP thường được thực hiện thông qua các phép affine trên hệ tọa độ hiển thị.
- Image registration khi hai ảnh chỉ khác nhau bởi dịch chuyển, quay, co giãn hoặc nghiêng.
- Chuyển đổi giữa các hệ tọa độ màn hình (screen coordinates) và tọa độ ảnh.

Lưu ý rằng với **ảnh SAR được geocode hoặc orthorectify**, mối quan hệ giữa tọa độ pixel và tọa độ địa lý trên toàn ảnh thường **không hoàn toàn là affine** do ảnh hưởng của địa hình, mô hình cảm biến và phép chiếu bản đồ. Trong nhiều trường hợp, affine chỉ là một xấp xỉ cục bộ; các phép biến đổi chính xác hơn sẽ dùng mô hình cảm biến, đa thức hoặc lưới biến dạng (warp grid).

---
# 9. Để ảnh SAR dùng affine transform

Để ảnh SAR sử dụng affine transform, với ví dụ cụ thể như ở sản phẩm Sentinel-1 L1 SLC hoặc GRD, ta cần phải tiến hành bước terrain corrected sản phẩm đấy lên phần mềm, 
