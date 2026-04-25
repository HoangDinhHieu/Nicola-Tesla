# Nicola-Tesla
# BÀI KIỂM TRA SỐ 2 — Hệ Quản Trị Cơ Sở Dữ Liệu SQL Server

## Thông Tin Cá Nhân

| Thông tin | Nội dung |
|---|---|
| **Họ và tên** | *Hoàng Đình Hiếu* |
| **Mã sinh viên** | K235480106025 |
| **Lớp** | *K59.KMT.K01* |
| **Chủ đề** | Quản Lý Thư Viện |
| **Tên Database** | `QuanLyThuVien_K235480106025` |

---

## Yêu Cầu Đầu Bài

Thực hiện xây dựng một hệ thống **quản lý thư viện** hoàn chỉnh trên SQL Server, đáp ứng các tiêu chí cụ thể sau:

- Toàn bộ quá trình thực hiện phải được ghi lại thông qua các **screenshot minh họa**, mỗi bức ảnh đi kèm câu lệnh SQL và chú thích rõ ràng về chức năng, mục đích xử lý và kết quả đạt được.
- Bài tập được nộp dưới dạng **repository GitHub (public)**, bao gồm: tệp `README.md` chứa toàn bộ nội dung và hình ảnh, tệp `baikiemtra2.sql` chứa toàn bộ mã SQL.
- Đánh giá dựa trên: **logic xử lý dữ liệu**, **quy tắc đặt tên (bướu Lạc Đà)**, và **commit history** rõ ràng.
- **Deadline:** 23:59:59 ngày 03 tháng 5 năm 2026.

---

## Giới Thiệu Về Hệ Thống Quản Lý Thư Viện

Xây dựng một hệ thống quản lý thư viện (`QuanLyThuVien_K235480106025`) từ nền tảng SQL Server, bao gồm các chức năng chủ yếu như: quản lý thông tin sách và thể loại, quản lý hồ sơ độc giả, và quản lý các phiếu mượn/trả sách.

Mỗi cuốn sách được phân loại theo thể loại (Văn học, Khoa học kỹ thuật, Lịch sử, Kinh tế, Thiếu nhi), có giá bìa và số lượng tồn kho theo dõi theo thời gian thực. Mỗi độc giả có hồ sơ cá nhân, thẻ thư viện và lịch sử mượn sách. Mỗi phiếu mượn lưu giữ mối quan hệ giữa độc giả và sách, cùng ngày mượn, ngày trả dự kiến và trạng thái.

Toàn bộ bài làm được chia thành **5 phần chính** theo thứ tự tăng độ phức tạp:

1. **Thiết Kế Cơ Sở Dữ Liệu** — khởi tạo các bảng với ràng buộc toàn vẹn và dữ liệu mẫu.
2. **Xây Dựng Function** — các hàm tính toán như tính tiền phạt, tra cứu sách, thống kê hoạt động.
3. **Xây Dựng Stored Procedure** — thủ tục xử lý nghiệp vụ như đăng ký mượn, tính công nợ, báo cáo.
4. **Trigger Xử Lý Nghiệp Vụ** — tự động hóa các tác vụ khi dữ liệu thay đổi.
5. **Cursor và Duyệt Dữ Liệu** — xử lý tuần tự từng bản ghi và so sánh hiệu năng với Set-based SQL.

---

## Phần 1: Khởi Tạo Database và Bảng

### Tạo Cơ Sở Dữ Liệu



<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/135c53b6-1f0a-4871-80b6-bc28eebff47c" />

*Tạo cơ sở dữ liệu `QuanLyThuVien_K235480106025`*

---

### Bảng `[TheLoai]` — Danh mục thể loại sách

- Sử dụng `MaTheLoai` làm **Khóa chính (PK)**, tự động tăng `IDENTITY(1,1)`.
- `TenTheLoai` dùng `NVARCHAR` để hỗ trợ tiếng Việt có dấu.

```sql
CREATE TABLE [TheLoai] (
    [MaTheLoai]   INT           NOT NULL IDENTITY(1,1),
    [TenTheLoai]  NVARCHAR(100) NOT NULL,
    [MoTa]        NVARCHAR(500) NULL,
    CONSTRAINT [PK_TheLoai] PRIMARY KEY ([MaTheLoai])
);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/39ac6613-db3f-4b3c-8858-2edc7355cbac" />


*Tạo bảng TheLoai — danh mục phân loại sách trong thư viện*

---
### Bảng `[Sach]` — Thông tin sách

- Sử dụng `MaSach` làm **Khóa chính (PK)**.
- `MaTheLoai` là **Khóa ngoại (FK)** tham chiếu đến bảng `TheLoai`.
- Kiểu `MONEY` được dùng cho `GiaBan` để tối ưu lưu trữ tiền tệ. Ràng buộc `CHECK` giá phải > 0.
- Ràng buộc `CHECK` năm xuất bản phải hợp lệ, tồn kho không âm.

```sql
CREATE TABLE [Sach] (
    [MaSach]        INT           NOT NULL IDENTITY(1,1),
    [TenSach]       NVARCHAR(300) NOT NULL,
    [TacGia]        NVARCHAR(200) NOT NULL,
    [NhaXuatBan]    NVARCHAR(200) NULL,
    [NamXuatBan]    INT           NULL,
    [GiaBan]        MONEY         NULL,
    [SoLuongTon]    INT           NOT NULL DEFAULT 0,
    [MaTheLoai]     INT           NOT NULL,
    CONSTRAINT [PK_Sach]            PRIMARY KEY ([MaSach]),
    CONSTRAINT [FK_Sach_TheLoai]    FOREIGN KEY ([MaTheLoai]) REFERENCES [TheLoai]([MaTheLoai]),
    CONSTRAINT [CK_Sach_NamXuatBan] CHECK ([NamXuatBan] >= 1000 AND [NamXuatBan] <= 2100),
    CONSTRAINT [CK_Sach_SoLuongTon] CHECK ([SoLuongTon] >= 0)
);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1448f850-6f52-46f2-83c4-ede2e6c353f6" />


