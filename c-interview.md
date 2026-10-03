# C Interview Notes — Intern Embedded Firmware

> **Mục tiêu:** Ôn phần C cho phỏng vấn Intern Embedded Firmware.  
> **Cách dùng:** Học theo từng chương, dùng mục lục để nhảy nhanh đến chủ đề cần ôn.  
> **Phạm vi:** Giữ nguyên các chủ đề của bản trước, chỉ chuẩn hóa trình bày và sửa bố cục.

<a id="muc-luc"></a>
## Mục lục

> Bấm vào tên chủ đề để chuyển nhanh đến phần cần ôn.

1. [Tổng quan](#chuong-01)
   - [1.1. Hệ thống nhúng là gì?](#muc-01-01)
2. [Quá trình biên dịch (Compiling)](#chuong-02)
   - [2.1. Tiền xử lý — Preprocessing](#muc-02-01)
   - [2.2. Biên dịch — Compilation](#muc-02-02)
   - [2.3. Hợp dịch — Assembly](#muc-02-03)
   - [2.4. Liên kết — Linking](#muc-02-04)
3. [Kiến thức C cơ bản](#chuong-03)
   - [3.1. Ký tự đặc biệt trong C](#muc-03-01)
   - [3.2. Comment trong C](#muc-03-02)
   - [3.3. Các kiểu dữ liệu trong C](#muc-03-03)
   - [3.4. Phương pháp bù 2](#muc-03-04)
   - [3.5. Toán tử `sizeof`](#muc-03-05)
   - [3.6. Biến](#muc-03-06)
4. [Hàm](#chuong-04)
   - [4.1. Khái niệm hàm](#muc-04-01)
   - [4.2. Định nghĩa, khai báo và khởi tạo](#muc-04-02)
   - [4.3. Đối số (Arguments) và tham số (Parameters)](#muc-04-03)
   - [4.4. Truyền giá trị và truyền địa chỉ bằng con trỏ](#muc-04-04)
   - [4.5. Hàm Variadic](#muc-04-05)
5. [Scope và Storage Class](#chuong-05)
   - [5.1. Phạm vi của biến (Variable Scope)](#muc-05-01)
   - [5.2. Lớp lưu trữ (Storage Classes)](#muc-05-02)
6. [Chuyển đổi kiểu và địa chỉ](#chuong-06)
   - [6.1. Ép kiểu (Conversion)](#muc-06-01)
   - [6.2. Địa chỉ của biến](#muc-06-02)
7. [Điều khiển luồng](#chuong-07)
   - [7.1. `if-else` và `switch-case`](#muc-07-01)
8. [Tổ chức bộ nhớ](#chuong-08)
   - [8.1. Memory layout](#muc-08-01)
   - [8.2. Các section](#muc-08-02)
9. [Vòng lặp](#chuong-09)
   - [9.1. `for`, `while`, `do-while`](#muc-09-01)
   - [9.2. `continue`](#muc-09-02)
   - [9.3. `break`](#muc-09-03)
10. [Mảng](#chuong-10)
   - [10.1. Mảng một chiều (Array)](#muc-10-01)
   - [10.2. Mảng hai chiều (2-D Array)](#muc-10-02)
   - [10.3. Mảng động và cấp phát bộ nhớ](#muc-10-03)
11. [Toán tử](#chuong-11)
   - [11.1. Các nhóm toán tử](#muc-11-01)
   - [11.2. Thao tác bit](#muc-11-02)
12. [Từ khóa quan trọng](#chuong-12)
   - [12.1. `volatile`](#muc-12-01)
   - [12.2. `const`](#muc-12-02)
13. [Kiểu dữ liệu tự định nghĩa](#chuong-13)
   - [13.1. `struct`](#muc-13-01)
   - [13.2. Trường bit (Bit fields)](#muc-13-02)
   - [13.3. `union`](#muc-13-03)
14. [Chuỗi ký tự (String)](#chuong-14)
15. [Con trỏ (Pointer)](#chuong-15)
   - [15.1. Khái niệm cơ bản](#muc-15-01)
   - [15.2. Dangling pointer sau `free()`](#muc-15-02)
   - [15.3. Dangling pointer khi trả về biến cục bộ](#muc-15-03)
   - [15.4. Dangling pointer khi biến ra khỏi phạm vi](#muc-15-04)
   - [15.5. `const` với con trỏ](#muc-15-05)
   - [15.6. Con trỏ với mảng](#muc-15-06)
   - [15.7. Mảng con trỏ đến chuỗi ký tự](#muc-15-07)
   - [15.8. Mảng con trỏ `void *`](#muc-15-08)
   - [15.9. Con trỏ hàm](#muc-15-09)
   - [15.10. Mảng con trỏ hàm](#muc-15-10)
   - [15.11. Con trỏ trong hàm](#muc-15-11)
   - [15.12. Con trỏ hàm làm callback](#muc-15-12)
   - [15.13. Thay đổi biến thông qua con trỏ](#muc-15-13)
   - [15.14. Truy cập thành viên struct bằng con trỏ](#muc-15-14)
   - [15.15. Con trỏ struct trong tham số hàm](#muc-15-15)
   - [15.16. Con trỏ hàm là member của struct](#muc-15-16)
16. [Các chủ đề C bổ sung](#chuong-16)
   - [16.1. `typedef`](#muc-16-01)
   - [16.2. Big Endian và Little Endian](#muc-16-02)
   - [16.3. `enum`](#muc-16-03)
   - [16.4. Overflow](#muc-16-04)
   - [16.5. Internal Linkage và External Linkage](#muc-16-05)
   - [16.6. Header guard](#muc-16-06)
   - [16.7. `static inline` function](#muc-16-07)
   - [16.8. Memory functions](#muc-16-08)

---

<a id="chuong-01"></a>
## 1. Tổng quan

<a id="muc-01-01"></a>
### 1.1. Hệ thống nhúng là gì?

- Hệ thống nhúng là một hệ thống điện tử hoặc máy tính tích hợp cả phần cứng và phần mềm, được thiết kế để thực hiện một chức năng chuyên biệt hoặc một nhóm nhiệm vụ cụ thể trong một hệ thống lớn hơn.

- **Đặc tính chung:**
  - Kích thước hệ thống nhỏ.
  - Giá thành rẻ.
  - Tiêu thụ ít năng lượng.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-02"></a>
## 2. Quá trình biên dịch (Compiling)

- Quá trình biên dịch là quá trình chuyển đổi từ ngôn ngữ bậc cao sang ngôn ngữ máy (machine code) để máy tính có thể hiểu và thực thi.

- Quá trình bao gồm 4 giai đoạn chính:
  1. Tiền xử lý (Preprocessing).
  2. Biên dịch (Compilation).
  3. Hợp dịch (Assembly).
  4. Liên kết (Linking).

<a id="muc-02-01"></a>
### 2.1. Tiền xử lý — Preprocessing

- **Bộ thực hiện:** Preprocessor.

- **Đầu vào:** file source `.c`.

- **Nhiệm vụ:**

  - Xóa các comment.

  - Xử lý các chỉ thị bắt đầu bằng **#**.

  - Xử lý **#include**.

  - Xử lý nội dung của file được include vào vị trí **#include**.

  - Xử lý **#define**.

  - Thay thế macro.

  - Xử lý **#if**, **#ifdef**, **#ifndef**, **#elif**, **#else**, **#endif**.

  - Xử lý các macro predefined như:

    - **__FILE__**
    - **__LINE__**
    - **__DATE__**
    - **__TIME__**

  - Không thực hiện kiểm tra cú pháp C đầy đủ.

- **Kết quả:** Tạo ra file `.i`.

Ví dụ:

```c
#include "gpio.h"
#define LED_PIN 5
int main(void)
{
    gpio_write(LED_PIN, 1);
}
```

Sau preprocessing, nội dung **gpio.h** và giá trị macro **LED_PIN** sẽ được thay thế vào source.

<a id="muc-02-02"></a>
### 2.2. Biên dịch — Compilation

- **Bộ thực hiện:** Compiler.

- **Đầu vào:** File `.i`.

- **Nhiệm vụ:**

  - Phân tích cú pháp.

  - Kiểm tra kiểu dữ liệu.

  - Kiểm tra khai báo biến, hàm.

  - Kiểm tra phép toán và biểu thức.

  - Phát hiện các lỗi: lỗi cú pháp, biến/hàm chưa khai báo, sai kiểu dữ liệu, gọi hàm sai tham số, ...

  - Sinh warning nếu phát hiện code đáng nghi.

  - Thực hiện tối ưu hóa tùy vào compiler flag:

    - `-O0`
    - `-O1`
    - `-O2`
    - `-O3`
    - `-Os`
    - `-Og`

  - Chuyển source C thành mã assembly phù hợp với kiến trúc đích.

Ví dụ:

`arm-none-eabi-gcc` sẽ sinh assembly dành cho ARM Cortex-M chứ không phải CPU của máy tính đang compile.

- **Kết quả:** Tạo ra file assembly `.s`.

Ví dụ C:

```c
int add(int a, int b)
{
    return a + b;
}
```

có thể được compiler chuyển thành assembly ARM tương ứng.

<a id="muc-02-03"></a>
### 2.3. Hợp dịch — Assembly

- **Bộ thực hiện:** Assembler.

- **Đầu vào:** File `.s`.

- **Nhiệm vụ:**

  - Chuyển mã Assembly thành mã máy mà CPU có thể thực thi.
  - Tạo ra file object chứa mã máy và thông tin cần thiết để linker xử lý ở bước sau.
  - Nếu bật chế độ debug, lưu thêm thông tin phục vụ việc debug.

- **Kết quả:** Tạo ra object file `.o`.

Ví dụ:

```text
main.s
  ↓
Assembler
  ↓
main.o
```

File `.o` **chưa phải firmware hoàn chỉnh**. Nó có thể vẫn chứa các symbol chưa được giải quyết.

Ví dụ:

**gpio_write**(...)**;**

Trong main.o, lời gọi gpio_write() có thể tồn tại dưới dạng một tham chiếu tới symbol chưa được giải quyết. Đến bước Linking, linker sẽ tìm definition của symbol này và xác định địa chỉ phù hợp.

<a id="muc-02-04"></a>
### 2.4. Liên kết — Linking

- **Bộ thực hiện:** Linker.

- **Đầu vào:** Các file object `.o` và các thư viện cần thiết.

- **Nhiệm vụ:**

  - Ghép các phần của chương trình đã được biên dịch lại với nhau.

  - Tìm và nối các hàm, biến được sử dụng ở file này nhưng được định nghĩa ở file khác.

  - Liên kết thêm các thư viện mà chương trình sử dụng.

  - Phát hiện các lỗi như:

    - Gọi một hàm nhưng không tìm thấy phần định nghĩa (**undefined reference**).
    - Một hàm hoặc biến bị định nghĩa nhiều lần không hợp lệ (multiple definition).

- **Kết quả:** Tạo ra file thực thi/chương trình đã được liên kết hoàn chỉnh, ví dụ .elf.

#### Ví dụ

File **main.c**:

```c
int add(int a, int b);
int main(void)
{
    return add(1, 2);
}
```

File **add.c**:

```c
int add(int a, int b)
{
    return a + b;
}
```

Khi biên dịch riêng **main.c**, compiler biết rằng hàm **add()** tồn tại nhờ declaration:

```c
int add(int a, int b);
```

Nhưng phần code thật của **add()** nằm trong **add.c**. Đến bước Linking, linker sẽ nối lời gọi **add()** trong **main.c** với phần định nghĩa **add()** trong **add.c**. Nếu không tìm thấy phần định nghĩa của **add()**, quá trình linking sẽ bị lỗi.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-03"></a>
## 3. Kiến thức C cơ bản

<a id="muc-03-01"></a>
### 3.1. Ký tự đặc biệt trong C

- Ký tự đặc biệt hay còn gọi là escape sequence, hoặc là chuỗi thoát trong C, là các chuỗi ký tự bắt đầu bởi dấu gạch chéo ngược như \n hay \a , nhằm biểu diễn các ký tự vốn không thể biểu diễn theo cách thông thường trong C.

- Ví dụ:

\n: Di chuyển con trỏ tới đầu dòng theo chiều dọc.

\r di chuyển con trỏ tới đầu dòng theo chiều ngang.

<a id="muc-03-02"></a>
### 3.2. Comment trong C

- Cách comment trên một dòng: //

- Cách comment trên nhiều dòng: /\*\*/

<a id="muc-03-03"></a>
### 3.3. Các kiểu dữ liệu trong C

- Kiểu dữ liệu là phần xác định khoảng giá trị mà một biến có thể lưu trữ hay giá trị mà một hàm có thể trả về.

- Kiểu dữ liệu của một biến xác định kích thước (số byte) của biến đó.

- Có 4 kiểu dữ liệu:

- Kiểu cơ bản:

  - Kiểu số nguyên (Integer types): char, signed char, unsigned char, short, int, long, long long (và các dạng unsigned tương ứng).
  - Kiểu số thực dấu phẩy động (Floating types): float, double, long double.

- Kiểu liệt kê: Từ khóa enum. (Bản chất vẫn là lưu số nguyên nhưng được tách riêng thành một nhóm).

- Kiểu trống rỗng: Từ khóa void. Biểu thị một tập hợp giá trị rỗng, dùng để định nghĩa hàm không trả về dữ liệu hoặc tạo con trỏ tổng quát void\*.

- Kiểu dẫn xuất: Đây là nhóm được xây dựng từ các kiểu dữ liệu khác, bao gồm:

  - Mảng (Array types)
  - Cấu trúc (Structure types - struct)
  - Hợp thể (Union types - union)
  - Hàm (Function types)
  - Con trỏ (Pointer types)

- Kích thước của kiểu dữ liệu tùy thuộc vào trình biên dịch, kiến trúc máy tính.

- Kiểm tra kích thước của kiểu dữ liệu sử dụng toán tử sizeof.

#### Kiểu `char`

#### `short` và `unsigned short`

#### `int` và `unsigned int`

#### `long` và `unsigned long`

#### `long long` và `unsigned long long`

#### Kiểu dấu phẩy động: `float`, `double`, `long double`

- Dấu phẩy động: Là phương pháp biểu diễn số thực mà trong đó vị trí của dấu phẩy thập phân có thể di chuyển linh hoạt.

- So sánh với dấu phẩy tĩnh: cố định hoàn toàn (Ví dụ: quy định luôn có 2 chữ số sau dấu phẩy).

- Hầu hết các bộ vi xử lý và trình biên dịch hiện này đều tuân theo chuẩn IEEE 754 để mã hóa số thực. Một số thực sẽ được chia thành 3 phần chính trong bộ nhớ:

- S (Sign): Dấu của số thực: 0 là dương, 1 là âm.
- E (Exponent): Phần mũ với cơ số 2.
- M (Mantissa/Fraction): Phần định trị, dùng để biểu diễn độ chính xác của số.

- float thường dùng IEEE 754 binary32:

1 bit Sign.

8 bit Exponent.

23 bit Fraction.

- double thường dùng IEEE 754 binary64:

1 bit Sign.

11 bit Exponent.

52 bit Fraction.

<a id="muc-03-04"></a>
### 3.4. Phương pháp bù 2

- Định nghĩa: Là phương pháp toán học chuẩn và phổ biến nhất dùng để biểu diễn và tính toán số nguyên có dấu.

- Bước 1 (Đảo bit - Bù 1): Đổi tất cả các bit 0 thành 1 và 1 thành 0.
- Bước 2 (Cộng 1): Cộng thêm 1 vào kết quả vừa tìm được ở Bước 1.

- Ví dụ: Tìm mã nhị phân của số -5

Số 5 gốc dạng dương: 0000 0101

Bước 1 (Đảo bit): 1111 1010

Bước 2 (Cộng thêm 1): 1111 1010 + 1 = 1111 1011

Kết quả: Trong biểu diễn bù 2 8-bit, `-5` có mẫu bit `1111 1011`.

- Tại sao dùng? gom phép trừ về phép cộng: thay vì 7 - 5 thì sẽ là 7 + (-5).

<a id="muc-03-05"></a>
### 3.5. Toán tử `sizeof`

- Toán tử sizeof cung cấp thông tin về kích thước của các kiểu dữ liệu.

<a id="muc-03-06"></a>
### 3.6. Biến

- Biến là vùng nhớ được đặt tên trên bộ nhớ máy tính (RAM) dùng để lưu trữ dữ liệu.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-04"></a>
## 4. Hàm

<a id="muc-04-01"></a>
### 4.1. Khái niệm hàm

- Hàm là một khối mã lệnh độc lập được đặt tên, dùng để thực hiện một nhóm nhiệm vụ hoặc một chức năng cụ thể nào đó trong chương trình.

- Tại sao phải dùng hàm?

- Code trở nên dễ đoc, mạch lạc
- Dễ debug khi gặp lỗi
- Dễ bảo trì khi cần thay đổi một chức năng
- Có khả năng tái sử dụng lại code

- Hàm có 4 thành phần cấu tạo:

- Kiểu dữ liệu trả về: Định nghĩa loại dữ liệu mà hàm sẽ trả ra sau khi xử lý xong (như int, float, char). Nếu hàm không trả về gì cả, kiểu dữ liệu sẽ là void.
- Tên hàm: Định danh dùng để gọi hàm khi cần sử dụng (Ví dụ: add, calculate_speed).
- Tham số đầu vào: Danh sách các biến nhận dữ liệu truyền từ ngoài vào để hàm xử lý (nằm trong dấu ngoặc đơn ()). Nếu hàm không cần tham số, ta để trống hoặc ghi void.
- Thân hàm: Tập hợp các câu lệnh thực thi logic được bao bọc trong cặp dấu ngoặc nhọn {}.

- Lời gọi hàm:

- Khi gọi hàm trong main, câu lệnh bên trong hàm sẽ được thực thi.
- Ví dụ:

```c
#include <stdio.h>
long long multiply(int x, int y)
{
    return x * y;
}
int main()
{
    long long a = 10, b = 30;
    int result = multiply(a, b);
    printf("result is:%d\n", result);
    return 0;
}
```

Lỗi:

Lỗi kiểu dữ liệu truyền vào: Biến a và b ở hàm main có kiểu là long long (64-bit), nhưng hàm multiply lại khai báo nhận vào tham số kiểu int (32-bit). Khi bạn gọi multiply(a, b), hệ thống sẽ ép kiểu ngầm định từ lớn xuống nhỏ (Downcasting/Truncation). Nếu giá trị của a hoặc b cực kỳ lớn (vượt quá giới hạn của int), dữ liệu sẽ bị cắt cụt và kết quả tính toán sẽ sai hoàn toàn.

Lỗi tính toán bên trong hàm: Câu lệnh return x \* y; bên trong hàm. Vì cả x và y đều là kiểu int, máy tính sẽ thực hiện phép nhân theo kiểu int trước. Nếu tích của x \* y vượt quá giới hạn của kiểu int (khoảng hơn 2 tỷ), hiện tượng tràn số (Integer Overflow) sẽ xảy ra trước khi nó kịp chuyển đổi thành kiểu long long để trả về.

Lỗi kiểu dữ liệu trả về: Hàm multiply được thiết kế để trả về một giá trị lớn kiểu long long (64-bit), nhưng ở hàm main, bạn lại dùng biến int result (32-bit) để hứng kết quả. Việc này làm mất hoàn toàn ý nghĩa của việc dùng long long ở hàm trên. Kết quả sau khi nhân xong dẫu có lớn và chuẩn đến đâu thì khi gán vào result cũng bị ép nhỏ lại thành int, gây nguy cơ tràn số một lần nữa.

Sửa:

```c
#include <stdio.h>
// 1. Sửa chữ L viết thường, đồng bộ tham số đầu vào thành long long
long long multiply(long long x, long long y)
{
    return x * y;
}
int main()
{
    long long a = 10, b = 30;
    // 2. Biến hứng kết quả phải là long long
    long long result = multiply(a, b);

    // 3. Sửa định dạng in từ %d thành %lld dành cho kiểu long long
    printf("result is:%lld\n", result);
    return 0;
}
```

- Khai báo nguyên mẫu hàm là cách cung cấp thông tin trước cho trình biên dịch về tên hàm, kiểu trả về và danh sách tham số của hàm mà chưa viết phần thân bên trong.

- Nếu không khai báo nguyên mẫu hàm, trình biên dịch (compiler) sẽ không nhận diện được hàm nếu bạn gọi hàm đó trước khi định nghĩa nó, dẫn đến lỗi biên dịch (Compile Error).

- Ví dụ:

```c
int sum(int a, int b); // Khai báo có tên tham số
int sum(int, int); // Khai báo không có tên tham số
```

<a id="muc-04-02"></a>
### 4.2. Định nghĩa, khai báo và khởi tạo

- Định nghĩa là việc yêu cầu hệ thống cấp phát vùng nhớ và xác định đầy đủ nội dung hoặc kiểu dữ liệu của đối tượng đó.

- Ví dụ:

```c
• int x; (Hệ thống cấp phát 4 bytes trong RAM cho x)
• int tinhTong(int a, int b) { return a + b; } (Xác định phần thân/nội dung xử lý của hàm)
```

- Khai báo là việc thông báo cho trình biên dịch biết sự tồn tại của một tên gọi (biến, hàm) và kiểu dữ liệu của nó để chương trình nhận diện, nhưng chưa cấp phát vùng nhớ (hoặc vùng nhớ đã được cấp ở nơi khác).

- Ví dụ:

```c
• extern int x; (Báo rằng biến x tồn tại ở file khác, file này chỉ dùng ké, không cấp bộ nhớ)
• int tinhTong(int a, int b); (Nguyên mẫu hàm - thông báo hàm này có tồn tại, phần thân ở chỗ khác)
```

- Khởi tạo là việc gán giá trị đầu tiên cho biến ngay tại thời điểm biến đó được định nghĩa (cấp phát vùng nhớ).

- Ví dụ:

|                                                                     |
|---------------------------------------------------------------------|
| int x = 10; (Vừa cấp bộ nhớ cho x, vừa nạp giá trị 10 vào ô nhớ đó) |

<a id="muc-04-03"></a>
### 4.3. Đối số (Arguments) và tham số (Parameters)

- Đối số hay còn gọi là tham số thực tế là giá trị thực tế được truyền vào hàm khi lời gọi hàm được thực hiện.

- Tham số hay còn gọi là tham số hình thức là biến được khai báo trong phần định nghĩa của hàm trong dấu (), dùng để nhận các giá trị tương ứng từ các đối số.

- Ví dụ:

```c
#include <stdio.h>
// Hai biến 'a' và 'b' ở đây là tham số (Parameters)
int tinhTong(int a, int b) {
    return a + b;
}
int main() {
    int x = 10;
    int y = 20;
    // Số '5', '3' hay biến 'x', 'y' ở đây là đối số (Arguments)
    int kq1 = tinhTong(5, 3);
    int kq2 = tinhTong(x, y);
    return 0;
}
```

<a id="muc-04-04"></a>
### 4.4. Truyền giá trị và truyền địa chỉ bằng con trỏ

- Trong C, **mọi đối số đều được truyền theo giá trị (pass by value)**. Nghĩa là hàm nhận một bản sao của giá trị được truyền vào.
- Nếu truyền một biến bình thường, thay đổi tham số trong hàm sẽ không làm thay đổi biến gốc.
- Nếu muốn hàm thay đổi biến gốc, ta truyền **địa chỉ của biến bằng con trỏ**. Bản thân địa chỉ này vẫn được truyền theo giá trị, nhưng thông qua con trỏ ta có thể truy cập và sửa dữ liệu ở biến gốc.

Ví dụ:

```c
#include <stdio.h>

void tangThamTri(int x)
{
    x = x + 10;            // Chỉ thay đổi bản sao
}

void tangQuaConTro(int *x)
{
    *x = *x + 10;          // Thay đổi biến gốc thông qua địa chỉ
}

int main(void)
{
    int n = 5;

    tangThamTri(n);
    printf("Sau tham tri: %d\n", n);       // 5

    tangQuaConTro(&n);
    printf("Sau con tro: %d\n", n);        // 15

    return 0;
}
```

So sánh ngắn:

| Cách truyền | Ý nghĩa | Khi nào dùng |
|---|---|---|
| Truyền giá trị | Hàm nhận bản sao của dữ liệu | Khi không muốn hàm sửa dữ liệu gốc |
| Truyền địa chỉ bằng con trỏ | Hàm nhận bản sao của địa chỉ và có thể sửa dữ liệu tại địa chỉ đó | Khi cần sửa biến gốc hoặc tránh sao chép dữ liệu lớn |

<a id="muc-04-05"></a>
### 4.5. Hàm Variadic

- Hàm Variadic là hàm cho phép bạn truyền vào số lượng đối số không cố định (muốn truyền bao nhiêu đối số tùy ý) khi gọi hàm. Ví dụ: printf(), scanf(), ...

- Hàm biến đổi (variadic) có ít nhất một đối số cố định, sau đó là dấu ba chấm (...) để cho phép truyền số lượng đối số thay đổi. Trình biên dịch sử dụng dấu ba chấm này để xử lý các đối số thêm vào.

- Các thành phần cơ bản:

va_list: kiểu dữ liệu dùng để lưu trữ danh sách các đối số

va_start: truy cập danh sách đối số (cần truyền vào tham số cố định cuối cùng)

va_arg: truy cập từng đối số trong danh sách các đối số (phải chỉ định rõ kiểu dữ liệu cần lấy)

va_copy: sao chép một biến kiểu va_list từ biến này sang biến khác

va_end: giải phóng và kết thúc việc đọc danh sách các đối số

```c
#include <stdio.h>
#include <stdarg.h> // Bắt buộc phải có thư viện này
// Hàm variadic tính tổng
int tinhTongNhieuSo(int count, ...) {
    va_list args;      // 1. Khai báo danh sách đối số
    int tong = 0;
    va_start(args, count); // 2. Khởi tạo danh sách đối số, lấy 'count' làm mốc
    for (int i = 0; i < count; i++) {
        // 3. Lấy từng đối số ra dưới dạng kiểu 'int' và cộng dồn
        tong += va_arg(args, int);
    }
    va_end(args);      // 4. Dọn dẹp bộ nhớ sau khi dùng xong
    return tong;
}
int main() {
    // Gọi hàm với 3 đối số
    printf("Tong 3 so: %d\n", tinhTongNhieuSo(3, 10, 20, 30)); // Kq: 60
    // Gọi hàm với 5 đối số
    printf("Tong 5 so: %d\n", tinhTongNhieuSo(5, 1, 2, 3, 4, 5)); // Kq: 15
    return 0;
}
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-05"></a>
## 5. Scope và Storage Class

<a id="muc-05-01"></a>
### 5.1. Phạm vi của biến (Variable Scope)

#### Biến cục bộ (Local Variable)

- Định nghĩa: Là biến được khai báo bên trong một hàm hoặc bên trong một khối lệnh (nằm giữa cặp ngoặc nhọn {}).

- Phạm vi (Scope): Chỉ có thể truy cập và sử dụng bên trong chính hàm hoặc khối lệnh chứa nó. Các hàm khác bên ngoài không thể nhìn thấy biến này.

- Vòng đời (Lifetime): Với biến cục bộ có automatic storage duration, thời gian tồn tại bắt đầu khi đi vào block và kết thúc khi rời block. Trong thực tế biến có thể được compiler đặt trên stack hoặc register tùy cách tối ưu.

- Nếu không được khởi tạo, giá trị của biến cục bộ là **không xác định (indeterminate)**; không nên đọc giá trị trước khi gán.

#### Biến toàn cục (Global Variable)

- Định nghĩa: Là biến được khai báo bên ngoài tất cả các hàm và khối lệnh (thường được đặt ở ngay đầu file mã nguồn).

- Phạm vi (Scope): Có phạm vi toàn bộ file. Tất cả các hàm nằm phía dưới vị trí khai báo đều có thể tự do truy cập, đọc và chỉnh sửa giá trị của biến này.

- Vòng đời (Lifetime): Sinh ra ngay khi chương trình bắt đầu chạy và tồn tại suốt vòng đời của chương trình cho đến khi chương trình bị kết thúc.

- Mặc định được khởi tạo với giá trị 0. (Giải thích: biến toàn cục không được khởi tạo sẽ nằm trên vùng BSS, được nạp giá trị bằng 0 trước khi vào main()).

```c
#include <stdio.h>
int global_num = 5; // Biến toàn cục
void Sum(int a, int b){
    int sum = a + b; // Biến cục bộ
    printf("Sum = %d\n", sum);
}
int main(){
    int local_num = 10;
    Sum(global_num, local_num);
}
```

- Hiện tượng che biến (Variable Shadowing) là hiện tượng xảy ra khi một biến được khai báo bên trong một phạm vi hẹp (trong hàm, trong vòng lặp, trong phạm vi {}, ... ) có cùng tên với một biến ở phạm vi rộng hơn bên ngoài. Lúc này, biến ở phạm vi hẹp sẽ che biến bên ngoài, khiến chương trình ưu tiên truy cập vào biến bên trong.

- Ví dụ:

```c
#include <stdio.h>
int valueA = 5;
int main()
{
  {
    int valueA = 20;
    printf("valueA: %d \n", valueA); // Kết quả: 20
  }
   printf("valueA: %d \n", valueA); // Kết quả: 5
}
```

<a id="muc-05-02"></a>
### 5.2. Lớp lưu trữ (Storage Classes)

- Storage class giúp mô tả **phạm vi sử dụng, thời gian tồn tại và linkage** của biến hoặc hàm. Vị trí vật lý thực tế của biến còn phụ thuộc compiler, kiến trúc và mức tối ưu.

#### `auto`

- Là storage class mặc định của biến cục bộ thông thường.
- Biến có block scope và automatic storage duration: được tạo khi đi vào block và hết thời gian tồn tại khi rời block.
- C standard **không bắt buộc** biến `auto` phải nằm trên Stack, dù trong thực tế compiler thường dùng stack hoặc register tùy trường hợp.
- Nếu không khởi tạo, giá trị của biến cục bộ là **không xác định (indeterminate)**.

```c
int x;
// tương đương về storage class với:
auto int x;
```

#### `register`

- Là lời gợi ý rằng biến được sử dụng thường xuyên và compiler có thể tối ưu cách lưu trữ/truy cập biến.
- Compiler **không bắt buộc** phải đặt biến vào thanh ghi CPU.
- Phạm vi và thời gian tồn tại tương tự biến `auto`.
- Không được dùng toán tử `&` để lấy địa chỉ của biến khai báo `register`.

```c
register int i;
```

#### `static`

- Với biến cục bộ: biến có phạm vi cục bộ nhưng tồn tại suốt thời gian chạy của chương trình, nên giá trị được giữ lại giữa các lần gọi hàm.
- Với biến/hàm ở file scope: `static` còn tạo **internal linkage**, nghĩa là tên đó chỉ được dùng trong translation unit hiện tại.
- Biến `static` không được khởi tạo tường minh sẽ được khởi tạo bằng 0.

Ví dụ biến `static` cục bộ:

```c
void counter(void)
{
    static int count = 0;
    count++;
}
```

#### `extern`

- Dùng để khai báo rằng một biến hoặc hàm có definition ở nơi khác.
- `extern` thường được dùng để truy cập một đối tượng có **external linkage** từ file khác.
- Bản thân declaration `extern` thường không tạo thêm một vùng lưu trữ mới cho biến.

```c
extern int global_score;
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-06"></a>
## 6. Chuyển đổi kiểu và địa chỉ

<a id="muc-06-01"></a>
### 6.1. Ép kiểu (Conversion)

- Ép kiểu tường minh: là việc lập trình viên chủ động viết mã lệnh để ép hệ thống phải chuyển đổi một kiểu dữ liệu này sang kiểu dữ liệu khác theo ý muốn.

```c
double pi = 3.14159;
int s1 = (int)pi; // s1 = 3 (Bị cắt bỏ hoàn toàn phần thập phân .14159)
```

- Ép kiểu ngầm định: là việc trình biên dịch tự động chuyển đổi một biến từ kiểu dữ liệu này sang kiểu dữ liệu khác khi thực hiện tính toán hoặc gán giá trị, mà không cần lập trình viên phải viết bất kỳ câu lệnh ép kiểu nào.

- Ví dụ:

```c
int a = 5;
double b = a; // Trình biên dịch tự chuyển 5 (int) thành 5.0 (double)
float c = 4.5f;
double ketQua = a + c; // a tự động được nâng lên float để cộng với c
```

<a id="muc-06-02"></a>
### 6.2. Địa chỉ của biến

- Địa chỉ biểu diễn vị trí của một object trong không gian địa chỉ của chương trình.

- Dùng toán tử `&` để lấy địa chỉ của một object khi phép lấy địa chỉ đó hợp lệ.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-07"></a>
## 7. Điều khiển luồng

<a id="muc-07-01"></a>
### 7.1. `if-else` và `switch-case`

|                 |                                                                                                             |                                                                                                        |
|-----------------|-------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| Tiêu chí        | if-else                                                                                                     | switch-case                                                                                            |
| Loại điều kiện  | Có thể dùng các biểu thức điều kiện và kiểm tra giá trị khác 0 / bằng 0. | So sánh giá trị của biểu thức `switch` với các nhãn `case` là hằng số nguyên phù hợp. |
| Kiểu dữ liệu    | Biểu thức điều kiện phải có kiểu phù hợp để đánh giá đúng/sai, thường là kiểu scalar. | Biểu thức `switch` dùng kiểu integer hoặc enum. |
| Số lượng nhánh  | Phù hợp với số nhánh ít. Nếu quá nhiều nhánh sẽ khiến code bị rối.                                          | Rất gọn gàng và dễ đọc kể cả khi có hàng chục nhánh (case).                                            |
| Tốc độ thực thi | Phụ thuộc vào điều kiện và cách compiler tối ưu. | Cũng phụ thuộc compiler. Compiler có thể tạo chuỗi so sánh hoặc jump table; không thể khẳng định `switch` luôn nhanh hơn `if-else`. |

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-08"></a>
## 8. Tổ chức bộ nhớ

<a id="muc-08-01"></a>
### 8.1. Memory layout

- Text/Code Segment: Thường chứa mã máy/lệnh thực thi của chương trình.

- Data Segment: Lưu trữ các biến toàn cục (global variables) và biến tĩnh (static variables) đã được khởi tạo.

- BSS Segment: Lưu trữ các biến toàn cục (global variables) và biến tĩnh (static variables) không được khởi tạo.

- Heap: Dùng cho việc cấp phát bộ nhớ động (dùng các hàm như malloc(), calloc(), free()).

- Stack:

- Biến cục bộ (Local Variables): Các biến được khai báo bên trong hàm hoặc khối mã, chỉ tồn tại trong phạm vi hàm hoặc khối đó.
- Con trỏ khung (Frame Pointer): Lưu vị trí bắt đầu của khung stack hiện tại, giúp dễ dàng truy cập các biến và tham số của hàm.
- Địa chỉ trả về (Return Address): Lưu địa chỉ của lệnh tiếp theo sau khi hàm gọi kết thúc, để chương trình có thể quay lại đúng vị trí sau khi thực hiện xong hàm.
- Tham số hàm (Function Parameters): Các giá trị được truyền vào khi gọi hàm, giúp thực hiện các phép tính và thao tác trong hàm.
- Lưu trữ thông tin điều khiển: Bao gồm các thông tin về trạng thái của CPU, các thanh ghi và con trỏ chương trình, cần thiết để phục hồi trạng thái của chương trình sau khi hàm kết thúc.

- Memory Leak: là lỗi xảy ra khi một chương trình xin cấp phát bộ nhớ từ vùng nhớ Heap nhưng không giải phóng sau khi không còn sử dụng. Điều này làm giảm dung lượng RAM của hệ thống, khiến chương trình chạy chậm lại hoặc có thể dẫn đến treo, lỗi chương trình.

- Các nguyên nhân:

- Quên không giải phóng free().
- Con trỏ không tham chiếu đúng khiến vùng nhớ không thể giải phóng.
- Sử dụng nhiều hàm cấp phát mà không có kế hoạch giải phóng.

- Cách khắc phục:

- Luôn luôn dùng free() tương ứng sau khi cấp phát.
- Dùng các công cụ kiểm tra tự động như Valgrind, …

<a id="muc-08-02"></a>
### 8.2. Các section

.text: chứa mã máy, đây là phần cốt lõi của Text Segment. Section này có thuộc tính chỉ đọc và thực thi.

.rodata: chứa các dữ liệu hằng số chỉ đọc.

.data: chứa các biến toàn cục (global) và biến tĩnh (static) đã được khởi tạo.

.bss: chứa các biến toàn cục (global) và biến tĩnh (static) chưa được khởi tạo hoặc được khởi tạo giá trị bằng 0.

Ngoài ra, còn có section:

.symtab: chứa bảng ký hiệu (symbol table) để linker biết và xử lý các tên hàm biến và địa chỉ của chúng.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-09"></a>
## 9. Vòng lặp

<a id="muc-09-01"></a>
### 9.1. `for`, `while`, `do-while`

|          |                                                                            |
|----------|----------------------------------------------------------------------------|
| Vòng lặp | Khi nào nên dùng?                                                          |
| for      | Khi biết trước số lần lặp cụ thể (duyệt mảng, chạy N lần).                 |
| while    | Khi chưa biết trước số lần lặp, cần kiểm tra điều kiện trước rồi mới làm.  |
| do-while | Khi chưa biết trước số lần lặp, nhưng hành động phải chạy ít nhất một lần. |

<a id="muc-09-02"></a>
### 9.2. `continue`

- continue được sử dụng bên trong các vòng lặp (for, while, do-while) để bỏ qua tất cả các câu lệnh còn lại trong lần lặp hiện tại và ngay lập tức nhảy sang lần lặp kế tiếp.

<a id="muc-09-03"></a>
### 9.3. `break`

- break được sử dụng để chấm dứt và thoát hẳn ra khỏi vòng lặp (for, while, do-while) hoặc khối lệnh switch-case ngay lập tức, bất chấp điều kiện vòng lặp có còn đúng hay không.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-10"></a>
## 10. Mảng

<a id="muc-10-01"></a>
### 10.1. Mảng một chiều (Array)

- Mảng là các cấu trúc dữ liệu lưu trữ nhiều giá trị cùng kiểu dữ liệu.

- Các phần tử trong mảng được lưu trữ tại các ô nhớ cạnh nhau trong bộ nhớ.

- Tại sao cần mảng?

- Quản lý dữ liệu lớn bằng một tên duy nhất: Thay vì tạo ra hàng trăm biến khác nhau, mảng cho phép bạn gộp tất cả các dữ liệu cùng kiểu vào một nơi và đặt cho nó một cái tên duy nhất. Ví dụ: int diem\[30\]; chỉ một dòng code, bạn đã có ngay 30 ô nhớ để lưu điểm.
- Dễ dàng kết hợp với vòng lặp (for, while): Vì các phần tử trong mảng được đánh số chỉ số (index) tăng dần từ 0 đến n-1, bạn có thể dùng vòng lặp để duyệt qua hàng nghìn dữ liệu chỉ với vài dòng code.
- Tối ưu hóa bộ nhớ (Lưu trữ liên tục): Trong bộ nhớ RAM, các phần tử của mảng được xếp nằm sát cạnh nhau liên tiếp.
- Code sạch sẽ, dễ bảo trì và nâng cấp: Khi số lượng dữ liệu thay đổi (ví dụ lớp tăng từ 30 lên 50 học sinh), bạn chỉ cần sửa kích thước mảng từ diem\[30\] thành diem\[50\]. Toàn bộ các đoạn code xử lý bên dưới (như vòng lặp) vẫn giữ nguyên cấu trúc, không cần phải viết thêm biến mới.

- Kích thước của mảng:

Kích thước mảng = Số phần tử x Kích thước của một phần tử

- **Mảng và con trỏ là hai kiểu khác nhau.** Tuy nhiên, trong hầu hết biểu thức, tên mảng sẽ được chuyển (array decay) thành con trỏ trỏ tới phần tử đầu tiên của mảng.

- Truy cập các phần tử trong mảng:

- Dùng chỉ số (index): someData\[1\]
- Dùng con trỏ (pointer): \*(someData+1)

<a id="muc-10-02"></a>
### 10.2. Mảng hai chiều (2-D Array)

- Mảng hai chiều cho phép lưu trữ dữ liệu theo dạng bảng, giúp tổ chức thông tin một cách hợp lý qua các hàng và cột.

- Các phần tử được lưu trữ theo cách sắp xếp liên tiếp, tức là một hàng sau một hàng.

- Khởi tạo mảng hai chiều:

Cách 1: int matrix\[2\]\[3\] = {{1, 2, 3}, {4, 5, 6}};

Cách 2:

int matrix\[2\]\[3\];

matrix\[0\]\[0\] = 1;

matrix\[0\]\[1\] = 2;

- Truyền mảng hai chiều tới hàm:

Khi truyền một mảng hai chiều vào một hàm cần chỉ định kích thước cột của mảng trong các tham số của hàm. Tuy nhiên, kích thước hàng có thể không được chỉ định hoặc được cung cấp như một tham số của hàm.

Tại sao khi truyền mảng 2 chiều bắt buộc phải chỉ định kích thước cột của mảng? Mảng 2 chiều trong C được lưu trữ theo kiểu hàng. các phần tử của một hàng được lưu trữ liên tiếp trong bộ nhớ. Khi truyền mảng vào hàm, trình biên dịch cần biết kích thước của cột để tính toán chính xác địa chỉ của các phần tử trong mảng.

Công thức tính địa chỉ nếu một mảng được định nghĩa là arr\[n\]\[m\] trong đó n là số hàng và m là số cột trong mảng thì:

address(arr\[i\]\[j\]) = (array base address) + (i \* m + j) \* (element size)

<a id="muc-10-03"></a>
### 10.3. Mảng động và cấp phát bộ nhớ

- Mảng động là vùng nhớ dùng để lưu nhiều phần tử và được cấp phát trong thời gian chạy.
- Thường được cấp phát từ Heap bằng các hàm `malloc`, `calloc`, `realloc` và giải phóng bằng `free`.

#### `malloc()`

- Yêu cầu cấp phát một vùng nhớ có kích thước xác định.
- Nếu thành công, trả về con trỏ `void *` trỏ tới vùng nhớ vừa cấp phát; nếu thất bại, trả về `NULL`.
- Nội dung vùng nhớ mới cấp phát **chưa được khởi tạo**.
- Trong C, không cần ép kiểu kết quả của `malloc()`.

```c
int *ptr = malloc(10 * sizeof(int));
```

#### `calloc()`

- Cấp phát vùng nhớ cho một số lượng phần tử và **đặt toàn bộ các byte của vùng nhớ về 0**.

```c
int *ptr = calloc(10, sizeof(int));
```

- Không nên hiểu đơn giản rằng `calloc()` luôn tạo ra giá trị `NULL` cho mọi kiểu con trỏ hoặc giá trị `0.0` cho mọi kiểu dấu phẩy động trên mọi hệ thống; điều chắc chắn là các byte được đặt về 0.

#### `realloc()`

- Thay đổi kích thước của vùng nhớ đã được cấp phát trước đó.
- `new_size` là **kích thước mới tổng cộng**, không phải số byte muốn cộng thêm.
- Hàm có thể giữ nguyên địa chỉ cũ hoặc chuyển dữ liệu sang một vùng nhớ mới.

```c
ptr = realloc(ptr, new_size);
```

#### `free()`

- Giải phóng vùng nhớ động khi không còn sử dụng.

```c
free(ptr);
ptr = NULL;
```

- Cần chú ý hiện tượng **phân mảnh bộ nhớ** khi cấp phát/giải phóng động nhiều lần.
- Khi dùng mảng cục bộ có kích thước lớn, cần chú ý nguy cơ **Stack Overflow**.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-11"></a>
## 11. Toán tử

<a id="muc-11-01"></a>
### 11.1. Các nhóm toán tử

|                  |                   |                                    |                         |
|------------------|-------------------|------------------------------------|-------------------------|
| Tên nhóm toán tử | Ký hiệu phổ biến  | Mục đích sử dụng                   | Ví dụ                   |
| Số học           | +, -, \*, /, %    | Tính toán giá trị toán học         | a + b                   |
| Quan hệ          | ==, !=, \>, \<    | So sánh hai vế với nhau            | a \> b                  |
| Logic            | &&, \\ \\ , !     | Kết hợp các biểu thức so sánh      | (a \> 0) && (b \< 5)    |
| Gán              | =, +=, -=         | Đưa giá trị vào ô nhớ              | x += 5 (x = x + 5)      |
| Tăng giảm        | ++, --            | Thay đổi nhanh 1 đơn vị            | i++ hoặc --i            |
| Thao tác bit     | &, \|, \<\<, \>\> | Xử lý dữ liệu nhị phân (tầng thấp) | đèn led                 |
| Điều kiện        | ? :               | Viết if-else ngắn trên 1 dòng      | max = (a \> b) ? a : b  |
| Kích thước       | sizeof            | Đo dung lượng bộ nhớ               | sizeof(int) (ra 4 byte) |

- ++i và i++ khác nhau như thế nào

```c
int i = 5;
int result = ++i; // 1. i tăng lên 6
                          // 2. result nhận giá trị mới của i (result = 6)
// Kết quả: i = 6, result = 6
```

```c
int i = 5;
int result = i++; // 1. result nhận giá trị hiện tại của i (result = 5)
                          // 2. i mới tăng lên 6
// Kết quả: i = 6, result = 5
```

- Toán tử một ngôi: Chỉ cần đúng 1 toán hạng đi kèm để thực hiện phép toán.

Ví dụ: i++, --count

- Toán tử hai ngôi: Cần chính xác 2 toán hạng (một vế trái và một vế phải) để thực hiện phép toán.

Ví dụ: a + b, x % y

- Toán tử ba ngôi: Cần chính xác 3 toán hạng để hoạt động. Trong ngôn ngữ C, chỉ có duy nhất 1 toán tử thuộc nhóm này, đó là Toán tử điều kiện (? :), dùng để viết thay thế cho cấu trúc if-else ngắn gọn trên một dòng.

Ví dụ: int max = (a \> b) ? a : b;

<a id="muc-11-02"></a>
### 11.2. Thao tác bit

set/clear/toggle/check bit, masking

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-12"></a>
## 12. Từ khóa quan trọng

<a id="muc-12-01"></a>
### 12.1. `volatile`

- `volatile` dùng để báo cho compiler biết rằng giá trị của object có thể thay đổi ngoài luồng thực thi thông thường của đoạn code hiện tại.
- Vì vậy, các lần đọc/ghi tới object `volatile` phải được compiler giữ lại theo đúng yêu cầu của chương trình, không được tự ý loại bỏ hoặc gộp các lần truy cập như với biến thông thường.
- `volatile` **không có nghĩa** là biến luôn nằm trong RAM, không đảm bảo thao tác là atomic và cũng không tự làm code trở nên thread-safe.

- Dùng khi nào?
  - Truy cập thanh ghi ngoại vi/memory-mapped register.
  - Biến được thay đổi trong ISR và được đọc ở code chính.
  - Một số trường hợp dữ liệu có thể thay đổi bởi phần cứng hoặc tác nhân bên ngoài luồng thực thi hiện tại.

<a id="muc-12-02"></a>
### 12.2. `const`

- Định nghĩa: `const` dùng để khai báo rằng dữ liệu không được phép sửa đổi thông qua tên hoặc con trỏ `const` đó. Nếu cố gán lại trực tiếp cho một object được khai báo `const`, compiler sẽ báo lỗi.

- Dùng khi nào? Bất cứ khi nào không muốn thay đổi giá trị của biến, tham số hàm ở các dòng lệnh tiếp theo.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-13"></a>
## 13. Kiểu dữ liệu tự định nghĩa

<a id="muc-13-01"></a>
### 13.1. `struct`

- Định nghĩa: Là một cấu trúc dữ liệu do người dùng tự định nghĩa, cho phép kết hợp nhiều thành phần có kiểu dữ liệu khác nhau.
- Truy cập thành viên bằng toán tử `.`; nếu có con trỏ tới struct thì thường dùng toán tử `->`.

#### Căn chỉnh dữ liệu (Alignment)

- Mỗi kiểu dữ liệu có thể có một **yêu cầu căn chỉnh (alignment requirement)**. Compiler thường đặt dữ liệu tại địa chỉ phù hợp với yêu cầu này để CPU truy cập hiệu quả và đúng quy tắc của kiến trúc.
- Dữ liệu không được căn chỉnh có thể làm việc truy cập chậm hơn; trên một số kiến trúc, truy cập không căn chỉnh thậm chí có thể gây lỗi.

Ví dụ với một CPU mà `int` yêu cầu alignment 4 byte:

- Nếu `int` bắt đầu tại địa chỉ phù hợp như `0x...04`, CPU có thể truy cập thuận lợi.
- Nếu bắt đầu tại địa chỉ không phù hợp như `0x...02`, CPU có thể phải thực hiện nhiều thao tác hơn hoặc kiến trúc có thể không cho phép truy cập trực tiếp.

#### Struct padding

- Compiler có thể chèn các byte **padding** giữa các member để mỗi member thỏa yêu cầu alignment của kiểu đó.
- Cuối struct cũng có thể có padding để các phần tử của một mảng struct được căn chỉnh đúng.
- Vì vậy, kích thước `struct` không nhất thiết bằng tổng kích thước các member.

Ví dụ compiler-specific để yêu cầu packed struct:

```c
__attribute__((packed))
```

- `packed` có thể tiết kiệm bộ nhớ nhưng có thể tạo ra truy cập không căn chỉnh, vì vậy cần dùng cẩn thận.
- Struct thường dùng để mô tả một đối tượng có nhiều thuộc tính hoặc gom nhiều trường dữ liệu liên quan vào cùng một kiểu.

<a id="muc-13-02"></a>
### 13.2. Trường bit (Bit fields)

- Định nghĩa: Là một tính năng cho phép chỉ định chính xác số lượng bit mà một thành phần bên trong struct được phép sử dụng.

- Ví dụ:

```c
struct HopHoiThoai {
    unsigned int co_vien  : 1; // Chỉ chiếm 1 bit (Giá trị: 0 hoặc 1)
    unsigned int mau_nen  : 3; // Chiếm 3 bit (Giá trị từ 0 đến 7)
    unsigned int trong_suot: 1; // Chỉ chiếm 1 bit (Giá trị: 0 hoặc 1)
};
```

- Dùng khi nào?

- Thao tác với các thanh ghi của vi điều khiển.
- Tiết kiệm RAM/ tối ưu hóa bộ nhớ.

<a id="muc-13-03"></a>
### 13.3. `union`

- Định nghĩa: Là kiểu dữ liệu đặc biệt do người dùng tự định nghĩa, cho phép lưu trữ nhiều biến có kiểu dữ liệu khác nhau tại cùng một vùng nhớ trên RAM.

- Khác struct? Tại một thời điểm bất kỳ, union chỉ có thể lưu trữ giá trị của một biến duy nhất.

- Kích thước của `union` phải đủ để chứa member lớn nhất và có thể lớn hơn do yêu cầu alignment.

- Dùng khi nào? chủ yếu là tiết kiệm bộ nhớ RAM.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-14"></a>
## 14. Chuỗi ký tự (String)

- Định nghĩa: Là tập hợp tất cả các ký tự được kết thúc bởi một ký tự rỗng (‘\0’ -null).

- Cách khởi tạo:

char str\[\] = “Hello World!”;

char str\[\] = {‘H’, ‘e’, ‘l’, ‘l’, ‘o’}; → sai, đây là một mảng lưu trữ các ký tự.

để trở thành chuỗi, ta cần thêm ký tự null ở cuối.

char str\[\] = {‘H’, ‘e’, ‘l’, ‘l’, ‘o’, ‘\0’};

char str1\[10\] = “Hello”; → 10 bytes

char str2\[\] = “Hello”; → 6 bytes

tuy kích thước khác nhau nhưng độ dài chuỗi lại bằng nhau strlen(str1) = strlen(str2) = 5

- Phân biệt chuỗi và ký tự:

char str\[\] = “A”; (string có chứa ký tự ‘\0’)

char str = ‘A’; (ký tự)

→ khai báo chuỗi gồm các ký tự trong dấu nháy kép, khai báo một ký tự gồm 1 ký tự trong dấu nháy đơn.

- Bảng mã ASCII mã hóa 128 ký tự khác nhau

- Các hàm thường dùng:

strlen(str): đếm và trả về số lượng ký tự có trong chuỗi (không tính ‘\0’).

strcpy(dest, src): sao chép toàn bộ nội dung của chuỗi src vào chuỗi dest.

strncpy(dest, src): sao chép tối đa n ký tự từ src sang dest.

strcat(dest, src): ghép chuỗi src vào phía sau đuôi chuỗi dest.

strncat(dest, src): ghép tối đa n ký tự của chuỗi src vào sau đuôi chuỗi dest.

strcmp(str1, str2): so sánh hai chuỗi theo thứ tự từ điển, trả về: 0 nếu 2 chuỗi giống hệt nhau, \> 0 nếu ký tự đầu tiên khác nhau của str1 lớn hơn str2, \< 0 nếu ký tự đầu tiên khác nhau của str1 nhỏ hơn str2.

strchr(str, ch): tìm kiếm sự xuất hiện đầu tiên của ký tự ch trong chuỗi str, trả về một con trỏ trỏ đến vị trí ký tự đó, hoặc trả về NULL nếu không tìm thấy.

strstr(haystack, needle): tìm kiếm một chuỗi con (needle) trong chuỗi mẹ (haystack), trả về con trỏ trỏ đến vị trí đầu tiên của chuỗi con, hoặc NULL nếu không tìm thấy.

strtok(str, delimiters): tách chuỗi str thành các chuỗi nhỏ hơn (token) dựa trên các ký tự phân cách (

delimiters).

- String literal

```c
char *str = "Hello";   // Không được sửa nội dung string literal
char str[] = "Hello";  // Tạo một mảng char có thể sửa đổi
```

Vị trí lưu string literal là do implementation/toolchain quyết định; trong embedded, nó thường được đặt ở vùng nhớ chỉ đọc như Flash/ROM.

- Đọc chuỗi an toàn hơn bằng `fgets()`. Không nên dùng `gets()` vì hàm này không kiểm soát kích thước buffer và đã bị loại khỏi chuẩn C từ C11.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-15"></a>
## 15. Con trỏ (Pointer)

<a id="muc-15-01"></a>
### 15.1. Khái niệm cơ bản

- Định nghĩa: Là một biến đặc biệt dùng để lưu trữ địa chỉ bộ nhớ của biến khác.

- Kích thước con trỏ phụ thuộc implementation/kiến trúc. Trên nhiều hệ thống, các object pointer có cùng kích thước và thường tương ứng với độ rộng địa chỉ của CPU, nhưng C standard không bắt buộc mọi loại con trỏ phải có cùng kích thước.

- Khi cố ý coi một địa chỉ số cố định là con trỏ (thường gặp trong embedded), có thể dùng ép kiểu tường minh, Ví dụ:

```c
int *ptr = (int *)0x12345678;
```

Việc địa chỉ này có hợp lệ để truy cập hay không phụ thuộc memory map và hệ thống đích.

- Dangling Pointer (con trỏ lủng lẳng): xảy ra khi một con trỏ trỏ đến vùng nhớ đã được giải phóng hoặc bị xóa khỏi bộ nhớ của chương trình.

Ví dụ:

<a id="muc-15-02"></a>
### 15.2. Dangling pointer sau `free()`

```c
#include <stdio.h>
#include <stdlib.h>
int main() {
    // allocated using malloc() during runtime
    int *ptr = malloc(sizeof(int));
    *ptr = 10;
    printf("Add: %p, Value: %d\n", (void *)ptr, *ptr);
    // memory block deallocated using free() function
    free(ptr);
    // here ptr acts as a dangling pointer
    // Không được dereference ptr ở đây: vùng nhớ đã được free
}
```

<a id="muc-15-03"></a>
### 15.3. Dangling pointer khi trả về biến cục bộ

```c
#include <stdio.h>
// definition of danglingPointer() function
int *danglingPointer() {
    // temp variable has local scope
    int temp = 10;
    // returning address of temp variable
    return &temp;
}
int main() {
    int *ptr = danglingPointer();
    printf("%d", *ptr);
    return 0;
}
```

<a id="muc-15-04"></a>
### 15.4. Dangling pointer khi biến ra khỏi phạm vi

```c
#include <stdio.h>
int main()  {
    int *ptr;
    // variables declared inside the block of will get destroyed
    // at the end of execution of this block
    {
        int temp = 10;
        ptr = &temp; // acting as normal pointer
        printf("Add: %p\n", (void *)ptr);
    }
    // temp is now removed from the memory (out of scope)
    // now ptr is a dangling pointer
    // Không được dereference ptr ở đây: temp đã hết lifetime
}
```

Cách tránh:

- Sau `free(ptr)`, có thể gán `ptr = NULL` để tránh vô tình dùng lại con trỏ cũ.
- Không trả về địa chỉ của biến cục bộ. Nếu thật sự cần dữ liệu tồn tại sau khi hàm kết thúc, dùng cách có lifetime phù hợp như `static` hoặc vùng nhớ động, tùy trường hợp.

<a id="muc-15-05"></a>
### 15.5. `const` với con trỏ

Có 3 dạng thường gặp:

1. **Con trỏ hằng** — không đổi được địa chỉ mà con trỏ đang giữ, nhưng có thể sửa dữ liệu được trỏ tới:

```c
int *const ptr = &num;
```

2. **Con trỏ tới dữ liệu `const`** — có thể đổi địa chỉ con trỏ, nhưng không được sửa dữ liệu thông qua con trỏ này:

```c
const int *ptr = &num;
```

3. **Con trỏ hằng tới dữ liệu `const`** — không đổi được địa chỉ và cũng không được sửa dữ liệu thông qua con trỏ:

```c
const int *const ptr = &num;
```

- Tại sao con trỏ phải có kiểu dữ liệu rõ ràng?

  - Kiểu con trỏ cho compiler biết kiểu object được truy cập khi dereference. Ví dụ `char *` truy cập một `char`, còn `int *` truy cập một `int`.
  - Kiểu con trỏ quyết định bước nhảy của pointer arithmetic. Ví dụ `ptr + 1` trên `char *` tăng theo `sizeof(char)`, còn trên `int *` tăng theo `sizeof(int)`.

- Con trỏ void\* là một loại con trỏ đặc biệt trong C, đại diện cho một địa chỉ vùng nhớ mà không gắn liền với bất kỳ kiểu dữ liệu nào.

Hai đặc tính của void\*:

Đặc tính 1 (Thoải mái gán): Bạn có thể gán địa chỉ của bất kỳ kiểu dữ liệu nào (int, float, char, struct...) cho con trỏ void\* mà không cần ép kiểu, và ngược lại. Trình biên dịch sẽ không báo lỗi.

Đặc tính 2 (Cấm đọc trực tiếp): Bạn tuyệt đối không thể dùng toán tử lấy giá trị (\*ptr) hoặc tính toán con trỏ (ptr + 1) trực tiếp trên một con trỏ void\* (vì compiler không biết phải đọc bao nhiêu byte (1, 4 hay 8 byte) từ địa chỉ đó).

Dùng khi nào?

Viết hàm cấp phát bộ nhớ động (malloc, calloc)

Viết các hàm đa năng, Ví dụ: swap 2 phần tử, …

<a id="muc-15-06"></a>
### 15.6. Con trỏ với mảng

```c
#include <stdio.h>
int main(){
  int arr[5] = {2, 4, 6, 8, 10}, i;

  for(i = 0; i < 5; i++){
    printf("[index %d] Add: %p, Val: %d\n", i, (void *)(arr + i), *(arr + i));
  }
}
```

Kết quả:

```c
[index 0] Add: 1052769632, Val: 2
[index 1] Add: 1052769636, Val: 4
[index 2] Add: 1052769640, Val: 6
[index 3] Add: 1052769644, Val: 8
[index 4] Add: 1052769648, Val: 10
```

<a id="muc-15-07"></a>
### 15.7. Mảng con trỏ đến chuỗi ký tự

```c
#include<stdio.h>
int main() {
    char *names[] = {“John", “Jane", “Doe"};

    printf(“ %s\n %s\n %s\n”, names[0], names[1], names[2]);
}
```

Kết quả:

```c
 John
 Jane
 Doe
```

<a id="muc-15-08"></a>
### 15.8. Mảng con trỏ `void *`

```c
#include<stdio.h>
int main() {
    int a = 10;
    float b = 20.5;
    char c = 'Z';
    void *arr[3]; // Array of pointers to void, for mixed types
    arr[0] = &a;
    arr[1] = &b;
    arr[2] = &c;
    printf("Integer value: %d\n", *((int *)arr[0]));
    printf("Float value: %f\n", *((float *)arr[1]));
    printf("Char value: %c\n", *((char *)arr[2]));
}
```

Kết quả:

```c
Integer value: 10
Float value: 20.500000
Char value: Z
```

<a id="muc-15-09"></a>
### 15.9. Con trỏ hàm

Cú pháp: \<Kiểu trả về\> (\*\<tên con trỏ\>)(\<danh sách tham số\>)

Ví dụ: int (\*operation)(int, int);

```c
#include <stdio.h>
int add(int a, int b) {
    return a + b;
}
int multiply(int a, int b) {
    return a * b;
}
int main() {
    int (*operation)(int, int);
    // Gán hàm add cho con trỏ hàm
    operation = add;
    printf("Addition: %d\n", (*operation)(5, 3)); // Kết quả: 8
    // Gán hàm multiply cho con trỏ hàm
    operation = multiply;
    printf("Multiplication: %d\n", (*operation)(5, 3)); // Kết quả: 15
}
```

<a id="muc-15-10"></a>
### 15.10. Mảng con trỏ hàm

```c
#include<stdio.h>
float add(int a, int b)     { return a + b; }
float subtract(int a, int b) { return a - b; }
float multiply(int a, int b) { return a * b; }
float divide(int a, int b)  { return a / (b*1.0); }
int main() {
    int a = 10,   b = 5;
    float (*operation[4])(int, int);
    operation[0] = add;
    operation[1] = subtract;
    operation[2] = multiply;
    operation[3] = divide;

    printf("Addition (a+b) = %.1f\n",       (*operation[0])(a, b)); // Kết quả: 15.0
    printf("Subtraction (a-b) = %.1f\n",    (*operation[1])(a, b)); // Kết quả: 5.0
    printf("Multiplication (a*b) = %.1f\n", (*operation[2])(a, b)); // Kết quả: 50.0
    printf("Division (a/b) = %.1f\n",       (*operation[3])(a, b)); // Kết quả: 2.0
}
```

<a id="muc-15-11"></a>
### 15.11. Con trỏ trong hàm

Khi truyền một con trỏ vào hàm, giá trị địa chỉ của con trỏ được copy vào tham số. Thông qua địa chỉ đó, hàm có thể đọc hoặc thay đổi object gốc.

Có thể trả về con trỏ từ một hàm, nhưng **không được trả về địa chỉ của biến cục bộ có automatic storage duration**, vì biến đó hết thời gian tồn tại khi hàm kết thúc.

Ví dụ sai:

```c
int *increment(int a)
{
    int b = a + 1;
    return &b;     // Sai: b hết lifetime khi hàm kết thúc
}
```

Con trỏ nhận được sau khi hàm kết thúc sẽ là **dangling pointer**; dereference nó tạo ra hành vi không hợp lệ.

Cách an toàn để trả về con trỏ từ một hàm trong các ví dụ đang học:

- Trả về địa chỉ của object có `static` storage duration.
- Trả về vùng nhớ được cấp phát động và yêu cầu bên gọi `free()` khi dùng xong.
- Trả về một trong các địa chỉ hợp lệ được truyền vào hàm từ bên gọi.

Ví dụ:

#### Trả về con trỏ tới biến `static`

```c
#include<stdio.h>
#define LEN  5
int *Current_MaxValue(int *nums, int len) {
    static int max = 0;
    while(len--)
    {
        if(max < *nums){
            max = *nums;
        }
        nums++;
    }
    return &max;
}
int main() {
    int nums[LEN] = {3,10,5,1,6};
    int *max;
    max = Current_MaxValue(nums, (int)LEN);
    printf("Max value = %d", *max); // Kết quả = 10
}
```

#### Trả về con trỏ tới vùng nhớ động

```c
#include<stdio.h>
#include<stdlib.h>
#define LEN 5
int *Current_MaxValue(int *nums, int len) {
    int *max = malloc(sizeof(int));
    *max = 0;
    while(len--){
        if(*max < *nums){
            *max = *nums;
        }
        nums++;
    }
    return max;
}
int main() {
    int nums[LEN] = {3,10,5,1,6};
    int *max = Current_MaxValue(nums, (int)LEN);
    printf("Max value = %d", *max); // Kết quả = 10
    free(max); // Giải phóng bộ nhớ động
}
```

#### Trả về địa chỉ được truyền từ hàm gọi

```c
#include<stdio.h>
int *max(int *a, int *b) {
    if (*a > *b) {
        return a;
    }
    return b;
}
int main() {
    int a = 10, b = 15;
    int *greater;
    greater = max(&a, &b);
    printf("Max value = %d", *greater); // Kết quả = 15
}
```

<a id="muc-15-12"></a>
### 15.12. Con trỏ hàm làm callback

Một cách khác để sử dụng con trỏ hàm là chuyển chúng sang các hàm khác làm đối số hàm. Chúng ta cũng gọi các hàm như hàm callback vì hàm nhận gọi chúng trở lại.

Ví dụ:

```c
#include<stdio.h>
int conditionalSum(int a, int b, int (*ptr)(int)) {
    a = (*ptr)(a);
    b = (*ptr)(b);
    return a + b;
}
int square(int a) { return a * a; }
int cube(int a)  { return a * a * a; }
int main() {
    int (*fp)(int);
    fp = square;
    // sum = 2^2 + 3^2, as fp points to function sqaure()
    int sum = conditionalSum(2, 3, fp);
    printf("Square sum = %d\n", sum);
    fp = cube;
    // sum = 2^3 + 3^3, as fp points to function cube()
    sum = conditionalSum(2, 3, fp);
    printf("Cubic sum = %d", sum);
}
```

Kết quả:

```c
 Square sum = 13
 Cubic sum = 35
```

<a id="muc-15-13"></a>
### 15.13. Thay đổi biến thông qua con trỏ

```c
#include<stdio.h>
void square(int *n){
    *n = (*n) * (*n);
}
int main() {
    int n = 5;
    square(&n);
    printf("n^2 = %d\n", n);
}
```

Kết quả:

|          |
|----------|
| n^2 = 25 |

<a id="muc-15-14"></a>
### 15.14. Truy cập thành viên struct bằng con trỏ

Có hai cách để truy cập thành viên của struct mà con trỏ struct trỏ đến:

Sử dụng toán tử dấu hoa thị (\*) và dấu chấm (.). 

Sử dụng toán tử mũi tên (→).

Ví dụ:

```c
#include <stdio.h>
#include <stdint.h>

struct HS {
    char ten[20];
    uint8_t tuoi;
};

int main(void)
{
    struct HS hocsinh = {"An", 20};
    struct HS *hs_pointer = &hocsinh;

    printf("Ten: %s - Tuoi: %u\n", hocsinh.ten, (unsigned)hocsinh.tuoi);
    printf("Ten: %s - Tuoi: %u\n", (*hs_pointer).ten, (unsigned)(*hs_pointer).tuoi);
    printf("Ten: %s - Tuoi: %u\n", hs_pointer->ten, (unsigned)hs_pointer->tuoi);
}
```

<a id="muc-15-15"></a>
### 15.15. Con trỏ struct trong tham số hàm

```c
#include <stdio.h>
#include <stdint.h>

typedef struct {
    uint8_t angle;
    uint8_t speed;
}Motor_tds;
void Init(Motor_tds *Motor)
{
    Motor->angle = 90;
    Motor->speed = 50;
}
int main() {
    Motor_tds  Motor_s;
    // Khởi tạo
    Init(&Motor_s);
    printf("Speed: %d - Angle: %d\n", Motor_s.speed, Motor_s.angle);
}
```

Kết quả:

|                       |
|-----------------------|
| Speed: 50 - Angle: 90 |

<a id="muc-15-16"></a>
### 15.16. Con trỏ hàm là member của struct

```c
#include <stdio.h>
typedef struct Rectangle {
    float length;
    float width;
    float (*area)(struct Rectangle*); // Con trỏ hàm tính diện tích
}Rectangle;
// Hàm tính diện tích
float calculateArea(Rectangle* rect) {
    return rect->length * rect->width;
}
int main() {
    // Khởi tạo một đối tượng Rectangle
    Rectangle rect;
    rect.length = 5.0;
    rect.width = 3.0;
    rect.area = calculateArea; // Gán hàm tính diện tích cho con trỏ hàm
    // Tính và in diện tích sử dụng macro
    printf("Area: %.2f\n", rect.area(&rect)); // Kết quả: 15.00
}
```

Kết quả:

|             |
|-------------|
| Area: 15.00 |

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-16"></a>
## 16. Các chủ đề C bổ sung

<a id="muc-16-01"></a>
### 16.1. `typedef`

- Định nghĩa: Là một từ khóa được sử dụng để đặt một cái tên mới (alias/biệt danh) cho một kiểu dữ liệu đã có sẵn.

Ví dụ:

```c
typedef unsigned long ulong;
typedef struct {
    char ten[30];
    int tuoi;
} SinhVien;
```

<a id="muc-16-02"></a>
### 16.2. Big Endian và Little Endian

- Big Endian: Byte có trọng số cao nhất (MSB) sẽ được lưu ở địa chỉ bộ nhớ nhỏ nhất (đầu tiên).

- Little Endian: Byte có trọng số thấp nhất (LSB) sẽ được lưu ở địa chỉ bộ nhớ nhỏ nhất (đầu tiên).

<a id="muc-16-03"></a>
### 16.3. `enum`

- Định nghĩa: Là một kiểu dữ liệu do người dùng tự định nghĩa trong C, cho phép bạn gộp một nhóm các hằng số số nguyên có liên quan lại với nhau và đặt cho chúng những cái tên gợi nhớ.

Ví dụ:

```c
// 1. Khai báo kiểu enum
enum Huong {
    DONG, // Tự động gán bằng 0
    TAY,  // Tự động gán bằng 1
    NAM,  // Tự động gán bằng 2
    BAC   // Tự động gán bằng 3
};
int main() {
    // 2. Khai báo biến từ enum và gán giá trị
    enum Huong huong_nha = NAM;
    if (huong_nha == NAM) {
        printf("Huong nha cua ban co gia tri so la: %d", huong_nha); // In ra: 2
    }
    return 0;
}
```

- Dùng khi nào?

Quản lý các trạng thái của hệ thống (State Machine).

Quản lý menu lựa chọn.

Định nghĩa các tập hợp cố định (ngày trong tuần, …).

<a id="muc-16-04"></a>
### 16.4. Overflow

- Định nghĩa: Là hiện tượng xảy ra khi kết quả của một phép tính toán hoặc một giá trị dữ liệu vượt quá phạm vi lưu trữ của kiểu dữ liệu cho phép.

Ví dụ: với một kiểu số nguyên chỉ biểu diễn được từ -128 đến 127, giá trị 128 nằm ngoài phạm vi biểu diễn. Trong code thực tế cần tránh dựa vào việc số nguyên có dấu tự động "quay vòng"; cách xử lý khi vượt phạm vi còn phụ thuộc loại phép toán và quy tắc của kiểu dữ liệu.

<a id="muc-16-05"></a>
### 16.5. Internal Linkage và External Linkage

- Liên kết nội bộ (Internal Linkage): Định danh chỉ được phép nhìn thấy và sử dụng duy nhất bên trong file .c nơi nó được khai báo. Tất cả các file .c khác trong dự án hoàn toàn không biết đến sự tồn tại của nó.

Ví dụ:

```c
// File: secret.c
static int secret_key = 999; // Biến toàn cục mang Internal Linkage
static void do_internal_job() { // Hàm mang Internal Linkage
    // ...
}
```

- Liên kết ngoài (External Linkage): Cùng một tên có thể tham chiếu tới cùng một hàm hoặc object từ các translation unit khác nhau. File muốn sử dụng cần có declaration phù hợp, thường đặt trong header.

Ví dụ:

```c
// File: library.c
int global_count = 100; // Biến toàn cục (Mặc định là External Linkage)
void print_message() {  // Hàm mặc định là External Linkage
    // ...
}
```

```c
// File: library.h
extern int global_count; // Báo cho compiler biết biến này nằm ở file khác
extern void print_message();
```

<a id="muc-16-06"></a>
### 16.6. Header guard

Header guard tránh nội dung của cùng một header bị xử lý lặp lại nhiều lần trong một translation unit.

```c
#ifndef
#define
#endif
```

<a id="muc-16-07"></a>
### 16.7. `static inline` function

- `inline` là gợi ý cho compiler rằng một hàm là ứng viên phù hợp để thay lời gọi hàm bằng trực tiếp phần code của hàm tại vị trí gọi.
- Compiler **có quyền quyết định** có inline thật hay không, tùy mức tối ưu và ngữ cảnh.
- `static inline` thường được dùng cho các hàm nhỏ đặt trong header: `static` giúp mỗi translation unit có bản riêng, còn `inline` cho phép compiler tối ưu lời gọi nếu phù hợp.

Ví dụ:

```c
static inline int square(int x)
{
    return x * x;
}
```

Nếu compiler thực hiện inline, thay vì tạo một lời gọi hàm riêng, biểu thức `x * x` có thể được chèn trực tiếp vào nơi gọi.

- **Ưu điểm có thể có:** giảm overhead của lời gọi hàm với các hàm nhỏ.
- **Nhược điểm có thể có:** nếu inline quá nhiều, kích thước code có thể tăng.
- **Dùng khi nào:** các hàm nhỏ, được gọi thường xuyên, đặc biệt là helper function trong code embedded.

<a id="muc-16-08"></a>
### 16.8. Memory functions

memcpy(): Sao chép một khối bộ nhớ có kích thước n byte từ vùng nhớ nguồn (src) sang vùng nhớ đích (dest).

Cú pháp: void\* memcpy(void \*dest, const void \*src, size_t n);

memmove(): Sao chép một khối bộ nhớ có kích thước n byte từ src sang dest, tương tự như memcpy.

Cú pháp: void\* memmove(void \*dest, const void \*src, size_t n);

memset(): Gán cùng một giá trị byte (giá trị `c` được chuyển về `unsigned char`) cho `n` byte liên tiếp trong vùng nhớ.

Cú pháp: void\* memset(void \*ptr, int c, size_t n);

memcmp(): So sánh bộ nhớ của hai khối dữ liệu s1 và s2 theo từng byte một, kéo dài đúng n byte.

Cú pháp: int memcmp(const void \*s1, const void \*s2, size_t n);

Giá trị trả về:

0: Nếu n byte của hai vùng nhớ giống hệt nhau.

\> 0: Tại byte đầu tiên khác biệt, giá trị của s1 lớn hơn s2.

\< 0: Tại byte đầu tiên khác biệt, giá trị của s1 nhỏ hơn s2.

[↑ Về mục lục](#muc-luc)
