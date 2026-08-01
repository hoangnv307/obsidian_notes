TIME chủ yếu hữu ích với dữ liệu mà thời điểm thu nhận thay đổi theo từng dòng ảnh. Với một ảnh Sentinel‑1 GRD thông thường, thời gian quét chỉ kéo dài vài
  giây nên biến này khá ít được dùng.

  Các trường hợp thực tế gồm:

  - Tạo band thời gian thu nhận cho từng pixel/dòng quét:

  TIME

  - Chuyển thời gian thành số giây kể từ lúc bắt đầu chụp:

  (TIME - 9132.123456) * 86400

  - Chỉ giữ dữ liệu thuộc một khoảng thời gian trong cảnh:

  TIME >= start_time && TIME <= end_time ? Amplitude_HH : NaN

  - Hiệu chỉnh giá trị thay đổi theo thời gian quét, ví dụ drift của cảm biến:

  radiance - correction_rate * (TIME - start_time) * 86400

  - Phân tích cảm biến quét theo dòng như Sentinel‑2, Landsat, MODIS, MERIS hoặc dữ liệu khí tượng, nơi các dòng không được ghi nhận đồng thời.
  - So sánh thời gian giữa các product đang mở:

  abs(TIME - $2.TIME) * 86400

  Biểu thức trên tạo độ lệch thời gian theo giây giữa hai product.

  - Kết hợp dữ liệu quỹ đạo hoặc hiện tượng thay đổi nhanh như mây, nhiệt độ bề mặt biển, cháy rừng hoặc chuyển động vật thể.

  Đối với ứng dụng SAR hiện tại, TIME hữu ích nhất cho:

  - biểu diễn thời gian azimuth theo từng dòng;
  - hiệu chỉnh phụ thuộc thời gian/quỹ đạo;
  - so sánh các cảnh SAR đa thời gian;
  - xử lý dữ liệu burst hoặc swath có thời gian thu nhận khác nhau.

  Tuy nhiên, để dùng chính xác cho Sentinel‑1 SLC/burst, ứng dụng cần hỗ trợ TimeCoding theo burst hoặc từng dòng. Hiện tại chương trình chỉ nội suy tuyến
  tính từ Start Time đến End Time, phù hợp với các sản phẩm đơn giản nhưng chưa mô tả đầy đủ ranh giới burst.