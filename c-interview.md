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

**Ví dụ:**

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

**Ví dụ:**

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

**Ví dụ:**

```text
main.s
  ↓
Assembler
  ↓
main.o
```

File `.o` **chưa phải firmware hoàn chỉnh**. Nó có thể vẫn chứa các symbol chưa được giải quyết.

**Ví dụ:**

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

- Ký tự đặc biệt, hay **escape sequence**, là các chuỗi ký tự bắt đầu bằng dấu `\` để biểu diễn những ký tự khó viết trực tiếp trong source code.

**Ví dụ:**

| Escape sequence | Ý nghĩa |
|---|---|
| `\n` | Xuống dòng. |
| `\r` | Đưa con trỏ về đầu dòng. |

<a id="muc-03-02"></a>
### 3.2. Comment trong C

- Comment một dòng:

```c
// Nội dung comment
```

- Comment nhiều dòng:

```c
/* Nội dung comment */
```

<a id="muc-03-03"></a>
### 3.3. Các kiểu dữ liệu trong C

- Kiểu dữ liệu xác định loại giá trị mà biến có thể lưu trữ hoặc hàm có thể trả về.
- Kích thước của kiểu dữ liệu phụ thuộc vào compiler và kiến trúc hệ thống.
- Có thể dùng toán tử `sizeof` để kiểm tra kích thước của một kiểu hoặc object.

**Các nhóm kiểu dữ liệu chính:**

- **Kiểu số nguyên:** `char`, `signed char`, `unsigned char`, `short`, `int`, `long`, `long long` và các dạng `unsigned` tương ứng.
- **Kiểu dấu phẩy động:** `float`, `double`, `long double`.
- **Kiểu liệt kê:** `enum`.
- **Kiểu rỗng:** `void`.
- **Kiểu dẫn xuất:** array, pointer, function, `struct`, `union`.

#### Kiểu `char`

#### `short` và `unsigned short`

#### `int` và `unsigned int`

#### `long` và `unsigned long`

#### `long long` và `unsigned long long`

#### Kiểu dấu phẩy động: `float`, `double`, `long double`

- Dấu phẩy động là cách biểu diễn số thực trong đó vị trí dấu phẩy có thể thay đổi nhờ phần số mũ.
- Phần lớn hệ thống hiện nay sử dụng chuẩn IEEE 754 để biểu diễn số thực.

Một số dấu phẩy động thường gồm 3 phần:

- **Sign:** dấu của số.
- **Exponent:** phần số mũ.
- **Mantissa/Fraction:** phần định trị, dùng để biểu diễn độ chính xác của số.

**`float` thường dùng IEEE 754 binary32:**

- 1 bit Sign.
- 8 bit Exponent.
- 23 bit Fraction.

**`double` thường dùng IEEE 754 binary64:**

- 1 bit Sign.
- 11 bit Exponent.
- 52 bit Fraction.

<a id="muc-03-04"></a>
### 3.4. Phương pháp bù 2

- Là phương pháp phổ biến để biểu diễn số nguyên có dấu trong hệ nhị phân.

**Cách lấy biểu diễn bù 2 của một số âm:**

1. Đảo tất cả các bit của số dương tương ứng.
2. Cộng thêm `1`.

**Ví dụ: biểu diễn `-5` với 8 bit**

```text
 5         = 0000 0101
Đảo bit    = 1111 1010
Cộng 1     = 1111 1011
```

**Kết quả:**

```text
-5 = 1111 1011
```

- Một lợi ích của bù 2 là CPU có thể xử lý phép trừ thông qua phép cộng với số âm.

<a id="muc-03-05"></a>
### 3.5. Toán tử `sizeof`

- `sizeof` trả về kích thước theo byte của một kiểu dữ liệu hoặc object.

**Ví dụ:**

```c
sizeof(int)
sizeof(double)
```

<a id="muc-03-06"></a>
### 3.6. Biến

- Biến là một object được đặt tên dùng để lưu trữ dữ liệu trong chương trình.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-04"></a>
## 4. Hàm

<a id="muc-04-01"></a>
### 4.1. Khái niệm hàm

- Hàm là một khối lệnh được đặt tên, dùng để thực hiện một nhiệm vụ cụ thể.

**Tại sao dùng hàm?**

- Code dễ đọc và mạch lạc hơn.
- Dễ debug và bảo trì.
- Tăng khả năng tái sử dụng code.

**Một hàm thường gồm:**

- **Kiểu trả về:** kiểu dữ liệu của giá trị mà hàm trả về; dùng `void` nếu không trả về dữ liệu.
- **Tên hàm:** tên dùng để gọi hàm.
- **Danh sách tham số:** dữ liệu đầu vào của hàm.
- **Thân hàm:** các câu lệnh xử lý nằm trong `{}`.

**Ví dụ có lỗi kiểu dữ liệu:**

```c
#include <stdio.h>

long long multiply(int x, int y)
{
    return x * y;
}

