Histogram trong ảnh SAR biểu diễn **phân bố số lượng pixel theo giá trị biên độ, cường độ hoặc hệ số tán xạ**. Nó không cho biết pixel nằm ở đâu, nhưng rất hữu ích để hiểu đặc tính thống kê của toàn ảnh hoặc một vùng ROI.

## 1. Chọn dải hiển thị và Auto Contrast

Đây là ứng dụng thường gặp nhất trong phần mềm xem ảnh SAR.

Ảnh SAR thường có:

- phần lớn pixel mang giá trị thấp hoặc trung bình;
    
- một số ít pixel rất sáng do tàu, nhà cao tầng hoặc corner reflector;
    
- dải động rất rộng.
    

Nếu ánh xạ trực tiếp từ giá trị nhỏ nhất đến lớn nhất:

```text
Min ------------------------------------------ Max
nhiều pixel tối                        vài pixel cực sáng
```

thì phần lớn ảnh có thể bị tối, vì vài pixel cực sáng kéo giãn toàn bộ thang hiển thị.

Histogram được dùng để chọn các percentile, ví dụ:

- cắt 1% số pixel thấp nhất;
    
- cắt 4% số pixel cao nhất;
    
- ánh xạ phần còn lại vào dải xám 0–255.
    

Trong SNAP, cơ chế tương tự được dùng để thiết lập dải hiển thị ban đầu. Đây là lý do histogram liên quan trực tiếp đến độ sáng và độ tương phản của quicklook.

**Lưu ý:** cắt 1% và 4% nghĩa là cắt theo **số pixel tích lũy**, không phải bỏ một số lượng bin cố định.

---

## 2. Phân biệt nước và đất trong phát hiện ngập

Mặt nước phẳng thường có backscatter thấp, còn đất, thực vật và đô thị thường cao hơn.

Histogram của một khu vực có cả nước và đất có thể xuất hiện hai vùng phân bố:

```text
Pixel count
   ^
   |    /\                 /\
   |   /  \               /  \
   |__/____\_____________/____\____> σ⁰
      nước                  đất
```

Có thể chọn một ngưỡng:
$$\sigma^0 < T \Rightarrow \text{nước}  $$
Ứng dụng:
- lập bản đồ ngập;
- xác định mặt nước;
- theo dõi hồ chứa;
- phân đoạn vùng biển và đất liền.

Tuy nhiên, bóng radar, đường nhựa hoặc đất rất nhẵn cũng có thể có giá trị thấp giống nước. Vì vậy histogram thường cần kết hợp với:
- dữ liệu địa hình;
- connectivity;
- mask shadow;
- ảnh trước sự kiện;
- phân cực khác.
---

## 3. Chọn ngưỡng phát hiện tàu hoặc mục tiêu sáng

Tàu trên biển thường tạo ra các pixel sáng trên nền biển tối.
Histogram có thể cho thấy:
- một cụm lớn tương ứng clutter biển;
- một phần đuôi nhỏ ở phía giá trị cao tương ứng mục tiêu sáng.

```text
Pixel count
   ^
   |       /\
   |      /  \____
   |_____/________\__________________> intensity
         clutter          đuôi mục tiêu
```

Có thể chọn ngưỡng sơ bộ:

[  
I > T \Rightarrow \text{ứng viên mục tiêu}  
]

Ứng dụng:

- phát hiện tàu;
    
- phát hiện phương tiện;
    
- tìm công trình kim loại;
    
- phát hiện corner reflector.
    

Tuy nhiên, histogram toàn cục chỉ thích hợp để khảo sát ban đầu. Thuật toán phát hiện mục tiêu SAR thực tế thường dùng **CFAR**, vì mức clutter thay đổi theo không gian.

---

## 4. Đánh giá hiệu quả lọc speckle

Chọn một ROI đồng nhất, chẳng hạn:

- ruộng đồng nhất;
    
- mặt biển;
    
- vùng rừng tương đối đồng nhất.
    

So sánh histogram trước và sau lọc:

### Trước lọc

- phân bố rộng;
    
- độ lệch chuẩn lớn;
    
- nhiều giá trị cực thấp và cực cao.
    

