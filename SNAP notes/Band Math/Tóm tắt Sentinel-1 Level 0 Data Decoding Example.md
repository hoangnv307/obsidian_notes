
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
- Ngoài ra, thông tin về quỹ đạo của vệ tinh còn được 