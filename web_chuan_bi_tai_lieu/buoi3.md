# Tài liệu SQL Cơ Bản

Tài liệu này tổng hợp các thao tác nền tảng trong SQL. Mỗi phần đều theo cấu trúc: **cú pháp tổng quát → giải thích ý nghĩa từng thành phần → ví dụ minh họa**. Chúng ta sẽ dùng chung 2 bảng dữ liệu mẫu xuyên suốt tài liệu:

**Bảng `nhan_vien`**

| id | ho_ten      | phong_ban | luong    | tuoi |
|----|-------------|-----------|----------|------|
| 1  | Nguyễn An   | Kinh doanh| 15000000 | 28   |
| 2  | Trần Bình   | Kỹ thuật  | 20000000 | 32   |
| 3  | Lê Chi      | Kỹ thuật  | 18000000 | 25   |
| 4  | Phạm Dũng   | Kinh doanh| 15000000 | 40   |
| 5  | Hồ Em       | Nhân sự   | 12000000 | 29   |

**Bảng `phong_ban`**

| ma_pb | ten_pb     | truong_phong |
|-------|------------|--------------|
| 1     | Kinh doanh | Nguyễn An    |
| 2     | Kỹ thuật   | Trần Bình    |
| 3     | Nhân sự    | Hồ Em        |
| 4     | Marketing  | NULL         |

---

## 1. Các thao tác cơ bản: SELECT, INSERT, UPDATE, DELETE

### 1.1. SELECT — Truy vấn dữ liệu

**Cấu trúc tổng quát:**

```sql
SELECT cot1, cot2, ...
FROM ten_bang;
```

**Giải thích:**
- `SELECT cot1, cot2, ...`: liệt kê các cột muốn lấy ra. Nếu muốn lấy tất cả các cột, dùng dấu `*` thay vì tên cột.
- `FROM ten_bang`: chỉ định bảng nguồn chứa dữ liệu cần truy vấn.

**Ví dụ:**

```sql
-- Lấy tất cả các cột
SELECT * FROM nhan_vien;

-- Chỉ lấy một số cột cần thiết
SELECT ho_ten, luong FROM nhan_vien;
```

Kết quả câu thứ 2:

| ho_ten    | luong    |
|-----------|----------|
| Nguyễn An | 15000000 |
| Trần Bình | 20000000 |
| ...       | ...      |

### 1.2. INSERT — Thêm dữ liệu mới

**Cấu trúc tổng quát:**

```sql
INSERT INTO ten_bang (cot1, cot2, ...)
VALUES (gia_tri1, gia_tri2, ...);
```

**Giải thích:**
- `INSERT INTO ten_bang (cot1, cot2, ...)`: chỉ định bảng và danh sách các cột sẽ được gán giá trị.
- `VALUES (gia_tri1, gia_tri2, ...)`: các giá trị tương ứng theo đúng thứ tự với danh sách cột ở trên. Nếu bỏ qua một cột, cột đó sẽ nhận giá trị mặc định hoặc `NULL`.

**Ví dụ:**

```sql
INSERT INTO nhan_vien (id, ho_ten, phong_ban, luong, tuoi)
VALUES (6, 'Vũ Giang', 'Marketing', 13000000, 27);
```

> Sau lệnh này, bảng `nhan_vien` sẽ có thêm 1 dòng dữ liệu với `id = 6`.

### 1.3. UPDATE — Cập nhật dữ liệu

**Cấu trúc tổng quát:**

```sql
UPDATE ten_bang
SET cot1 = gia_tri_moi1, cot2 = gia_tri_moi2, ...
WHERE dieu_kien;
```

**Giải thích:**
- `UPDATE ten_bang`: chỉ định bảng cần sửa dữ liệu.
- `SET cot = gia_tri_moi`: gán giá trị mới cho các cột cần cập nhật.
- `WHERE dieu_kien`: xác định những dòng nào sẽ bị ảnh hưởng. **Bắt buộc phải có** nếu không toàn bộ các dòng trong bảng sẽ bị cập nhật.

**Ví dụ:**

```sql
-- Tăng lương 10% cho nhân viên phòng Kỹ thuật
UPDATE nhan_vien
SET luong = luong * 1.1
WHERE phong_ban = 'Kỹ thuật';
```

### 1.4. DELETE — Xóa dữ liệu

**Cấu trúc tổng quát:**

```sql
DELETE FROM ten_bang
WHERE dieu_kien;
```