*Tạo bảng Sach — lưu trữ toàn bộ thông tin đầu sách trong kho*

---

### Bảng `[DocGia]` — Hồ sơ độc giả

- Có `MaDocGia` làm **Khóa chính (PK)**.
- Ràng buộc `CHECK` giới tính chỉ nhận `'Nam'`, `'Nữ'`, hoặc `'Khác'`.
- Ràng buộc `CHECK` ngày sinh không được ở tương lai.
- Cột `TrangThai` dùng kiểu `BIT`: `1` = đang hoạt động, `0` = đã khóa thẻ.

```sql
CREATE TABLE [DocGia] (
    [MaDocGia]      INT           NOT NULL IDENTITY(1,1),
    [HoTen]         NVARCHAR(150) NOT NULL,
    [GioiTinh]      NVARCHAR(10)  NOT NULL DEFAULT N'Nam',
    [NgaySinh]      DATE          NULL,
    [DiaChi]        NVARCHAR(300) NULL,
    [SoDienThoai]   VARCHAR(15)   NULL,
    [Email]         VARCHAR(150)  NULL,
    [NgayDangKy]    DATE          NOT NULL DEFAULT GETDATE(),
    [TrangThai]     BIT           NOT NULL DEFAULT 1,
    CONSTRAINT [PK_DocGia]          PRIMARY KEY ([MaDocGia]),
    CONSTRAINT [CK_DocGia_GioiTinh] CHECK ([GioiTinh] IN (N'Nam', N'Nữ', N'Khác')),
    CONSTRAINT [CK_DocGia_NgaySinh] CHECK ([NgaySinh] <= GETDATE())
);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7be9870f-a017-4dcd-a1a8-c1c5782ec5db" />


*Tạo bảng DocGia — lưu hồ sơ độc giả đăng ký thẻ thư viện*

---

### Bảng `[PhieuMuon]` — Giao dịch mượn/trả sách

- Có `MaPhieuMuon` làm **Khóa chính (PK)**, tự tăng.
- Có **2 Khóa ngoại (FK)**: `MaDocGia` → `DocGia`, `MaSach` → `Sach`.
- Ràng buộc logic: `NgayTraDuKien` phải lớn hơn hoặc bằng `NgayMuon`.
- `NgayTraThucTe = NULL` có nghĩa là sách **chưa được trả**.
- `TrangThai` có ràng buộc `CHECK` chỉ nhận 3 giá trị hợp lệ.

```sql
CREATE TABLE [PhieuMuon] (
    [MaPhieuMuon]    INT           NOT NULL IDENTITY(1,1),
    [MaDocGia]       INT           NOT NULL,
    [MaSach]         INT           NOT NULL,
    [NgayMuon]       DATE          NOT NULL DEFAULT GETDATE(),
    [NgayTraDuKien]  DATE          NOT NULL,
    [NgayTraThucTe]  DATE          NULL,
    [SoLuongMuon]    INT           NOT NULL DEFAULT 1,
    [TrangThai]      NVARCHAR(20)  NOT NULL DEFAULT N'Đang mượn',
    [GhiChu]         NVARCHAR(500) NULL,
    CONSTRAINT [PK_PhieuMuon]           PRIMARY KEY ([MaPhieuMuon]),
    CONSTRAINT [FK_PhieuMuon_DocGia]    FOREIGN KEY ([MaDocGia]) REFERENCES [DocGia]([MaDocGia]),
    CONSTRAINT [FK_PhieuMuon_Sach]      FOREIGN KEY ([MaSach])   REFERENCES [Sach]([MaSach]),
    CONSTRAINT [CK_PhieuMuon_NgayTra]   CHECK ([NgayTraDuKien] >= [NgayMuon]),
    CONSTRAINT [CK_PhieuMuon_SoLuong]   CHECK ([SoLuongMuon] >= 1),
    CONSTRAINT [CK_PhieuMuon_TrangThai] CHECK ([TrangThai] IN (N'Đang mượn', N'Đã trả', N'Quá hạn'))
);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b9f6e694-c4b2-41b1-9540-c7a67b22b815" />


*Tạo bảng PhieuMuon — ghi lại toàn bộ giao dịch mượn và trả sách, có FK ràng buộc với cả DocGia lẫn Sach*

---

### Bảng `[LichSuSoLuong]` — Nhật ký thay đổi tồn kho

Bảng này **không được thao tác trực tiếp bởi người dùng**. Nó chỉ được ghi dữ liệu tự động bởi **Trigger** mỗi khi có phiếu mượn mới. Mục đích là lưu vết để kiểm toán (audit trail).

```sql
CREATE TABLE [LichSuSoLuong] (
    [MaLichSu]      INT          NOT NULL IDENTITY(1,1),
    [MaSach]        INT          NOT NULL,
    [SoLuongTruoc]  INT          NOT NULL,
    [SoLuongSau]    INT          NOT NULL,
    [ThoiGian]      DATETIME     NOT NULL DEFAULT GETDATE(),
    [HanhDong]      NVARCHAR(50) NOT NULL,
    CONSTRAINT [PK_LichSuSoLuong] PRIMARY KEY ([MaLichSu])
);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/862ac6d6-833e-40a5-aa0d-db6d2e4ba519" />


*Tạo bảng LichSuSoLuong — dùng làm nhật ký tự động, chỉ Trigger mới được phép ghi vào đây*

---

### Chèn Dữ Liệu Mẫu Vào Các Bảng

```sql
INSERT INTO [TheLoai] ([TenTheLoai], [MoTa]) VALUES
(N'Văn học',            N'Tiểu thuyết, truyện ngắn, thơ ca trong và ngoài nước'),
(N'Khoa học kỹ thuật',  N'Sách về công nghệ, lập trình, kỹ thuật'),
(N'Lịch sử',            N'Sách lịch sử Việt Nam và thế giới'),
(N'Kinh tế',            N'Sách về kinh doanh, tài chính, quản trị'),
(N'Thiếu nhi',          N'Sách dành cho trẻ em và thanh thiếu niên');

