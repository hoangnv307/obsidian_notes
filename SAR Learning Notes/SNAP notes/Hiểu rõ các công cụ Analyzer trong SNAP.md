- Các công cụ Analyzer được phát triển trong Sentinel Toolbox. 
==Plot = Đồ thị/Biểu đồ==
- Các công cụ Analyzer bao gồm:
	- Correlative Plot: Đồ thị tương quan 
	- Scatter Plot: Đồ thị phát xạ
	- Profile Plot: Đồ thị mặt cắt
	- Infomation
	- Geo-coding
	- Histogram
	- Statistics
	- Metadata Plot
Sau đây là phân tích chi tiết ý nghĩa, mục đích dùng của các loại công cụ trên.
## Correlative Plot
Đây là giao diện của đồ thị tương quan: 
![[Pasted image 20260726153955.png]]
Trường hợp thực tế sử dụng Correlative Plot: [[Trường hợp sử dụng Correlative Plot]]
## Scatter Plot 
Đồ thị này cho phép người dùng vẽ đồ thị một nút dữ liệu raster với một nút dữ liệu raster khác. ![[Pasted image 20260726164657.png]]
Ứng dụng: [[Trường hợp sử dụng Scatter Plot]]
## Profile Plot
- Công cụ này được kích hoạt chỉ khi sản phẩm có chứa vector data.
- Đồ thị mặt cắt hoạt động ở 2 chế độ: 
	- Chế độ classic: Dựa vào một dạng hình học. 
	- Chế độ tương quan: Cho phép người dùng so sánh dữ liệu vệ tinh với dữ liệu thực địa.
### Chế độ classic
Giá trị của trục x được cung cấp bởi giá trị `path length`, với một path bao gồm 300 pixels thì sẽ bao gồm 300 giá trị. Trục y được xác định bởi band ảnh được chọn. 
![[Pasted image 20260726174933.png|637]]
Đồ thị Profile còn hỗ trợ mapping vị trí trong band ảnh tới vị trí trong đồ thị (như hình trên).
Ứng dụng:[[Trường hợp sử dụng Profile Plot Classic Mode]]
### Chế độ correlative
- Khác với classic mode vẽ data band ảnh dựa trên mỗi pixel của vector, correlatice mode hiển thị trường dữ liệu của vector nhập vào (các đỉnh) (màu đỏ trên hình) và giá trị của pixel (màu xanh). 
![[Pasted image 20260727083908.png]]
Ứng dụng: [[Trường hợp sử dụng Profile Plot Correlative Mode]]

## Histogram Plot
Ứng dụng: [[Trường hợp sử dụng Histogram Plot]]
## Statistic Plot
![[Pasted image 20260727145744.png]]
Bộ thông tin thống kê được hiển thị bao gồm:
- Tổng số pixels
- Giá trị địa vật lý cực đại/cực tiểu
- Trung bình đại số của các giá trị 