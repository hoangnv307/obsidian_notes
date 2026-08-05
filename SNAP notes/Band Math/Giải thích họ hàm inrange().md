Họ `inrange` kiểm tra một vector giá trị có nằm trong các giới hạn tương ứng hay không. Kết quả là `1.0` nếu đạt tất cả điều kiện, ngược lại là `0.0`, nên rất phù hợp để tạo mask.

Thứ tự tham số:

```text
(value₁,...,valueₙ,
 lower₁,...,lowerₙ,
 upper₁,...,upperₙ)
```

Số tham số phải là bội số của 3.

### `inrange(...)`

Kiểm tra trực tiếp:

```text
lowerᵢ <= valueᵢ <= upperᵢ
```

Ví dụ mask mặt nước SAR bằng ngưỡng minh họa:

```text
inrange(VV_dB, VH_dB,
        -25,   -35,
        -15,   -22)
```

Kết quả bằng `1` khi:

```text
-25 <= VV_dB <= -15
-35 <= VH_dB <= -22
```

Có thể dùng kết quả làm mask hoặc trong biểu thức:

```text
inrange(...) ? 1 : 0
```

### `inrange_deriv(...)`

Kiểm tra các thay đổi giữa hai phần tử liên tiếp:

```text
Δvalueᵢ = valueᵢ − valueᵢ₋₁
Δlowerᵢ = lowerᵢ − lowerᵢ₋₁
Δupperᵢ = upperᵢ − upperᵢ₋₁
```

Sau đó kiểm tra:

```text
Δlowerᵢ <= Δvalueᵢ <= Δupperᵢ
```

Ví dụ kiểm tra backscatter ba thời điểm không thay đổi quá ±3 dB mỗi bước:

```text
inrange_deriv(VV_t1, VV_t2, VV_t3,
              0,     -3,    -6,
              0,      3,     6)
```

Ở đây sai phân của giới hạn dưới là `-3`, còn giới hạn trên là `+3`. Kết quả `0` có thể biểu thị một thay đổi bất thường.

Hàm này không kiểm tra giá trị ban đầu, chỉ kiểm tra xu hướng giữa các thời điểm.

### `inrange_integ(...)`

Tính tổng tích lũy rồi kiểm tra:

```text
Σlower <= Σvalue <= Σupper
```

Ví dụ kiểm tra tổng sai lệch backscatter qua ba thời điểm:

```text
inrange_integ(delta_t1, delta_t2, delta_t3,
              -2,       -2,       -2,
               2,        2,        2)
```

Các giới hạn tích lũy lần lượt là:

```text
[-2, 2], [-4, 4], [-6, 6]
```

Hàm này có thể phát hiện sai lệch nhỏ nhưng kéo dài qua nhiều thời điểm. Nên áp dụng cho anomaly hoặc độ thay đổi, không nên cộng trực tiếp giá trị backscatter dB nếu tổng đó không có ý nghĩa vật lý.

Ứng dụng SAR thực tế:

- Tạo mask nước, đất trống hoặc vùng đô thị bằng khoảng `VV/VH`.
- Lọc theo backscatter kết hợp góc tới.
- Kiểm tra độ ổn định của chuỗi ảnh đa thời gian.
- Phát hiện thay đổi bất thường hoặc thay đổi kéo dài.
- Kiểm tra đặc trưng phân cực có nằm trong miền mẫu hay không.

Các ngưỡng phải hiệu chỉnh theo sensor, polarization, incidence angle và dữ liệu đã giảm speckle; các con số trên chỉ minh họa. Implementation: [band_maths_expression.cpp](/home/hoangnv307/code/sar_application/src/engine/core/datamodel/raster/band_maths_expression.cpp:920).