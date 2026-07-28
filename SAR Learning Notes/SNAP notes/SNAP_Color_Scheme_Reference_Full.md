

Nguồn: `color_palettes_schemes.txt` của SNAP.

## 1. Ý nghĩa các trường trong file SNAP

| Trường | Ý nghĩa |
|---|---|
| ID | Tên color scheme |
| MIN/MAX | Khoảng giá trị được ánh xạ màu |
| LOG_SCALE | Có dùng ánh xạ logarit hay không |
| CPD_FILENAME | File palette màu `.cpd` |
| COLORBAR_LABELS | Các mốc hiển thị trên thanh màu |
| DESCRIPTION | Ý nghĩa vật lý của dữ liệu |

---
# 2. Ocean Optical Properties

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| absorption | Absorption coefficient | oceancolor_standard.cpd | Log | Hấp thụ ánh sáng trong nước |
| adg_s | Gelbstoff and detrital absorption spectral slope | oceancolor_standard.cpd | Linear | Phân tích chất hữu cơ hòa tan |
| angstrom | Angstrom coefficient | oceancolor_standard.cpd | Linear | Đặc tính aerosol/khí quyển |
| aot | Aerosol optical thickness | oceancolor_standard.cpd | Linear | Giám sát aerosol |
| bb | Total backscatter | oceancolor_standard.cpd | Log | Tán xạ tổng trong nước |
| bbp | Particle backscatter | oceancolor_standard.cpd | Log | Hạt lơ lửng trong nước |
| bbp_giop | Particle backscatter GIOP | oceancolor_standard.cpd | Log | Mô hình quang học biển |
| bbw | Water backscatter coefficient | oceancolor_standard.cpd | Log | Tính chất quang học nước biển |

---

# 3. Chlorophyll and Ocean Biology

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| chlor_a | Chlorophyll-a concentration | oceancolor_standard.cpd | Log | Theo dõi sinh khối thực vật biển |
| chlor_a_uni | Chlorophyll-a universal palette | universal_bluered.cpd | Log | Hiển thị chlorophyll |
| chlor_a_bluegreen | Chlorophyll blue-green | chlor_blue_green.cpd | Log | Phân biệt vùng nghèo/giàu chlorophyll |
| chlor_a_uni_bg | Chlorophyll blue-green universal | universal_bluegreen.cpd | Log | Palette thân thiện màu |
| calcite | Calcite concentration | oceancolor_standard.cpd | Log | Khoáng vật carbonate |
| BSi | Biogenic Silica | oceancolor_standard.cpd | Log | Silica sinh học |
| pic | Particulate inorganic carbon | oceancolor_standard.cpd | Log | Carbon vô cơ dạng hạt |
| poc | Particulate organic carbon | oceancolor_standard.cpd | Log | Carbon hữu cơ dạng hạt |

---

# 4. Vegetation Indices

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| NDVI | Normalized Difference Vegetation Index | oceancolor_ndvi.cpd | Linear | Giám sát thực vật |
| ndvi | NDVI | oceancolor_ndvi.cpd | Linear | Phân tích thảm phủ |
| EVI | Enhanced Vegetation Index | oceancolor_ndvi.cpd | Linear | Sinh khối thực vật |
| evi | Enhanced Vegetation Index | oceancolor_ndvi.cpd | Linear | Phân tích cây trồng |

---

# 5. Terrain and Elevation

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| elevation | Elevation above sea level | gray_scale.cpd | Linear | DEM |
| elev | Elevation | oceancolor_standard.cpd | Linear | Hiển thị cao độ |
| bathymetry | Ocean depth | smooth_inv_blue.cpd | Log | Độ sâu đáy biển |
| topography | Land topography | smooth_green.cpd | Log | Địa hình mặt đất |
| topography_ETOP | Global relief | topography.cpd | Linear | DEM toàn cầu |

---

# 6. Ocean Physical Parameters

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| BulkSST | Sea Surface Temperature | oceancolor_sst.cpd | Linear | Nhiệt độ mặt biển |
| sst | Sea Surface Temperature | oceancolor_sst.cpd | Linear | SST |
| SSS | Sea Surface Salinity | oceancolor_sss.cpd | Linear | Độ mặn biển |
| anc_SSS | Reference SSS | oceancolor_sss.cpd | Linear | Hiệu chỉnh độ mặn |
| scat_wind_speed | Wind speed | oceancolor_standard.cpd | Linear | Gió mặt biển |
| solz | Solar zenith angle | oceancolor_standard.cpd | Linear | Góc chiếu mặt trời |

---

# 7. Radiation and Reflectance

| ID | Đại lượng | Palette | Scale | Ứng dụng |
|---|---|---|---|---|
| Rrs_* | Remote sensing reflectance | oceancolor_standard.cpd | Linear | Phản xạ mặt nước |
| nLw_* | Normalized water leaving radiance | oceancolor_standard.cpd | Linear | Bức xạ rời mặt nước |
| rhos | Surface reflectance | gray_scale.cpd | Log | Phản xạ bề mặt |
| rhos_red | Red reflectance | standard_red.cpd | Log | Kênh đỏ |
| rhos_green | Green reflectance | standard_green.cpd | Log | Kênh xanh lá |
| rhos_blue | Blue reflectance | standard_blue.cpd | Log | Kênh xanh dương |

---

# 8. Quality / Diagnostic

| ID | Đại lượng | Kiểu dữ liệu | Ứng dụng |
|---|---|---|---|
| flag | Quality flags | Categorical | Mask dữ liệu lỗi |
| chisqr_giop | Chi-square | Continuous | Đánh giá chất lượng mô hình |
| rrsdiff_giop | Reflectance difference | Continuous | Kiểm tra sai khác |
| chlor_a_owterr | Chlorophyll error | Continuous | Sai số ước lượng |
| pixels | Number of pixels | Integer | Thống kê ảnh |
| scenes | Number of scenes | Integer | Thống kê dữ liệu |
| owt | Optical water type | Categorical | Phân loại nước |
| owtd | Dominant optical water type | Categorical | Nhóm nước quang học |

---

# 9. Liên hệ với SAR Visualization

| Kiểu dữ liệu | Scheme nên dùng | Ví dụ SAR |
|---|---|---|
| Continuous magnitude | Sequential | Amplitude, sigma0 |
| Signed difference | Diverging | Change detection |
| Angular value | Cyclic | Wrapped phase InSAR |
| Class label | Qualitative | Classification |
| DEM | Terrain/sequential | Terrain correction |
| Soil moisture | Sequential | SAR soil moisture inversion |

---

# 10. Nguyên tắc chọn palette

- Giá trị tăng một chiều → Sequential
- Có điểm 0 hoặc giá trị trung tâm → Diverging
- Nhãn phân loại → Qualitative
- Góc pha 0–360 độ → Cyclic
- Dữ liệu log range lớn → Log scale

