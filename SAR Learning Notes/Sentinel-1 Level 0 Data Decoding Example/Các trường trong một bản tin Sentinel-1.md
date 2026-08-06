

Với Sentinel-1C/D, ESA ban hành tài liệu SPPDU riêng nên không nên mặc định vị trí bit và bảng mã hoàn toàn giống A/B. 

Về tổng thể, một packet gồm:

$$
\text{Space Packet}
=
\underbrace{\text{Primary Header}}_{6\ \text{byte}}
+
\underbrace{\text{Secondary Header}}_{62\ \text{byte}}
+
\underbrace{\text{User Data}}_{\text{các mẫu radar đã nén}}
$$

Secondary Header chứa thời gian, thông tin cấu hình radar, trạng thái ăng-ten, loại tín hiệu và số lượng mẫu radar. 

---

## 1. Hai trường do phần mềm giải mã bổ sung

### Acquisition Chunk

Đây **không phải trường vật lý nằm trong SPPDU chính thức**.

Một *acquisition chunk* là một nhóm packet liên tiếp có cùng đặc tính thu nhận, thường là:

- cùng `Swath Number`;
- cùng `Number of Quads`;
- cùng loại dữ liệu hoặc cấu hình thu nhận phù hợp.

Decoder sử dụng khái niệm này để chia dòng packet Level-0 thành các đoạn dữ liệu có thể giải mã thành ma trận ảnh riêng biệt. 

### Packet Number

Thông thường đây là **chỉ số packet trong file hoặc trong bảng metadata do decoder tạo ra**, ví dụ packet thứ 0, 1, 2, 3...

Nó không phải trường CCSDS chính thức và cần phân biệt với:

| Trường | Ý nghĩa |
|---|---|
| `Packet Number` | Chỉ số do phần mềm giải mã gán |
| `Packet Sequence Count` | Bộ đếm 14 bit trong CCSDS Primary Header |
| `Space Packet Count` | Bộ đếm 32 bit của thiết bị SAR trong một datatake |

---

# 2. CCSDS Primary Header

## Packet Version Number

Cho biết phiên bản của chuẩn Space Packet.

Đối với Sentinel-1 SAR:

- độ dài: 3 bit;
- giá trị cố định: `000`.

## Packet Type

Phân biệt packet dữ liệu nguồn/telemetry với telecommand.

Đối với packet SAR Sentinel-1, trường này có giá trị cố định `0`.

## Secondary Header Flag

Cho biết packet có chứa Secondary Header hay không.

Đối với Sentinel-1:

- `1`: có Secondary Header;
- giá trị luôn bằng `1`.

## PID

`PID` là mã nhận dạng tiến trình hoặc nguồn sinh packet.

Đối với Sentinel-1 SAR:

- độ dài: 7 bit;
- giá trị: 65.

## PCAT

`PCAT` là mã phân loại packet do tiến trình đó sinh ra.

Đối với Sentinel-1 SAR:

- độ dài: 4 bit;
- giá trị: 12.

`PID` và `PCAT` hợp lại tạo thành APID:

$$
APID=(PID\ll4)\;|\;PCAT
$$

Với Sentinel-1 SAR:

$$
APID=65\times16+12=1052
$$

## Sequence Flags

Cho biết packet có phải một phần của chuỗi packet bị phân đoạn hay không.

Đối với Sentinel-1:

- giá trị `11`;
- nghĩa là packet độc lập, không bị phân đoạn.

## Packet Sequence Count

Bộ đếm packet 14 bit của CCSDS:

- bắt đầu từ 0;
- tăng 1 sau mỗi packet;
- sau 16383 sẽ quay lại 0.

Do đó:

$$
0\leq \text{Packet Sequence Count}\leq16383
$$

Trường này thường được dùng để phát hiện packet bị mất hoặc sai thứ tự.

## Packet Data Length

Cho biết độ dài phần dữ liệu sau Primary Header.

Theo quy tắc CCSDS:

$$
L_{\text{Packet Data Field}}
=
\text{Packet Data Length}+1
$$

Đơn vị là byte. Nó bao gồm Secondary Header và Radar Sample Data, không bao gồm Primary Header 6 byte. 

---

# 3. Thời gian của packet

