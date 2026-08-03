Trong BandMathsExpressionEditor hiện có các trạng thái liên quan đến expression:

  - Expression rỗng: không hiển thị cảnh báo; validation label bị xóa và vẫn có thể nhấn OK.
  - Expression còn ký tự placeholder @: hiển thị inline:
    Replace '@' by inserting an element.

  - Expression hợp lệ: hiển thị inline màu xanh:
    Ok, no errors.


  - Nếu có lỗi, editor mở hộp thoại tiêu đề Error, nội dung là lỗi đang hiển thị inline và giữ editor mở.
  - Nếu expression rỗng hoặc hợp lệ, editor được đóng với trạng thái Accepted.

  Luồng nằm tại src/ui/product/band_maths_expression_editor.cpp:370 và phần tạo lỗi tại src/engine/core/datamodel/raster/band_maths_expression.cpp:1255.

  Lưu ý: expression rỗng hiện không bị cảnh báo ngay trong Expression Editor; chỉ khi quay về Band Maths và nhấn OK thì cửa sổ cha mới báo expression không
  hợp lệ.