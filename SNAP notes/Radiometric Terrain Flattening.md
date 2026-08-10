Action **`Radar > Radiometric > Radiometric Terrain Flattening`** gọi operator `Terrain-Flattening`. Nó thực hiện Radiometric Terrain Correction theo thuật toán của David Small: dùng DEM để mô phỏng diện tích thực sự được radar chiếu sáng, rồi chuẩn hóa `Beta0` thành `Gamma0` terrain-flattened.

Nó không thực hiện geocoding/map projection; sau action này vẫn cần chạy `Terrain Correction`.

## Luồng tổng thể

```text
Mở Radiometric Terrain Flattening
  ↓
Kiểm tra sản phẩm SAR chưa map-project
  ↓
Chuẩn bị Beta0 hoặc ma trận PolSAR
  ↓
Đọc DEM, quỹ đạo và hình học radar
  ↓
Oversample DEM
  ↓
Tính diện tích chiếu sáng của từng ô DEM
  ↓
Ánh xạ diện tích đó vào lưới range/azimuth
  ↓
Tạo simulated reference area
  ↓
Gamma0 TF = Beta0 / simulated reference area
  ↓
Tạo sản phẩm vẫn ở radar geometry
```

Action được đăng ký tại [layer.xml](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing-ui/src/main/resources/eu/esa/sar/sar/layer.xml:116>).

## 1. Các tham số trên dialog

Dialog cho phép chọn:

- Source bands.
- Digital Elevation Model.
- DEM resampling method.
- External DEM và NoData.
- Có áp dụng EGM cho external DEM hay không.
- Output simulated image.
- Output terrain-flattened Sigma0.
- Mask vùng không có elevation.
- Additional overlap.
- Oversampling multiple.

Các giá trị mặc định:

| Tham số | Mặc định |
|---|---|
| DEM | Copernicus 30m Global DEM |
| DEM resampling | Bilinear |
| Output Gamma0 | Luôn bật |
| Output Sigma0 TF | Tắt |
| Output simulated image | Tắt |
| Mask areas without elevation | Bật |
| Additional overlap | 0.1 |
| Oversampling multiple | 1.0 |
| External DEM Apply EGM | Tắt |

`Output Terrain Flattened Gamma0` luôn được chọn và bị khóa, tức action luôn tạo Gamma0 TF.

Xem [TerrainFlatteningOpUI.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing-ui/src/main/java/eu/esa/sar/sar/gpf/ui/geometric/TerrainFlatteningOpUI.java:41>).

## 2. Kiểm tra sản phẩm đầu vào

Operator kiểm tra:

- Đây là sản phẩm SAR.
- Sản phẩm chưa map-project.
- Có geocoding.
- Có incidence-angle tie-point grid.
- Có orbit state vectors và thông tin timing.
- Có range và azimuth pixel spacing.

Điều kiện “chưa map-project” đảm bảo Terrain Flattening chạy trước Terrain Correction.

Xem [TerrainFlatteningOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/TerrainFlatteningOp.java:190>).

## 3. Chuẩn bị dữ liệu Beta0

### Nếu đầu vào chưa calibration

Code tự tạo một `CalibrationOp` nội bộ và yêu cầu:

```text
outputBetaBand = true
```

Nghĩa là người dùng không bắt buộc phải chạy action `Calibrate` trước; Terrain Flattening có thể tự tạo Beta0 trung gian.

```text
DN hoặc I/Q
→ Calibration nội bộ
→ Beta0
→ Terrain Flattening
```

### Nếu đầu vào đã calibration

Code không calibration lại. Sản phẩm phải chứa band bắt đầu bằng `Beta0`.

Nếu đầu vào chỉ có:

```text
Sigma0_VV
Sigma0_VH
```

mà không có `Beta0`, operator sẽ báo:

```text
TerrainFlattening requires beta0 as input
```

Đây là lưu ý quan trọng: nếu định chạy `Calibrate` thủ công trước, phải bật `Output beta0 band`.

### Sản phẩm polarimetric

Operator cũng chấp nhận:

- T3
- C3
- C2

Trong trường hợp này, các thành phần ma trận được terrain-flatten trực tiếp.

## 4. Tạo sản phẩm đích trong radar geometry

Sản phẩm đầu ra:

- Giữ nguyên chiều rộng và chiều cao.
- Giữ lưới range/azimuth của đầu vào.
- Sao chép metadata, geocoding và product nodes.
- Ghi DEM và phương pháp DEM resampling vào metadata.
- Đặt `abs_calibration_flag = 1`.

