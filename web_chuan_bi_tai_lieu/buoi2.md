# Tài liệu ôn tập: Lý thuyết thiết kế Cơ sở dữ liệu

---

## 1. Lý thuyết cơ bản về thiết kế cơ sở dữ liệu

### 1.1. Cơ sở dữ liệu (Database) là gì?

Cơ sở dữ liệu là một tập hợp dữ liệu có tổ chức, được lưu trữ có hệ thống để dễ dàng truy xuất, quản lý và cập nhật. Ví dụ: dữ liệu khách hàng, đơn hàng, sản phẩm của một cửa hàng.

### 1.2. Vì sao cần thiết kế CSDL trước khi tạo bảng?

Nếu thiết kế sai ngay từ đầu, hệ thống sẽ gặp các vấn đề:

- **Dư thừa dữ liệu:** một thông tin bị lưu lặp lại ở nhiều nơi.
- **Bất thường khi cập nhật:** sửa dữ liệu ở chỗ này nhưng quên sửa ở chỗ khác → dữ liệu mâu thuẫn nhau.
- **Bất thường khi thêm:** không thể thêm dữ liệu mới vì thiếu thông tin liên quan.
- **Bất thường khi xóa:** xóa 1 dòng dữ liệu vô tình làm mất luôn thông tin khác không liên quan.

**Ví dụ minh họa:** Giả sử bạn gộp chung thông tin đặt bàn và thông tin khách hàng vào **1 bảng duy nhất**:

| reservation_id | customer_name | phone | table_name | reservation_date |
|---|---|---|---|---|
| 1 | Nguyễn Văn A | 0901xxx | Bàn 1 | 20/08/2026 |
| 2 | Nguyễn Văn A | 0901xxx | Bàn 3 | 21/08/2026 |

→ Tên và SĐT của "Nguyễn Văn A" bị lặp lại ở 2 dòng (dư thừa dữ liệu). Nếu khách đổi số điện thoại, bạn phải sửa ở **tất cả các dòng** liên quan đến khách đó — dễ sót, dễ sai (update anomaly).

**Giải pháp:** tách thành các bảng riêng biệt (Customer, Table, Reservation) và liên kết bằng khóa ngoại — đây chính là mục tiêu của thiết kế CSDL và chuẩn hóa.

### 1.3. Các bước thiết kế CSDL cơ bản

1. **Khảo sát yêu cầu nghiệp vụ** — xác định hệ thống cần quản lý những đối tượng gì (khách hàng, bàn, đơn đặt...).
2. **Xác định thực thể (Entity) và thuộc tính (Attribute)** — ví dụ: Khách hàng có các thuộc tính: mã KH, tên, SĐT, email.
3. **Xác định mối quan hệ (Relationship)** giữa các thực thể — ví dụ: 1 khách hàng có thể có nhiều lượt đặt bàn.
4. **Vẽ lược đồ E-R (Entity-Relationship Diagram).**
5. **Chuyển lược đồ E-R sang mô hình quan hệ** (các bảng, khóa chính, khóa ngoại).
6. **Chuẩn hóa dữ liệu** (1NF, 2NF, 3NF...) để loại bỏ dư thừa.
7. **Cài đặt vật lý (Physical Design):** viết lệnh `CREATE TABLE`, chọn kiểu dữ liệu, ràng buộc...

---

## 2. Lược đồ quan hệ E-R (Entity-Relationship Diagram)

### 2.1. Các thành phần chính

| Thành phần | Ký hiệu (chuẩn Chen) | Ý nghĩa |
|---|---|---|
| **Thực thể (Entity)** | Hình chữ nhật | Một đối tượng cụ thể trong thực tế cần quản lý (VD: Khách hàng, Bàn) |
| **Thuộc tính (Attribute)** | Hình elip | Đặc điểm mô tả thực thể (VD: tên, SĐT) |
| **Khóa chính (Primary Key)** | Thuộc tính gạch chân | Thuộc tính xác định duy nhất 1 thực thể |
| **Mối quan hệ (Relationship)** | Hình thoi | Sự liên kết giữa 2 hay nhiều thực thể (VD: "Đặt") |
| **Đường nối** | Đường thẳng | Nối thực thể với thuộc tính hoặc với mối quan hệ |

### 2.2. Các loại thuộc tính

