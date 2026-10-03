# Ghi chú phỏng vấn C

> **Mục tiêu:** Ôn phần C cho phỏng vấn.  
> **Cách dùng:** Học theo từng chương, dùng mục lục để nhảy nhanh đến chủ đề cần ôn.  

<a id="muc-luc"></a>
## Mục lục

1. [Tổng quan](#chuong-01)
   - [1.1. Hệ thống nhúng là gì?](#muc-01-01)
2. [Quá trình biên dịch](#chuong-02)
   - [2.1. Tiền xử lý](#muc-02-01)
   - [2.2. Biên dịch](#muc-02-02)
   - [2.3. Hợp dịch](#muc-02-03)
   - [2.4. Liên kết](#muc-02-04)
   - [2.5. Thư viện tĩnh và thư viện động](#muc-02-05)
3. [Kiến thức C cơ bản](#chuong-03)
   - [3.1. Ký tự đặc biệt trong C](#muc-03-01)
   - [3.2. Chú thích trong C](#muc-03-02)
   - [3.3. Các kiểu dữ liệu trong C](#muc-03-03)
   - [3.4. Phương pháp bù 2](#muc-03-04)
   - [3.5. Toán tử `sizeof`](#muc-03-05)
   - [3.6. Biến](#muc-03-06)
4. [Hàm](#chuong-04)
   - [4.1. Khái niệm hàm](#muc-04-01)
   - [4.2. Định nghĩa, khai báo và khởi tạo](#muc-04-02)
   - [4.3. Đối số và tham số](#muc-04-03)
   - [4.4. Truyền giá trị và truyền địa chỉ bằng con trỏ](#muc-04-04)
   - [4.5. Hàm có số lượng đối số thay đổi](#muc-04-05)
5. [Phạm vi và lớp lưu trữ](#chuong-05)
   - [5.1. Phạm vi của biến](#muc-05-01)
   - [5.2. Lớp lưu trữ](#muc-05-02)
6. [Chuyển đổi kiểu và địa chỉ](#chuong-06)
   - [6.1. Chuyển đổi kiểu](#muc-06-01)
   - [6.2. Địa chỉ của biến](#muc-06-02)
7. [Điều khiển luồng](#chuong-07)
   - [7.1. `if-else` và `switch-case`](#muc-07-01)
8. [Tổ chức bộ nhớ](#chuong-08)
   - [8.1. Bố cục bộ nhớ](#muc-08-01)
   - [8.2. Các section](#muc-08-02)
9. [Vòng lặp](#chuong-09)
   - [9.1. `for`, `while`, `do-while`](#muc-09-01)
   - [9.2. `continue`](#muc-09-02)
   - [9.3. `break`](#muc-09-03)
10. [Mảng](#chuong-10)
   - [10.1. Mảng một chiều](#muc-10-01)
   - [10.2. Mảng hai chiều](#muc-10-02)
   - [10.3. Mảng động và cấp phát bộ nhớ](#muc-10-03)
11. [Toán tử](#chuong-11)
   - [11.1. Các nhóm toán tử](#muc-11-01)
   - [11.2. Thao tác bit](#muc-11-02)
12. [Từ khóa quan trọng](#chuong-12)
   - [12.1. `volatile`](#muc-12-01)
   - [12.2. `const`](#muc-12-02)
13. [Kiểu dữ liệu tự định nghĩa](#chuong-13)
   - [13.1. `struct`](#muc-13-01)
   - [13.2. Trường bit](#muc-13-02)
   - [13.3. `union`](#muc-13-03)
14. [Chuỗi ký tự](#chuong-14)
15. [Con trỏ](#chuong-15)
   - [15.1. Khái niệm cơ bản](#muc-15-01)
   - [15.2. Con trỏ treo sau `free()`](#muc-15-02)
   - [15.3. Con trỏ treo khi trả về biến cục bộ](#muc-15-03)
   - [15.4. Con trỏ treo khi biến ra khỏi phạm vi](#muc-15-04)
   - [15.5. `const` với con trỏ](#muc-15-05)
   - [15.6. Con trỏ với mảng](#muc-15-06)
   - [15.7. Mảng con trỏ đến chuỗi ký tự](#muc-15-07)
   - [15.8. Mảng con trỏ `void *`](#muc-15-08)
   - [15.9. Con trỏ hàm](#muc-15-09)
   - [15.10. Mảng con trỏ hàm](#muc-15-10)
   - [15.11. Con trỏ trong hàm](#muc-15-11)
   - [15.12. Con trỏ hàm làm hàm gọi lại](#muc-15-12)
   - [15.13. Thay đổi biến thông qua con trỏ](#muc-15-13)
   - [15.14. Truy cập thành viên `struct` bằng con trỏ](#muc-15-14)
   - [15.15. Con trỏ `struct` trong tham số hàm](#muc-15-15)
   - [15.16. Con trỏ hàm là thành viên của `struct`](#muc-15-16)
16. [Các chủ đề C bổ sung](#chuong-16)
   - [16.1. `typedef`](#muc-16-01)
   - [16.2. Big Endian và Little Endian](#muc-16-02)
   - [16.3. `enum`](#muc-16-03)
   - [16.4. Tràn số](#muc-16-04)
   - [16.5. Liên kết nội bộ và liên kết ngoài](#muc-16-05)
   - [16.6. Bảo vệ tệp tiêu đề](#muc-16-06)
   - [16.7. Hàm `static inline`](#muc-16-07)
   - [16.8. Các hàm thao tác bộ nhớ](#muc-16-08)
   - [16.9. Các lỗi phổ biến trong C và cách gỡ lỗi](#muc-16-09)
17. [C trong phần mềm nhúng](#chuong-17)
   - [17.1. Cấp phát động](#muc-17-01)
   - [17.2. `volatile`, thanh ghi và ISR](#muc-17-02)
   - [17.3. Tính xác định](#muc-17-03)
   - [17.4. Những gì cần nhớ khi phỏng vấn](#muc-17-04)

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
## 2. Quá trình biên dịch

- Quá trình biên dịch là quá trình chuyển đổi từ ngôn ngữ bậc cao sang mã máy để bộ xử lý có thể hiểu và thực thi.

- Quá trình bao gồm 4 giai đoạn chính:
  1. Tiền xử lý (Preprocessing).
  2. Biên dịch (Compilation).
  3. Hợp dịch (Assembly).
  4. Liên kết (Linking).

<a id="muc-02-01"></a>
### 2.1. Tiền xử lý

- **Bộ thực hiện:** Bộ tiền xử lý (Preprocessor).

- **Đầu vào:** Tệp nguồn `.c`.

- **Nhiệm vụ:**

  - Xóa các chú thích.

  - Xử lý các chỉ thị bắt đầu bằng **#**.

  - Xử lý **#include**.

  - Xử lý nội dung của tệp được chèn tại vị trí **#include**.

  - Xử lý **#define**.

  - Thay thế macro.

  - Xử lý **#if**, **#ifdef**, **#ifndef**, **#elif**, **#else**, **#endif**.

  - Xử lý các macro được định nghĩa sẵn như:

    - **__FILE__**
    - **__LINE__**
    - **__DATE__**
    - **__TIME__**

  - Không thực hiện kiểm tra cú pháp C đầy đủ.

- **Kết quả:** Tạo ra tệp `.i`.

**Ví dụ:**

```c
#include "gpio.h"
#define LED_PIN 5
int main(void)
{
    gpio_write(LED_PIN, 1);
}
```

Sau bước tiền xử lý, nội dung **gpio.h** và giá trị macro **LED_PIN** sẽ được đưa vào mã nguồn tương ứng.

<a id="muc-02-02"></a>
### 2.2. Biên dịch

- **Bộ thực hiện:** Trình biên dịch (Compiler).

- **Đầu vào:** Tệp `.i`.

- **Nhiệm vụ:**

  - Phân tích cú pháp.

  - Kiểm tra kiểu dữ liệu.

  - Kiểm tra khai báo biến, hàm.

  - Kiểm tra phép toán và biểu thức.

  - Phát hiện các lỗi: lỗi cú pháp, biến/hàm chưa khai báo, sai kiểu dữ liệu, gọi hàm sai tham số, ...

  - Sinh cảnh báo nếu phát hiện mã nguồn có dấu hiệu bất thường.

  - Thực hiện tối ưu hóa tùy theo cờ biên dịch:

    - `-O0`
    - `-O1`
    - `-O2`
    - `-O3`
    - `-Os`
    - `-Og`

  - Chuyển mã nguồn C thành mã hợp ngữ phù hợp với kiến trúc đích.

**Ví dụ:**

`arm-none-eabi-gcc` sẽ sinh mã hợp ngữ dành cho ARM Cortex-M chứ không phải cho CPU của máy đang thực hiện biên dịch.

- **Kết quả:** Tạo ra tệp hợp ngữ `.s`.

Ví dụ C:

```c
int add(int a, int b)
{
    return a + b;
}
```

có thể được trình biên dịch chuyển thành mã hợp ngữ ARM tương ứng.

<a id="muc-02-03"></a>
### 2.3. Hợp dịch

- **Bộ thực hiện:** Trình hợp dịch (Assembler).

- **Đầu vào:** Tệp `.s`.

- **Nhiệm vụ:**

  - Chuyển mã Assembly thành mã máy mà CPU có thể thực thi.
  - Tạo ra tệp đối tượng chứa mã máy và thông tin cần thiết để trình liên kết xử lý ở bước sau.
  - Nếu bật chế độ gỡ lỗi, lưu thêm thông tin phục vụ việc gỡ lỗi.

- **Kết quả:** Tạo ra tệp đối tượng `.o`.

**Ví dụ:**

```text
main.s
  ↓
Assembler
  ↓
main.o
```

Tệp `.o` **chưa phải firmware hoàn chỉnh**. Nó có thể vẫn chứa các ký hiệu (symbol) chưa được giải quyết.

**Ví dụ:**

**gpio_write**(...)**;**

Trong `main.o`, lời gọi `gpio_write()` có thể tồn tại dưới dạng một tham chiếu tới ký hiệu chưa được giải quyết. Đến bước liên kết, trình liên kết sẽ tìm phần định nghĩa của ký hiệu này và xác định địa chỉ phù hợp.

<a id="muc-02-04"></a>
### 2.4. Liên kết

- **Bộ thực hiện:** Trình liên kết (Linker).

- **Đầu vào:** Các tệp đối tượng `.o` và các thư viện cần thiết.

- **Nhiệm vụ:**

  - Ghép các phần của chương trình đã được biên dịch lại với nhau.

  - Tìm và nối các hàm, biến được sử dụng ở tệp này nhưng được định nghĩa ở tệp khác.

  - Liên kết thêm các thư viện mà chương trình sử dụng.

  - Phát hiện các lỗi như:

    - Gọi một hàm nhưng không tìm thấy phần định nghĩa (`undefined reference`).
    - Một hàm hoặc biến bị định nghĩa nhiều lần không hợp lệ (`multiple definition`).

- **Kết quả:** Tạo ra tệp thực thi/chương trình đã được liên kết hoàn chỉnh, ví dụ `.elf`.

#### Ví dụ

Tệp **main.c**:

```c
int add(int a, int b);
int main(void)
{
    return add(1, 2);
}
```

Tệp **add.c**:

```c
int add(int a, int b)
{
    return a + b;
}
```

Khi biên dịch riêng **main.c**, trình biên dịch biết rằng hàm **add()** tồn tại nhờ phần khai báo:

```c
int add(int a, int b);
```

Nhưng phần mã của **add()** nằm trong **add.c**. Đến bước liên kết, trình liên kết sẽ nối lời gọi **add()** trong **main.c** với phần định nghĩa **add()** trong **add.c**. Nếu không tìm thấy phần định nghĩa của **add()**, quá trình liên kết sẽ bị lỗi.

<a id="muc-02-05"></a>
### 2.5. Thư viện tĩnh và thư viện động

#### Thư viện tĩnh — Static Library

- Thư viện tĩnh là tập hợp nhiều tệp đối tượng `.o` được đóng gói lại thành một thư viện.
- Với GCC, thư viện tĩnh thường có đuôi `.a`.

**Ví dụ:**

```text
add.o
sub.o
mul.o
   ↓
libmath.a
```

- Khi liên kết chương trình, trình liên kết lấy các phần cần thiết từ thư viện tĩnh và đưa chúng vào chương trình cuối cùng.

```text
main.o + libmath.a
        ↓
      Linker
        ↓
    firmware.elf
```

**Ưu điểm:**

- Chương trình sau khi liên kết không cần thư viện bên ngoài để chạy.
- Phù hợp với hệ thống nhúng Bare-metal.
- Việc triển khai đơn giản.

**Nhược điểm:**

- Mã của thư viện được đưa vào chương trình nên có thể làm tăng kích thước tệp thực thi.
- Nếu thư viện thay đổi thì chương trình thường phải liên kết lại.

**Ví dụ tạo Static Library bằng GCC:**

```bash
arm-none-eabi-ar rcs libdriver.a gpio.o uart.o
```

Sau đó có thể liên kết thư viện này với các tệp `.o` khác để tạo firmware.

#### Thư viện động — Dynamic Library / Shared Library

- Thư viện động không được sao chép toàn bộ vào chương trình tại thời điểm liên kết.
- Chương trình lưu thông tin để sử dụng thư viện đó khi được nạp hoặc trong lúc chạy.
- Trên Linux thường có đuôi `.so`.
- Trên Windows thường có đuôi `.dll`.

**Ví dụ:**

```text
application
    |
    └── sử dụng libmath.so khi chương trình chạy
```

**Ưu điểm:**

- Giảm kích thước của từng chương trình.
- Nhiều chương trình có thể dùng chung một thư viện.
- Có thể cập nhật thư viện mà không cần nhúng toàn bộ mã thư viện vào từng chương trình.

**Nhược điểm:**

- Chương trình phụ thuộc vào thư viện bên ngoài.
- Nếu thiếu thư viện hoặc sai phiên bản, chương trình có thể không chạy được.
- Cơ chế nạp và quản lý phức tạp hơn thư viện tĩnh.

#### So sánh

| Tiêu chí | Static Library | Dynamic Library |
|---|---|---|
| Thời điểm chính | Liên kết vào chương trình khi Linking | Được nạp/liên kết khi chương trình được nạp hoặc chạy |
| Đuôi thường gặp | `.a` | `.so`, `.dll` |
| Mã thư viện nằm trong chương trình | Có | Không toàn bộ |
| Phụ thuộc thư viện bên ngoài khi chạy | Không | Có |
| Kích thước chương trình | Thường lớn hơn | Thường nhỏ hơn |
| Bare-metal Embedded | Rất phổ biến | Hầu như không dùng |
| Embedded Linux | Có thể dùng | Có thể dùng |

#### Trong Embedded

- Với vi điều khiển như STM32 chạy Bare-metal hoặc RTOS, **Static Library phổ biến hơn nhiều**.
- Firmware thường được liên kết hoàn chỉnh thành một tệp `.elf`, sau đó có thể chuyển thành `.bin` hoặc `.hex`.
- Dynamic Library thường gặp hơn trên các hệ thống có hệ điều hành như Embedded Linux.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-03"></a>
## 3. Kiến thức C cơ bản

<a id="muc-03-01"></a>
### 3.1. Ký tự đặc biệt trong C

- Ký tự đặc biệt, hay **chuỗi thoát (escape sequence)**, là các chuỗi ký tự bắt đầu bằng dấu `\` để biểu diễn những ký tự khó viết trực tiếp trong mã nguồn.

**Ví dụ:**

| Chuỗi thoát | Ý nghĩa |
|---|---|
| `\n` | Xuống dòng. |
| `\r` | Đưa con trỏ về đầu dòng. |

<a id="muc-03-02"></a>
### 3.2. Chú thích trong C

- Chú thích một dòng:

```c
// Nội dung chú thích
```

- Chú thích nhiều dòng:

```c
/* Nội dung chú thích */
```

<a id="muc-03-03"></a>
### 3.3. Các kiểu dữ liệu trong C

- Kiểu dữ liệu xác định loại giá trị mà biến có thể lưu trữ hoặc hàm có thể trả về.
- Kích thước của kiểu dữ liệu phụ thuộc vào trình biên dịch và kiến trúc hệ thống.
- Có thể dùng toán tử `sizeof` để kiểm tra kích thước của một kiểu hoặc đối tượng dữ liệu.

**Các nhóm kiểu dữ liệu chính:**

- **Kiểu số nguyên:** `char`, `signed char`, `unsigned char`, `short`, `int`, `long`, `long long` và các dạng `unsigned` tương ứng.
- **Kiểu dấu phẩy động:** `float`, `double`, `long double`.
- **Kiểu liệt kê:** `enum`.
- **Kiểu rỗng:** `void`.
- **Kiểu dẫn xuất:** mảng, con trỏ, hàm, `struct`, `union`.

#### Kiểu `char`

#### `short` và `unsigned short`

#### `int` và `unsigned int`

#### `long` và `unsigned long`

#### `long long` và `unsigned long long`

#### Kiểu dấu phẩy động: `float`, `double`, `long double`

- Dấu phẩy động là cách biểu diễn số thực trong đó vị trí dấu phẩy có thể thay đổi nhờ phần số mũ.
- Phần lớn hệ thống hiện nay sử dụng chuẩn IEEE 754 để biểu diễn số thực.

Một số dấu phẩy động thường gồm 3 phần:

- **Sign:** bit dấu của số.
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

- `sizeof` trả về kích thước theo byte của một kiểu dữ liệu hoặc đối tượng dữ liệu.

**Ví dụ:**

```c
sizeof(int)
sizeof(double)
```

<a id="muc-03-06"></a>
### 3.6. Biến

- Biến là một vùng dữ liệu được đặt tên dùng để lưu trữ giá trị trong chương trình.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-04"></a>
## 4. Hàm

<a id="muc-04-01"></a>
### 4.1. Khái niệm hàm

- Hàm là một khối lệnh được đặt tên, dùng để thực hiện một nhiệm vụ cụ thể.

**Tại sao dùng hàm?**

- Mã nguồn dễ đọc và mạch lạc hơn.
- Dễ gỡ lỗi và bảo trì.
- Tăng khả năng tái sử dụng mã nguồn.

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

    printf("Ket qua: %d\n", result);
    return 0;
}
```

**Các lỗi cần chú ý:**

- `a` và `b` có kiểu `long long`, nhưng tham số `x`, `y` lại là `int` nên có nguy cơ mất dữ liệu khi chuyển kiểu.
- Phép nhân `x * y` được thực hiện theo kiểu `int` trước; nếu vượt phạm vi của `int` thì có thể bị tràn số trước khi kết quả được trả về dưới dạng `long long`.
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

    printf("Ket qua: %lld\n", result);
    return 0;
}
```

#### Nguyên mẫu hàm

- Nguyên mẫu hàm cung cấp cho trình biên dịch thông tin về tên hàm, kiểu trả về và danh sách tham số trước khi phần định nghĩa hàm xuất hiện.

**Ví dụ:**

```c
int sum(int a, int b);
int sum(int, int);
```

<a id="muc-04-02"></a>
### 4.2. Định nghĩa, khai báo và khởi tạo

#### Định nghĩa

- Định nghĩa cung cấp đầy đủ đối tượng dữ liệu hoặc phần thân của hàm.

**Ví dụ:**

```c
int x;

int tinhTong(int a, int b)
{
    return a + b;
}
```

#### Khai báo

- Khai báo thông báo cho trình biên dịch biết tên và kiểu của một biến hoặc hàm.

**Ví dụ:**

```c
extern int x;
int tinhTong(int a, int b);
```

#### Khởi tạo

- Khởi tạo là gán giá trị ban đầu cho biến ngay khi biến được định nghĩa.

**Ví dụ:**

```c
int x = 10;
```

<a id="muc-04-03"></a>
### 4.3. Đối số và tham số

- **Đối số (argument):** giá trị thực tế được truyền vào khi gọi hàm.
- **Tham số (parameter):** biến được khai báo trong phần định nghĩa hàm để nhận đối số.

**Ví dụ:**

```c
#include <stdio.h>

int tinhTong(int a, int b)   // a, b là tham số
{
    return a + b;
}

int main(void)
{
    int x = 10;
    int y = 20;

    int kq1 = tinhTong(5, 3);   // 5, 3 là đối số
    int kq2 = tinhTong(x, y);   // x, y là đối số

    return 0;
}
```

<a id="muc-04-04"></a>
### 4.4. Truyền giá trị và truyền địa chỉ bằng con trỏ

- Trong C, **mọi đối số đều được truyền theo giá trị** (*pass by value*).
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
### 4.5. Hàm có số lượng đối số thay đổi

- Hàm này cho phép nhận số lượng đối số thay đổi.
- Hàm có ít nhất một tham số cố định, sau đó là `...` để biểu thị phần đối số có số lượng thay đổi.

**Các thành phần cơ bản:**

- `va_list`: lưu trạng thái khi duyệt danh sách đối số.
- `va_start`: bắt đầu truy cập các đối số có số lượng thay đổi.
- `va_arg`: lấy từng đối số theo kiểu được chỉ định.
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
## 5. Phạm vi và lớp lưu trữ

<a id="muc-05-01"></a>
### 5.1. Phạm vi của biến

#### Biến cục bộ

- Định nghĩa: Là biến được khai báo bên trong một hàm hoặc bên trong một khối lệnh (nằm giữa cặp ngoặc nhọn {}).

- **Phạm vi:** Chỉ có thể truy cập và sử dụng bên trong chính hàm hoặc khối lệnh chứa nó. Các hàm khác bên ngoài không thể nhìn thấy biến này.

- **Thời gian tồn tại:** Với biến cục bộ có thời gian lưu trữ tự động, biến bắt đầu tồn tại khi chương trình đi vào khối lệnh và hết thời gian tồn tại khi rời khối. Trong thực tế, trình biên dịch có thể đặt biến trên ngăn xếp hoặc thanh ghi tùy cách tối ưu.

- Nếu không được khởi tạo, giá trị của biến cục bộ là **không xác định**; không nên đọc giá trị trước khi gán.

#### Biến toàn cục

- **Định nghĩa:** Là biến được khai báo bên ngoài tất cả các hàm và khối lệnh, thường được đặt gần đầu tệp mã nguồn.

- **Phạm vi:** Có phạm vi trong tệp kể từ vị trí khai báo phù hợp. Các hàm có thể truy cập biến khi tên biến nằm trong phạm vi nhìn thấy của chúng.

- **Thời gian tồn tại:** Tồn tại trong suốt thời gian chạy của chương trình.

- Mặc định được khởi tạo với giá trị 0. (Giải thích: biến toàn cục không được khởi tạo sẽ nằm trên vùng BSS, được nạp giá trị bằng 0 trước khi vào main()).

```c
#include <stdio.h>
int global_num = 5; // Biến toàn cục
void Sum(int a, int b){
    int sum = a + b; // Biến cục bộ
    printf("Tong = %d\n", sum);
}
int main(){
    int local_num = 10;
    Sum(global_num, local_num);
}
```

- Hiện tượng che biến là hiện tượng xảy ra khi một biến được khai báo bên trong một phạm vi hẹp (trong hàm, trong vòng lặp, trong phạm vi {}, ... ) có cùng tên với một biến ở phạm vi rộng hơn bên ngoài. Lúc này, biến ở phạm vi hẹp sẽ che biến bên ngoài, khiến chương trình ưu tiên truy cập vào biến bên trong.

- Ví dụ:

```c
#include <stdio.h>
int valueA = 5;
int main()
{
  {
    int valueA = 20;
    printf("Gia tri A: %d \n", valueA); // Kết quả: 20
  }
   printf("Gia tri A: %d \n", valueA); // Kết quả: 5
}
```

<a id="muc-05-02"></a>
### 5.2. Lớp lưu trữ

- Lớp lưu trữ giúp mô tả **phạm vi sử dụng, thời gian tồn tại và kiểu liên kết** của biến hoặc hàm. Vị trí vật lý thực tế của biến còn phụ thuộc trình biên dịch, kiến trúc và mức tối ưu.

#### `auto`

- Là lớp lưu trữ mặc định của biến cục bộ thông thường.
- Biến có phạm vi khối và thời gian lưu trữ tự động: được tạo khi đi vào khối và hết thời gian tồn tại khi rời khối.
- Chuẩn C **không bắt buộc** biến `auto` phải nằm trên ngăn xếp, dù trong thực tế trình biên dịch thường dùng ngăn xếp hoặc thanh ghi tùy trường hợp.
- Nếu không khởi tạo, giá trị của biến cục bộ là **không xác định**.

```c
int x;
// tương đương về lớp lưu trữ với:
auto int x;
```

#### `register`

- Là lời gợi ý rằng biến được sử dụng thường xuyên và trình biên dịch có thể tối ưu cách lưu trữ/truy cập biến.
- Trình biên dịch **không bắt buộc** phải đặt biến vào thanh ghi CPU.
- Phạm vi và thời gian tồn tại tương tự biến `auto`.
- Không được dùng toán tử `&` để lấy địa chỉ của biến khai báo `register`.

```c
register int i;
```

#### `static`

- Với biến cục bộ: biến có phạm vi cục bộ nhưng tồn tại suốt thời gian chạy của chương trình, nên giá trị được giữ lại giữa các lần gọi hàm.
- Với biến/hàm ở phạm vi tệp: `static` còn tạo **liên kết nội bộ**, nghĩa là tên đó chỉ được dùng trong đơn vị dịch hiện tại.
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

- Dùng để khai báo rằng một biến hoặc hàm có phần định nghĩa ở nơi khác.
- `extern` thường được dùng để truy cập một đối tượng có **liên kết ngoài** từ tệp khác.
- Bản thân khai báo `extern` thường không tạo thêm một vùng lưu trữ mới cho biến.

```c
extern int global_score;
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-06"></a>
## 6. Chuyển đổi kiểu và địa chỉ

<a id="muc-06-01"></a>
### 6.1. Chuyển đổi kiểu

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

- Địa chỉ biểu diễn vị trí của một đối tượng dữ liệu trong không gian địa chỉ của chương trình.

- Dùng toán tử `&` để lấy địa chỉ của một đối tượng dữ liệu khi phép lấy địa chỉ đó hợp lệ.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-07"></a>
## 7. Điều khiển luồng

<a id="muc-07-01"></a>
### 7.1. `if-else` và `switch-case`

| Tiêu chí | `if-else` | `switch-case` |
|---|---|---|
| Loại điều kiện | Có thể dùng các biểu thức điều kiện và kiểm tra giá trị khác `0` / bằng `0`. | So sánh biểu thức `switch` với các nhãn `case` là hằng số nguyên phù hợp. |
| Kiểu dữ liệu | Biểu thức điều kiện thường là kiểu vô hướng. | Biểu thức `switch` dùng kiểu số nguyên hoặc `enum`. |
| Số lượng nhánh | Phù hợp khi điều kiện linh hoạt hoặc số nhánh không quá nhiều. | Dễ đọc khi có nhiều nhánh dựa trên một giá trị. |
| Tốc độ thực thi | Phụ thuộc vào điều kiện và cách trình biên dịch tối ưu. | Cũng phụ thuộc trình biên dịch; có thể dùng chuỗi so sánh hoặc bảng nhảy. |

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-08"></a>
## 8. Tổ chức bộ nhớ

<a id="muc-08-01"></a>
### 8.1. Bố cục bộ nhớ

- **Vùng mã (Text/Code Segment):** thường chứa mã máy/lệnh thực thi của chương trình.
- **Vùng dữ liệu (Data Segment):** thường chứa biến toàn cục hoặc biến `static` đã được khởi tạo với giá trị khác 0.
- **Vùng BSS:** thường chứa biến toàn cục hoặc biến `static` chưa khởi tạo hoặc được khởi tạo bằng 0.
- **Heap:** vùng nhớ thường được sử dụng cho cấp phát động như `malloc()`, `calloc()`, `realloc()`.
- **Stack:** thường được dùng cho khung ngăn xếp của hàm, biến cục bộ, địa chỉ trả về và dữ liệu điều khiển liên quan đến lời gọi hàm; chi tiết phụ thuộc trình biên dịch và kiến trúc.

#### Rò rỉ bộ nhớ

- Rò rỉ bộ nhớ xảy ra khi chương trình cấp phát vùng nhớ động nhưng không giải phóng khi không còn sử dụng.

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
- **`.data`:** chứa biến toàn cục hoặc biến `static` đã được khởi tạo.
- **`.bss`:** chứa biến toàn cục hoặc biến `static` chưa khởi tạo hoặc được khởi tạo bằng 0.
- **`.symtab`:** chứa bảng ký hiệu để các công cụ như trình liên kết và trình gỡ lỗi sử dụng.

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
### 10.1. Mảng một chiều

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

- **Mảng và con trỏ là hai kiểu khác nhau.** Tuy nhiên, trong hầu hết biểu thức, tên mảng sẽ tự chuyển thành con trỏ tới phần tử đầu tiên.

**Truy cập phần tử:**

```c
someData[1]
*(someData + 1)
```

<a id="muc-10-02"></a>
### 10.2. Mảng hai chiều

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

- Trình biên dịch cần biết kích thước của các chiều phía sau để tính đúng địa chỉ phần tử.
- Với `arr[n][m]`, địa chỉ phần tử có thể hiểu theo công thức:

```text
dia_chi(arr[i][j])
= dia_chi_co_so + (i × m + j) × kich_thuoc_phan_tu
```

<a id="muc-10-03"></a>
### 10.3. Mảng động và cấp phát bộ nhớ

- Mảng động là vùng nhớ dùng để lưu nhiều phần tử và được cấp phát trong thời gian chạy.
- Thường được cấp phát từ Heap bằng `malloc()`, `calloc()`, `realloc()` và giải phóng bằng `free()`.
- Các hàm này được khai báo trong:

```c
#include <stdlib.h>
```

#### So sánh `malloc()`, `calloc()` và `realloc()`

| Hàm | Mục đích | Tham số kích thước | Giá trị ban đầu | Thành công trả về | Thất bại trả về |
|---|---|---|---|---|---|
| `malloc()` | Cấp phát vùng nhớ mới | Tổng số byte | Chưa được khởi tạo | `void *` trỏ tới vùng nhớ mới | `NULL` |
| `calloc()` | Cấp phát vùng nhớ mới cho nhiều phần tử | Số phần tử và kích thước mỗi phần tử | Toàn bộ byte được đặt về `0` | `void *` trỏ tới vùng nhớ mới | `NULL` |
| `realloc()` | Thay đổi kích thước vùng nhớ đã cấp phát | Kích thước mới tổng cộng | Giữ lại dữ liệu cũ trong phần kích thước còn phù hợp | `void *` trỏ tới vùng nhớ sau khi thay đổi kích thước | `NULL` |

> Trong C, `void *` do `malloc()`, `calloc()` và `realloc()` trả về có thể gán trực tiếp cho con trỏ kiểu dữ liệu phù hợp mà không cần ép kiểu.

#### `malloc()`

- `malloc()` yêu cầu cấp phát một vùng nhớ có kích thước xác định theo **tổng số byte**.
- Vùng nhớ mới **chưa được khởi tạo**.
- Nếu thành công, trả về con trỏ `void *` trỏ tới vùng nhớ mới.
- Nếu thất bại, trả về `NULL`.

**Ví dụ:**

```c
int *ptr = malloc(10 * sizeof(int));

if (ptr == NULL) {
    // Cấp phát thất bại
}
```

Có thể hiểu:

```text
malloc(10 * sizeof(int))
        ↓
cấp phát đủ chỗ cho 10 int
        ↓
giá trị ban đầu chưa xác định
```

#### `calloc()`

- `calloc()` cấp phát vùng nhớ cho nhiều phần tử.
- Nhận riêng:
  - Số lượng phần tử.
  - Kích thước của mỗi phần tử.
- Toàn bộ byte trong vùng nhớ mới được đặt về `0`.
- Nếu thành công, trả về con trỏ `void *` trỏ tới vùng nhớ mới.
- Nếu thất bại, trả về `NULL`.

**Ví dụ:**

```c
int *ptr = calloc(10, sizeof(int));

if (ptr == NULL) {
    // Cấp phát thất bại
}
```

Có thể hiểu:

```text
calloc(10, sizeof(int))
       ↓
10 phần tử × sizeof(int)
       ↓
toàn bộ byte được đặt về 0
```

So sánh nhanh:

```c
int *a = malloc(10 * sizeof(int));
int *b = calloc(10, sizeof(int));
```

```text
a -> vùng nhớ chưa được khởi tạo
b -> toàn bộ byte trong vùng nhớ được đặt về 0
```

> Không nên hiểu rằng `calloc()` luôn tạo ra giá trị `NULL` cho mọi kiểu con trỏ hoặc `0.0` cho mọi kiểu dấu phẩy động trên mọi hệ thống; điều được đảm bảo là các byte được đặt về `0`.

#### `realloc()`

- `realloc()` dùng để thay đổi kích thước của vùng nhớ đã được cấp phát trước đó.
- Tham số kích thước là **kích thước mới tổng cộng**, tính theo byte.
- Nếu thành công:
  - Trả về con trỏ tới vùng nhớ sau khi thay đổi kích thước.
  - Địa chỉ mới có thể giống hoặc khác địa chỉ cũ.
  - Dữ liệu cũ được giữ lại trong phần kích thước còn phù hợp.
- Nếu thất bại:
  - Trả về `NULL`.
  - Vùng nhớ cũ vẫn còn được cấp phát và con trỏ cũ vẫn còn hợp lệ.

**Khi tăng kích thước vùng nhớ:**

- Nếu vùng nhớ cũ **có thể mở rộng tại chỗ**, `realloc()` có thể giữ nguyên địa chỉ và mở rộng vùng hiện tại.
- Nếu vùng nhớ cũ **không thể mở rộng tại chỗ**, `realloc()` có thể chuyển dữ liệu sang một vùng nhớ mới đủ lớn; khi thành công, vùng nhớ cũ được giải phóng và con trỏ trả về trỏ tới vùng nhớ mới.
- Dữ liệu cũ được giữ lại trong phần kích thước tương ứng.
- Phần vùng nhớ mới được thêm vào **không được khởi tạo**, nên có giá trị không xác định.
- Nếu `realloc()` thất bại, vùng nhớ cũ **không bị giải phóng**.

Có thể hình dung:

```text
Có thể mở rộng tại chỗ:
[ dữ liệu cũ ][ phần mới chưa khởi tạo ]
^
địa chỉ có thể giữ nguyên

Không thể mở rộng tại chỗ:
vùng cũ                     vùng mới
[ dữ liệu cũ ]   --->   [ dữ liệu cũ ][ phần mới chưa khởi tạo ]
                        ^
                    địa chỉ trả về

realloc() thất bại:
→ trả về NULL
→ vùng cũ vẫn còn nguyên
```

**Ví dụ:**

```c
int *ptr = malloc(5 * sizeof(int));
```

Ban đầu:

```text
ptr -> [ ][ ][ ][ ][ ]
```

Muốn tăng lên 10 phần tử:

```c
int *temp = realloc(ptr, 10 * sizeof(int));

if (temp != NULL) {
    ptr = temp;
} else {
    // realloc thất bại.
    // ptr vẫn giữ vùng nhớ cũ.
}
```

Sau khi `realloc()` thành công, có thể hình dung:

```text
ptr -> [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ]
```

Địa chỉ có thể:

```text
Trường hợp 1:
Vùng nhớ cũ còn đủ chỗ để mở rộng
→ địa chỉ có thể giữ nguyên.

Trường hợp 2:
Không thể mở rộng tại chỗ
→ hệ thống có thể cấp phát vùng mới
→ sao chép dữ liệu cũ
→ trả về địa chỉ mới.
```

**Không nên viết trực tiếp:**

```c
ptr = realloc(ptr, new_size);
```

Vì nếu `realloc()` thất bại và trả về `NULL`, ta sẽ làm mất con trỏ đang giữ địa chỉ vùng nhớ cũ.

Nên dùng con trỏ tạm:

```c
int *temp = realloc(ptr, new_size);

if (temp != NULL) {
    ptr = temp;
}
```

#### `free()`

- `free()` giải phóng vùng nhớ động khi không còn sử dụng.
- `free()` không trả về giá trị.

**Ví dụ:**

```c
free(ptr);
ptr = NULL;
```

- Sau `free()`, không được tiếp tục giải tham chiếu địa chỉ cũ.
- Gán con trỏ về `NULL` giúp giảm nguy cơ vô tình sử dụng lại con trỏ đã được giải phóng.

#### Ý cần nhớ

```text
malloc()
→ cấp phát vùng nhớ
→ chưa khởi tạo
→ thành công: trả về con trỏ
→ thất bại: NULL

calloc()
→ cấp phát vùng nhớ
→ đặt toàn bộ byte về 0
→ thành công: trả về con trỏ
→ thất bại: NULL

realloc()
→ thay đổi kích thước vùng đã cấp phát
→ thành công: trả về con trỏ mới hoặc cùng địa chỉ cũ
→ thất bại: NULL, vùng nhớ cũ vẫn còn

free()
→ giải phóng vùng nhớ
→ không trả về giá trị
```

**Lưu ý trong hệ thống nhúng:**

- Cấp phát/giải phóng động nhiều lần có thể gây phân mảnh bộ nhớ.
- Thời gian cấp phát có thể khó dự đoán hơn bộ nhớ tĩnh.
- Với hệ thống yêu cầu tính xác định cao, cần cân nhắc kỹ trước khi dùng cấp phát động.
- Mảng cục bộ quá lớn có thể làm tăng nguy cơ tràn ngăn xếp.

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

**Tăng trước:**

```c
int i = 5;
int result = ++i;

// i = 6
// result = 6
```

**Tăng sau:**

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

- Thao tác bit là việc đọc hoặc thay đổi trực tiếp từng bit trong một giá trị.
- Trong Embedded, thao tác bit thường được dùng để:
  - Điều khiển các bit trong thanh ghi ngoại vi.
  - Bật/tắt một chức năng phần cứng.
  - Kiểm tra các cờ trạng thái.
  - Đóng gói nhiều trạng thái vào một biến.

#### Các toán tử thao tác bit

| Toán tử | Ý nghĩa |
|---|---|
| `&` | AND bit |
| `|` | OR bit |
| `^` | XOR bit |
| `~` | Đảo bit |
| `<<` | Dịch trái |
| `>>` | Dịch phải |

**Ví dụ:**

```c
uint8_t a = 0b00001100;
uint8_t b = 0b00001010;

a & b;   // 00001000
a | b;   // 00001110
a ^ b;   // 00000110
~a;      // Đảo tất cả các bit
```

#### Mặt nạ bit

- Mặt nạ bit là một giá trị dùng để xác định bit nào cần thao tác.

**Ví dụ muốn thao tác với bit số 3:**

```c
uint8_t mask = (1U << 3);
```

**Kết quả:**

```text
00001000
```

#### Bật một bit

- Dùng toán tử OR `|`.

```c
value |= (1U << bit);
```

**Ví dụ bật bit số 3:**

```c
value |= (1U << 3);
```

Nếu ban đầu:

```text
value = 00000000
```

Sau khi bật bit số 3:

```text
value = 00001000
```

#### Xóa một bit

- Dùng AND `&` kết hợp với đảo bit `~`.

```c
value &= ~(1U << bit);
```

**Ví dụ xóa bit số 3:**

```c
value &= ~(1U << 3);
```

#### Đảo trạng thái một bit

- Dùng XOR `^`.

```c
value ^= (1U << bit);
```

- Nếu bit đang bằng `0` thì thành `1`.
- Nếu bit đang bằng `1` thì thành `0`.

#### Kiểm tra một bit

- Dùng AND `&`.

```c
if (value & (1U << bit))
{
    // Bit đang bằng 1
}
```

**Ví dụ kiểm tra bit số 3:**

```c
if (value & (1U << 3))
{
    // Bit số 3 đang được bật
}
```

#### Dịch bit

**Dịch trái:**

```c
value << n;
```

- Các bit được dịch sang trái `n` vị trí.

**Ví dụ:**

```text
00000001 << 3
00001000
```

**Dịch phải:**

```c
value >> n;
```

- Các bit được dịch sang phải `n` vị trí.

**Ví dụ:**

```text
00001000 >> 3
00000001
```

#### Ví dụ trong Embedded

Giả sử bit số 5 của một thanh ghi dùng để bật một ngoại vi:

```c
#define ENABLE_BIT (1U << 5)

REGISTER |= ENABLE_BIT;      // Bật
REGISTER &= ~ENABLE_BIT;     // Tắt
REGISTER ^= ENABLE_BIT;      // Đảo trạng thái

if (REGISTER & ENABLE_BIT)
{
    // Ngoại vi đang được bật
}
```

#### Phân biệt toán tử bit và toán tử logic

- `&`, `|`, `^`, `~`: thao tác trên từng bit.
- `&&`, `||`, `!`: dùng cho biểu thức logic.

**Ví dụ:**

```c
a & b     // AND bit
a && b    // AND logic
```

#### Công thức cần nhớ

```c
value |=  (1U << n);   // Bật bit
value &= ~(1U << n);   // Xóa bit
value ^=  (1U << n);   // Đảo bit
value &   (1U << n);   // Kiểm tra bit
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-12"></a>
## 12. Từ khóa quan trọng

<a id="muc-12-01"></a>
### 12.1. `volatile`

- `volatile` dùng để báo cho trình biên dịch biết rằng giá trị của đối tượng có thể thay đổi ngoài luồng thực thi thông thường của đoạn mã hiện tại.
- Vì vậy, các lần đọc/ghi tới đối tượng `volatile` phải được trình biên dịch giữ lại theo đúng yêu cầu của chương trình, không được tự ý loại bỏ hoặc gộp các lần truy cập như với biến thông thường.
- `volatile` **không có nghĩa** là biến luôn nằm trong RAM, không đảm bảo thao tác là nguyên tử và cũng không tự làm mã nguồn trở nên an toàn khi nhiều luồng cùng truy cập.

- Dùng khi nào?
  - Truy cập thanh ghi ngoại vi hoặc thanh ghi ánh xạ bộ nhớ.
  - Biến được thay đổi trong ISR và được đọc ở mã nguồn chính.
  - Một số trường hợp dữ liệu có thể thay đổi bởi phần cứng hoặc tác nhân bên ngoài luồng thực thi hiện tại.

<a id="muc-12-02"></a>
### 12.2. `const`

- Định nghĩa: `const` dùng để khai báo rằng dữ liệu không được phép sửa đổi thông qua tên hoặc con trỏ `const` đó. Nếu cố gán lại trực tiếp cho một đối tượng được khai báo `const`, trình biên dịch sẽ báo lỗi.

- Dùng khi nào? Bất cứ khi nào không muốn thay đổi giá trị của biến, tham số hàm ở các dòng lệnh tiếp theo.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-13"></a>
## 13. Kiểu dữ liệu tự định nghĩa

<a id="muc-13-01"></a>
### 13.1. `struct`

- `struct` cho phép gom nhiều thành phần có kiểu dữ liệu khác nhau vào cùng một kiểu dữ liệu.
- Dùng `.` để truy cập thành viên từ một biến `struct`.
- Dùng `->` để truy cập thành viên thông qua con trỏ `struct`.

#### Căn chỉnh dữ liệu

- Mỗi kiểu dữ liệu có thể có một **yêu cầu căn chỉnh**.
- Trình biên dịch thường đặt dữ liệu tại địa chỉ phù hợp với yêu cầu căn chỉnh để CPU truy cập hiệu quả và đúng quy tắc của kiến trúc.
- Truy cập dữ liệu không căn chỉnh có thể chậm hơn; trên một số kiến trúc còn có thể gây lỗi.

**Ví dụ với kiểu `int` yêu cầu căn chỉnh 4 byte:**

- Địa chỉ như `0x...04` phù hợp với yêu cầu căn chỉnh 4 byte.
- Địa chỉ như `0x...02` có thể là địa chỉ không căn chỉnh cho `int`.

#### Phần đệm của `struct`

- Trình biên dịch có thể chèn các byte **đệm** giữa các thành viên để đáp ứng yêu cầu căn chỉnh.
- Cuối `struct` cũng có thể có byte đệm.
- Vì vậy, `sizeof(struct)` không nhất thiết bằng tổng `sizeof` của từng thành viên.

**Ví dụ phụ thuộc trình biên dịch để yêu cầu `struct` được đóng gói:**

```c
__attribute__((packed))
```

- `packed` có thể tiết kiệm bộ nhớ nhưng cũng có thể tạo truy cập không căn chỉnh.

<a id="muc-13-02"></a>
### 13.2. Trường bit

- Trường bit cho phép chỉ định số bit mà một thành viên trong `struct` sử dụng.

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
- Có thể gặp khi mô tả thanh ghi, nhưng cách bố trí trường bit phụ thuộc cách triển khai nên cần dùng cẩn thận.

<a id="muc-13-03"></a>
### 13.3. `union`

- `union` cho phép nhiều thành viên dùng chung cùng một vùng nhớ.
- Tại một thời điểm, vùng nhớ chung đó chứa biểu diễn của thành viên đang được sử dụng.
- Kích thước của `union` phải đủ chứa thành viên lớn nhất và có thể lớn hơn do yêu cầu căn chỉnh.
- Thường được dùng khi muốn tiết kiệm bộ nhớ hoặc diễn giải cùng vùng dữ liệu theo nhiều kiểu khác nhau.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-14"></a>
## 14. Chuỗi ký tự

- Trong C, chuỗi ký tự là một mảng `char` kết thúc bằng ký tự rỗng `\0`.

### Cách khởi tạo

```c
char str[] = "Hello World!";
```

Mảng ký tự sau **chưa phải chuỗi hợp lệ** vì thiếu `\0`:

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

### Phân biệt chuỗi và ký tự

```c
char str[] = "A";  // Chuỗi: gồm 'A' và '\0'
char ch    = 'A';  // Một ký tự
```

- Chuỗi dùng dấu nháy kép `"..."`.
- Một ký tự dùng dấu nháy đơn `'...'`.

### ASCII

- ASCII chuẩn mã hóa 128 giá trị ký tự.

### Các hàm xử lý chuỗi thường dùng

- `strlen(str)`: trả về độ dài chuỗi, không tính `\0`.
- `strcpy(dest, src)`: sao chép chuỗi `src` sang `dest`.
- `strncpy(dest, src, n)`: sao chép tối đa `n` ký tự.
- `strcat(dest, src)`: nối `src` vào cuối `dest`.
- `strncat(dest, src, n)`: nối tối đa `n` ký tự.
- `strcmp(str1, str2)`: so sánh hai chuỗi.
- `strchr(str, ch)`: tìm lần xuất hiện đầu tiên của ký tự `ch`.
- `strstr(haystack, needle)`: tìm chuỗi con.
- `strtok(str, delimiters)`: tách chuỗi thành các phần nhỏ theo các ký tự phân cách.

### Hằng chuỗi

```c
char *str = "Hello";   // Không được sửa nội dung hằng chuỗi
char arr[] = "Hello";  // Mảng char có thể sửa đổi
```

- Vị trí lưu hằng chuỗi do cách triển khai và chuỗi công cụ quyết định; trong hệ nhúng, hằng chuỗi thường được đặt ở vùng nhớ chỉ đọc như Flash/ROM.
- Khi đọc chuỗi, `fgets()` an toàn hơn `gets()`. `gets()` không kiểm soát kích thước vùng đệm và đã bị loại khỏi chuẩn C từ C11.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-15"></a>
## 15. Con trỏ

<a id="muc-15-01"></a>
### 15.1. Khái niệm cơ bản

- Con trỏ là một biến dùng để lưu địa chỉ.
- Kích thước con trỏ phụ thuộc cách triển khai và kiến trúc hệ thống.

**Ví dụ ép một địa chỉ số thành con trỏ:**

```c
int *ptr = (int *)0x12345678;
```

- Địa chỉ đó có hợp lệ để truy cập hay không phụ thuộc sơ đồ ánh xạ bộ nhớ của hệ thống đích.
- **Con trỏ treo (dangling pointer)** là con trỏ vẫn giữ một địa chỉ nhưng đối tượng/vùng nhớ tại đó đã hết thời gian tồn tại hoặc đã được giải phóng.

<a id="muc-15-02"></a>
### 15.2. Con trỏ treo sau `free()`

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(sizeof(int));

    *ptr = 10;
    printf("Dia chi: %p, Gia tri: %d\n", (void *)ptr, *ptr);

    free(ptr);
    ptr = NULL;

    return 0;
}
```

- Sau `free()`, không được giải tham chiếu địa chỉ cũ.
- Gán `ptr = NULL` giúp giảm nguy cơ vô tình dùng lại con trỏ đã được giải phóng.

<a id="muc-15-03"></a>
### 15.3. Con trỏ treo khi trả về biến cục bộ

```c
int *danglingPointer(void)
{
    int temp = 10;
    return &temp;   // Sai: temp hết thời gian tồn tại khi hàm kết thúc
}
```

- Khi hàm kết thúc, `temp` không còn tồn tại nên con trỏ trả về trở thành con trỏ treo.

<a id="muc-15-04"></a>
### 15.4. Con trỏ treo khi biến ra khỏi phạm vi

```c
#include <stdio.h>

int main(void)
{
    int *ptr;

    {
        int temp = 10;
        ptr = &temp;
        printf("Dia chi: %p\n", (void *)ptr);
    }

    // Không được giải tham chiếu ptr tại đây vì temp đã hết thời gian tồn tại.
    return 0;
}
```

**Cách tránh:**

- Không giữ địa chỉ của đối tượng sau khi đối tượng hết thời gian tồn tại.
- Sau `free(ptr)`, có thể gán `ptr = NULL`.
- Không trả về địa chỉ của biến cục bộ có thời gian lưu trữ tự động.

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

- Kiểu con trỏ cho trình biên dịch biết kiểu đối tượng được truy cập khi giải tham chiếu.
- Kiểu con trỏ quyết định bước nhảy của phép toán số học con trỏ.

**Ví dụ:**

```text
char *ptr  -> ptr + 1 tăng theo sizeof(char)
int  *ptr  -> ptr + 1 tăng theo sizeof(int)
```

#### Con trỏ `void *`

- `void *` là con trỏ tổng quát có thể giữ địa chỉ của nhiều loại đối tượng dữ liệu khác nhau.
- Không thể giải tham chiếu trực tiếp `void *` vì trình biên dịch chưa biết kiểu dữ liệu cần đọc.
- Theo chuẩn C, phép toán số học con trỏ trực tiếp trên `void *` không hợp lệ.

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
        printf("[chi so %d] Dia chi: %p, Gia tri: %d\n",
               i,
               (void *)(arr + i),
               *(arr + i));
    }

    return 0;
}
```

**Kết quả minh họa:**

```text
[chi so 0] Dia chi: ..., Gia tri: 2
[chi so 1] Dia chi: ..., Gia tri: 4
[chi so 2] Dia chi: ..., Gia tri: 6
[chi so 3] Dia chi: ..., Gia tri: 8
[chi so 4] Dia chi: ..., Gia tri: 10
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

    printf("Gia tri so nguyen: %d\n", *((int *)arr[0]));
    printf("Gia tri float: %f\n", *((float *)arr[1]));
    printf("Gia tri char: %c\n", *((char *)arr[2]));

    return 0;
}
```

**Kết quả:**

```text
Gia tri so nguyen: 10
Gia tri float: 20.500000
Gia tri char: Z
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
    printf("Phep cong: %d\n", operation(5, 3));

    operation = multiply;
    printf("Phep nhan: %d\n", operation(5, 3));

    return 0;
}
```

**Kết quả:**

```text
Phep cong: 8
Phep nhan: 15
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

    printf("Cong = %.1f\n", operation[0](a, b));
    printf("Tru = %.1f\n", operation[1](a, b));
    printf("Nhan = %.1f\n", operation[2](a, b));
    printf("Chia = %.1f\n", operation[3](a, b));

    return 0;
}
```

**Kết quả:**

```text
Cong = 15.0
Tru = 5.0
Nhan = 50.0
Chia = 2.0
```

<a id="muc-15-11"></a>
### 15.11. Con trỏ trong hàm

- Khi truyền con trỏ vào hàm, **giá trị địa chỉ** của con trỏ được sao chép vào tham số.
- Thông qua địa chỉ đó, hàm có thể đọc hoặc thay đổi đối tượng gốc.
- Không được trả về địa chỉ của biến cục bộ có thời gian lưu trữ tự động.

**Ví dụ sai:**

```c
int *increment(int a)
{
    int b = a + 1;
    return &b;   // Sai: b hết thời gian tồn tại khi hàm kết thúc
}
```

**Các cách trả về con trỏ hợp lệ trong các ví dụ đang học:**

- Trả về địa chỉ của đối tượng có thời gian lưu trữ `static`.
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
### 15.12. Con trỏ hàm làm hàm gọi lại

- Một hàm có thể nhận con trỏ hàm làm đối số và gọi hàm được truyền vào. Đây là một cách triển khai **hàm gọi lại (callback)**.

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
    printf("Tong binh phuong = %d\n", sum);

    sum = conditionalSum(2, 3, cube);
    printf("Tong lap phuong = %d\n", sum);

    return 0;
}
```

**Kết quả:**

```text
Tong binh phuong = 13
Tong lap phuong = 35
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
### 15.14. Truy cập thành viên `struct` bằng con trỏ

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
### 15.15. Con trỏ `struct` trong tham số hàm

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

    printf("Toc do: %u - Goc: %u\n",
           (unsigned)motor.speed,
           (unsigned)motor.angle);

    return 0;
}
```

**Kết quả:**

```text
Toc do: 50 - Goc: 90
```

<a id="muc-15-16"></a>
### 15.16. Con trỏ hàm là thành viên của `struct`

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

    printf("Dien tich: %.2f\n", rect.area(&rect));
    return 0;
}
```

**Kết quả:**

```text
Dien tich: 15.00
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-16"></a>
## 16. Các chủ đề C bổ sung

<a id="muc-16-01"></a>
### 16.1. `typedef`

- `typedef` tạo một tên mới (bí danh) cho một kiểu dữ liệu đã có.

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

- Quản lý trạng thái trong máy trạng thái (State Machine).
- Quản lý menu lựa chọn.
- Biểu diễn một tập hợp giá trị cố định có tên rõ ràng.

<a id="muc-16-04"></a>
### 16.4. Tràn số

- Tràn số xảy ra khi kết quả của phép tính nằm ngoài phạm vi biểu diễn của kiểu dữ liệu.

**Ví dụ:**

- Với kiểu chỉ biểu diễn được từ `-128` đến `127`, giá trị `128` nằm ngoài phạm vi biểu diễn.
- Trong mã nguồn thực tế không nên dựa vào việc số nguyên có dấu tự động quay vòng; cách xử lý khi vượt phạm vi phụ thuộc kiểu dữ liệu và quy tắc của C.

<a id="muc-16-05"></a>
### 16.5. Liên kết nội bộ và liên kết ngoài

#### Liên kết nội bộ

- Tên chỉ được dùng để tham chiếu tới cùng đối tượng hoặc hàm trong đơn vị dịch hiện tại.

**Ví dụ:**

```c
// secret.c
static int secret_key = 999;

static void do_internal_job(void)
{
    // ...
}
```

#### Liên kết ngoài

- Cùng một tên có thể tham chiếu tới cùng đối tượng hoặc hàm từ các đơn vị dịch khác nhau.
- Tệp muốn sử dụng cần có khai báo phù hợp, thường được đặt trong tệp tiêu đề.

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
### 16.6. Bảo vệ tệp tiêu đề

- Cơ chế bảo vệ tệp tiêu đề giúp tránh nội dung của cùng một tệp tiêu đề bị xử lý lặp lại nhiều lần trong một đơn vị dịch.

**Mẫu:**

```c
#ifndef MY_HEADER_H
#define MY_HEADER_H

// Nội dung tệp tiêu đề

#endif
```

<a id="muc-16-07"></a>
### 16.7. Hàm `static inline`

- `inline` cho trình biên dịch biết một hàm là ứng viên phù hợp để chèn trực tiếp mã hàm tại nơi gọi, nhưng trình biên dịch vẫn có quyền quyết định có thực hiện hay không.
- `static inline` thường được dùng cho các hàm nhỏ trong tệp tiêu đề.

**Ví dụ:**

```c
static inline int square(int x)
{
    return x * x;
}
```

**Ưu điểm có thể có:**

- Giảm chi phí của lời gọi hàm nhỏ.

**Nhược điểm có thể có:**

- Chèn trực tiếp quá nhiều có thể làm tăng kích thước mã.

**Dùng khi nào?**

- Hàm nhỏ, được gọi thường xuyên, đặc biệt là hàm hỗ trợ trong mã nhúng.

<a id="muc-16-08"></a>
### 16.8. Các hàm thao tác bộ nhớ

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

---

<a id="muc-16-09"></a>
### 16.9. Các lỗi phổ biến trong C và cách gỡ lỗi

Có thể chia lỗi trong C thành 5 nhóm chính:

```text
Compile-time
→ lỗi/cảnh báo khi biên dịch

Link-time
→ lỗi khi liên kết các tệp đối tượng và thư viện

Run-time / Memory
→ chương trình đã biên dịch nhưng gặp lỗi khi chạy hoặc quản lý bộ nhớ sai

Logic
→ chương trình chạy nhưng cho kết quả sai

Embedded-specific
→ lỗi liên quan tới phần cứng, ISR, thanh ghi và tính nguyên tử
```

> **Lưu ý:** Nhiều lỗi bộ nhớ trong C dẫn tới **undefined behavior** — hành vi không xác định. Chương trình có thể bị dừng, cho kết quả sai hoặc thậm chí có vẻ chạy bình thường. Vì vậy không nên dựa vào việc “chạy không lỗi” để kết luận mã nguồn là đúng.

#### 1. Compile-time — lỗi khi biên dịch

| Lỗi | Xảy ra khi nào? | Tại sao? | Cách gỡ lỗi |
|---|---|---|---|
| Sai cú pháp | Thiếu `;`, ngoặc `{}` không cân xứng, viết sai từ khóa... | Mã nguồn không tuân theo cú pháp của C. | Đọc vị trí lỗi compiler báo; kiểm tra cả dòng ngay trước đó vì lỗi cú pháp có thể được phát hiện muộn hơn vị trí gây lỗi. |
| Chưa khai báo | Dùng biến, hàm hoặc kiểu dữ liệu trước khi có khai báo phù hợp. | Compiler chưa biết tên hoặc kiểu của thành phần đang được sử dụng. | Kiểm tra khai báo, prototype và tệp tiêu đề cần `#include`. |
| Sai hoặc không tương thích kiểu | Truyền sai kiểu tham số, gán giữa các kiểu không phù hợp, chuyển đổi có nguy cơ mất dữ liệu. | Kiểu dữ liệu của biểu thức không phù hợp với ngữ cảnh sử dụng. | Đọc cảnh báo compiler, kiểm tra prototype và kiểu của biến; chỉ ép kiểu khi thực sự hiểu chuyển đổi đang thực hiện. |

**Ví dụ prototype hợp lệ trước phần định nghĩa:**

```c
int add(int a, int b);

int main(void)
{
    return add(1, 2);
}

int add(int a, int b)
{
    return a + b;
}
```

Không bắt buộc hàm phải được định nghĩa trước `main()`, nhưng phải có **khai báo phù hợp** trước khi gọi.

**Khi gỡ lỗi nên bật cảnh báo compiler**, ví dụ với GCC/Clang:

```bash
-Wall -Wextra -Wpedantic
```

Có thể dùng thêm các cảnh báo như `-Wconversion` hoặc `-Wshadow` khi dự án phù hợp.

#### 2. Link-time — lỗi khi liên kết

| Lỗi | Xảy ra khi nào? | Tại sao? | Cách gỡ lỗi |
|---|---|---|---|
| `undefined reference` | Compiler đã thấy khai báo nhưng linker không tìm thấy phần định nghĩa cần thiết. | Quên biên dịch/liên kết tệp `.c`, thiếu thư viện, sai tên hàm hoặc chỉ khai báo mà không định nghĩa. | Kiểm tra tệp nguồn có được đưa vào quá trình build không; kiểm tra thư viện, prototype và tên symbol. |
| `multiple definition` | Linker tìm thấy nhiều phần định nghĩa không hợp lệ của cùng một symbol. | Định nghĩa biến/hàm toàn cục ở nhiều tệp, hoặc `#include` trực tiếp tệp `.c`. | Chỉ `#include` tệp `.h`; đặt khai báo `extern` trong header và chỉ có một phần định nghĩa thật trong một tệp `.c`. |

Ví dụ đúng với biến toàn cục:

```c
// counter.h
extern int counter;
```

```c
// counter.c
int counter = 0;
```

```c
// main.c
#include "counter.h"
```

#### 3. Run-time / Memory — lỗi khi chạy và quản lý bộ nhớ

| Lỗi | Xảy ra khi nào? | Tại sao? | Cách gỡ lỗi |
|---|---|---|---|
| Biến chưa khởi tạo | Đọc biến cục bộ trước khi gán giá trị. | Giá trị ban đầu của biến tự động không được xác định. | Khởi tạo biến khi khai báo; bật cảnh báo compiler. |
| Wild pointer — con trỏ hoang | Con trỏ chưa được khởi tạo nhưng đã bị giải tham chiếu. | Con trỏ đang chứa địa chỉ không xác định. | Khởi tạo con trỏ bằng địa chỉ hợp lệ hoặc `NULL`; kiểm tra trước khi dùng. |
| Null pointer dereference | Giải tham chiếu con trỏ `NULL`. | `NULL` không trỏ tới một đối tượng hợp lệ để truy cập. | Kiểm tra `ptr != NULL` trước khi giải tham chiếu khi con trỏ có thể rỗng. |
| Dangling pointer — con trỏ treo | Con trỏ vẫn giữ địa chỉ của vùng nhớ đã `free()` hoặc đối tượng đã hết vòng đời. | Đối tượng đích không còn tồn tại nhưng con trỏ vẫn giữ địa chỉ cũ. | Không sử dụng sau `free()`; có thể gán `ptr = NULL`; không trả về địa chỉ biến cục bộ tự động. |
| Use-after-free | Truy cập vùng nhớ sau khi đã `free()`. | Vùng nhớ không còn thuộc quyền sử dụng của chương trình tại con trỏ đó. | Theo dõi rõ quyền sở hữu vùng nhớ; sau `free()` không đọc/ghi qua con trỏ cũ. |
| Memory leak — rò rỉ bộ nhớ | Cấp phát bằng `malloc()`/`calloc()`/`realloc()` nhưng mất con trỏ hoặc không `free()` khi không còn dùng. | Vùng nhớ vẫn được giữ trên Heap nhưng chương trình không còn cách hợp lệ để giải phóng. | Ghép rõ mỗi lần cấp phát với nơi giải phóng; kiểm tra các nhánh `return`; dùng công cụ kiểm tra bộ nhớ khi môi trường hỗ trợ. |
| Out-of-bounds / Buffer overflow | Truy cập vượt giới hạn mảng hoặc bộ đệm. | C không tự kiểm tra biên khi dùng `[]` hoặc con trỏ. | Kiểm tra chỉ số và kích thước trước khi đọc/ghi; truyền kích thước bộ đệm cùng với con trỏ. |
| Double free | Gọi `free()` hai lần với cùng một vùng nhớ. | Lần `free()` thứ hai tác động lên vùng không còn được cấp phát cho con trỏ đó. | Quy định rõ một chủ thể chịu trách nhiệm giải phóng; sau `free()` có thể đặt con trỏ về `NULL`. |
| Invalid free | Gọi `free()` với địa chỉ không được trả về bởi hàm cấp phát động hợp lệ. | `free()` chỉ được dùng với vùng nhớ động phù hợp hoặc `NULL`. | Không `free()` biến cục bộ, biến toàn cục hoặc con trỏ đã bị dịch khỏi địa chỉ gốc. |
| Stack overflow | Stack không còn đủ không gian. | Đệ quy quá sâu, đệ quy vô hạn hoặc khai báo biến/mảng cục bộ quá lớn. | Kiểm tra độ sâu lời gọi; giảm dữ liệu cục bộ lớn; theo dõi mức sử dụng Stack nếu RTOS/toolchain hỗ trợ. |
| Stack buffer overflow / stack smashing | Ghi vượt bộ đệm cục bộ trên Stack. | Ghi đè sang dữ liệu khác trong stack frame. | Kiểm tra kích thước bộ đệm; dùng API có giới hạn kích thước; stack protector có thể giúp phát hiện nhưng không thay thế việc sửa lỗi. |
| Heap corruption | Cấu trúc dữ liệu Heap bị phá hỏng. | Ghi vượt vùng cấp phát, double free hoặc thao tác sai con trỏ Heap. | Kiểm tra vị trí cấp phát/giải phóng, kích thước vùng và mọi phép toán con trỏ liên quan. |
| Truy cập vùng nhớ không hợp lệ | Đọc/ghi địa chỉ không được phép hoặc không tồn tại. | Con trỏ sai, vượt biên, địa chỉ phần cứng sai... | Trên hệ điều hành có thể gặp Segmentation Fault; trên MCU có thể gặp HardFault/BusFault/MemManage Fault tùy kiến trúc. Dùng debugger để xem địa chỉ gây lỗi. |
| Chia số nguyên cho `0` | Mẫu số của phép chia số nguyên bằng `0`. | Trong C, chia số nguyên cho `0` là **undefined behavior**. | Kiểm tra mẫu số trước phép chia; đặt breakpoint hoặc assertion tại nơi tính mẫu số. |

**Ví dụ con trỏ treo và use-after-free:**

```c
int *ptr = malloc(sizeof(int));

if (ptr != NULL) {
    *ptr = 10;
    free(ptr);

    // *ptr = 20;   // Sai: use-after-free
    ptr = NULL;
}
```

**Ví dụ vượt phạm vi:**

```c
int arr[5];

arr[0] = 10;   // Hợp lệ
arr[4] = 50;   // Hợp lệ

// arr[5] = 60;   // Sai: vượt phạm vi
```

**Công cụ gỡ lỗi hữu ích khi môi trường hỗ trợ:**

- GDB hoặc debugger của IDE.
- JTAG/SWD trên vi điều khiển.
- AddressSanitizer (ASan) để phát hiện nhiều lỗi bộ nhớ khi chạy trên môi trường/toolchain hỗ trợ.
- UndefinedBehaviorSanitizer (UBSan) để phát hiện nhiều dạng hành vi không xác định.
- Công cụ phân tích tĩnh như `cppcheck` hoặc `clang-tidy`.

#### 4. Logic — chương trình chạy nhưng kết quả sai

| Lỗi | Xảy ra khi nào? | Tại sao? | Cách gỡ lỗi |
|---|---|---|---|
| Off-by-one | Vòng lặp chạy thừa hoặc thiếu đúng một phần tử. | Dùng `<=` thay cho `<`, hoặc xác định sai chỉ số đầu/cuối. | Kiểm tra rõ miền chỉ số, đặc biệt với mảng có `N` phần tử thì chỉ số hợp lệ thường là `0` đến `N - 1`. |
| Nhầm `=` và `==` | Viết phép gán trong điều kiện thay vì phép so sánh. | `=` trả về một giá trị nên biểu thức vẫn có thể hợp lệ về cú pháp. | Bật cảnh báo compiler; kiểm tra các biểu thức điều kiện. |
| Tràn số | Kết quả vượt phạm vi kiểu dữ liệu. | Kiểu dữ liệu không đủ biểu diễn kết quả. | Chọn kiểu phù hợp; kiểm tra trước khi tính; nhớ rằng unsigned integer có quy tắc modulo còn signed integer overflow là undefined behavior. |
| Chuyển đổi kiểu sai | Ép kiểu hoặc chuyển kiểu làm mất dữ liệu/đổi ý nghĩa giá trị. | Khác kích thước, signed/unsigned hoặc kiểu số thực/số nguyên. | Đọc cảnh báo chuyển đổi; kiểm tra phạm vi trước khi ép kiểu. |
| Thao tác bit sai | Dùng sai mặt nạ hoặc nhầm `&`, `|` với `&&`, `||`. | Toán tử bit và toán tử logic có mục đích khác nhau. | In giá trị ở dạng hex/binary; kiểm tra mask từng bước. |
| Xử lý chuỗi sai | Quên `'\0'`, vùng đích quá nhỏ, nối/sao chép quá nhiều dữ liệu. | Hàm xử lý chuỗi C dựa vào ký tự kết thúc và thường không tự biết kích thước vùng đích. | Theo dõi độ dài và sức chứa; dùng kích thước bộ đệm rõ ràng; kiểm tra `strlen()` khi phù hợp. |
| Sai định dạng `printf()` | Format specifier không khớp kiểu của đối số. | `printf()` là hàm có số lượng đối số thay đổi; nó dựa vào chuỗi định dạng để diễn giải dữ liệu. | Bật cảnh báo compiler và đối chiếu từng `%...` với kiểu thực tế. |

**Ví dụ off-by-one:**

```c
int arr[10];

for (int i = 0; i < 10; i++) {
    arr[i] = 0;
}
```

Nếu viết:

```c
for (int i = 0; i <= 10; i++)
```

thì lần cuối sẽ truy cập `arr[10]`, vượt phạm vi của mảng.

#### 5. Lỗi thường gặp riêng trong Embedded

| Lỗi | Xảy ra khi nào? | Tại sao? | Cách gỡ lỗi |
|---|---|---|---|
| Thiếu `volatile` khi cần | Main/ISR hoặc phần cứng thay đổi một giá trị nhưng compiler không thấy luồng thay đổi thông thường. | Compiler có thể tối ưu việc đọc/ghi theo giả định của chương trình bình thường. | Dùng `volatile` đúng nơi như thanh ghi phần cứng hoặc biến chia sẻ với ISR khi phù hợp. |
| Hiểu sai `volatile` | Dùng `volatile` và cho rằng truy cập đã nguyên tử hoặc an toàn đồng thời. | `volatile` chỉ ảnh hưởng cách compiler giữ các lần truy cập cần thiết; không cung cấp khóa hay tính nguyên tử. | Kiểm tra kích thước truy cập, kiến trúc CPU và dùng critical section/atomic/cơ chế đồng bộ phù hợp. |
| Dữ liệu dùng chung với ISR bị cập nhật không nhất quán | Main và ISR cùng đọc/ghi dữ liệu nhiều byte hoặc chuỗi thao tác không nguyên tử. | ISR có thể xen vào giữa quá trình cập nhật. | Bảo vệ vùng tới hạn khi cần; thiết kế giao tiếp main/ISR đơn giản; kiểm tra tài liệu MCU/RTOS. |
| Truy cập sai thanh ghi | Sai địa chỉ, sai độ rộng, sai mặt nạ bit hoặc dùng thanh ghi không đúng chế độ. | Peripheral được điều khiển trực tiếp qua memory-mapped register. | Đối chiếu datasheet/reference manual; xem thanh ghi bằng debugger; kiểm tra địa chỉ và mask. |
| Stack quá nhỏ | Task RTOS hoặc chương trình bare-metal dùng nhiều biến cục bộ/lời gọi sâu hơn dự kiến. | Stack có kích thước hữu hạn và thường nhỏ trong MCU. | Theo dõi high-water mark nếu RTOS hỗ trợ; kiểm tra linker map; giảm biến cục bộ lớn. |

#### Quy trình gỡ lỗi ngắn gọn

Khi gặp lỗi, có thể đi theo thứ tự:

```text
1. Đọc toàn bộ lỗi và cảnh báo của compiler/linker.
        ↓
2. Xác định lỗi xảy ra ở Compile-time, Link-time hay Run-time.
        ↓
3. Tái hiện lỗi bằng trường hợp nhỏ và ổn định nhất có thể.
        ↓
4. Dùng breakpoint và xem giá trị biến/con trỏ.
        ↓
5. Với lỗi bộ nhớ:
   kiểm tra địa chỉ + kích thước + vòng đời + quyền sở hữu.
        ↓
6. Với Embedded:
   xem fault status, thanh ghi CPU/peripheral và Stack.
        ↓
7. Sửa nguyên nhân gốc rồi bật lại cảnh báo để kiểm tra.
```

**Ý cần nhớ khi phỏng vấn:**

- Compile-time: compiler phát hiện vấn đề trong mã nguồn.
- Link-time: linker không ghép được các symbol thành chương trình hoàn chỉnh.
- Run-time/Memory: mã đã build nhưng truy cập dữ liệu/bộ nhớ sai khi chạy.
- Logic: chương trình chạy nhưng thuật toán hoặc kết quả sai.
- Embedded: ngoài mã C còn phải kiểm tra thanh ghi, ISR, Stack và hành vi phần cứng.
- Các lỗi như vượt biên, use-after-free, signed integer overflow hoặc chia số nguyên cho `0` có thể dẫn tới **undefined behavior**, không nhất thiết lúc nào cũng làm chương trình dừng ngay.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-17"></a>
## 17. C trong phần mềm nhúng

<a id="muc-17-01"></a>
### 17.1. Cấp phát động

Các hàm cấp phát động thường gặp:

- `malloc()`
- `calloc()`
- `realloc()`
- `free()`

Trong hệ thống nhúng:

- RAM thường hạn chế.
- Cấp phát/giải phóng nhiều lần có thể gây phân mảnh Heap.
- Thời gian cấp phát có thể khó dự đoán hơn bộ nhớ tĩnh hoặc vùng nhớ có kích thước cố định.
- Hệ thống chạy lâu cần đặc biệt cẩn thận với rò rỉ bộ nhớ.
- Nếu `realloc()` làm thay đổi địa chỉ vùng nhớ, các con trỏ cũ trỏ vào vùng đã được thay thế không còn được dùng nữa.

**Ưu tiên khi phù hợp:**

- Biến cục bộ có kích thước cố định.
- Biến `static`.
- Mảng có kích thước xác định trước.
- Bộ đệm được cấp phát cố định từ đầu.

Không có nghĩa là cấp phát động luôn bị cấm; cần tuân theo yêu cầu tài nguyên và quy ước của dự án.

<a id="muc-17-02"></a>
### 17.2. `volatile`, thanh ghi và ISR

Trong Embedded C, `volatile` thường gặp khi một giá trị có thể thay đổi ngoài luồng thực thi thông thường của đoạn mã hiện tại, ví dụ:

- Thanh ghi ngoại vi.
- Biến được thay đổi trong ISR.
- Dữ liệu có thể được phần cứng cập nhật.

**Ví dụ truy cập thanh ghi ánh xạ bộ nhớ:**

```c
#include <stdint.h>

volatile uint32_t *reg =
    (volatile uint32_t *)0x40000000u;

*reg |= (1U << 5);
```

`volatile` giúp trình biên dịch giữ lại các lần đọc/ghi cần thiết đối với đối tượng đó.

**Lưu ý quan trọng:**

- `volatile` không làm thao tác trở thành nguyên tử.
- `volatile` không tự giải quyết vấn đề tranh chấp dữ liệu giữa nhiều luồng hoặc giữa mã chính và ISR.
- Khi chia sẻ dữ liệu với ISR, ngoài `volatile` còn phải xét kích thước dữ liệu, tính nguyên tử của thao tác và cơ chế đồng bộ phù hợp với nền tảng.

<a id="muc-17-03"></a>
### 17.3. Tính xác định

Phần mềm nhúng thường quan tâm tới:

- Thời gian thực thi có thể dự đoán.
- Sử dụng RAM/Flash có thể kiểm soát.
- Không rò rỉ tài nguyên.
- Không tạo cấp phát bất ngờ trong luồng xử lý quan trọng.
- Tránh hành vi không xác định do con trỏ sai, vượt phạm vi mảng hoặc truy cập vùng nhớ không hợp lệ.

Khi dùng C, cần đặc biệt chú ý:

- Kích thước và vòng đời của bộ đệm.
- Con trỏ và con trỏ treo.
- `malloc()`, `calloc()`, `realloc()` và `free()`.
- Tràn số và chuyển đổi kiểu.
- Truy cập thanh ghi bằng `volatile`.
- Thao tác bit.
- Kích thước `struct`, phần đệm và căn chỉnh dữ liệu.
- Các hàm xử lý chuỗi và bộ nhớ như `strcpy()`, `memcpy()`, `memmove()` phải dùng với kích thước vùng đích phù hợp.

<a id="muc-17-04"></a>
### 17.4. Những gì cần nhớ khi phỏng vấn

#### Nhóm bắt buộc

- Quá trình tiền xử lý, biên dịch, hợp dịch và liên kết.
- Kiểu dữ liệu cơ bản và `sizeof`.
- Phạm vi biến và thời gian tồn tại.
- `static`, `extern`, `const`, `volatile`.
- Truyền tham số trong C là truyền theo giá trị.
- Con trỏ và phép toán con trỏ.
- Mảng và mối liên hệ giữa mảng với con trỏ.
- Chuỗi ký tự và ký tự `\0`.
- `struct`, `union`, `enum`.
- Căn chỉnh và phần đệm của `struct`.
- Hàm con trỏ và callback.
- `malloc()`, `calloc()`, `realloc()`, `free()`.
- Rò rỉ bộ nhớ và con trỏ treo.
- Thao tác bit: bật, xóa, đảo và kiểm tra bit.
- Big Endian và Little Endian.
- Liên kết nội bộ và liên kết ngoài.
- Bảo vệ tệp tiêu đề.
- `memcpy()`, `memmove()`, `memset()`, `memcmp()`.

#### Nhóm nên biết

- Thư viện tĩnh và thư viện động.
- Hàm có số lượng đối số thay đổi.
- Trường bit.
- `static inline`.
- Tràn số.
- Con trỏ `void *`.
- Các trường hợp con trỏ treo thường gặp.
- Cách dùng `realloc()` an toàn với con trỏ tạm.
- Sự khác nhau giữa `memcpy()` và `memmove()`.

#### Chưa cần học sâu ở mức Intern

- Chi tiết ABI.
- Tối ưu hóa trình biên dịch ở mức chuyên sâu.
- Bộ cấp phát bộ nhớ tùy chỉnh.
- Kỹ thuật macro phức tạp.
- Chi tiết nội bộ của trình liên kết và định dạng tệp đối tượng.
- Tối ưu hóa Assembly chuyên sâu.
- Các kỹ thuật quản lý bộ nhớ thời gian thực nâng cao nếu vị trí không yêu cầu.

### Câu hỏi phỏng vấn tự kiểm tra

1. Quá trình từ tệp `.c` đến tệp thực thi gồm những giai đoạn nào?
2. `static` với biến cục bộ khác gì `static` ở phạm vi tệp?
3. `extern` dùng để làm gì?
4. Tại sao trong C mọi đối số đều được xem là truyền theo giá trị?
5. Mảng khác con trỏ như thế nào?
6. Tại sao `ptr[i]` tương đương về ý nghĩa với `*(ptr + i)`?
7. Chuỗi C kết thúc bằng ký tự gì?
8. `struct` và `union` khác nhau như thế nào?
9. Tại sao `sizeof(struct)` có thể lớn hơn tổng kích thước các thành viên?
10. Con trỏ hàm dùng để làm gì?
11. Callback là gì?
12. `malloc()` khác `calloc()` ở điểm nào?
13. Khi `malloc()` hoặc `calloc()` thất bại, chúng trả về gì?
14. `realloc()` có thể làm thay đổi địa chỉ vùng nhớ không?
15. Nếu `realloc()` thất bại, vùng nhớ cũ có còn tồn tại không?
16. Tại sao nên dùng con trỏ tạm khi gọi `realloc()`?
17. Rò rỉ bộ nhớ là gì?
18. Con trỏ treo là gì?
19. `volatile` thường được dùng trong những trường hợp nào trong Embedded?
20. `volatile` có đảm bảo thao tác nguyên tử hay an toàn đa luồng không?
21. Công thức bật, xóa, đảo và kiểm tra một bit là gì?
22. Big Endian và Little Endian khác nhau như thế nào?
23. `memcpy()` và `memmove()` khác nhau ở điểm nào?
24. Tại sao cấp phát động cần được cân nhắc trong hệ thống nhúng?
25. Khi viết Embedded C, tại sao cần quan tâm tới tính xác định của thời gian và bộ nhớ?

#### Memory Management

1. Tại sao cần chia bộ nhớ thành nhiều vùng?
2. Biến toàn cục không khởi tạo nằm ở đâu?
3. Hai biến global có cùng giá trị khởi tạo `0` và `10` — tại sao chúng không nằm trong cùng một vùng nhớ?
4. Khi chương trình gọi một hàm lồng nhau nhiều lần (đệ quy), vùng nhớ nào bị ảnh hưởng nhiều nhất?
5. Tại sao biến `const` thường được đặt trong vùng `.rodata` thay vì `.data`?
6. Nếu bạn muốn dữ liệu tồn tại suốt vòng đời chương trình, bạn nên đặt nó ở vùng nhớ nào?
7. Tại sao vùng `.bss` không chiếm nhiều dung lượng trong file `.bin`, nhưng lại chiếm RAM khi chạy?
8. Điều gì xảy ra với Stack khi hàm kết thúc, nhưng biến `static` trong hàm đó vẫn được giữ giá trị?
9. Lỗi Memory Leak xảy ra khi nào? Tại sao? Cách debug.
10. Lỗi Stack Overflow xảy ra khi nào? Tại sao? Cách debug.
11. Lỗi Segmentation Fault xảy ra khi nào? Tại sao? Cách debug.
12. Lỗi Stack Smashing là gì? Cách compiler phát hiện bằng cơ chế Canary.
13. Lỗi Heap Corruption là gì? Cách phát hiện bằng AddressSanitizer.
14. Lỗi Dangling Pointer là gì? Tại sao nguy hiểm? Cách khắc phục.
15. Khi nào nên dùng AddressSanitizer thay vì Valgrind để debug lỗi bộ nhớ?
16. Lỗi Wild Pointer là gì?

[↑ Về mục lục](#muc-luc)

