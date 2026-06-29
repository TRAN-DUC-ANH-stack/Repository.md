# 📂 Tài liệu Chi Tiết – Class `InvoiceRepository`
### Restaurant Management System

---

## 📌 Mục lục
1. [Tổng quan](#1-tổng-quan)
2. [Kế thừa & Kiến trúc](#2-kế-thừa--kiến-trúc)
3. [Constructor](#3-constructor)
4. [Phương thức – Step-by-Step](#4-phương-thức--step-by-step)
   - 4.1 [findById()](#41-findbyidstring-id)
   - 4.2 [parseLine()](#42-parselinestring-line)
   - 4.3 [toLine()](#43-tolineinvoice-invoice)
   - 4.4 [findByOrderId()](#44-findbyorderidstring-orderid)
   - 4.5 [findByDateRange()](#45-findbydaterangedate-from-date-to)
5. [UML Class Diagram](#5-uml-class-diagram)
6. [Quan hệ với các Class khác](#6-quan-hệ-với-các-class-khác)
7. [Cấu trúc file lưu trữ](#7-cấu-trúc-file-lưu-trữ)
8. [Luồng xử lý điển hình](#8-luồng-xử-lý-điển-hình)
9. [Lưu ý & Điểm cần cải thiện](#9-lưu-ý--điểm-cần-cải-thiện)

---

## 1. Tổng quan

| Mục              | Chi tiết                                                           |
|------------------|--------------------------------------------------------------------|
| **Tên class**    | `InvoiceRepository`                                                |
| **Package**      | `com.mycompany.restaurantmanagement.repository`                    |
| **Kế thừa**      | `BaseRepository<Invoice, String>`                                  |
| **Mục đích**     | Quản lý toàn bộ thao tác **đọc / ghi / tìm kiếm** dữ liệu hóa đơn từ file lưu trữ. Đây là tầng **Repository** trong mô hình phân tầng (Layered Architecture), tách biệt logic truy cập dữ liệu khỏi logic nghiệp vụ. |

### Import sử dụng

| Import                                    | Mục đích                                              |
|-------------------------------------------|-------------------------------------------------------|
| `AppConfig`                               | Lấy đường dẫn file hóa đơn (`INVOICE_FILE_PATH`)     |
| `Invoice`                                 | Model đối tượng hóa đơn                               |
| `InvoiceStatus`                           | Enum trạng thái hóa đơn                               |
| `Order`                                   | Model đơn hàng (dùng khi parse dòng dữ liệu)          |
| `java.util.ArrayList`                     | Tạo danh sách kết quả trả về                          |
| `java.util.Date`                          | Xử lý ngày tháng trong tìm kiếm theo khoảng thời gian |
| `java.util.List`                          | Kiểu trả về danh sách                                 |

---

## 2. Kế thừa & Kiến trúc

```
BaseRepository<Invoice, String>
         ▲
         │  extends
         │
InvoiceRepository
```

- `InvoiceRepository` **kế thừa** `BaseRepository<T, ID>` với:
  - `T = Invoice` → kiểu đối tượng quản lý
  - `ID = String` → kiểu khóa định danh (mã hóa đơn)
- Class cha `BaseRepository` cung cấp sẵn:
  - Thuộc tính `data` – danh sách `List<Invoice>` chứa toàn bộ dữ liệu đã load
  - Các phương thức CRUD chung (`save`, `delete`, `findAll`, ...)
  - Cơ chế đọc/ghi file thông qua `parseLine()` và `toLine()`
- `InvoiceRepository` **override** các phương thức trừu tượng để định nghĩa cách parse/serialize riêng cho `Invoice`.

---

## 3. Constructor

```java
public InvoiceRepository() {
    super(AppConfig.INVOICE_FILE_PATH);
}
```

**Mục đích:** Khởi tạo repository và chỉ định file dữ liệu.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  super(AppConfig.INVOICE_FILE_PATH);
```
- Gọi constructor của class cha `BaseRepository`.
- `AppConfig.INVOICE_FILE_PATH` là một hằng số `String` chứa đường dẫn file lưu trữ hóa đơn.
  - Ví dụ: `"data/invoices.txt"` hoặc `"src/main/resources/invoices.dat"`
- Constructor cha sẽ thực hiện:
  - Lưu đường dẫn file vào thuộc tính nội bộ.
  - Đọc toàn bộ file, gọi `parseLine()` cho từng dòng để tạo đối tượng `Invoice`.
  - Nạp tất cả đối tượng vào danh sách `data`.
- **Kết quả:** Sau khi constructor chạy xong, `data` đã chứa đầy đủ danh sách hóa đơn từ file.

---

## 4. Phương thức – Step-by-Step

---

### 4.1 `findById(String id)`

```java
@Override
public Invoice findById(String id) {
    for (Invoice invoice : data) {
        if (invoice.getInvoiceId().equals(id)) {
            return invoice;
        }
    }
    return null;
}
```

**Mục đích:** Tìm và trả về hóa đơn theo mã `invoiceId`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository`.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  for (Invoice invoice : data)
```
- Duyệt tuần tự qua toàn bộ danh sách `data` (kiểu `List<Invoice>`).
- Mỗi vòng lặp, biến `invoice` lần lượt trỏ đến từng phần tử trong danh sách.
- **Độ phức tạp:** O(n) – tuyến tính theo số lượng hóa đơn.

```
BƯỚC 2 ▶  if (invoice.getInvoiceId().equals(id))
```
- Gọi `getInvoiceId()` lấy mã hóa đơn của phần tử hiện tại.
- Dùng `.equals()` (không phải `==`) để so sánh nội dung chuỗi.
- ⚠️ Nếu `invoice.getInvoiceId()` trả về `null` → `NullPointerException`.

```
BƯỚC 3a ▶  return invoice;   (nếu tìm thấy)
```
- Dừng vòng lặp ngay lập tức và trả về đối tượng `Invoice` khớp.
- Không tiếp tục duyệt các phần tử còn lại.

```
BƯỚC 3b ▶  return null;   (sau khi vòng lặp kết thúc mà không tìm thấy)
```
- Trả về `null` báo hiệu không có hóa đơn nào khớp với `id`.

#### 📋 Ví dụ:
```java
Invoice inv = repo.findById("INV001");
if (inv != null) {
    inv.printInvoice();
} else {
    System.out.println("Không tìm thấy hóa đơn INV001");
}
```

---

### 4.2 `parseLine(String line)`

```java
@Override
protected Invoice parseLine(String line) {
    String[] d = line.split("\\|");

    Invoice invoice = new Invoice();

    invoice.setInvoiceId(d[0]);

    Order order = new Order(d[1], null);
    invoice.setOrder(order);

    invoice.setTotalAmount(Double.parseDouble(d[2]));

    try {
        invoice.setIssueDate(
            new java.text.SimpleDateFormat("yyyy-MM-dd").parse(d[3])
        );
    } catch (Exception e) {
        invoice.setIssueDate(new Date());
    }

    invoice.setStatus(InvoiceStatus.valueOf(d[4]));

    return invoice;
}
```

**Mục đích:** Chuyển đổi một dòng văn bản từ file thành đối tượng `Invoice`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository` – đây là trái tim của cơ chế đọc file.

#### 📄 Cấu trúc dòng dữ liệu trong file:
```
INV001|ORD001|250000.0|2026-06-28|UNPAID
  [0]    [1]     [2]      [3]       [4]
```

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  String[] d = line.split("\\|");
```
- `"\\|"` là regex để tách theo ký tự `|` (phải escape vì `|` là ký tự đặc biệt trong regex).
- Kết quả: mảng `d` chứa từng trường dữ liệu.
- **Ví dụ:**
  ```
  Input:  "INV001|ORD001|250000.0|2026-06-28|UNPAID"
  Output: d = ["INV001", "ORD001", "250000.0", "2026-06-28", "UNPAID"]
  ```

```
BƯỚC 2 ▶  Invoice invoice = new Invoice();
```
- Tạo đối tượng `Invoice` rỗng bằng constructor mặc định.
- Sẽ gán từng thuộc tính thủ công ở các bước tiếp theo.

```
BƯỚC 3 ▶  invoice.setInvoiceId(d[0]);
```
- Gán `d[0]` = `"INV001"` làm mã hóa đơn.

```
BƯỚC 4 ▶  Order order = new Order(d[1], null);
           invoice.setOrder(order);
```
- Tạo đối tượng `Order` tạm với chỉ `orderId = d[1]` (VD: `"ORD001"`), tham số thứ 2 là `null`.
- ⚠️ **Lưu ý quan trọng:** Đây chỉ là đối tượng `Order` **thiếu thông tin** (chỉ có ID, không có items, bàn, nhân viên,...). Dữ liệu đầy đủ của `Order` phải được load riêng từ `OrderRepository`.
- Gán vào hóa đơn để duy trì mối liên kết.

```
BƯỚC 5 ▶  invoice.setTotalAmount(Double.parseDouble(d[2]));
```
- `Double.parseDouble("250000.0")` → chuyển chuỗi `"250000.0"` thành số `250000.0`.
- ⚠️ Nếu `d[2]` không phải định dạng số hợp lệ → `NumberFormatException`.

```
BƯỚC 6 ▶  try {
               invoice.setIssueDate(
                   new java.text.SimpleDateFormat("yyyy-MM-dd").parse(d[3])
               );
           } catch (Exception e) {
               invoice.setIssueDate(new Date());
           }
```
- **Try block:**
  - Tạo `SimpleDateFormat` với pattern `"yyyy-MM-dd"` để đọc ngày dạng `"2026-06-28"`.
  - `.parse(d[3])` chuyển chuỗi ngày thành đối tượng `Date`.
  - Gán ngày đã parse vào `issueDate`.
- **Catch block:**
  - Nếu `d[3]` không đúng format (VD: `"28/06/2026"`, `""`, `null`) → `ParseException`.
  - Fallback: gán `issueDate = new Date()` (ngày hiện tại) thay vì crash chương trình.

```
BƯỚC 7 ▶  invoice.setStatus(InvoiceStatus.valueOf(d[4]));
```
- `InvoiceStatus.valueOf("UNPAID")` → chuyển chuỗi thành hằng số enum `InvoiceStatus.UNPAID`.
- ⚠️ Nếu `d[4]` không khớp với bất kỳ giá trị enum nào (VD: `"unpaid"` viết thường) → `IllegalArgumentException`.

```
BƯỚC 8 ▶  return invoice;
```
- Trả về đối tượng `Invoice` đã được gán đầy đủ thuộc tính từ dòng file.

---

### 4.3 `toLine(Invoice invoice)`

```java
@Override
protected String toLine(Invoice invoice) {
    return invoice.getInvoiceId()
        + "|"
        + (invoice.getOrder() != null ? invoice.getOrder().getOrderId() : "null")
        + "|"
        + invoice.getTotalAmount()
        + "|"
        + new java.text.SimpleDateFormat("yyyy-MM-dd").format(invoice.getIssueDate())
        + "|"
        + invoice.getStatus();
}
```

**Mục đích:** Chuyển đổi đối tượng `Invoice` thành một dòng văn bản để ghi vào file. Đây là **nghịch đảo** của `parseLine()`.  
**`@Override`:** Ghi đè phương thức trừu tượng từ `BaseRepository`.

#### 🔢 Chạy từng dòng (nối chuỗi):

```
BƯỚC 1 ▶  invoice.getInvoiceId()
```
- Lấy mã hóa đơn. **Ví dụ:** `"INV001"`

```
BƯỚC 2 ▶  + "|" +
```
- Nối ký tự phân tách `|`.

```
BƯỚC 3 ▶  (invoice.getOrder() != null ? invoice.getOrder().getOrderId() : "null")
```
- **Toán tử 3 ngôi (ternary operator):**
  - Nếu `order != null` → lấy `orderId` của order. **Ví dụ:** `"ORD001"`
  - Nếu `order == null` → ghi chuỗi `"null"` để tránh lỗi `NullPointerException`.

```
BƯỚC 4 ▶  + "|" + invoice.getTotalAmount()
```
- Nối tổng tiền. `Double` tự động chuyển thành `String`. **Ví dụ:** `"250000.0"`

```
BƯỚC 5 ▶  + "|" + new SimpleDateFormat("yyyy-MM-dd").format(invoice.getIssueDate())
```
- Tạo `SimpleDateFormat` với pattern `"yyyy-MM-dd"`.
- `.format(Date)` chuyển đối tượng `Date` → chuỗi ngày theo format.
- ⚠️ Nếu `issueDate == null` → `NullPointerException`.
- **Ví dụ:** `Date("Sun Jun 28 2026")` → `"2026-06-28"`

```
BƯỚC 6 ▶  + "|" + invoice.getStatus()
```
- `InvoiceStatus` enum tự động gọi `toString()` khi nối chuỗi. **Ví dụ:** `"UNPAID"`

#### 📋 Kết quả đầy đủ:
```
INV001|ORD001|250000.0|2026-06-28|UNPAID
```

---

### 4.4 `findByOrderId(String orderId)`

```java
public Invoice findByOrderId(String orderId) {
    for (Invoice invoice : data) {
        if (invoice.getOrder() != null
                && invoice.getOrder().getOrderId().equals(orderId)) {
            return invoice;
        }
    }
    return null;
}
```

**Mục đích:** Tìm hóa đơn theo `orderId` – biết đơn hàng, tìm hóa đơn tương ứng.

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  for (Invoice invoice : data)
```
- Duyệt tuần tự toàn bộ danh sách hóa đơn trong `data`.

```
BƯỚC 2 ▶  if (invoice.getOrder() != null
                && invoice.getOrder().getOrderId().equals(orderId))
```
- **Điều kiện kép với `&&` (AND):**

  - **Vế trái:** `invoice.getOrder() != null`
    - Kiểm tra null trước để tránh `NullPointerException` ở vế phải.
    - Java dùng **short-circuit evaluation**: nếu vế trái `false` → không chạy vế phải.

  - **Vế phải:** `invoice.getOrder().getOrderId().equals(orderId)`
    - Lấy `Order` từ hóa đơn → lấy `orderId` của Order → so sánh với tham số.
    - Chỉ chạy khi `order != null` (nhờ vế trái đã kiểm tra).

```
BƯỚC 3a ▶  return invoice;   (nếu khớp)
```
- Dừng vòng lặp, trả về hóa đơn đầu tiên có `orderId` khớp.
- ⚠️ Mỗi đơn hàng thường chỉ có **1 hóa đơn**, nên trả về phần tử đầu tiên là hợp lý.

```
BƯỚC 3b ▶  return null;   (nếu không tìm thấy)
```
- Không có hóa đơn nào thuộc `orderId` đó.

#### 📋 Ví dụ:
```java
Invoice inv = repo.findByOrderId("ORD001");
// Tìm hóa đơn của đơn hàng ORD001
```

---

### 4.5 `findByDateRange(Date from, Date to)`

```java
public List<Invoice> findByDateRange(Date from, Date to) {
    List<Invoice> result = new ArrayList<>();

    for (Invoice invoice : data) {
        Date date = invoice.getIssueDate();

        if (
            date != null
            && !date.before(from)
            && !date.after(to)
        ) {
            result.add(invoice);
        }
    }

    return result;
}
```

**Mục đích:** Trả về danh sách tất cả hóa đơn có `issueDate` nằm trong khoảng `[from, to]` (bao gồm cả hai đầu).

#### 🔢 Chạy từng dòng:

```
BƯỚC 1 ▶  List<Invoice> result = new ArrayList<>();
```
- Tạo danh sách rỗng để chứa các hóa đơn thỏa điều kiện.
- Dùng `ArrayList` vì số lượng kết quả chưa biết trước, `ArrayList` tự mở rộng động.

```
BƯỚC 2 ▶  for (Invoice invoice : data)
```
- Duyệt toàn bộ danh sách hóa đơn.

```
BƯỚC 3 ▶  Date date = invoice.getIssueDate();
```
- Lấy ngày xuất hóa đơn vào biến cục bộ `date` để dùng nhiều lần (tránh gọi getter lặp lại).

```
BƯỚC 4 ▶  if (date != null && !date.before(from) && !date.after(to))
```
- **Điều kiện 3 vế với `&&`:**

  | Vế                    | Ý nghĩa                              | Kết quả nếu FALSE |
  |-----------------------|--------------------------------------|-------------------|
  | `date != null`        | Hóa đơn phải có ngày                 | Bỏ qua hóa đơn   |
  | `!date.before(from)`  | `date >= from` (không trước `from`)  | Bỏ qua hóa đơn   |
  | `!date.after(to)`     | `date <= to` (không sau `to`)        | Bỏ qua hóa đơn   |

- **Tại sao dùng `!before` và `!after` thay vì `>=` và `<=`?**
  - `Date` không hỗ trợ toán tử `>=`, `<=` trực tiếp.
  - `date.before(from)` = `date < from` → `!date.before(from)` = `date >= from`
  - `date.after(to)` = `date > to` → `!date.after(to)` = `date <= to`
  - Cả 3 vế đảm bảo: `from <= date <= to`

- **Ví dụ minh họa:**
  ```
  from = 2026-06-01,  to = 2026-06-30

  date = 2026-06-15  → date != null ✓, !before(from) ✓, !after(to) ✓  → THÊM VÀO
  date = 2026-05-31  → !date.before(from) ✗  → BỎ QUA
  date = 2026-07-01  → !date.after(to) ✗     → BỎ QUA
  date = null        → date != null ✗         → BỎ QUA
  ```

```
BƯỚC 5 ▶  result.add(invoice);
```
- Thêm hóa đơn thỏa điều kiện vào danh sách kết quả.

```
BƯỚC 6 ▶  return result;
```
- Trả về danh sách sau khi duyệt hết `data`.
- Nếu không có hóa đơn nào thỏa điều kiện → trả về `ArrayList` rỗng (không phải `null`).

#### 📋 Ví dụ:
```java
Date from = new SimpleDateFormat("yyyy-MM-dd").parse("2026-06-01");
Date to   = new SimpleDateFormat("yyyy-MM-dd").parse("2026-06-30");

List<Invoice> juneInvoices = repo.findByDateRange(from, to);
System.out.println("Hóa đơn tháng 6: " + juneInvoices.size());
```

---

## 5. UML Class Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│              BaseRepository<Invoice, String>                    │
│                     <<abstract>>                                │
├─────────────────────────────────────────────────────────────────┤
│ # data         : List<Invoice>                                  │
│ # filePath     : String                                         │
├─────────────────────────────────────────────────────────────────┤
│ + findAll()    : List<Invoice>                                  │
│ + save(t)      : void                                           │
│ + delete(id)   : void                                           │
│ + findById(id) : Invoice          ← abstract                   │
│ # parseLine(line) : Invoice       ← abstract                   │
│ # toLine(t)    : String           ← abstract                   │
└─────────────────────────────────────────────────────────────────┘
                        ▲
                        │  extends
                        │
┌─────────────────────────────────────────────────────────────────┐
│                   InvoiceRepository                             │
├─────────────────────────────────────────────────────────────────┤
│   (không có thuộc tính mới – dùng data từ BaseRepository)      │
├─────────────────────────────────────────────────────────────────┤
│ + InvoiceRepository()                                           │
├─────────────────────────────────────────────────────────────────┤
│ + findById(id: String)                   : Invoice   @Override  │
│ # parseLine(line: String)                : Invoice   @Override  │
│ # toLine(invoice: Invoice)               : String    @Override  │
│ + findByOrderId(orderId: String)         : Invoice              │
│ + findByDateRange(from: Date, to: Date)  : List<Invoice>        │
└─────────────────────────────────────────────────────────────────┘
          │                    │
          │ uses                │ uses
          ▼                    ▼
   ┌────────────┐       ┌───────────────┐
   │  Invoice   │       │  AppConfig    │
   │  (model)   │       │ INVOICE_FILE  │
   └────────────┘       │    _PATH      │
          │             └───────────────┘
          │ uses
          ▼
   ┌──────────────────┐
   │  InvoiceStatus   │
   │    <<enum>>      │
   ├──────────────────┤
   │  UNPAID          │
   │  PAID            │
   └──────────────────┘
```

---

## 6. Quan hệ với các Class khác

| Class / Interface      | Loại quan hệ  | Mô tả                                                                            |
|------------------------|---------------|----------------------------------------------------------------------------------|
| `BaseRepository`       | Inheritance   | `InvoiceRepository` **kế thừa** toàn bộ cơ sở hạ tầng đọc/ghi file              |
| `Invoice`              | Dependency    | Kiểu dữ liệu chính được quản lý; tạo, đọc, trả về trong mọi phương thức         |
| `Order`                | Dependency    | Tạo đối tượng `Order` tạm trong `parseLine()` để gắn vào `Invoice`              |
| `InvoiceStatus`        | Dependency    | Dùng `valueOf()` để chuyển chuỗi từ file thành enum                              |
| `AppConfig`            | Dependency    | Cung cấp hằng số đường dẫn file `INVOICE_FILE_PATH`                              |

---

## 7. Cấu trúc file lưu trữ

File text lưu trữ hóa đơn có định dạng **pipe-separated** (`|`), mỗi dòng là một hóa đơn:

```
invoiceId | orderId | totalAmount | issueDate  | status
──────────────────────────────────────────────────────
INV001    | ORD001  | 250000.0    | 2026-06-28 | PAID
INV002    | ORD002  | 180000.0    | 2026-06-28 | UNPAID
INV003    | ORD003  | 320000.5    | 2026-06-27 | PAID
```

### Quy tắc:
| Index | Trường        | Kiểu dữ liệu | Bắt buộc | Ghi chú                                  |
|-------|---------------|--------------|----------|------------------------------------------|
| `d[0]`| `invoiceId`   | String       | ✅        | Mã định danh duy nhất                    |
| `d[1]`| `orderId`     | String       | ✅        | Mã đơn hàng liên kết                     |
| `d[2]`| `totalAmount` | Double       | ✅        | Phải là số thập phân hợp lệ              |
| `d[3]`| `issueDate`   | String       | ✅        | Format: `yyyy-MM-dd`; fallback = ngày nay|
| `d[4]`| `status`      | String       | ✅        | Phải khớp tên hằng số trong `InvoiceStatus` |

---

## 8. Luồng xử lý điển hình

```
┌───────────────────────────────────────────────────────────────────┐
│                 KHỞI TẠO & ĐỌC DỮ LIỆU TỪ FILE                  │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  new InvoiceRepository()                                          │
│       │                                                           │
│       ▼                                                           │
│  super(AppConfig.INVOICE_FILE_PATH)                               │
│       │                                                           │
│       ▼                                                           │
│  BaseRepository đọc file theo từng dòng                          │
│       │                                                           │
│       ▼                                                           │
│  parseLine("INV001|ORD001|250000.0|2026-06-28|UNPAID")           │
│       │  → split("|") → d[0..4]                                   │
│       │  → tạo Invoice + gán từng thuộc tính                     │
│       │  → trả về Invoice object                                  │
│       ▼                                                           │
│  data = [Invoice1, Invoice2, Invoice3, ...]   ← sẵn sàng dùng   │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────┐
│                    TÌM KIẾM & SỬ DỤNG                            │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  findById("INV001")        → duyệt data, so sánh invoiceId       │
│  findByOrderId("ORD001")   → duyệt data, so sánh orderId         │
│  findByDateRange(d1, d2)   → duyệt data, lọc theo khoảng ngày   │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────┐
│                    GHI DỮ LIỆU XUỐNG FILE                        │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  BaseRepository.save(invoice)                                     │
│       │                                                           │
│       ▼                                                           │
│  toLine(invoice)                                                  │
│       │  → nối các thuộc tính bằng "|"                           │
│       │  → format Date thành "yyyy-MM-dd"                        │
│       │  → trả về "INV001|ORD001|250000.0|2026-06-28|UNPAID"     │
│       ▼                                                           │
│  Ghi chuỗi vào file                                              │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

---

## 9. Lưu ý & Điểm cần cải thiện

| # | Vấn đề                                                                                  | Khuyến nghị                                                                   |
|---|-----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| 1 | `parseLine()`: `InvoiceStatus.valueOf(d[4])` ném `IllegalArgumentException` nếu sai tên | Bọc trong `try-catch` hoặc dùng Enum helper method để kiểm tra trước          |
| 2 | `parseLine()`: `Double.parseDouble(d[2])` ném `NumberFormatException` nếu sai format    | Bọc trong `try-catch` hoặc kiểm tra `d[2]` trước khi parse                   |
| 3 | `parseLine()`: `Order` được tạo chỉ có `orderId`, thiếu dữ liệu đầy đủ                 | Sau khi load, nên inject đầy đủ `Order` từ `OrderRepository` (lazy loading)   |
| 4 | `toLine()`: Không kiểm tra `issueDate == null` trước khi `.format()`                    | Thêm `invoice.getIssueDate() != null ? ... : ""`                              |
| 5 | `findByDateRange()`: Không kiểm tra `from` hoặc `to` là `null`                          | Thêm validation đầu hàm: `if (from == null \|\| to == null) return result;`   |
| 6 | Tất cả `find*()` đều dùng vòng lặp O(n)                                                 | Nếu dữ liệu lớn, nên dùng `HashMap<String, Invoice>` để tìm kiếm O(1)        |
| 7 | `SimpleDateFormat` tạo mới mỗi lần gọi `parseLine()`/`toLine()`                        | Khai báo là `static final` để tái sử dụng, tiết kiệm bộ nhớ                  |

---

*📝 Tài liệu được tạo từ source code `InvoiceRepository.java` – Restaurant Management System.*  
*🗓️ Ngày tạo: 28/06/2026*
