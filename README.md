# ĐỀ TÀI NHÓM 7

# QUẢN LÝ BÁN VÉ MÁY BAY TRONG HỆ CƠ SỞ DỮ LIỆU PHÂN TÁN

## Nội dung nghiên cứu

- Khái niệm giao dịch (Transaction)

- Tính chất ACID

- Giao dịch phân tán (Distributed Transaction)

- Điều khiển tương tranh (Concurrency Control)

- Vấn đề deadlock trong hệ phân tán

- Ứng dụng trong hệ thống quản lý bán vé máy bay


## 1 Tạo Giao dịch thành công ( Transaction)
### ( Dùng làm ví dụ :	Giao dịch phân tán) 
| Nội dung | Giá trị |
|---|---|
| Nơi thực hiện | Hà Nội |
| Server dữ liệu | SQL3 - TP.HCM |
| Chuyến bay | VN302 |
| Tuyến bay | TP.HCM → Đà Nẵng |
| Khách hàng | Nguyễn Văn B |
| Ghế đặt | GHE20 |

---

# Quy trình đặt vé

## 1. Kiểm tra thông tin trước khi đặt

```sql
SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
WHERE MaChuyenBay = 'VN302';

SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.GheNgoi_HCM
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE20';

SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.VeMayBay_HCM
WHERE MaChuyenBay = 'VN302';
````

Mục đích:

* Kiểm tra chuyến bay có tồn tại hay không
* Kiểm tra ghế GHE20 còn trống
* Kiểm tra danh sách vé hiện tại

---

## 2. Đặt ghế cho khách

```sql
UPDATE [SQL3].QuanLyVeMayBay.dbo.GheNgoi_HCM
SET TrangThai = N'Đã đặt'
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE20'
  AND TrangThai = N'Trống';
