Ví dụ bạn xây dựng CFAR.

CFAR cần biết:

```text
Background

↓

PDF

↓

Threshold

↓

Detection
```

Nếu PDF sai

↓

Threshold sai

↓

PFA sai

↓

Detection sai.

Nên bước đầu tiên luôn là:

> **Clutter của mình thuộc phân bố nào?**

Đó chính là vấn đề mà đoạn tài liệu đang nói tới.

---

# Câu cuối cùng rất đáng chú ý

> **"...irrespective of whether any model is actually a good fit to the data."**

Đây là một ý rất sâu.

Giả sử thực tế clutter không phải:

* Gaussian
* Gamma
* Weibull
* K-distribution

mà là một phân bố hoàn toàn khác.

Bạn vẫn chỉ có 4 lựa chọn trên.

Khi đó kiểm định chỉ có thể nói:

> Trong 4 mô hình này,

**K-distribution giống dữ liệu nhất.**

Điều đó **không có nghĩa** K-distribution là mô hình đúng tuyệt đối.

Nó chỉ là **mô hình phù hợp nhất trong tập các mô hình mà bạn đã đưa ra để so sánh**.

Đây là một nguyên tắc rất phổ biến trong mô hình hóa thống kê: **"mô hình tốt nhất" không đồng nghĩa với "mô hình đúng", mà là "mô hình mô tả dữ liệu tốt nhất trong số các ứng viên hiện có".** Điều này cũng giải thích vì sao trong xử lý ảnh SAR, các nhà nghiên cứu liên tục đề xuất các mô hình clutter mới khi nhận thấy các mô hình cũ chưa mô tả tốt dữ liệu thực tế.