INSERT INTO [Sach] ([TenSach],[TacGia],[NhaXuatBan],[NamXuatBan],[GiaBan],[SoLuongTon],[MaTheLoai]) VALUES
(N'Số đỏ',               N'Vũ Trọng Phụng',   N'NXB Văn học',     1936, 65000,  5, 1),
(N'Dế mèn phiêu lưu ký', N'Tô Hoài',          N'NXB Kim Đồng',    1941, 55000,  8, 5),
(N'Lập trình C# cơ bản', N'Nguyễn Văn An',    N'NXB KHKT',        2020, 180000, 3, 2),
(N'Clean Code',           N'Robert C. Martin', N'NXB Lao động',    2019, 210000, 4, 2),
(N'Đắc nhân tâm',        N'Dale Carnegie',    N'NXB Tổng hợp',    2018, 89000,  6, 4),
(N'Nhà giả kim',          N'Paulo Coelho',     N'NXB Hội nhà văn', 2020, 79000,  7, 1),
(N'Lịch sử Việt Nam',    N'Nhiều tác giả',    N'NXB KHXH',        2017, 350000, 2, 3),
(N'Tư duy nhanh và chậm',N'Daniel Kahneman',  N'NXB Thế giới',    2021, 149000, 5, 4);

INSERT INTO [DocGia] ([HoTen],[GioiTinh],[NgaySinh],[DiaChi],[SoDienThoai],[Email]) VALUES
(N'Nguyễn Thị Mai',  N'Nữ',  '2000-03-15', N'Hà Nội',    '0912345678', 'mai.nguyen@email.com'),
(N'Trần Văn Hùng',   N'Nam', '1998-07-22', N'TP.HCM',    '0987654321', 'hung.tran@email.com'),
(N'Lê Thị Hoa',      N'Nữ',  '2001-11-05', N'Đà Nẵng',   '0909123456', 'hoa.le@email.com'),
(N'Phạm Minh Tuấn',  N'Nam', '1999-05-30', N'Hải Phòng', '0933456789', 'tuan.pham@email.com'),
(N'Hoàng Thị Lan',   N'Nữ',  '2002-08-18', N'Cần Thơ',   '0965789012', 'lan.hoang@email.com');

INSERT INTO [PhieuMuon] ([MaDocGia],[MaSach],[NgayMuon],[NgayTraDuKien],[SoLuongMuon],[TrangThai]) VALUES
(1, 3, '2026-04-01', '2026-04-15', 1, N'Đã trả'),
(2, 4, '2026-04-05', '2026-04-19', 1, N'Đang mượn'),
(3, 1, '2026-04-10', '2026-04-24', 1, N'Đang mượn'),
(4, 6, '2026-03-20', '2026-04-03', 1, N'Quá hạn'),
(5, 5, '2026-04-15', '2026-04-29', 2, N'Đang mượn'),
(1, 7, '2026-04-18', '2026-05-02', 1, N'Đang mượn');
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2eec6e52-c09b-4c5f-8e47-08fec1f03863" />


*Chèn dữ liệu mẫu vào các bảng: 5 thể loại, 8 cuốn sách, 5 độc giả và 6 phiếu mượn*

---

## Phần 2: Xây Dựng Function

### Các Loại Hàm Có Sẵn (Built-in Functions)

SQL Server cung cấp hàng trăm hàm có sẵn để xử lý dữ liệu, được chia thành các nhóm chính:

- **Hàm chuỗi (String Functions):** `LEN()`, `SUBSTRING()`, `REPLACE()`, `UPPER()`, `STRING_AGG()`...
- **Hàm ngày tháng (Date/Time Functions):** `GETDATE()`, `DATEDIFF()`, `DATEADD()`, `FORMAT()`...
- **Hàm toán học (Mathematical Functions):** `ABS()`, `ROUND()`, `CEILING()`, `FLOOR()`...
- **Hàm tổng hợp (Aggregate Functions):** `SUM()`, `COUNT()`, `MAX()`, `MIN()`, `AVG()`...
- **Hàm chuyển đổi kiểu (Conversion Functions):** `CAST()`, `CONVERT()`, `TRY_CAST()`...
- **Hàm hệ thống (System Functions):** `ISNULL()`, `COALESCE()`, `IIF()`, `NEWID()`, `CHOOSE()`...

### Tìm Hiểu Chi Tiết Một Số Built-in Functions

**`DATEDIFF()` — Trụ cột của bài toán thư viện:**

Đây là hàm quan trọng nhất trong bài toán quản lý mượn sách, vì toàn bộ logic tính tiền phạt đều xoay quanh việc đếm số ngày chênh lệch giữa ngày trả dự kiến và ngày trả thực tế.

