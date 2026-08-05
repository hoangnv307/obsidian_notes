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