**Giải thích:**
- `DELETE FROM ten_bang`: chỉ định bảng cần xóa dữ liệu.
- `WHERE dieu_kien`: xác định những dòng nào sẽ bị xóa. Tương tự UPDATE, thiếu `WHERE` sẽ xóa toàn bộ dữ liệu trong bảng.

**Ví dụ:**

```sql
-- Xóa nhân viên có id = 6
DELETE FROM nhan_vien
WHERE id = 6;
```

### 1.5. Từ khóa AS — Đặt bí danh (alias)

**Cấu trúc tổng quát:**

```sql
SELECT cot AS ten_moi
FROM ten_bang AS bi_danh_bang;
```

**Giải thích:**
- `cot AS ten_moi`: đổi tên hiển thị của cột trong kết quả trả về (không đổi tên cột thật trong bảng).
- `ten_bang AS bi_danh_bang`: đặt tên tắt (bí danh) cho bảng, thường dùng khi câu lệnh có nhiều bảng để viết ngắn gọn hơn. Từ khóa `AS` có thể lược bỏ (viết trực tiếp `ten_bang bi_danh_bang`) mà vẫn hợp lệ.

**Ví dụ:**

```sql
SELECT ho_ten AS "Họ và tên", luong AS "Lương (VNĐ)"
FROM nhan_vien AS nv;
```

Kết quả:

| Họ và tên  | Lương (VNĐ) |
|------------|-------------|
| Nguyễn An  | 15000000    |

### 1.6. DISTINCT — Loại bỏ giá trị trùng lặp

**Cấu trúc tổng quát:**

```sql
SELECT DISTINCT cot1, cot2, ...
FROM ten_bang;
```

**Giải thích:**
- `DISTINCT` đặt ngay sau `SELECT`, áp dụng cho toàn bộ danh sách cột phía sau nó: chỉ giữ lại các **tổ hợp giá trị duy nhất**, loại bỏ những dòng bị trùng hoàn toàn.

**Ví dụ:**

```sql
-- Liệt kê danh sách các phòng ban (không trùng)
SELECT DISTINCT phong_ban FROM nhan_vien;
```

Kết quả:

| phong_ban  |
|------------|
| Kinh doanh |
| Kỹ thuật   |
| Nhân sự    |

Nếu không có `DISTINCT`, "Kinh doanh" sẽ xuất hiện 2 lần (vì có 2 nhân viên thuộc phòng này).

---

## 2. Lọc dữ liệu: WHERE, HAVING

### 2.1. WHERE — Lọc dòng trước khi nhóm

**Cấu trúc tổng quát:**

```sql
SELECT cot1, cot2, ...
FROM ten_bang
WHERE dieu_kien;
```

**Giải thích:**
- `WHERE dieu_kien`: áp dụng cho **từng dòng dữ liệu gốc**, thực thi trước khi nhóm (GROUP BY). Dòng nào không thỏa điều kiện sẽ bị loại bỏ trước khi các bước xử lý tiếp theo diễn ra.
- Có thể kết hợp nhiều điều kiện bằng `AND`, `OR`, và dùng các toán tử so sánh (`=`, `>`, `<`, `<>`, `LIKE`, `IN`, `BETWEEN`...).

**Ví dụ:**

```sql
-- Lấy nhân viên có lương >= 15 triệu
SELECT ho_ten, luong
FROM nhan_vien
WHERE luong >= 15000000;
```

Kết hợp nhiều điều kiện:

```sql
SELECT ho_ten
FROM nhan_vien
WHERE phong_ban = 'Kỹ thuật' AND tuoi < 30;
```

### 2.2. HAVING — Lọc sau khi nhóm

**Cấu trúc tổng quát:**

```sql
SELECT cot_nhom, ham_tong_hop(cot)
FROM ten_bang
GROUP BY cot_nhom
HAVING dieu_kien_tren_ham_tong_hop;
```

**Giải thích:**
- `HAVING` luôn đi sau `GROUP BY`, dùng để lọc trên **kết quả đã được nhóm** — thường là điều kiện liên quan đến hàm tổng hợp (`SUM`, `COUNT`, `AVG`...) mà `WHERE` không làm được vì `WHERE` chạy trước khi dữ liệu được nhóm.

**Ví dụ:**

