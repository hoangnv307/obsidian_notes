# 1. Toán tử ba ngôi 
- Chỉ giữ lại khoảng dữ liệu góc tới mong muốn
```
incidenceAngle >= 30.0 && incidenceAngle <= 45.0 ? Sigma0_VV : NaN
```
- Tạo ảnh nhị phân phát hiện vùng tán xạ mạnh
```
Sigma0_VV_dB > -8.0 ? 1 : 0
```

- Chọn giữa 2 band, kết quả là giá trị lớn hơn giữa 2 phân cực của pixel
```
Sigma0_VV > Sigma0_VH ? Sigma0_VV : Sigma0_VH
```
# 2. Toán tử logic
- Giả sử có 2 mask `layover_mask` và `shadow_mask
```
layover_mask || shadow_mask
```
- Phát hiện vùng sáng trên 2 band
```
Sigma0_VV_dB > -8.0 || Sigma0_VH_dB > -15.0
```
- Kết hợp 2 vùng địa lý: 
```
sea_mask || coastal_mask
```

- Tìm vùng tối trên biển
```
sea_mask && Sigma0_VV_dB < -18.0
```