Phần hiển thị trạng thái của biểu thức trong **Band Maths Expression Editor** có 4 trường hợp:

| Trường hợp | Nội dung hiển thị | Màu |
|---|---|---|
| Biểu thức trống hoặc chỉ có khoảng trắng | Không hiển thị trạng thái, tooltip cũng rỗng | Mặc định |
| Còn ký tự giữ chỗ `@` | `Replace '@' by inserting an element.` | Đỏ đậm `#800000` |
| Biểu thức hợp lệ | `Ok, no errors.` | Xanh lá `#008000` |
| Biểu thức không hợp lệ | Hiển thị thông báo lỗi cụ thể; tooltip chứa cùng nội dung | Đỏ đậm `#800000` |

Các lỗi không hợp lệ cụ thể có thể gồm:

- Symbol không tồn tại: `Undefined symbol 'missing'.`
- Biểu thức chưa hoàn chỉnh: `Incomplete expression.`
- Lỗi cú pháp khác từ ExprTk.
- Product nguồn đã đóng: `Band Maths source product is no longer open: $<id>`
- Tên node không thể biểu diễn trong Band Maths.
- Các raster tham chiếu không cùng kích thước: `Referenced rasters must all be of the same size`
- Exception khác phát sinh trong quá trình chuẩn bị/đánh giá biểu thức.

Trạng thái được cập nhật khi mở editor, mỗi lần nội dung thay đổi và trước khi bấm **OK**. Nếu còn lỗi khi bấm **OK**, editor hiện hộp thoại `Error` và không đóng.

Nguồn chính: [band_maths_expression_editor.cpp](/home/hoangnv307/code/sar_application/src/ui/product/band_maths_expression_editor.cpp:526), logic kiểm tra: [band_maths_expression.cpp](/home/hoangnv307/code/sar_application/src/engine/core/datamodel/raster/band_maths_expression.cpp:1272), kiểm thử UI: [test_band_maths_dialog.cpp](/home/hoangnv307/code/sar_application/test/ui/test_band_maths_dialog.cpp:233).