[Inference] “**Hàm theo range**” ở đây nghĩa là Doppler Centroid **không được coi là một giá trị duy nhất cho toàn bộ chiều range**, mà được mô hình hóa thành một hàm phụ thuộc vào vị trí range (hay fast time/slant range).

Ví dụ sau khi estimation, bạn có thể thu được Doppler centroid tại một số vị trí range:

|Range bin|Doppler Centroid|
|--:|--:|
|1000|320 Hz|
|2000|335 Hz|
|3000|351 Hz|
|4000|368 Hz|

Sau bước **unwrap + ambiguity resolution**, các điểm này được polynomial fitting để tạo một hàm liên tục, chẳng hạn:

$f_{DC}(\tau)=c_0+c_1(\tau-\tau_0)+c_2(\tau-\tau_0)^2$

Trong đó $\tau$ là **fast time**, tương ứng với vị trí theo range. Vì:

$R=\frac{c\tau}{2}$

nên cũng có thể hiểu tương đương là:

$f_{DC}=f_{DC}(R)$

Hình dung trên một echo line:

```text
                    RANGE →
Near range                              Far range
     │                                      │
     ▼                                      ▼
─────●────────●────────●────────●───────────●────
   τ₁       τ₂       τ₃       τ₄          τ₅

 fDC(τ₁)  fDC(τ₂)  fDC(τ₃)  fDC(τ₄)   fDC(τ₅)
   320      330      342      355        368 Hz
```

Nhưng có một điểm rất quan trọng: **không phải Doppler centroid tại mỗi range bin được tính từ riêng một echo line**. Doppler là thông tin theo **azimuth**, nên tại một vị trí range $\tau_i$, processor cần lấy nhiều echo lines:

```text
                     RANGE →
                 τ₁    τ₂    τ₃    τ₄
echo line 1      ●     ●     ●     ●
echo line 2      ●     ●     ●     ●
echo line 3      ●     ●     ●     ●
echo line 4      ●     ●     ●     ●
echo line 5      ●     ●     ●     ●
                 ↑
              cố định τ₂
                 │
                 ↓
          FFT/correlation theo
             AZIMUTH
                 ↓
              fDC(τ₂)
```

Processor thực tế thường chia dữ liệu thành các **range blocks**, estimate $f_{DC}$ cho từng block, rồi fit các kết quả đó. Vì thế polynomial fitting biến một tập điểm rời rạc:

$(\tau_1,f_1),(\tau_2,f_2),\ldots,(\tau_N,f_N)$

thành:

$\boxed{f_{DC}(\tau)}$

Đây chính là ý của câu **“thu được hàm Doppler Centroid cuối cùng theo range”**.

Và lý do vật lý khiến $f_{DC}$ thay đổi theo range là các range khác nhau tương ứng với **góc nhìn/slant range khác nhau**, nên hình học radar–mặt đất và thành phần vận tốc LOS có thể khác nhau. Vì thế có thể có:

$f_{DC}(\tau_1)\neq f_{DC}(\tau_2)$

Trong ngữ cảnh Sentinel-1, bước tiếp theo quan trọng là phân biệt **$f_{DC}(\tau)$ theo range** này với việc **Doppler centroid estimate còn được cung cấp tại nhiều azimuth times**. Hai chiều biến thiên này khác nhau và giải thích khá rõ cấu trúc `dcEstimateList` mà bạn từng hỏi.