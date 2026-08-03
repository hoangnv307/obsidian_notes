Đoạn note sau đây sẽ dựa trên tài liệu SRD để liệt kê ra các bài test về chức năng band math.
Các phần test sẽ bao gồm 2 phần test chính:
1. Test modal window Band Maths
2. Test band math expression editor
# Các bài test cho modal window Band Maths
- [ ] Cửa sổ modal này và mọi cửa sổ modal khác không được có nút minimum, maximum, chỉ có nút X (kiểm tra trên VM Linux)
- [ ] Drop-down list `Target product` tự scale độ rộng, tùy theo tên sản phẩm dài nhất mà product explorer chứa. 
- [ ] Trường `Name` phải không cho nhập giá trị rỗng, không cho nhập trùng tên band, mask, TPG đã tồn tại của target product.
- [ ] Kiểm tra lại tính logic của checkbox `Virtual`, đảm bảo chương trình không dùng nhiều CPU usage hơn SNAP.
- [ ] Check các trường hợp biên của trường `Replace NaN and infinity `