- **Thuộc tính đơn (Simple):** không chia nhỏ được — VD: tuổi.
- **Thuộc tính phức hợp (Composite):** chia được thành nhiều phần — VD: địa chỉ = số nhà + đường + thành phố.
- **Thuộc tính đa trị (Multivalued):** có thể có nhiều giá trị — VD: 1 khách hàng có thể có nhiều số điện thoại. Ký hiệu: elip đôi.
- **Thuộc tính suy diễn (Derived):** tính được từ thuộc tính khác — VD: tuổi tính từ ngày sinh. Ký hiệu: elip nét đứt.

### 2.3. Bậc của mối quan hệ (Cardinality)

Bậc thể hiện **số lượng thực thể tham gia tối đa** vào một mối quan hệ:

| Loại | Ý nghĩa | Ví dụ |
|---|---|---|
| **1-1 (một-một)** | 1 thực thể A liên kết với đúng 1 thực thể B | 1 người có đúng 1 CCCD |
| **1-N (một-nhiều)** | 1 thực thể A liên kết với nhiều thực thể B | 1 khách hàng có thể có **nhiều** lượt đặt bàn |
| **N-N (nhiều-nhiều)** | Nhiều A liên kết với nhiều B | 1 sản phẩm có thể xuất hiện trong nhiều đơn hàng, 1 đơn hàng có nhiều sản phẩm |

**Ví dụ minh họa E-R cho hệ thống đặt bàn nhà hàng:**

```
[Khách hàng] ----(1)---- <Đặt> ----(N)---- [Bàn]
```

Diễn giải: 1 khách hàng có thể **thực hiện nhiều** lượt đặt (quan hệ Đặt), và mỗi lượt đặt gắn với **1 bàn cụ thể**. Đây là quan hệ 1-N giữa Khách hàng và lượt Đặt bàn, còn giữa Bàn và lượt Đặt bàn cũng là 1-N (1 bàn có thể được đặt nhiều lần vào các thời điểm khác nhau).

### 2.4. Thực thể yếu (Weak Entity)

Là thực thể **không có khóa chính riêng**, phải phụ thuộc vào 1 thực thể khác (gọi là thực thể chủ) để xác định. Ký hiệu: hình chữ nhật đôi.

**Ví dụ:** "Chi tiết đặt bàn" nếu tồn tại thì phụ thuộc hoàn toàn vào "Đặt bàn" — không có "Đặt bàn" thì "Chi tiết đặt bàn" không có ý nghĩa tồn tại độc lập.

---

## 3. Mô hình dữ liệu quan hệ (Relational Data Model)

### 3.1. Khái niệm cơ bản

Mô hình quan hệ biểu diễn dữ liệu dưới dạng các **bảng (table/relation)**, mỗi bảng gồm:

- **Dòng (Row / Tuple / Record):** một bản ghi cụ thể.
- **Cột (Column / Attribute / Field):** một thuộc tính của dữ liệu.
- **Miền giá trị (Domain):** tập giá trị hợp lệ mà 1 cột có thể nhận (VD: cột "giới tính" chỉ nhận "Nam"/"Nữ").

### 3.2. Khóa (Key)

| Loại khóa | Định nghĩa | Ví dụ |
|---|---|---|
| **Khóa chính (Primary Key - PK)** | Cột (hoặc tổ hợp cột) xác định duy nhất 1 dòng, không được NULL, không trùng | `customer_id` trong bảng Customer |
| **Khóa ngoại (Foreign Key - FK)** | Cột tham chiếu đến khóa chính của bảng khác, dùng để liên kết 2 bảng | `customer_id` trong bảng Reservation, tham chiếu đến Customer |
| **Khóa dự tuyển (Candidate Key)** | Các cột có khả năng làm khóa chính (VD: cả `customer_id` và `email` đều có thể làm PK vì đều duy nhất) | |
| **Siêu khóa (Super Key)** | Tập hợp cột (có thể dư) vẫn đảm bảo xác định duy nhất 1 dòng | `{customer_id, phone}` |

**Ví dụ minh họa 3 bảng đã tạo ở phần trước:**

```
Customer (customer_id [PK], customer_name, phone, email, address)
Restaurant_Table (table_id [PK], table_name, max_capacity, description)
Reservation (reservation_id [PK], customer_id [FK → Customer], table_id [FK → Restaurant_Table],
             reservation_date, reservation_time, number_of_guests, status)
```

