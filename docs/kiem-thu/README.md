# Kịch bản kiểm thử – Digital Travel ERP

Thư mục này chứa kịch bản kiểm thử (test case) thủ công cho toàn bộ hệ thống, chia theo phân hệ. Mỗi thành viên viết test case cho phần mình phụ trách.

## Danh mục tài liệu

| File | Phân hệ | Người viết |
|---|---|---|
| [TC-DB.md](TC-DB.md) | Cơ sở dữ liệu Oracle | Nguyễn Trần Thủy Tiên |
| [TC-BE-GIAO-DICH.md](TC-BE-GIAO-DICH.md) | Backend: xác thực, khách hàng, đặt tour, thanh toán, voucher | Nguyễn Tuấn Vũ |
| [TC-BE-VAN-HANH.md](TC-BE-VAN-HANH.md) | Backend: quản trị, sản phẩm, điều hành, vận hành HDV, kế toán | Bùi Phan Thành Trí |
| [TC-FE-ADMIN.md](TC-FE-ADMIN.md) | Giao diện cổng Quản trị | Đoàn Thị Thùy Trang |
| [TC-FE-KH-HDV.md](TC-FE-KH-HDV.md) | Giao diện cổng Khách hàng và cổng HDV | Lê Thị Thanh Tiền |

## Môi trường kiểm thử

| Thành phần | Cấu hình |
|---|---|
| CSDL | Oracle, schema sạch, chạy `KhoiTaoBang.sql` rồi các file dữ liệu mẫu theo `Backend/HUONGDAN.md` |
| Backend | `Backend/` chạy bằng `mvnw spring-boot:run`, cổng mặc định 8080 |
| FE Quản trị | `Frontend/Admin`, `npm run dev` |
| FE Khách hàng | `Frontend/Khachhang`, `npm run dev` |
| FE HDV | `Frontend/HDV`, `npm run dev` |
| Công cụ gọi API | Postman hoặc Swagger UI của backend |

## Tài khoản kiểm thử (dữ liệu mẫu)

Mật khẩu mặc định của các tài khoản dưới đây là `password` (theo `Backend/HUONGDAN.md`).

| Vai trò | Tên đăng nhập |
|---|---|
| ADMIN | `admin` |
| DIEUHANH | `manager01` |
| SANPHAM | `sanpham01` |
| KINHDOANH | `sales01` |
| KETOAN | `ketoan01` |
| HDV | `hdv01` … `hdv07` |
| KHACHHANG | `khach01`, `khach02`, `khach03` |

## Quy ước

- **Mã test case:** `<PHÂN HỆ>-<NHÓM>-<STT>`, ví dụ `BE-DT-05` là test case số 5 của nhóm Đặt tour, phân hệ Backend.
- **Loại test case:**
  - `Chính`: luồng đúng.
  - `Lỗi`: dữ liệu sai hoặc trạng thái không hợp lệ.
  - `Quyền`: chưa đăng nhập hoặc sai vai trò.
- **Kết quả mong đợi với API:** gồm mã HTTP và nội dung chính của phản hồi `ApiResponse`. API lỗi trả 400 (dữ liệu hoặc trạng thái sai), 401 (chưa đăng nhập hoặc token sai), 403 (sai quyền), 404 (không tìm thấy).
- **Trạng thái:** `Chưa chạy` / `Đạt` / `Không đạt`. Khi `Không đạt`, ghi lại kết quả thực tế và tạo issue trên GitHub.

## Mẫu bảng test case

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| XX-YY-01 | Chính | … | … | … | … | | Chưa chạy |

## Quy trình

1. Mỗi thành viên tạo nhánh `test/<phân hệ>` từ `develop` và viết test case vào file của mình.
2. Mở Pull Request vào `develop`, chọn reviewer theo phân công.
3. Reviewer kiểm tra:
   - Test case có bám đúng API và quy tắc nghiệp vụ.
   - Có đủ luồng đúng, luồng lỗi và phân quyền.
4. Trưởng nhóm merge sau khi có Approve.
5. Khi chạy kiểm thử thật, cập nhật cột "Kết quả thực tế" và "Trạng thái" qua một PR mới.