```sql
-- Lấy các phòng ban có tổng quỹ lương > 25 triệu
SELECT phong_ban, SUM(luong) AS tong_luong
FROM nhan_vien
GROUP BY phong_ban
HAVING SUM(luong) > 25000000;
```

Kết quả:

| phong_ban  | tong_luong |
|------------|------------|
| Kinh doanh | 30000000   |
| Kỹ thuật   | 38000000   |

**Phân biệt nhanh:** `WHERE` lọc dữ liệu thô trước khi gom nhóm; `HAVING` lọc kết quả sau khi đã gom nhóm và tính toán.

---

## 3. Kết hợp bảng và kết quả: JOIN, UNION

### 3.1. INNER JOIN — Chỉ lấy dòng khớp ở cả 2 bảng

**Cấu trúc tổng quát:**

```sql
SELECT bang1.cot, bang2.cot
FROM bang1
INNER JOIN bang2 ON bang1.cot_chung = bang2.cot_chung;
```

**Giải thích:**
- `INNER JOIN bang2`: kết hợp bảng hiện tại với `bang2`.
- `ON bang1.cot_chung = bang2.cot_chung`: điều kiện khớp nối — chỉ những dòng có giá trị trùng khớp ở cả 2 bảng mới xuất hiện trong kết quả. Dòng nào không khớp ở một trong hai bảng sẽ bị loại bỏ hoàn toàn.

**Ví dụ:**

```sql
SELECT nv.ho_ten, pb.ten_pb
FROM nhan_vien nv
INNER JOIN phong_ban pb ON nv.phong_ban = pb.ten_pb;
```

Kết quả: chỉ những nhân viên có phòng ban tồn tại trong bảng `phong_ban` mới xuất hiện (phòng "Marketing" không có nhân viên nên không ảnh hưởng ở đây).

### 3.2. LEFT JOIN — Giữ toàn bộ bảng bên trái

**Cấu trúc tổng quát:**

```sql
SELECT bang1.cot, bang2.cot
FROM bang1
LEFT JOIN bang2 ON bang1.cot_chung = bang2.cot_chung;
```

**Giải thích:**
- Giữ **toàn bộ** các dòng của bảng bên trái (`bang1`), dù có khớp với `bang2` hay không.
- Nếu một dòng ở `bang1` không tìm được dòng khớp bên `bang2`, các cột lấy từ `bang2` sẽ nhận giá trị `NULL`.

**Ví dụ:**

```sql
SELECT pb.ten_pb, nv.ho_ten
FROM phong_ban pb
LEFT JOIN nhan_vien nv ON pb.ten_pb = nv.phong_ban;
```

Kết quả:

| ten_pb     | ho_ten    |
|------------|-----------|
| Kinh doanh | Nguyễn An |
| Kinh doanh | Phạm Dũng |
| Kỹ thuật   | Trần Bình |
| Kỹ thuật   | Lê Chi    |
| Nhân sự    | Hồ Em     |
| Marketing  | NULL      |

Phòng "Marketing" vẫn xuất hiện dù không có nhân viên nào, vì `LEFT JOIN` giữ toàn bộ dòng từ bảng bên trái (`phong_ban`).

### 3.3. UNION — Gộp kết quả của 2 truy vấn

**Cấu trúc tổng quát:**

```sql
SELECT cot1, cot2 FROM bang1 WHERE dieu_kien1
UNION
SELECT cot1, cot2 FROM bang2 WHERE dieu_kien2;
```

**Giải thích:**
- `UNION` gộp kết quả theo **chiều dọc** (nối thêm dòng), yêu cầu 2 truy vấn có **cùng số lượng cột** và **kiểu dữ liệu tương ứng** ở từng vị trí cột.
- `UNION` tự động loại bỏ các dòng trùng lặp; nếu muốn giữ nguyên tất cả (kể cả trùng), dùng `UNION ALL`.

**Ví dụ:**

```sql
SELECT ho_ten FROM nhan_vien WHERE phong_ban = 'Kinh doanh'
UNION
SELECT ho_ten FROM nhan_vien WHERE tuoi < 30;
```

Kết quả (không trùng lặp dù Nguyễn An thỏa cả 2 điều kiện):

| ho_ten    |
|-----------|
| Nguyễn An |
| Phạm Dũng |
| Lê Chi    |
| Hồ Em     |

---

## 4. Tổng hợp và nhóm dữ liệu: COUNT, SUM, AVG, GROUP BY

### 4.1. Các hàm tổng hợp (Aggregate Functions)