→ `customer_id` và `table_id` trong bảng `Reservation` là **khóa ngoại**, giúp biết lượt đặt đó là của khách nào, đặt bàn nào — mà không cần lặp lại toàn bộ thông tin khách hàng/bàn vào bảng Reservation.

### 3.3. Các ràng buộc toàn vẹn (Integrity Constraints)

- **Ràng buộc thực thể (Entity Integrity):** khóa chính không được NULL.
- **Ràng buộc tham chiếu (Referential Integrity):** giá trị khóa ngoại phải tồn tại trong bảng được tham chiếu (VD: `customer_id` trong Reservation phải là 1 `customer_id` có thật trong bảng Customer).
- **Ràng buộc miền giá trị (Domain Constraint):** dữ liệu nhập vào phải đúng kiểu, đúng phạm vi cho phép (VD: `number_of_guests` phải > 0).

### 3.4. Chuyển từ E-R sang mô hình quan hệ (quy tắc nhanh)

1. Mỗi **thực thể** → 1 bảng.
2. Mỗi **thuộc tính đơn** → 1 cột trong bảng.
3. **Thuộc tính khóa** → khóa chính của bảng.
4. Quan hệ **1-N** → thêm khóa ngoại của bên "1" vào bảng bên "N" (VD: thêm `customer_id` vào bảng Reservation).
5. Quan hệ **N-N** → tạo bảng trung gian mới, chứa khóa chính của cả 2 bảng làm khóa ngoại (VD: bảng `OrderDetail` chứa `order_id` và `product_id`).
6. **Thuộc tính đa trị** → tách thành bảng riêng.

---

## 4. Chuẩn hóa dữ liệu (Normalization): 1NF, 2NF, 3NF

### 4.1. Mục đích chuẩn hóa

Chuẩn hóa là quá trình tổ chức lại dữ liệu để:
- Loại bỏ **dư thừa dữ liệu**.
- Tránh các **bất thường** khi thêm/sửa/xóa (đã nói ở mục 1.2).
- Đảm bảo tính **nhất quán** của dữ liệu.

Chuẩn hóa được thực hiện qua các **dạng chuẩn (Normal Form)** tăng dần mức độ chặt chẽ: 1NF → 2NF → 3NF → ...

### 4.2. Dạng chuẩn 1 (1NF - First Normal Form)

**Điều kiện:** mỗi cột chỉ chứa **giá trị nguyên tố (atomic)** — nghĩa là không được chứa nhiều giá trị trong 1 ô, không có nhóm lặp.

**Ví dụ VI PHẠM 1NF:**

| customer_id | customer_name | phone_numbers |
|---|---|---|
| 1 | Nguyễn Văn A | 0901xxx, 0912yyy |

→ Cột `phone_numbers` chứa **nhiều giá trị** trong 1 ô — vi phạm 1NF.

**Cách sửa (đạt 1NF):** tách thành nhiều dòng, hoặc tạo bảng riêng cho số điện thoại:

| customer_id | customer_name | phone |
|---|---|---|
| 1 | Nguyễn Văn A | 0901xxx |
| 1 | Nguyễn Văn A | 0912yyy |

Hoặc tốt hơn — tách hẳn bảng `Customer_Phone (customer_id, phone)` riêng.

### 4.3. Dạng chuẩn 2 (2NF - Second Normal Form)

**Điều kiện:** 
1. Đã đạt 1NF.
2. Mọi thuộc tính **không khóa** phải phụ thuộc **hoàn toàn** vào **toàn bộ** khóa chính (không phụ thuộc vào chỉ 1 phần của khóa chính — chỉ áp dụng khi khóa chính gồm **nhiều cột**, gọi là khóa phức hợp).

**Ví dụ VI PHẠM 2NF:** Bảng `OrderDetail` có khóa chính phức hợp `(order_id, product_id)`:

| order_id | product_id | product_name | quantity |
|---|---|---|---|
| 1 | 101 | Cà phê sữa | 2 |
| 1 | 102 | Trà đào | 1 |

