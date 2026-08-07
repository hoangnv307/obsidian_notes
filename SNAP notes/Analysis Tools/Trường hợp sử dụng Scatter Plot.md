Trong **SNAP**, **Scatter Plot** và **Correlative Plot** đều là biểu đồ phân tán, nhưng mục đích hoàn toàn khác nhau:

- **Scatter Plot**: so sánh **hai band hoặc hai raster** trong cùng một sản phẩm ảnh.
    
- **Correlative Plot**: so sánh **ảnh raster với dữ liệu tham chiếu bên ngoài** (CSV, ship track, in situ, AIS, ...).
    

Scatter Plot chủ yếu dùng để **khám phá dữ liệu (exploratory analysis)**: tìm mối quan hệ giữa hai band, phát hiện cụm dữ liệu, ngưỡng phân loại hoặc điểm bất thường. ([ArcGIS](https://doc.arcgis.com/en/insights/2022.1/create/scatter-plot.htm?utm_source=chatgpt.com "Create and use a scatter plot—ArcGIS Insights | Documentation"))

---

# 1. Phân biệt nước và đất bằng hai band SAR

Đây là một trường hợp rất phổ biến.

Giả sử ảnh Sentinel-1 có:

- Band VV
    
- Band VH
    

Mỗi pixel sẽ tạo thành một điểm:

```text
x = VV

y = VH
```

Ví dụ:

Bạn sẽ thấy:

- nước tạo thành một cụm,
    
- đất tạo thành cụm khác,
    
- rừng nằm ở vùng khác nữa.
    

Điều này giúp bạn chọn ngưỡng hoặc xây dựng thuật toán phân loại.

---

# 2. Kiểm tra Radiometric Calibration

Giả sử bạn vừa viết xong module:

```text
Radiometric Calibration
```

và sinh ra hai band:

```text
Sigma0 VV

Sigma0 VH
```

Scatter Plot giúp trả lời:

> Hai phân cực này có mối quan hệ như mong đợi không?

Ví dụ

```text
VV
↑

         Rừng

     ●●●●

 Đất
  ●●

          Nước
             ●

----------------------→ VH
```

Nếu kết quả bất thường (ví dụ nước lại có VH rất lớn) thì có thể pipeline calibration đang có vấn đề.

---

# 3. Hỗ trợ phân loại (Classification)

Đây là cách rất nhiều thuật toán phân loại cổ điển hoạt động.

Giả sử Sentinel-2 có:

- NDVI
    
- NDWI
    

Scatter Plot:

```text
NDWI

↑

Nước

●●●●●

         Cây

            ●●●●

Đất

 ●●

------------------------→ NDVI
```

Bạn sẽ thấy các cụm tách biệt rõ ràng.

Từ đó có thể:

- chọn ngưỡng,
    
- tạo rule-based classifier,
    
- đánh giá feature có phân biệt lớp tốt hay không.
    

---

# 4. Kiểm tra mối quan hệ giữa hai band

Ví dụ Sentinel-2:

```text
Band 4 (Red)

Band 8 (NIR)
```

Scatter Plot:

```text
Red

↑

●

 ●

   ●

      ●

          ●

----------------------→ NIR
```

Nếu các điểm gần nằm trên một đường thẳng:

→ Hai band tương quan mạnh.

Nếu hoàn toàn ngẫu nhiên:

→ Hai band mang thông tin khác nhau.

Điều này hữu ích khi lựa chọn đặc trưng (feature selection) trước khi huấn luyện mô hình.

---

# 5. Phát hiện Outlier

Giả sử 99% pixel tạo thành cụm:

```text
●●●●●●●●●
```

nhưng có vài điểm:

```text
                     ●

                ●
```

Các điểm này có thể là:

- tàu,
    
- công trình,
    
- lỗi cảm biến,
    
- nhiễu.
    

Scatter Plot giúp nhận ra chúng ngay lập tức. ([ArcGIS](https://doc.arcgis.com/en/insights/2022.1/create/scatter-plot.htm?utm_source=chatgpt.com "Create and use a scatter plot—ArcGIS Insights | Documentation"))

---

# 6. Kiểm tra kết quả lọc Speckle

Ví dụ

Band A:

```text
Original
```

Band B:

```text
Lee Filter
```

Scatter Plot:

```text
Original

↑

      ●
    ●
  ●
 ●

----------------------→ Filtered
```

Nếu:

- gần đường chéo
    

→ bộ lọc giữ tốt giá trị gốc.

Nếu:

- phân tán mạnh
    

→ bộ lọc làm thay đổi dữ liệu khá nhiều.

---

# 7. So sánh hai thời điểm

Ví dụ

Sentinel-1:

```text
Before Flood

↓

After Flood
```

Scatter Plot:

```text
Before

↑

●●●●●●●

        ●

           ●

----------------------→ After
```

Những điểm lệch xa khỏi đường chéo là nơi đã thay đổi nhiều.

Ví dụ:

- ngập lụt,
    
- cháy rừng,
    
- xây dựng.
    

---

# 8. Kiểm tra hai thuật toán xử lý

Giả sử bạn có hai module:

```text
Terrain Correction A

Terrain Correction B
```

Scatter Plot:

```text
Algorithm A

↑

●

 ●

  ●

----------------------→ Algorithm B
```

Nếu gần đường

```text
y=x
```

→ Hai thuật toán cho kết quả gần giống.

---

# Trong SNAP, Scatter Plot khác Correlative Plot như thế nào?

|Scatter Plot|Correlative Plot|
|---|---|
|Band A vs Band B|Raster vs dữ liệu tham chiếu|
|Hai trục đều lấy từ ảnh|Một trục lấy từ ảnh, một trục lấy từ CSV, Shapefile, AIS, dữ liệu thực địa...|
|Phân tích mối quan hệ giữa các band|Kiểm chứng ảnh bằng số liệu ngoài hiện trường|
|Chủ yếu dùng để khám phá dữ liệu và hỗ trợ phân loại|Chủ yếu dùng để đánh giá, hiệu chỉnh và kiểm định thuật toán|

---

## Với lĩnh vực SAR mà bạn đang làm

Nếu bạn phát triển một phần mềm tương tự SNAP, mình nghĩ **Scatter Plot** sẽ đặc biệt hữu ích ở các bước sau:

- **Đánh giá Radiometric Calibration:** so sánh Sigma0 VV và Sigma0 VH để kiểm tra tính nhất quán.
    
- **Phân tích đặc trưng cho phát hiện mục tiêu:** ví dụ Intensity vs Local Contrast, Intensity vs Texture, hoặc các cặp đặc trưng khác để xem tàu, biển và đất có tạo thành các cụm riêng hay không.
    
- **Đánh giá các bộ lọc speckle:** so sánh giá trị pixel trước và sau lọc.
    
- **Phân tích thay đổi (Change Detection):** so sánh giá trị backscatter của cùng một khu vực ở hai thời điểm để nhanh chóng nhận ra các vùng biến động.
    

Những trường hợp này đều tận dụng đúng thế mạnh của Scatter Plot: **hiển thị mối quan hệ giữa hai biến số của cùng một tập dữ liệu**, giúp bạn phát hiện cụm dữ liệu, xu hướng và các điểm bất thường trước khi xây dựng hoặc tinh chỉnh thuật toán.

**VV vs VH**

Ví dụ các cụm dữ liệu trong ảnh SAR.

|type|vh|vv|
|---|---|---|
|Water|-30|-22|
|Water|-29|-21|
|Water|-31|-23|
|Land|-15|-10|
|Land|-13|-8|
|Land|-14|-9|
|Forest|-10|-7|
|Forest|-9|-6|