int main(void)
{
    long long a = 10, b = 30;
    int result = multiply(a, b);

    printf("result is: %d\n", result);
    return 0;
}
```

**Các lỗi cần chú ý:**

- `a` và `b` có kiểu `long long`, nhưng tham số `x`, `y` lại là `int` nên có nguy cơ mất dữ liệu khi chuyển kiểu.
- Phép nhân `x * y` được thực hiện theo kiểu `int` trước; nếu vượt phạm vi của `int` thì có thể bị overflow trước khi kết quả được trả về dưới dạng `long long`.
- Hàm trả về `long long`, nhưng `result` lại là `int`, nên tiếp tục có nguy cơ mất dữ liệu.

**Cách sửa:**

```c
#include <stdio.h>

long long multiply(long long x, long long y)
{
    return x * y;
}

int main(void)
{
    long long a = 10, b = 30;
    long long result = multiply(a, b);

    printf("result is: %lld\n", result);
    return 0;
}
```

#### Nguyên mẫu hàm (Function Prototype)

- Nguyên mẫu hàm cung cấp cho compiler thông tin về tên hàm, kiểu trả về và danh sách tham số trước khi phần định nghĩa hàm xuất hiện.

**Ví dụ:**

```c
int sum(int a, int b);
int sum(int, int);
```

<a id="muc-04-02"></a>
### 4.2. Định nghĩa, khai báo và khởi tạo

#### Định nghĩa (Definition)

- Định nghĩa cung cấp đầy đủ object hoặc phần thân của hàm.

**Ví dụ:**

```c
int x;

int tinhTong(int a, int b)
{
    return a + b;
}
```

#### Khai báo (Declaration)

- Khai báo thông báo cho compiler biết tên và kiểu của một biến hoặc hàm.

**Ví dụ:**

```c
extern int x;
int tinhTong(int a, int b);
```

#### Khởi tạo (Initialization)

- Khởi tạo là gán giá trị ban đầu cho biến ngay khi biến được định nghĩa.

**Ví dụ:**

```c
int x = 10;
```

<a id="muc-04-03"></a>
### 4.3. Đối số (Arguments) và tham số (Parameters)

- **Argument:** giá trị thực tế được truyền vào khi gọi hàm.
- **Parameter:** biến được khai báo trong phần định nghĩa hàm để nhận argument.

**Ví dụ:**

```c
#include <stdio.h>

int tinhTong(int a, int b)   // a, b là parameters
{
    return a + b;
}

int main(void)
{
    int x = 10;
    int y = 20;

    int kq1 = tinhTong(5, 3);   // 5, 3 là arguments
    int kq2 = tinhTong(x, y);   // x, y là arguments

    return 0;
}
```

<a id="muc-04-04"></a>
### 4.4. Truyền giá trị và truyền địa chỉ bằng con trỏ

- Trong C, **mọi đối số đều được truyền theo giá trị (pass by value)**.
- Nếu truyền một biến bình thường, hàm nhận bản sao của giá trị đó.
- Nếu muốn hàm thay đổi biến gốc, ta truyền **địa chỉ của biến bằng con trỏ**. Bản thân địa chỉ cũng được truyền theo giá trị.

**Ví dụ:**

```c
#include <stdio.h>

void tangThamTri(int x)
{
    x = x + 10;
}

void tangQuaConTro(int *x)
{
    *x = *x + 10;
}

int main(void)
{
    int n = 5;

    tangThamTri(n);
    printf("Sau tham tri: %d\n", n);

    tangQuaConTro(&n);
    printf("Sau con tro: %d\n", n);

    return 0;
}
```

**Kết quả:**

```text
Sau tham tri: 5
Sau con tro: 15
```

| Cách truyền | Ý nghĩa | Khi nào dùng |
|---|---|---|
| Truyền giá trị | Hàm nhận bản sao của dữ liệu. | Khi không muốn hàm sửa dữ liệu gốc. |
| Truyền địa chỉ bằng con trỏ | Hàm nhận bản sao của địa chỉ và có thể sửa dữ liệu tại địa chỉ đó. | Khi cần sửa biến gốc hoặc tránh sao chép dữ liệu lớn. |

<a id="muc-04-05"></a>
### 4.5. Hàm Variadic

- Hàm Variadic cho phép nhận số lượng argument thay đổi.
- Hàm variadic có ít nhất một tham số cố định, sau đó là `...`.

**Các thành phần cơ bản:**

- `va_list`: lưu trạng thái khi duyệt danh sách argument.
- `va_start`: bắt đầu truy cập các argument biến đổi.
- `va_arg`: lấy từng argument theo kiểu được chỉ định.
- `va_copy`: sao chép một `va_list`.
- `va_end`: kết thúc việc sử dụng `va_list`.

**Ví dụ:**

```c
#include <stdio.h>
#include <stdarg.h>

int tinhTongNhieuSo(int count, ...)
{
    va_list args;
    int tong = 0;

    va_start(args, count);

    for (int i = 0; i < count; i++) {
        tong += va_arg(args, int);
    }

    va_end(args);
    return tong;
}

int main(void)
{
    printf("Tong 3 so: %d\n", tinhTongNhieuSo(3, 10, 20, 30));
    printf("Tong 5 so: %d\n", tinhTongNhieuSo(5, 1, 2, 3, 4, 5));
    return 0;
}
```

**Kết quả:**

```text
Tong 3 so: 60
Tong 5 so: 15
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