**Cấu trúc tổng quát:**

```sql
SELECT COUNT(cot), SUM(cot), AVG(cot), MAX(cot), MIN(cot)
FROM ten_bang;
```

**Giải thích:**
- `COUNT(cot)`: đếm số dòng có giá trị khác `NULL` ở cột đó (`COUNT(*)` đếm toàn bộ số dòng).
- `SUM(cot)`: tính tổng giá trị của cột (chỉ dùng cho cột số).
- `AVG(cot)`: tính giá trị trung bình.
- `MAX(cot)` / `MIN(cot)`: lấy giá trị lớn nhất / nhỏ nhất.
- Khi không có `GROUP BY`, các hàm này tính trên **toàn bộ bảng** và trả về đúng 1 dòng kết quả duy nhất.

**Ví dụ:**

```sql
SELECT
    COUNT(*) AS so_luong_nv,
    SUM(luong) AS tong_luong,
    AVG(luong) AS luong_trung_binh,
    MAX(luong) AS luong_cao_nhat,
    MIN(luong) AS luong_thap_nhat
FROM nhan_vien;
```

Kết quả:

| so_luong_nv | tong_luong | luong_trung_binh | luong_cao_nhat | luong_thap_nhat |
|-------------|------------|-------------------|-----------------|-------------------|
| 5           | 80000000   | 16000000          | 20000000        | 12000000          |

### 4.2. GROUP BY — Nhóm dữ liệu theo cột

**Cấu trúc tổng quát:**

```sql
SELECT cot_nhom, ham_tong_hop(cot)
FROM ten_bang
GROUP BY cot_nhom;
```

**Giải thích:**
- `GROUP BY cot_nhom`: gom các dòng có cùng giá trị ở `cot_nhom` thành một nhóm; hàm tổng hợp phía trên sẽ tính riêng cho từng nhóm thay vì toàn bảng.
- **Quy tắc quan trọng:** mọi cột xuất hiện trong `SELECT` mà không nằm trong hàm tổng hợp thì bắt buộc phải có trong `GROUP BY`.

**Ví dụ:**

```sql
SELECT phong_ban, COUNT(*) AS so_nv, AVG(luong) AS luong_tb
FROM nhan_vien
GROUP BY phong_ban;
```

Kết quả:

| phong_ban  | so_nv | luong_tb   |
|------------|-------|------------|
| Kinh doanh | 2     | 15000000   |
| Kỹ thuật   | 2     | 19000000   |
| Nhân sự    | 1     | 12000000   |

---

## 5. Truy vấn con (Subquery)

**Cấu trúc tổng quát:**

```sql
SELECT cot1, cot2
FROM ten_bang
WHERE cot TOAN_TU (
    SELECT ...
    FROM ten_bang_khac
    WHERE dieu_kien
);
```

**Giải thích:**
- Subquery (truy vấn con) là một câu `SELECT` được đặt lồng bên trong một câu `SELECT` khác (gọi là truy vấn ngoài). Truy vấn con luôn được thực thi trước để tạo ra dữ liệu trung gian, sau đó truy vấn ngoài mới dùng kết quả đó để tiếp tục xử lý.
- `TOAN_TU` là toán tử nối kết quả subquery với truy vấn ngoài, tùy vào subquery trả về gì:
  - Trả về **một giá trị duy nhất** → dùng toán tử so sánh: `=`, `>`, `<`...
  - Trả về **một danh sách giá trị** → dùng `IN`
  - Subquery cũng có thể đặt trong `FROM` (đóng vai trò một bảng tạm) thay vì trong `WHERE`.
- Nếu subquery có tham chiếu đến cột của truy vấn ngoài (ví dụ `WHERE bang_trong.cot = bang_ngoai.cot`), nó được gọi là **correlated subquery** — loại này sẽ chạy lại một lần cho mỗi dòng của truy vấn ngoài, thay vì chỉ chạy một lần duy nhất.

**Ví dụ: Subquery trả về danh sách (dùng với IN):**

**Employee**

| name | department_id |
| ---- | ------------: |
| An   |             1 |
| Bình |             2 |
| Chi  |             3 |

**Department**

| department_id | department_name |
| ------------: | --------------- |
|             1 | IT              |
|             2 | HR              |


```sql
--- Muốn tìm nhân viên thuộc phòng ban tồn tại trong bảng Department.
SELECT name
FROM Employee
WHERE department_id IN (
    SELECT department_id
    FROM Department
);
```