## Coarse Time

Phần nguyên của thời gian packet, đơn vị giây.

Nó biểu diễn số giây nguyên theo hệ thời gian mà Sentinel-1 sử dụng, nominally dựa trên GPS time.

## Fine Time

Phần lẻ dưới một giây của timestamp.

Thời điểm packet được xác định bằng cách kết hợp:

$$
T_{\text{packet}}
=
T_{\text{coarse}}+T_{\text{fine}}
$$

Hai trường này cho biết thời điểm dữ liệu được tạo hoặc thu nhận với độ phân giải thời gian cao hơn một giây. 

---

# 4. Fixed Ancillary Data

## Sync

`Sync Marker` là một mẫu bit cố định dùng để:

- xác định đúng vị trí bắt đầu Secondary Header;
- phát hiện mất đồng bộ khi đọc file;
- kiểm tra parser có đang đọc đúng byte hay không.

Đối với Sentinel-1A/B, giá trị sync marker là:

```text
0x352EF853
```

## Data Take ID

Mã định danh của một **datatake**.

Một datatake là một hoạt động thu nhận SAR được lập kế hoạch, thực thi trên vệ tinh, lưu trữ và truyền xuống mặt đất.

Các packet thuộc cùng một lần thu nhận thường có cùng `Data Take ID`.

## ECC Number

Là mã xác định chế độ vận hành của thiết bị SAR đang được thực thi.

Nó giúp phân biệt các nhóm hoạt động như:

- Stripmap;
- Interferometric Wide Swath;
- Extra Wide Swath;
- Wave;
- calibration;
- noise measurement;
- instrument characterization.

Không nên chỉ dựa vào `ECC Number` để suy ra toàn bộ cấu hình; cần kết hợp thêm `Swath Number`, `Signal Type`, `Test Mode` và radar database.

## Test Mode

Cho biết thiết bị đang ở chế độ thử nghiệm hay chế độ vận hành thông thường.

Trường này thường được dùng cùng `ECC Number` để nhận biết packet:

- measurement;
- calibration;
- noise;
- instrument test hoặc characterization.

## Rx Channel ID

Xác định kênh thu đã tạo ra dữ liệu trong packet.

Trong chế độ phân cực kép, trường này đặc biệt quan trọng vì nó cho biết packet hiện tại thuộc kênh thu nào, ví dụ:

- kênh thu H;
- kênh thu V.

## Instrument Configuration ID

Mã nhận dạng bộ cấu hình thiết bị được lấy từ cơ sở dữ liệu cấu hình radar trên vệ tinh.

Một cấu hình có thể xác định:

- dạng xung;
- PRF/PRI;
- sampling window;
- beam;
- decimation filter;
- gain hoặc attenuation;
- chế độ nén dữ liệu.

Trường này cho phép hệ thống mặt đất xác định bộ tham số nào đã được sử dụng khi packet được tạo. 

---

# 5. Sub-commutated Ancillary Data

Do không đủ chỗ để truyền toàn bộ telemetry phụ trong mọi packet, Sentinel-1 truyền luân phiên từng word telemetry qua nhiều packet.

## Sub-commutated Ancillary Data Word Index

Cho biết word telemetry hiện tại thuộc vị trí nào trong chu kỳ sub-commutation.

- `0`: không có ancillary word hợp lệ;
- `1–64`: vị trí của word trong chu kỳ;
- sau 64 quay lại 1.

## Sub-commutated Ancillary Data Word

Giá trị 16 bit của ancillary word tương ứng với index.

Ý nghĩa của word phụ thuộc vào `Word Index`. Một chu kỳ hoàn chỉnh có thể chứa các nhóm thông tin như:

- vị trí và vận tốc vệ tinh;
- attitude;
- quaternion;
- angular rate;
- nhiệt độ các khối điện tử hoặc timing unit.

Muốn giải mã trường này phải dùng bảng ánh xạ:

$$
\text{Word Index}
\longrightarrow
\text{Loại telemetry}
$$

Không thể diễn giải giá trị `Word` nếu chỉ đọc riêng nó mà không có `Word Index`. 

---

# 6. Các bộ đếm của thiết bị

## Space Packet Count

Bộ đếm packet 32 bit do thiết bị SAR quản lý.

