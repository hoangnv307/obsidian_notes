`TIME` trong **Band Maths Expression** của SNAP là thời điểm thu nhận tại dòng ảnh đang được tính, biểu diễn dưới dạng **MJD2000**:

- `TIME = 0` → `01-01-2000 00:00:00 UTC`
- Đơn vị là **ngày**
- `TIME = 1.5` → `02-01-2000 12:00:00 UTC`
- Giá trị thường thay đổi theo tọa độ dòng `Y`, nhưng giống nhau trên cùng một dòng.

SNAP lấy thời gian từ `Scene Time Coding` nếu có. Nếu không, nó nội suy tuyến tính giữa `Start Time` và `End Time` của product:

```text
TIME(Y) = startTime
        + Y × (endTime - startTime) / (imageHeight - 1)
```

Nếu product không có thông tin thời gian, `TIME` trả về `NaN`.

Ví dụ:

```text
TIME >= 7305
```

chọn các pixel có thời gian từ khoảng `01-01-2020` trở đi, vì ngày đó cách mốc `01-01-2000` khoảng 7305 ngày.

Lưu ý: mặc dù `TIME` nằm trong danh sách **Constants** của giao diện Expression Editor, nó không nhất thiết là hằng số cho toàn ảnh. `MJD` là tên tương thích cũ và có cùng ý nghĩa với `TIME`.