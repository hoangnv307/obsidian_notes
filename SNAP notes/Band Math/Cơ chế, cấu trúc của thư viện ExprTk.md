---
tags:
  - AI
---




## 1. ExprTk thực chất là gì?

ExprTk là một **bộ phân tích và thực thi biểu thức toán học tại thời điểm chạy** dành cho C++. Nó cho phép chương trình nhận một công thức dưới dạng chuỗi:

```cpp
"2 * x + sin(y)"
```

sau đó phân tích chuỗi này và tính giá trị của nó.

Điểm quan trọng là: ExprTk vẫn chỉ là một thư viện C++ thông thường, chủ yếu nằm trong file header `exprtk.hpp`. Nhưng bên trong thư viện có một hệ thống gần giống **trình biên dịch ngôn ngữ thu nhỏ**, gồm lexer, parser, bảng ký hiệu, cây cú pháp và bộ thực thi. Ba thành phần chính mà API công khai cung cấp là:

- `exprtk::symbol_table<T>`
- `exprtk::expression<T>`
- `exprtk::parser<T>` citeturn640461view5turn447310view0

---

# 2. Nó khác gì một hàm C++ tự viết?

Giả sử bạn tự viết:

```cpp
double calculate(double x, double y)
{
    return 2.0 * x + std::sin(y);
}
```

Công thức này được xác định khi bạn viết mã nguồn. Trình biên dịch C++ sẽ chuyển nó thành mã máy khi build chương trình.

Trong ExprTk, công thức có thể được nhập khi chương trình đang chạy:

```cpp
std::string formula = "2 * x + sin(y)";
```

Sau đó:

```cpp
parser.compile(formula, expression);
double result = expression.value();
```

Khác biệt cơ bản là:

| Hàm C++ tự viết | ExprTk |
|---|---|
| Công thức nằm trong source code | Công thức là chuỗi văn bản |
| Muốn đổi công thức phải sửa và build lại | Có thể đổi công thức khi chương trình đang chạy |
| C++ compiler phân tích biểu thức | Parser của ExprTk phân tích biểu thức |
| Được chuyển thành mã máy | Được chuyển thành cấu trúc nội bộ của ExprTk |
| Biến là tham số hoặc biến C++ | Biến được đăng ký trong `symbol_table` |

Vì vậy ExprTk phù hợp với những ứng dụng như:

- Band Maths giống SNAP;
- công thức do người dùng nhập;
- hệ thống rule hoặc điều kiện;
- calculator;
- xử lý vector bằng biểu thức;
- cấu hình thuật toán bằng công thức thay vì hard-code.

---

# 3. Luồng xử lý tổng quát của ExprTk

Một cách đơn giản, ExprTk xử lý theo chuỗi sau:

```text
Chuỗi biểu thức
      ↓
Lexer / Tokenizer
      ↓
Kiểm tra và chuẩn hóa token
      ↓
Parser
      ↓
AST
      ↓
Tối ưu AST
      ↓
Expression đã compile
      ↓
Đánh giá expression.value()
```

Ví dụ đầu vào:

```text
2 * x + sin(y)
```

## Bước 1 — Lexer: tách chuỗi thành token

Lexer đọc từng ký tự và tạo ra các token:

```text
NUMBER(2)
OPERATOR(*)
SYMBOL(x)
OPERATOR(+)
FUNCTION(sin)
LEFT_PAREN
SYMBOL(y)
RIGHT_PAREN
```

Token có thể hiểu là **đơn vị nhỏ nhất có ý nghĩa đối với ngôn ngữ**.

Chẳng hạn:

```text
123.4
```

là token số, còn:

```text
x
```

là token định danh, và:

```text
>=
```

là token toán tử.

ExprTk có các bước tiền xử lý token như thay thế token, ghép toán tử nhiều ký tự, kiểm tra số, kiểm tra ngoặc và kiểm tra chuỗi token hợp lệ. Ví dụ hai token `>` và `=` có thể được ghép thành toán tử `>=`; cú pháp nhân ngầm như `2x` có thể được chuyển thành `2 * x`. 