Band được đổi tên:

```text
Beta0_VV → Gamma0_VV
Beta0_VH → Gamma0_VH
```

Nếu bật `Output Terrain Flattened Sigma0`:

```text
Beta0_VV → Sigma0_VV
```

Lưu ý đây là Sigma0 đã hiệu chỉnh ảnh hưởng địa hình, không phải Sigma0 calibration thông thường.

## 5. Nạp và oversample DEM

DEM được nạp từ:

- DEM cài sẵn qua `DEMFactory`.
- External DEM qua `FileElevationModel`.

Nếu external DEM bật EGM, SNAP chuyển độ cao theo mô hình trọng trường tương ứng.

Thuật toán yêu cầu DEM có sampling đủ nhỏ so với pixel SAR. Nếu DEM thô hơn, code tự tính hệ số oversampling dựa trên:

- Độ phân giải DEM.
- Range spacing.
- Azimuth spacing.
- `Oversampling Multiple`.

Ví dụ với Copernicus 30 m và ảnh SAR có spacing 10 m, DEM sẽ được oversample trước khi mô phỏng địa hình.

Oversampling chỉ làm lưới DEM dày hơn; nó không tạo thêm chi tiết địa hình thật. Với chênh lệch độ phân giải lớn, multilook ảnh SAR trước có thể hợp lý hơn.

## 6. Tính vùng tile cần mở rộng

Địa hình ở tile lân cận có thể chiếu năng lượng vào tile hiện tại. Vì vậy operator không chỉ tính đúng phạm vi tile đầu ra.

Code:

1. Lấy mẫu một số điểm trong và quanh tile.
2. Ánh xạ chúng qua DEM và Range–Doppler.
3. Ước lượng độ lệch range/azimuth.
4. Mở rộng tile theo bốn hướng.
5. Cộng thêm `Additional Overlap`, mặc định 10%.

Tham số này giúp giảm đường nối hoặc artefact ở ranh giới tile.

## 7. Tạo lưới DEM cục bộ

Đối với vùng mở rộng, SNAP:

1. Xác định giới hạn latitude/longitude.
2. Mở rộng thêm biên khoảng 20 DEM samples.
3. Đọc elevation.
4. Nội suy/oversample DEM.
5. Tạo các ô địa hình nhỏ từ bốn điểm lân cận:

```text
t00 ─── t10
 │       │
t01 ─── t11
```

Mỗi điểm chứa latitude, longitude và elevation.

Nếu DEM trả NoData:

- Khi mask bật: vùng đó bị bỏ qua.
- Khi mask tắt: code sử dụng EGM elevation làm giá trị thay thế gần mực nước biển.

## 8. Ánh xạ từng ô DEM vào radar geometry

Mỗi điểm DEM được đổi sang tọa độ ECEF WGS84:

$$
(\varphi,\lambda,h)\rightarrow(X,Y,Z)
$$

Sau đó operator:

1. Tìm zero-Doppler time.
2. Nội suy vị trí vệ tinh.
3. Tính slant range.
4. Áp dụng bistatic correction hoặc residual correction.
5. Có thể áp dụng atmospheric path delay nếu parameter nội bộ được bật.
6. Tính azimuth index.
7. Tính range index, trực tiếp cho SLC hoặc qua SRGR coefficients cho GRD.

Kết quả là mỗi ô DEM được ánh xạ về một vị trí số thực trong lưới range/azimuth của ảnh SAR.

## 9. Tính diện tích chiếu sáng cho Gamma0

Đây là phần cốt lõi.

Với bốn đỉnh địa hình, code xác định vector hướng slant range:

$$
\mathbf{s}
=
\frac{\mathbf{S}-\mathbf{P}}
     {\left\|\mathbf{S}-\mathbf{P}\right\|}
$$

Trong đó:

- $\mathbf S$ là vị trí vệ tinh.
- $\mathbf P$ là vị trí ô địa hình.

Bốn đỉnh địa hình được chiếu lên mặt phẳng vuông góc với hướng slant range:

$$
\mathbf p
=
\mathbf t-(\mathbf t\cdot\mathbf s)\mathbf s
$$

Ô tứ giác sau khi chiếu được chia thành hai tam giác. Code dùng công thức Heron để tính tổng diện tích hai tam giác:

$$
A_{\gamma}
=
A_{\triangle 1}+A_{\triangle 2}
$$