| Tiêu chí | `if-else` | `switch-case` |
|---|---|---|
| Loại điều kiện | Có thể dùng các biểu thức điều kiện và kiểm tra giá trị khác `0` / bằng `0`. | So sánh biểu thức `switch` với các nhãn `case` là hằng số nguyên phù hợp. |
| Kiểu dữ liệu | Biểu thức điều kiện thường là kiểu scalar. | Biểu thức `switch` dùng kiểu integer hoặc `enum`. |
| Số lượng nhánh | Phù hợp khi điều kiện linh hoạt hoặc số nhánh không quá nhiều. | Dễ đọc khi có nhiều nhánh dựa trên một giá trị. |
| Tốc độ thực thi | Phụ thuộc vào điều kiện và cách compiler tối ưu. | Cũng phụ thuộc compiler; có thể dùng chuỗi so sánh hoặc jump table. |

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-08"></a>
## 8. Tổ chức bộ nhớ

<a id="muc-08-01"></a>
### 8.1. Memory layout

- **Text/Code Segment:** thường chứa mã máy/lệnh thực thi của chương trình.
- **Data Segment:** thường chứa biến global/static đã được khởi tạo với giá trị khác 0.
- **BSS Segment:** thường chứa biến global/static chưa khởi tạo hoặc được khởi tạo bằng 0.
- **Heap:** vùng nhớ thường được sử dụng cho cấp phát động như `malloc()`, `calloc()`, `realloc()`.
- **Stack:** thường được dùng cho stack frame của hàm, biến cục bộ, địa chỉ trả về và dữ liệu điều khiển liên quan đến lời gọi hàm; chi tiết phụ thuộc compiler và kiến trúc.

#### Memory Leak

- Memory leak xảy ra khi chương trình cấp phát vùng nhớ động nhưng không giải phóng khi không còn sử dụng.

**Nguyên nhân thường gặp:**

- Quên gọi `free()`.
- Làm mất con trỏ duy nhất đang giữ địa chỉ của vùng nhớ đã cấp phát.
- Cấp phát lặp lại nhiều lần nhưng không có kế hoạch giải phóng.

**Cách hạn chế:**

- Mỗi vùng nhớ động cần có quy tắc sở hữu và thời điểm giải phóng rõ ràng.
- Gọi `free()` khi vùng nhớ không còn cần thiết.
- Có thể dùng công cụ phân tích bộ nhớ khi môi trường hỗ trợ.

<a id="muc-08-02"></a>
### 8.2. Các section

- **`.text`:** chứa mã máy của chương trình.
- **`.rodata`:** chứa dữ liệu chỉ đọc.
- **`.data`:** chứa biến global/static đã được khởi tạo.
- **`.bss`:** chứa biến global/static chưa khởi tạo hoặc được khởi tạo bằng 0.
- **`.symtab`:** chứa symbol table để các công cụ như linker/debugger sử dụng.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-09"></a>
## 9. Vòng lặp

<a id="muc-09-01"></a>
### 9.1. `for`, `while`, `do-while`

| Vòng lặp | Khi nào nên dùng? |
|---|---|
| `for` | Khi biết trước số lần lặp cụ thể, ví dụ duyệt mảng hoặc chạy `N` lần. |
| `while` | Khi chưa biết trước số lần lặp và cần kiểm tra điều kiện trước. |
| `do-while` | Khi thân vòng lặp phải chạy ít nhất một lần rồi mới kiểm tra điều kiện. |

<a id="muc-09-02"></a>
### 9.2. `continue`

- `continue` bỏ qua phần còn lại của lần lặp hiện tại và chuyển sang lần lặp tiếp theo.

<a id="muc-09-03"></a>
### 9.3. `break`

- `break` thoát ngay khỏi vòng lặp hoặc khối `switch-case` hiện tại.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-10"></a>
## 10. Mảng

<a id="muc-10-01"></a>
### 10.1. Mảng một chiều (Array)

- Mảng lưu trữ nhiều phần tử **cùng kiểu dữ liệu**.
- Các phần tử của mảng nằm liên tiếp trong bộ nhớ.

**Tại sao dùng mảng?**

- Quản lý nhiều phần tử bằng một tên chung.
- Dễ kết hợp với vòng lặp.
- Thuận tiện khi cần xử lý dữ liệu liên tiếp trong bộ nhớ.

**Kích thước mảng:**

```text
Kích thước mảng = Số phần tử × Kích thước của một phần tử
```

- **Mảng và con trỏ là hai kiểu khác nhau.** Tuy nhiên, trong hầu hết biểu thức, tên mảng sẽ được chuyển (array decay) thành con trỏ tới phần tử đầu tiên.

**Truy cập phần tử:**

```c
someData[1]
*(someData + 1)
```

<a id="muc-10-02"></a>
### 10.2. Mảng hai chiều (2-D Array)

- Mảng hai chiều thường được dùng để biểu diễn dữ liệu dạng hàng và cột.
- Trong C, các phần tử được lưu theo từng hàng liên tiếp trong bộ nhớ.

**Khởi tạo:**

