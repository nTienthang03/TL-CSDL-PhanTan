# TL-CSDL-PhanTan
code tạo SQL
````
/* =========================================================
   ĐỀ TÀI: QUẢN LÝ BÁN VÉ MÁY BAY HN - DN - HCM
   KỸ THUẬT: PHÂN MẢNH NGANG THEO NƠI ĐI
   Chạy từng phần đúng máy SQL tương ứng
========================================================= */


/* =========================================================
   PHẦN 1: CHẠY TRÊN CẢ 3 MÁY SQL
========================================================= */

CREATE DATABASE QuanLyVeMayBay;
GO

USE QuanLyVeMayBay;
GO


/* =========================================================
   SQL1 - HÀ NỘI
========================================================= */

USE QuanLyVeMayBay;
GO

CREATE TABLE ChuyenBay_HN (
    MaChuyenBay VARCHAR(10) PRIMARY KEY,
    HangBay NVARCHAR(50) NOT NULL,
    NoiDi NVARCHAR(50) NOT NULL,
    NoiDen NVARCHAR(50) NOT NULL,
    NgayBay DATE NOT NULL,
    GioBay TIME NOT NULL,
    GioDen TIME NOT NULL,
    TongSoGhe INT NOT NULL,
    SoGheConLai INT NOT NULL,
    GiaVe DECIMAL(18,2) NOT NULL,
    TrangThai NVARCHAR(30) NOT NULL,

    CONSTRAINT CK_HN_SoGhe CHECK (SoGheConLai >= 0 AND SoGheConLai <= TongSoGhe),
    CONSTRAINT CK_HN_NoiDi CHECK (NoiDi = N'Hà Nội')
);
GO

INSERT INTO ChuyenBay_HN VALUES
('VN101', N'Vietnam Airlines', N'Hà Nội', N'Đà Nẵng', '2026-06-01', '08:00', '09:20', 100, 100, 1200000, N'Còn vé'),
('VN102', N'Vietjet Air', N'Hà Nội', N'TP.HCM', '2026-06-01', '10:30', '12:40', 120, 120, 1500000, N'Còn vé'),
('VN103', N'Bamboo Airways', N'Hà Nội', N'Đà Nẵng', '2026-06-02', '14:00', '15:20', 90, 90, 1100000, N'Còn vé');
GO

SELECT * FROM ChuyenBay_HN;
GO


/* =========================================================
   SQL2 - ĐÀ NẴNG
========================================================= */

USE QuanLyVeMayBay;
GO

CREATE TABLE ChuyenBay_DN (
    MaChuyenBay VARCHAR(10) PRIMARY KEY,
    HangBay NVARCHAR(50) NOT NULL,
    NoiDi NVARCHAR(50) NOT NULL,
    NoiDen NVARCHAR(50) NOT NULL,
    NgayBay DATE NOT NULL,
    GioBay TIME NOT NULL,
    GioDen TIME NOT NULL,
    TongSoGhe INT NOT NULL,
    SoGheConLai INT NOT NULL,
    GiaVe DECIMAL(18,2) NOT NULL,
    TrangThai NVARCHAR(30) NOT NULL,

    CONSTRAINT CK_DN_SoGhe CHECK (SoGheConLai >= 0 AND SoGheConLai <= TongSoGhe),
    CONSTRAINT CK_DN_NoiDi CHECK (NoiDi = N'Đà Nẵng')
);
GO

INSERT INTO ChuyenBay_DN VALUES
('VN201', N'Vietnam Airlines', N'Đà Nẵng', N'Hà Nội', '2026-06-01', '07:30', '08:50', 100, 100, 1200000, N'Còn vé'),
('VN202', N'Vietjet Air', N'Đà Nẵng', N'TP.HCM', '2026-06-01', '11:00', '12:30', 95, 95, 1300000, N'Còn vé'),
('VN203', N'Bamboo Airways', N'Đà Nẵng', N'Hà Nội', '2026-06-02', '16:00', '17:20', 80, 80, 1150000, N'Còn vé');
GO

SELECT * FROM ChuyenBay_DN;
GO


/* =========================================================
   SQL3 - TP.HCM
========================================================= */

USE QuanLyVeMayBay;
GO

CREATE TABLE ChuyenBay_HCM (
    MaChuyenBay VARCHAR(10) PRIMARY KEY,
    HangBay NVARCHAR(50) NOT NULL,
    NoiDi NVARCHAR(50) NOT NULL,
    NoiDen NVARCHAR(50) NOT NULL,
    NgayBay DATE NOT NULL,
    GioBay TIME NOT NULL,
    GioDen TIME NOT NULL,
    TongSoGhe INT NOT NULL,
    SoGheConLai INT NOT NULL,
    GiaVe DECIMAL(18,2) NOT NULL,
    TrangThai NVARCHAR(30) NOT NULL,

    CONSTRAINT CK_HCM_SoGhe CHECK (SoGheConLai >= 0 AND SoGheConLai <= TongSoGhe),
    CONSTRAINT CK_HCM_NoiDi CHECK (NoiDi = N'TP.HCM')
);
GO

