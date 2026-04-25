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
