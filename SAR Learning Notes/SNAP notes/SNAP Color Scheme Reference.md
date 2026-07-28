

Nguồn runtime đã đối chiếu:

- `snap-ui/src/main/resources/auxdata/color_schemes/color_palette_schemes.xml`
- `snap-ui/src/main/resources/auxdata/color_schemes/color_palette_scheme_lookup.xml`
- `snap-ui/src/main/resources/auxdata/color_palettes/*.{cpd,cpt}`

> [!IMPORTANT]
> `color_palette_schemes.txt` là nguồn chuyển đổi cũ. SNAP hiện đọc
> `color_palette_schemes.xml`. Danh mục hiện có **71 Scheme hoạt động**;
> `chlor_a_oc` và `256PtLinear` đang bị comment nên không được tính.

## 1. Scheme là gì?

Một Scheme là preset hoàn chỉnh gồm:

```text
Palette + Range (MIN/MAX) + Linear/Log scaling
```

Scheme không bị khóa theo loại cảm biến. SNAP vẫn cho phép chọn một Scheme
quang học cho band SAR, nhưng range và ý nghĩa vật lý thường không phù hợp.
Bảng dưới phân loại theo nguồn dữ liệu mà Scheme được thiết kế để biểu diễn,
không phải giới hạn kỹ thuật của giao diện.

## 2. Ý nghĩa các trường

| Trường | Ý nghĩa |
|---|---|
| `name` / ID | Định danh Scheme, được file lookup liên kết với tên band |
| `VERBOSE_NAME` | Tên dài hiển thị trong Scheme selector |
| `MIN` / `MAX` | Khoảng giá trị mặc định được ánh xạ lên palette |
| `LOG_SCALE` | `true`: ánh xạ logarithmic; `false`: ánh xạ tuyến tính |
| `STANDARD_FILENAME` | Palette chuẩn `.cpd` hoặc `.cpt` |
| `UNIVERSAL_FILENAME` | Palette thay thế hỗ trợ khả năng phân biệt màu |
| `COLORBAR_TITLE` | Tiêu đề dự kiến của thanh màu |
| `COLORBAR_LABELS` | Các mốc nhãn trên thanh màu, phân cách bằng dấu phẩy |
| `DESCRIPTION` | Mô tả Scheme; hiện được dùng làm tooltip |
| `PRIMARY` | Xếp Scheme vào nhóm Primary trong selector |

> [!NOTE]
> `COLORBAR_TITLE` và `COLORBAR_LABELS` được SNAP nạp vào `ColorSchemeInfo`,
> nhưng mã Java hiện tại chưa sử dụng chúng để vẽ color bar.

## 3. Quy ước cột “Sensor / satellite data”

| Giá trị | Ý nghĩa |
|---|---|
| **Optical** | Đại lượng đo hoặc suy ra từ ảnh quang học |
| **Thermal optical** | Đại lượng chủ yếu lấy từ kênh hồng ngoại nhiệt |
| **Microwave radiometer (not SAR)** | Dữ liệu từ radiometer vi ba; không phải ảnh SAR |
| **Scatterometer radar (not SAR)** | Dữ liệu radar đo gió diện rộng; không phải imaging SAR |
| **SAR & Optical** | Raster phụ trợ/sản phẩm dẫn xuất có thể dùng cùng cả hai |

> [!IMPORTANT]
> SNAP mặc định **không có Scheme chuyên cho band SAR** như `Intensity_HV`,
> `Amplitude_VV`, `Sigma0_HH`, `Gamma0_VH`, coherence hoặc phase InSAR.

## 4. Ocean optical properties, atmosphere and biology

