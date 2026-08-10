Sau khi đối chiếu code, `Terrain-Correction` trong SNAP thực chất là operator **Range–Doppler orthorectification**: tạo lưới bản đồ đích, rồi với mỗi pixel đích dùng DEM và quỹ đạo để tìm ngược vị trí range/azimuth tương ứng trong ảnh SAR nguồn.

Nó không chỉ “dịch pixel”; nó còn resample ảnh, tạo geocoding mới và có một số hiệu chỉnh/tầng dữ liệu tùy chọn.

## Luồng tổng thể trong code

```text
Ảnh SAR ở radar geometry
        ↓
Đọc orbit, timing và thông số range
        ↓
Tạo lưới bản đồ đích
        ↓
Lấy độ cao DEM tại từng pixel đích
        ↓
(lat, lon, height) → tọa độ ECEF
        ↓
Giải phương trình zero-Doppler
        ↓
Tính slant range
        ↓
Tìm range index + azimuth index trong ảnh nguồn
        ↓
Nội suy giá trị ảnh nguồn
        ↓
Ghi ảnh map-projected + metadata + band tùy chọn
```

Operator đăng ký dưới alias `Terrain-Correction` tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:92>).

## 1. Kiểm tra sản phẩm đầu vào

Khi khởi tạo, code yêu cầu:

- Là sản phẩm SAR.
- Chưa map-project.
- Không còn ở dạng TOPSAR burst.

Vì vậy:

- Sentinel‑1 IW/EW SLC phải `TOPSAR-Deburst` trước Terrain Correction.
- Sentinel‑1 SM SLC không có burst nên không cần Deburst.
- Ảnh đã terrain-corrected/map-projected không được chạy lại trực tiếp.

Luồng kiểm tra nằm tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:294>).

## 2. Đọc thông tin hình học SAR

SNAP lấy từ abstract metadata:

- Mission.
- Radar wavelength.
- Range pixel spacing.
- First/last line time.
- Line time interval.
- Orbit state vectors.
- Cờ SRGR.
- Slant range tới pixel đầu tiên đối với slant-range product.
- SRGR coefficients đối với ground-range product.
- Chiều near-range nằm bên trái hay bên phải.
- Thông tin bistatic correction đã được IPF áp dụng hay chưa.

Nếu thiếu orbit state vector, range spacing hoặc SRGR coefficients cần thiết, operator dừng với lỗi. Xem [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:414>).

`Apply-Orbit-File` là bước riêng: Terrain Correction chỉ sử dụng orbit vector hiện có trong metadata, không tự tải precise orbit.

## 3. Tạo lưới bản đồ đầu ra

SNAP xác định:

- CRS đầu ra, mặc định `WGS84(DD)`.
- Phạm vi không gian của sản phẩm.
- Chiều rộng và chiều cao ảnh đích.
- Pixel spacing.
- Geocoding của raster đích.

Nếu người dùng không nhập pixel spacing, code chọn:

$$
\Delta_{\text{output}}
=
\max(\Delta_{\text{range}},\Delta_{\text{azimuth}})
$$

Mục đích là tránh tạo pixel đầu ra nhỏ hơn đáng kể so với độ phân giải sampling nguồn.

Có thể chọn:

- WGS84.
- UTM hoặc CRS khác.
- Pixel spacing theo mét hoặc độ.
- Căn lưới theo một standard grid origin.

Phần tạo target product nằm tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:539>).

## 4. Nạp DEM

Mặc định hiện tại trong code:

```text
DEM: Copernicus 30m Global DEM
DEM resampling: Bilinear
```

SNAP hỗ trợ:

- DEM cài sẵn thông qua `DEMFactory`.
- External DEM.
- Chuyển độ cao geoid sang ellipsoid bằng EGM khi được cấu hình.
- Giá trị NoData của DEM.
- Mask vùng không có elevation.

Mỗi tile đầu ra được mở rộng thêm một pixel xung quanh để tính hình học cục bộ và góc tới. Xem [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:873>).

## 5. Truy ngược từng pixel bản đồ về ảnh SAR

Đây là phần chính.

