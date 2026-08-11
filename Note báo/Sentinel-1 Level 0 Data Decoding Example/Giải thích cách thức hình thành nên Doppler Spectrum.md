[Inference] Không hẳn. Chỗ này cần sửa cách hiểu từ câu trả lời trước: **không nên nói “mỗi echo line có một Doppler centroid riêng vì nó có một $v_{LOS}$ riêng”.** Doppler trong SAR được hình thành chủ yếu khi ta quan sát **cùng một range cell/scatterer qua nhiều echo lines liên tiếp theo azimuth**.

Một **echo line đơn lẻ** là tín hiệu thu sau **một pulse**, và trục của nó chủ yếu là **fast time/range**:

```text
Echo line k
      range / fast time →
┌───────────────────────────────────────┐
│ x[k,0] x[k,1] x[k,2] ... x[k,N-1]   │
└───────────────────────────────────────┘
```

Nếu chỉ lấy riêng line này, bạn **chưa có chuỗi slow-time để tạo Doppler spectrum theo azimuth**.

Khi vệ tinh phát nhiều pulse liên tiếp, ta xếp các echo line thành ma trận raw SAR:

```text
                    RANGE / fast time →
                 r0    r1    r2    r3       ...
               ┌─────┬─────┬─────┬─────┬─────
echo line 0     │     │     │     │     │
echo line 1     │     │     │     │     │
echo line 2     │     │     │  X  │     │
echo line 3     │     │     │  X  │     │
echo line 4     │     │     │  X  │     │
echo line 5     │     │     │  X  │     │
               ↓
          AZIMUTH / slow time
```

Giả sử một scatterer nằm gần range bin $r_3$. Radar nhìn thấy nó trong hàng trăm/hàng nghìn pulse liên tiếp. Ta lấy chuỗi:

$x[0,r_3],x[1,r_3],x[2,r_3],\ldots$

rồi FFT theo **azimuth/slow-time**:

$X(f_a,r_3)=\operatorname{FFT}_{\eta}{x(\eta,r_3)}$

thì mới thu được **Doppler spectrum** của range bin đó.

Trong khoảng thời gian synthetic aperture, $v_{LOS}$ của scatterer thay đổi vì hình học radar–target thay đổi, nên Doppler tức thời cũng thay đổi:

$f_D(\eta)=-\frac{2}{\lambda}v_{LOS}(\eta)$

Ví dụ broadside lý tưởng, khi vệ tinh tiến đến target:

```text
       trước closest approach        closest        sau closest approach

SAR ●  -------->              SAR ● -------->          SAR ● -------->
       \                           |                         /
        \                          |                        /
         \                         |                       /
          X                        X                      X

   vLOS ≠ 0                    vLOS = 0                vLOS ≠ 0
   fD > 0                      fD = 0                  fD < 0
```

Do đó Doppler history của **một target** có dạng gần tuyến tính quanh closest approach:

$f_D(\eta)\approx f_{DC}+K_a(\eta-\eta_c)$

Với broadside lý tưởng, $f_{DC}\approx0$:

```text
Doppler frequency
       ^
 +fa   |\
       | \
       |  \
       |   \
  0 ---|----●----------------  ← fDC ≈ 0
       |     \
       |      \
 -fa   |       \
       +----------------------> slow time η
              ηc
```

Còn nếu antenna squint thì toàn bộ Doppler history bị dịch:

```text
Doppler frequency
       ^
       |
       |\
       | \
fDC ---|--●-------------------  ← chẳng hạn +500 Hz
       |   \
       |    \
  0 ---|----------------------
       +----------------------> slow time η
             ηc
```

Vì thế có **hai khái niệm khác nhau**:

**Doppler tức thời** $f_D(\eta)$ thay đổi từ pulse này sang pulse khác đối với cùng scatterer, còn **Doppler centroid** $f_{DC}$ mô tả vị trí trung tâm của phổ Doppler thu được từ một tập các pulse/azimuth samples.

Có thêm một điểm quan trọng liên quan trực tiếp đến câu hỏi “trên một echo line thì sao?”. Trên cùng một echo line có rất nhiều **range bins**, tức nhiều vùng mặt đất ở các slant range khác nhau. Hình học LOS của chúng hơi khác nhau, nên Doppler centroid có thể phụ thuộc range:

$f_{DC}=f_{DC}(\tau)$

và khi xét toàn bộ acquisition, nó còn có thể thay đổi chậm theo azimuth:

$f_{DC}=f_{DC}(\eta,\tau)$

Nhưng điều đó **không có nghĩa là bạn FFT một echo line theo range rồi gọi phổ thu được là Doppler spectrum**. FFT theo range liên quan tới **range frequency/chirp**, còn Doppler spectrum SAR mà ta đang nói tới được hình thành theo **azimuth từ nhiều echo lines**.

Đây cũng là lý do trong xử lý Sentinel-1, khi nói **Doppler Centroid Estimation**, processor phải dùng một **block gồm nhiều range lines**, chứ không thể nhìn một sample hay một echo line đơn độc rồi xác định đầy đủ Doppler centroid.

Nếu bạn muốn, bước tiếp theo rất đáng làm là tôi có thể lấy chính **ma trận `radar_data` L0 Sentinel-1 mà bạn đang dùng** và chỉ ra cụ thể: **FFT theo chiều nào → Doppler spectrum xuất hiện ở đâu → từ spectrum đó Sentinel-1 ước lượng $f_{DC}$ như thế nào**.