```sql
-- Tính số ngày mỗi phiếu mượn đã ở trạng thái mượn
SELECT MaPhieuMuon, NgayMuon, NgayTraDuKien,
       DATEDIFF(DAY, NgayMuon, NgayTraDuKien) AS SoNgayDuKien
FROM [PhieuMuon]
WHERE NgayTraThucTe IS NOT NULL;
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bad74801-4f55-447f-a849-ce82b078c2cb" />


*`DATEDIFF()` tính khoảng cách ngày giữa ngày mượn và ngày trả — nền tảng cho bài toán tính phạt*

---

**`IIF()` — Rẽ nhánh nhanh gọn thay cho CASE WHEN:**

```sql
-- Kiểm tra tình trạng tồn kho của từng cuốn sách
SELECT TenSach, SoLuongTon,
       IIF(SoLuongTon > 0, N'Còn sách', N'Hết sách') AS TinhTrang
FROM [Sach];
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/06b22a08-3987-4a4a-9b52-9c99d0b6982a" />


*`IIF()` giúp dịch dữ liệu số thành nhãn có ý nghĩa, ngắn gọn hơn nhiều so với CASE WHEN*

---

**`TRY_CAST()` — Ép kiểu an toàn, không gây lỗi:**

```sql
-- Ép kiểu an toàn: nếu giá trị không hợp lệ sẽ trả về NULL thay vì crash
SELECT TRY_CAST('2026-13-45' AS DATE) AS KetQua; -- Trả về NULL
SELECT TRY_CAST('abc123' AS INT)      AS KetQua; -- Trả về NULL
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0d2dc372-646b-4d29-99ab-327058d32e94" />


*`TRY_CAST()` rất hữu ích khi import dữ liệu thô từ file Excel hoặc CSV có thể chứa giá trị lỗi định dạng*

---

### Hàm Do Người Dùng Tự Viết (User-Defined Functions — UDFs)

**a) Mục đích**

UDF được sử dụng để đóng gói các logic nghiệp vụ phức tạp thành một khối xử lý có thể tái sử dụng nhiều lần. Giúp các câu lệnh `SELECT`, `UPDATE` trở nên ngắn gọn, dễ đọc và dễ bảo trì. Tăng tính nhất quán cho toàn hệ thống — logic chỉ viết một lần, sửa một lần là sửa được toàn bộ.

**b) Phân loại và khi nào dùng**

| Loại | Trả về | Dùng khi nào |
|---|---|---|
| **Scalar Function** | 1 giá trị duy nhất (số, chuỗi, ngày...) | Tính toán trên từng dòng, ví dụ: tính tiền phạt, tính thuế |
| **Inline TVF** | Bảng kết quả (1 câu SELECT) | Lọc dữ liệu có tham số, thay thế View cứng nhắc |
| **Multi-statement TVF** | Bảng kết quả (nhiều câu lệnh) | Logic phức tạp nhiều bước không thể viết trong 1 SELECT |

**c) Tại sao vẫn cần UDF khi đã có Built-in Functions?**

Mặc dù SQL Server cung cấp nhiều hàm dựng sẵn, nhưng các hàm này chỉ xử lý các tác vụ cơ bản và mang tính chung. Trong thực tế, mỗi hệ thống có quy tắc nghiệp vụ riêng mà không hàm nào có thể biết trước. Ví dụ: SQL Server không thể biết rằng *"thư viện của trường phạt 2.000đ/ngày trễ hạn"* hay *"mượn quá 5 cuốn cùng lúc thì bị giới hạn"* — đó là quy tắc riêng của từng tổ chức, và chỉ UDF mới giải quyết được.

---

### Viết 1 Scalar Function — Tính Tiền Phạt Mượn Sách Quá Hạn

**Ý tưởng (Scenario): "Bộ máy tính phạt tự động"**

Tình huống: Thư viện quy định mỗi ngày trả sách trễ hạn bị phạt **2.000 VNĐ/ngày**. Thủ thư cần một công cụ để tính tiền phạt ngay lập tức khi khách đến trả sách, mà không cần bấm máy tính thủ công. Hàm này sẽ được gọi lặp lại ở nhiều nơi: trong báo cáo, trong thủ tục tính hóa đơn, trong CURSOR...

**Luồng xử lý tổng quát:**

- **Bước 1.** Hàm nhận vào 2 tham số: `@NgayTraDuKien` và `@NgayTraThucTe`.
- **Bước 2.** Nếu `@NgayTraThucTe` là NULL (sách chưa trả), lấy ngày hiện tại `GETDATE()` để tính cho đến hôm nay.
- **Bước 3.** Tính số ngày chênh lệch bằng `DATEDIFF(DAY, ...)`.
- **Bước 4.** Nếu chênh lệch > 0 (quá hạn) thì nhân với 2.000. Nếu ≤ 0 (còn hạn) thì trả về 0.
- **Bước 5.** Trả về số tiền phạt dạng `MONEY`.

```sql
CREATE FUNCTION [dbo].[fn_TinhTienPhat]
(
    @NgayTraDuKien DATE,
    @NgayTraThucTe DATE   -- NULL = sách chưa trả, tính đến hôm nay
)
RETURNS MONEY
AS
BEGIN
    DECLARE @NgayTra DATE      = ISNULL(@NgayTraThucTe, GETDATE());
    DECLARE @SoNgayQuaHan INT  = DATEDIFF(DAY, @NgayTraDuKien, @NgayTra);
    DECLARE @TienPhat MONEY    = 0;

    IF @SoNgayQuaHan > 0
        SET @TienPhat = @SoNgayQuaHan * 2000;

    RETURN @TienPhat;
END;
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e4552d38-dc39-4f74-a564-fd9c706a2ac0" />


*Tạo Scalar Function `fn_TinhTienPhat` — đóng gói logic tính phạt để tái sử dụng ở nhiều nơi trong hệ thống*