Với mỗi pixel \((x_t,y_t)\) của ảnh đích, SNAP lấy:

$$
(\varphi,\lambda,h)
$$

gồm latitude, longitude và elevation từ DEM.

Sau đó đổi sang tọa độ Descartes ECEF trên WGS84:

$$
(\varphi,\lambda,h)\rightarrow
\mathbf P=(X,Y,Z)
$$

Trong code:

```java
GeoUtils.geo2xyzWGS84(lat, lon, alt, data.earthPoint);
```

### Giải zero-Doppler time

SNAP tìm thời điểm vệ tinh mà điểm mặt đất thỏa điều kiện zero Doppler. Hiểu đơn giản, đây là thời điểm điểm đó được radar quan sát ở vị trí azimuth tương ứng.

$$
f_D(t,\mathbf P)=0
$$

Orbit position và velocity được nội suy từ các orbit state vector.

### Tính slant range

Tại zero-Doppler time:

$$
R=\left\|\mathbf S(t)-\mathbf P\right\|
$$

Trong đó:

- \(\mathbf S(t)\): vị trí vệ tinh.
- \(\mathbf P\): vị trí điểm mặt đất.
- \(R\): khoảng cách slant range.

### Hiệu chỉnh thời gian bistatic

Code xử lý hai trường hợp:

- Nếu sản phẩm chưa có bistatic correction: áp dụng full correction.
- Với Sentinel‑1 đã được IPF bulk-correct: áp dụng residual phụ thuộc range.

Phần thực thi nằm tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:1508>).

### Atmospheric Path Delay tùy chọn

Nếu bật `Apply Atmospheric Path Delay Correction`, code cộng thêm tropospheric delay vào slant range:

```text
slantRange += atmosphericPathDelay
```

Mục đích là làm mô hình khoảng cách hình học khớp hơn với trục range của sản phẩm, có thể cải thiện vị trí khoảng vài mét theo mô tả tham số.

Mặc định tùy chọn này tắt.

## 6. Tính tọa độ range/azimuth của ảnh nguồn

Azimuth index được tính từ zero-Doppler time:

$$
i_a=
\frac{t_{\text{ZD}}-t_{\text{first line}}}
     {\Delta t_{\text{line}}}
$$

Range index phụ thuộc loại sản phẩm.

### Với slant-range product như SLC

Về nguyên tắc:

$$
i_r=
\frac{R-R_{\text{near}}}
     {\Delta R}
$$

### Với ground-range product

SNAP dùng các SRGR coefficients trong metadata để chuyển slant range sang ground-range index.

Nếu near range nằm phía bên phải ảnh, code đảo index:

$$
i_r' = W-1-i_r
$$

Sau đó code kiểm tra điểm có nằm trong raster nguồn và footprint hợp lệ hay không. Điểm không hợp lệ được ghi NoData.

## 7. Nội suy giá trị ảnh nguồn

Tọa độ \((i_r,i_a)\) thường là số thực, vì vậy SNAP phải nội suy raster nguồn.

Mặc định:

```text
Image resampling: Bilinear interpolation
```

Các lựa chọn trong code gồm:

- Nearest neighbour.
- Bilinear.
- Cubic convolution.
- Bicubic.
- BISINC 5/11/21 điểm.

Với dữ liệu phân loại/index coding, code yêu cầu nearest neighbour để không tạo ra class value trung gian.

Phần nội suy nằm tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:1565>).

Kết quả của bước này là:

```text
giá trị ở radar coordinates
→ giá trị tại pixel map coordinates
```

## 8. Các band phụ có thể tạo

Terrain Correction có thể xuất thêm:

- `elevation`.
- `latitude`.
- `longitude`.
- `incidenceAngleFromEllipsoid`.
- `localIncidenceAngle`.
- `projectedLocalIncidenceAngle`.
- `layoverShadowMask`.

Các góc có ý nghĩa khác nhau:

- `incidenceAngleFromEllipsoid`: dùng ellipsoid, không xét hướng dốc DEM.
- `localIncidenceAngle`: góc giữa tia radar và pháp tuyến địa hình.
- `projectedLocalIncidenceAngle`: góc local được chiếu vào range plane, được dùng bởi một số công thức radiometric normalization.

