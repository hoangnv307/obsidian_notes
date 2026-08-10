Một số chuẩn
- SD 01 ----- CCSDS 133.0-B-1 ---- Space Packet Protocol
### 1.3.1 Quy ước và định nghĩa
- Dữ liệu Space Packet được biểu diễn dưới dạng một chuỗi các octets.
- Vị trí bắt đầu của 1 tham số/data word nằm trong một chuỗi được định nghĩa bằng "Octet Offset" và "Bit Offset" bên trong octet đó. "Octet Offset" tham chiếu đến điểm bắt đầu của cả Space Packet.

# 3. SAR Space Packet Format
- Một Sentinel-1 SAR Space Packet tổng thể sẽ được biểu diễn như ở Table 2.4-1 và có kích thước tổng bằng ==cơ số lần của 4 octets== 
![[Pasted image 20260810144709.png]]
## 3.1 Packet Primary Header
![[Pasted image 20260810145105.png]]
## 3.2 Packet Secondary Header
- Theo chuẩn [SD 01], PSH bao gồm 1 Time Code Field và một Ancillary Data Field.
- PSH cung cấp thông tin của một số `Services` được shown ở Table 3.2-1. Mỗi  trường Service chiếm 1 số nguyên lần octets. 
![[Pasted image 20260810150335.png]]
### 3.2.1 Datation Service (Dịch vụ đánh dấu thời gian)
>[!note]
>The time stamp value is a sample of the local instrument time at a specific event within the PRI where the packet data has been acquired.

Trường này bao gồm 6 octets như ở Table 3.2-2:
![[Pasted image 20260810150831.png]]