```c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};
```

Hoặc:

```c
int matrix[2][3];

matrix[0][0] = 1;
matrix[0][1] = 2;
```

**Khi truyền mảng hai chiều vào hàm:**

- Compiler cần biết kích thước của các chiều phía sau để tính đúng địa chỉ phần tử.
- Với `arr[n][m]`, địa chỉ phần tử có thể hiểu theo công thức:

```text
address(arr[i][j])
= base_address + (i × m + j) × element_size
```

<a id="muc-10-03"></a>
### 10.3. Mảng động và cấp phát bộ nhớ

- Mảng động là vùng nhớ dùng để lưu nhiều phần tử và được cấp phát trong thời gian chạy.
- Thường được cấp phát từ Heap bằng `malloc`, `calloc`, `realloc` và giải phóng bằng `free`.

#### `malloc()`

- Yêu cầu cấp phát một vùng nhớ có kích thước xác định.
- Thành công: trả về `void *` trỏ tới vùng nhớ mới.
- Thất bại: trả về `NULL`.
- Vùng nhớ mới **chưa được khởi tạo**.
- Trong C, không cần ép kiểu kết quả của `malloc()`.

**Ví dụ:**

```c
int *ptr = malloc(10 * sizeof(int));
```

#### `calloc()`

- Cấp phát vùng nhớ cho nhiều phần tử và đặt toàn bộ byte trong vùng nhớ mới về `0`.

**Ví dụ:**

```c
int *ptr = calloc(10, sizeof(int));
```

- Không nên hiểu đơn giản rằng `calloc()` luôn tạo ra giá trị `NULL` cho mọi kiểu con trỏ hoặc `0.0` cho mọi kiểu floating-point trên mọi hệ thống; điều chắc chắn là các byte được đặt về `0`.

#### `realloc()`

- Thay đổi kích thước của vùng nhớ đã được cấp phát trước đó.
- `new_size` là **kích thước mới tổng cộng**.
- Địa chỉ trả về có thể giống hoặc khác địa chỉ cũ.

**Ví dụ:**

```c
ptr = realloc(ptr, new_size);
```

#### `free()`

- Giải phóng vùng nhớ động khi không còn sử dụng.

**Ví dụ:**

```c
free(ptr);
ptr = NULL;
```

**Lưu ý:**

- Cấp phát/giải phóng động nhiều lần có thể gây phân mảnh bộ nhớ.
- Mảng cục bộ quá lớn có thể làm tăng nguy cơ stack overflow.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-11"></a>
## 11. Toán tử

<a id="muc-11-01"></a>
### 11.1. Các nhóm toán tử

| Nhóm toán tử | Ký hiệu phổ biến | Mục đích | Ví dụ |
|---|---|---|---|
| Số học | `+`, `-`, `*`, `/`, `%` | Tính toán số học. | `a + b` |
| Quan hệ | `==`, `!=`, `>`, `<` | So sánh hai giá trị. | `a > b` |
| Logic | `&&`, `||`, `!` | Kết hợp hoặc đảo điều kiện. | `(a > 0) && (b < 5)` |
| Gán | `=`, `+=`, `-=` | Gán hoặc cập nhật giá trị. | `x += 5` |
| Tăng giảm | `++`, `--` | Tăng hoặc giảm 1. | `i++`, `--i` |
| Thao tác bit | `&`, `|`, `<<`, `>>` | Xử lý dữ liệu theo bit. | `value << 1` |
| Điều kiện | `?:` | Viết biểu thức điều kiện ngắn. | `(a > b) ? a : b` |
| Kích thước | `sizeof` | Lấy kích thước theo byte. | `sizeof(int)` |

#### `++i` và `i++`

**Pre-increment:**

```c
int i = 5;
int result = ++i;

// i = 6
// result = 6
```

**Post-increment:**

```c
int i = 5;
int result = i++;

// result = 5
// i = 6
```

#### Phân loại theo số toán hạng

- **Toán tử một ngôi:** cần 1 toán hạng. Ví dụ: `i++`, `--count`.
- **Toán tử hai ngôi:** cần 2 toán hạng. Ví dụ: `a + b`, `x % y`.
- **Toán tử ba ngôi:** trong C có toán tử điều kiện `?:`.

```c
int max = (a > b) ? a : b;
```

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

- `struct` cho phép gom nhiều thành phần có kiểu dữ liệu khác nhau vào cùng một kiểu dữ liệu.
- Dùng `.` để truy cập member từ một biến struct.
- Dùng `->` để truy cập member thông qua con trỏ struct.

#### Căn chỉnh dữ liệu (Alignment)

- Mỗi kiểu dữ liệu có thể có một **alignment requirement**.
- Compiler thường đặt dữ liệu tại địa chỉ phù hợp với yêu cầu căn chỉnh để CPU truy cập hiệu quả và đúng quy tắc của kiến trúc.
- Truy cập dữ liệu không căn chỉnh có thể chậm hơn; trên một số kiến trúc còn có thể gây lỗi.

**Ví dụ với kiểu `int` yêu cầu alignment 4 byte:**

