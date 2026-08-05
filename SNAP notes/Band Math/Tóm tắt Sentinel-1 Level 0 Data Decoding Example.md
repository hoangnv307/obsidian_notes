
Tài liệu tham khảo:
[1] Sentinel-1 Level 0 Data Decoding Example: [Link bài viết](https://github.com/Rich-Hall/sentinel1Level0DecodingDemo/blob/main/sentinel1Level0DecodingDemo.ipynb)
[2] ESA2022_Sentinel-1 Level 1 Detailed Algorithm Definition
[3] SAR Space Packet Protocol Data Unit
[4] Các tài liệu của Sentinel-1: [Link](https://sentiwiki.copernicus.eu/web/document-library#DocumentLibrary-SENTINEL-1DocumentsLibrary-S1-Documents)
# 1. Imports và setup 
- Sản phẩm được xét trong bài, chế độ stripmap:
`sao_paulo/s1a-s6-raw-s-vv-20251226t214356-20251226t214427-062491-07d496.dat`

- Lưu lại đường dẫn tới file .dat (file chứa các gói bản tin gửi xuống mặt đất)
# 2. Extract File Metadata
- Một packet thông thường sẽ ba gồm đầu ra của radar instrument tương ứng của một radar echo. 
- Các packets cũng có thể bao gồm các loại dữ liệu khác, ví dụ background noise measurements for instrument calibration. 
- Tác giả sẽ trích metadata từ mỗi packet trên một subset (vì thao tác trên cả ảnh sẽ tốn thời gian).
- Một packet sẽ bao gồm các trường như sau:
	- Acquistion chunk
	- Packet Number: Index của mỗi packet
	- Packet Version Number
	- Packet Type
	- Secondary Header Flag
	- PID
	- PCAT
	- Sequence Flags
	- Packet Sequence Count
	- Packet Data Length
	- Coarse Time
	- Fine Time
	- Sync
	- Data Take ID
	- ECC Number
	- Test Mode
	- Rx Channel ID
	- Instrument Configuration ID
	- Sub-commutated Ancilliary Data Word Index
	- Sub-commutated Ancilliary Data Word
	- Space Packet Count
	- PRI Count
	- Error Flag
	- BAQ Mode
	- BAQ Block Length
	- Range Decimation
	- Rx Gain
	- Tx Ramp Rate
	- Tx Pulse Start Frequency
	- Tx Pulse Length
	- Rank
	- PRI
	- SWST
	- SWL
	- SAS SSB Flag
	- Polarisation
	- Temperature Compensation
	- Elevation Beam Address
	- Azimuth Beam Address
	- SAS Test Mode
	- Cal Type
	- Calibration Beam Address
	- Calibration Mode
	- Tx Pulse Number
	- Signal Type
	- Swap Flag
	- Swath Number
	- Number of Quads
Tổng cộng ma trận được trích có 46 cột, 50706 hàng tương ứng với số bản tin.
- Ngoài ra, thông tin về quỹ đạo của vệ tinh còn được đính kèm vào các bản tin với tần số thấp hơn, vì vậy tác giả cần thêm một bước để trích xuất trường thông tin này:
	- X-axis position ECEF
	- Y-axis position ECEF
	- Z-axis position ECEF
	- X-axis velocity ECEF
	- Y-axis velocity ECEF
	- Z-axis velocity ECEF
	- POD Solution Data Timestamp
	- Q0 Attitude Quaternion
	- Q1 Attitude Quaternion
	- Q2 Attitude Quaternion
	- Q3 Attitude Quaternion
	- Omega-X Angular Rate
	- Omega-Y Angular Rate
	- Omega-Z Angular Rate
	- Attitude Data Timestamp
Tổng cộng 15 trường với 762 hàng (số quỹ đạo được đính kèm bản tin) tương ứng.

# 3. Extract data
## 3.1 Select Packets to Process
- Tác giả ở dây chỉ muốn trích xuất những bản tin mang thông tin về radar echoes (dựa vào trường dữ liệu `Signal Type.
- Nhắc lại về khái niệm `Acquistion chunk`, một acquistition chunk là một single continuous segment của SAR acquistition (echoes, noise, calibration, etc.),  where the instrument configuration is fixed and the radar records one uninterrupted sequence.  ==It is one stable, coherent block of data for a given signal type that can be processed as a unit.==
```python
# iter_chunks_matching can be passed any constant param you see in the below lists.
# Here, we're using it to iterate over all the chunks which have an echo signal type (as opposed to a noise measurement or a calibration operation)
for chunk in l0file.iter_chunks_matching(signal_type=sentinel1decoder.enums.SignalType.ECHO):
    print(f"Chunk {chunk}:")
    for key, val in l0file.get_acquisition_chunk_constants(chunk).items():
        print(f"\t{key}: {val}")
```

```txt
Chunk 13:
	signal_type: Echo
	swath_num: 6
	num_quads: 9975
	baq_mode: FDBAQ MODE 0
	swst: 8.333617017329499e-05
	swl: 0.00042669824216607814
	pri: 0.000601150045968743
	elevation_beam_address: 5
Chunk 14:
	signal_type: Echo
	swath_num: 6
	num_quads: 9993
	baq_mode: FDBAQ MODE 0
	swst: 8.160444029437422e-05
	swl: 0.00042744421811392094
	pri: 0.000601150045968743
	elevation_beam_address: 5
```

- Trong ví dụ này, tác giả sẽ tập trung vào vùng bờ biển quanh cảng Santos.
```python
selected_chunk = 13
selection = l0file.get_acquisition_chunk_metadata(selected_chunk)
selection
```
Kết quả in ra packet gồm 46 trường dữ liệu như ở trên, với bản tin từ 408 đến 30341, tổng cộng 29934 bản tin echoes.

## 3.2 Extract Raw I/Q Sensor Data
- Tác giả tiến hành trích dữ liệu IQ thành một ma trận dữ liệu 2 chiều, với chiều ngang là fast time $\tau$ và chiều dọc là slow time $\eta$ .
```python
# Decode the IQ data
radar_data = l0file.get_acquisition_chunk_data(selected_chunk)
assert radar_data.dtype == np.complex64

# Cache this data so we can retreive it more quickly next time we want it
l0file.save_acquisition_chunk_data(selected_chunk)
```
- Sau khi vẽ amplitude của dữ liệu IQ tho, ta thấy dữ liệu cũng có một cấu trúc cụ thể, nhưng ta chưa thể phân biệt/nhận dạng được các vật thể. 
```python
# Plot the raw IQ data extracted from the data file
plt.figure(figsize=(12, 12))
plt.title("Sentinel-1 Raw I/Q Sensor Output")
# We're just going to plot every 20th row/col value for speed here
plt.imshow(abs(radar_data[::20,::20]), vmin=0, vmax=15, origin='lower')
plt.xlabel("Fast Time (down range)")
plt.ylabel("Slow Time (cross range)")
plt.show()
```
![[Pasted image 20260805181828.png]]

# 4. Image Processing
- Ở phần này, tác giả sẽ thực thi các bước của thuật toán Range-Doppler:
	1. Range Compression
	2. Transform to Range-Doppler domain 
	3. Range Cell Migration Correction (RCMC)
	4. Azimuth compression
	5. Transform to time domain
	6. Image formation
## 4.1 Define auxiliary parameters
- Tác giả định nghĩa các tham số cần cho quá trình lấy nét ảnh như sau: 
	- Kích thước ảnh 
	- Các tham số của xung truyền sử dụng để tổng hợp một xung replica Tx
	- Sample rates theo chiều range và azimuth 
	- Giá trị fastime $\tau$ tương ứng với mỗi rang sample dọc theo một range line, và slant range tương ứng của closest approach $R_0$ for each of these range samples.
	- Giá trị tần số theo trục range $f_\tau$ 