## 9. Layover/shadow mask

Nếu bật `Save layover shadow mask`, SNAP quét địa hình theo hướng near-to-far range.

### Layover

Code kiểm tra tính đơn điệu của slant range. Nếu một điểm xa hơn theo mặt đất nhưng có slant range không tăng như dự kiến, điểm được đánh dấu layover.

### Shadow

Code theo dõi elevation angle lớn nhất theo hướng nhìn. Điểm bị địa hình phía trước che khuất được đánh dấu shadow.

Các giá trị mask:

```text
0 = bình thường
1 = layover
2 = shadow
3 = vừa layover vừa shadow
```

Logic nằm tại [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:1217>).

Operator chỉ đánh dấu vùng layover/shadow; nó không thể khôi phục thông tin radar đã mất tại các vùng đó.

## 10. Radiometric normalization tùy chọn

Mặc định:

```text
applyRadiometricNormalization = false
```

Do đó Terrain Correction thông thường chỉ resample các band nguồn sang map geometry.

Nếu bật tùy chọn này, operator có thể:

- Tạo/calibrate `Sigma0`.
- Tạo `Gamma0` virtual band.
- Tạo `Beta0` virtual band.
- Dùng ellipsoid incidence angle, local incidence angle hoặc projected local incidence angle tùy cấu hình.

Trong vòng lặp pixel, code gọi calibrator sau khi đã tìm được vị trí range/azimuth nguồn:

```java
v = calibrator.applyCalibration(...);
```

Xem [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:1089>).

Điểm cần phân biệt:

- Tùy chọn radiometric normalization trong Terrain Correction có thể calibration/chuẩn hóa theo góc tới.
- Nó vẫn không phải thuật toán Terrain Flattening theo illuminated area của DEM.
- Với workflow Sentinel‑1 hiện đại, dễ kiểm soát nhất vẫn là chạy `Calibration` riêng trước Terrain Correction.

## 11. Cập nhật sản phẩm đầu ra

SNAP tạo sản phẩm hậu tố:

```text
_TC
```

Metadata được cập nhật:

- `srgr_flag = 1`.
- Kích thước raster mới.
- Tọa độ bốn góc.
- Map projection.
- Pixel spacing.
- DEM đã dùng.
- `is_terrain_corrected = 1`.
- Look directions.
- Georeferencing mới.

Xem [RangeDopplerGeocodingOp.java](</home/hoangnv307/code/snap/microwave-toolbox/sar-op-sar-processing/src/main/java/eu/esa/sar/sar/gpf/geometric/RangeDopplerGeocodingOp.java:811>).

## Các giá trị mặc định đáng chú ý

| Tham số | Mặc định |
|---|---|
| DEM | Copernicus 30m Global DEM |
| DEM resampling | Bilinear |
| Image resampling | Bilinear |
| CRS | WGS84 latitude/longitude |
| Pixel spacing | Tự tính từ range/azimuth spacing |
| Mask sea/no elevation | Bật |
| Save source bands | Bật |
| Radiometric normalization | Tắt |
| Save layover/shadow | Tắt |
| Atmospheric delay correction | Tắt |

## Với Sentinel‑1 SM SLC

Luồng phù hợp là:

```text
SM SLC
→ Apply Orbit File
→ Thermal Noise Removal
→ Calibration thành Sigma0/Beta0
→ Multilook nếu cần
→ Terrain Flattening nếu địa hình dốc
→ Terrain Correction
```

Khi đến Terrain Correction, operator thực hiện:

```text
tọa độ bản đồ + DEM
→ vị trí 3D mặt đất
→ zero-Doppler time
→ slant range
→ range/azimuth nguồn
→ nội suy backscatter
→ ảnh geocoded
```

Tóm lại, Terrain Correction trong SNAP sửa **hình học và hệ tọa độ** của ảnh SAR. Các chức năng radiometric, incidence-angle bands và layover/shadow mask là tùy chọn đi kèm, không phải bản chất chính của operator.