
Đối với ảnh vệ tinh SAR, **Profile Plot (Classic Mode)** trong SNAP là một công cụ tuy đơn giản nhưng lại rất hữu ích trong quá trình **phân tích chất lượng ảnh, kiểm tra thuật toán xử lý và phân tích đối tượng**. Nó không chỉ dùng để "vẽ biểu đồ cường độ theo một đường cắt" mà còn giúp đánh giá rất nhiều đặc tính của dữ liệu SAR.

Theo tài liệu của SNAP, Classic Mode hoạt động bằng cách lấy giá trị pixel của band đang chọn dọc theo một đường (transect/polyline) hoặc biên của một hình học vector rồi biểu diễn thành đồ thị. Trục X là khoảng cách (số pixel hoặc chiều dài đường), trục Y là giá trị pixel của band. SNAP cũng cho phép hiển thị độ lệch chuẩn (standard deviation) của vùng lân cận quanh từng pixel. citeturn0search0turn0search1

Dưới đây là những ứng dụng thực tế nhất trong xử lý ảnh SAR.

---

# 1. Phân tích phản xạ radar theo mặt cắt (Radar Backscatter Profile)

Đây là ứng dụng phổ biến nhất.

Ví dụ vẽ một đường cắt ngang qua

```
Biển ---- Bờ ---- Thành phố
```

ta sẽ thu được

```
Sigma0

 ^
 |                  ████
 |               ███
 |            ██
 |___________█________________> Distance
     biển   bờ   đô thị
```

Ý nghĩa

- mặt nước → phản xạ rất thấp
- bãi cát → tăng nhẹ
- khu đô thị → phản xạ rất mạnh

Nhờ vậy có thể

- xác định ranh giới
- kiểm tra sự chuyển tiếp
- phân tích đặc điểm từng loại địa vật

---

# 2. Đánh giá độ phân giải không gian (Spatial Resolution)

Đây là ứng dụng rất quan trọng khi phát triển thuật toán SAR.

Ví dụ chọn một đường đi qua

- góc nhà
- cạnh đường
- reflector

Profile sẽ cho biết cạnh chuyển từ

```
0
0
0
255
255
255
```

hay

```
0
20
60
120
180
230
255
```

Cạnh càng dốc

→ ảnh càng sắc nét

Có thể dùng để

- so sánh thuật toán RDA
- so sánh PFA
- BPA
- thuật toán autofocus

---

# 3. Đánh giá Point Spread Function (PSF)

Đây gần như là ứng dụng tiêu chuẩn trong SAR.

Nếu đi qua một Corner Reflector

```
           *
```

Profile sẽ có dạng

```
      ^
      |
      |          /\
      |         /  \
______|________/____\__________
```

Từ profile có thể đo

- Peak value
- 3 dB Width
- PSLR
- ISLR

Đây đều là các chỉ số đánh giá chất lượng ảnh SAR.

---

# 4. Kiểm tra Side Lobe

Sau Range Compression hoặc Azimuth Compression

Corner reflector thường có

```
      |
      |        *
      |      * | *
      |    *   |   *
______|___*____|____*__________
```

Hai bên đỉnh chính chính là

Side Lobes

Profile Plot giúp

- đo biên độ sidelobe
- kiểm tra cửa sổ Kaiser/Hamming/Taylor
- đánh giá matched filter

---

# 5. Kiểm tra Speckle

Nếu chọn một vùng đồng nhất

```
Ruộng

████████████
```

Profile sẽ không phẳng

mà dao động

```
   ^
   | /\   /\ /\     /\
   |/  \_/  \  \/\_/  \
```

Biên độ dao động

→ mức speckle

Sau khi áp dụng

- Lee
- Frost
- Gamma MAP

Profile sẽ mượt hơn

→ dễ đánh giá hiệu quả lọc.

---

# 6. So sánh trước và sau xử lý

Ví dụ

Raw

```
^^^^^^^^^^^^^^^^^^^^^^
```

Sau Calibration

```
______________________
```

Sau Radiometric Terrain Correction

```
_____/¯¯¯¯¯¯\_________
```

Chỉ cần vẽ cùng một transect là thấy ngay

- nhiễu giảm
- tín hiệu thay đổi
- biên sắc hơn hay không

---

# 7. Kiểm tra hiệu quả Radiometric Calibration

Sau khi chuyển DN

→ Sigma0

→ Gamma0

→ Beta0

Profile trên cùng một khu vực sẽ thay đổi theo quy luật mong muốn.

Ví dụ

```
DN

███

↓

Sigma0

██████

↓

Gamma0

████████
```

Có thể kiểm tra

- calibration đúng chưa
- scale đúng chưa

---

# 8. Đánh giá Noise Floor

Ví dụ chọn đường hoàn toàn nằm trên biển

```
~~~~~~~~~~~~~~~~~~~~~~
```

Profile

```
0.01
0.03
0.02
0.04
0.01
```

