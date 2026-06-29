# 💳 Tài liệu Chi Tiết – Class `PaymentRepository`
### Restaurant Management System

---

## 📌 Mục lục
1. [Tổng quan](#1-tổng-quan)
2. [Kế thừa & Kiến trúc](#2-kế-thừa--kiến-trúc)
3. [Constructor](#3-constructor)
4. [Phương thức – Step-by-Step](#4-phương-thức--step-by-step)
   - 4.1 [findById()](#41-findbyidstring-id)
   - 4.2 [parseLine()](#42-parselinestring-line)
   - 4.3 [toLine()](#43-tolinepayment-payment)
   - 4.4 [findByInvoiceId()](#44-findbyinvoiceidstring-invoiceid)
5. [UML Class Diagram](#5-uml-class-diagram)
6. [Quan hệ với các Class khác](#6-quan-hệ-với-các-class-khác)
7. [Cấu trúc file lưu trữ](#7-cấu-trúc-file-lưu-trữ)
8. [So sánh với InvoiceRepository](#8-so-sánh-với-invoicerepository)
9. [Luồng xử lý điển hình](#9-luồng-xử-lý-điển-hình)
10. [Lưu ý & Điểm cần cải thiện](#10-lưu-ý--điểm-cần-cải-thiện)

---

## 1. Tổng quan

| Mục             | Chi tiết                                                                       |
|-----------------|--------------------------------------------------------------------------------|
| **Tên class**   | `PaymentRepository`                                                            |
| **Package**     | `com.mycompany.restaurantmanagement.repository`                                |
| **Kế thừa**     | `BaseRepository<Payment, String>`                                              |
| **Mục đích**    | Quản lý toàn bộ thao tác **đọc / ghi / tìm kiếm** dữ liệu giao dịch thanh toán từ file lưu trữ. Đây là tầng **Repository** chịu trách nhiệm persistence (lưu trữ bền vững) cho class `Payment`. |

### Import sử dụng

| Import                                       | Mục đích                                                  |
|----------------------------------------------|-----------------------------------------------------------|
| `AppConfig`                                  | Lấy hằng số đường dẫn file `PAYMENT_FILE_PATH`           |
| `Payment`                                    | Model đối tượng giao dịch thanh toán                      |
| `PaymentMethod`                              | Enum phương thức thanh toán (`CASH`, `QR`, ...)           |
| `PaymentStatus`                              | Enum trạng thái giao dịch (`PENDING`, `SUCCESS`, `CANCELLED`) |
| `java.util.Date`                             | Xử lý ngày giờ giao dịch khi parse từ timestamp          |

---

## 2. Kế thừa & Kiến trúc

```
BaseRepository<Payment, String>
           ▲
           │  extends
           │
  PaymentRepository
```

- `PaymentRepository` **kế thừa** `BaseRepository<T, ID>` với:
  - `T  = Payment` → kiểu đối tượng quản lý
  - `ID = String`  → kiểu khóa định danh (`paymentId`)
- Class cha `BaseRepository` cung cấp sẵn:
  - Thuộc tính `data` – danh sách `List<Payment>` chứa toàn bộ dữ liệu đã load từ file
  - Các phương thức CRUD chung: `save()`, `delete()`, `findAll()`, ...
  - Cơ chế đọc/ghi file thông qua 2 phương thức trừu tượng `parseLine()` và `toLine()`
- `PaymentRepository` **override** 3 phương thức bắt buộc và thêm 1 phương thức tùy chỉnh.

---

## 3. Constructor

```java
public PaymentRepository() {
    super(AppConfig.PAYMENT_FILE_PATH);
}
```

**Mục đích:** Khởi tạo repository và chỉ định file dữ liệu thanh toán.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  super(AppConfig.PAYMENT_FILE_PATH);
```
- Gọi constructor của class cha `BaseRepository`.
- `AppConfig.PAYMENT_FILE_PATH` là hằng số `String` chứa đường dẫn file.
  - Ví dụ: `"data/payments.txt"` hoặc `"src/main/resources/payments.dat"`
- Constructor cha thực hiện tuần tự:
  1. Lưu đường dẫn file vào thuộc tính nội bộ.
  2. Mở file và đọc từng dòng.
  3. Gọi `parseLine(line)` cho mỗi dòng → tạo đối tượng `Payment`.
  4. Nếu `parseLine()` trả về `null` (dòng lỗi) → bỏ qua dòng đó.
  5. Nạp tất cả đối tượng hợp lệ vào danh sách `data`.
- **Kết quả:** Sau constructor, `data` chứa đầy đủ danh sách giao dịch từ file, sẵn sàng để truy vấn.

---

## 4. Phương thức – Step-by-Step

---

### 4.1 `findById(String id)`

```java
@Override
public Payment findById(String id) {
    if (id == null) {
        return null;
    }
    for (Payment payment : data) {
        if (payment.getPaymentId().equals(id)) {
            return payment;
        }
    }
    return null;
}
```

**Mục đích:** Tìm và trả về giao dịch thanh toán theo `paymentId`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository`.  
**Điểm khác biệt so với `InvoiceRepository.findById()`:** Có thêm kiểm tra `null` cho tham số đầu vào.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  if (id == null) { return null; }
```
- **Null guard** – kiểm tra tham số truyền vào trước khi xử lý.
- Nếu `id == null` → trả về `null` ngay lập tức, không vào vòng lặp.
- Điều này ngăn `NullPointerException` ở bước so sánh `.equals(id)` bên dưới.
- Đây là **defensive programming** – lập trình phòng thủ, tốt hơn `InvoiceRepository`.

```
BƯỚC 2 ▶  for (Payment payment : data)
```
- Duyệt tuần tự toàn bộ danh sách `data` (kiểu `List<Payment>`).
- Mỗi vòng lặp, biến `payment` trỏ đến từng phần tử.
- **Độ phức tạp:** O(n) – tuyến tính.

```
BƯỚC 3 ▶  if (payment.getPaymentId().equals(id))
```
- Gọi `getPaymentId()` lấy mã giao dịch của phần tử hiện tại.
- Dùng `.equals()` so sánh nội dung chuỗi (không phải địa chỉ bộ nhớ như `==`).
- ⚠️ Nếu `payment.getPaymentId()` là `null` → vẫn có thể `NullPointerException`.
  - An toàn hơn nếu dùng: `id.equals(payment.getPaymentId())` vì `id` đã được kiểm tra null ở Bước 1.

```
BƯỚC 4a ▶  return payment;   (nếu tìm thấy)
```
- Dừng vòng lặp ngay, trả về đối tượng `Payment` khớp với `id`.

```
BƯỚC 4b ▶  return null;   (sau vòng lặp, không tìm thấy)
```
- Không có giao dịch nào khớp → trả về `null`.

#### 📋 Ví dụ:
```java
Payment p = repo.findById("PAY001");
if (p != null) {
    p.recordTransaction();
} else {
    System.out.println("Không tìm thấy giao dịch PAY001");
}
```

---

### 4.2 `parseLine(String line)`

```java
@Override
protected Payment parseLine(String line) {
    try {
        String[] d = line.split("\\|");

        if (d.length < 6) {
            return null;
        }

        Payment payment = new Payment(
                d[0],
                d[1],
                Double.parseDouble(d[2]),
                new Date(Long.parseLong(d[3])),
                PaymentMethod.valueOf(d[5])
        );

        payment.setStatus(PaymentStatus.valueOf(d[4]));

        return payment;

    } catch (Exception e) {
        return null;
    }
}
```

**Mục đích:** Chuyển đổi một dòng văn bản từ file thành đối tượng `Payment`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository`.  
**Đặc điểm nổi bật:** Toàn bộ logic được bọc trong `try-catch` – nếu có lỗi bất kỳ, trả về `null` thay vì crash.

#### 📄 Cấu trúc dòng dữ liệu trong file:
```
PAY001|INV001|250000.0|1751082600000|SUCCESS|CASH
 [0]    [1]      [2]       [3]          [4]    [5]
```

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  try { ... } catch (Exception e) { return null; }
```
- Bao toàn bộ logic trong khối `try`.
- Bất kỳ exception nào xảy ra bên trong (`NumberFormatException`,
  `IllegalArgumentException`, `ArrayIndexOutOfBoundsException`, ...) đều bị bắt.
- **Catch block:** Trả về `null` thay vì để chương trình crash.
- **Ưu điểm:** Dòng dữ liệu lỗi bị bỏ qua, các dòng còn lại vẫn load bình thường.
- **Nhược điểm:** Exception bị nuốt hoàn toàn – không biết nguyên nhân lỗi là gì.

```
BƯỚC 2 ▶  String[] d = line.split("\\|");
```
- `"\\|"` là regex tách theo ký tự `|` (phải escape vì `|` là ký tự đặc biệt trong regex).
- **Ví dụ:**
  ```
  Input:  "PAY001|INV001|250000.0|1751082600000|SUCCESS|CASH"
  Output: d = ["PAY001","INV001","250000.0","1751082600000","SUCCESS","CASH"]
  ```

```
BƯỚC 3 ▶  if (d.length < 6) { return null; }
```
- Kiểm tra mảng có đủ 6 phần tử không (index 0 đến 5).
- Nếu dòng file thiếu cột (VD: dòng trống, dữ liệu bị cắt xén) → trả về `null` sớm.
- Đây là **validation đầu vào** quan trọng, ngăn `ArrayIndexOutOfBoundsException`.

```
BƯỚC 4 ▶  Payment payment = new Payment(
               d[0],                          // paymentId
               d[1],                          // invoiceId
               Double.parseDouble(d[2]),      // amount
               new Date(Long.parseLong(d[3])),// paymentDate
               PaymentMethod.valueOf(d[5])    // method
           );
```
- Gọi constructor 5 tham số của `Payment`.
- Từng tham số được xử lý:

  | Tham số                         | Giá trị mẫu           | Xử lý                                                    |
  |---------------------------------|-----------------------|----------------------------------------------------------|
  | `d[0]`                          | `"PAY001"`            | Gán trực tiếp làm `paymentId`                            |
  | `d[1]`                          | `"INV001"`            | Gán trực tiếp làm `invoiceId`                            |
  | `Double.parseDouble(d[2])`      | `"250000.0"` → `250000.0` | Chuyển chuỗi → số thực Double                        |
  | `new Date(Long.parseLong(d[3]))`| `"1751082600000"` → `Date` | **Timestamp Unix (milliseconds)** → đối tượng `Date` |
  | `PaymentMethod.valueOf(d[5])`   | `"CASH"` → `CASH`     | Chuyển chuỗi → hằng số enum `PaymentMethod`              |

- **⚡ Điểm quan trọng – Timestamp Unix:**
  - `Long.parseLong("1751082600000")` → số `1751082600000L` (milliseconds kể từ 01/01/1970 UTC).
  - `new Date(1751082600000L)` → đối tượng `Date` tương ứng với thời điểm đó.
  - Khác với `InvoiceRepository` dùng `"yyyy-MM-dd"`, `PaymentRepository` lưu ngày dạng **timestamp số** – chính xác đến millisecond, không mất thông tin giờ/phút/giây.

- ⚠️ `PaymentMethod.valueOf(d[5])` gây `IllegalArgumentException` nếu `d[5]` không khớp tên enum → bị bắt bởi `catch`.

- **Lưu ý index `d[5]` cho `method`:** Constructor nhận `method` là tham số cuối, nhưng trong file `method` ở vị trí index `[5]`, còn `status` ở `[4]`. Thứ tự file và thứ tự constructor **không giống nhau** – cần chú ý khi đọc code.

```
BƯỚC 5 ▶  payment.setStatus(PaymentStatus.valueOf(d[4]));
```
- Constructor `Payment(...)` luôn tự đặt `status = PENDING` (mặc định).
- Dòng này **ghi đè** lại bằng giá trị thực từ file (`d[4]`).
- `PaymentStatus.valueOf("SUCCESS")` → enum `PaymentStatus.SUCCESS`.
- ⚠️ Nếu `d[4]` không hợp lệ → `IllegalArgumentException` → bị `catch` bắt → `return null`.

```
BƯỚC 6 ▶  return payment;
```
- Trả về đối tượng `Payment` đã được gán đầy đủ thuộc tính.

---

### 4.3 `toLine(Payment payment)`

```java
@Override
protected String toLine(Payment payment) {
    return payment.getPaymentId()
            + "|"
            + payment.getInvoiceId()
            + "|"
            + payment.getAmount()
            + "|"
            + payment.getPaymentDate().getTime()
            + "|"
            + payment.getStatus()
            + "|"
            + payment.getMethod();
}
```

**Mục đích:** Chuyển đổi đối tượng `Payment` thành dòng văn bản để ghi vào file. Đây là **nghịch đảo** của `parseLine()`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository`.

#### 🔢 Chạy từng dòng (nối chuỗi):

```
BƯỚC 1 ▶  payment.getPaymentId()
```
- Lấy mã giao dịch. **Ví dụ:** `"PAY001"`

```
BƯỚC 2 ▶  + "|" + payment.getInvoiceId()
```
- Nối ký tự phân cách `|` và mã hóa đơn. **Ví dụ:** `"|INV001"`

```
BƯỚC 3 ▶  + "|" + payment.getAmount()
```
- `Double` tự động chuyển thành `String` khi nối `+`. **Ví dụ:** `"|250000.0"`

```
BƯỚC 4 ▶  + "|" + payment.getPaymentDate().getTime()
```
- **Điểm then chốt:** `.getTime()` trả về số `long` – milliseconds kể từ epoch (01/01/1970 UTC).
- Lưu ngày dưới dạng số nguyên thay vì chuỗi text → chính xác tuyệt đối, không phụ thuộc locale hay timezone.
- ⚠️ Nếu `paymentDate == null` → `NullPointerException` (không có null-check).
- **Ví dụ:** `Date` lúc `28/06/2026 10:30:00` → `"|1751082600000"`

```
BƯỚC 5 ▶  + "|" + payment.getStatus()
```
- `PaymentStatus` enum tự gọi `.toString()` khi nối chuỗi.
- **Ví dụ:** `"|SUCCESS"`

```
BƯỚC 6 ▶  + "|" + payment.getMethod()
```
- `PaymentMethod` enum tự gọi `.toString()`.
- **Ví dụ:** `"|CASH"`

#### 📋 Kết quả đầy đủ:
```
PAY001|INV001|250000.0|1751082600000|SUCCESS|CASH
```

> **💡 Lưu ý thứ tự:** Trong file, `status` (index 4) đứng **trước** `method` (index 5).  
> Trong constructor `Payment(...)`, `method` là tham số cuối và không có `status`.  
> Đây là lý do `parseLine()` phải gọi thêm `payment.setStatus(...)` riêng sau khi tạo object.

---

### 4.4 `findByInvoiceId(String invoiceId)`

```java
public Payment findByInvoiceId(String invoiceId) {
    if (invoiceId == null) {
        return null;
    }
    for (Payment payment : data) {
        if (invoiceId.equals(payment.getInvoiceId())) {
            return payment;
        }
    }
    return null;
}
```

**Mục đích:** Tìm giao dịch thanh toán theo `invoiceId` – biết hóa đơn, tìm giao dịch tương ứng.  
**Lưu ý kỹ thuật:** Hàm này dùng `invoiceId.equals(payment.getInvoiceId())` thay vì `payment.getInvoiceId().equals(invoiceId)` – đây là **cách viết an toàn hơn**.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  if (invoiceId == null) { return null; }
```
- Null guard giống `findById()` – kiểm tra tham số đầu vào trước.
- Nếu `invoiceId == null` → trả về `null` ngay, không vào vòng lặp.

```
BƯỚC 2 ▶  for (Payment payment : data)
```
- Duyệt tuần tự toàn bộ danh sách `data`.

```
BƯỚC 3 ▶  if (invoiceId.equals(payment.getInvoiceId()))
```
- **Kỹ thuật "Yoda condition" (đảo ngược vế so sánh):**
  - `invoiceId.equals(payment.getInvoiceId())` – gọi `.equals()` trên biến **đã kiểm tra null** (`invoiceId`).
  - Ngay cả khi `payment.getInvoiceId()` trả về `null`, hàm `equals()` xử lý được (trả về `false`).
  - Nếu viết ngược lại `payment.getInvoiceId().equals(invoiceId)` thì sẽ crash nếu `getInvoiceId()` trả về `null`.
- So sánh `invoiceId` của đối tượng đang duyệt với tham số truyền vào.

```
BƯỚC 4a ▶  return payment;   (nếu tìm thấy)
```
- Trả về giao dịch đầu tiên có `invoiceId` khớp.
- Giả định mỗi hóa đơn chỉ có **một** giao dịch thanh toán.

```
BƯỚC 4b ▶  return null;   (sau vòng lặp)
```
- Không tìm thấy giao dịch nào khớp.

#### 📋 So sánh hai cách viết `.equals()`:

| Cách viết                                      | Khi `getInvoiceId()` = null | An toàn? |
|------------------------------------------------|-----------------------------|----------|
| `invoiceId.equals(payment.getInvoiceId())`     | Trả về `false`              | ✅ An toàn |
| `payment.getInvoiceId().equals(invoiceId)`     | `NullPointerException`      | ❌ Nguy hiểm |

#### 📋 Ví dụ:
```java
Payment p = repo.findByInvoiceId("INV001");
if (p != null) {
    System.out.println("Giao dịch: " + p.getPaymentId());
    System.out.println("Trạng thái: " + p.getPaymentStatus());
}
```

---

## 5. UML Class Diagram

```
┌────────────────────────────────────────────────────────────────────┐
│             BaseRepository<Payment, String>                        │
│                      <<abstract>>                                  │
├────────────────────────────────────────────────────────────────────┤
│ # data      : List<Payment>                                        │
│ # filePath  : String                                               │
├────────────────────────────────────────────────────────────────────┤
│ + findAll() : List<Payment>                                        │
│ + save(t)   : void                                                 │
│ + delete(id): void                                                 │
│ + findById(id)       : Payment    ← abstract                      │
│ # parseLine(line)    : Payment    ← abstract                      │
│ # toLine(t)          : String     ← abstract                      │
└────────────────────────────────────────────────────────────────────┘
                          ▲
                          │  extends
                          │
┌────────────────────────────────────────────────────────────────────┐
│                     PaymentRepository                              │
├────────────────────────────────────────────────────────────────────┤
│   (không có thuộc tính mới – dùng data từ BaseRepository)         │
├────────────────────────────────────────────────────────────────────┤
│ + PaymentRepository()                                              │
├────────────────────────────────────────────────────────────────────┤
│ + findById(id: String)              : Payment    @Override         │
│ # parseLine(line: String)           : Payment    @Override         │
│ # toLine(payment: Payment)          : String     @Override         │
│ + findByInvoiceId(invoiceId: String): Payment                      │
└────────────────────────────────────────────────────────────────────┘
          │                      │
          │ uses                  │ uses
          ▼                      ▼
   ┌────────────┐        ┌───────────────┐
   │  Payment   │        │   AppConfig   │
   │  (model)   │        │ PAYMENT_FILE  │
   └────────────┘        │    _PATH      │
          │              └───────────────┘
    ┌─────┴──────┐
    ▼            ▼
┌──────────┐  ┌───────────────┐
│ Payment  │  │ PaymentStatus │
│  Method  │  │   <<enum>>    │
│ <<enum>> │  ├───────────────┤
├──────────┤  │ PENDING       │
│ CASH     │  │ SUCCESS       │
│ QR       │  │ CANCELLED     │
└──────────┘  └───────────────┘
```

---

## 6. Quan hệ với các Class khác

| Class / Interface  | Loại quan hệ | Mô tả                                                                           |
|--------------------|--------------|---------------------------------------------------------------------------------|
| `BaseRepository`   | Inheritance  | Kế thừa cơ sở hạ tầng đọc/ghi file, danh sách `data`, các method CRUD chung   |
| `Payment`          | Dependency   | Kiểu dữ liệu chính; tạo, đọc, trả về trong mọi phương thức                    |
| `PaymentMethod`    | Dependency   | Dùng `valueOf()` để chuyển chuỗi từ file thành enum phương thức thanh toán     |
| `PaymentStatus`    | Dependency   | Dùng `valueOf()` để chuyển chuỗi từ file thành enum trạng thái                 |
| `AppConfig`        | Dependency   | Cung cấp hằng số đường dẫn file `PAYMENT_FILE_PATH`                            |

---

## 7. Cấu trúc file lưu trữ

File text lưu trữ giao dịch có định dạng **pipe-separated** (`|`), mỗi dòng là một giao dịch:

```
paymentId | invoiceId | amount    | paymentDate   | status  | method
──────────────────────────────────────────────────────────────────────
PAY001    | INV001    | 250000.0  | 1751082600000 | SUCCESS | CASH
PAY002    | INV002    | 180000.0  | 1751082700000 | PENDING | QR
PAY003    | INV003    | 320000.5  | 1751082800000 | CANCELLED | CASH
```

### Quy tắc từng cột:

| Index  | Trường        | Kiểu trong file | Kiểu Java sau parse         | Ghi chú                                         |
|--------|---------------|-----------------|-----------------------------|--------------------------------------------------|
| `d[0]` | `paymentId`   | String          | `String`                    | Mã định danh duy nhất                           |
| `d[1]` | `invoiceId`   | String          | `String`                    | Mã hóa đơn liên kết                             |
| `d[2]` | `amount`      | Số thực         | `Double`                    | Phải là số hợp lệ                               |
| `d[3]` | `paymentDate` | Số nguyên       | `Date` (qua timestamp ms)   | Milliseconds kể từ epoch – chính xác đến ms     |
| `d[4]` | `status`      | String          | `PaymentStatus` (enum)      | Phải khớp tên hằng số enum                     |
| `d[5]` | `method`      | String          | `PaymentMethod` (enum)      | Phải khớp tên hằng số enum                     |

### ⚡ Tại sao dùng Timestamp thay vì chuỗi ngày?

| Tiêu chí           | Timestamp (`long`)            | Chuỗi ngày (`"yyyy-MM-dd"`)       |
|--------------------|-------------------------------|-----------------------------------|
| Độ chính xác       | Đến millisecond               | Chỉ đến ngày                      |
| Parse              | `new Date(long)` – không lỗi | `SimpleDateFormat.parse()` – có thể lỗi format |
| Locale/Timezone    | Không phụ thuộc               | Phụ thuộc locale hệ thống         |
| Dung lượng file    | Nhiều ký tự hơn               | Ít ký tự hơn                      |
| Đọc bằng mắt       | Khó đọc                       | Dễ đọc                            |

---

## 8. So sánh với `InvoiceRepository`

| Tiêu chí                       | `PaymentRepository`               | `InvoiceRepository`                    |
|--------------------------------|-----------------------------------|----------------------------------------|
| Null guard trong `findById()`  | ✅ Có (`if id == null`)           | ❌ Không có                            |
| Bọc `parseLine()` trong try-catch | ✅ Có – toàn bộ logic           | ❌ Không – từng phần riêng lẻ          |
| Kiểm tra `d.length`            | ✅ Có (`d.length < 6`)           | ❌ Không có                            |
| Lưu ngày giờ                   | Timestamp ms (`long`)             | Chuỗi `"yyyy-MM-dd"`                  |
| Cách gọi `.equals()`           | `invoiceId.equals(getInvoiceId())` – an toàn | `getInvoiceId().equals(id)` – có thể NPE |
| Tạo đối tượng trong parseLine  | Constructor 5 tham số             | Constructor mặc định + setters        |

> **Kết luận:** `PaymentRepository` có **chất lượng code cao hơn** `InvoiceRepository` về mặt xử lý lỗi và null safety.

---

## 9. Luồng xử lý điển hình

```
┌──────────────────────────────────────────────────────────────────────┐
│                    KHỞI TẠO & ĐỌC DỮ LIỆU TỪ FILE                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  new PaymentRepository()                                             │
│        │                                                             │
│        ▼                                                             │
│  super(AppConfig.PAYMENT_FILE_PATH)                                  │
│        │                                                             │
│        ▼                                                             │
│  BaseRepository đọc file từng dòng                                  │
│        │                                                             │
│        ▼                                                             │
│  parseLine("PAY001|INV001|250000.0|1751082600000|SUCCESS|CASH")      │
│        │  → split("|")    → d[0..5]                                  │
│        │  → d.length >= 6 → OK                                       │
│        │  → new Payment(d[0], d[1], parseDouble(d[2]),               │
│        │                new Date(parseLong(d[3])), valueOf(d[5]))    │
│        │  → setStatus(valueOf(d[4]))                                 │
│        │  → return Payment object                                    │
│        ▼                                                             │
│  data = [Payment1, Payment2, ...]   ← sẵn sàng truy vấn            │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                       TÌM KIẾM & SỬ DỤNG                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  findById("PAY001")          → null guard → duyệt data → trả về    │
│  findByInvoiceId("INV001")   → null guard → duyệt data → trả về    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                      GHI DỮ LIỆU XUỐNG FILE                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BaseRepository.save(payment)                                        │
│        │                                                             │
│        ▼                                                             │
│  toLine(payment)                                                     │
│        │  → nối các thuộc tính bằng "|"                             │
│        │  → paymentDate.getTime() → timestamp ms                    │
│        │  → "PAY001|INV001|250000.0|1751082600000|SUCCESS|CASH"     │
│        ▼                                                             │
│  Ghi chuỗi vào file                                                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 10. Lưu ý & Điểm cần cải thiện

| # | Vấn đề                                                                                              | Khuyến nghị                                                                              |
|---|-----------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| 1 | `parseLine()`: `catch (Exception e)` nuốt hoàn toàn exception, không log nguyên nhân lỗi           | Thêm `System.err.println("parseLine error: " + e.getMessage())` hoặc dùng Logger        |
| 2 | `toLine()`: Không kiểm tra `paymentDate == null` trước khi `.getTime()`                            | Thêm: `(payment.getPaymentDate() != null ? payment.getPaymentDate().getTime() : 0)`     |
| 3 | `toLine()`: Không kiểm tra các trường khác có thể `null` (`paymentId`, `status`, `method`)         | Thêm null-check hoặc bọc trong `try-catch`                                              |
| 4 | `findById()`: `payment.getPaymentId().equals(id)` vẫn có thể NPE nếu `getPaymentId()` là `null`   | Đổi thành `id.equals(payment.getPaymentId())` (giống cách `findByInvoiceId` đã làm đúng)|
| 5 | Tất cả `find*()` đều O(n) – duyệt toàn bộ danh sách                                               | Dùng `HashMap<String, Payment>` để tìm O(1) khi dữ liệu lớn                            |
| 6 | `parseLine()` dùng `d[5]` cho `method` nhưng file có `status` ở `d[4]`, `method` ở `d[5]`         | Thêm comment rõ ràng về thứ tự index để tránh nhầm khi bảo trì sau này                 |

---

*📝 Tài liệu được tạo từ source code `PaymentRepository.java` – Restaurant Management System.*  
*🗓️ Ngày tạo: 28/06/2026*