Đây là diện tích tham chiếu dùng để tạo Gamma0 terrain-flattened.

Phần tính toán nằm tại [TerrainFlatteningOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/TerrainFlatteningOp.java:1301>).

## 10. Tính reference area cho Sigma0 tùy chọn

Nếu bật `Output Terrain Flattened Sigma0`, operator còn tính diện tích của ô địa hình trong không gian 3D, không chiếu lên mặt phẳng vuông góc slant range:

$$
A_{\sigma}
=
A_{\triangle 1,\mathrm{3D}}
+
A_{\triangle 2,\mathrm{3D}}
$$

Reference area này được dùng riêng cho output Sigma0 TF.

## 11. Loại vùng radar shadow

Code quét các điểm theo hướng near range đến far range, hoặc ngược lại tùy vị trí quỹ đạo.

Nó theo dõi elevation angle lớn nhất. Điểm nằm sau địa hình có elevation angle thấp hơn bị xem là shadow và không được cộng vào reference-area image.

Operator cũng loại các vùng foreshortening/layover có simulated area quá nhỏ, vì phép chia ở đó không đáng tin cậy.

Ngưỡng nội bộ hiện tại là:

```text
threshold = 0.05
```

Các pixel không đáng tin cậy được ghi NoData.

## 12. Phân bố illuminated area vào pixel SAR

Một ô DEM thường ánh xạ vào tọa độ range/azimuth dạng số thực.

SNAP phân bố diện tích của nó vào bốn pixel SAR lân cận bằng trọng số song tuyến tính:

```text
(ir0, ia0)  (ir1, ia0)
(ir0, ia1)  (ir1, ia1)
```

Tổng đóng góp từ các ô DEM tạo thành `gamma0ReferenceArea` cho từng pixel SAR.

## 13. Chuẩn hóa Beta0

Diện tích tham chiếu Beta0 của một pixel radar được lấy từ sampling:

$$
A_{\beta}
=
\Delta_{\mathrm{azimuth}}
\Delta_{\mathrm{range}}
$$

Reference ratio cho Gamma0 được tính gần tương đương:

$$
R_{\gamma}
=
\frac{A_{\gamma}}{A_{\beta}}
$$

Với GRD, code hiệu chỉnh thêm tỷ lệ biến đổi SRGR.

Giá trị terrain-flattened cuối cùng:

$$
\gamma^0_{\mathrm{TF}}
=
\frac{\beta^0}{R_{\gamma}}
$$

Trong code, đây chính là:

```java
target = sourceBeta0 / simulatedReferenceRatio;
```

Xem [TerrainFlatteningOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/TerrainFlatteningOp.java:901>).

Nếu xuất Sigma0 TF:

$$
\sigma^0_{\mathrm{TF}}
=
\frac{\beta^0}{R_{\sigma}}
$$

## 14. Output Simulated Image

Nếu bật `Output Simulated Image`, SNAP tạo band:

```text
simulatedImage
```

Band này chứa reference-area ratio đã chuẩn hóa:

$$
R_{\gamma}
=
\frac{A_{\gamma}}{A_{\beta}}
$$

Nó hữu ích để:

- Kiểm tra ảnh hưởng địa hình.
- Phân tích vùng shadow/layover.
- Multitemporal compositing theo simulated area.
- Chẩn đoán artefact do DEM.

Nó không phải backscatter thực tế.

## Action này không thực hiện gì?

Radiometric Terrain Flattening không thực hiện:

- Thermal Noise Removal.
- GRD Border Noise Removal.
- Speckle Filter.
- Multilook.
- Chuyển sang dB.
- Map projection.
- Range–Doppler Terrain Correction.

Đầu ra vẫn ở radar geometry, cùng kích thước với ảnh nguồn.

## Luồng phù hợp cho Sentinel‑1 SM SLC

Có thể để action tự calibration:

```text
SM SLC
→ Apply Orbit File
→ Thermal Noise Removal
→ Radiometric Terrain Flattening
   ├── Calibration nội bộ thành Beta0
   └── Beta0 → Gamma0 TF
→ Terrain Correction
→ dB
```

Hoặc calibration thủ công:

```text
SM SLC
→ Apply Orbit File
→ Thermal Noise Removal
→ Calibrate với Output Beta0
→ Radiometric Terrain Flattening
→ Terrain Correction
→ dB
```

Nếu đã chạy `Calibrate` nhưng chỉ xuất `Sigma0`, Terrain Flattening sẽ không tìm thấy `Beta0` và không thể tiếp tục.