[[ Ví dụ cho các biểu thức Band Math]]
## Toán tử ba ngôi 
`A ? B : C`
`if A then B else C`
## Toán tử logic 
- `X || Y` hoặc `X or Y` Logical OR
- `X && Y` hoặc `X and Y` Logical AND
## Binary Comparison Operator
Toán tử này reture true hoặc false
- `X == Y`
- `X!= Y`
- `X < Y`
- `X<=Y`
- `X > Y`
- `X >= Y`
## Binary Bitwise Operators
- X | Y
- X ^ Y
- X & Y
## Arithmetic Operators
- X + Y
- X - Y
- X * Y
- X / Y
- X % Y
## Unary Operators
- \+ X
- \- X
- !X hoặc not X: Logical NOT of boolean argument X
- ~X: bitwise NOT of 
## Mathematical Constants
- PI
- E: natural logarithm
- NaN
- X: Tọa độ x của pixel hiện tại 
- Y: Tọa độ y của pixel hiện tại
- LAT: Giá trị của vĩ độ pixel hiện tại
- LON: Giá trị của kinh độ pixel hiện tại
- TIME: [[Ý nghĩa và ứng dụng của biến TIME]]
## Mathematical Functions
- sqrt(X)
- pow(X, Y): X mũ Y
- exp(X)
- exp10(X)
- log(X)
- log10(X)
- sin(X): X ở radians
- sinh(X)
- cos(X)
- cosh(X)
- tan(X)
- tanh(X)
- asin(X): arcsine
- acos(X)
- atan(X)
- atan2(Y, X): trả về góc trên tọa độ cực của X, Y
- sech(X): Trả về hyperbolic secant của góc X
- cosech(): Trả về hyperbolic cosecant của góc X
- ampl(R, I): Trả về hàm biên độ của số phức, tương đương với sqrt(R* R + I * I)
- phase(R, I): Trả về hàm pha của số phức, tương đương với atan2(I,R)
- rad(X): chuyển X từ độ sang radian
- deg(X): chuyển X từ radian sang độ
- abs(X)
- sign(X): trả về dấu của A, bao gồm {-1, 0,+1}
- min(X, Y)
- max(X, Y)
- floor(X): Trả về số double lớn nhất (gần nhất với $+\infty$), bé hơn hoặc bằng X và là 1 số nguyên, giả sử nếu X = 45.56 thì trả về 45, X = -45.67 thì trả về -46.
- round(X): Trả về số kiểu double gần nhất với X. Giá trị được làm tròn tới một giá trị nguyên bằng cách cộng 1/2, lấy floor của kết quả, và cast kết quả về kiểu long. 
- ceil(X): Trả về số double nhỏ nhất (gần nhất với $-\infty$), lớn hơn hoặc bằng X và bằng với một số nguyên
- rint(X): Trả về giá trị double gần với giá trị của X nhất và bằng với một số nguyên. Nếu 2 giá trị double đều là 2 số nguyên có khoảng cách tới X bằng nhau, kết quả là  số nguyên chẵn (ví dụ X = 51.5 thì rint(X) = 52). 
- feq(X, Y): so sánh xem 2 số có xấp xỉ bằng nhau hay không? (sai số $\epsilon$ mặc định là 1e-6)
- feq(X, Y, EPS): EPS là dung sai tuyệt đối
- fneq(X, Y): so sánh 2 số không xấp xỉ bằng nhau
- fneq(X, Y, EPS)
- nan(X): check xem số đó có phải NaN Không? 
- inf(X): Trả về true nếu X là infinity large
- random_uniform(): trả về pseudo-random, phân phối đều từ 0.0 đến 1.0
- random_gaussian(): Trả về pseudo-random, phân phối Gaussian ("normally)
- stddev(X, Y, ...): Trả về độ lệch chuẩn của các phần tử được tách nhau bởi dấu ",". 
- coef_var(X, Y, ...): Trả về hệ số biến thiên (CV) của các phần tử được tách nhau bởi dấu ","
- bit_set(X, N): Trả về true hoặc falase dựa vào bit thứ N của giá trị X. 
- avg()
- distance()
- distance_deriv()
- distance_integ()
- inrange()
- inrange_deriv()
- inrange_integ()
- sq()

## Một số gợi ý thêm từ codex
Có. Parser hiện không tắt built-in của ExprTk, nên các hàm dưới đây có thể gõ trực tiếp dù chưa xuất hiện trong menu Functions.

  Những hàm đáng bổ sung nhất:

  - clamp(min, X, max): giới hạn backscatter, cường độ hoặc giá trị dB vào một khoảng; rất hữu ích để loại ảnh hưởng của outlier trước khi hiển thị.
  - asinh(X): nén dải động nhưng vẫn xử lý tốt giá trị gần 0 hoặc âm; phù hợp dữ liệu SAR hơn log(X) trong một số trường hợp.
  - hypot(X, Y): tính sqrt(X² + Y²) ổn định hơn; dùng cho biên độ từ hai thành phần I/Q. Chức năng gần giống ampl(R, I).
  - log1p(X): tính log(1 + X) chính xác hơn khi X rất nhỏ; hữu ích với dữ liệu cường độ đã chuẩn hóa.
  - expm1(X): tính exp(X) - 1 chính xác hơn gần 0.
  - sum(X, Y, ...): tổng nhiều band hoặc nhiều kênh phân cực.
  - mul(X, Y, ...): tích nhiều đầu vào.
  - roundn(X, N): làm tròn đến N chữ số thập phân.
  - trunc(X) và frac(X): lấy phần nguyên và phần thập phân; hữu ích cho chỉ số chu kỳ hoặc xử lý pha.
  - log2(X), logn(X, N) và root(X, N): logarithm cơ số tùy ý và căn bậc N.
  - erf(X), erfc(X), ncdf(X): hữu ích cho ngưỡng xác suất, phát hiện bất thường hoặc mô hình thống kê nhiễu.
  - sinc(X): có ích trong mô phỏng đáp ứng xung range/azimuth và nghiên cứu xử lý tín hiệu SAR.

  Ít cần đưa vào menu phổ thông hơn:

  - cot, sec, csc: ít được dùng trực tiếp trong Band Maths.
  - iclamp: hành vi khó hiểu với người dùng phổ thông.
  - equal, not_equal: đã gần tương đương feq, fneq.
  - deg2rad, rad2deg: đã có rad và deg.

  Nếu giữ menu gọn, tôi đề xuất thêm trước:

  clamp(@, @, @)
  asinh(@)
  hypot(@, @)
  log1p(@)
  expm1(@)
  sum(@, @, ...)
  mul(@, @, ...)
  roundn(@, @)
  trunc(@)
  frac(@)

  Nhóm erf, erfc, ncdf, sinc có thể thêm sau vì chuyên biệt hơn.