INSERT INTO ChuyenBay_HCM VALUES
('VN301', N'Vietnam Airlines', N'TP.HCM', N'Hà Nội', '2026-06-01', '09:00', '11:10', 150, 150, 1600000, N'Còn vé'),
('VN302', N'Vietjet Air', N'TP.HCM', N'Đà Nẵng', '2026-06-01', '13:00', '14:30', 110, 110, 1350000, N'Còn vé'),
('VN303', N'Bamboo Airways', N'TP.HCM', N'Hà Nội', '2026-06-02', '18:00', '20:10', 130, 130, 1550000, N'Còn vé');
GO

SELECT * FROM ChuyenBay_HCM;
GO
`````
----------------------
# Thực hiện yêu cầu 
```
1. Linked Server
2. Truy vấn phân tán
3. Transaction đặt vé
4. Distributed Transaction
5. Điều khiển tương tranh
6. Deadlock demo
 ```

# 1. KHÁI NIỆM GIAO DỊCH (TRANSACTION)

# Tình huống thực tế

Ngày lễ 30/4, khách hàng:

```text id="qj5oqd"
Nguyễn Văn A
```

đến quầy vé tại:

```text id="7kw1qn"
Hà Nội (SQL1)
```

đặt chuyến bay:

```text id="v4g6r0"
VN301
TP.HCM → Hà Nội
```

Dữ liệu chuyến bay nằm tại:

```text id="ji2jvl"
SQL3 – TP.HCM
```

---

# Hệ thống phải làm gì?

## Bước 1

SQL1 kiểm tra chuyến bay tại SQL3:

```sql id="3v6ncb"
SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
WHERE MaChuyenBay = 'VN301';
```

---

## Bước 2

Nếu còn ghế:

```text id="5h9w3u"
SoGheConLai > 0
```

thì:

* trừ ghế,
* tạo vé cho khách.

---

# SQL thực hiện

```sql id="gkqv2i"
UPDATE [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
SET SoGheConLai = SoGheConLai - 1
WHERE MaChuyenBay = 'VN301';

INSERT INTO [SQL3].QuanLyVeMayBay.dbo.VeMayBay_HCM
(MaChuyenBay, TenKhachHang)
VALUES ('VN301', N'Nguyễn Văn A');
```

---

# Làm được gì?

✅ Đặt vé từ Hà Nội
✅ Cập nhật dữ liệu tại TP.HCM
✅ Đồng bộ số ghế

---

# Kết quả

```text id="k6n11s"
Số ghế:
150 → 149
```

Xuất hiện vé mới:

```text id="w1l5g8"
Nguyễn Văn A
```

---

# Kết luận

```text id="mh3ik1"
Transaction là tập hợp nhiều thao tác xử lý dữ liệu
được thực hiện như một công việc thống nhất.
```

---

# VÍ DỤ GIAO DỊCH KHÔNG THÀNH CÔNG (ROLLBACK)

# Tình huống thực tế

Khách hàng:

```text id="x7d6eh"
Trần Văn B
```

đặt vé:

```text id="y3w7c5"
VN301
```

Trong lúc xử lý:

* hệ thống trừ ghế thành công,
* nhưng lỗi mạng xảy ra trước khi tạo vé.

---

# Nếu không dùng Transaction

## Điều gì xảy ra?

Hệ thống:

* đã trừ ghế,
* nhưng không tạo được vé.

---

# Hậu quả thực tế

```text id="9j0s0s"
Khách không có vé
nhưng số ghế vẫn bị mất
```

=> dữ liệu sai.

---

# Hệ thống phải làm gì?

Rollback toàn bộ giao dịch.

---

# SQL thực hiện

```sql id="x4gf1d"
BEGIN TRAN;

BEGIN TRY

    UPDATE [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
    SET SoGheConLai = SoGheConLai - 1
    WHERE MaChuyenBay = 'VN301';

    -- Lỗi giả lập
    INSERT INTO BangKhongTonTai
    VALUES ('ERROR');

    COMMIT;

END TRY

BEGIN CATCH

    ROLLBACK;

    PRINT N'Giao dịch thất bại - Đã rollback';

END CATCH;
```

---

# Làm được gì?

✅ Hủy toàn bộ giao dịch khi xảy ra lỗi
✅ Trả lại số ghế ban đầu
✅ Tránh sai dữ liệu

---

# Kết quả

```text id="z37kag"
Số ghế không thay đổi
vẫn giữ nguyên:
149
```

Không tạo vé lỗi.

---

# Kết luận

```text id="s8f1sd"
Nếu một bước trong transaction thất bại,
toàn bộ giao dịch sẽ bị hủy (ROLLBACK)
để đảm bảo dữ liệu chính xác.
```