- Địa chỉ như `0x...04` phù hợp với alignment 4 byte.
- Địa chỉ như `0x...02` có thể là địa chỉ không căn chỉnh cho `int`.

#### Struct padding

- Compiler có thể chèn các byte **padding** giữa các member để đáp ứng alignment.
- Cuối struct cũng có thể có padding.
- Vì vậy, `sizeof(struct)` không nhất thiết bằng tổng `sizeof` của từng member.

**Ví dụ compiler-specific để yêu cầu packed struct:**

```c
__attribute__((packed))
```

- `packed` có thể tiết kiệm bộ nhớ nhưng cũng có thể tạo truy cập không căn chỉnh.

<a id="muc-13-02"></a>
### 13.2. Trường bit (Bit fields)

- Bit field cho phép chỉ định số bit mà một member trong `struct` sử dụng.

**Ví dụ:**

```c
struct HopHoiThoai {
    unsigned int co_vien    : 1;
    unsigned int mau_nen    : 3;
    unsigned int trong_suot : 1;
};
```

**Dùng khi nào?**

- Mô tả các trường dữ liệu nhỏ theo bit.
- Tiết kiệm bộ nhớ trong một số trường hợp.
- Có thể gặp khi mô tả register, nhưng cách bố trí bit field phụ thuộc implementation nên cần dùng cẩn thận.

<a id="muc-13-03"></a>
### 13.3. `union`

- `union` cho phép nhiều member dùng chung cùng một vùng nhớ.
- Tại một thời điểm, vùng nhớ chung đó chứa representation của member đang được sử dụng.
- Kích thước của `union` phải đủ chứa member lớn nhất và có thể lớn hơn do alignment.
- Thường được dùng khi muốn tiết kiệm bộ nhớ hoặc diễn giải cùng vùng dữ liệu theo nhiều kiểu khác nhau.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-14"></a>
## 14. Chuỗi ký tự (String)

- Trong C, string là một mảng `char` kết thúc bằng ký tự null `\0`.

### Cách khởi tạo

```c
char str[] = "Hello World!";
```

Mảng ký tự sau **chưa phải string hợp lệ** vì thiếu `\0`:

```c
char str[] = {'H', 'e', 'l', 'l', 'o'};
```

Cách đúng:

```c
char str[] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

### Kích thước và độ dài

```c
char str1[10] = "Hello";
char str2[]   = "Hello";
```

- `sizeof(str1)` là `10` byte.
- `sizeof(str2)` là `6` byte, gồm 5 ký tự và `\0`.
- `strlen(str1)` và `strlen(str2)` đều bằng `5`.

### Phân biệt string và ký tự

```c
char str[] = "A";  // String: gồm 'A' và '\0'
char ch    = 'A';  // Một ký tự
```

- String dùng dấu nháy kép `"..."`.
- Một ký tự dùng dấu nháy đơn `'...'`.

### ASCII

- ASCII chuẩn mã hóa 128 giá trị ký tự.

### Các hàm string thường dùng

- `strlen(str)`: trả về độ dài string, không tính `\0`.
- `strcpy(dest, src)`: sao chép string `src` sang `dest`.
- `strncpy(dest, src, n)`: sao chép tối đa `n` ký tự.
- `strcat(dest, src)`: nối `src` vào cuối `dest`.
- `strncat(dest, src, n)`: nối tối đa `n` ký tự.
- `strcmp(str1, str2)`: so sánh hai string.
- `strchr(str, ch)`: tìm lần xuất hiện đầu tiên của ký tự `ch`.
- `strstr(haystack, needle)`: tìm substring.
- `strtok(str, delimiters)`: tách string thành các token theo delimiter.

### String literal

```c
char *str = "Hello";   // Không được sửa nội dung string literal
char arr[] = "Hello";  // Mảng char có thể sửa đổi
```

- Vị trí lưu string literal do implementation/toolchain quyết định; trong embedded, string literal thường được đặt ở vùng nhớ chỉ đọc như Flash/ROM.
- Khi đọc chuỗi, `fgets()` an toàn hơn `gets()`. `gets()` không kiểm soát kích thước buffer và đã bị loại khỏi chuẩn C từ C11.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-15"></a>
## 15. Con trỏ (Pointer)

<a id="muc-15-01"></a>
### 15.1. Khái niệm cơ bản

- Con trỏ là một biến dùng để lưu địa chỉ.
- Kích thước con trỏ phụ thuộc implementation và kiến trúc hệ thống.

**Ví dụ ép một địa chỉ số thành con trỏ:**

```c
int *ptr = (int *)0x12345678;
```

- Địa chỉ đó có hợp lệ để truy cập hay không phụ thuộc memory map của hệ thống đích.
- **Dangling pointer** là con trỏ vẫn giữ một địa chỉ nhưng object/vùng nhớ tại đó đã hết lifetime hoặc đã được giải phóng.

<a id="muc-15-02"></a>
### 15.2. Dangling pointer sau `free()`

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(sizeof(int));

    *ptr = 10;
    printf("Add: %p, Value: %d\n", (void *)ptr, *ptr);

    free(ptr);
    ptr = NULL;

    return 0;
}
```

