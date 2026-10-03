# Ghi chú phỏng vấn C++

> **Mục tiêu:** Ôn phần C++ cần thiết cho phỏng vấn.  
> **Tiền đề:** Đã nắm phần C cơ bản và C cho hệ thống nhúng. Tài liệu này tập trung vào những phần C++ khác C hoặc thường được hỏi khi dùng C++ trong phần mềm nhúng.  

<a id="muc-luc"></a>
## Mục lục

1. [Tổng quan C++ trong hệ thống nhúng](#chuong-01)
   - [1.1. C++ khác C ở điểm nào?](#muc-01-01)
   - [1.2. `bool`](#muc-01-02)
   - [1.3. `nullptr`](#muc-01-03)
   - [1.4. `namespace`](#muc-01-04)
2. [Tham chiếu](#chuong-02)
   - [2.1. Khái niệm tham chiếu](#muc-02-01)
   - [2.2. Con trỏ và tham chiếu](#muc-02-02)
   - [2.3. Tham chiếu `const`](#muc-02-03)
3. [Hàm trong C++](#chuong-03)
   - [3.1. Nạp chồng hàm](#muc-03-01)
   - [3.2. Đối số mặc định](#muc-03-02)
   - [3.3. Truyền tham số](#muc-03-03)
   - [3.4. Hàm `inline`](#muc-03-04)
4. [Lớp và đối tượng](#chuong-04)
   - [4.1. `class` và đối tượng](#muc-04-01)
   - [4.2. `class` và `struct`](#muc-04-02)
   - [4.3. `public`, `private`, `protected`](#muc-04-03)
   - [4.4. Hàm thành viên](#muc-04-04)
   - [4.5. Con trỏ `this`](#muc-04-05)
   - [4.6. Thành viên `static`](#muc-04-06)
5. [Hàm dựng, Hàm hủy và vòng đời đối tượng](#chuong-05)
   - [5.1. Hàm dựng (constructor)](#muc-05-01)
   - [5.2. Danh sách khởi tạo](#muc-05-02)
   - [5.3. Hàm hủy (destructor)](#muc-05-03)
   - [5.4. Vòng đời đối tượng](#muc-05-04)
6. [`const` trong C++](#chuong-06)
   - [6.1. Đối tượng `const`](#muc-06-01)
   - [6.2. Hàm thành viên `const`](#muc-06-02)
   - [6.3. `const` với tham số](#muc-06-03)
7. [RAII](#chuong-07)
   - [7.1. Khái niệm RAII](#muc-07-01)
   - [7.2. Ví dụ RAII](#muc-07-02)
8. [Sao chép đối tượng](#chuong-08)
   - [8.1. Hàm dựng sao chép](#muc-08-01)
   - [8.2. Phép gán sao chép](#muc-08-02)
   - [8.3. Sao chép nông và sao chép sâu](#muc-08-03)
9. [Ngữ nghĩa di chuyển](#chuong-09)
   - [9.1. Ý tưởng của di chuyển tài nguyên](#muc-09-01)
   - [9.2. `std::move`](#muc-09-02)
   - [9.3. Hàm dựng di chuyển và phép gán di chuyển](#muc-09-03)
10. [Kế thừa và đa hình](#chuong-10)
    - [10.1. Kế thừa](#muc-10-01)
    - [10.2. Hàm `virtual`](#muc-10-02)
    - [10.3. `override`](#muc-10-03)
    - [10.4. Hàm thuần ảo và lớp trừu tượng](#muc-10-04)
    - [10.5. Hàm hủy ảo](#muc-10-05)
11. [Khuôn mẫu](#chuong-11)
    - [11.1. Khuôn mẫu hàm](#muc-11-01)
    - [11.2. Khuôn mẫu lớp](#muc-11-02)
12. [STL cơ bản cho hệ thống nhúng](#chuong-12)
    - [12.1. `std::array`](#muc-12-01)
    - [12.2. `std::vector`](#muc-12-02)
    - [12.3. `std::string`](#muc-12-03)
    - [12.4. `std::pair`](#muc-12-04)
    - [12.5. Bộ lặp và vòng lặp phạm vi](#muc-12-05)
    - [12.6. `std::algorithm`](#muc-12-06)
13. [Quản lý bộ nhớ trong C++](#chuong-13)
    - [13.1. `new` và `delete`](#muc-13-01)
    - [13.2. `new[]` và `delete[]`](#muc-13-02)
    - [13.3. `std::unique_ptr`](#muc-13-03)
    - [13.4. `std::shared_ptr`](#muc-13-04)
14. [Ép kiểu trong C++](#chuong-14)
    - [14.1. `static_cast`](#muc-14-01)
    - [14.2. `reinterpret_cast`](#muc-14-02)
    - [14.3. `const_cast`](#muc-14-03)
    - [14.4. `dynamic_cast`](#muc-14-04)
15. [`enum class` và `constexpr`](#chuong-15)
    - [15.1. `enum class`](#muc-15-01)
    - [15.2. `constexpr`](#muc-15-02)
    - [15.3. `static constexpr`](#muc-15-03)
16. [Tương tác giữa C và C++](#chuong-16)
    - [16.1. Mã hóa tên ký hiệu](#muc-16-01)
    - [16.2. `extern "C"`](#muc-16-02)
17. [Ngoại lệ và RTTI](#chuong-17)
    - [17.1. Ngoại lệ](#muc-17-01)
    - [17.2. RTTI](#muc-17-02)
18. [C++ trong phần mềm nhúng](#chuong-18)
    - [18.1. Cấp phát động](#muc-18-01)
    - [18.2. Chi phí của `virtual`](#muc-18-02)
    - [18.3. Tính xác định](#muc-18-03)
    - [18.4. Những gì cần nhớ khi phỏng vấn](#muc-18-04)

---

<a id="chuong-01"></a>
## 1. Tổng quan C++ trong hệ thống nhúng

<a id="muc-01-01"></a>
### 1.1. C++ khác C ở điểm nào?

- C++ phát triển từ C nhưng bổ sung nhiều cơ chế giúp tổ chức chương trình lớn tốt hơn.
- Trong phần mềm nhúng, C++ thường được dùng để:
  - Đóng gói trình điều khiển và trạng thái phần cứng trong `class`.
  - Quản lý tài nguyên bằng hàm dựng/hàm hủy và RAII.
  - Tạo giao diện chung bằng kế thừa và hàm `virtual`.
  - Viết mã tổng quát bằng khuôn mẫu.
  - Tận dụng kiểm tra kiểu mạnh hơn và các tiện ích của thư viện chuẩn.
  
<a id="muc-01-02"></a>
### 1.2. `bool`

- `bool` là kiểu logic có hai giá trị:
  - `true`
  - `false`

**Ví dụ:**

```cpp
bool is_ready = true;

if (is_ready) {
    // Hệ thống đã sẵn sàng
}
```

<a id="muc-01-03"></a>
### 1.3. `nullptr`

- `nullptr` biểu diễn con trỏ rỗng trong C++ hiện đại.
- Nên ưu tiên `nullptr` thay cho `NULL` hoặc số `0` khi làm việc với con trỏ.
- `nullptr` có kiểu riêng nên giúp trình biên dịch phát hiện một số lỗi tốt hơn.

**Ví dụ:**

```cpp
int *ptr = nullptr;

if (ptr == nullptr) {
    // ptr chưa trỏ tới đối tượng nào
}
```

<a id="muc-01-04"></a>
### 1.4. `namespace`

- `namespace` là một phạm vi đặt tên dùng để nhóm các thành phần liên quan và tránh xung đột tên. `std` là namespace của thư viện chuẩn C++, chứa nhiều thành phần như `cout`, `cin`, `vector`, `string`, ...

**Ví dụ:**

```cpp
#include <iostream>

// Nhóm các hàm và biến liên quan đến cảm biến
namespace Sensor {
    int readValue = 0;

    void init() {
        std::cout << " Cảm biến đã được khởi tạo.\n";
    }

    int readData() {
        readValue = 42; // Mô phỏng việc đọc dữ liệu từ phần cứng
        return readValue;
    }
}

// Nhóm các hàm và biến liên quan đến động cơ
namespace Motor {
    int currentSpeed = 0;

    void init() {
        std::cout << " Động cơ đã được khởi tạo.\n";
    }

    void spin(int speed) {
        currentSpeed = speed;
        std::cout << " Động cơ đang quay với tốc độ: " << speed << " vòng/phút.\n";
    }

    void stop() {
        currentSpeed = 0;
        std::cout << " Động cơ đã dừng.\n";
    }
}

int main() {
    // Nhờ namespace, có thể gọi hai hàm init() cùng tên mà không bị xung đột
    Sensor::init();
    Motor::init();

    // Sử dụng các chức năng thuộc từng namespace
    int data = Sensor::readData();
    if (data > 40) {
        Motor::spin(1000);
    } else {
        Motor::stop();
    }

    return 0;
}
```

- Toán tử `::` được gọi là **toán tử phạm vi**.
- Có thể dùng `using`, nhưng trong tệp tiêu đề và dự án lớn nên hạn chế `using namespace ...` vì dễ gây xung đột tên nếu bạn tự viết một hàm trùng tên với thư viện chuẩn.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-02"></a>
## 2. Tham chiếu

<a id="muc-02-01"></a>
### 2.1. Khái niệm tham chiếu

- Tham chiếu là một tên khác dùng để tham chiếu tới một đối tượng đã tồn tại.
- Tham chiếu phải được khởi tạo khi khai báo.
- Sau khi đã tham chiếu tới một đối tượng, tham chiếu không thể chuyển sang tham chiếu tới đối tượng khác.

**Ví dụ:**

```cpp
int value = 10;
int &ref = value;

ref = 20;
```

**Kết quả:**

```text
value = 20
```

- `ref` và `value` cùng biểu diễn một đối tượng.

<a id="muc-02-02"></a>
### 2.2. Con trỏ và tham chiếu

| Tiêu chí | Con trỏ | Tham chiếu |
|---|---|---|
| Cú pháp | `int *ptr` | `int &ref` |
| Có thể là rỗng | Có thể dùng `nullptr` | Thông thường phải tham chiếu tới một đối tượng hợp lệ |
| Có thể đổi đối tượng đang trỏ/tham chiếu | Có | Không |
| Truy cập giá trị | `*ptr` | Dùng trực tiếp như biến |
| Số học con trỏ | Có | Không |

**Ví dụ:**

```cpp
int a = 10;
int b = 20;

int *ptr = &a;
ptr = &b;

int &ref = a;
// ref = b; không làm ref chuyển sang b.
// Câu lệnh này gán giá trị của b cho a.
```

<a id="muc-02-03"></a>
### 2.3. Tham chiếu `const`

- `const T&` cho phép hàm nhận một đối tượng mà không sao chép toàn bộ đối tượng (tức là không tạo ra bản sao của đối tượng giống như truyền tham trị).
- Hàm không được sửa đối tượng thông qua tham chiếu `const`.
- Đây là cách rất thường dùng khi truyền `struct`, `class` hoặc đối tượng lớn.

**Ví dụ:**

```cpp
struct SensorData
{
    int temperature;
    int humidity;
};

void printData(const SensorData &data)
{
    // Có thể đọc data
    // Không được sửa data.temperature tại đây
}
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-03"></a>
## 3. Hàm trong C++

<a id="muc-03-01"></a>
### 3.1. Nạp chồng hàm

- C++ cho phép nhiều hàm có cùng tên nhưng có **danh sách tham số khác nhau**.
- Cơ chế này gọi là **nạp chồng hàm (function overloading)**.
- Khi gọi hàm, trình biên dịch sẽ dựa vào các đối số được truyền vào để chọn phiên bản phù hợp.

#### Điều kiện để nạp chồng hàm

Các hàm cùng tên phải có danh sách tham số khác nhau theo ít nhất một trong các cách sau:

**Khác kiểu dữ liệu của tham số:**

```cpp
void print(int x);
void print(double x);
```

**Khác số lượng tham số:**

```cpp
void print(int x);
void print(int x, int y);
```

**Khác thứ tự kiểu dữ liệu của tham số:**

```cpp
void bieuDien(int x, double y);
void bieuDien(double x, int y);
```

#### Những trường hợp không tạo thành nạp chồng hợp lệ

**Không thể nạp chồng chỉ bằng kiểu trả về:**

```cpp
int getValue();
float getValue();   // Lỗi
```

- Trình biên dịch không thể chọn hàm chỉ dựa vào kiểu trả về.

**Tên tham số khác nhau không tạo thành hàm khác:**

```cpp
void print(int x);
void print(int y);  // Cùng một danh sách kiểu tham số
```

**Giá trị mặc định của tham số không tạo thành một phiên bản nạp chồng mới:**

```cpp
void print(int x);
void print(int x = 10);  // Không phải nạp chồng hợp lệ
```

**`const` ở mức ngoài cùng của tham số truyền theo giá trị không tạo kiểu tham số khác:**

```cpp
void print(int x);
void print(const int x);  // Không nạp chồng được
```

Tuy nhiên, với con trỏ hoặc tham chiếu, `const` có thể tạo ra kiểu tham số khác:

```cpp
void print(int *p);
void print(const int *p);  // Hợp lệ
```

#### Ý cần nhớ

> **Nạp chồng hàm yêu cầu các hàm cùng tên phải khác nhau về danh sách kiểu tham số; không thể phân biệt chỉ bằng kiểu trả về.**

<a id="muc-03-02"></a>
### 3.2. Đối số mặc định

- Có thể đặt giá trị mặc định cho tham số.
- Nếu người gọi không truyền đối số đó, giá trị mặc định được sử dụng.

**Ví dụ:**

```cpp
void setSpeed(int speed = 50)
{
    // ...
}

int main()
{
    setSpeed();      // tốc độ = 50
    setSpeed(80);    // tốc độ = 80
}
```

<a id="muc-03-03"></a>
### 3.3. Truyền tham số

Ba cách thường gặp:

#### Truyền theo giá trị

```cpp
void update(int value)
{
    value++;
}
```

- Hàm nhận bản sao.
- Thay đổi bên trong hàm không ảnh hưởng đối tượng gốc.

#### Truyền bằng con trỏ

```cpp
void update(int *value)
{
    if (value != nullptr) {
        (*value)++;
    }
}
```

- Có thể truyền `nullptr`.
- Phải giải tham chiếu để truy cập đối tượng.

#### Truyền bằng tham chiếu

```cpp
void update(int &value)
{
    value++;
}
```

- Cú pháp gọn hơn con trỏ khi đối tượng bắt buộc phải tồn tại.

#### Truyền bằng tham chiếu `const`

```cpp
void process(const SensorData &data)
{
    // Đọc data nhưng không sửa
}
```

- Rất thường dùng cho đối tượng lớn chỉ cần đọc.

<a id="muc-03-04"></a>
### 3.4. Hàm `inline`

- `inline` cho biết hàm là ứng viên để trình biên dịch chèn mã trực tiếp tại nơi gọi hàm (với hàm thông thường, khi lời gọi hàm được gọi thì CPU sẽ phải tạo Stack Frame).
- Trình biên dịch vẫn có quyền quyết định có thực hiện chèn hay không.
- `inline` còn có vai trò liên quan đến quy tắc định nghĩa hàm trong nhiều đơn vị dịch.

**Ví dụ:**

```cpp
inline int square(int x)
{
    return x * x;
}
```

- Trong hệ thống nhúng, các hàm rất nhỏ có thể được viết dưới dạng `inline` hoặc `static inline`.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-04"></a>
## 4. Lớp và đối tượng

<a id="muc-04-01"></a>
### 4.1. `class` và đối tượng

- `class` mô tả dữ liệu và hành vi của một loại đối tượng.
- Đối tượng là một thực thể được tạo từ `class`.

**Ví dụ:**

```cpp
class Motor
{
public:
    void setSpeed(int value)
    {
        speed = value;
    }

    int getSpeed() const
    {
        return speed;
    }

private:
    int speed = 0;
};

int main()
{
    Motor motor;

    motor.setSpeed(50);
    int speed = motor.getSpeed();
}
```

- `Motor` là lớp.
- `motor` là đối tượng của lớp `Motor`.

<a id="muc-04-02"></a>
### 4.2. `class` và `struct`

Trong C++, `class` và `struct` gần giống nhau về khả năng.

Khác biệt mặc định quan trọng:

| | `class` | `struct` |
|---|---|---|
| Quyền truy cập mặc định | `private` | `public` |
| Kiểu kế thừa mặc định | `private` | `public` |

**Ví dụ:**

```cpp
struct Point
{
    int x;
    int y;
};
```

```cpp
class Point
{
    int x;   // private mặc định
    int y;
};
```

<a id="muc-04-03"></a>
### 4.3. `public`, `private`, `protected`

- `public`: có thể truy cập từ bên ngoài lớp.
- `private`: chỉ lớp và các thành phần được phép mới truy cập trực tiếp.
- `protected`: lớp dẫn xuất (lớp con) có thể truy cập, nhưng mã bên ngoài lớp không truy cập trực tiếp.

**Ví dụ:**

```cpp
class Device
{
public:
    void start()
    {
        state = 1;
    }

private:
    int state = 0;
};
```


#### Ví dụ kinh điển về đóng gói

- **Đóng gói** là việc gom dữ liệu và các hàm thao tác với dữ liệu đó vào cùng một lớp.
- Dữ liệu bên trong thường được đặt `private` để mã bên ngoài không thể sửa trực tiếp.
- Việc đọc hoặc thay đổi dữ liệu được thực hiện thông qua các hàm `public` do lớp cung cấp.
- Mục đích là bảo vệ trạng thái của đối tượng và kiểm soát cách dữ liệu được thay đổi.

**Ví dụ:**

```cpp
#include <iostream>

class BankAccount
{
private:
    int balance = 0;  // Dữ liệu được đóng gói và bảo vệ

public:
    void deposit(int amount)
    {
        if (amount > 0) {
            balance += amount;
        }
    }

    bool withdraw(int amount)
    {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }

        return false;
    }

    int getBalance() const
    {
        return balance;
    }
};

int main()
{
    BankAccount account;

    account.deposit(1000);
    account.withdraw(300);

    std::cout << account.getBalance() << "\n";

    // account.balance = 100000;
    // Lỗi: balance là private nên mã bên ngoài không được sửa trực tiếp.

    return 0;
}
```

**Giải thích:**

- `balance` được đặt `private`, nên bên ngoài lớp không thể gán tùy ý.
- Muốn tăng số dư phải đi qua `deposit()`.
- Muốn giảm số dư phải đi qua `withdraw()`, nên lớp có thể kiểm tra điều kiện trước khi thay đổi.
- `getBalance()` chỉ đọc dữ liệu nên được khai báo `const`.

**Ý cần nhớ:**

> Đóng gói không chỉ là “giấu biến bằng `private`”, mà là **kiểm soát cách trạng thái của đối tượng được truy cập và thay đổi**.

<a id="muc-04-04"></a>
### 4.4. Hàm thành viên

- Hàm được khai báo bên trong lớp gọi là hàm thành viên.
- Hàm thành viên có thể truy cập các thành viên của đối tượng.

```cpp
class Led
{
public:
    void on()
    {
        state = true;
    }

private:
    bool state = false;
};
```

<a id="muc-04-05"></a>
### 4.5. Con trỏ `this`

- Trong hàm thành viên không phải `static`, `this` là con trỏ trỏ tới đối tượng đang gọi hàm thành viên hiện tại.
- `this->thanh_vien` biểu diễn cách truy cập một thành viên của chính đối tượng đó.

**Ví dụ:**

```cpp
class Motor
{
public:
    void setSpeed(int speed)
    {
        this->speed = speed;
    }

private:
    int speed = 0;
};
```
Khi gọi
```cpp
Motor motor;
motor.setSpeed(50);
```
thì bên trong setSpeed():
```cpp
this
```
sẽ trỏ tới chính đối tượng:
```cpp
motor
```

- Ở đây `this->speed` là biến thành viên.
- `speed` bên phải là tham số hàm.

<a id="muc-04-06"></a>
### 4.6. Thành viên `static`

- Thành viên dữ liệu `static` thuộc về lớp, dùng chung giữa các đối tượng.
- Hàm thành viên `static` không có con trỏ `this`.

**Ví dụ:**

```cpp
class Motor
{
public:
    static int count;
};

int Motor::count = 0;
```
thì count chỉ có `một bản duy nhất`, dùng chung cho mọi đối tượng Motor.
```cpp
Motor m1;
Motor m2;

m1.count = 5;

std::cout << m2.count;
```
Kết quả:
```cpp
5
```
Vì:
```cpp
m1.count
m2.count
Motor::count
```
đều đang truy cập `cùng một biến` count.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-05"></a>
## 5. Hàm dựng, hàm hủy và vòng đời đối tượng

<a id="muc-05-01"></a>
### 5.1. Hàm dựng (constructor)

- Hàm dựng là hàm đặc biệt được gọi khi đối tượng được tạo.
- Hàm dựng có cùng tên với lớp và không có kiểu trả về.
- Dùng để đưa đối tượng vào trạng thái hợp lệ ban đầu.

**Ví dụ:**

```cpp
class Motor
{
public:
    Motor()
    {
        speed = 0;
    }

private:
    int speed;
};
```

#### Hàm dựng có tham số

```cpp
class Motor
{
public:
    Motor(int initial_speed)
    {
        speed = initial_speed;
    }

private:
    int speed;
};

Motor motor(50);
```

#### Nạp chồng hàm dựng (Overload)

```cpp
class Motor {
private:
    int speed;
public:
    // 1. Hàm dựng mặc định (không tham số)
    Motor() {
        speed = 0; 
    }

    // 2. Hàm dựng có tham số (Nạp chồng)
    Motor(int initial_speed) {
        this->speed = initial_speed; 
    }
};

int main() {
    Motor motor1;        // Gọi hàm dựng 1 (speed sẽ bằng 0)
    Motor motor2(1000);  // Gọi hàm dựng 2 (speed sẽ bằng 1000)
}
```

<a id="muc-05-02"></a>
### 5.2. Danh sách khởi tạo

- Nên dùng **danh sách khởi tạo (danh sách khởi tạo thành viên)** để khởi tạo thành viên.
- Một số thành viên như tham chiếu hoặc thành viên `const` bắt buộc phải được khởi tạo theo cách này.

**Ví dụ:**

```cpp
class Motor
{
public:
    Motor(int initial_speed)
        : speed(initial_speed)
    {
    }

private:
    int speed;
};
```

<a id="muc-05-03"></a>
### 5.3. Hàm hủy (destructor)

- Hàm hủy được gọi khi đối tượng kết thúc vòng đời.
- Tên hàm hủy là tên lớp có thêm `~` ở phía trước.
- Hàm hủy không nhận tham số và không có kiểu trả về (một lớp `chỉ có thể tồn tại duy nhất một hàm hủy`).

**Ví dụ:**

```cpp
class Device
{
public:
    Device()
    {
        // Khởi tạo tài nguyên
    }

    ~Device()
    {
        // Giải phóng tài nguyên
    }
};
```

<a id="muc-05-04"></a>
### 5.4. Vòng đời đối tượng

Ví dụ đối tượng cục bộ:

```cpp
void run()
{
    Motor motor(50);

    // đối tượng motor tồn tại trong phạm vi hàm
}
```

- Hàm dựng chạy khi `motor` được tạo.
- Hàm hủy chạy khi `motor` hết vòng đời (tức là hàm `run()` kết thúc) khi rời khỏi phạm vi.
- Các đối tượng cục bộ trong cùng phạm vi thường bị hủy theo thứ tự ngược với thứ tự được tạo.

**Ví dụ:**

```cpp
void test()
{
    Device a;
    Device b;

    // Khi rời khỏi hàm:
    // b được hủy trước
    // a được hủy sau
}
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-06"></a>
## 6. `const` trong C++

<a id="muc-06-01"></a>
### 6.1. Đối tượng `const`

```cpp
const int max_speed = 100;
```

- Không được sửa giá trị thông qua tên `max_speed`.

Với đối tượng lớp:

```cpp
const Motor motor;
```

- Đối tượng `const` chỉ được phép gọi các hàm thành viên được khai báo `const`, tức là các hàm cam kết không thay đổi trạng thái của đối tượng thông qua `this`.

**Ví dụ:**

```cpp
#include <iostream>

class Motor {
private:
    int speed = 0;

public:
    Motor(int initial_speed) {
        this->speed = initial_speed;
    }

    // Hàm thành viên thường:
    // Không có từ khóa const ở cuối nên hàm này
    // có quyền thay đổi trạng thái của đối tượng.
    void spin(int new_speed) {
        this->speed = new_speed;  // Thay đổi tốc độ của động cơ
    }

    // Hàm thành viên const:
    // const ở cuối nghĩa là hàm này không được phép
    // thay đổi trạng thái của đối tượng thông qua this.
    int get_speed() const {
        // this->speed = 100;
        // Lỗi biên dịch vì get_speed() là hàm const.

        return this->speed;  // Chỉ đọc giá trị speed nên hợp lệ
    }
};

int main() {
    const Motor motor(1000);

    // HỢP LỆ:
    // motor là const và get_speed() cũng là hàm const.
    std::cout << "Toc do hien tai: "
              << motor.get_speed()
              << "\n";

    // KHÔNG HỢP LỆ:
    // spin() là hàm thành viên thường, có khả năng sửa speed.
    // Vì motor là đối tượng const nên không được gọi hàm này.
    // motor.spin(2000);

    return 0;
}

```

<a id="muc-06-02"></a>
### 6.2. Hàm thành viên `const`

- Từ khóa `const` sau danh sách tham số cho biết hàm không thay đổi trạng thái quan sát được của đối tượng thông qua `this`.

**Ví dụ:**

```cpp
class Motor
{
public:
    int getSpeed() const
    {
        return speed;
    }

private:
    int speed = 0;
};
```

- Hàm đọc thường nên là hàm `const` nếu không sửa đối tượng.

<a id="muc-06-03"></a>
### 6.3. `const` với tham số

**Truyền tham chiếu `const`:**

```cpp
void process(const SensorData &data)
{
    // Chỉ đọc
}
```

**Truyền con trỏ tới dữ liệu `const`:**

```cpp
void process(const SensorData *data)
{
    if (data != nullptr) {
        // Chỉ đọc thông qua data
    }
}
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-07"></a>
## 7. RAII

<a id="muc-07-01"></a>
### 7.1. Khái niệm RAII

RAII là viết tắt của **Resource Acquisition Is Initialization** — có thể hiểu là **gắn việc quản lý tài nguyên với quá trình khởi tạo và vòng đời đối tượng**.

Ý tưởng chính:

- Tài nguyên được gắn với vòng đời của một đối tượng.
- Hàm dựng thiết lập hoặc nhận quyền sở hữu tài nguyên.
- Hàm hủy giải phóng tài nguyên.
- Khi đối tượng ra khỏi phạm vi, việc dọn dẹp diễn ra tự động.

Tài nguyên có thể là:

- Vùng nhớ động.
- Khóa đồng bộ.
- Tệp.
- Kết nối.
- Tài nguyên phần cứng hoặc định danh tài nguyên của trình điều khiển.

**Lợi ích:**

- Giảm nguy cơ quên giải phóng tài nguyên, dẫn đến `Memory Leak`.
- Mã nguồn dễ kiểm soát hơn.
- Hoạt động tốt khi hàm có nhiều đường `return`.

<a id="muc-07-02"></a>
### 7.2. Ví dụ RAII

```cpp
class PeripheralGuard
{
public:
    PeripheralGuard()
    {
        enablePeripheral();
    }

    ~PeripheralGuard()
    {
        disablePeripheral();
    }

private:
    void enablePeripheral()
    {
        // Bật ngoại vi
    }

    void disablePeripheral()
    {
        // Tắt ngoại vi
    }
};
```

Sử dụng:

```cpp
void run()
{
    PeripheralGuard guard;

    // Ngoại vi đang được quản lý trong phạm vi này
}
```

- Khi `guard` được tạo, hàm dựng chạy.
- Khi hàm kết thúc, hàm hủy tự chạy.
- Người dùng lớp không cần nhớ gọi hàm dọn dẹp thủ công.

> Trong C++ hiện đại, RAII là một trong những nguyên tắc quan trọng nhất cần hiểu.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-08"></a>
## 8. Sao chép đối tượng

<a id="muc-08-01"></a>
### 8.1. Hàm dựng sao chép

- Hàm dựng sao chép tạo một đối tượng mới từ một đối tượng đã tồn tại.

**Dạng thường gặp:**

```cpp
class Data
{
public:
    Data(const Data &other)
    {
        value = other.value;
    }

private:
    int value = 0;
};
```

**Ví dụ gọi:**

```cpp
Data a;
Data b = a; // tương đương với Data b(a);
```

<a id="muc-08-02"></a>
### 8.2. Phép gán sao chép

- Phép gán sao chép gán nội dung từ một đối tượng đã có sang một đối tượng khác đã tồn tại.

**Dạng thường gặp:**

```cpp
class Data
{
public:
    Data &operator=(const Data &other)
    {
        if (this != &other) {
            value = other.value;
        }

        return *this;
    }

private:
    int value = 0;
};
```

**Ví dụ:**

```cpp
Data a;
Data b;

b = a;
```

<a id="muc-08-03"></a>
### 8.3. Sao chép nông và sao chép sâu

#### Sao chép nông

- Chỉ sao chép trực tiếp giá trị các thành viên.
- Nếu thành viên là con trỏ sở hữu vùng nhớ, hai đối tượng có thể cùng giữ một địa chỉ.
- Điều này có thể dẫn tới sửa chung dữ liệu hoặc giải phóng cùng vùng nhớ nhiều lần.

Nếu chỉ sao chép `data`, cả hai đối tượng có thể giữ cùng một địa chỉ.

#### Sao chép sâu

- Tạo vùng tài nguyên mới cho đối tượng đích.
- Sao chép nội dung tài nguyên thay vì chỉ sao chép địa chỉ.

**Ví dụ vấn đề:**

```cpp
class Buffer
{
public:
    int *data;
};
```
Nếu sao chép nông:
```cpp
obj1.data ──┐
            ├──> cùng một vùng nhớ
obj2.data ──┘
```
Nếu sao chép sâu:
```cpp
obj1.data ─────> vùng nhớ A

obj2.data ─────> vùng nhớ B
```

**Ý cần nhớ khi phỏng vấn:**

- Với lớp chỉ chứa kiểu dữ liệu thông thường, cơ chế sao chép mặc định thường có thể đủ.
- Với lớp **sở hữu tài nguyên**, phải suy nghĩ rõ cách sao chép và giải phóng tài nguyên.
- Đây là lý do hàm dựng sao chép và phép gán sao chép quan trọng.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-09"></a>
## 9. Ngữ nghĩa di chuyển

<a id="muc-09-01"></a>
### 9.1. Ý tưởng của di chuyển tài nguyên

- Sao chép tạo bản sao tài nguyên.
- Di chuyển chuyển quyền sở hữu tài nguyên từ đối tượng này sang đối tượng khác.
- Cơ chế di chuyển có thể tránh việc sao chép dữ liệu lớn không cần thiết.

Ví dụ ý tưởng:

```text
Sao chép:
A ---- dữ liệu
B ---- bản sao dữ liệu

Di chuyển:
A ---- không còn sở hữu dữ liệu
B ---- nhận tài nguyên cũ của A
```

<a id="muc-09-02"></a>
### 9.2. `std::move`

- `std::move()` không tự di chuyển dữ liệu.
- Nó chuyển biểu thức sang dạng cho phép hàm dựng di chuyển hoặc phép gán di chuyển được lựa chọn nếu kiểu dữ liệu hỗ trợ.

```cpp
#include <utility>

Buffer b = std::move(a);
```

- Sau khi di chuyển, đối tượng nguồn vẫn phải ở trạng thái hợp lệ để bị hủy hoặc gán lại.
- Không nên giả định dữ liệu cũ của đối tượng nguồn vẫn giữ nguyên.

<a id="muc-09-03"></a>
### 9.3. Hàm dựng di chuyển và phép gán di chuyển

**Dạng hàm dựng di chuyển:**

```cpp
class Buffer
{
public:
    Buffer(Buffer &&other)
    {
        data = other.data;
        other.data = nullptr;
    }

private:
    int *data = nullptr;
};
```

**Dạng phép gán di chuyển:**

```cpp
Buffer &operator=(Buffer &&other)
{
    if (this != &other) {
        delete data;

        data = other.data;
        other.data = nullptr;
    }

    return *this;
}
```

Ở mức Intern cần hiểu:

- Tại sao cơ chế di chuyển tồn tại.
- `std::move()` dùng để làm gì.
- Di chuyển thường chuyển quyền sở hữu tài nguyên.
- Sau khi di chuyển, đối tượng nguồn vẫn hợp lệ nhưng trạng thái cụ thể có thể thay đổi.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-10"></a>
## 10. Kế thừa và đa hình

<a id="muc-10-01"></a>
### 10.1. Kế thừa

- Kế thừa cho phép lớp dẫn xuất (lớp con) sử dụng hoặc mở rộng phương thức, hành vi của lớp cơ sở (lớp cha).

**Ví dụ:**

```cpp
class Device
{
public:
    void init()
    {
        // Khởi tạo chung
    }
};

class Motor : public Device
{
public:
    void setSpeed(int speed)
    {
        // ...
    }
};
```


#### Ví dụ kinh điển về kế thừa

- **Kế thừa** cho phép lớp con sử dụng lại các thuộc tính, phương thức của lớp cha và bổ sung thuộc tính, phương thức riêng.
- Quan hệ thường được hiểu theo kiểu **“là một”**.

Ví dụ:

```text
Car là một Vehicle
Motorcycle là một Vehicle
```

**Ví dụ C++:**

```cpp
#include <iostream>

class Vehicle
{
public:
    void start()
    {
        std::cout << "Phuong tien khoi dong\n";
    }
};

class Car : public Vehicle
{
public:
    void openTrunk()
    {
        std::cout << "Mo cop xe\n";
    }
};

int main()
{
    Car car;

    car.start();      // Kế thừa từ Vehicle
    car.openTrunk();  // Hàm riêng của Car

    return 0;
}
```

**Giải thích:**

- `Car : public Vehicle` nghĩa là `Car` kế thừa công khai từ `Vehicle`.
- `Car` nhận được hàm `start()` từ lớp cha.
- `Car` có thể bổ sung thêm hành vi riêng như `openTrunk()`.
- Nhờ kế thừa, phần chức năng chung không cần viết lại trong từng lớp con.

**Ý cần nhớ:**

> Kế thừa chủ yếu dùng để **tái sử dụng và mở rộng phương thức, hành vi của lớp cơ sở** khi giữa hai lớp có quan hệ “là một”.

<a id="muc-10-02"></a>
### 10.2. Hàm `virtual`

- Hàm `virtual` cho phép lời gọi hàm được chọn theo kiểu đối tượng thực tế khi truy cập thông qua con trỏ hoặc tham chiếu tới lớp cơ sở (lớp cha).
- Đây là cơ sở của **đa hình lúc chạy**.

**Ví dụ:**

```cpp
class Sensor
{
public:
    virtual int read()
    {
        return 0;
    }
};

class TemperatureSensor : public Sensor
{
public:
    int read() override
    {
        return 25;
    }
};
```

Sử dụng:

```cpp
TemperatureSensor temperature_sensor;
Sensor &sensor = temperature_sensor;

int value = sensor.read();
```

**Kết quả:**

```text
value = 25
```


#### Ví dụ kinh điển về đa hình

- **Đa hình** cho phép cùng một lời gọi hàm nhưng hành vi thực tế khác nhau tùy đối tượng.
- Trong C++, đa hình lúc chạy thường được thực hiện bằng:
  - Kế thừa.
  - Hàm `virtual`.
  - Con trỏ hoặc tham chiếu tới lớp cơ sở.

**Ví dụ:**

```cpp
#include <iostream>

class Animal
{
public:
    virtual void sound() const
    {
        std::cout << "Am thanh cua dong vat\n";
    }

    virtual ~Animal() = default;
};

class Dog : public Animal
{
public:
    void sound() const override
    {
        std::cout << "Cho sua\n";
    }
};

class Cat : public Animal
{
public:
    void sound() const override
    {
        std::cout << "Meo keu\n";
    }
};

void makeSound(const Animal &animal)
{
    animal.sound();
}

int main()
{
    Dog dog;
    Cat cat;

    makeSound(dog);
    makeSound(cat);

    return 0;
}
```

**Kết quả:**

```text
Cho sua
Meo keu
```

**Giải thích:**

- `makeSound()` chỉ nhận một tham chiếu `const Animal&`.
- Khi truyền `Dog`, lời gọi `animal.sound()` chạy `Dog::sound()`.
- Khi truyền `Cat`, lời gọi giống hệt nhưng chạy `Cat::sound()`.
- `virtual` làm cho hàm được lựa chọn theo **kiểu thực tế của đối tượng** tại thời gian chạy.

Có thể hình dung:

```text
Cùng một lời gọi:
animal.sound()

Dog -> Dog::sound()
Cat -> Cat::sound()
```

**Ý cần nhớ:**

> Đa hình cho phép mã phía trên làm việc thông qua một **giao diện chung**, còn mỗi lớp con tự quyết định cách thực hiện hành vi đó.

<a id="muc-10-03"></a>
### 10.3. `override`

- `override` cho trình biên dịch kiểm tra rằng hàm thực sự đang ghi đè một hàm `virtual` ở lớp cơ sở (lớp cha).
- Nên dùng `override` khi ghi đè hàm ảo.

```cpp
int read() override
{
    return 25;
}
```

Nếu chữ ký không khớp với hàm ở lớp cơ sở, trình biên dịch sẽ báo lỗi.

<a id="muc-10-04"></a>
### 10.4. Hàm thuần ảo và lớp trừu tượng

- Hàm thuần ảo có dạng:

```cpp
virtual int read() = 0;
```

- Lớp có ít nhất một hàm thuần ảo là lớp trừu tượng (abstract).
- Không thể tạo trực tiếp đối tượng của lớp trừu tượng (abstract).
- Thường dùng để định nghĩa một giao diện chung.

**Ví dụ giao diện trình điều khiển:**

```cpp
class Sensor
{
public:
    virtual int read() = 0;
    virtual ~Sensor() = default;
};
```

Lớp cụ thể:

```cpp
class TemperatureSensor : public Sensor
{
public:
    int read() override
    {
        return 25;
    }
};
```


#### Ví dụ kinh điển về trừu tượng hóa

- **Trừu tượng hóa** là chỉ cung cấp cho bên sử dụng những gì cần thiết và che giấu chi tiết triển khai bên trong.
- Người dùng chỉ cần biết **đối tượng làm được gì**, không cần biết toàn bộ **nó làm như thế nào**.
- Trong C++, có thể tạo trừu tượng hóa bằng:
  - Lớp.
  - Hàm `public` che giấu dữ liệu và xử lý bên trong.
  - Lớp trừu tượng và hàm thuần ảo.

**Ví dụ gần với Embedded:**

```cpp
#include <iostream>

class Sensor
{
public:
    virtual int read() = 0;
    virtual ~Sensor() = default;
};

class TemperatureSensor : public Sensor
{
public:
    int read() override
    {
        // Chi tiết thật có thể gồm:
        // 1. Đọc thanh ghi ADC.
        // 2. Chuyển đổi giá trị ADC.
        // 3. Tính ra nhiệt độ.
        return 25;
    }
};

void printSensorValue(Sensor &sensor)
{
    std::cout << sensor.read() << "\n";
}

int main()
{
    TemperatureSensor sensor;

    printSensorValue(sensor);

    return 0;
}
```

**Giải thích:**

Phần chương trình sử dụng cảm biến chỉ cần biết:

```cpp
sensor.read();
```

Nó không cần biết bên trong `TemperatureSensor::read()` thực hiện:

```text
Đọc ADC
    ↓
Xử lý dữ liệu
    ↓
Chuyển đổi sang nhiệt độ
    ↓
Trả về kết quả
```

Lớp `Sensor` chỉ định nghĩa **giao diện chung**:

```cpp
virtual int read() = 0;
```

Còn lớp cụ thể quyết định cách thực hiện:

```cpp
class TemperatureSensor : public Sensor
```

**Ý cần nhớ:**

> Trừu tượng hóa là **ẩn chi tiết triển khai và chỉ đưa ra giao diện cần thiết cho người sử dụng**.

#### Phân biệt nhanh đóng gói và trừu tượng hóa

| Khái niệm | Ý chính |
|---|---|
| Đóng gói | Bảo vệ và kiểm soát dữ liệu/trạng thái bên trong đối tượng. |
| Trừu tượng hóa | Ẩn chi tiết triển khai và chỉ cung cấp giao diện cần thiết. |

Ví dụ với một `Motor`:

```text
Đóng gói:
speed là private
→ Không cho bên ngoài sửa trực tiếp.

Trừu tượng hóa:
motor.start()
→ Người dùng không cần biết bên trong phải cấu hình GPIO,
  PWM hoặc thanh ghi phần cứng như thế nào.
```

<a id="muc-10-05"></a>
### 10.5. Hàm hủy ảo

- Nếu một lớp được dùng làm lớp cơ sở đa hình và có thể bị hủy thông qua con trỏ lớp cơ sở, hàm hủy thường cần là `virtual`.

**Ví dụ:**

```cpp
class Device
{
public:
    virtual ~Device() = default;
};
```

Điều này giúp hàm hủy của lớp dẫn xuất được gọi đúng khi hủy đối tượng thông qua con trỏ lớp cơ sở.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-11"></a>
## 11. Khuôn mẫu (template)

<a id="muc-11-01"></a>
### 11.1. Khuôn mẫu hàm

- Khuôn mẫu cho phép viết một khuôn mã dùng được với nhiều kiểu dữ liệu.
- Trình biên dịch tạo phiên bản phù hợp khi khuôn mẫu được sử dụng.

**Ví dụ:**

```cpp
template <typename T>
T maxValue(T a, T b)
{
    return (a > b) ? a : b;
}
```

Sử dụng:

```cpp
int a = maxValue(10, 20);
float b = maxValue(1.5f, 2.5f);
```

<a id="muc-11-02"></a>
### 11.2. Khuôn mẫu lớp

```cpp
template <typename T>
class ValueHolder
{
public:
    explicit ValueHolder(T value)
        : value_(value)
    {
    }

    T get() const
    {
        return value_;
    }

private:
    T value_;
};
```

Sử dụng:

```cpp
ValueHolder<int> a(10);
ValueHolder<float> b(2.5f);
```

**Trong hệ thống nhúng:**

- Khuôn mẫu có thể tạo mã tổng quát mà không cần đa hình lúc chạy.
- Nhiều quyết định có thể được xử lý trong lúc biên dịch.
- Tuy nhiên dùng quá nhiều khuôn mẫu có thể làm tăng kích thước mã hoặc khiến lỗi biên dịch khó đọc hơn.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-12"></a>
## 12. STL cơ bản cho hệ thống nhúng

> Không phải dự án nhúng nào cũng sử dụng toàn bộ STL. Cần hiểu đặc tính tài nguyên của từng kiểu chứa trước khi dùng.

<a id="muc-12-01"></a>
### 12.1. `std::array`

- `std::array<T, N>` là mảng có kích thước cố định.
- Kích thước `N` được xác định tại lúc biên dịch và không thay đổi trong thời gian chạy.
- Dữ liệu được chứa trực tiếp trong đối tượng.
- Không tự cấp phát động cho các phần tử.
- Phù hợp với hệ thống nhúng khi số lượng phần tử đã biết trước.

**Ví dụ:**

```cpp
#include <array>

std::array<int, 4> values = {1, 2, 3, 4};

int first = values[0];
```

#### Các hàm thường dùng với `std::array`

| Hàm | Ý nghĩa |
|---|---|
| `size()` | Trả về số lượng phần tử của mảng. |
| `empty()` | Kiểm tra mảng có rỗng hay không. Với `std::array<T, N>`, chỉ trả về `true` khi `N == 0`. |
| `at(index)` | Truy cập phần tử theo chỉ số và có kiểm tra phạm vi. |
| `operator[]` | Truy cập phần tử theo chỉ số nhưng không kiểm tra phạm vi. |
| `front()` | Trả về phần tử đầu tiên. |
| `back()` | Trả về phần tử cuối cùng. |
| `data()` | Trả về con trỏ tới phần tử đầu tiên của vùng dữ liệu liên tiếp. |
| `fill(value)` | Gán cùng một giá trị cho toàn bộ phần tử. |
| `begin()` | Trả về bộ lặp trỏ tới phần tử đầu tiên. |
| `end()` | Trả về bộ lặp trỏ tới vị trí ngay sau phần tử cuối cùng. |

**Ví dụ:**

```cpp
#include <array>
#include <iostream>

int main()
{
    std::array<int, 4> values = {10, 20, 30, 40};

    std::cout << values.size() << "\n";   // 4
    std::cout << values.front() << "\n";  // 10
    std::cout << values.back() << "\n";   // 40

    values.at(1) = 25;                    // Phần tử thứ 2 thành 25
    values.fill(5);                       // Toàn bộ phần tử thành 5

    int *ptr = values.data();             // Trỏ tới phần tử đầu tiên

    return 0;
}
```

**Lưu ý:**

- `at()` an toàn hơn `operator[]` vì có kiểm tra phạm vi, nhưng có thể phát sinh ngoại lệ `std::out_of_range`.
- Trong một số dự án nhúng không dùng ngoại lệ, cần hiểu quy ước của dự án trước khi dựa vào `at()`.

<a id="muc-12-02"></a>
### 12.2. `std::vector`

- `std::vector<T>` là mảng động có thể thay đổi kích thước.
- Các phần tử được lưu liên tiếp trong bộ nhớ.
- Thông thường sử dụng vùng nhớ động để lưu dữ liệu.
- Khi không đủ sức chứa, `vector` có thể cấp phát vùng nhớ mới lớn hơn và chuyển/sao chép dữ liệu sang vùng mới.

**Ví dụ:**

```cpp
#include <vector>

std::vector<int> values;

values.push_back(10);
values.push_back(20);
```

#### Các hàm thường dùng với `std::vector`

| Hàm | Ý nghĩa |
|---|---|
| `size()` | Số phần tử hiện có. |
| `capacity()` | Số phần tử có thể chứa trước khi cần cấp phát lại. |
| `empty()` | Kiểm tra `vector` có rỗng hay không. |
| `push_back(value)` | Thêm một phần tử vào cuối. |
| `emplace_back(...)` | Tạo trực tiếp phần tử ở cuối từ các đối số truyền vào. |
| `pop_back()` | Xóa phần tử cuối cùng. |
| `clear()` | Xóa toàn bộ phần tử nhưng không bắt buộc giải phóng sức chứa đã cấp phát. |
| `reserve(n)` | Yêu cầu chuẩn bị sức chứa ít nhất `n` phần tử. |
| `resize(n)` | Thay đổi số lượng phần tử thành `n`. |
| `at(index)` | Truy cập theo chỉ số có kiểm tra phạm vi. |
| `operator[]` | Truy cập theo chỉ số không kiểm tra phạm vi. |
| `front()` | Phần tử đầu tiên. |
| `back()` | Phần tử cuối cùng. |
| `data()` | Con trỏ tới vùng dữ liệu liên tiếp. |
| `insert()` | Chèn phần tử vào vị trí xác định. |
| `erase()` | Xóa phần tử hoặc một vùng phần tử. |
| `begin()` / `end()` | Lấy bộ lặp đầu và cuối. |

**Ví dụ:**

```cpp
#include <vector>

int main()
{
    std::vector<int> values;

    values.reserve(5);       // Chuẩn bị sức chứa cho ít nhất 5 phần tử

    values.push_back(10);
    values.push_back(20);
    values.push_back(30);

    int first = values.front();
    int last  = values.back();

    values.pop_back();       // Xóa 30

    values.resize(5);        // Thay đổi số phần tử thành 5

    return 0;
}
```

#### Phân biệt `size()` và `capacity()`

```cpp
std::vector<int> values;

values.reserve(10);
values.push_back(1);
values.push_back(2);
```

Lúc này có thể hiểu:

```text
size()     = 2
capacity() >= 10
```

- `size()` là số phần tử đang tồn tại.
- `capacity()` là sức chứa hiện tại trước khi cần cấp phát lại.

#### `reserve()` và `resize()` khác nhau

- `reserve(n)`:
  - Chuẩn bị vùng nhớ để chứa ít nhất `n` phần tử.
  - Không làm thay đổi số phần tử hiện tại.

- `resize(n)`:
  - Thay đổi số phần tử thực sự thành `n`.
  - Có thể tạo thêm hoặc xóa bớt phần tử.

**Trong hệ thống nhúng:**

- Cần cân nhắc khi hệ thống hạn chế RAM hoặc cần thời gian thực thi xác định.
- Việc `vector` tăng sức chứa có thể dẫn tới cấp phát lại vùng nhớ.
- Nếu biết trước số phần tử tối đa, `reserve()` có thể giúp giảm số lần cấp phát lại.
- Nếu kích thước luôn cố định, `std::array` thường dễ kiểm soát tài nguyên hơn.

<a id="muc-12-03"></a>
### 12.3. `std::string`

- `std::string` quản lý chuỗi ký tự tự động.
- Dễ sử dụng hơn mảng `char`.
- Có thể liên quan tới cấp phát động tùy độ dài chuỗi và cách triển khai thư viện.

**Ví dụ:**

```cpp
#include <string>

std::string name = "Sensor";
name += "_1";
```

#### Các hàm thường dùng với `std::string`

| Hàm | Ý nghĩa |
|---|---|
| `size()` / `length()` | Trả về số ký tự trong chuỗi. |
| `empty()` | Kiểm tra chuỗi có rỗng hay không. |
| `clear()` | Xóa toàn bộ nội dung chuỗi. |
| `push_back(ch)` | Thêm một ký tự vào cuối. |
| `pop_back()` | Xóa ký tự cuối. |
| `append(str)` | Nối thêm chuỗi vào cuối. |
| `operator+=` | Nối thêm chuỗi hoặc ký tự. |
| `find(str)` | Tìm vị trí xuất hiện đầu tiên của chuỗi con. |
| `substr(pos, count)` | Tạo chuỗi con từ vị trí xác định. |
| `compare(str)` | So sánh nội dung hai chuỗi. |
| `c_str()` | Trả về con trỏ `const char *` tới chuỗi kết thúc bằng `\0`. |
| `front()` | Ký tự đầu tiên. |
| `back()` | Ký tự cuối cùng. |

**Ví dụ:**

```cpp
#include <string>

int main()
{
    std::string name = "Sensor";

    name.append("_1");          // "Sensor_1"
    name.push_back('A');        // "Sensor_1A"

    std::size_t pos = name.find("Sensor");

    std::string part = name.substr(0, 6);  // "Sensor"

    const char *c_text = name.c_str();

    return 0;
}
```

#### `find()` và `std::string::npos`

- Nếu tìm thấy, `find()` trả về vị trí bắt đầu.
- Nếu không tìm thấy, trả về `std::string::npos`.

```cpp
std::string text = "UART_OK";

if (text.find("OK") != std::string::npos) {
    // Tìm thấy chuỗi "OK"
}
```

**Trong hệ thống nhúng:**

- Với hệ thống rất hạn chế tài nguyên hoặc yêu cầu tính xác định cao, cần xem xét kỹ trước khi dùng `std::string`.
- Nếu chỉ cần bộ đệm ký tự cố định, mảng `char` hoặc cấu trúc dữ liệu có kích thước cố định có thể dễ kiểm soát bộ nhớ hơn.

<a id="muc-12-04"></a>
### 12.4. `std::pair`

- `std::pair<T1, T2>` gom hai giá trị vào cùng một đối tượng.
- Hai giá trị có thể có kiểu dữ liệu khác nhau.

**Ví dụ:**

```cpp
#include <utility>

std::pair<int, bool> result = {25, true};

int value = result.first;
bool valid = result.second;
```

#### Các thao tác thường dùng

| Thao tác | Ý nghĩa |
|---|---|
| `first` | Truy cập giá trị thứ nhất. |
| `second` | Truy cập giá trị thứ hai. |
| `std::make_pair(a, b)` | Tạo một `pair` từ hai giá trị. |
| `swap()` | Hoán đổi nội dung với một `pair` khác. |

**Ví dụ:**

```cpp
#include <utility>

int main()
{
    std::pair<int, bool> result = std::make_pair(25, true);

    int value = result.first;
    bool valid = result.second;

    std::pair<int, bool> other = {30, false};

    result.swap(other);

    return 0;
}
```

**Ứng dụng thường gặp:**

- Trả về hai giá trị từ một hàm.

```cpp
std::pair<int, bool> readSensor()
{
    int value = 25;
    bool valid = true;

    return {value, valid};
}
```

<a id="muc-12-05"></a>
### 12.5. Bộ lặp và vòng lặp phạm vi

- Bộ lặp là đối tượng dùng để duyệt các phần tử của kiểu chứa.
- Có thể hình dung bộ lặp có cách sử dụng gần giống con trỏ tới phần tử.

#### `begin()` và `end()`

```cpp
std::array<int, 4> values = {1, 2, 3, 4};

for (auto it = values.begin(); it != values.end(); ++it) {
    int value = *it;
}
```

- `begin()` trỏ tới phần tử đầu tiên.
- `end()` trỏ tới vị trí ngay sau phần tử cuối cùng.
- Không được giải tham chiếu `end()`.

#### `cbegin()` và `cend()`

- Trả về bộ lặp chỉ đọc.

```cpp
for (auto it = values.cbegin(); it != values.cend(); ++it) {
    // *it chỉ được đọc
}
```

#### `rbegin()` và `rend()`

- Dùng để duyệt ngược từ cuối về đầu.

```cpp
for (auto it = values.rbegin(); it != values.rend(); ++it) {
    // Duyệt ngược
}
```

#### Vòng lặp phạm vi

```cpp
std::array<int, 4> values = {1, 2, 3, 4};

for (int value : values) {
    // value là bản sao của từng phần tử
}
```

Nếu muốn sửa phần tử:

```cpp
for (int &value : values) {
    value++;
}
```

Nếu chỉ đọc và muốn tránh sao chép đối tượng lớn:

```cpp
for (const auto &value : values) {
    // Chỉ đọc
}
```

- `auto` cho phép trình biên dịch suy luận kiểu dữ liệu từ biểu thức khởi tạo.

<a id="muc-12-06"></a>
### 12.6. `std::algorithm`

- `<algorithm>` cung cấp nhiều thuật toán có thể làm việc với vùng dữ liệu thông qua bộ lặp.

#### Các thuật toán cơ bản nên biết

| Hàm | Ý nghĩa |
|---|---|
| `std::find()` | Tìm một giá trị trong vùng dữ liệu. |
| `std::count()` | Đếm số phần tử có giá trị xác định. |
| `std::sort()` | Sắp xếp vùng dữ liệu. |
| `std::min()` | Trả về giá trị nhỏ hơn trong hai giá trị. |
| `std::max()` | Trả về giá trị lớn hơn trong hai giá trị. |
| `std::min_element()` | Tìm phần tử nhỏ nhất trong một vùng. |
| `std::max_element()` | Tìm phần tử lớn nhất trong một vùng. |
| `std::reverse()` | Đảo ngược thứ tự phần tử. |
| `std::fill()` | Gán cùng một giá trị cho một vùng phần tử. |
| `std::copy()` | Sao chép một vùng phần tử sang vùng khác. |

#### `std::find()`

```cpp
#include <algorithm>
#include <array>

std::array<int, 4> values = {10, 20, 30, 40};

auto it = std::find(values.begin(), values.end(), 30);

if (it != values.end()) {
    // Đã tìm thấy 30
}
```

#### `std::sort()`

```cpp
#include <algorithm>
#include <array>

std::array<int, 4> values = {4, 1, 3, 2};

std::sort(values.begin(), values.end());
```

Kết quả:

```text
1 2 3 4
```

#### `std::min_element()` và `std::max_element()`

```cpp
#include <algorithm>
#include <array>

std::array<int, 4> values = {4, 1, 8, 2};

auto min_it = std::min_element(values.begin(), values.end());
auto max_it = std::max_element(values.begin(), values.end());

int min_value = *min_it;  // 1
int max_value = *max_it;  // 8
```

#### `std::fill()`

```cpp
std::array<int, 4> values;

std::fill(values.begin(), values.end(), 0);
```

Kết quả:

```text
0 0 0 0
```

#### `std::reverse()`

```cpp
std::array<int, 4> values = {1, 2, 3, 4};

std::reverse(values.begin(), values.end());
```

Kết quả:

```text
4 3 2 1
```

#### Ý cần nhớ ở mức Intern

- `std::array`: ưu tiên khi kích thước cố định.
- `std::vector`: kích thước thay đổi, thường có cấp phát động.
- `std::string`: thuận tiện cho chuỗi nhưng cần để ý bộ nhớ động.
- `std::pair`: gom hai giá trị thành một đối tượng.
- `begin()` và `end()`: xác định vùng dữ liệu để duyệt hoặc truyền cho thuật toán.
- Các thuật toán như `find`, `sort`, `min_element`, `max_element`, `fill`, `reverse` nên biết cách dùng cơ bản.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-13"></a>
## 13. Quản lý bộ nhớ trong C++

<a id="muc-13-01"></a>
### 13.1. `new` và `delete`

- `new` cấp phát vùng nhớ và tạo đối tượng.
- `delete` hủy đối tượng và giải phóng vùng nhớ tương ứng.

**Ví dụ:**

```cpp
Motor *motor = new Motor();

delete motor;
motor = nullptr;
```

Khác biệt quan trọng so với `malloc/free`:

- `new` gọi hàm dựng.
- `delete` gọi hàm hủy.
- `new` trả về con trỏ đúng kiểu, không cần ép kiểu.
- Không được trộn cặp:
  - `new` với `free()`
  - `malloc()` với `delete`

<a id="muc-13-02"></a>
### 13.2. `new[]` và `delete[]`

```cpp
int *values = new int[10];

delete[] values;
values = nullptr;
```

- Vùng nhớ tạo bằng `new[]` phải giải phóng bằng `delete[]`.

<a id="muc-13-03"></a>
### 13.3. `std::unique_ptr`

- `std::unique_ptr` biểu diễn quyền sở hữu duy nhất đối với một đối tượng động.
- Khi `unique_ptr` hết vòng đời, đối tượng được giải phóng tự động.
- Không sao chép được, nhưng có thể chuyển quyền sở hữu bằng cơ chế di chuyển.

**Ví dụ:**

```cpp
#include <memory>

auto motor = std::make_unique<Motor>();
```

Hoặc:

```cpp
std::unique_ptr<Motor> motor = std::make_unique<Motor>();
```

**Lợi ích:**

- Giảm nguy cơ quên `delete`.
- Thể hiện rõ ai đang sở hữu tài nguyên.
- Phù hợp với tư tưởng RAII.

<a id="muc-13-04"></a>
### 13.4. `std::shared_ptr`

- `std::shared_ptr` cho phép nhiều con trỏ thông minh cùng chia sẻ quyền sở hữu một đối tượng.
- Đối tượng được giải phóng khi không còn `shared_ptr` nào sở hữu nó.
- Thường cần cơ chế đếm số lượng tham chiếu nên có chi phí quản lý lớn hơn `unique_ptr`.

Ở mức Intern Embedded Firmware:

- Biết mục đích của `shared_ptr`.
- Biết nó có chi phí quản lý.
- Không cần học sâu về `weak_ptr` hoặc các trường hợp phức tạp nếu JD không yêu cầu.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-14"></a>
## 14. Ép kiểu trong C++

<a id="muc-14-01"></a>
### 14.1. `static_cast`

- Dùng cho các chuyển đổi kiểu có ý nghĩa rõ ràng và được trình biên dịch kiểm tra ở mức phù hợp.

**Ví dụ:**

```cpp
float value = 3.8f;
int integer = static_cast<int>(value);
```

**Kết quả:**

```text
integer = 3
```

<a id="muc-14-02"></a>
### 14.2. `reinterpret_cast`

- Dùng cho chuyển đổi mức thấp giữa các kiểu con trỏ hoặc giữa địa chỉ số và con trỏ trong các trường hợp phù hợp với nền tảng.
- Cần dùng rất cẩn thận.

**Ví dụ trong hệ thống nhúng:**

```cpp
#include <cstdint>

volatile std::uint32_t *reg =
    reinterpret_cast<volatile std::uint32_t *>(0x40000000u);
```

- Địa chỉ chỉ hợp lệ nếu đúng với sơ đồ ánh xạ bộ nhớ của vi điều khiển đích.

<a id="muc-14-03"></a>
### 14.3. `const_cast`

- Dùng để thêm hoặc loại bỏ thuộc tính `const`/`volatile` trong một số trường hợp.
- Không nên dùng để cố sửa một đối tượng thực sự được định nghĩa là `const`.

**Ví dụ:**

```cpp
const int value = 10;
const int *ptr = &value;

// int *p = const_cast<int *>(ptr);
// Không được dùng p để sửa value vì value thực sự là const.
```

<a id="muc-14-04"></a>
### 14.4. `dynamic_cast`

- Dùng với hệ thống kiểu đa hình để kiểm tra/chuyển đổi kiểu tại lúc chạy.
- Thường liên quan tới RTTI.
- Một số dự án nhúng tắt RTTI nên `dynamic_cast` có thể không được sử dụng.

Ở mức Intern:

- Biết mục đích.
- Không cần học sâu nếu dự án không dùng RTTI.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-15"></a>
## 15. `enum class` và `constexpr`

<a id="muc-15-01"></a>
### 15.1. `enum class`

- `enum class` tạo kiểu liệt kê có phạm vi tên riêng và kiểm tra kiểu chặt hơn `enum` truyền thống.

**Ví dụ:**

```cpp
enum class State
{
    Idle,
    Running,
    Error
};

State state = State::Idle;
```

- Phải truy cập giá trị theo phạm vi như `State::Idle`.
- Tránh làm các tên như `Idle`, `Running` tràn ra phạm vi bên ngoài.

<a id="muc-15-02"></a>
### 15.2. `constexpr`

- `constexpr` cho biết một giá trị hoặc hàm có thể tham gia vào tính toán lúc biên dịch khi điều kiện cho phép.
- Hữu ích khi giá trị đã biết từ lúc biên dịch.

**Ví dụ:**

```cpp
constexpr int BUFFER_SIZE = 128;
```

Hàm `constexpr`:

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int value = square(5);
```

<a id="muc-15-03"></a>
### 15.3. `static constexpr`

- Thường dùng cho hằng số thuộc về một lớp.

```cpp
class Uart
{
public:
    static constexpr int DEFAULT_BAUDRATE = 115200;
};
```

Sử dụng:

```cpp
int baudrate = Uart::DEFAULT_BAUDRATE;
```

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-16"></a>
## 16. Tương tác giữa C và C++

<a id="muc-16-01"></a>
### 16.1. Mã hóa tên ký hiệu

- C++ hỗ trợ nạp chồng hàm nên trình biên dịch thường mã hóa thêm thông tin kiểu vào tên ký hiệu của hàm.
- Cơ chế này thường được gọi là **mã hóa tên ký hiệu**.
- C không có cơ chế nạp chồng hàm tương tự, nên quy ước tên ký hiệu khác C++.

Ví dụ C++:

```cpp
void send(int value);
void send(float value);
```

Hai hàm cùng tên cần được trình biên dịch phân biệt trong tệp đối tượng.

<a id="muc-16-02"></a>
### 16.2. `extern "C"`

- Khi C++ cần gọi hàm được biên dịch theo giao diện C, thường dùng `extern "C"` để yêu cầu liên kết theo quy ước tên của C.

**Ví dụ:**

```cpp
extern "C" void HAL_Init(void);
```

Với tệp tiêu đề dùng được cho cả C và C++:

```c
#ifdef __cplusplus
extern "C" {
#endif

void driver_init(void);

#ifdef __cplusplus
}
#endif
```

- `__cplusplus` được định nghĩa khi mã đang được biên dịch bằng trình biên dịch C++.

**Trong hệ thống nhúng:**

- Rất thường gặp khi:
  - C++ gọi thư viện hoặc trình điều khiển được viết bằng C.
  - Dự án dùng HAL được viết bằng C nhưng phần ứng dụng được viết bằng C++.
  - ISR hoặc giao diện lập trình ứng dụng (API) của hệ thống yêu cầu giao diện C.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-17"></a>
## 17. Ngoại lệ và RTTI

<a id="muc-17-01"></a>
### 17.1. Ngoại lệ

C++ có cơ chế xử lý ngoại lệ:

```cpp
try {
    // Mã có thể phát sinh exception
}
catch (...) {
    // Xử lý
}
```

Các từ khóa chính:

- `try`
- `throw`
- `catch`

**Trong hệ thống nhúng:**

- Một số dự án không sử dụng ngoại lệ để giảm kích thước mã, giảm phụ thuộc thời gian chạy hoặc giữ hành vi dễ dự đoán hơn.
- Có dự án vẫn sử dụng ngoại lệ nếu nền tảng và yêu cầu cho phép.

Ở mức Intern:

- Biết ngoại lệ dùng để làm gì.
- Biết một số dự án nhúng có thể tắt ngoại lệ.
- Không cần học sâu cơ chế unwinding.

<a id="muc-17-02"></a>
### 17.2. RTTI

RTTI là viết tắt của **Run-Time Type Information** — **thông tin kiểu tại thời gian chạy**.

- Cho phép chương trình nhận biết một số thông tin kiểu trong lúc chạy.
- Các cơ chế thường liên quan:
  - `dynamic_cast`
  - `typeid`

**Trong hệ thống nhúng:**

- RTTI có thể bị tắt trong một số dự án để giảm chi phí tài nguyên.
- Nếu RTTI bị tắt, không thể dựa vào các tính năng cần RTTI.

[↑ Về mục lục](#muc-luc)

---

<a id="chuong-18"></a>
## 18. C++ trong phần mềm nhúng

<a id="muc-18-01"></a>
### 18.1. Cấp phát động

Các thành phần có thể sử dụng cấp phát động:

- `new`
- `std::vector`
- `std::string`
- Một số con trỏ thông minh và kiểu chứa khác.

Trong hệ thống nhúng:

- RAM có thể rất hạn chế.
- Cấp phát/giải phóng nhiều lần có thể gây phân mảnh heap.
- Thời gian cấp phát có thể khó dự đoán hơn bộ nhớ tĩnh.
- Hệ thống chạy lâu cần đặc biệt cẩn thận với rò rỉ bộ nhớ.

**Ưu tiên khi phù hợp:**

- Biến tĩnh.
- Biến cục bộ có kích thước cố định.
- `std::array`.
- Bộ đệm có kích thước xác định trước.

Không có nghĩa là cấp phát động luôn bị cấm; cần tuân theo yêu cầu của dự án.

<a id="muc-18-02"></a>
### 18.2. Chi phí của `virtual`

- Hàm `virtual` thường được triển khai bằng cơ chế bảng hàm ảo và lời gọi gián tiếp.
- Điều này có thể tạo thêm:
  - Một lượng dữ liệu quản lý cho đối tượng/lớp.
  - Một mức chi phí nhỏ cho lời gọi hàm.
  - Phụ thuộc vào trình biên dịch và kiến trúc.

**Kết luận:**

- Không cần tránh `virtual` một cách tuyệt đối.
- Dùng khi lợi ích thiết kế đáng giá và tài nguyên cho phép.
- Với phần mềm nhúng có tài nguyên rất hạn chế, cần biết chi phí trước khi sử dụng.

<a id="muc-18-03"></a>
### 18.3. Tính xác định

Phần mềm nhúng thường quan tâm tới:

- Thời gian thực thi có thể dự đoán.
- Sử dụng RAM/Flash có thể kiểm soát.
- Không rò rỉ tài nguyên.
- Không tạo cấp phát bất ngờ ở luồng xử lý quan trọng.

Khi dùng C++, cần chú ý:

- Cấp phát động.
- Kiểu chứa có thể tự cấp phát.
- Ngoại lệ nếu dự án cho phép.
- Điều phối hàm ảo.
- Hàm dựng/hàm hủy có thể chạy tự động theo vòng đời đối tượng.

<a id="muc-18-04"></a>
### 18.4. Những gì cần nhớ khi phỏng vấn

#### Nhóm bắt buộc

- Tham chiếu và khác biệt với con trỏ.
- `nullptr`.
- Nạp chồng hàm.
- `class` và đối tượng.
- `public`, `private`, `protected`.
- `class` khác `struct`.
- Hàm dựng và hàm hủy.
- Danh sách khởi tạo thành viên.
- Đối tượng lifetime.
- Hàm thành viên `const`.
- RAII.
- Hàm dựng sao chép.
- Phép gán sao chép.
- Sao chép nông và sao chép sâu.
- Kế thừa.
- `virtual`, `override`.
- Hàm thuần ảo.
- Lớp trừu tượng.
- Hàm hủy ảo.
- Khuôn mẫu cơ bản.
- `std::array`.
- Hiểu `std::vector`/`std::string` có thể dùng cấp phát động.
- `new/delete`.
- `std::unique_ptr`.
- `static_cast`, `reinterpret_cast`.
- `enum class`.
- `constexpr`.
- `extern "C"`.

#### Nhóm nên biết

- Ngữ nghĩa di chuyển.
- `std::move`.
- Hàm dựng di chuyển và phép gán di chuyển.
- `std::pair`.
- Bộ lặp.
- `std::algorithm`.
- `std::shared_ptr`.
- `const_cast`.
- `dynamic_cast`.
- Ngoại lệ và RTTI ở mức khái niệm.

#### Chưa cần học sâu ở mức Intern

- Siêu lập trình bằng khuôn mẫu.
- Chuyển tiếp hoàn hảo.
- SFINAE.
- `concept` của C++ ở mức nâng cao.
- Đồng trình (`coroutine`).
- Bộ cấp phát tùy chỉnh nâng cao.
- Đa kế thừa phức tạp.
- Ngoại lệ internals.
- ABI C++ chuyên sâu.

### Câu hỏi phỏng vấn tự kiểm tra

1. Tham chiếu khác con trỏ như thế nào?
2. Tại sao nên dùng `nullptr` thay cho `NULL`?
3. `class` khác `struct` trong C++ ở điểm nào?
4. Hàm dựng và hàm hủy chạy khi nào?
5. Tại sao dùng danh sách khởi tạo thành viên?
6. Hàm thành viên `const` nghĩa là gì?
7. RAII giải quyết vấn đề gì?
8. Hàm dựng sao chép khác phép gán sao chép như thế nào?
9. Sao chép nông có thể gây lỗi gì với đối tượng sở hữu con trỏ?
10. Di chuyển khác sao chép như thế nào?
11. `std::move()` có tự di chuyển dữ liệu không?
12. `virtual` dùng để làm gì?
13. Khi nào hàm hủy của lớp cơ sở cần là `virtual`?
14. Hàm thuần ảo là gì?
15. Khuôn mẫu khác `virtual` về thời điểm xử lý như thế nào?
16. Tại sao `std::array` thường phù hợp với hệ thống nhúng hơn `std::vector` khi kích thước cố định?
17. `new/delete` khác `malloc/free` ở điểm quan trọng nào?
18. `unique_ptr` giải quyết vấn đề gì?
19. `static_cast` khác `reinterpret_cast` như thế nào?
20. `enum class` tốt hơn `enum` truyền thống ở điểm nào?
21. `constexpr` dùng để làm gì?
22. Tại sao cần `extern "C"` khi C++ gọi mã C?
23. Tại sao một số dự án nhúng không dùng ngoại lệ hoặc RTTI?
24. Những tính năng C++ nào có thể gây cấp phát động?
25. Khi dùng C++ trong phần mềm nhúng, tại sao cần quan tâm tới tính xác định của thời gian và bộ nhớ?

[↑ Về mục lục](#muc-luc)