- bắt đầu từ đầu datatake;
- tăng sau mỗi packet được thiết bị sinh ra;
- phạm vi lớn hơn nhiều so với `Packet Sequence Count`.

Đây thường là bộ đếm hữu ích nhất để kiểm tra tính liên tục của packet trong một datatake dài.

## PRI Count

Bộ đếm số PRI kể từ khi hoạt động ECC hiện tại bắt đầu.

`PRI` là khoảng thời gian giữa hai lần phát xung liên tiếp. Vì thế `PRI Count` cho biết packet hiện tại liên quan tới PRI thứ bao nhiêu trong chuỗi vận hành radar.

Khoảng nhảy bất thường của `PRI Count` không phải lúc nào cũng có nghĩa là mất packet, vì một số PRI có thể không tạo packet dữ liệu. 

---

# 7. Cấu hình nén và thu tín hiệu radar

## Error Flag

Cho biết thiết bị thu đã phát hiện lỗi parity hoặc lỗi cấu hình trong thông tin SSB.

Thông thường:

- `0`: không phát hiện lỗi;
- `1`: packet có cấu hình không nhất quán và thường không nên dùng để xử lý mẫu radar.

## BAQ Mode

Xác định phương pháp lượng tử hóa và nén mẫu radar.

Các chế độ Sentinel-1A/B bao gồm:

| Mã | Chế độ |
|---:|---|
| 0 | Bypass, không BAQ |
| 3 | BAQ 3 bit |
| 4 | BAQ 4 bit |
| 5 | BAQ 5 bit |
| 12 | FDBAQ Mode 0 |
| 13 | FDBAQ Mode 1 |
| 14 | FDBAQ Mode 2 |

Trong đó:

- BAQ: Block Adaptive Quantization;
- FDBAQ: Flexible Dynamic Block Adaptive Quantization.

`BAQ Mode` quyết định cách diễn giải bitstream trong phần Radar Sample Data.

## BAQ Block Length

Số lượng mẫu radar được gom vào một block để tính tham số lượng tử hóa BAQ.

Trong mỗi block, thiết bị phân tích thống kê biên độ tín hiệu rồi chọn thang lượng tử hóa phù hợp. Vì vậy các block khác nhau có thể sử dụng threshold hoặc scale factor khác nhau.

## Range Decimation

Chọn bộ lọc và tỷ lệ decimation theo chiều range.

Đây không đơn giản chỉ là “bỏ mỗi $N$ mẫu”. Nó thường gồm:

1. lọc số để hạn chế aliasing;
2. giảm tần số lấy mẫu;
3. chọn băng thông thích hợp với chế độ SAR.

Trường này ảnh hưởng trực tiếp đến:

- sampling frequency hiệu dụng;
- khoảng cách giữa các mẫu range;
- số lượng mẫu trong cửa sổ thu.

## Rx Gain

Thông số điều khiển gain hoặc attenuation trên đường thu radar.

Trong tài liệu xử lý Sentinel-1, trường này thường được diễn giải như mức attenuation của receiver. Vì vậy cần áp dụng đúng conversion law trước khi hiểu nó dưới dạng dB.

## Tx Ramp Rate

Tốc độ biến thiên tần số của xung chirp phát, thường ký hiệu $K_r$.

Đơn vị vật lý là Hz/s.

## Tx Pulse Start Frequency: $f_0$

Tần số bắt đầu của xung chirp.

Đối với xung LFM:

$$
f(t)=f_0+K_rt
$$

với:

- $f_0$: Tx Pulse Start Frequency;
- $K_r$: Tx Ramp Rate;
- $t$: thời gian tính từ đầu xung.

## Tx Pulse Length

Độ dài thời gian của xung phát, thường ký hiệu $T_p$.

Xung được mô tả trong khoảng:

$$
0\leq t<T_p
$$

Băng thông chirp xấp xỉ:

$$
B\approx |K_r|T_p
$$

## Rank

Số khoảng PRI giữa xung phát và echo đang được thu.

Ví dụ, `Rank = 8` nghĩa là echo trong cửa sổ thu hiện tại có nguồn gốc từ xung được phát khoảng 8 PRI trước đó.

