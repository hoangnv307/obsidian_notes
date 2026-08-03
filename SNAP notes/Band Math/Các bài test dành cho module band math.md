Đoạn note sau đây sẽ dựa trên tài liệu SRD để liệt kê ra các bài test về chức năng band math.
Các phần test sẽ bao gồm 2 phần test chính:
1. Test modal window Band Maths
2. Test band math expression editor
# Todo 
- [x] Xóa các icon thừa trên các buttons
- [x] Giảm size chữ cảnh báo xuống.
- [ ] Sửa bug QT_SCALE_FACTOR=1.25 thì menu chọn từ menu bar bị lỗi hiển thị
# Các bài test cho modal window Band Maths
- [ ] Cửa sổ modal này và mọi cửa sổ modal khác không được có nút minimum, maximum, chỉ có nút X (kiểm tra trên VM Linux)
- [x] Drop-down list `Target product` tự scale độ rộng, tùy theo tên sản phẩm dài nhất mà product explorer chứa. 
- [x] Trường `Name` phải không cho nhập giá trị rỗng, không cho nhập trùng tên band, mask, TPG đã tồn tại của target product.
- [ ] Kiểm tra lại tính logic của checkbox `Virtual`, đảm bảo chương trình không dùng nhiều CPU usage hơn SNAP.
- [x] Check các trường hợp biên của trường `Replace NaN and infinity results by`, mặc định sẽ là `NaN`, xác định xem như nào là infinity? Có phải 1.0/0.0 là infty không? Check xem các gái trị như kiểu max của data DN xem có phải là infty không? Nhập thử giá trị như Hoang, 10e100 xem có được không? -> Kết quả là nếu vượt quá max,min double thì sẽ chuyển sang Infinity, nếu không parse được sang double thì sẽ cảnh báo, nhập (+)Infinity/-Infinity, không quan tâm viết hoa hay thường. 
- [x] Nút `Load...` chỉ hiển thị filter cho .txt, kiểm tra xem file được parse đúng. 
- [ ] Nút `Save...` đặt tên file mặc định là `myExpression.txt`, nếu save file có cùng tên với file đã có sẵn, cửa sổ hiện lên hỏi người dùng có muốn overwrite file đã có hay không? 

# Các bài test cho modal window Band Maths Expression Editor
- [ ] Test các bài test kết hợp biểu thức
- [ ] Thêm một vài các biểu thức ngoài SNAP hữu ích
- [ ] Check xem các hàm fuzzy 2 tham số lấy epsilon bằng bao nhiêu. 
- [ ] Các các nút thao tác + nút tắt đã hoạt động ổn định chưa? Check xem số lần lưu lịch sử là bao nhiêu? Lưu vào đâu? 
- [x] Check xem có những trường hợp cảnh báo nào? [[Những cảnh báo  trạng thái band math editor ]]

