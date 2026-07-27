**Correlative Mode** của Profile Plot không chủ yếu dùng để xem giá trị SAR biến đổi theo một đường cắt như Classic Mode. Nó dùng để **đặt giá trị raster SAR cạnh dữ liệu đo ngoài thực địa tại các vị trí điểm tương ứng**, nhằm kiểm tra xem đại lượng SAR có phản ánh được đại lượng thực tế hay không.

Theo tài liệu SNAP, người dùng chọn:

- một lớp vector điểm làm **Point data source**;
    
- một trường số trong bảng thuộc tính làm **Data field**;
    
- một band raster đang hiển thị.
    

SNAP sau đó vẽ hai chuỗi dữ liệu:

1. giá trị band SAR tại vị trí của từng điểm vector;
    
2. giá trị đo ngoài thực địa lưu trong trường thuộc tính của điểm đó. ([STEP](https://step.esa.int/main/wp-content/help/versions/9.0.0/snap/org.esa.snap.snap.help/desktop/ProfilePlotDialog.html "step.esa.int"))
    

Ví dụ, mỗi điểm khảo sát có thể chứa:

```text
Point_ID | Longitude | Latitude | Soil_Moisture
P01      | ...       | ...      | 18.5
P02      | ...       | ...      | 24.2
P03      | ...       | ...      | 31.7
```

Khi chọn band `Sigma0_VV` và trường `Soil_Moisture`, SNAP lấy `Sigma0_VV` tại P01, P02, P03… rồi đặt cạnh các giá trị độ ẩm tương ứng.

## 1. Đánh giá quan hệ giữa backscatter và độ ẩm đất

Đây là một trong những ứng dụng điển hình nhất:

- raster: `Sigma0_VV`, `Sigma0_VH` hoặc `Gamma0`;
    
- dữ liệu điểm: độ ẩm đất đo tại hiện trường;
    
- mục tiêu: xem độ ẩm tăng có đi kèm sự thay đổi của backscatter hay không.
    

Ví dụ:

|Điểm|σ⁰ VV|Độ ẩm đất|
|---|--:|--:|
|P01|−16 dB|12%|
|P02|−13 dB|20%|
|P03|−10 dB|31%|

Nếu hai chuỗi thay đổi tương đối đồng dạng, band SAR có khả năng nhạy với độ ẩm trong điều kiện khảo sát đó. Các chiến dịch hiệu chuẩn–kiểm chứng của ESA cũng sử dụng cảm biến độ ẩm đất tại nhiều vị trí để đối chiếu với phép đo radar. ([Earth Online](https://earth.esa.int/eogateway/documents/20142/0/Hydrosoil-campaign-final-report.pdf?utm_source=chatgpt.com "HydroSoil Ground based SAR experiment for soil moisture ..."))

Tuy nhiên, không nên kết luận từ đồ thị rằng độ ẩm là nguyên nhân duy nhất, vì backscatter còn phụ thuộc độ nhám, thảm thực vật, góc tới và phân cực.

## 2. Kiểm chứng sinh khối và đặc tính rừng

Có thể so sánh:

- backscatter HV/VH;
    
- phân cực tổng hợp;
    
- entropy, alpha hoặc các thành phần phân rã PolSAR;
    
- độ cao tán rừng hoặc coherence;
    

với dữ liệu điểm:

- sinh khối trên mặt đất;
    
- đường kính thân cây;
    
- chiều cao cây;
    
- mật độ rừng;
    
- thể tích gỗ.
    

Ứng dụng thực tế là xác định đại lượng SAR nào nhạy nhất với sinh khối hoặc cấu trúc rừng, từ đó lựa chọn feature cho mô hình ước lượng.

Ví dụ:

```text
Raster:      Sigma0_HV
Point field: Biomass_t_ha
```

Nếu `Sigma0_HV` tăng theo sinh khối ở miền chưa bão hòa, nó có thể được dùng làm biến đầu vào cho mô hình hồi quy. Correlative Mode chỉ hỗ trợ quan sát và xuất dữ liệu; việc tính phương trình hồi quy hoặc hệ số tương quan chi tiết thường cần thực hiện tiếp trong Python, R hoặc Excel.

## 3. Kiểm chứng chuyển dịch InSAR bằng GNSS hoặc đo thủy chuẩn

Đây là ứng dụng rất quan trọng đối với sản phẩm InSAR:

- raster: vận tốc dịch chuyển hoặc displacement;
    
- vector point: vị trí trạm GNSS, mốc thủy chuẩn hoặc điểm đo mặt đất;
    
- data field: chuyển dịch đo tại điểm.
    

Ví dụ:

|Trạm|InSAR LOS|GNSS chiếu lên LOS|
|---|--:|--:|
|G01|−21 mm/năm|−19 mm/năm|
|G02|−8 mm/năm|−10 mm/năm|
|G03|3 mm/năm|2 mm/năm|

Correlative Mode giúp nhìn nhanh:

- sai lệch có tính hệ thống;
    
- một số trạm không khớp;
    
- vùng có nhiễu hoặc mất coherence;
    
- khả năng tồn tại offset trong sản phẩm InSAR.
    

ESA có các quy trình calibration/validation so sánh không gian giữa tốc độ biến dạng DInSAR và dữ liệu GNSS để đánh giá sai số dư. ([Earth Online](https://earth.esa.int/eogateway/documents/20142/2986799/Cal_Val_approach_for_DInSAR_Deformation_Rates_Products_using_GNSS_data_.pdf/89d1262a-ff81-3389-0f14-8e543656b07f?utm_source=chatgpt.com "Cal_Val approach for DInSAR Deformation Rates Products ..."))

Lưu ý rằng GNSS 3D cần được chiếu sang hướng nhìn LOS của SAR trước khi so sánh trực tiếp với displacement LOS.

## 4. Kiểm chứng bản đồ ngập lụt

Có thể dùng:

- raster: `Sigma0`, chênh lệch backscatter trước–sau lũ hoặc xác suất ngập;
    
- dữ liệu điểm: trạng thái thực địa `ngập/không ngập`, độ sâu nước hoặc mực nước đo được.
    

Ví dụ:

```text
Point_ID | Flooded | Water_depth_cm
P01      | 1       | 45
P02      | 0       | 0
P03      | 1       | 82
```

Ứng dụng:

- xem các điểm ngập có backscatter thấp hơn hay không;
    
- đánh giá ngưỡng phân loại nước;
    
- tìm các điểm bị phân loại sai;
    
- nhận biết ảnh hưởng của cây ngập hoặc đô thị ngập, nơi tín hiệu không nhất thiết tối như mặt nước trống.
    

Với trường dữ liệu nhị phân `0/1`, đồ thị chủ yếu giúp kiểm tra trực quan. Để đánh giá chính thức nên tính confusion matrix, precision, recall và F1 bên ngoài SNAP.

## 5. Phân tích cây trồng và dữ liệu nông nghiệp

Có thể đối chiếu chuỗi SAR với:

- chiều cao cây;
    
- độ ẩm cây;
    
- chỉ số diện tích lá;
    
- ngày gieo trồng;
    
- giai đoạn sinh trưởng;
    
- năng suất khảo sát;
    
- loại cây.
    

Ví dụ:

```text
Raster:      Sigma0_VH
Point field: Plant_height_cm
```

Điều này giúp kiểm tra xem phân cực VH có tăng cùng sự phát triển của tán cây hay không.

Với dữ liệu nhiều thời điểm, nên tạo các trường hoặc lớp điểm tương ứng đúng ngày chụp. Nếu dữ liệu thực địa đo ngày 1 nhưng ảnh SAR chụp ngày 15, kết quả có thể không đại diện cho cùng trạng thái bề mặt.

## 6. Đánh giá độ chính xác của phân loại ảnh SAR

Giả sử raster chứa:

- xác suất lớp rừng;
    
- xác suất lớp nước;
    
- chỉ số phục vụ phân loại;
    
- đầu ra của thuật toán phát hiện tàu hoặc công trình.
    

Vector điểm chứa nhãn kiểm chứng:

```text
Class_ID = 0, 1, 2...
```

Correlative Mode có thể giúp xem phân bố giá trị raster tại các điểm thuộc từng lớp. Ví dụ:

- điểm nước có `Sigma0_VV` thấp;
    
- điểm đô thị có `Sigma0_VV` cao;
    
- điểm rừng có `Sigma0_VH` tương đối cao.
    

Tuy nhiên, Correlative Mode không thay thế một công cụ validation hoàn chỉnh. Nó không tự động cung cấp confusion matrix hay độ chính xác tổng thể theo mô tả của tài liệu Profile Plot.

## 7. Kiểm chứng mô hình độ cao SAR

Có thể so sánh:

- raster: DEM tạo từ InSAR;
    
- vector point: điểm đo GNSS, điểm thủy chuẩn hoặc độ cao khảo sát;
    
- data field: cao độ tham chiếu.
    

Ứng dụng:

- phát hiện bias độ cao;
    
- tìm điểm có sai số lớn;
    
- đánh giá khu vực địa hình dốc;
    
- phát hiện ảnh hưởng của mất coherence hoặc lỗi unwrap.
    

Ví dụ:

```text
S1_DEM tại điểm: 125.4 m
Survey elevation: 121.8 m
Residual:          3.6 m
```

Correlative Mode giúp quan sát hai chuỗi. Sau khi xuất dữ liệu, có thể tính:

[  
e_i = h_{\mathrm{SAR},i}-h_{\mathrm{ref},i}  
]

và các chỉ số như bias, MAE hoặc RMSE.

## 8. Hiệu chuẩn radiometric bằng mục tiêu tham chiếu

Với các mục tiêu có độ phản xạ radar đã biết:

- corner reflector;
    
- transponder;
    
- vùng mục tiêu đồng nhất đã được đặc trưng;
    

có thể lưu giá trị tham chiếu vào thuộc tính vector và so sánh với giá trị raster đã hiệu chuẩn.

Ứng dụng:

- kiểm tra bias radiometric;
    
- so sánh kết quả giữa các cảnh;
    
- phát hiện sự không đồng đều theo range;
    
- kiểm tra sản phẩm `Beta0`, `Sigma0` hoặc `Gamma0`.
    

Tuy nhiên, đánh giá hiệu chuẩn chính xác thường cần tích phân năng lượng mục tiêu trong một ROI, không chỉ đọc một pixel duy nhất.

## 9. Phát hiện điểm ngoại lai trong dữ liệu

Correlative Mode đặc biệt hữu ích để phát hiện các điểm mà:

- dữ liệu SAR khác xa phép đo hiện trường;
    
- tọa độ điểm có thể bị sai;
    
- điểm nằm trên biên hai lớp phủ;
    
- điểm bị cloud/shadow trong dữ liệu bổ trợ quang học;
    
- ảnh SAR tại điểm bị layover, shadow hoặc mất coherence;
    
- đơn vị hoặc thang đo thuộc tính bị sai.
    

Ví dụ, hầu hết điểm có quan hệ hợp lý nhưng P17 sai rất lớn. Khi đó cần kiểm tra lại:

- tọa độ P17;
    
- ngày đo;
    
- giá trị thuộc tính;
    
- CRS;
    
- chất lượng raster tại khu vực đó.
    

## 10. So sánh SAR với dữ liệu từ cảm biến khác

“External data” không nhất thiết chỉ là số đo trực tiếp ngoài hiện trường. Nó có thể là dữ liệu được gắn vào vector điểm từ:

- trạm thời tiết;
    
- cảm biến IoT;
    
- phao đo biển;
    
- GNSS;
    
- lidar;
    
- khảo sát địa chất;
    
- dữ liệu quang học đã trích tại điểm;
    
- cơ sở dữ liệu kiểm kê rừng.
    

Ví dụ:

```text
SAR coherence ↔ mức độ hư hại khảo sát
SAR backscatter ↔ độ nhám bề mặt
InSAR velocity ↔ GNSS velocity
SAR-derived DEM ↔ surveyed elevation
VH/VV ratio ↔ crop height
```

## Vai trò của ROI Mask

SNAP cho phép sử dụng ROI mask để loại các điểm không thỏa điều kiện khỏi profile. ([STEP](https://step.esa.int/main/wp-content/help/versions/9.0.0/snap/org.esa.snap.snap.help/desktop/ProfilePlotDialog.html "step.esa.int"))

Ví dụ, khi nghiên cứu độ ẩm đất, có thể chỉ giữ:

```text
land_cover == bare_soil
```

và loại:

- rừng;
    
- khu đô thị;
    
- mặt nước;
    
- vùng layover/shadow.
    

Việc này quan trọng vì nếu trộn nhiều loại bề mặt, quan hệ SAR–độ ẩm có thể bị che khuất bởi ảnh hưởng của lớp phủ.

## Những giới hạn cần chú ý

### Speckle

Không nên luôn lấy đúng một pixel SAR tại một điểm đo. Một phép đo thực địa thường đại diện cho một vùng, trong khi một pixel SAR có speckle mạnh.

Nên cân nhắc:

- lấy trung bình cửa sổ 3×3, 5×5 hoặc ROI phù hợp;
    
- sử dụng dữ liệu multilook;
    
- bảo đảm cửa sổ không vượt qua ranh giới lớp phủ.
    

SNAP có thể biểu diễn độ lệch chuẩn quanh các điểm dưới dạng error bar trong cấu hình tương ứng. ([STEP](https://step.esa.int/main/wp-content/help/versions/9.0.0/snap/org.esa.snap.snap.help/desktop/ProfilePlotDialog.html "step.esa.int"))

### Khớp không gian

Cần kiểm tra:

- CRS;
    
- độ chính xác GPS;
    
- pixel spacing;
    
- terrain correction;
    
- sai số geolocation;
    
- điểm có nằm đúng loại địa vật hay không.
    

### Khớp thời gian

Ảnh và phép đo phải càng gần nhau về thời gian càng tốt, nhất là với:

- độ ẩm đất;
    
- ngập lụt;
    
- cây trồng;
    
- tuyết;
    
- mặt biển.
    

### Đơn vị và hướng biến đổi

Hai đường trên cùng đồ thị không nhất thiết có cùng đơn vị. Ví dụ:

- `Sigma0_VV`: dB;
    
- soil moisture: %.
    

Vì vậy, mục tiêu ban đầu là quan sát sự đồng biến, nghịch biến và các điểm bất thường, chứ không phải so sánh trực tiếp độ lớn của hai đường.

## So sánh nhanh với Classic Mode

|Classic Mode|Correlative Mode|
|---|---|
|Phân tích dọc theo line/polygon|Phân tích tại các điểm khảo sát|
|Trục X thường là vị trí trên đường|Mỗi vị trí tương ứng một điểm vector|
|Chỉ chủ yếu dùng dữ liệu raster|So sánh raster với thuộc tính vector|
|Tìm cạnh, peak, sidelobe, biến đổi không gian|Calibration, validation và ground truth|
|Hữu ích cho kiểm tra PSF và profile địa hình|Hữu ích cho soil moisture, GNSS, biomass, DEM và khảo sát thực địa|

**Tóm lại:** trong ảnh SAR, Correlative Mode là công cụ **đối chiếu dữ liệu SAR với ground truth hoặc dữ liệu tham chiếu tại các điểm địa lý**. Giá trị lớn nhất của nó nằm ở calibration/validation, lựa chọn feature, phát hiện ngoại lai và kiểm tra nhanh xem sản phẩm SAR có nhất quán với hiện tượng thực tế hay không. SNAP cũng cho phép chuyển sang Table View hoặc sao chép dữ liệu dạng bảng để phân tích định lượng tiếp trong phần mềm khác. ([STEP](https://step.esa.int/main/wp-content/help/versions/9.0.0/snap/org.esa.snap.snap.help/desktop/ProfilePlotDialog.html "step.esa.int"))