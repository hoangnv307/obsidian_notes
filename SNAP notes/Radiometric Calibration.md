Theo luồng hiện tại của SNAP Microwave Toolbox (`2d44359ae`, ngày 02/07/2026), radiometric correction cho Sentinel‑1 nên được hiểu thành hai phần:

1. Làm sạch các sai lệch bức xạ: border noise và thermal noise.
2. Radiometric calibration: chuyển DN thành \(\sigma^0\), \(\beta^0\) hoặc \(\gamma^0\).

## Luồng khuyến nghị

### Với Sentinel‑1 GRD

```text
Đọc Level-1 GRD
  ↓
Apply Orbit File (khuyến nghị, nhưng không phải hiệu chỉnh bức xạ)
  ↓
Remove GRD Border Noise
  ↓
Thermal Noise Removal
  ↓
Calibration → Sigma0/Beta0/Gamma0 dạng tuyến tính
  ↓
Speckle Filter hoặc Multilook (nếu cần)
  ↓
Terrain Flattening (nếu địa hình dốc và cần định lượng)
  ↓
Terrain Correction
  ↓
Chuyển sang dB nếu cần
```

### Với Sentinel‑1 TOPS SLC

```text
Đọc Level-1 SLC
  ↓
TOPSAR Split / Apply Orbit File (nếu cần)
  ↓
Thermal Noise Removal
  ↓
Calibration
  ↓
TOPSAR Deburst
  ↓
Multilook / Speckle Filter
  ↓
Terrain Flattening (tùy mục đích)
  ↓
Terrain Correction
  ↓
Chuyển sang dB
```

Điểm quan trọng từ code: calibration của TOPS SLC phức phải được thực hiện trước Deburst; SNAP chủ động báo lỗi nếu thứ tự ngược lại. Xem [Sentinel1Calibrator.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/calibrators/Sentinel1Calibrator.java:90>).

## Các bước chi tiết

### 1. Đọc và kiểm tra sản phẩm

SNAP kiểm tra:

- Đây là sản phẩm SAR Sentinel‑1.
- Mode thuộc IW, EW, SM hoặc WV đối với Calibration.
- Loại sản phẩm là SLC hoặc GRD.
- Sản phẩm chưa phải coregistered stack.
- Với TOPS SLC, chưa được Deburst.
- Sản phẩm có metadata calibration và các band/phân cực hợp lệ.

`CalibrationOp` là operator điều phối. Nó đọc mission từ metadata rồi chọn `Sentinel1Calibrator` thông qua factory, thay vì chứa trực tiếp thuật toán Sentinel‑1. Xem [CalibrationOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/CalibrationOp.java:119>).

### 2. Loại bỏ GRD border noise

Chỉ áp dụng cho GRD. Operator này:

- Tìm các vùng “no-value” ở rìa ảnh.
- Dùng tín hiệu co-polarization HH hoặc VV để phát hiện biên.
- Mask các mẫu biên có giá trị thấp hoặc chứa artefact xử lý.
- Xử lý khác nhau tùy phiên bản IPF.

Bước này phải đứng trước Calibration vì operator yêu cầu đầu vào chưa được hiệu chỉnh tuyệt đối (`checkIfCalibrated(false)`). Xem [RemoveGRDBorderNoiseOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/RemoveGRDBorderNoiseOp.java:118>).

Không áp dụng bước này cho SLC.

### 3. Loại bỏ thermal noise

SNAP đọc noise LUT trong annotation theo từng:

- Polarization.
- Sub-swath.
- Vị trí range.
- Thời gian hoặc dòng azimuth.

Sau đó nội suy LUT để có mức nhiễu tại từng pixel:

\[
P_{\text{clean}} = P_{\text{input}} - N
\]

Trong đó:

- Với GRD amplitude: \(P_{\text{input}}=DN^2\).
- Với SLC phức: \(P_{\text{input}}=I^2+Q^2\).
- \(N\) là noise power nội suy từ noise LUT.

SNAP cho phép Thermal Noise Removal chạy cả trước lẫn sau Calibration. Nếu đầu vào đã calibrated, noise được đổi sang cùng thang đo:

\[
N_{\text{calibrated}}=\frac{N}{L^2}
\]

Tuy nhiên graph chuẩn `Sentinel1SLCtoGRD` của SNAP dùng thứ tự:

```text
ThermalNoiseRemoval → Calibration → TOPSAR-Deburst
```

Xem [Sentinel1SLCtoGRD.xml](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sentinel1-ui/src/main/resources/eu/esa/sar/sentinel1/graphs/internal/sentinel1/Sentinel1SLCtoGRDGraph.xml:10>) và [Sentinel1RemoveThermalNoiseOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/Sentinel1RemoveThermalNoiseOp.java:584>).

Một số lưu ý:

- Không chạy lại nếu metadata cho biết thermal noise đã được loại bỏ.
- Với slice, thermal noise cần được loại trước Slice Assembly.
- `clipNegativeValues=true` mặc định thay kết quả âm bằng \(10^{-5}\).
- Nếu sau đó sẽ multilook, lọc không gian hoặc tổng hợp thời gian, nên cân nhắc `clipNegativeValues=false`; clip trước khi lấy trung bình có thể gây positive bias.

### 4. Chọn polarization và loại kết quả calibration

Nếu không chọn polarization, SNAP lấy toàn bộ polarization từ metadata.

Các đầu ra hỗ trợ:

- `Sigma0`: diện tích tán xạ chuẩn hóa trên mặt đất; đầu ra mặc định và phổ biến nhất.
- `Beta0`: radar brightness theo hình học slant range.
- `Gamma0`: chuẩn hóa theo mặt phẳng vuông góc với hướng nhìn.
- `DN`: khôi phục thang DN bằng DN LUT, chủ yếu phục vụ kiểm tra hoặc chuyển đổi ngược.

