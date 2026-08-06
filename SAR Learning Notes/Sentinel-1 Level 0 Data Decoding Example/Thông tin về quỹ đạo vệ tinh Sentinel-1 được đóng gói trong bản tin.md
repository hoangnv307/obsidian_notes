



Các trường này là **metadata quỹ đạo (Orbit)** và **tư thế (Attitude)** được đóng gói trong mỗi **Sentinel-1 Space Packet Protocol Data Unit (SPDU)**. Chúng được sử dụng để xử lý Level-0 → Level-1, đặc biệt là **geolocation**, **Doppler estimation**, **motion compensation** và **SAR focusing**.

Dưới đây là giải thích chi tiết từng trường.

---

# 1. X-axis position ECEF

Vị trí của vệ tinh theo trục **X** trong hệ tọa độ **ECEF (Earth-Centered Earth-Fixed)**.

Đơn vị:

- mét (m)

Ví dụ

$$
X = 3\,156\,129.91\;m
$$

---

# 2. Y-axis position ECEF

Tọa độ Y trong hệ ECEF.

Đơn vị

- mét

Ví dụ

$$
Y=1\,481\,411.31\;m
$$

---

# 3. Z-axis position ECEF

Tọa độ Z trong hệ ECEF.

Đơn vị

- mét

Ví dụ

$$
Z=6\,248\,898.48\;m
$$

---

## Ba giá trị này tạo thành vector

$$
\mathbf r=
\begin{bmatrix}
X\\
Y\\
Z
\end{bmatrix}
$$

đó là vị trí tức thời của vệ tinh.

---

# Hệ tọa độ ECEF là gì?

ECEF = Earth Centered Earth Fixed

- gốc tại tâm Trái Đất
- quay cùng Trái Đất

```
           Z
           |
           |
           |
           O--------Y
          /
         /
        X
```

Khác với hệ ECI, ECEF luôn quay cùng Trái Đất nên thuận tiện cho việc tính vị trí điểm ảnh trên mặt đất.

---

# 4. X-axis velocity ECEF

Thành phần vận tốc theo trục X.

Đơn vị

m/s

Ví dụ

$$
V_x=6696.66\;m/s
$$

---

# 5. Y-axis velocity ECEF

Vận tốc theo trục Y.

Ví dụ

$$
V_y=347.26\;m/s
$$

---

# 6. Z-axis velocity ECEF

Vận tốc theo trục Z.

Ví dụ

$$
V_z=-3456.91\;m/s
$$

---

## Ba giá trị tạo thành vector vận tốc

$$
\mathbf v=
\begin{bmatrix}
V_x\\
V_y\\
V_z
\end{bmatrix}
$$

---

# Tại sao SAR cần vector vận tốc?

Rất nhiều phép tính cần đến:

- Doppler centroid
- Doppler rate
- Squint angle
- Beam steering
- Slant range
- Motion compensation
- Azimuth focusing

Ví dụ

Doppler phụ thuộc

$$
f_D=
-\frac{2}{\lambda}
\frac{\mathbf v\cdot\mathbf R}
{|\mathbf R|}
$$

---

# 7. POD Solution Data Timestamp

POD = **Precise Orbit Determination**

Đây là thời điểm mà nghiệm quỹ đạo (orbit solution) tương ứng có hiệu lực.

Ví dụ

```
2026-08-06
12:03:15.123456
```

hoặc

```
GPS Time
```

hoặc

```
TAI
```

tuỳ cách lưu.

---

## Dùng để làm gì?

Nội suy quỹ đạo.

Ví dụ

Có dữ liệu orbit mỗi

```
1 s
```

nhưng radar phát

```
3000 pulse/s
```

thì phải nội suy vị trí cho từng pulse.

---

# 8–11. Quaternion Attitude

```
Q0
Q1
Q2
Q3
```

là quaternion biểu diễn tư thế vệ tinh.

Thông thường

$$
q=
(q_0,q_1,q_2,q_3)
$$

---

## Quaternion dùng để làm gì?