---

## Bước 2 — Parser: hiểu cấu trúc biểu thức

Danh sách token vẫn chưa cho biết toán tử nào cần thực hiện trước.

Ví dụ:

```text
2 + 3 * x
```

Không thể thực hiện đơn giản từ trái sang phải, vì phép nhân có độ ưu tiên cao hơn phép cộng. Parser phải hiểu nó là:

```text
2 + (3 * x)
```

chứ không phải:

```text
(2 + 3) * x
```

Tài liệu ExprTk cho biết nó sử dụng **recursive descent parser**, tức là parser đệ quy đi qua các quy tắc ngữ pháp theo từng mức ưu tiên. 

Có thể hình dung các hàm parser tương tự:

```cpp
parse_expression();
parse_term();
parse_factor();
parse_primary();
```

Trong đó:

```text
expression → cộng, trừ
term       → nhân, chia
factor     → lũy thừa
primary    → số, biến, hàm, ngoặc
```

Đây chỉ là mô hình khái niệm; tên hàm nội bộ thực tế có thể khác.

---

# 4. AST là gì?

AST là viết tắt của **Abstract Syntax Tree**, dịch là:

> Cây cú pháp trừu tượng.

Nó biểu diễn **ý nghĩa và cấu trúc của biểu thức**, thay vì giữ nguyên toàn bộ ký tự như người dùng đã nhập.

Với biểu thức:

```text
2 * x + sin(y)
```

AST có thể hình dung như sau:

```text
             Add
            /   \
        Multiply Sin
        /     \    \
   Constant   x     y
      2
```

Hoặc viết gọn:

```text
        +
       / \
      *  sin
     / \   \
    2   x   y
```

Các nút lá là:

```text
2, x, y
```

Các nút trung gian là:

```text
Multiply, Sin, Add
```

ExprTk mô tả `expression` là cấu trúc giữ AST của biểu thức. Việc đánh giá được thực hiện bằng cách duyệt AST theo thứ tự hậu tự, tức **post-order traversal**. citeturn256653view1turn256653view5

---

## 5. Post-order traversal là gì?

Post-order nghĩa là:

1. Tính cây con bên trái.
2. Tính cây con bên phải.
3. Thực hiện nút hiện tại.

Với cây:

```text
        +
       / \
      *  sin
     / \   \
    2   x   y
```

Thứ tự đánh giá là:

```text
2
x
2 * x
y
sin(y)
2 * x + sin(y)
```

Hay biểu diễn dạng hậu tố:

```text
2 x * y sin +
```

Nếu $x=3$ và $y=0$, quá trình là:

```text
2
3
2 * 3 = 6
0
sin(0) = 0
6 + 0 = 6
```

---

# 6. Symbol table hoạt động như thế nào?

Symbol table là **bảng ánh xạ tên trong biểu thức sang đối tượng C++ thực tế**.

Ví dụ biểu thức có:

```text
x + gain * sin(angle)
```

thì symbol table có thể chứa:

```text
"x"     → tham chiếu đến biến C++ x
"gain"  → tham chiếu đến biến C++ gain
"angle" → tham chiếu đến biến C++ angle
"sin"   → hàm sin
```

Ví dụ sử dụng:

```cpp
#include <iostream>
#include <string>
#include "exprtk.hpp"

int main()
{
    using symbol_table_t = exprtk::symbol_table<double>;
    using expression_t   = exprtk::expression<double>;
    using parser_t       = exprtk::parser<double>;

    double x = 3.0;
    double y = 0.0;

    symbol_table_t symbol_table;
    symbol_table.add_variable("x", x);
    symbol_table.add_variable("y", y);
    symbol_table.add_constants();

    expression_t expression;
    expression.register_symbol_table(symbol_table);

    parser_t parser;

    const std::string formula = "2 * x + sin(y)";

    if (!parser.compile(formula, expression))
    {
        std::cerr << "Compile error: "
                  << parser.error()
                  << '\n';
        return 1;
    }

    std::cout << expression.value() << '\n';
}
```

