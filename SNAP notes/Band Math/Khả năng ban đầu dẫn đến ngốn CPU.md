Nguyên nhân chính của pan là canvas vẽ lại toàn bộ viewport cho mỗi mouse-move:

  1. Mỗi chuyển động chuột gọi NotifyViewportChanged(): src/ui/render/layer_canvas.cpp:572.
  2. Hàm này gọi update() không kèm vùng giới hạn: src/ui/render/layer_canvas.cpp:711.
  3. paintEvent() xóa toàn bộ canvas bằng fillRect(rect()), duyệt mọi layer và vẽ lại tất cả tile đang nhìn thấy: src/ui/render/layer_canvas.cpp:453.
  4. Từng tile lại được biến đổi và drawImage() bằng QPainter: src/ui/render/concurrent_multi_level_renderer.cpp:553.

  Do đó cache hiện tại chỉ tránh đọc/tính lại tile; nó không tránh việc liên tục raster hóa toàn bộ màn hình. Không có giới hạn FPS, debounce hay cơ chế dịch
  framebuffer cũ rồi chỉ vẽ phần mới lộ ra.

  Zoom còn tốn thêm CPU vì:

  - Có thể schedule tối đa 64 tile trên QThreadPool toàn cục 16 luồng: src/ui/render/concurrent_multi_level_renderer.cpp:21.
  - TileImage tiếp tục quét toàn bộ ảnh để kiểm tra alpha.
  - Khi tác vụ bị hủy, vòng raw → ARGB không kiểm tra cờ hủy, nên vẫn có thể chạy đến hết.

  SNAP dùng JAI tile cache và thường bọc trực tiếp DataBuffer thành ảnh tương thích thiết bị, giảm các lượt copy pixel. Bản SNAP được so sánh cũng là bản tối
  ưu, trong khi ứng dụng hiện tại mặc định chạy Debug: auto.sh:10.

  Kết luận sửa lại:

  - Pan cao CPU: redraw toàn viewport theo từng mouse-move.
  - Zoom cao hơn: redraw toàn viewport cộng pipeline tile nhiều lượt và song song rộng.
  - Debug build khuếch đại cả hai.
  - World View là vấn đề CPU độc lập khi tab đó hiển thị, không giải thích trường hợp bạn vừa mô tả.

  Mình chưa sửa mã. Mẫu runtime vừa lấy không bắt được lúc bạn thao tác, nên thứ tự trên là kết luận từ luồng mã; muốn định lượng tỷ lệ từng điểm nóng cần
  chạy bản RelWithDebInfo dưới profiler trong đúng lúc pan/zoom.