| ID | Đại lượng / ứng dụng | Palette | Scale | Sensor / satellite data |
|---|---|---|---|---|
| `absorption` | Hệ số hấp thụ trong nước | `oceancolor_standard.cpd` | Log | Optical |
| `adg_s` | Độ dốc phổ hấp thụ gelbstoff và detritus | `oceancolor_standard.cpd` | Linear | Optical |
| `angstrom` | Hệ số Ångström của aerosol | `oceancolor_standard.cpd` | Linear | Optical |
| `aot` | Aerosol optical thickness | `oceancolor_standard.cpd` | Linear | Optical |
| `bb` | Tổng hệ số tán xạ ngược trong nước | `oceancolor_standard.cpd` | Log | Optical |
| `bbp` | Tán xạ ngược bởi hạt | `oceancolor_standard.cpd` | Log | Optical |
| `bbp_s` | Độ dốc phổ tán xạ ngược bởi hạt | `oceancolor_standard.cpd` | Linear | Optical |
| `bbp_giop` | Tán xạ ngược bởi hạt từ mô hình GIOP | `oceancolor_standard.cpd` | Log | Optical |
| `bbw` | Tán xạ ngược của nước biển | `oceancolor_standard.cpd` | Log | Optical |
| `BSi` | Biogenic silica | `oceancolor_standard.cpd` | Log | Optical |
| `calcite` | Nồng độ calcite | `oceancolor_standard.cpd` | Log | Optical |
| `cdom_index` | Chỉ số CDOM | `oceancolor_standard.cpd` | Linear | Optical |
| `chlor_a` | Nồng độ chlorophyll-a | `oceancolor_standard.cpd` | Log | Optical |
| `chlor_a_owterr` | Sai số tương đối chlorophyll-a của thuật toán OWT | `oceancolor_standard.cpd` | Linear | Optical |
| `chlor_a_bluegreen` | Chlorophyll với palette xanh lam–xanh lục | `chlor_blue_green.cpd` | Log | Optical |
| `chlor_a_uni` | Chlorophyll với palette universal xanh lam–đỏ | `universal_bluered.cpd` | Log | Optical |
| `chlor_a_uni_bg` | Chlorophyll với palette universal xanh lam–xanh lục | `universal_bluegreen.cpd` | Log | Optical |
| `CI` | Cyanobacteria Index | `oceancolor_standard.cpd` | Log | Optical |
| `epsilon` | Epsilon của hiệu chỉnh khí quyển/aerosol | `oceancolor_standard.cpd` | Linear | Optical |
| `flh` | Fluorescence Line Height | `oceancolor_standard.cpd` | Linear | Optical |
| `fqy` | Fluorescence Quantum Yield | `oceancolor_standard.cpd` | Linear | Optical |
| `ipar` | Instantaneous Photosynthetically Available Radiation | `oceancolor_standard.cpd` | Linear | Optical |
| `Kd_lee` | Hệ số suy giảm ánh sáng khuếch tán | `oceancolor_standard.cpd` | Log | Optical |
| `Kd_490` | Hệ số suy giảm ánh sáng tại 490 nm | `oceancolor_standard.cpd` | Log | Optical |
| `nLw_lt_430` | Water-leaving radiance chuẩn hóa, dưới 430 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `nLw_430_479` | Water-leaving radiance chuẩn hóa, 430–479 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `nLw_480_529` | Water-leaving radiance chuẩn hóa, 480–529 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `nLw_530_599` | Water-leaving radiance chuẩn hóa, 530–599 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `nLw_ge_600` | Water-leaving radiance chuẩn hóa, từ 600 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `nw` | Chiết suất nước biển | `oceancolor_standard.cpd` | Linear | Optical |
| `owt` | Optical Water Type chuẩn hóa | `oceancolor_standard.cpd` | Linear | Optical |
| `owtd` | Optical Water Type chiếm ưu thế | `oceancolor_standard.cpd` | Linear | Optical |
| `par` | Photosynthetically Available Radiation | `oceancolor_standard.cpd` | Linear | Optical |
| `pic` | Particulate Inorganic Carbon | `oceancolor_standard.cpd` | Log | Optical |
| `poc` | Particulate Organic Carbon | `oceancolor_standard.cpd` | Log | Optical |
| `rhos` | Phản xạ bề mặt | `gray_scale.cpd` | Log | Optical |
| `rhos_red` | Phản xạ bề mặt kênh đỏ | `standard_red.cpd` | Log | Optical |
| `rhos_green` | Phản xạ bề mặt kênh xanh lục | `standard_green.cpd` | Log | Optical |
| `rhos_blue` | Phản xạ bề mặt kênh xanh lam | `standard_blue.cpd` | Log | Optical |
| `Rrs_lt_430` | Remote-sensing reflectance dưới 430 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `Rrs_430_459` | Remote-sensing reflectance 430–459 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `Rrs_460_499` | Remote-sensing reflectance 460–499 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `Rrs_ge_500` | Remote-sensing reflectance từ 500 nm | `oceancolor_standard.cpd` | Linear | Optical |
| `rrsdiff_giop` | Sai khác tương đối Rrs của mô hình GIOP | `oceancolor_standard.cpd` | Linear | Optical |
| `solz` | Góc thiên đỉnh Mặt Trời | `oceancolor_standard.cpd` | Linear | Optical |
| `Zeu` | Độ sâu tầng ưu quang | `oceancolor_zphotic.cpd` | Linear | Optical |
| `Zhl_morel` | Độ sâu heated layer theo Morel | `oceancolor_zphotic.cpd` | Linear | Optical |
| `Zhl` | Độ sâu heated layer | `oceancolor_zphotic.cpd` | Linear | Optical |
| `Zp_10_lee` | Độ sâu còn 10% ánh sáng theo Lee | `oceancolor_zphotic.cpd` | Linear | Optical |
| `Zp_50_lee` | Độ sâu còn 50% ánh sáng theo Lee | `oceancolor_zphotic.cpd` | Linear | Optical |
| `Zsd` | Độ sâu Secchi | `oceancolor_zphotic.cpd` | Linear | Optical |