### Sau lọc

- phân bố hẹp hơn;
    
- phương sai giảm;
    
- giá trị trung bình nên được giữ gần như ổn định.
    

Ứng dụng để so sánh:

- Lee;
    
- Frost;
    
- Gamma MAP;
    
- Refined Lee;
    
- multilooking.
    

Không nên chỉ kết luận bộ lọc tốt vì histogram hẹp hơn. Lọc quá mạnh cũng làm mất cạnh và mục tiêu nhỏ. Cần kiểm tra thêm profile, texture và khả năng bảo toàn cạnh.

---

## 5. Kiểm tra radiometric calibration

Histogram giúp so sánh ảnh trước và sau khi chuyển từ DN sang:

- Beta Nought;
    
- Sigma Nought;
    
- Gamma Nought;
    
- giá trị dB.
    

Ví dụ, sau calibration:
$$\sigma^0_{\mathrm{dB}}=10\log_{10}(\sigma^0)  $$
Histogram sẽ thay đổi về trục giá trị và hình dạng.

Có thể dùng để phát hiện:
- sai scale factor;
- quên chuyển sang dB;
- dùng nhầm (10\log_{10}) và (20\log_{10});
- dữ liệu âm hoặc `NoData` bị xử lý sai;
- calibration tạo ra giá trị bất thường.

Ví dụ, nếu histogram Sigma Nought dB của một cảnh đất liền tập trung tại những giá trị phi lý, đó là dấu hiệu cần kiểm tra lại luồng calibration và metadata.

---

## 6. So sánh hai ảnh SAR trước và sau một sự kiện

Histogram có thể được dùng để đánh giá thay đổi phân bố toàn cảnh hoặc trong cùng một ROI.

Ví dụ:

- trước và sau lũ;
- trước và sau cháy rừng;
- trước và sau thu hoạch;
- trước và sau đô thị hóa;
- hai thời điểm khác nhau của băng tuyết.

Nếu vùng bị ngập tăng lên, histogram có thể dịch về phía backscatter thấp hơn.
Nếu khu vực xây dựng mới tăng, phần đuôi giá trị cao có thể tăng lên.
Histogram chỉ cho biết **phân bố đã thay đổi**, không cho biết vị trí thay đổi. Vì vậy nên kết hợp với:
- ảnh difference;
- log-ratio;
- change map;
- ROI theo khu vực.
---
## 7. Phân tích các loại lớp phủ bề mặt

Có thể vẽ histogram riêng cho từng ROI:

- nước;
    
- rừng;
    
- ruộng;
    
- đất trống;
    
- đô thị.
    

Ví dụ:

|Lớp phủ|Đặc điểm histogram thường gặp|
|---|---|
|Nước phẳng|Tập trung ở backscatter thấp|
|Rừng|Giá trị trung bình, phân bố tương đối rộng|
|Đô thị|Giá trị cao và đuôi sáng dài|
|Đất trống|Phụ thuộc mạnh vào độ ẩm và độ nhám|
|Ruộng|Thay đổi theo mùa vụ, hướng hàng cây và độ ẩm|

Ứng dụng:

- lựa chọn feature;
    
- đặt ngưỡng phân loại;
    
- kiểm tra khả năng phân tách lớp;
    
- đánh giá phân cực VV, VH, HH hoặc HV.
    

Nếu histogram của hai lớp chồng lấn nhiều, chỉ dùng một band sẽ khó phân loại chúng chính xác.

---

## 8. Lựa chọn band hoặc phân cực thích hợp

Có thể so sánh histogram của cùng một ROI trên:

- VV;
    
- VH;
    
- VV/VH;
    
- span;
    
- entropy;
    
- coherence.
    

Ví dụ, nếu histogram của rừng và đất trống chồng lấn mạnh ở VV nhưng tách tốt hơn ở VH, thì VH có thể là feature hữu ích hơn cho bài toán đó.

Histogram giúp trả lời nhanh:

- band nào có độ tương phản tốt hơn;
    
- phân cực nào phân biệt lớp tốt hơn;
    
- dữ liệu tuyến tính hay dB phù hợp hơn cho hiển thị hoặc threshold.
    

