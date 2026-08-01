## Toán tử ba ngôi 
`A ? B : C`
## Toán tử bit 
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
- round(X): Trả về số kiểu long gần nhất với X. Giá trị được làm tròn tới một giá trị nguyên bằng cách cộng 1/2, lấy floor của kết quả, và cast kết quả về kiểu long. 
- ceil(X): Trả về số double nhỏ nhất (gần nhất với $-\infty$), lớn hơn hoặc bằng X và bằng với một số nguyên
- rint(X): Trả về giá trị double gần với giá trị của X nhất và bằng với một số nguyên. Nếu 2 giá trị double đều là 2 số nguyên có khoảng cách tới X bằng nhau, kết quả là  số nguyên chẵn (ví dụ X = 51.5 thì rint(X) = 52). 
- feq(X, Y): so sánh xem 2 số có xấp xỉ bằng nhau hay không? (sai số $\epsilon$ mặc định của thư viện, cần check lại)
- feq(X, Y, EPS): EPS là dung sai tuyệt đối
- fneq(X, Y): so sánh 2 số không xấp xỉ bằng nhau
- fneq(X, Y, EPS)
- nan(X): check xem số đó có phải NaN Không? 
- inf(X): Trả về true nếu X là infinity large
- random_uniform(): trả về pseudo-random, phân phối chuẩn từ 0.0 đến 1.0
- random_gaussian(): Trả về pseudo-random, phân phối Gaussian ("normally)
- stddev(X, Y, ...): Trả về độ lệch chuẩn của các phần tử được tách nhau bởi dấu ",". 
- coef_var(X, Y, ...): Trả về hệ số biến thiên (CV) của các phần tử được tách nhau bởi dấu ","
- bit_set(X, N): Trả về true hoặc falase dựa vào bit thứ N của giá trị X. 