→ `product_name` chỉ phụ thuộc vào `product_id` (1 phần của khóa chính), **không phụ thuộc vào `order_id`** → vi phạm 2NF. Vì nếu 1 sản phẩm đổi tên, bạn phải sửa ở **tất cả các đơn hàng** có sản phẩm đó (dư thừa + dễ sai).

**Cách sửa (đạt 2NF):** tách `product_name` ra bảng `Product` riêng:

```
Product (product_id [PK], product_name)
OrderDetail (order_id [FK], product_id [FK], quantity)
```

### 4.4. Dạng chuẩn 3 (3NF - Third Normal Form)

**Điều kiện:**
1. Đã đạt 2NF.
2. Không có **phụ thuộc bắc cầu (Transitive Dependency)** — nghĩa là thuộc tính không khóa không được phụ thuộc vào 1 thuộc tính không khóa khác, mà chỉ được phụ thuộc trực tiếp vào khóa chính.

**Ví dụ VI PHẠM 3NF:** Bảng `Reservation` gộp thêm thông tin nhà hàng:

| reservation_id | customer_id | restaurant_id | restaurant_name | restaurant_address |
|---|---|---|---|---|
| 1 | 5 | 10 | Nhà hàng Sen | 123 Lê Lợi |
| 2 | 8 | 10 | Nhà hàng Sen | 123 Lê Lợi |

→ `restaurant_name` và `restaurant_address` phụ thuộc vào `restaurant_id`, mà `restaurant_id` lại là thuộc tính **không khóa** (không phải khóa chính của bảng Reservation) → đây là **phụ thuộc bắc cầu**: `reservation_id → restaurant_id → restaurant_name`. Vi phạm 3NF.

**Cách sửa (đạt 3NF):** tách thành bảng `Restaurant` riêng:

```
Restaurant (restaurant_id [PK], restaurant_name, restaurant_address)
Reservation (reservation_id [PK], customer_id [FK], restaurant_id [FK], ...)
```

### 4.5. Bảng tóm tắt 3 dạng chuẩn

| Dạng chuẩn | Yêu cầu chính | Mục đích |
|---|---|---|
| **1NF** | Giá trị mỗi cột phải nguyên tố, không lặp nhóm | Loại bỏ dữ liệu đa trị trong 1 ô |
| **2NF** | Đạt 1NF + thuộc tính không khóa phụ thuộc **toàn bộ** khóa chính | Loại bỏ phụ thuộc vào 1 phần của khóa (áp dụng khi khóa phức hợp) |
| **3NF** | Đạt 2NF + không có phụ thuộc bắc cầu | Loại bỏ phụ thuộc gián tiếp giữa các thuộc tính không khóa |

### 4.6. Áp dụng vào ví dụ hệ thống đặt bàn (đã chuẩn hóa đúng)

3 bảng bạn đã tạo (`Customer`, `Restaurant_Table`, `Reservation`) thực chất **đã đạt 3NF**:

- Mỗi cột đều chứa giá trị nguyên tố → đạt 1NF.
- Khóa chính mỗi bảng là 1 cột đơn (`customer_id`, `table_id`, `reservation_id`) nên không có vấn đề phụ thuộc 1 phần → đạt 2NF.
- Không có thuộc tính nào phụ thuộc gián tiếp qua 1 thuộc tính không khóa khác → đạt 3NF.

Đây chính là lý do vì sao nên tách 3 bảng riêng thay vì gộp chung — nó đảm bảo dữ liệu không dư thừa và tránh các lỗi khi cập nhật/xóa dữ liệu.

---

## 5. Tóm tắt nhanh trước buổi học

- **Thiết kế CSDL** = quy trình xác định thực thể → vẽ E-R → chuyển sang bảng quan hệ → chuẩn hóa → cài đặt.
- **E-R diagram**: thực thể (chữ nhật), thuộc tính (elip), quan hệ (hình thoi), chú ý bậc quan hệ 1-1, 1-N, N-N.
- **Mô hình quan hệ**: dữ liệu là các bảng, khóa chính xác định duy nhất 1 dòng, khóa ngoại dùng để liên kết bảng.
- **Chuẩn hóa**: 1NF (giá trị nguyên tố) → 2NF (phụ thuộc toàn bộ khóa) → 3NF (không phụ thuộc bắc cầu) — mục tiêu cuối cùng là **giảm dư thừa, tránh bất thường dữ liệu**.