- Sau `free()`, không được dereference địa chỉ cũ.
- Gán `ptr = NULL` giúp giảm nguy cơ vô tình dùng lại con trỏ đã được giải phóng.

<a id="muc-15-03"></a>
### 15.3. Dangling pointer khi trả về biến cục bộ

```c
int *danglingPointer(void)
{
    int temp = 10;
    return &temp;   // Sai: temp hết lifetime khi hàm kết thúc
}
```

- Khi hàm kết thúc, `temp` không còn tồn tại nên con trỏ trả về trở thành dangling pointer.

<a id="muc-15-04"></a>
### 15.4. Dangling pointer khi biến ra khỏi phạm vi

```c
#include <stdio.h>

int main(void)
{
    int *ptr;

    {
        int temp = 10;
        ptr = &temp;
        printf("Add: %p\n", (void *)ptr);
    }

    // Không được dereference ptr tại đây vì temp đã hết lifetime.
    return 0;
}
```

**Cách tránh:**

- Không giữ địa chỉ của object sau khi object hết lifetime.
- Sau `free(ptr)`, có thể gán `ptr = NULL`.
- Không trả về địa chỉ của biến cục bộ có automatic storage duration.

<a id="muc-15-05"></a>
### 15.5. `const` với con trỏ

Có 3 dạng thường gặp:

1. **Con trỏ hằng:** không đổi được địa chỉ mà con trỏ đang giữ, nhưng có thể sửa dữ liệu được trỏ tới.

```c
int *const ptr = &num;
```

2. **Con trỏ tới dữ liệu `const`:** có thể đổi địa chỉ con trỏ, nhưng không được sửa dữ liệu thông qua con trỏ này.

```c
const int *ptr = &num;
```

3. **Con trỏ hằng tới dữ liệu `const`:** không đổi được địa chỉ và cũng không được sửa dữ liệu thông qua con trỏ.

```c
const int *const ptr = &num;
```

#### Tại sao con trỏ cần có kiểu dữ liệu?

- Kiểu con trỏ cho compiler biết kiểu object được truy cập khi dereference.
- Kiểu con trỏ quyết định bước nhảy của pointer arithmetic.

**Ví dụ:**

```text
char *ptr  -> ptr + 1 tăng theo sizeof(char)
int  *ptr  -> ptr + 1 tăng theo sizeof(int)
```

#### Con trỏ `void *`

- `void *` là con trỏ tổng quát có thể giữ địa chỉ của nhiều loại object khác nhau.
- Không thể dereference trực tiếp `void *` vì compiler chưa biết kiểu dữ liệu cần đọc.
- Theo chuẩn C, pointer arithmetic trực tiếp trên `void *` không hợp lệ.

**Dùng khi nào?**

- Các hàm cấp phát bộ nhớ động như `malloc()`.
- Các hàm tổng quát cần nhận địa chỉ của nhiều kiểu dữ liệu khác nhau.

<a id="muc-15-06"></a>
### 15.6. Con trỏ với mảng

**Ví dụ:**

```c
#include <stdio.h>

int main(void)
{
    int arr[5] = {2, 4, 6, 8, 10};

    for (int i = 0; i < 5; i++) {
        printf("[index %d] Add: %p, Val: %d\n",
               i,
               (void *)(arr + i),
               *(arr + i));
    }

    return 0;
}
```

**Kết quả minh họa:**

```text
[index 0] Add: ..., Val: 2
[index 1] Add: ..., Val: 4
[index 2] Add: ..., Val: 6
[index 3] Add: ..., Val: 8
[index 4] Add: ..., Val: 10
```

<a id="muc-15-07"></a>
### 15.7. Mảng con trỏ đến chuỗi ký tự

**Ví dụ:**

```c
#include <stdio.h>

int main(void)
{
    const char *names[] = {"John", "Jane", "Doe"};

    printf("%s\n%s\n%s\n", names[0], names[1], names[2]);
    return 0;
}
```

**Kết quả:**

```text
John
Jane
Doe
```

<a id="muc-15-08"></a>
### 15.8. Mảng con trỏ `void *`

**Ví dụ:**

```c
#include <stdio.h>

int main(void)
{
    int a = 10;
    float b = 20.5f;
    char c = 'Z';

    void *arr[3];
    arr[0] = &a;
    arr[1] = &b;
    arr[2] = &c;

    printf("Integer value: %d\n", *((int *)arr[0]));
    printf("Float value: %f\n", *((float *)arr[1]));
    printf("Char value: %c\n", *((char *)arr[2]));

    return 0;
}
```

**Kết quả:**

```text
Integer value: 10
Float value: 20.500000
Char value: Z
```

<a id="muc-15-09"></a>
### 15.9. Con trỏ hàm

**Cú pháp:**

```c
<kiểu_trả_về> (*<tên_con_trỏ>)(<danh_sách_tham_số>);
```

**Ví dụ khai báo:**

```c
int (*operation)(int, int);
```

**Ví dụ sử dụng:**

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int multiply(int a, int b)
{
    return a * b;
}