```

Mục đích:

* Chuyển trạng thái ghế từ `Trống` sang `Đã đặt`

---

## 3. Giảm số ghế còn lại của chuyến bay

```sql
UPDATE [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
SET SoGheConLai = SoGheConLai - 1
WHERE MaChuyenBay = 'VN302'
  AND SoGheConLai > 0;
```

Mục đích:

* Cập nhật số lượng ghế còn lại sau khi khách đặt vé

---

## 4. Tạo vé máy bay

```sql
INSERT INTO [SQL3].QuanLyVeMayBay.dbo.VeMayBay_HCM
(
    MaChuyenBay,
    TenKhachHang,
    MaGhe
)
VALUES
(
    'VN302',
    N'Nguyễn Văn B',
    'GHE20'
);
```

Mục đích:

* Lưu thông tin vé của khách hàng vào hệ thống

---

## 5. Kiểm tra sau khi đặt vé

```sql
SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
WHERE MaChuyenBay = 'VN302';

SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.GheNgoi_HCM
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE20';

SELECT *
FROM [SQL3].QuanLyVeMayBay.dbo.VeMayBay_HCM
WHERE MaChuyenBay = 'VN302';
```

Mục đích:

* Kiểm tra ghế đã chuyển sang trạng thái `Đã đặt`
* Kiểm tra số ghế còn lại đã giảm
* Kiểm tra vé đã được tạo thành công

---

# Giao dịch hoàn chỉnh (Transaction)

```sql
BEGIN TRANSACTION;

BEGIN TRY

    UPDATE [SQL3].QuanLyVeMayBay.dbo.GheNgoi_HCM
    SET TrangThai = N'Đã đặt'
    WHERE MaChuyenBay = 'VN302'
      AND MaGhe = 'GHE20'
      AND TrangThai = N'Trống';

    UPDATE [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
    SET SoGheConLai = SoGheConLai - 1
    WHERE MaChuyenBay = 'VN302'
      AND SoGheConLai > 0;

    INSERT INTO [SQL3].QuanLyVeMayBay.dbo.VeMayBay_HCM
    (
        MaChuyenBay,
        TenKhachHang,
        MaGhe
    )
    VALUES
    (
        'VN302',
        N'Nguyễn Văn B',
        'GHE20'
    );

    COMMIT TRANSACTION;

    PRINT N'Đặt vé thành công';

END TRY

BEGIN CATCH

    ROLLBACK TRANSACTION;

    PRINT N'Đặt vé thất bại';
    PRINT ERROR_MESSAGE();

END CATCH;
```

---

# Kết quả đạt được
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0e7e8dfe-5f00-4c03-b2b7-8812a128a1fb" />

Sau khi giao dịch thành công:

* Ghế `GHE20` chuyển sang trạng thái `Đã đặt`
* Số ghế còn lại của chuyến bay giảm đi 1
* Hệ thống tạo thêm một vé mới cho khách hàng `Nguyễn Văn B`

---

# Ý nghĩa của Transaction

Transaction giúp đảm bảo:

* Tính toàn vẹn dữ liệu
* Không bị mất đồng bộ dữ liệu
* Nếu xảy ra lỗi thì toàn bộ thao tác sẽ được hoàn tác (`ROLLBACK`)
* Tránh tình trạng:

  * Trừ ghế nhưng chưa tạo vé
  * Tạo vé nhưng chưa cập nhật ghế
  * Nhiều người đặt cùng một ghế

```
```
# PHẦN 2. TÍNH CHẤT ACID CỦA GIAO DỊCH

## Đề tài: Quản lý bán vé máy bay trong hệ cơ sở dữ liệu phân tán

---

# Tổng quan ACID trong hệ thống Quản lý bán vé máy bay

ACID là bốn tính chất quan trọng nhất của một giao dịch trong cơ sở dữ liệu, giúp đảm bảo tính toàn vẹn và độ tin cậy của hệ thống, đặc biệt trong môi trường phân tán.

| Tính chất | Ý nghĩa | Ý nghĩa trong bán vé máy bay |
|---|---|---|
| **A - Atomicity** | Tính nguyên tử | Toàn bộ quy trình đặt vé phải thành công hoặc hoàn toàn thất bại |
| **C - Consistency** | Tính nhất quán | Dữ liệu sau giao dịch phải thỏa mãn tất cả ràng buộc (ghế không âm, trạng thái đồng bộ) |
| **I - Isolation** | Tính cô lập | Nhiều khách hàng đặt vé cùng lúc không gây xung đột dữ liệu |
| **D - Durability** | Tính bền vững | Sau khi xác nhận vé, dữ liệu phải được lưu vĩnh viễn dù server crash |

---

# 2.1. A - Atomicity (Tính Nguyên tử)

## Mô tả

Toàn bộ các thao tác trong giao dịch được thực hiện như một đơn vị duy nhất:

```text
Hoặc tất cả thành công
Hoặc tất cả bị hủy bỏ
```

---

# Test Case minh họa

```sql
-- TEST ATOMICITY: Đặt vé nhưng lỗi ở bước cuối cùng

-- ====================== TEST ATOMICITY ======================
-- BEFORE
SELECT '=== TRUOC GIAO DICH ===' AS TrangThai;
SELECT SoGheConLai, TrangThai FROM ChuyenBay_HCM WHERE MaChuyenBay = 'VN302';
SELECT TrangThai FROM GheNgoi_HCM WHERE MaChuyenBay = 'VN302' AND MaGhe = 'GHE25';

-- ====================== THỰC HIỆN GIAO DỊCH ======================
BEGIN TRANSACTION;
BEGIN TRY
    -- 1. Cập nhật trạng thái ghế
    UPDATE GheNgoi_HCM 
    SET TrangThai = N'Đã đặt'
    WHERE MaChuyenBay = 'VN302' AND MaGhe = 'GHE25' AND TrangThai = N'Trống';

    -- 2. Giảm số ghế còn lại
    UPDATE ChuyenBay_HCM 
    SET SoGheConLai = SoGheConLai - 1 
    WHERE MaChuyenBay = 'VN302';

    -- 3. TẠO LỖI CỐ Ý (đây là dòng quan trọng)
    RAISERROR('Lỗi thử nghiệm để kiểm tra Atomicity', 16, 1);  

    -- Nếu không muốn dùng RAISERROR, dùng cách này:
    -- INSERT INTO VeMayBay_HCM (MaChuyenBay, TenKhachHang, MaGhe)
    -- VALUES ('VN302', N'Test', NULL);  -- Vi phạm NOT NULL nếu có

    COMMIT TRANSACTION;
    PRINT N'=== THÀNH CÔNG ===';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT N'=== ATOMICITY: ĐÃ ROLLBACK TOÀN BỘ GIAO DỊCH ===';
    PRINT N'Lỗi: ' + ERROR_MESSAGE();
END CATCH
GO

-- AFTER
SELECT '=== SAU GIAO DICH (Atomicity) ===' AS TrangThai;
SELECT SoGheConLai, TrangThai FROM ChuyenBay_HCM WHERE MaChuyenBay = 'VN302';
SELECT TrangThai FROM GheNgoi_HCM WHERE MaChuyenBay = 'VN302' AND MaGhe = 'GHE25';
```

---
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/63e1c8b1-3b87-48f6-82be-e97c0dfdffaa" />

# Kết quả mong đợi
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d595b237-e9d6-447c-8211-8611de2d6c73" />


```text
Nếu một bước trong transaction bị lỗi,
toàn bộ giao dịch sẽ bị rollback.
Điều này đảm bảo dữ liệu không bị trạng thái nửa thành công.
```

---

# 2.2. C - Consistency (Tính Nhất quán)

## Mô tả

Giao dịch phải đưa cơ sở dữ liệu:

```text
Từ trạng thái nhất quán này
Sang trạng thái nhất quán khác
```

Dữ liệu sau giao dịch luôn phải hợp lệ.

---

# Test Case minh họa

```sql
-- TEST CONSISTENCY: Thử đặt vé khi hết ghế

BEGIN TRANSACTION;

BEGIN TRY

    -- Giả lập chuyến bay đã hết ghế
    UPDATE ChuyenBay_HCM
    SET SoGheConLai = 0
    WHERE MaChuyenBay = 'VN302';

    -- Thử đặt ghế
    UPDATE GheNgoi_HCM
    SET TrangThai = N'Đã đặt'
    WHERE MaChuyenBay = 'VN302'
      AND MaGhe = 'GHE30';

    -- Giảm số ghế còn lại
    UPDATE ChuyenBay_HCM
    SET SoGheConLai = SoGheConLai - 1
    WHERE MaChuyenBay = 'VN302';

    COMMIT TRANSACTION;

END TRY

BEGIN CATCH

    ROLLBACK TRANSACTION;

    PRINT N'=== CONSISTENCY: ĐÃ ROLLBACK do vi phạm ràng buộc ===';

    PRINT ERROR_MESSAGE();

END CATCH
```

---

# Ràng buộc khuyến nghị

```sql
ALTER TABLE ChuyenBay_HCM
ADD CONSTRAINT CK_SoGheConLai
CHECK (SoGheConLai >= 0);
```

---

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f9d92f9e-c0db-470d-b58c-5c6db74c17eb" />


# Kết luận Consistency

```text
Consistency giúp dữ liệu luôn đúng và hợp lệ.

Các ràng buộc như:
- CHECK
- PRIMARY KEY
- FOREIGN KEY

giúp bảo vệ tính nhất quán của hệ thống.
```

---

# 2.3. I - Isolation (Tính Cô lập)

## Mục này e dùng để  Minh họa điều khiển tương tranh luôn ạ : Ở phần ACID, demo này được dùng để minh họa tính Isolation.
Ở phần Điều khiển tương tranh, demo này được dùng để giải thích cơ chế khóa UPDLOCK, HOLDLOCK giúp SQL Server đảm bảo Isolation.
---

# Mục tiêu

Minh họa tính chất:

```text
Isolation - Tính cô lập
```

Trong hệ thống phân tán:

- SQL1 → Hà Nội
- SQL3 → TP.HCM

---

# Tình huống mô phỏng

```text
Máy TP.HCM đang cập nhật chuyến VN302
và giữ khóa dữ liệu trong 8 giây.

Trong lúc đó,
máy Hà Nội cũng muốn cập nhật cùng dữ liệu
thông qua Linked Server.
```

---

# Mục tiêu kiểm tra

```text
Transaction thứ hai phải chờ
cho đến khi transaction thứ nhất COMMIT.
```

Điều này giúp tránh:

- Lost Update
- Dirty Write
- Sai lệch dữ liệu

---

# TAB 1 — Chạy trên SQL3 / TP.HCM

```sql
/* =====================================================
   ISOLATION - TAB 1
   Chạy trên SQL3 - TP.HCM
   Giữ khóa chuyến VN302 trong 8 giây
===================================================== */

USE QuanLyVeMayBay;
GO

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN TRANSACTION;

UPDATE ChuyenBay_HCM WITH (UPDLOCK, HOLDLOCK)
SET SoGheConLai = SoGheConLai - 1
WHERE MaChuyenBay = 'VN302';

PRINT N'SQL3 đang giữ khóa chuyến VN302';

WAITFOR DELAY '00:00:08';

COMMIT TRANSACTION;

PRINT N'SQL3 đã COMMIT, giải phóng khóa';
```

---

# Giải thích Tab 1

| Thành phần | Ý nghĩa |
|---|---|
| `REPEATABLE READ` | Đảm bảo dữ liệu đang đọc không bị thay đổi |
| `UPDLOCK` | Khóa cập nhật dữ liệu |
| `HOLDLOCK` | Giữ khóa đến hết transaction |
| `WAITFOR DELAY` | Giả lập transaction chạy chậm |

---

# TAB 2 — Chạy trên SQL1 / Hà Nội

## Chạy ngay khi Tab 1 đang chờ 8 giây

```sql
/* =====================================================
   ISOLATION - TAB 2
   Chạy trên SQL1 - Hà Nội
   Cập nhật dữ liệu VN302 nằm ở SQL3
===================================================== */

UPDATE [SQL3].QuanLyVeMayBay.dbo.ChuyenBay_HCM
SET SoGheConLai = SoGheConLai - 1
WHERE MaChuyenBay = 'VN302';

PRINT N'SQL1 đã cập nhật xong chuyến VN302 trên SQL3';
```

---

# Kết quả 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/010ec4a9-546f-4237-95f6-795fc63aaa3f" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2efd1c98-f8db-4be7-9a0c-e52b252e7f14" />

# Kết luận

```text
Isolation giúp nhiều khách hàng đặt vé cùng lúc
mà vẫn đảm bảo dữ liệu chính xác.
```
```text
Isolation đảm bảo khi một giao dịch đang cập nhật dữ liệu,
giao dịch khác dù chạy từ máy Hà Nội
cũng không được sửa ngay dữ liệu đó.

SQL Server bắt giao dịch thứ hai
phải chờ đến khi giao dịch thứ nhất COMMIT.
```

# 2.4. D - Durability (Tính Bền vững)

Có. Test **Durability** không nhất thiết phải reset SQL Server.
Bạn có thể test bằng cách:

```text
1. COMMIT giao dịch
2. Đóng tab query hiện tại
3. Mở tab query mới
4. SELECT lại dữ liệu
```

Nếu dữ liệu vẫn còn thì chứng minh được:

```text
Dữ liệu đã COMMIT được lưu bền vững
```

---

## Code COMMIT kiểm tra Durability

Chạy trên **SQL3 - TP.HCM**:

```sql
USE QuanLyVeMayBay;
GO

BEGIN TRANSACTION;

BEGIN TRY

    INSERT INTO VeMayBay_HCM
    (
        MaChuyenBay,
        TenKhachHang,
        MaGhe,
        NgayDat
    )
    VALUES
    (
        'VN302',
        N'Test Durability',
        'GHE40',
        GETDATE()
    );

    UPDATE GheNgoi_HCM
    SET TrangThai = N'Đã đặt'
    WHERE MaChuyenBay = 'VN302'
      AND MaGhe = 'GHE40';

    COMMIT TRANSACTION;

    PRINT N'ĐÃ COMMIT - Dữ liệu đã được lưu';

END TRY

BEGIN CATCH

    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    PRINT N'LỖI - ĐÃ ROLLBACK';
    PRINT ERROR_MESSAGE();

END CATCH;
```

---

Sau khi chạy xong code trên:

1. Đóng tab query vừa chạy.
2. Mở **New Query** mới.
3. Chạy:

```sql
USE QuanLyVeMayBay;
GO

SELECT *
FROM VeMayBay_HCM
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE40';
```

Nếu vẫn thấy:

```text
Test Durability | VN302 | GHE40
```

thì demo **Durability thành công**.

---
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d7bb631f-869e-4994-b522-06eabbb264c7" />

# PHẦN 3 Giao dịch phân tán đã lấy ví dụ  làm ở phần ## 1. Tạo giao dịch thành công (Transaction)

| Nội dung | Giá trị |
|---|---|
| Nơi thực hiện | SQL1 - Hà Nội |
| Server chứa dữ liệu | SQL3 - TP.HCM |
| Chuyến bay | VN302 |
| Tuyến bay | TP.HCM → Đà Nẵng |
| Khách hàng | Nguyễn Văn B |
| Ghế đặt | GHE20 |
| Mục tiêu | Đặt vé từ Hà Nội cho chuyến bay có dữ liệu lưu tại TP.HCM |


#  Phần 4 Điều khiển tương tranh 
## Đã thực hành cùng với  2.3. I - Isolation (Tính Cô lập)

```text
Nhiều khách cùng đặt vé một chuyến bay
→ hệ thống phải kiểm soát ai được cập nhật trước
→ người còn lại phải chờ
```

## Ví dụ thực tế

```text
Khách A đặt ghế GHE20 chuyến VN302
Khách B cũng đặt ghế GHE20 chuyến VN302 cùng lúc
```

## Nếu không có điều khiển tương tranh

```text
Cả A và B có thể cùng đặt thành công một ghế
→ bán trùng ghế
→ sai dữ liệu
```

## Có điều khiển tương tranh

SQL Server sẽ khóa dữ liệu:

```text
Khách A đang đặt GHE20
→ SQL Server khóa ghế GHE20
→ Khách B phải chờ
→ A xong thì B mới được xử lý
```
````

# 5. Vấn đề Deadlock

**Deadlock** là tình trạng hai hoặc nhiều giao dịch giữ tài nguyên của nhau và chờ nhau, làm cho các giao dịch không thể tiếp tục thực hiện.

Trong hệ thống bán vé máy bay, deadlock có thể xảy ra khi nhiều nhân viên hoặc nhiều giao dịch cùng cập nhật các chuyến bay khác nhau cùng lúc.

---

### Ví dụ thực tế

```text
Nhân viên A đang cập nhật chuyến VN301
Nhân viên B đang cập nhật chuyến VN302

Sau đó:

Nhân viên A cần cập nhật tiếp VN302
Nhân viên B cần cập nhật tiếp VN301

Khi đó:

A đang giữ khóa VN301 và chờ VN302
B đang giữ khóa VN302 và chờ VN301

=> Hai giao dịch chờ nhau, không giao dịch nào tiếp tục được.

Nếu xảy ra deadlock
Transaction 1 giữ VN301 → chờ VN302
Transaction 2 giữ VN302 → chờ VN301

Kết quả:

Hai transaction bị kẹt
Hệ thống không thể tiếp tục xử lý nếu không có cơ chế giải quyết
SQL Server xử lý deadlock

SQL Server sẽ tự động phát hiện deadlock và chọn một giao dịch làm nạn nhân.

Giao dịch bị chọn sẽ bị:

ROLLBACK

Giao dịch còn lại tiếp tục thực hiện.

Mở **2 tab query trên SQL3 - TP.HCM**.

---

### Tab 1 — Chạy trên SQL3

```sql
USE QuanLyVeMayBay;
GO

BEGIN TRANSACTION;

UPDATE GheNgoi_HCM
SET TrangThai = N'Đang xử lý'
WHERE MaChuyenBay = 'VN301'
  AND MaGhe = 'GHE30';

PRINT N'Tab 1 đã khóa VN301 - GHE30';

WAITFOR DELAY '00:00:10';

UPDATE GheNgoi_HCM
SET TrangThai = N'Đang xử lý'
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE30';

COMMIT TRANSACTION;

PRINT N'Tab 1 COMMIT';
```

---

### Tab 2 — Chạy trên SQL3

Chạy ngay khi Tab 1 đang chờ 10 giây:

```sql
USE QuanLyVeMayBay;
GO

BEGIN TRANSACTION;

UPDATE GheNgoi_HCM
SET TrangThai = N'Đang xử lý'
WHERE MaChuyenBay = 'VN302'
  AND MaGhe = 'GHE30';

PRINT N'Tab 2 đã khóa VN302 - GHE30';

WAITFOR DELAY '00:00:10';

UPDATE GheNgoi_HCM
SET TrangThai = N'Đang xử lý'
WHERE MaChuyenBay = 'VN301'
  AND MaGhe = 'GHE30';

COMMIT TRANSACTION;

PRINT N'Tab 2 COMMIT';
```

---


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/780ac4e8-0ef0-42f0-b7f8-6f6cded7c095" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/706a3e97-ca4f-42af-825d-7b3c7c4c897e" />

### . Giải thích kết quả

Trong demo:

```text
Tab 1 khóa VN301 - GHE30
Tab 2 khóa VN302 - GHE30
```

Sau đó:

```text
Tab 1 cần VN302 - GHE30 nhưng Tab 2 đang giữ.
Tab 2 cần VN301 - GHE30 nhưng Tab 1 đang giữ.
```

Vì vậy xảy ra vòng chờ:

```text
Tab 1 chờ Tab 2
Tab 2 chờ Tab 1
```

Đây chính là **deadlock**.