---

## 9. Phát hiện saturation, clipping và lỗi kiểu dữ liệu

Histogram là công cụ QA rất hiệu quả.

### Clipping ở giá trị cực đại

Nếu có một cột rất lớn ở giá trị tối đa:

```text
0 ... 65534 65535
             █████
```

có thể dữ liệu đã bị saturation hoặc bị ép kiểu khi chuyển sang `uint16`.

### Clipping ở giá trị 0

Một đỉnh rất lớn tại 0 có thể do:

- NoData bị gán bằng 0;
    
- giá trị âm bị ép về 0;
    
- vùng ngoài swath;
    
- lỗi đọc raster;
    
- lỗi xử lý tile.
    

### Quantization

Histogram có các khoảng trống đều nhau có thể cho thấy dữ liệu bị lượng tử hóa quá thô hoặc bị chuyển đổi kiểu dữ liệu không phù hợp.

---

## 10. Phát hiện NoData và pixel không hợp lệ

Histogram giúp nhận biết các giá trị đặc biệt:

- 0;
    
- −9999;
    
- giá trị tối thiểu của kiểu số;
    
- NaN;
    
- Infinity.
    

Nếu không áp dụng valid-pixel mask, các giá trị này có thể:

- làm sai mean và standard deviation;
    
- làm lệch percentile;
    
- khiến auto contrast hiển thị sai;
    
- tạo một đỉnh giả rất lớn trong histogram.
    

Trong phần mềm SAR, histogram nên được tính sau khi áp dụng:

```text
Valid mask
NoData mask
NaN/Inf filtering
ROI mask
```

---

## 11. Ước lượng nền nhiễu và noise floor

Với ROI trên biển hoặc vùng có tín hiệu rất thấp, histogram giúp quan sát:

- trung tâm phân bố nền;
    
- độ rộng của nhiễu;
    
- đuôi bất thường;
    
- sự khác nhau theo range.
    

Ứng dụng:

- đánh giá thermal noise removal;
    
- tìm noise floor;
    
- phát hiện noise ramp;
    
- so sánh các vùng near range và far range;
    
- kiểm tra dải nhiễu ở biên swath.
    

Nếu histogram vùng biển dịch đáng kể giữa các sub-swath, có thể tồn tại sai khác về noise hoặc calibration.

---

## 12. Phát hiện seam giữa burst hoặc sub-swath

Vẽ histogram riêng cho:

- burst A và burst B;
    
- IW1, IW2 và IW3;
    
- hai tile ở hai phía đường ghép.
    

Nếu cùng một loại bề mặt nhưng histogram bị dịch:

```text
Burst A: tập trung quanh −14 dB
Burst B: tập trung quanh −11 dB
```

thì có thể tồn tại:

- radiometric discontinuity;
    
- normalization chưa đúng;
    
- lỗi calibration;
    
- lỗi ghép burst;
    
- khác biệt noise floor.
    

Histogram đặc biệt hữu ích khi seam khó nhìn bằng mắt do ảnh đang được auto stretch riêng.

---

## 13. Đánh giá ảnh hưởng của terrain correction

Có thể so sánh histogram:

- trước terrain correction;
    
- sau terrain correction;
    
- Sigma Nought và Gamma Nought;
    
- sườn hướng về radar và sườn quay khỏi radar.
    

Mục tiêu là kiểm tra:

- phân bố radiometric có thay đổi hợp lý không;
    
- có xuất hiện nhiều pixel bất thường không;
    
- layover và shadow đã được mask đúng chưa;
    
- nội suy có tạo quá nhiều giá trị 0 hoặc NoData không.
    

---

## 14. Kiểm tra kết quả multilooking

Multilooking làm giảm speckle bằng cách trung bình nhiều look.

Histogram sau multilook thường:

- ít phân tán hơn;
    
- giảm đuôi cực trị;
    
- ổn định hơn trong vùng đồng nhất.
    

Có thể dùng histogram kết hợp với **Equivalent Number of Looks — ENL**:

[  
ENL \approx \frac{\mu^2}{\sigma^2}  
]

