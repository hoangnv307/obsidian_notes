
Trong xử lý ảnh SAR, **Sigma nought (σ⁰), Beta nought (β⁰), Gamma nought (γ⁰)** là ba dạng **chuẩn hóa cường độ phản xạ radar (radiometric calibration)** thường gặp. Chúng đều biểu diễn "độ mạnh tín hiệu radar phản xạ từ mặt đất", nhưng **khác nhau ở cách chuẩn hóa theo hình học chiếu xạ của radar**.

Đây là một chủ đề rất quan trọng khi làm việc với Sentinel-1, TerraSAR-X, COSMO-SkyMed, v.v.
## 0. Tóm tắt nhanh
![[Pasted image 20260728170947.png]]

---

## 1. Vì sao cần chuẩn hóa tín hiệu SAR?

SAR đo được **backscatter power** (công suất tín hiệu phản xạ quay về anten). Nhưng giá trị đo thô phụ thuộc vào nhiều yếu tố:

- khoảng cách từ vệ tinh tới điểm ảnh (slant range)
- công suất phát
- gain anten
- góc nhìn radar
- diện tích mặt đất mà một pixel đại diện
- địa hình

Ví dụ:
Hai pixel có cùng một loại đất:
- Pixel A ở gần radar (near range)
- Pixel B ở xa radar (far range)

Nếu chỉ nhìn DN intensity, chúng có thể khác nhau do hình học chứ không phải do bề mặt khác nhau.
Vì vậy cần chuyển sang các đại lượng chuẩn hóa.

---
# 2. Beta nought (β⁰)

## Định nghĩa

Beta nought là backscatter được chuẩn hóa theo **diện tích trên mặt phẳng slant range**.

Nói đơn giản:

> β⁰ biểu diễn công suất phản xạ radar trên một đơn vị diện tích trong mặt phẳng nhìn thấy bởi radar.

Công thức:
$$
\beta^0 =
\frac{P_r}{P_t}
\frac{4\pi R^4}{\lambda^2 G^2 A_{slant}}
$$

Trong thực tế xử lý SAR thường:

$$
\beta^0 = \frac{|DN|^2}{K}
$$

với:

- \(DN\): giá trị ảnh SAR
- \(K\): hệ số calibration

---

## Hình học của β⁰

Radar nhìn thấy mặt đất theo **slant range**:

```
          Satellite
              *
             /
            /
           /
----------/-------------- Ground

       slant range
```

Diện tích pixel được tính trên mặt phẳng nghiêng này.

Do đó:

$$
A_{pixel}= \Delta r \times \Delta az
$$

với:

- Δr: khoảng cách trong slant range
- Δaz: azimuth spacing

---

## Khi nào dùng β⁰?

β⁰ thường là sản phẩm trung gian.

Ví dụ:

- kiểm tra calibration
- phân tích dữ liệu SAR chưa terrain correction
- xử lý interferometry
- một số thuật toán nội bộ của processor

Trong Sentinel-1 GRD:

$$
DN \rightarrow \beta^0
$$

thường là bước đầu tiên.

---

# 3. Sigma nought (σ⁰)

Đây là dạng phổ biến nhất.

## Định nghĩa

Sigma nought là backscatter chuẩn hóa theo:

> diện tích thật trên mặt đất chiếu vuông góc với phương thẳng đứng.

Nói cách khác:

$$
\sigma^0 =
\frac{\text{backscatter power}}
{\text{ground area}}
$$

---

Quan hệ giữa β⁰ và σ⁰:

$$
\sigma^0 = \beta^0 \sin(\theta_i)
$$

hoặc:

$$
\beta^0 = \frac{\sigma^0}{\sin(\theta_i)}
$$

trong đó:

$$
\theta_i
$$

là **incidence angle**.

---

## Vì sao có sin(theta)?

Hãy nhìn hình:

```
              Radar
                *
               /
              /
             /
------------/-------------- Ground
          θ

```

Radar chiếu xiên nên diện tích nhìn thấy trong slant range lớn hơn diện tích mặt đất.

Quan hệ:

$$
A_{ground}
=
\frac{A_{slant}}
{\sin(\theta)}
$$


Vì:

$$
\sigma^0 =
\frac{P}{A_{ground}}
$$

nên:

$$
\sigma^0
=
\beta^0 \sin(\theta)
$$


---

## Ví dụ

Giả sử:

$$
\beta^0=-10dB
$$

Incidence angle:

