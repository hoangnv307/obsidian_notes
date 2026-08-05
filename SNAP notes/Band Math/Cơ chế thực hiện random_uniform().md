
```
random_uniform() hiện dùng cơ chế:

  1. Mỗi thread tạo riêng một bộ sinh giả ngẫu nhiên std::mt19937_64.
  2. Bộ sinh được khởi tạo một lần bằng std::random_device.
  3. std::uniform_real_distribution<double>(0.0, 1.0) chuyển kết quả thành phân phối đều trong khoảng [0, 1).
  4. Mỗi lần biểu thức được tính, hàm sinh ra giá trị tiếp theo trong chuỗi.

  thread_local std::mt19937_64 generator(std::random_device{}());
  thread_local std::uniform_real_distribution<double> distribution(0.0, 1.0);
  return distribution(generator);

  Hàm được đánh dấu có side effect nên ExprTk không tối ưu nó thành một hằng số; mỗi pixel sẽ nhận một lần sinh mới. Nếu gọi hai lần
  trong cùng biểu thức, ví dụ random_uniform() + random_uniform(), hai số độc lập kế tiếp sẽ được lấy.

  Một số đặc điểm:

  - Không có seed cố định hoặc seed do người dùng cấu hình, nên kết quả không tái lập giữa các lần chạy.
  - Mỗi thread có chuỗi riêng, tránh tranh chấp dữ liệu.
  - Đây là PRNG chất lượng tốt cho mô phỏng/xử lý ảnh, không phù hợp cho mật mã.
  - Quá trình validation cũng đánh giá biểu thức một lần, nên sẽ sinh và bỏ qua một giá trị trước khi bắt đầu tính raster.

  Nguồn: src/engine/core/datamodel/raster/band_maths_expression.cpp:811, đăng ký với ExprTk tại src/engine/core/datamodel/raster/
  band_maths_expression.cpp:1021.
```

## ChatGPT giải thích 
 
`std::mt19937_64` là tên của một **bộ sinh số giả ngẫu nhiên (pseudo-random number generator - PRNG)** trong thư viện chuẩn C++ `<random>`. Nó là biến thể 64-bit của thuật toán **Mersenne Twister**.

Tên `mt19937_64` được ghép từ:

| Thành phần | Ý nghĩa |
|---|---|
| `mt` | **Mersenne Twister** — tên thuật toán sinh số giả ngẫu nhiên |
| `19937` | Chu kỳ (period) của bộ sinh: $2^{19937}-1$ |
| `_64` | Mỗi lần sinh ra một số nguyên 64-bit |

---

## 1. Mersenne Twister là gì?

Một PRNG không tạo ra số ngẫu nhiên thật, mà tạo ra một chuỗi số **xác định** dựa trên một trạng thái nội bộ (state).

Ví dụ:

```
seed = 12345

       +----------------+
state →| Mersenne       |
       | Twister        |
       +----------------+
              |
              v
        154823746293
        987123456789
        238746192837
        ...
```

Nếu dùng cùng seed thì **luôn tạo ra cùng một chuỗi**:

```cpp
std::mt19937_64 gen(12345);

std::cout << gen() << "\n";
std::cout << gen() << "\n";
```

sẽ luôn cho cùng kết quả.

---

## 2. Vì sao gọi là "Mersenne Twister"?

Tên này xuất phát từ tính chất toán học:

Nó sử dụng một trạng thái có kích thước:

$$
19937 \text{ bits}
$$

và chu kỳ:

$$
T = 2^{19937}-1
$$

Điều này nghĩa là:

- Sau khoảng $2^{19937}-1$ lần sinh số thì chuỗi mới lặp lại.
- Đây là một con số cực kỳ lớn:

$$
2^{19937} \approx 10^{6001}
$$

Nên trong thực tế gần như không bao giờ gặp chu kỳ lặp.

---

## 3. `mt19937` khác `mt19937_64` như thế nào?

C++ có hai phiên bản:

### `std::mt19937`

Sinh số nguyên 32-bit:

```cpp
std::mt19937 gen;
uint32_t x = gen();
```

Giá trị:

$$
0 \le x < 2^{32}
$$

---

### `std::mt19937_64`

Sinh số nguyên 64-bit:

```cpp
std::mt19937_64 gen;
uint64_t x = gen();
```

Giá trị:

$$
0 \le x < 2^{64}
$$

Ví dụ:

```
mt19937:
10110110 01101001 00101101 11001010

mt19937_64:
10110110 01101001 00101101 11001010
11010111 00101010 11101001 01011100
```

`mt19937_64` cho không gian số lớn hơn.

---

## 4. Trong `random_uniform()` nó đóng vai trò gì?

Đoạn bạn đưa:

> Mỗi thread tạo riêng một bộ sinh giả ngẫu nhiên std::mt19937_64

có nghĩa là:

```
Thread 1
   |
   +--> mt19937_64 state A
            |
            +--> 873462983746
            +--> 129837461923
            +--> ...

Thread 2
   |
   +--> mt19937_64 state B
            |
            +--> 928374612983
            +--> ...
```

Mỗi thread có một chuỗi riêng, tránh việc nhiều thread cùng truy cập một generator.

---

Sau đó:

```cpp
std::uniform_real_distribution<double>(0.0,1.0)
```

lấy số nguyên 64-bit:

Ví dụ:

```
mt19937_64
        |
        v
1234567890123456789
        |
        v
uniform_real_distribution
        |
        v
0.669432781
```

Nó biến số nguyên thành số thực:

$$
x \in [0,2^{64}-1]
$$

thành:

$$
y=\frac{x}{2^{64}}
$$

nên:

$$
0 \le y < 1
$$

---

## 5. Vì sao dùng `mt19937_64` thay vì `rand()`?

So sánh:

| | `rand()` | `mt19937_64` |
|-|-|-|
| Chuẩn C++ | C cũ | C++11 |
| Kích thước trạng thái | nhỏ | 19937 bits |
| Chu kỳ | thường $2^{31}$ | $2^{19937}-1$ |
| Chất lượng thống kê | thấp hơn | rất tốt |
| Thread-safe | không | cần quản lý riêng |
| Tốc độ | nhanh | nhanh |
| Dùng Monte Carlo | không khuyến nghị | phù hợp |

---

## 6. Nhưng `mt19937_64` có phải random thật không?

Không.

Nó là **deterministic PRNG**.

Ví dụ:

```cpp
std::mt19937_64 gen(42);

for(int i=0;i<5;i++)
    std::cout << gen() << "\n";
```

luôn cho:

```
13930160852258120406
11788048577503494824
13874630024467741450
2513787319205155662
...
```

Nếu biết seed `42`, ta dự đoán được toàn bộ chuỗi.

---

## 7. Tại sao dùng `std::random_device` để seed?

Trong đoạn của bạn:

> Bộ sinh được khởi tạo một lần bằng std::random_device

Ví dụ:

```cpp
std::random_device rd;

std::mt19937_64 gen(rd());
```

`random_device` cố lấy entropy từ hệ thống:

- Linux: `/dev/urandom`
- Windows: Crypto API

Ví dụ:

```
random_device
       |
       v
seed = 938472938472938

       |
       v

mt19937_64(seed)
       |
       v

0.12342
0.98374
0.51239
```

Sau khi seed xong, `mt19937_64` tự chạy rất nhanh mà không cần gọi lại `random_device`.

---

## 8. Liên hệ với SNAP Band Maths

Trong trường hợp `random_uniform()` của SNAP:

```
Expression evaluation
        |
        v
random_uniform()
        |
        v
thread-local mt19937_64
        |
        v
uniform_real_distribution
        |
        v
pixel value
```

Ví dụ:

```
BandMath:
Amplitude_VV * random_uniform()
```

mỗi pixel sẽ nhận một giá trị:

```
Pixel(0,0): 0.38472
Pixel(0,1): 0.92831
Pixel(0,2): 0.12391
...
```

Nhưng nếu chạy lại lần sau:

```
Pixel(0,0): 0.77182
Pixel(0,1): 0.09122
...
```

vì seed mới được tạo từ `random_device`.

---

Tóm lại:

> **`std::mt19937_64` là một bộ sinh số giả ngẫu nhiên Mersenne Twister 64-bit, có trạng thái 19937 bit và chu kỳ $2^{19937}-1$. Trong `random_uniform()` nó đóng vai trò tạo ra chuỗi số nguyên giả ngẫu nhiên tốc độ cao, sau đó `uniform_real_distribution` chuyển chúng thành số thực trong khoảng $[0,1)$.**