## 5. Vegetation indices

| ID | Đại lượng / ứng dụng | Palette | Scale | Sensor / satellite data |
|---|---|---|---|---|
| `NDVI` | Normalized Difference Vegetation Index | `oceancolor_ndvi.cpd` | Linear | Optical |
| `ndvi` | Normalized Difference Vegetation Index | `oceancolor_ndvi.cpd` | Linear | Optical |
| `EVI` | Enhanced Vegetation Index | `oceancolor_ndvi.cpd` | Linear | Optical |
| `evi` | Enhanced Vegetation Index | `oceancolor_ndvi.cpd` | Linear | Optical |

Các cặp viết hoa/viết thường có range hơi khác nhau trong cấu hình SNAP; vì
vậy chúng được giữ thành các ID riêng thay vì gộp.

## 6. Ocean physical and microwave-derived parameters

| ID | Đại lượng / ứng dụng | Palette | Scale | Sensor / satellite data |
|---|---|---|---|---|
| `BulkSST` | Nhiệt độ bề mặt biển VIIRS-N | `oceancolor_sst.cpd` | Linear | Thermal optical |
| `sst` | Nhiệt độ bề mặt biển | `oceancolor_sst.cpd` | Linear | Mainly thermal optical |
| `SSS` | Độ mặn bề mặt biển | `oceancolor_sss.cpd` | Linear | Microwave radiometer (not SAR) |
| `anc_SSS` | SSS tham chiếu/phụ trợ | `oceancolor_sss.cpd` | Linear | Auxiliary / microwave radiometer |
| `rad_sm` | Soil moisture | `sm.cpd` | Linear | Microwave radiometer (not SAR) |
| `scat_wind_speed` | Tốc độ gió từ scatterometer | `oceancolor_standard.cpd` | Linear | Scatterometer radar (not SAR) |

`rad_sm` không nên được mô tả mặc định là “SAR soil moisture inversion”:
tên và nguồn Scheme hướng tới sản phẩm radiometer soil moisture. Một raster
soil-moisture suy ra từ SAR vẫn có thể dùng palette này nếu đơn vị/range khớp.

## 7. Terrain, elevation and bathymetry

| ID | Đại lượng / ứng dụng | Palette | Scale | Sensor / satellite data |
|---|---|---|---|---|
| `elev` | Cao độ | `oceancolor_standard.cpd` | Linear | SAR & Optical (DEM/derived) |
| `elevation` | Cao độ so với mực nước biển | `gray_scale.cpdthe` ⚠️ | Linear | SAR & Optical (DEM/derived) |
| `bathymetry` | Độ sâu đáy biển | `smooth_inv_blue.cpd` | Log | SAR & Optical (auxiliary/derived) |
| `bathymetry_deep` | Độ sâu đáy biển, palette CPT | `deep.cpt` | Log | SAR & Optical (auxiliary/derived) |
| `topography` | Địa hình mặt đất | `smooth_green.cpd` | Log | SAR & Optical (DEM/derived) |
| `topography_ETOP` | Địa hình theo ETOPO1 | `topography.cpd` | Linear | SAR & Optical (DEM/derived) |