Điểm đáng chú ý là symbol table thường giữ **tham chiếu đến biến**, không chỉ sao chép giá trị vào AST. Vì vậy bạn có thể thay đổi `x` rồi đánh giá lại mà không cần compile lại biểu thức:

```cpp
x = 3.0;
std::cout << expression.value() << '\n';

x = 10.0;
std::cout << expression.value() << '\n';
```

AST vẫn giữ nguyên, nhưng nút biến `x` đọc giá trị mới.

ExprTk cũng cảnh báo rằng vòng đời của biến C++ phải dài ít nhất bằng vòng đời của symbol table và các expression phụ thuộc. Nếu biến bị hủy nhưng expression vẫn giữ tham chiếu tới nó, việc đánh giá sau đó có thể dẫn đến undefined behavior. citeturn640461view5

Có thể hình dung:

```text
AST node Variable(x)
          │
          └──────────> double x trong chương trình C++
```

Chứ không nhất thiết là:

```text
AST node Variable(x) chứa bản sao giá trị 3.0
```

---

# 7. Compile trong ExprTk có phải biên dịch thành mã máy không?

Không nên hiểu như vậy.

Khi gọi:

```cpp
parser.compile(formula, expression);
```

từ “compile” ở đây chủ yếu có nghĩa là:

- phân tích chuỗi;
- kiểm tra cú pháp;
- giải quyết tên biến và hàm;
- xây dựng AST;
- thực hiện một số tối ưu;
- lưu cấu trúc đã xử lý vào `expression`.

Tài liệu công khai hiện tại của ExprTk mô tả `expression` là AST và mô tả việc thực thi bằng cách duyệt AST. Nó không mô tả một bước JIT chuyển biểu thức thành mã máy. citeturn256653view1turn256653view5

Vì thế không nên nhầm:

```text
ExprTk compile
```

với:

```text
GCC/Clang compile C++ thành machine code
```

---

# 8. Bytecode là gì?

Bytecode là một cách biểu diễn chương trình dưới dạng **danh sách lệnh trung gian tuyến tính**.

Ví dụ AST:

```text
        +
       / \
      *  sin
     / \   \
    2   x   y
```

có thể được chuyển thành bytecode giả định:

```text
PUSH_CONST 2
LOAD_VAR   x
MUL
LOAD_VAR   y
SIN
ADD
RETURN
```

Một máy ảo dạng stack có thể thực hiện như sau:

```text
Lệnh          Stack
--------------------------------
PUSH_CONST 2  [2]
LOAD_VAR x    [2, x]
MUL           [2*x]
LOAD_VAR y    [2*x, y]
SIN           [2*x, sin(y)]
ADD           [2*x + sin(y)]
```

Bytecode thường gồm:

```text
opcode + operand
```

Ví dụ:

```text
LOAD_VAR 5
```

Trong đó:

- `LOAD_VAR` là opcode;
- `5` có thể là chỉ số của biến trong bảng biến.

Bytecode là mã trung gian dành cho một interpreter hoặc virtual machine, không phải mã máy của CPU. CPython là một ví dụ quen thuộc: compiler tạo các bytecode instruction và interpreter thực thi chúng. citeturn409085search1turn409085search5

---

# 9. ExprTk có sử dụng bytecode không?

Theo tài liệu công khai hiện tại, cơ chế được mô tả chính thức là:

```text
Expression giữ AST
        ↓
Duyệt AST theo post-order
        ↓
Tính kết quả
```

Tài liệu chính thức không mô tả ExprTk như một stack-based bytecode virtual machine và không trình bày một tầng bytecode riêng. Vì vậy, dựa trên tài liệu công khai, cách gọi chính xác hơn là:

> ExprTk là một expression parser và AST evaluator có các bước tối ưu khi compile.

Không nên tự động suy ra:

```text
ExprTk string → AST → bytecode → VM
```

Có thể bạn đã đọc khái niệm bytecode trong tài liệu nói chung về expression evaluator, hoặc từ một thư viện khác. AST và bytecode thường cùng xuất hiện trong thiết kế trình thông dịch, nhưng một thư viện có thể:

```text
String → AST → đánh giá trực tiếp
```

hoặc:

```text
String → AST → bytecode → máy ảo
```

ExprTk được tài liệu chính thức mô tả theo mô hình đầu tiên. citeturn256653view0turn256653view1

---

# 10. ExprTk tối ưu AST như thế nào?

ExprTk không đơn giản chỉ tạo cây rồi giữ nguyên. Thư viện hỗ trợ các tối ưu như:

- constant folding;
- strength reduction;
- dead code elimination. citeturn256653view2turn640461view4

## Constant folding

Các phép tính chỉ chứa hằng số được tính ngay khi compile.

Ví dụ:

```text
x + 2 * 3
```

có thể được biến đổi thành:

```text
x + 6
```

AST ban đầu:

```text
       +
      / \
     x   *
        / \
       2   3
```

AST sau tối ưu:

```text
       +
      / \
     x   6
```

Như vậy mỗi lần gọi `expression.value()`, thư viện không phải tính lại $2\times3$.

Tài liệu ExprTk đưa ví dụ dạng:

```text
2 + (3 - x / y)
```

được rút gọn thành:

```text
5 - x / y
```

khi có thể áp dụng constant folding. 

---

## Strength reduction

Strength reduction biến phép tính thành dạng tương đương nhưng có chi phí thấp hơn hoặc ít nút hơn.

Ví dụ:

```text
(x / y) / z
```

có thể đổi thành:

```text
x / (y * z)
```

Hoặc:

```text
(2 * x) * (3 * y)
```

thành:

```text
6 * (x * y)
```

ExprTk lưu ý rằng một số phép biến đổi đại số có thể ảnh hưởng đến độ chính xác số hoặc nguy cơ tràn số ở gần giới hạn kiểu dữ liệu. Do đó tùy chọn strength reduction có thể được tắt. 

---

## Dead code elimination

Dead code là đoạn mã không ảnh hưởng đến kết quả cuối cùng.

Ví dụ khái niệm:

```text
x := 10;
x := 20;
x
```

Nếu phép gán đầu tiên không có tác dụng phụ và chắc chắn bị ghi đè, nó có thể là mã thừa.

ExprTk có cơ chế phát hiện và loại bỏ một số statement không đóng góp vào kết quả cuối cùng; tối ưu này được bật mặc định theo tài liệu. citeturn640461view4

---

# 11. Các cơ chế khác bên trong ExprTk

Ngoài AST, ExprTk còn sử dụng một số cơ chế đáng chú ý.

## Kiểm tra cú pháp và token

Trước khi xây dựng AST hoàn chỉnh, ExprTk có thể kiểm tra:

- ngoặc có cân bằng hay không;
- token số có hợp lệ không;
- chuỗi toán tử có hợp lệ không;
- độ sâu parser;
- độ sâu AST;
- kích thước expression. citeturn640461view1turn640461view3

Ví dụ:

```text
x + * 3
```

sẽ bị phát hiện là chuỗi token không hợp lệ.

## Semantic analysis

Cú pháp đúng chưa chắc ý nghĩa đã đúng.

Ví dụ:

```text
x + unknown_variable
```

có thể đúng về ngữ pháp nhưng `unknown_variable` không tồn tại trong symbol table.

Hoặc:

```text
sin(x, y)
```

có tên hàm hợp lệ nhưng sai số lượng tham số.

ExprTk phân loại lỗi thành các nhóm như syntax, token, numeric, symbol table và lexer. citeturn640461view0turn640461view1