int main(void)
{
    int (*operation)(int, int);

    operation = add;
    printf("Addition: %d\n", operation(5, 3));

    operation = multiply;
    printf("Multiplication: %d\n", operation(5, 3));

    return 0;
}
```

**Kết quả:**

```text
Addition: 8
Multiplication: 15
```

<a id="muc-15-10"></a>
### 15.10. Mảng con trỏ hàm

```c
#include <stdio.h>

float add(int a, int b)      { return a + b; }
float subtract(int a, int b) { return a - b; }
float multiply(int a, int b) { return a * b; }
float divide(int a, int b)   { return a / (b * 1.0f); }

int main(void)
{
    int a = 10, b = 5;
    float (*operation[4])(int, int);

    operation[0] = add;
    operation[1] = subtract;
    operation[2] = multiply;
    operation[3] = divide;

    printf("Addition = %.1f\n", operation[0](a, b));
    printf("Subtraction = %.1f\n", operation[1](a, b));
    printf("Multiplication = %.1f\n", operation[2](a, b));
    printf("Division = %.1f\n", operation[3](a, b));

    return 0;
}
```

**Kết quả:**

```text
Addition = 15.0
Subtraction = 5.0
Multiplication = 50.0
Division = 2.0
```

<a id="muc-15-11"></a>
### 15.11. Con trỏ trong hàm

- Khi truyền con trỏ vào hàm, **giá trị địa chỉ** của con trỏ được copy vào tham số.
- Thông qua địa chỉ đó, hàm có thể đọc hoặc thay đổi object gốc.
- Không được trả về địa chỉ của biến cục bộ có automatic storage duration.

**Ví dụ sai:**

```c
int *increment(int a)
{
    int b = a + 1;
    return &b;   // Sai: b hết lifetime khi hàm kết thúc
}
```

**Các cách trả về con trỏ hợp lệ trong các ví dụ đang học:**

- Trả về địa chỉ của object có `static` storage duration.
- Trả về vùng nhớ được cấp phát động và yêu cầu bên gọi `free()`.
- Trả về một địa chỉ hợp lệ đã được truyền vào hàm.

#### Trả về con trỏ tới biến `static`

```c
#include <stdio.h>
#define LEN 5

int *Current_MaxValue(int *nums, int len)
{
    static int max = 0;

    while (len--) {
        if (max < *nums) {
            max = *nums;
        }
        nums++;
    }

    return &max;
}
```

#### Trả về con trỏ tới vùng nhớ động

```c
#include <stdlib.h>

int *createValue(int value)
{
    int *ptr = malloc(sizeof(int));

    if (ptr != NULL) {
        *ptr = value;
    }

    return ptr;
}
```

#### Trả về địa chỉ được truyền từ hàm gọi

```c
int *max(int *a, int *b)
{
    if (*a > *b) {
        return a;
    }

    return b;
}
```

<a id="muc-15-12"></a>
### 15.12. Con trỏ hàm làm callback

- Một hàm có thể nhận con trỏ hàm làm argument và gọi hàm được truyền vào. Đây là một cách triển khai callback.

**Ví dụ:**

```c
#include <stdio.h>

int conditionalSum(int a, int b, int (*callback)(int))
{
    a = callback(a);
    b = callback(b);
    return a + b;
}

int square(int a) { return a * a; }
int cube(int a)   { return a * a * a; }

int main(void)
{
    int sum = conditionalSum(2, 3, square);
    printf("Square sum = %d\n", sum);

    sum = conditionalSum(2, 3, cube);
    printf("Cubic sum = %d\n", sum);

    return 0;
}
```

**Kết quả:**

```text
Square sum = 13
Cubic sum = 35
```

<a id="muc-15-13"></a>
### 15.13. Thay đổi biến thông qua con trỏ

```c
#include <stdio.h>

void square(int *n)
{
    *n = (*n) * (*n);
}

int main(void)
{
    int n = 5;
    square(&n);
    printf("n^2 = %d\n", n);
    return 0;
}
```

**Kết quả:**

```text
n^2 = 25
```

<a id="muc-15-14"></a>
### 15.14. Truy cập thành viên struct bằng con trỏ

Có hai cách tương đương:

```c
(*hs_pointer).ten
hs_pointer->ten
```

**Ví dụ:**

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

    printf("Ten: %s - Tuoi: %u\n",
           hs_pointer->ten,
           (unsigned)hs_pointer->tuoi);

    return 0;
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
} Motor_tds;

void Init(Motor_tds *motor)
{
    motor->angle = 90;
    motor->speed = 50;
}

int main(void)
{
    Motor_tds motor;
    Init(&motor);

    printf("Speed: %u - Angle: %u\n",
           (unsigned)motor.speed,
           (unsigned)motor.angle);

    return 0;
}
```

**Kết quả:**

```text
Speed: 50 - Angle: 90
```

<a id="muc-15-16"></a>
### 15.16. Con trỏ hàm là member của struct

```c
#include <stdio.h>

typedef struct Rectangle {
    float length;
    float width;
    float (*area)(struct Rectangle *);
} Rectangle;

float calculateArea(Rectangle *rect)
{
    return rect->length * rect->width;
}

int main(void)
{
    Rectangle rect;

    rect.length = 5.0f;
    rect.width = 3.0f;
    rect.area = calculateArea;

    printf("Area: %.2f\n", rect.area(&rect));
    return 0;
}
```