```sql
-- Khai thác hàm: hiển thị danh sách phiếu mượn kèm tiền phạt
SELECT
    pm.[MaPhieuMuon],
    dg.[HoTen],
    s.[TenSach],
    pm.[NgayMuon],
    pm.[NgayTraDuKien],
    pm.[NgayTraThucTe],
    pm.[TrangThai],
    dbo.[fn_TinhTienPhat](pm.[NgayTraDuKien], pm.[NgayTraThucTe]) AS [TienPhat_VND]
FROM [PhieuMuon] pm
JOIN [DocGia] dg ON pm.[MaDocGia] = dg.[MaDocGia]
JOIN [Sach]   s  ON pm.[MaSach]   = s.[MaSach];
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/66920946-b61c-4404-84a7-e055bbbd5d06" />


*Kết quả: Phiếu đúng hạn hoặc đã trả đúng hạn hiển thị `0.00`, phiếu trả trễ hiển thị số tiền phạt tương ứng. Hàm tự động dùng `GETDATE()` nếu chưa có ngày trả thực tế.*

---

### Viết 1 Inline Table-Valued Function — Tra Cứu Sách Theo Thể Loại

**Ý tưởng (Scenario): "Màn hình tra cứu tự phục vụ của độc giả"**

Tình huống: Thư viện có một màn hình tra cứu tự phục vụ. Độc giả chọn thể loại muốn đọc và hệ thống trả về danh sách sách còn trong kho. Yêu cầu là phải lọc được theo từng thể loại khác nhau tùy theo lựa chọn — đây là điểm mà View thông thường không đáp ứng được (View không nhận tham số), nên cần dùng **Inline TVF**.

**Luồng xử lý tổng quát:**

- **Bước 1.** Hàm nhận vào `@MaTheLoai` làm tham số đầu vào.
- **Bước 2.** Thực hiện JOIN bảng `Sach` với bảng `TheLoai` để lấy tên thể loại.
- **Bước 3.** Lọc theo `@MaTheLoai` truyền vào **và** điều kiện `SoLuongTon > 0` (chỉ lấy sách còn hàng).
- **Bước 4.** Trả về bảng kết quả ngay lập tức (không qua biến bảng trung gian).

```sql
CREATE FUNCTION [dbo].[fn_GetSachTheoTheLoai]
(
    @MaTheLoai INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT
        s.[MaSach],
        s.[TenSach],
        s.[TacGia],
        s.[NamXuatBan],
        s.[GiaBan],
        s.[SoLuongTon],
        tl.[TenTheLoai]
    FROM [Sach] s
    JOIN [TheLoai] tl ON s.[MaTheLoai] = tl.[MaTheLoai]
    WHERE s.[MaTheLoai] = @MaTheLoai
      AND s.[SoLuongTon] > 0
);
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4150ec64-3b5d-4abc-9e92-7e47bdc73d39" />


*Tạo Inline TVF `fn_GetSachTheoTheLoai` — hoạt động như một View có tham số, SQL Server có thể tối ưu hóa (inline) nó vào truy vấn cha hiệu quả hơn Multi-statement TVF*

```sql
-- Khai thác: xem sách thể loại "Khoa học kỹ thuật" (MaTheLoai = 2) còn trong kho
SELECT * FROM dbo.[fn_GetSachTheoTheLoai](2);
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/28acb18b-8807-43ec-8c1c-ddad1224a54a" />


*Kết quả trả về danh sách sách thể loại "Khoa học kỹ thuật" còn tồn kho, sẵn sàng cho mượn*

---

### Viết 1 Multi-statement Table-Valued Function — Thống Kê Hiệu Quả Hoạt Động Sách

**Ý tưởng (Scenario): "Bảng tin tức hàng tháng trên màn hình của ban giám đốc thư viện"**

Tình huống: Ban giám đốc thư viện cần xem báo cáo hàng tháng: cuốn sách nào đang được mượn nhiều nhất (cần nhập thêm), cuốn nào nằm im trên giá (cần làm chương trình khuyến đọc). Báo cáo này cần phân loại và gán nhãn đánh giá cho từng đầu sách — logic này phức tạp nhiều bước, không thể biểu diễn trong một câu SELECT đơn, nên phải dùng **Multi-statement TVF** với biến bảng.

**Luồng xử lý tổng quát:**

- **Bước 1.** Hàm nhận vào `@Thang` và `@Nam` để xác định kỳ báo cáo.
- **Bước 2.** Khai báo biến bảng `@BangThongKe` để lưu kết quả trung gian.
- **Bước 3.** INSERT vào biến bảng: lấy tất cả sách, LEFT JOIN với `PhieuMuon` theo tháng/năm, đếm số lần mượn. Cột đánh giá ban đầu để `N'Chưa xét'`.
- **Bước 4.** Dùng **3 lệnh UPDATE riêng biệt** để gán nhãn: Hot (≥ 3 lần), Bình thường (1–2 lần), Ế (0 lần). Đây là phần logic nhiều bước không thể viết bằng 1 SELECT.
- **Bước 5.** Trả về toàn bộ biến bảng.

```sql
CREATE FUNCTION [dbo].[fn_ThongKeSachTheoThang]
(
    @Thang INT,
    @Nam   INT
)
RETURNS @BangThongKe TABLE (
    [MaSach]          INT,
    [TenSach]         NVARCHAR(300),
    [TenTheLoai]      NVARCHAR(100),
    [SoLanDuocMuon]   INT,
    [DanhGiaHoatDong] NVARCHAR(50)
)
AS
BEGIN
    -- Bước 1: Đẩy dữ liệu cơ bản vào biến bảng (tất cả sách + đếm lần mượn)
    INSERT INTO @BangThongKe ([MaSach],[TenSach],[TenTheLoai],[SoLanDuocMuon],[DanhGiaHoatDong])
    SELECT
        s.[MaSach], s.[TenSach], tl.[TenTheLoai],
        COUNT(pm.[MaPhieuMuon]),
        N'Chưa xét'
    FROM [Sach] s
    LEFT JOIN [TheLoai]   tl ON s.[MaTheLoai] = tl.[MaTheLoai]
    LEFT JOIN [PhieuMuon] pm ON s.[MaSach]    = pm.[MaSach]
         AND MONTH(pm.[NgayMuon]) = @Thang
         AND YEAR(pm.[NgayMuon])  = @Nam
    GROUP BY s.[MaSach], s.[TenSach], tl.[TenTheLoai];

    -- Bước 2: Phân loại bằng 3 lệnh UPDATE riêng biệt (logic nhiều bước)
    UPDATE @BangThongKe SET [DanhGiaHoatDong] = N' Sách Hot — Cần nhập thêm'
    WHERE [SoLanDuocMuon] >= 3;

    UPDATE @BangThongKe SET [DanhGiaHoatDong] = N' Bình thường'
    WHERE [SoLanDuocMuon] > 0 AND [SoLanDuocMuon] < 3;

    UPDATE @BangThongKe SET [DanhGiaHoatDong] = N' Sách Ế — Cần khuyến khích'
    WHERE [SoLanDuocMuon] = 0;

    RETURN;