Với phân tích đất phủ GRD thông thường, chọn `Sigma0_VV`, `Sigma0_VH` hoặc các polarization tương ứng là đủ.

Nếu chuẩn bị Terrain Flattening cho vùng địa hình dốc, `Beta0` thường là đầu vào phù hợp với luồng Terrain Flattening của SNAP.

### 5. Đọc calibration vectors

SNAP đọc từ metadata gốc:

```text
calibration
  └─ calibrationDataSet
       └─ calibration
            ├─ adsHeader
            └─ calibrationVectorList
```

Mỗi vector chứa:

- Dòng azimuth và thời gian.
- Các vị trí pixel theo range.
- `sigmaNought[]`.
- `betaNought[]`.
- `gamma[]`.
- `dn[]`.

Các vector được ánh xạ theo polarization và sub-swath. Xem [Sentinel1Calibrator.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/calibrators/Sentinel1Calibrator.java:187>).

### 6. Nội suy calibration LUT

Đối với mỗi pixel, SNAP tìm:

- Hai calibration vector trước và sau theo azimuth.
- Hai điểm LUT trái và phải theo range.

Sau đó thực hiện nội suy song tuyến tính:

\[
L(x,y) =
(1-\mu_y)\left[(1-\mu_x)L_{00}+\mu_xL_{01}\right]
+
\mu_y\left[(1-\mu_x)L_{10}+\mu_xL_{11}\right]
\]

Nhờ vậy hệ số calibration thay đổi liên tục theo cả range và azimuth, thay vì dùng một hằng số cho toàn ảnh.

### 7. Chuyển mẫu đầu vào sang công suất

SNAP chuẩn hóa dữ liệu đầu vào thành intensity/power:

\[
P =
\begin{cases}
DN^2, & \text{amplitude}\\
I^2+Q^2, & \text{SLC phức}\\
DN, & \text{intensity}\\
10^{DN_{dB}/10}, & \text{intensity dB}
\end{cases}
\]

Sau đó áp dụng LUT được chọn:

\[
C_t = \frac{P}{L_t^2}
\]

Trong đó \(t\) là `Sigma0`, `Beta0`, `Gamma0` hoặc `DN`.

Phần cài đặt nằm tại [Sentinel1Calibrator.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/calibrators/Sentinel1Calibrator.java:327>).

Nếu chuyển từ một band đã calibrated sang loại khác, SNAP hoàn tác LUT cũ rồi áp dụng LUT mới:

\[
C_t=C_s\frac{L_s^2}{L_t^2}
\]

### 8. Xử lý SLC phức

Mặc định Calibration biến cặp I/Q thành intensity calibrated.

Nếu chọn lưu complex output, SNAP:

1. Tính \(I^2+Q^2\).
2. Calibrate công suất.
3. Lấy căn bậc hai.
4. Gắn lại thành phần pha chuẩn hóa của I hoặc Q.

Như vậy biên độ được calibration nhưng pha được bảo toàn. Complex output chỉ dùng khi xử lý tiếp cần thông tin pha; GRD không có lựa chọn này.

### 9. Cập nhật metadata

Sau calibration, SNAP:

- Đặt `abs_calibration_flag=true`.
- Đổi sample type thành `DETECTED` nếu không lưu complex.
- Tạo sản phẩm hậu tố `_Cal`.
- Tạo band dạng `Sigma0_VV`, `Sigma0_VH`, `Beta0_*` hoặc `Gamma0_*`.

Điều này cũng ngăn một số operator vô tình calibration lại sản phẩm.

## Terrain Flattening không đồng nghĩa với Calibration

Đây là hai hiệu chỉnh khác nhau:

- `Calibration`: loại ảnh hưởng của hệ thống cảm biến và chuyển DN sang backscatter vật lý.
- `Terrain Flattening`: hiệu chỉnh biến thiên bức xạ do độ dốc và hướng địa hình bằng DEM.

Với khu vực tương đối phẳng:

```text
Sigma0 → Terrain Correction
```

thường đủ.

Với khu vực đồi núi hoặc cần so sánh định lượng:

```text
Beta0 → Terrain Flattening → Gamma0 terrain-flattened → Terrain Correction
```

sẽ phù hợp hơn. `Gamma0` lấy trực tiếp từ calibration LUT không hoàn toàn tương đương `Gamma0` đã terrain-flattened bằng DEM.

## Thang tuyến tính và dB

Nên giữ dữ liệu ở thang tuyến tính trong các bước:

- Thermal noise subtraction.
- Multilook.
- Speckle filtering.
- Temporal averaging.
- Terrain flattening.

Chỉ chuyển sang dB ở cuối:

\[
C_{dB}=10\log_{10}(C_{\text{linear}})
\]

Giao diện Calibration của SNAP thực tế ẩn lựa chọn “Save in dB” đối với Sentinel‑1, thể hiện chủ ý xử lý Sentinel‑1 ở dạng intensity tuyến tính. Xem [CalibrationOpUI.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration-ui/src/main/java/eu/esa/sar/calibration/gpf/ui/CalibrationOpUI.java:205>).

Tóm lại, chuỗi tối thiểu đáng dùng cho Sentinel‑1 GRD là:

```text
Remove GRD Border Noise
→ Thermal Noise Removal
→ Calibration (Sigma0 linear)
→ Terrain Correction
→ dB
```

Thêm Terrain Flattening trước Terrain Correction khi địa hình dốc hoặc bài toán yêu cầu backscatter có khả năng so sánh định lượng theo địa hình.