**Kết quả:**

```text
Area: 15.00
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-16"></a>
## 16. Các chủ đề C bổ sung

<a id="muc-16-01"></a>
### 16.1. `typedef`

- `typedef` tạo một tên mới (alias) cho một kiểu dữ liệu đã có.

**Ví dụ:**

```c
typedef unsigned long ulong;

typedef struct {
    char ten[30];
    int tuoi;
} SinhVien;
```

<a id="muc-16-02"></a>
### 16.2. Big Endian và Little Endian

- **Big Endian:** byte có trọng số cao nhất được lưu ở địa chỉ thấp hơn.
- **Little Endian:** byte có trọng số thấp nhất được lưu ở địa chỉ thấp hơn.

<a id="muc-16-03"></a>
### 16.3. `enum`

- `enum` cho phép gom một nhóm hằng số nguyên có liên quan và đặt tên dễ nhớ cho chúng.

**Ví dụ:**

```c
enum Huong {
    DONG,   // 0
    TAY,    // 1
    NAM,    // 2
    BAC     // 3
};

int main(void)
{
    enum Huong huong_nha = NAM;
    return 0;
}
```

**Dùng khi nào?**

- Quản lý trạng thái trong State Machine.
- Quản lý menu lựa chọn.
- Biểu diễn một tập hợp giá trị cố định có tên rõ ràng.

<a id="muc-16-04"></a>
### 16.4. Overflow

- Overflow xảy ra khi kết quả của phép tính nằm ngoài phạm vi biểu diễn của kiểu dữ liệu.

**Ví dụ:**

- Với kiểu chỉ biểu diễn được từ `-128` đến `127`, giá trị `128` nằm ngoài phạm vi biểu diễn.
- Trong code thực tế không nên dựa vào việc số nguyên có dấu tự động quay vòng; cách xử lý khi vượt phạm vi phụ thuộc kiểu dữ liệu và quy tắc của C.

<a id="muc-16-05"></a>
### 16.5. Internal Linkage và External Linkage

#### Internal Linkage

- Tên chỉ được dùng để tham chiếu tới cùng object hoặc function trong translation unit hiện tại.

**Ví dụ:**

```c
// secret.c
static int secret_key = 999;

static void do_internal_job(void)
{
    // ...
}
```

#### External Linkage

- Cùng một tên có thể tham chiếu tới cùng object hoặc function từ các translation unit khác nhau.
- File muốn sử dụng cần có declaration phù hợp, thường được đặt trong header.

**Ví dụ:**

```c
// library.c
int global_count = 100;

void print_message(void)
{
    // ...
}
```

```c
// library.h
extern int global_count;
void print_message(void);
```

<a id="muc-16-06"></a>
### 16.6. Header guard

- Header guard giúp tránh nội dung của cùng một header bị xử lý lặp lại nhiều lần trong một translation unit.

**Mẫu:**

```c
#ifndef MY_HEADER_H
#define MY_HEADER_H

// Nội dung header

#endif
```

<a id="muc-16-07"></a>
### 16.7. `static inline` function

- `inline` cho compiler biết một hàm là ứng viên phù hợp để inline, nhưng compiler vẫn có quyền quyết định có inline thật hay không.
- `static inline` thường được dùng cho các hàm nhỏ trong header.

**Ví dụ:**

```c
static inline int square(int x)
{
    return x * x;
}
```

**Ưu điểm có thể có:**

- Giảm overhead của lời gọi hàm nhỏ.

**Nhược điểm có thể có:**

- Inline quá nhiều có thể làm tăng kích thước code.

**Dùng khi nào?**

- Hàm nhỏ, được gọi thường xuyên, đặc biệt là helper function trong code embedded.

<a id="muc-16-08"></a>
### 16.8. Memory functions

#### `memcpy()`

- Sao chép `n` byte từ vùng nhớ nguồn `src` sang vùng nhớ đích `dest`.

**Cú pháp:**

```c
void *memcpy(void *dest, const void *src, size_t n);
```

#### `memmove()`

- Sao chép `n` byte từ `src` sang `dest`.
- Khác với `memcpy()`, `memmove()` dùng được khi hai vùng nhớ có thể chồng lấn nhau.

**Cú pháp:**

```c
void *memmove(void *dest, const void *src, size_t n);
```

#### `memset()`

- Gán cùng một giá trị byte cho `n` byte liên tiếp trong vùng nhớ.

**Cú pháp:**

```c
void *memset(void *ptr, int c, size_t n);
```

#### `memcmp()`

- So sánh `n` byte của hai vùng nhớ `s1` và `s2`.

**Cú pháp:**

```c
int memcmp(const void *s1, const void *s2, size_t n);
```

**Giá trị trả về:**

- `0`: `n` byte của hai vùng nhớ giống nhau.
- `> 0`: tại byte khác đầu tiên, byte của `s1` lớn hơn byte của `s2`.
- `< 0`: tại byte khác đầu tiên, byte của `s1` nhỏ hơn byte của `s2`.

[↑ Về mục lục](#muc-luc)