Nếu xuất hiện

```
0.2
0.4
0.7
```

thì có thể

- Thermal Noise
- Stripe
- Processing Artifact

---

# 9. Kiểm tra Seam giữa các Burst (TOPS/ScanSAR)

Nếu profile đi qua chỗ ghép burst

```
Burst A | Burst B
```

Profile lý tưởng

```
______________
```

Nếu có lỗi

```
_________
        \
         \______
```

hoặc

```
___________
           |
___________|
```

→ có thể phát hiện

- radiometric jump
- seam

---

# 10. Phân tích kết cấu bề mặt

Ví dụ profile qua

```
Rừng
```

sẽ dao động mạnh

```
/\/\/\/\/\/\/\/\
```

Qua

```
Đường băng
```

```
________________
```

Qua

```
Khu dân cư
```

```
/\/\_/\/\/\/\/\__
```

Nhờ vậy có thể

- phân biệt texture
- hỗ trợ phân loại

---

# 11. Kiểm tra Registration giữa hai ảnh

Nếu hai ảnh SAR đã đăng ký

Profile của chúng gần như trùng nhau.

Nếu lệch

```
Image A

     /\
____/  \____

Image B

       /\
______/  \____
```

=> registration sai.

---

# 12. Phân tích Shadow và Layover

Ví dụ profile qua một tòa nhà

```
Radar →

███████
```

sẽ có dạng

```
Layover

      /\
_____/  \______

Shadow

______________
              \
               \_____
```

Nhờ đó xác định

- shadow
- layover
- foreshortening

---

# 13. Kiểm tra chất lượng thuật toán RCMC

Nếu RCMC chưa chính xác

đỉnh reflector sẽ bị kéo dài

```
      /\
     /  \
____/    \____
```

Nếu tốt

```
      /\
_____/  \_____
```

Đây là cách rất nhiều kỹ sư SAR dùng để đánh giá thuật toán.

---

# 14. Kiểm tra Autofocus

Một reflector

Trước autofocus

```
      /----\
_____/      \_____
```

Sau autofocus

```
       /\
______/  \______
```

Đỉnh hẹp hơn

→ ảnh nét hơn.

---

# 15. Debug phần mềm SAR (rất phù hợp với dự án của bạn)

Đối với phần mềm SAR desktop mà bạn đang phát triển, Profile Plot là một công cụ gần như bắt buộc vì nó cho phép so sánh trực tiếp kết quả xử lý giữa các bước.

Bạn có thể dùng nó để:

- so sánh profile giữa ảnh của phần mềm và SNAP trên cùng một transect;
- kiểm tra ảnh sau từng bước của chuỗi xử lý (Range Compression → RCMC → Azimuth Compression → Multi-looking → Calibration → Speckle Filter);
- đánh giá các chỉ tiêu như Peak, 3 dB Width, PSLR và ISLR trên các corner reflector;
- phát hiện các lỗi như clipping, sai scale, seam giữa các burst hoặc các dải nhiễu (striping);
- xuất dữ liệu profile sang bảng (Table View hoặc Copy Data to Clipboard) để so sánh định lượng bằng Excel hoặc Python. SNAP cũng cung cấp bảng gồm chỉ số pixel, tọa độ ảnh, tọa độ địa lý, giá trị trung bình trong cửa sổ, độ lệch chuẩn và các trường dữ liệu liên quan. citeturn0search4turn0search0

---

## Tóm tắt

| Ứng dụng | Mức độ sử dụng trong SAR |
|---------|--------------------------|
| Kiểm tra biên địa vật | ⭐⭐⭐⭐⭐ |
| Đánh giá độ phân giải | ⭐⭐⭐⭐⭐ |
| Đo PSF (PSLR, ISLR, 3 dB Width) | ⭐⭐⭐⭐⭐ |
| Kiểm tra Speckle | ⭐⭐⭐⭐ |
| So sánh trước/sau xử lý | ⭐⭐⭐⭐⭐ |
| Kiểm tra Calibration | ⭐⭐⭐⭐ |
| Phát hiện Seam | ⭐⭐⭐⭐ |
| Kiểm tra Registration | ⭐⭐⭐⭐ |
| Phân tích Shadow/Layover | ⭐⭐⭐⭐ |
| Debug thuật toán SAR (RDA, CSA, PFA, BPA...) | ⭐⭐⭐⭐⭐ |

Nếu bạn đang phát triển một phần mềm xử lý ảnh SAR tương tự SNAP, tôi khuyến nghị triển khai **Profile Plot** như một công cụ phân tích kỹ thuật thay vì chỉ là tiện ích hiển thị. Các phần mềm SAR chuyên nghiệp (SNAP, ENVI SARscape, GAMMA, ISCE...) đều sử dụng profile để kiểm chứng chất lượng thuật toán và dữ liệu trong quá trình phát triển và đánh giá sản phẩm.