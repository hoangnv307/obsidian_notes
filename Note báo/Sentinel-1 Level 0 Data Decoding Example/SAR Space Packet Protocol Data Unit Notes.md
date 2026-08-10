### 1.3.1 Quy ước và định nghĩa
- Dữ liệu Space Packet được biểu diễn dưới dạng một chuỗi các octets.
- Vị trí bắt đầu của 1 tham số/data word nằm trong một chuỗi được định nghĩa bằng "Octet Offset" và "Bit Offset" bên trong octet đó. "Octet Offset" tham chiếu đến điểm bắt đầu của cả Space Packet.

# 3. SAR Space Packet Format
- Một Sentinel-1 SAR Space Packet tổng thể sẽ được biểu diễn như ở Table 2.4-1 và có kích thước tổng bằng ==cơ số lần của 4 octets== 
![[Pasted image 20260810144709.png]]
## 3.1 Packet Primary Header
![[Pasted image 20260810145105.png]]