Rank rất quan trọng khi tìm đúng:

- Tx Pulse Number;
- beam phát;
- thông số chirp;

của xung đã tạo ra echo.

## PRI

`Pulse Repetition Interval`, khoảng thời gian giữa hai xung phát liên tiếp.

Quan hệ với PRF:

$$
PRF=\frac{1}{PRI}
$$

## SWST

`Sampling Window Start Time`.

Khoảng thời gian từ mốc PRI tới thời điểm bắt đầu lấy mẫu tín hiệu thu.

SWST quyết định vị trí gần của cửa sổ range.

## SWL

`Sampling Window Length`.

Độ dài thời gian của cửa sổ lấy mẫu. Nó ảnh hưởng đến số mẫu range:
$$
N_{\text{sample}}
\approx
f_s\,SWL
$$
trong đó $f_s$ là tần số lấy mẫu sau range decimation. 

---

# 8. SAS Setting Selector Bus

`SAS` là SAR Antenna Subsystem. Các trường này mô tả cấu hình ăng-ten trong PRI hiện tại.

## SAS SSB Flag

Cho biết phần SAS SSB đang được diễn giải theo cấu trúc nào:

- imaging/noise configuration;
- calibration configuration.

Trường này quyết định các bit tiếp theo cần được đọc như beam imaging hay beam calibration.

## Polarisation

Mô tả phân cực phát và cấu hình thu.

Các mã Sentinel-1A/B gồm:

| Mã | Ý nghĩa |
|---:|---|
| 0 | Chỉ phát H |
| 1 | Phát H, thu H |
| 2 | Phát H, thu V |
| 3 | Phát H, có các kênh thu V và H |
| 4 | Chỉ phát V |
| 5 | Phát V, thu H |
| 6 | Phát V, thu V |
| 7 | Phát V, có các kênh thu V và H |

Trong chế độ dual-polarisation, cần dùng thêm `Rx Channel ID` để xác định packet cụ thể chứa kênh thu H hay V.

## Temperature Compensation

Cho biết chức năng bù theo nhiệt độ của SAR Antenna Subsystem có đang được kích hoạt hay không.

Bù nhiệt được dùng để điều chỉnh cấu hình antenna frontend hoặc amplifier nhằm hạn chế biến thiên đáp ứng do nhiệt độ.

## Elevation Beam Address

Địa chỉ beam theo phương elevation được sử dụng trong PRI hiện tại.

Sentinel-1A/B có thể chọn một trong 16 địa chỉ beam elevation.

Beam elevation quyết định chủ yếu:

- góc incidence;
- vị trí swath theo chiều range;
- pattern chiếu sáng theo phương elevation.

## Azimuth Beam Address

Địa chỉ beam theo phương azimuth trong PRI hiện tại.

Có thể biểu diễn một trong 1024 địa chỉ beam azimuth.

Trường này đặc biệt quan trọng trong các chế độ có steering theo azimuth như TOPS, Wave hoặc các chuỗi calibration.

## SAS Test Mode

Cho biết SAR Antenna Subsystem đang thực hiện chế độ test hay calibration thông thường.

Trường này chỉ có ý nghĩa khi phần SAS SSB được diễn giải theo cấu trúc calibration.

## Cal Type

Xác định loại calibration đang thực hiện. Các loại được định nghĩa gồm:

| Mã | Loại calibration |
|---:|---|
| 0 | Tx Calibration |
| 1 | Rx Calibration |
| 2 | EPDN Calibration |
| 3 | TA Calibration |
| 4 | APDN Calibration |
| 7 | Tx H Calibration Isolation |

## Calibration Beam Address

Địa chỉ beam antenna được dùng cho calibration.

Tương tự azimuth beam address, trường này có thể chọn một trong 1024 beam calibration. 

---

# 9. SES Setting Selector Bus

`SES` là SAR Electronic Subsystem. Nhóm này mô tả tín hiệu điện tử được sinh hoặc thu trong PRI.

## Calibration Mode

Xác định loại chuỗi calibration hoặc characterization đang được sử dụng.

Các chế độ bao gồm:

| Mã | Ý nghĩa |
|---:|---|
| 0 | Interleaved internal calibration dựa trên PCC2 |
| 1 | Preamble/postamble internal calibration dựa trên PCC2 |
| 2 | PCC32 characterization |
| 3 | RF672 characterization |

