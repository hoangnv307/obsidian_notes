- Tên tiếng việt: Tạo băng ảnh bằng biểu thức toán học
- Dàn ý triển khai, gợi ý những chủ đề:
	1. Expression parsing
	2. Variable binding
	3. Pixel-wise computation
	4. Virtual band 
	5. Tile-based processing
	6. Các nút tính năng thao tác
# Tạo biểu đồ ca sử dụng
Những tính năng người dùng tương tác: 
- Cửa sổ cấu hình băng ảnh được tạo
	- Chọn tên, mô tả, đơn vị của pixel
	- Chọn loại băng ảnh: Virtual/Nonvirtual 
	- Lựa chọn thay thế các pixel có giá trị NaN/Infinity
	- Trường nhập biểu thức
	- Load/save biểu thức toán học
- Cửa sổ hiển thị trình soạn thảo biểu thức:
	- Chọn nguồn dữ liệu nhập nhanh, hiển thị nguồn dữ liệu nhập nhanh
	- Chọn biểu thức nhập nhanh
	- Trường nhập biểu thức 
	- Các công cụ giúp chỉnh sửa biểu thức: chọn tất cả, xóa tất cả, redo, undo, history back, history forward. 
	- Thanh trạng thái hợp lệ của biểu thức
	- Ok/Cancel/Help
- Biểu đồ ca chốt: 
![[Pasted image 20260806104614.png]]
- Cấu hình băng ảnh đích bao gồm 2 chức năng luôn được thực hiện bao gồm: 
	- Cấu hình thông tin băng ảnh: tên băng, mô tả về băng, tên đơn vị pixel của băng, sản phẩm đích chứa nó. 
	- Cấu hình tính chất băng: băng là băng virtual hay non virtual, các pixel NaN/infinity được thế bởi giá trị nào?
- Soạn thảo biểu thức toán học, chức năng này bao gồm:
	- Bộ biên dịch cú pháp đầu vào (vùng nhập biểu thức) từ người dùng
	- Các công cụ hỗ trợ thao tác, bao gồm: lưu thành file, load file, công cụ hỗ trợ nguồn dữ liệu sẵn để nhập vào, công cụ hỗ trợ nhập 