## Template theo kiểu số

Các lớp thường được tạo theo kiểu:

```cpp
exprtk::expression<double>
exprtk::expression<float>
```

Điều này cho phép cùng một engine hoạt động với các kiểu số khác nhau ở cấp C++ template.

## Hàm do người dùng định nghĩa

Bạn có thể đăng ký hàm C++ vào symbol table:

```cpp
double square(double x)
{
    return x * x;
}
```

sau đó dùng trong biểu thức:

```text
square(x) + 10
```

AST sẽ có một node đại diện cho lời gọi hàm và liên kết nó với hàm đã đăng ký trong symbol table.

---

# 12. So sánh ba cách triển khai

| Cách | Biểu diễn nội bộ | Đặc điểm |
|---|---|---|
| Hàm C++ hard-code | Mã máy sau khi build | Nhanh, nhưng công thức cố định |
| Parser tự viết đơn giản | Thường là token, RPN hoặc AST đơn giản | Dễ kiểm soát nhưng phải tự xử lý rất nhiều trường hợp |
| ExprTk | Token, parser, symbol table, AST và tối ưu | Công thức linh hoạt, tính năng ngôn ngữ phong phú |
| Bytecode VM | Danh sách opcode tuyến tính | Cần compiler bytecode và virtual machine |

Nếu bạn tự viết một thư viện chỉ hỗ trợ:

```text
+, -, *, /
```

thì có thể dùng thuật toán shunting-yard để chuyển biểu thức sang RPN:

```text
2 * x + y
```

thành:

```text
2 x * y +
```

Nhưng khi muốn hỗ trợ thêm:

```text
if
for
while
vector
string
assignment
user-defined function
short-circuit logic
```

thì hệ thống nhanh chóng trở thành một ngôn ngữ lập trình nhỏ. ExprTk đã triển khai sẵn phần lớn cơ sở hạ tầng đó; tài liệu hiện tại liệt kê cả cấu trúc điều khiển, vòng lặp, vector, chuỗi, phép gán và các kiểm tra runtime. citeturn447310view0

---

# 13. Cách sử dụng ExprTk hiệu quả

Mô hình sử dụng nên là:

```text
1. Tạo biến
2. Đăng ký symbol table
3. Compile expression một lần
4. Thay đổi giá trị biến
5. Gọi expression.value() nhiều lần
```

Ví dụ:

```cpp
double x = 0.0;

symbol_table.add_variable("x", x);
expression.register_symbol_table(symbol_table);

parser.compile("sin(x) + x * x", expression);

for (int i = 0; i < 1000; ++i)
{
    x = i * 0.01;
    const double y = expression.value();

    // Sử dụng y...
}
```

Không nên làm như sau trong vòng lặp:

```cpp
for (...)
{
    parser.compile("sin(x) + x * x", expression);
    expression.value();
}
```

bởi vì parser phải lặp lại quá trình token hóa, kiểm tra, xây dựng và tối ưu AST dù công thức không thay đổi.

---

# 14. Mô hình tư duy ngắn gọn nhất

Bạn có thể hình dung ExprTk như sau:

```text
ExprTk = một trình thông dịch ngôn ngữ toán học nhỏ viết bằng C++
```

Trong đó:

```text
Lexer
```

hiểu từng “từ” của công thức.

```text
Parser
```

hiểu cấu trúc và thứ tự ưu tiên.

```text
Symbol table
```

liên kết tên biến và hàm với dữ liệu C++.

```text
AST
```

lưu ý nghĩa của biểu thức dưới dạng cây.

```text
Optimizer
```

rút gọn cây khi có thể.

```text
Evaluator
```

duyệt cây để tính kết quả.

Còn bytecode là một phương án kiến trúc khác:

```text
AST → danh sách opcode → virtual machine
```

nhưng tài liệu công khai hiện tại của ExprTk mô tả việc đánh giá trực tiếp trên AST, không mô tả một máy ảo bytecode riêng.