END;
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/26a766d6-9f45-4a6d-ad81-f6aee5966e12" />


*Tạo Multi-statement TVF `fn_ThongKeSachTheoThang` — dùng biến bảng và nhiều bước UPDATE để phân loại sách, điều không thể làm trong Inline TVF*

```sql
-- Khai thác: báo cáo hiệu quả hoạt động sách tháng 4 năm 2026
SELECT * FROM dbo.[fn_ThongKeSachTheoThang](4, 2026)
ORDER BY [SoLanDuocMuon] DESC;
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a4ed50c0-3265-44c0-8f7c-a15676b558fc" />


*Kết quả: Mỗi cuốn sách được gán nhãn rõ ràng. Sách bị đánh " Sách Ế" sẽ được đưa vào chương trình khuyến khích đọc sách đặc biệt của thư viện.*

---

## Phần 3: Xây Dựng Stored Procedure

### Stored Procedure Hệ Thống (System Stored Procedures)

Trong SQL Server, hệ quản trị cung cấp nhiều Stored Procedure hệ thống nhằm hỗ trợ quản trị, truy vấn thông tin và thao tác với cơ sở dữ liệu.

**Các nhóm Stored Procedure phổ biến:**

- **Nhóm quản lý đối tượng:** Xem cấu trúc bảng, đổi tên, kiểm tra metadata.
- **Nhóm bảo mật:** Quản lý user, login, phân quyền truy cập.
- **Nhóm thông tin hệ thống:** Cung cấp thông tin về cấu trúc database, bảng, cột, index.
- **Nhóm hiệu năng:** Theo dõi hoạt động hệ thống, kiểm tra cache, tài nguyên.

**Đặc điểm chung:** Thường có tiền tố `sp_`, được tích hợp sẵn, gọi trực tiếp không cần định nghĩa.

### Một Vài System SP Và Cách Dùng

**`sp_helptext` — Trình soi mã nguồn ẩn:**

Khi có quá nhiều Function/Trigger/SP cũ do người khác viết mà không biết logic bên trong là gì, SP này in ra toàn bộ source code của đối tượng đó.

```sql
EXEC sp_helptext 'fn_TinhTienPhat'; -- In ra source code của hàm đã viết
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cfe69f6a-a422-495e-81ad-9e73c8500963" />


*`sp_helptext` trả về toàn bộ source code của `fn_TinhTienPhat` — vô cùng hữu ích khi bàn giao dự án hoặc kiểm tra logic của code cũ*

---

**`sp_spaceused` — Kế toán dung lượng:**

Báo cáo nhanh xem một bảng đang chiếm bao nhiêu dung lượng trên ổ cứng và có bao nhiêu dòng dữ liệu — nhanh hơn nhiều so với `COUNT(*)`.

```sql
EXEC sp_spaceused 'PhieuMuon'; -- Kiểm tra bảng PhieuMuon nặng bao nhiêu KB
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4cb5d943-9b25-418e-9884-44e91453b9ca" />

*`sp_spaceused` báo cáo số dòng và dung lượng bảng `PhieuMuon` — hữu ích để theo dõi sự phình to của dữ liệu theo thời gian vận hành*

---

### Viết 01 Stored Procedure Có Kiểm Tra Logic — Đăng Ký Mượn Sách

**Ý tưởng (Scenario): "Quầy đăng ký mượn sách nhanh"**

Tình huống: Độc giả đến quầy thủ thư và muốn mượn một cuốn sách. Thủ thư chỉ cần nhập **Mã độc giả** và **Mã sách** vào hệ thống — toàn bộ việc kiểm tra điều kiện hợp lệ sẽ được thực hiện tự động. Nếu có bất kỳ vấn đề nào (thẻ bị khóa, sách hết kho, ngày trả không hợp lệ...), hệ thống báo ngay ra màn hình thay vì gây lỗi SQL khó hiểu.

**Luồng xử lý tổng quát:**

- **Bước 1.** SP nhận vào: `@MaDocGia`, `@MaSach`, `@NgayTraDuKien`, `@SoLuong`.
- **Bước 2.** Kiểm tra độc giả: có tồn tại và `TrangThai = 1` (đang hoạt động) không? Nếu không → báo lỗi, thoát.
- **Bước 3.** Kiểm tra sách: có tồn tại và `SoLuongTon >= @SoLuong` không? Nếu không đủ → báo lỗi kèm số còn lại, thoát.
- **Bước 4.** Kiểm tra ngày trả: phải sau ngày hôm nay. Nếu không → báo lỗi, thoát.
- **Bước 5.** Tất cả hợp lệ: INSERT vào `PhieuMuon` và trừ `SoLuongTon` trong bảng `Sach`.
- **Bước 6.** In thông báo thành công.

```sql
CREATE PROCEDURE [dbo].[sp_MuonSach]
    @MaDocGia       INT,
    @MaSach         INT,
    @NgayTraDuKien  DATE,
    @SoLuong        INT = 1