---

## 6. Thứ tự thực thi logic của truy vấn SQL

**Cấu trúc tổng quát (thứ tự viết câu lệnh):**

```sql
SELECT cot1, cot2
FROM ten_bang
JOIN ...
WHERE dieu_kien
GROUP BY cot_nhom
HAVING dieu_kien_nhom
ORDER BY cot_sap_xep
LIMIT so_dong;
```

**Giải thích:** Đây là thứ tự **viết** câu lệnh, nhưng cơ sở dữ liệu lại **thực thi** theo một trình tự khác — hiểu đúng trình tự này giúp giải thích được nhiều "quy tắc lạ" trong SQL (ví dụ vì sao không dùng được alias trong `WHERE`).

**Thứ tự thực thi thực tế:**

```
1. FROM        → Xác định bảng nguồn dữ liệu
2. JOIN / ON    → Kết hợp các bảng lại với nhau
3. WHERE        → Lọc từng dòng dữ liệu thô
4. GROUP BY     → Gom nhóm các dòng đã lọc
5. HAVING       → Lọc trên kết quả đã nhóm
6. SELECT       → Chọn cột / tính toán biểu thức, đặt alias (AS)
7. DISTINCT     → Loại bỏ dòng trùng lặp
8. ORDER BY     → Sắp xếp kết quả
9. LIMIT/OFFSET → Giới hạn số dòng trả về
```

### Ví dụ minh họa toàn bộ luồng thực thi

```sql
SELECT phong_ban, COUNT(*) AS so_nv
FROM nhan_vien
WHERE tuoi > 25
GROUP BY phong_ban
HAVING COUNT(*) >= 2
ORDER BY so_nv DESC
LIMIT 5;
```

Diễn giải theo đúng thứ tự thực thi:

1. **FROM** `nhan_vien` → lấy toàn bộ 5 dòng dữ liệu gốc.
2. **WHERE** `tuoi > 25` → loại bỏ Lê Chi (25 tuổi), còn lại 4 dòng.
3. **GROUP BY** `phong_ban` → gom thành 3 nhóm: Kinh doanh (2), Kỹ thuật (1), Nhân sự (1).
4. **HAVING** `COUNT(*) >= 2` → chỉ giữ nhóm "Kinh doanh" (2 người).
5. **SELECT** → chọn `phong_ban` và tính `COUNT(*)`.
6. **ORDER BY** `so_nv DESC` → sắp xếp giảm dần (ở đây chỉ có 1 dòng nên không thay đổi).
7. **LIMIT 5** → giữ tối đa 5 dòng kết quả.

Kết quả cuối cùng:

| phong_ban  | so_nv |
|------------|-------|
| Kinh doanh | 2     |

**Vì sao thứ tự này quan trọng?**

Vì `WHERE` chạy **trước** `SELECT`, nên không thể dùng alias được đặt trong `SELECT` để lọc trong `WHERE`. Ngược lại, `ORDER BY` chạy **sau** `SELECT` nên hoàn toàn có thể dùng alias đó.

```sql
-- LỖI: không dùng được alias trong WHERE
SELECT luong * 1.1 AS luong_moi
FROM nhan_vien
WHERE luong_moi > 20000000;  -- ❌ Sai vì WHERE chạy trước khi luong_moi được tạo ra

-- ĐÚNG: dùng alias trong ORDER BY
SELECT luong * 1.1 AS luong_moi
FROM nhan_vien
ORDER BY luong_moi DESC;  -- ✅ Hợp lệ vì ORDER BY chạy sau SELECT
```

---

## Tổng kết nhanh

| Chủ đề            | Từ khóa chính                                            |
|--------------------|-----------------------------------------------------------|
| Thao tác dữ liệu   | `SELECT`, `INSERT`, `UPDATE`, `DELETE`                    |
| Alias & lọc trùng  | `AS`, `DISTINCT`                                          |
| Lọc dữ liệu         | `WHERE` (trước nhóm), `HAVING` (sau nhóm)                 |
| Kết hợp dữ liệu     | `JOIN` (theo cột), `UNION` (theo dòng)                    |
| Tổng hợp            | `COUNT`, `SUM`, `AVG`, `GROUP BY`                         |
| Truy vấn lồng        | `Subquery` trong `WHERE`, `FROM`, `IN`, correlated subquery |
| Thứ tự thực thi     | `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT` |