trong đó (\mu) và (\sigma) được tính trên một ROI đồng nhất ở miền intensity tuyến tính.

Histogram giúp xác nhận trực quan sự giảm phân tán, còn ENL cung cấp chỉ số định lượng.

---

## 15. Kiểm tra đầu ra từng bước của bộ xử lý SAR

Trong quá trình phát triển bộ xử lý ảnh SAR, histogram có thể dùng tại từng công đoạn:

- raw magnitude;
    
- sau range compression;
    
- sau RCMC;
    
- sau azimuth compression;
    
- sau detection;
    
- sau multilooking;
    
- sau calibration;
    
- sau chuyển dB;
    
- sau terrain correction.
    

Nó giúp phát hiện nhanh:

- giá trị bị tràn;
    
- scale tăng hoặc giảm bất thường;
    
- ảnh chứa quá nhiều zero;
    
- clipping;
    
- NaN;
    
- sai phép tính magnitude;
    
- sai chuẩn hóa FFT;
    
- sai kiểu dữ liệu đầu ra.
    

Ví dụ, nếu sau mỗi FFT/IFFT năng lượng tăng theo kích thước FFT, cần kiểm tra quy ước normalization.

## Histogram toàn ảnh và histogram ROI

Hai loại này phục vụ mục đích khác nhau:

|Histogram toàn ảnh|Histogram ROI|
|---|---|
|Tốt cho auto contrast|Tốt cho phân tích lớp phủ|
|Phát hiện clipping toàn cục|Đánh giá speckle vùng đồng nhất|
|Kiểm tra phân bố sản phẩm|Chọn threshold nước/đất|
|Có thể bị trộn nhiều loại địa vật|Có ý nghĩa vật lý rõ hơn|
|Phù hợp cho quicklook|Phù hợp cho calibration và validation|

Trong SAR, **histogram ROI thường có giá trị phân tích cao hơn histogram toàn cảnh**, vì histogram toàn cảnh trộn lẫn nước, rừng, đô thị, địa hình và shadow.

## Những lưu ý quan trọng khi triển khai

### Chọn miền giá trị phù hợp

Histogram của cùng một ảnh trong các miền sau sẽ rất khác:

- amplitude tuyến tính;
    
- intensity tuyến tính;
    
- amplitude dB;
    
- intensity/backscatter dB.
    

Phải ghi rõ tên band và đơn vị trên trục X.

### Dùng bin hợp lý

Quá ít bin:

- mất chi tiết phân bố;
    
- khó thấy nhiều mode.
    

Quá nhiều bin:

- histogram nhiễu;
    
- tốn tài nguyên;
    
- không cần thiết cho quicklook.
    

Khoảng 256–512 bin thường hợp lý cho hiển thị tổng quát, nhưng không phải giá trị tối ưu cho mọi bài toán.

### Không dùng min/max thô cho hiển thị

Một vài mục tiêu cực sáng có thể làm min–max stretch vô dụng. Percentile stretch thường ổn định hơn.

### Phải dùng valid mask

Không đưa NoData, NaN, Inf và vùng ngoài ảnh vào thống kê.

## Tóm tắt

Các trường hợp thực tế quan trọng nhất của histogram trong ảnh SAR là:

1. thiết lập auto contrast và dải hiển thị;
    
2. chọn ngưỡng nước–đất và phát hiện ngập;
    
3. khảo sát ngưỡng phát hiện mục tiêu sáng;
    
4. đánh giá giảm speckle và multilooking;
    
5. kiểm tra radiometric calibration;
    
6. so sánh phân bố trước–sau sự kiện;
    
7. phân tích lớp phủ và lựa chọn phân cực;
    
8. phát hiện clipping, saturation, NoData và lỗi dữ liệu;
    
9. đánh giá thermal noise và seam giữa burst;
    
10. debug từng bước của chuỗi xử lý SAR.
    

Đối với phần mềm của bạn, histogram nên được xem là cả **công cụ hiển thị**, **công cụ phân tích thống kê ROI** và **công cụ kiểm soát chất lượng dữ liệu**, chứ không chỉ là biểu đồ phục vụ chỉnh độ sáng.