AS
BEGIN
    SET NOCOUNT ON;

    -- Kiểm tra độc giả tồn tại và đang hoạt động
    IF NOT EXISTS (
        SELECT 1 FROM [DocGia]
        WHERE [MaDocGia] = @MaDocGia AND [TrangThai] = 1
    )
    BEGIN
        RAISERROR(N'Độc giả không tồn tại hoặc thẻ đã bị khóa!', 16, 1);
        RETURN;
    END

    -- Kiểm tra sách tồn tại
    DECLARE @TonKho INT;
    SELECT @TonKho = [SoLuongTon] FROM [Sach] WHERE [MaSach] = @MaSach;
    IF @TonKho IS NULL
    BEGIN
        RAISERROR(N'Sách không tồn tại trong hệ thống!', 16, 1);
        RETURN;
    END

    -- Kiểm tra đủ số lượng
    IF @TonKho < @SoLuong
    BEGIN
        RAISERROR(N'Không đủ sách! Hiện chỉ còn %d cuốn trong kho.', 16, 1, @TonKho);
        RETURN;
    END

    -- Kiểm tra ngày trả hợp lệ
    IF @NgayTraDuKien <= CAST(GETDATE() AS DATE)
    BEGIN
        RAISERROR(N'Ngày trả dự kiến phải sau ngày hôm nay!', 16, 1);
        RETURN;
    END

    -- Tất cả hợp lệ: tạo phiếu mượn và trừ tồn kho
    INSERT INTO [PhieuMuon] ([MaDocGia],[MaSach],[NgayMuon],[NgayTraDuKien],[SoLuongMuon],[TrangThai])
    VALUES (@MaDocGia, @MaSach, GETDATE(), @NgayTraDuKien, @SoLuong, N'Đang mượn');

    UPDATE [Sach]
    SET [SoLuongTon] = [SoLuongTon] - @SoLuong
    WHERE [MaSach] = @MaSach;

    PRINT N' Đăng ký mượn sách thành công!';
END;
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c9621097-1c0f-4d29-b9fc-89144b0293ac" />

*Tạo Stored Procedure `sp_MuonSach` — bao gồm đầy đủ các lớp kiểm tra logic nghiệp vụ trước khi ghi dữ liệu*

```sql
-- Thử đăng ký mượn thành công
EXEC [dbo].[sp_MuonSach] @MaDocGia=3, @MaSach=8,
     @NgayTraDuKien='2026-05-10', @SoLuong=1;

-- Thử đăng ký với số lượng vượt tồn kho (sách số 7 chỉ còn 2 cuốn)
EXEC [dbo].[sp_MuonSach] @MaDocGia=2, @MaSach=7,
     @NgayTraDuKien='2026-05-10', @SoLuong=99;
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c1c5ef75-4763-4fff-9444-915f6d98c3b3" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0fb6551f-c739-4b37-9ed9-dce401c35336" />


*Lần gọi đầu in ra " Đăng ký mượn sách thành công!". Lần gọi thứ hai báo lỗi rõ ràng: "Không đủ sách! Hiện chỉ còn 2 cuốn trong kho." — thân thiện hơn nhiều so với lỗi SQL thô.*

---

### Viết 01 Stored Procedure Có Tham Số OUTPUT — Tra Soát Công Nợ Độc Giả

**Ý tưởng (Scenario): "Tra soát nợ nhanh tại quầy trả sách"**

Tình huống: Khi độc giả đến trả sách, thủ thư cần biết ngay: khách này đang nợ bao nhiêu tiền phạt và có mấy phiếu đang quá hạn. Kết quả này không cần hiển thị dạng bảng — nó sẽ được phần mềm (C#, Java...) đọc về để hiển thị lên UI hoặc in hóa đơn. Tham số `OUTPUT` là lựa chọn phù hợp nhất cho trường hợp này.

**Luồng xử lý tổng quát:**

- **Bước 1.** SP nhận vào `@MaDocGia` và 2 biến `OUTPUT`: `@TongTienPhat`, `@SoPhieuQuaHan`.
- **Bước 2.** Dùng 1 câu `SELECT` tổng hợp: tính tổng tiền phạt (gọi hàm `fn_TinhTienPhat`) và đếm số phiếu quá hạn (dùng `CASE WHEN`).
- **Bước 3.** Gán kết quả vào 2 biến `OUTPUT`.
- **Bước 4.** Xử lý trường hợp độc giả không có phiếu nào bằng `ISNULL(..., 0)`.

```sql
CREATE PROCEDURE [dbo].[sp_TinhCongNoDocGia]
    @MaDocGia       INT,
    @TongTienPhat   MONEY OUTPUT,
    @SoPhieuQuaHan  INT   OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT
        @TongTienPhat  = SUM(dbo.[fn_TinhTienPhat]([NgayTraDuKien],[NgayTraThucTe])),
        @SoPhieuQuaHan = COUNT(CASE
                            WHEN ISNULL([NgayTraThucTe], GETDATE()) > [NgayTraDuKien]
                            THEN 1 END)
    FROM [PhieuMuon]
    WHERE [MaDocGia] = @MaDocGia;

    SET @TongTienPhat  = ISNULL(@TongTienPhat, 0);
    SET @SoPhieuQuaHan = ISNULL(@SoPhieuQuaHan, 0);