## Tx Pulse Number

Địa chỉ của waveform phát được chọn trong chirp generator của SES.

Một điểm dễ nhầm:

- trường này mô tả pulse được phát trong PRI hiện tại;
- echo trong packet có thể do pulse phát từ `Rank` PRI trước.

Do đó, khi tìm waveform đã tạo ra một echo, cần quay lại packet hoặc cấu hình tại:

$$
PRI_{\text{Tx}}
=
PRI_{\text{Rx}}-\text{Rank}
$$

## Signal Type

Cho biết loại tín hiệu thực sự của PRI hiện tại.

|  Mã | Loại tín hiệu              |
| --: | -------------------------- |
|   0 | Echo                       |
|   1 | Noise                      |
|   8 | Tx Calibration             |
|   9 | Rx Calibration             |
|  10 | EPDN Calibration           |
|  11 | TA Calibration             |
|  12 | APDN Calibration           |
|  15 | Tx H Calibration Isolation |

Đây là trường quan trọng để không nhầm dữ liệu echo với noise hoặc calibration pulse.

## Swap Flag

Có thể xác nhận đây là một bit trạng thái `Swap` trong SES SSB.

Tuy nhiên, phần mô tả trường tôi đối chiếu không giải thích đầy đủ ánh xạ vật lý của thao tác swap. Vì vậy không nên tự coi nó là:

- đảo byte;
- đổi endian;
- đảo I/Q;
- hoặc đổi H/V;

nếu chưa đối chiếu đúng bảng cấu hình SES và phiên bản tài liệu tương ứng.

## Swath Number

Mã nhận dạng swath đang được sử dụng trong PRI hiện tại.

Swath Number liên kết đến một tập tham số cấu hình, ví dụ:

- Tx Pulse Number;
- Tx Pulse Length;
- Tx Pulse Start Frequency;
- Tx Ramp Rate;
- Range Decimation;
- SWST;
- SWL;
- PRI;
- Rank;
- Rx Gain;
- Elevation Beam Address.

Nó không chỉ là nhãn hiển thị như `IW1`, `IW2`, `IW3`; cần dùng bảng ánh xạ của mode/radar database để chuyển mã nhị phân thành tên swath.

## Number of Quads

Cho biết số nhóm mẫu radar được chứa trong packet.

Một quad gồm bốn giá trị:

$$
I_{\text{even}},\quad I_{\text{odd}},
\quad Q_{\text{even}},\quad Q_{\text{odd}}
$$

Tương đương với hai mẫu phức:

$$
s_{\text{even}}
=
I_{\text{even}}+jQ_{\text{even}}
$$

$$
s_{\text{odd}}
=
I_{\text{odd}}+jQ_{\text{odd}}
$$

Do đó, với $N_Q$ quads:

$$
N_{\text{complex samples}}=2N_Q
$$

`Number of Quads`, cùng `BAQ Mode` và `BAQ Block Length`, được dùng để xác định số bit cần đọc và kích thước dữ liệu sau giải nén. 

---

## Tóm tắt luồng diễn giải một packet

Khi decoder đọc một packet Sentinel-1, trình tự hợp lý là:

1. Dùng `Sync` để kiểm tra đồng bộ.
2. Đọc Primary Header để xác định kích thước và thứ tự packet.
3. Dùng `Coarse Time` và `Fine Time` để gán timestamp.
4. Dùng `Data Take ID`, `ECC Number` và `Swath Number` để xác định hoạt động thu.
5. Dùng `Signal Type` để phân biệt echo, noise và calibration.
6. Dùng `BAQ Mode`, `BAQ Block Length` và `Number of Quads` để giải nén mẫu.
7. Dùng `Rx Channel ID` và `Polarisation` để gán kênh HH, HV, VH hoặc VV.
8. Dùng `PRI Count`, `Rank`, `Tx Pulse Number`, `PRI`, `SWST` và `SWL` để tái tạo timeline phát–thu.
9. Dùng beam address để xác định beam antenna tương ứng.
10. Gom các packet liên tiếp tương thích thành một `Acquisition Chunk`.