$$
\theta=30^\circ
$$

Ta có:

$$
10\log_{10}(\sin30^\circ)
=
-3dB
$$

nên:

$$
\sigma^0=-13dB
$$


---

## Ứng dụng σ⁰

Đây là dạng được dùng nhiều nhất:

### Phân loại lớp phủ

Ví dụ:

| Đối tượng | σ⁰ |
|-|-|
| Biển | -20 đến -5 dB |
| Rừng | -15 đến -5 dB |
| Thành phố | -5 đến +5 dB |
| Đất nông nghiệp | -15 đến -7 dB |

---

### Sentinel-1 GRD

Khi bạn mở Sentinel-1 GRD trong SNAP:

```
Amplitude
    |
Calibration
    |
Sigma0
```

thì thường đang tạo:

$$
\sigma^0
$$

---

# 4. Gamma nought (γ⁰)

Gamma nought là dạng chuẩn hóa theo **bề mặt vuông góc với tia radar**.

Định nghĩa:

$$
\gamma^0 =
\frac{\sigma^0}{\cos(\theta)}
$$


Hay:

$$
\sigma^0=\gamma^0\cos(\theta)
$$


---

## Ý tưởng vật lý

Sigma nought vẫn bị ảnh hưởng bởi góc incidence:

```
Flat terrain


       Radar
          \
           \
------------\-----------

        θ

```

Nếu cùng một loại đất nhưng góc nhìn thay đổi:

σ⁰ thay đổi.

Gamma nought cố gắng loại bỏ ảnh hưởng này.

---

## Quan hệ ba đại lượng

Quan hệ cơ bản:

$$
\boxed{
\sigma^0 = \beta^0 \sin(\theta)
}
$$

$$
\boxed{
\gamma^0=\frac{\sigma^0}{\cos(\theta)}
}
$$

Suy ra:

$$
\boxed{
\gamma^0=\beta^0\tan(\theta)
}
$$


---

# 5. So sánh nhanh

| Đại lượng | Chuẩn hóa theo | Công thức | Ý nghĩa |
|-|-|-|-|
| β⁰ | Slant range area | \(β^0\) | Radar nhìn thấy gì |
| σ⁰ | Ground area | \(β^0\sinθ\) | Backscatter trên mặt đất |
| γ⁰ | Radar-normal area | \(σ^0/\cosθ\) | Ít phụ thuộc góc hơn |

---

# 7. Trong Sentinel-1 nên dùng cái nào?

Với Sentinel-1:

| Mục đích | Nên dùng |
|-|-|
| Hiển thị ảnh SAR thông thường | σ⁰ |
| Land cover classification | σ⁰ |
| Crop monitoring | σ⁰ hoặc γ⁰ |
| Terrain correction | γ⁰ |
| Mountain area | γ⁰ + RTC |
| Interferometry | β⁰ hoặc σ⁰ |
| Radiometric analysis | σ⁰ |

---

# 8. Một điểm rất quan trọng với địa hình

Ở vùng núi, σ⁰ bị ảnh hưởng mạnh bởi slope:

Ví dụ:

```
Radar →

       / mountain
      /
_____/________
```

Một sườn núi hướng về radar:

- diện tích hiệu dụng nhỏ
- σ⁰ tăng giả

Một sườn quay đi:

- σ⁰ giảm giả


Khi đó người ta dùng:

- Terrain Flattening
- Radiometric Terrain Correction (RTC)

để tạo:

$$
\gamma^0_{terrain}
$$

Đây là lý do trong các sản phẩm như Sentinel-1 Analysis Ready Data (ARD), người ta thường cung cấp **γ⁰ terrain corrected**.

---

Tóm lại:

- **β⁰**: tín hiệu radar trên **slant range**, gần với dữ liệu SAR gốc.
- **σ⁰**: tín hiệu trên **diện tích mặt đất**, dạng phổ biến nhất trong viễn thám SAR.
- **γ⁰**: đã loại bỏ ảnh hưởng góc incidence tốt hơn, phù hợp vùng địa hình và so sánh đa thời gian.

Nếu làm **SAR processor từ L0 → L1** như dự án của bạn, thì chuỗi logic thường là:

$$
L0
\rightarrow
SLC
\rightarrow
\beta^0
\rightarrow
\sigma^0
\rightarrow
\gamma^0_{RTC}
$$

trong đó calibration radiometric là bước chuyển từ biên độ phức sang các đại lượng vật lý có ý nghĩa.