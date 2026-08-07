Đây thực ra là một câu hỏi rất hay, vì **Correlative Plot không phải là công cụ mà người dùng phổ thông hay dùng**, mà chủ yếu dành cho **người phát triển thuật toán** hoặc **nhà nghiên cứu**. Trong thực tế, nó thường được dùng để **kiểm chứng (validation)** hoặc **đánh giá (evaluation)** kết quả xử lý ảnh.

Dưới đây là một số ví dụ thực tế.

---

# Ví dụ 1: Đánh giá thuật toán ước lượng tốc độ gió từ Sentinel-1 (phổ biến nhất)

Giả sử bạn xây dựng thuật toán:

```
Ảnh Sentinel-1
      ↓
Sigma0
      ↓
CMOD5
      ↓
Wind Speed
```

Sau khi chạy xong, ảnh của bạn chứa một band:

```
Wind Speed (m/s)
```

Đồng thời, trong ngày vệ tinh bay qua có các **phao khí tượng** ngoài biển đo được:

|Lat|Lon|Wind đo thực tế|
|---|---|---|
|...|...|5.1|
|...|...|7.3|
|...|...|8.5|

Bạn import bảng này vào SNAP.

Correlative Plot sẽ tạo các cặp dữ liệu:

|Wind đo thực tế|Wind từ Sentinel|
|---|---|
|5.1|5.4|
|7.3|7.1|
|8.5|8.8|

Sau đó vẽ scatter plot.

Nếu các điểm nằm gần đường (y=x), điều đó cho thấy thuật toán ước lượng gió hoạt động tốt.

---

# Ví dụ 2: Đánh giá NDVI với số liệu đo ngoài đồng

Bạn có ảnh Sentinel-2:

```
NDVI
```

Đồng thời nhóm khảo sát đi ra đồng ruộng và đo:

```
LAI
Biomass
Độ che phủ
```

tại 200 vị trí GPS.

Ví dụ:

|Lat|Lon|LAI|
|---|---|---|
|...|...|1.3|
|...|...|2.5|
|...|...|4.0|

Correlative Plot sẽ ghép:

```
LAI đo thực tế

↓

NDVI của pixel tương ứng
```

để xem NDVI phản ánh mức độ sinh trưởng của cây trồng tốt đến đâu.

---

# Ví dụ 3: So sánh độ cao DEM

Bạn có

- DEM từ vệ tinh
    
- GPS RTK khảo sát địa hình
    

GPS:

|Lat|Lon|Elevation|
|---|---|---|
|...|...|15.2|
|...|...|18.6|
|...|...|22.8|

Correlative Plot:

```
GPS Elevation
        vs
DEM Elevation
```

Nếu:

```
R² = 0.99
```

thì DEM rất chính xác.

---

# Ví dụ 4: Đánh giá thuật toán phát hiện tàu (gần với lĩnh vực SAR của bạn)

Giả sử bạn viết một thuật toán:

```
SAR

↓

Ship Detector

↓

Ship Score
```

Bạn có dữ liệu AIS:

```
Tàu thật sự ở đâu
```

Sau đó lấy tại vị trí từng tàu:

```
AIS Position

↓

Ship Score
```

Scatter plot có thể là:

```
Ship Score

↑
|
|       •
|      •
|   •
|
|_____________________>

Ship Length
```

hoặc

```
Ship Score

vs

Radar Cross Section
```

để xem tàu lớn có điểm số cao hơn hay không.

---

# Ví dụ 5: Theo dõi tàu khảo sát biển

Đây chính là ví dụ được SNAP ghi trong tài liệu:

```
Ship Track
```

Giả sử tàu chạy như sau:

```
----------------------->

đo

Sea Surface Temperature

độ mặn

chlorophyll
```

Mỗi giây tàu ghi:

```
GPS

Temperature

Salinity

Chlorophyll
```

Đồng thời Sentinel-3 bay qua.

Correlative Plot sẽ ghép:

```
Nhiệt độ nước từ tàu

↓

Nhiệt độ từ Sentinel
```

hoặc

```
Chlorophyll đo thực địa

↓

Chlorophyll tính từ vệ tinh
```

---

# Ví dụ 6: Trong phát triển sản phẩm SAR

Đây là ví dụ mình nghĩ sẽ gần với công việc của bạn nhất.

Giả sử bạn đang phát triển module:

```
Radiometric Calibration
```

Bạn muốn kiểm tra:

```
Calibration mới

↓

Sigma0
```

có đúng chưa.

Bạn có một tập các **Corner Reflector** hoặc **Radar Calibration Target** ngoài thực địa.

Mỗi reflector có:

```
Radar Cross Section chuẩn
```

Correlative Plot sẽ so sánh:

```
Sigma0 sau Calibration

↓

Radar Cross Section chuẩn
```

Nếu kết quả tốt:

- Regression gần đường thẳng.
    
- R² cao.
    
- Sai số nhỏ.
    

---

# Tại sao không dùng Histogram?

Điểm khác biệt quan trọng là:

**Histogram** chỉ cho biết phân bố giá trị của **một tập dữ liệu**.

Ví dụ:

```
Sigma0

↓

Histogram
```

Bạn biết ảnh có nhiều pixel sáng hay tối.

Nhưng bạn **không biết**:

> Pixel sáng đó có tương ứng với giá trị thực ngoài hiện trường hay không?

Correlative Plot trả lời chính câu hỏi đó bằng cách ghép **hai nguồn dữ liệu** tại cùng vị trí địa lý.

---

## Tóm lại

Correlative Plot được dùng khi bạn có:

- **Một ảnh raster** (Sigma0, NDVI, SST, DEM, Intensity, Chlorophyll, ...)
    
- **Một bộ dữ liệu tham chiếu** (CSV, Shapefile, GPS, AIS, phao đo, ship track, khảo sát thực địa...)
    

và muốn trả lời câu hỏi:

> **Giá trị trên ảnh vệ tinh có phản ánh đúng giá trị đo ngoài thực địa hoặc dữ liệu tham chiếu hay không?**

Trong lĩnh vực **SAR** mà bạn đang tìm hiểu, các ứng dụng phổ biến nhất là:

1. Kiểm chứng thuật toán **Radiometric Calibration**.
    
2. Đánh giá mô hình **ước lượng tham số địa vật** (gió, độ ẩm đất, v.v.) từ ảnh SAR.
    
3. So sánh **giá trị backscatter** với số liệu đo thực địa.
    
