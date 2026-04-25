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

