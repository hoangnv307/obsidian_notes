Ba hàm này dùng để đo mức khác nhau giữa hai vector. Danh sách tham số được chia đôi:

```text
(a₁, a₂, ..., aₙ, b₁, b₂, ..., bₙ)
```

Số tham số phải chẵn và tối thiểu là 2; nếu lẻ, chương trình trả `NaN`.

- `distance(...)`: khoảng cách Euclid giữa hai vector.

  ```text
  √Σ(bᵢ - aᵢ)²
  ```

  Ví dụ:

  ```text
  distance(1, 2, 3, 2, 4, 6) = √14
  ```

- `distance_deriv(...)`: tính sai phân bậc nhất của từng vector rồi đo khoảng cách giữa hai dãy sai phân.

  ```text
  √Σ[(bᵢ-bᵢ₋₁) - (aᵢ-aᵢ₋₁)]²
  ```

  Hàm này so sánh hình dạng hoặc tốc độ thay đổi, giảm ảnh hưởng của độ lệch hằng. Nếu `B = A + một hằng số`, kết quả bằng `0`.

  ```text
  distance_deriv(1, 2, 3, 2, 4, 6) = √2
  ```

- `distance_integ(...)`: tính tổng tích lũy của từng vector rồi đo khoảng cách giữa hai dãy tổng.

  ```text
  Aᵢ = a₁ + ... + aᵢ
  Bᵢ = b₁ + ... + bᵢ
  kết quả = √Σ(Bᵢ - Aᵢ)²
  ```

  Hàm này làm nổi bật sai lệch tích lũy:

  ```text
  distance_integ(1, 2, 3, 2, 4, 6) = √46
  ```

`deriv` ở đây là sai phân rời rạc, còn `integ` chỉ là tổng tích lũy, không sử dụng khoảng cách bước lấy mẫu. Công thức hiện tại giống SNAP khi truyền đúng số lượng tham số chẵn. Nguồn: [band_maths_expression.cpp](/home/hoangnv307/code/sar_application/src/engine/core/datamodel/raster/band_maths_expression.cpp:891).

# Ứng dụng
Có. Với ảnh SAR, ứng dụng dễ hiểu nhất là phát hiện thay đổi đa thời gian.

Giả sử đã đồng đăng ký ảnh trước và sau sự kiện, có các band:

```text
VV_before, VH_before
VV_after,  VH_after
```

Biểu thức:

```text
distance(VV_before, VH_before,
         VV_after,  VH_after)
```

tương đương:

```text
√[(VV_after−VV_before)² + (VH_after−VH_before)²]
```

Ví dụ dữ liệu dB:

```text
Trước: VV = -10, VH = -17
Sau:   VV =  -5, VH = -12

distance = √(5² + 5²) ≈ 7.07
```

Khoảng cách lớn cho biết đặc trưng tán xạ đã thay đổi mạnh. Có thể tạo mask:

```text
distance(VV_before, VH_before,
         VV_after,  VH_after) > 4.0
```

Ứng dụng gồm:

- Phát hiện ngập lụt, phá rừng, cháy rừng hoặc xây dựng mới.
- So sánh đặc trưng phân cực `VV/VH`, `HH/HV` với mẫu của một loại bề mặt.
- Tìm pixel bất thường trong chuỗi ảnh SAR nhiều thời điểm.
- `distance_deriv`: so sánh xu hướng thay đổi theo thời gian, ít nhạy với độ lệch nền cố định.
- `distance_integ`: nhấn mạnh thay đổi nhỏ nhưng kéo dài qua nhiều thời điểm; ít phổ biến hơn.

Trước khi dùng nên:

- Hiệu chỉnh radiometric về `Sigma0` hoặc `Gamma0`.
- Đồng đăng ký chính xác các ảnh.
- Dùng cùng miền giá trị, không trộn band tuyến tính với band dB.
- Giảm speckle hoặc tính trên dữ liệu đã làm trơn; khoảng cách trực tiếp từng pixel thường khá nhiễu.
- Không dùng các hàm này thay cho coherence hoặc phase trong bài toán giao thoa SAR. Chúng chỉ so sánh các giá trị band dạng số.