END;
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/abc24576-1d1b-45fd-8ef7-76f058500434" />

*Tạo Stored Procedure `sp_TinhCongNoDocGia` — dùng tham số OUTPUT để trả về nhiều giá trị mà không cần SELECT dạng bảng*

```sql
-- Khai thác SP với OUTPUT
DECLARE @Tien MONEY, @SoPhieu INT;
EXEC [dbo].[sp_TinhCongNoDocGia]
    @MaDocGia = 4,
    @TongTienPhat  = @Tien    OUTPUT,
    @SoPhieuQuaHan = @SoPhieu OUTPUT;

SELECT
    dg.[HoTen]      AS [TenDocGia],          
    @SoPhieu        AS [SoPhieuQuaHan],
    @Tien           AS [TongTienPhat_VND],
    FORMAT(@Tien, 'N0') + N' đồng' AS [TongTienPhat_ChuSo]
FROM [DocGia] dg
WHERE dg.[MaDocGia] = 4;                     
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1ec36fce-5065-44e7-9596-da9688ae123c" />

*Tham số OUTPUT cho phép SP trả về nhiều giá trị cùng lúc. Ảnh hiển thị độc giả Phạm Minh Tuấn đang có công nợ tiền phạt — thông tin này được phần mềm đọc về để hiển thị trên giao diện người dùng.*

---

### Viết 01 Stored Procedure Trả Về Result Set — Báo Cáo Tổng Hợp Mượn Sách

**Ý tưởng (Scenario): "Màn hình quản trị buổi sáng của trưởng thư viện"**

Tình huống: Mỗi buổi sáng, trưởng thư viện muốn xem báo cáo tổng hợp: ai đang mượn sách gì, khi nào trả, đang nợ bao nhiêu tiền phạt. Báo cáo cần lọc được theo khoảng thời gian và trạng thái tùy chọn. Dữ liệu cần JOIN từ 4 bảng: `PhieuMuon` + `DocGia` + `Sach` + `TheLoai`.

**Luồng xử lý tổng quát:**

- **Bước 1.** SP nhận vào 3 tham số tùy chọn: `@TuNgay`, `@DenNgay`, `@TrangThai` (đều có thể `NULL` = lấy tất cả).
- **Bước 2.** Nếu không truyền ngày, mặc định lấy 30 ngày gần nhất.
- **Bước 3.** Thực hiện JOIN 4 bảng để lấy đầy đủ thông tin: tên độc giả, tên sách, thể loại, trạng thái...
- **Bước 4.** Áp dụng bộ lọc linh hoạt: kỹ thuật `@TrangThai IS NULL OR pm.TrangThai = @TrangThai` cho phép không truyền tham số thì lấy tất cả.
- **Bước 5.** Tính tiền phạt cho từng dòng bằng cách gọi lại `fn_TinhTienPhat`.
- **Bước 6.** Trả về toàn bộ tập kết quả (Result Set).

```sql
CREATE PROCEDURE [dbo].[sp_BaoCaoMuonSach]
    @TuNgay     DATE         = NULL,
    @DenNgay    DATE         = NULL,
    @TrangThai  NVARCHAR(20) = NULL
AS
BEGIN
    SET NOCOUNT ON;

    IF @TuNgay  IS NULL SET @TuNgay  = DATEADD(DAY, -30, GETDATE());
    IF @DenNgay IS NULL SET @DenNgay = GETDATE();

    SELECT
        pm.[MaPhieuMuon],
        dg.[HoTen]     AS [TenDocGia],
        dg.[SoDienThoai],
        s.[TenSach],
        tl.[TenTheLoai],
        pm.[NgayMuon],
        pm.[NgayTraDuKien],
        pm.[NgayTraThucTe],
        pm.[SoLuongMuon],
        pm.[TrangThai],
        dbo.[fn_TinhTienPhat](pm.[NgayTraDuKien], pm.[NgayTraThucTe]) AS [TienPhat_VND],
        DATEDIFF(DAY, pm.[NgayMuon], ISNULL(pm.[NgayTraThucTe], GETDATE())) AS [SoNgayMuon]
    FROM [PhieuMuon] pm
    JOIN [DocGia]  dg ON pm.[MaDocGia]  = dg.[MaDocGia]
    JOIN [Sach]    s  ON pm.[MaSach]    = s.[MaSach]
    JOIN [TheLoai] tl ON s.[MaTheLoai]  = tl.[MaTheLoai]
    WHERE pm.[NgayMuon] BETWEEN @TuNgay AND @DenNgay
      AND (@TrangThai IS NULL OR pm.[TrangThai] = @TrangThai)
    ORDER BY pm.[NgayMuon] DESC;
END;
GO
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/013d45a2-f46f-4266-9f07-e2cc14856e6a" />

*Tạo Stored Procedure `sp_BaoCaoMuonSach` — JOIN 4 bảng, hỗ trợ bộ lọc linh hoạt, tái sử dụng Scalar Function để tính phạt ngay trong SELECT*

```sql
-- Xem tất cả phiếu mượn từ tháng 3 đến hết tháng 4
EXEC [dbo].[sp_BaoCaoMuonSach]
    @TuNgay = '2026-03-01', @DenNgay = '2026-04-30';

-- Chỉ xem các phiếu đang quá hạn
EXEC [dbo].[sp_BaoCaoMuonSach] @TrangThai = N'Quá hạn';
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e864e0fa-f3ae-4730-9e04-5f7f632130d4" />

*Kết quả trả về tập dữ liệu đầy đủ từ 4 bảng JOIN nhau. Kỹ thuật optional filter (`@TrangThai IS NULL`) cho phép linh hoạt lọc hoặc lấy tất cả chỉ với 1 SP duy nhất.*

---