Mô tả:

- yaw
- pitch
- roll

mà không gặp hiện tượng gimbal lock như Euler angles.

---

Ví dụ

```
Body Frame
```

↓

Quaternion

↓

ECEF

↓

Radar beam direction

---

Một quaternion luôn chuẩn hóa

$$
q_0^2+
q_1^2+
q_2^2+
q_3^2
=1
$$

---

Ví dụ

```
Q0=0.998
Q1=0.010
Q2=0.050
Q3=-0.020
```

---

## Trong Sentinel-1

Quaternion dùng để tính

- antenna boresight
- Doppler
- pointing angle
- incidence angle
- geolocation

---

# 12–14. Angular Rate

```
Omega-X
Omega-Y
Omega-Z
```

là tốc độ quay quanh ba trục vệ tinh.

Đơn vị

```
rad/s
```

hoặc

```
deg/s
```

tùy định dạng.

---

Ví dụ

```
ωx = 0.001 rad/s
ωy = -0.0002 rad/s
ωz = 0.0008 rad/s
```

---

## Ý nghĩa

Cho biết vệ tinh đang quay nhanh thế nào.

Ví dụ

```
yaw steering
```

khi vệ tinh bám theo hướng bay.

---

Nếu

```
ω = 0
```

thì vệ tinh hoàn toàn ổn định.

---

Nếu

```
ωz ≠0
```

thì vệ tinh đang quay quanh trục Z.

---

# Dùng để làm gì?

Angular rate phục vụ:

- attitude propagation
- motion compensation
- antenna pointing prediction
- autofocus (nếu cần)
- Doppler correction

---

# 15. Attitude Data Timestamp

Thời điểm của bộ dữ liệu attitude.

Ví dụ

```
12:03:15.125000
```

---

## Tại sao khác POD timestamp?

Orbit và attitude thường được cập nhật độc lập.

Ví dụ

Orbit

```
1 Hz
```

Attitude

```
8 Hz
```

hoặc

```
32 Hz
```

nên mỗi bộ dữ liệu có timestamp riêng để nội suy chính xác đến thời điểm phát từng xung radar.

---

# Mối quan hệ giữa các trường

```
                 Orbit
      X,Y,Z,Vx,Vy,Vz
             │
             │
      Orbit Timestamp
             │
             ▼
     Nội suy vị trí vệ tinh
             │
             ▼
        Slant Range
             │
             ▼
        Doppler Model

──────────────────────────────────────

      Quaternion (Q0..Q3)
             │
             ▼
      Hướng antenna

             │
             ▼
     Beam Steering

             │
             ▼
     Incidence Angle

             │
             ▼
        Geolocation

──────────────────────────────────────

 Omega-X,Y,Z
      │
      ▼
Attitude propagation

──────────────────────────────────────

Attitude Timestamp
      │
      ▼
Nội suy attitude tới từng pulse
```

## Trong chuỗi xử lý Sentinel-1 Level-0 → Level-1

Các trường này được sử dụng xuyên suốt nhưng đặc biệt quan trọng ở các bước:

1. **Tính quỹ đạo và vận tốc tức thời** của vệ tinh tại thời điểm từng xung phát (nội suy từ Position/Velocity và POD Timestamp).
2. **Tính hướng chùm sóng (antenna boresight)** từ Quaternion và Angular Rate (nội suy theo Attitude Timestamp).
3. **Ước lượng Doppler centroid và Doppler rate**, vì chúng phụ thuộc đồng thời vào vận tốc vệ tinh và hướng nhìn của anten.
4. **Motion compensation và geolocation**, xác định chính xác vị trí từng mẫu ảnh trên mặt đất.
5. **SAR focusing (RDA, CSA, Omega-K,...)**, trong đó các thông tin quỹ đạo và tư thế là đầu vào của các mô hình hình học và Doppler. Chúng không trực tiếp tham gia phép nén xung, nhưng quyết định độ chính xác của việc căn chỉnh hình học và hội tụ ảnh.