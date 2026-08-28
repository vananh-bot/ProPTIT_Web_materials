# [BUỔI 3] SQL cơ bản


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

Dùng để lấy dữ liệu từ một hoặc nhiều bảng.

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

```sql
INSERT INTO nhan_vien (id, ho_ten, phong_ban, luong, tuoi)
VALUES (6, 'Vũ Giang', 'Marketing', 13000000, 27);
```

> Sau lệnh này, bảng `nhan_vien` sẽ có thêm 1 dòng dữ liệu với `id = 6`.

### 1.3. UPDATE — Cập nhật dữ liệu

```sql
-- Tăng lương 10% cho nhân viên phòng Kỹ thuật
UPDATE nhan_vien
SET luong = luong * 1.1
WHERE phong_ban = 'Kỹ thuật';
```

**Lưu ý quan trọng:** Luôn có `WHERE` khi UPDATE, nếu không toàn bộ các dòng trong bảng sẽ bị cập nhật.

### 1.4. DELETE — Xóa dữ liệu

```sql
-- Xóa nhân viên có id = 6
DELETE FROM nhan_vien
WHERE id = 6;
```

**Lưu ý:** Tương tự UPDATE, thiếu `WHERE` sẽ xóa toàn bộ dữ liệu trong bảng.

### 1.5. Từ khóa AS — Đặt bí danh (alias)

Dùng để đổi tên cột hoặc bảng trong kết quả trả về, giúp dễ đọc hơn.

```sql
SELECT ho_ten AS "Họ và tên", luong AS "Lương (VNĐ)"
FROM nhan_vien AS nv;
```

Kết quả:

| Họ và tên  | Lương (VNĐ) |
|------------|-------------|
| Nguyễn An  | 15000000    |

### 1.6. DISTINCT — Loại bỏ giá trị trùng lặp

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

`WHERE` áp dụng cho **từng dòng dữ liệu gốc**, thực thi trước khi nhóm (GROUP BY).

```sql
-- Lấy nhân viên có lương >= 15 triệu
SELECT ho_ten, luong
FROM nhan_vien
WHERE luong >= 15000000;
```

Có thể kết hợp nhiều điều kiện:

```sql
SELECT ho_ten
FROM nhan_vien
WHERE phong_ban = 'Kỹ thuật' AND tuoi < 30;
```

### 2.2. HAVING — Lọc sau khi nhóm

`HAVING` dùng để lọc trên **kết quả đã được nhóm** (thường đi kèm hàm tổng hợp như SUM, COUNT...).

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

```sql
SELECT nv.ho_ten, pb.ten_pb
FROM nhan_vien nv
INNER JOIN phong_ban pb ON nv.phong_ban = pb.ten_pb;
```

Kết quả: chỉ những nhân viên có phòng ban tồn tại trong bảng `phong_ban` mới xuất hiện (phòng "Marketing" không có nhân viên nên không ảnh hưởng ở đây).

### 3.2. LEFT JOIN — Giữ toàn bộ bảng bên trái

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

`UNION` gộp theo chiều dọc, yêu cầu 2 truy vấn có cùng số cột và kiểu dữ liệu tương ứng. `UNION` tự loại bỏ trùng lặp, còn `UNION ALL` thì giữ nguyên tất cả (kể cả trùng).

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

**Quy tắc quan trọng:** Mọi cột xuất hiện trong `SELECT` mà không nằm trong hàm tổng hợp thì bắt buộc phải có trong `GROUP BY`.

---

## 5. Truy vấn con (Subquery)

### 5.1. Subquery trong WHERE

```sql
-- Lấy nhân viên có lương cao hơn mức lương trung bình
SELECT ho_ten, luong
FROM nhan_vien
WHERE luong > (SELECT AVG(luong) FROM nhan_vien);
```

Truy vấn con `(SELECT AVG(luong) FROM nhan_vien)` được tính trước, trả về giá trị `16000000`, sau đó truy vấn ngoài sẽ lọc các nhân viên có lương lớn hơn giá trị đó.

### 5.2. Subquery trong FROM (bảng tạm)

```sql
SELECT phong_ban, luong_tb
FROM (
    SELECT phong_ban, AVG(luong) AS luong_tb
    FROM nhan_vien
    GROUP BY phong_ban
) AS bang_tam
WHERE luong_tb > 14000000;
```

### 5.3. Subquery với IN

```sql
-- Lấy nhân viên thuộc các phòng ban có trưởng phòng
SELECT ho_ten
FROM nhan_vien
WHERE phong_ban IN (
    SELECT ten_pb FROM phong_ban WHERE truong_phong IS NOT NULL
);
```

### 5.4. Correlated Subquery (truy vấn con tương quan)

Loại subquery mà mỗi dòng của truy vấn ngoài sẽ chạy lại truy vấn con một lần, vì subquery tham chiếu đến cột của truy vấn ngoài.

```sql
-- Lấy nhân viên có lương cao nhất trong phòng ban của mình
SELECT ho_ten, phong_ban, luong
FROM nhan_vien nv1
WHERE luong = (
    SELECT MAX(luong)
    FROM nhan_vien nv2
    WHERE nv2.phong_ban = nv1.phong_ban
);
```

---

## 6. Thứ tự thực thi logic của truy vấn SQL

Mặc dù ta **viết** SQL theo thứ tự `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY`, nhưng cơ sở dữ liệu lại **thực thi** theo thứ tự khác:

```
1. FROM        → Xác định bảng nguồn dữ liệu (bao gồm JOIN)
2. JOIN / ON    → Kết hợp các bảng lại với nhau
3. WHERE        → Lọc từng dòng dữ liệu thô
4. GROUP BY     → Gom nhóm các dòng đã lọc
5. HAVING       → Lọc trên kết quả đã nhóm
6. SELECT       → Chọn cột / tính toán biểu thức, alias (AS)
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
- Đây là lý do vì sao ta **không thể dùng alias đặt trong `SELECT` để lọc trong `WHERE`** (vì `WHERE` chạy trước `SELECT`), nhưng **có thể dùng alias đó trong `ORDER BY`** (vì `ORDER BY` chạy sau `SELECT`).

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