> [!WARNING]
> `elevation` trong XML thực sự trỏ tới `gray_scale.cpdthe`. File này không
> tồn tại và có vẻ là lỗi gõ của SNAP; Scheme sẽ bị đánh dấu disabled.
> Không tự sửa thành `gray_scale.cpd` trong bảng tham chiếu vì như vậy không
> còn phản ánh đúng cấu hình runtime.

## 8. Quality, diagnostic and binned-product metadata

| ID | Đại lượng / ứng dụng | Palette | Scale | Sensor / satellite data |
|---|---|---|---|---|
| `flag` | Quality flags | `oceancolor_standard.cpd` | Linear | Mainly optical products |
| `chisqr_giop` | Chi-square trên mỗi bậc tự do của GIOP | `oceancolor_standard.cpd` | Linear | Optical |
| `pixels` | Số pixel trong mỗi bin | `oceancolor_standard.cpd` | Linear | Mainly optical binned products |
| `scenes` | Số scene trong mỗi bin | `oceancolor_standard.cpd` | Linear | Mainly optical binned products |

> [!WARNING]
> Cấu hình SNAP đặt chuỗi `Aerosol optical thickness` vào
> `COLORBAR_LABELS` của Scheme `flag`. Đây có vẻ là dữ liệu sai cột, không
> phải danh sách nhãn số hợp lệ.

## 9. Liên hệ với SAR visualization

Không có Scheme tích hợp nào trong danh mục trên được thiết kế trực tiếp cho
backscatter, coherence hoặc phase SAR. Với SAR nên chọn palette/range theo
kiểu raster:

| Kiểu dữ liệu SAR | Palette phù hợp | Scaling |
|---|---|---|
| `Intensity_*`, `Amplitude_*` tuyến tính | Grayscale hoặc sequential | Có thể dùng log để nén dynamic range |
| `Sigma0_*`, `Gamma0_*`, `Beta0_*` tuyến tính | Grayscale hoặc sequential | Có thể dùng log |
| Backscatter đã ở dB | Grayscale hoặc sequential | Linear; không log lần nữa |
| Coherence `[0,1]` | Sequential | Linear |
| Sai khác có tâm tại 0 | Diverging | Linear |
| Wrapped phase `[-π,π]` hoặc `[0,2π]` | Cyclic | Linear |
| Nhãn phân loại | Qualitative/discrete | Không nội suy |
| DEM dùng trong terrain correction | Terrain/sequential | Linear hoặc theo DEM |
| Soil moisture suy ra từ SAR | Sequential, có thể thử `rad_sm` | Chỉ dùng nếu range/đơn vị khớp |

## 10. Nguyên tắc chọn palette

- Giá trị tăng một chiều → sequential.
- Có điểm 0 hoặc giá trị trung tâm có ý nghĩa → diverging.
- Nhãn phân loại → qualitative/discrete.
- Góc pha tuần hoàn → cyclic.
- Dữ liệu tuyến tính có dynamic range lớn → có thể log.
- Dữ liệu đã ở dB → linear, không áp dụng log lần thứ hai.

## 11. Kết quả kiểm tra tính đúng đắn và đầy đủ

- [x] Đổi nguồn chính từ tên sai `color_palettes_schemes.txt` sang file
  runtime `color_palette_schemes.xml`.
- [x] Bổ sung ý nghĩa `VERBOSE_NAME`, `UNIVERSAL_FILENAME`, `PRIMARY` và
  `COLORBAR_TITLE`.
- [x] Liệt kê đủ 71 Scheme đang hoạt động trong XML.
- [x] Không tính `chlor_a_oc` và `256PtLinear` vì đang bị comment.
- [x] Tách `Rrs_*` và `nLw_*` thành đúng các ID thực tế.
- [x] Bổ sung các Scheme còn thiếu như `bbp_s`, `cdom_index`, `flh`, `fqy`,
  `ipar`, `Kd_*`, `rad_sm`, `Z*`, `bathymetry_deep` và `CI`.
- [x] Sửa nhận định “SAR soil moisture inversion” thành ghi chú có điều kiện.
- [x] Ghi nhận đúng lỗi `gray_scale.cpdthe` và dữ liệu bất thường của `flag`.
- [x] Thêm phân loại loại ảnh/cảm biến cho từng Scheme.
