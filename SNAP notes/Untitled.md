Đúng, phần trước mình nhấn mạnh TOPS SLC nên chưa làm rõ Stripmap SLC. Với Sentinel‑1 SM SLC, luồng đơn giản hơn vì không có burst/sub-swath kiểu TOPS và không cần `TOPSAR-Deburst`.

## Luồng cho Sentinel‑1 Stripmap SLC

```text
Đọc SM SLC
  ↓
Apply Orbit File
  ↓
Thermal Noise Removal
  ↓
Radiometric Calibration
  ↓
Multilook — nếu cần chuyển sang ảnh detected/backscatter
  ↓
Speckle Filter — tùy mục đích
  ↓
Terrain Flattening — nếu địa hình dốc
  ↓
Terrain Correction
  ↓
Chuyển sang dB
```

### 1. Không chạy GRD Border Noise Removal

`Remove-GRD-Border-Noise` chỉ chấp nhận sản phẩm GRD. SM SLC không có loại border artefact này, nên bỏ qua hoàn toàn bước đó.

### 2. Thermal Noise Removal cho SM SLC

SNAP hỗ trợ Thermal Noise Removal cho Sentinel‑1 SM SLC. Operator chấp nhận:

- Mode: IW, EW, SM.
- Product type: SLC, GRD.

Xem [Sentinel1RemoveThermalNoiseOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/Sentinel1RemoveThermalNoiseOp.java:137>).

Khác TOPS:

- IW/EW từ IPF 2.9 có cả noise range vector và noise azimuth vector.
- SM không có noise azimuth vector.
- Vì vậy SNAP dùng các noise vector theo range và nội suy giữa các dòng azimuth, không xây dựng noise block theo burst/sub-swath.

Với mẫu phức:

$$
P = I^2 + Q^2
$$

Sau đó:

$$
P_{\text{clean}} = P - N
$$

Nếu sẽ multilook hoặc lấy trung bình về sau, cân nhắc tắt `Clip Negative Values` để tránh positive bias.

### 3. Radiometric Calibration

`Sentinel1Calibrator` hỗ trợ SM SLC trực tiếp. Nó đọc calibration vector tương ứng polarization và swath, nội suy LUT theo range và azimuth, rồi tính:

$$
C_t=\frac{I^2+Q^2}{L_t^2}
$$

Trong đó $L_t$ có thể là LUT của:

- `Sigma0`
- `Beta0`
- `Gamma0`
- `DN`

Không giống TOPS SLC, SM SLC không có ràng buộc “calibration trước deburst”, vì Stripmap không có bước Deburst.

### 4. Chọn intensity hay complex output

Nếu mục tiêu là backscatter định lượng hoặc phân loại:

```text
outputImageInComplex = false
outputSigmaBand = true
```

Đầu ra sẽ là intensity:

```text
Sigma0_VV
Sigma0_VH
```

Nếu cần giữ pha cho giao thoa hoặc xử lý coherent:

```text
outputImageInComplex = true
```

SNAP hiệu chỉnh biên độ nhưng bảo toàn pha:

$$
I_{\text{cal}}
=
\sqrt{\frac{I^2+Q^2}{L^2}}
\frac{I}{\sqrt{I^2+Q^2}}
$$

$$
Q_{\text{cal}}
=
\sqrt{\frac{I^2+Q^2}{L^2}}
\frac{Q}{\sqrt{I^2+Q^2}}
$$

Sau đó vẫn có thể tạo virtual intensity từ cặp I/Q. Logic này nằm trong [Sentinel1Calibrator.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/calibrators/Sentinel1Calibrator.java:422>).

## Hai luồng theo mục đích

### Phân tích backscatter

```text
SM SLC
→ Apply Orbit File
→ Thermal Noise Removal
→ Calibration thành Sigma0 hoặc Beta0 intensity
→ Multilook
→ Speckle Filter
→ Terrain Flattening nếu cần
→ Terrain Correction
→ dB
```

Có thể multilook để giảm speckle và tạo kích thước pixel gần vuông.

### InSAR hoặc xử lý pha

```text
SM SLC
→ Apply Orbit File
→ Thermal Noise Removal
→ Calibration với complex output, nếu cần biên độ calibrated
→ Coregistration
→ Interferogram
→ Phase Filtering
→ Geocoding/Terrain Correction
```

Không thực hiện các bước làm mất pha như:

- Calibration sang detected intensity.
- Speckle filter dành cho intensity.
- Multilook quá sớm nếu vẫn cần độ phân giải pha ban đầu.

Ngoài ra, `CalibrationOp` không cho chạy trên sản phẩm đã trở thành coregistered stack. Nếu cần calibration complex cho InSAR, phải làm trước coregistration.

## So sánh ngắn

| Bước | TOPS IW/EW SLC | Stripmap SM SLC |
|---|---:|---:|
| Thermal noise removal | Có | Có |
| Calibration | Có | Có |
| TOPSAR Split | Thường có | Không |
| TOPSAR Deburst | Bắt buộc trước xử lý ảnh liên tục | Không |
| Calibration phải trước Deburst | Có | Không áp dụng |
| Noise azimuth vector mới | Có | Không |
| Multilook | Tùy mục đích | Tùy mục đích |
| Terrain correction | Có | Có |

Vì vậy, luồng radiometric cốt lõi của SM SLC chỉ là:

```text
Thermal Noise Removal → Calibration
```

Phần còn lại là xử lý hình học, giảm speckle hoặc chuẩn hóa địa hình tùy sản phẩm đầu ra mong muốn.