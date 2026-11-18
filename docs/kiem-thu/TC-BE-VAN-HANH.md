# Kịch bản kiểm thử – Backend: quản trị, sản phẩm, điều hành và kế toán

- **Người viết:** Bùi Phan Thành Trí
- **Phạm vi:**
  - `NhanVienController`
  - `TaiKhoanAdminController`
  - `QuanTriController`
  - `TourMauController`
  - `TourThucTeController`
  - `TourCongKhaiController`
  - `DichVuThemController`
  - `HanhDongXanhController`
  - `PhanCongController`
  - `VanHanhController`
  - `QuyetToanController`
  - `PowerBiController`
  - `DanhGiaController`
  - Tác vụ nền `TourThucTeScheduler`
- **Công cụ:** Postman hoặc Swagger UI, header `Authorization: Bearer <token>`.

## A. Luồng chính

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| BE-QT-01 | Chính | Tạo tài khoản nhân viên | Đăng nhập `admin` | `POST /api/quan-tri/dang-ky-nhan-vien` vai trò HDV | 200. Có tài khoản và hồ sơ nhân viên mới | | Chưa chạy |
| BE-QT-02 | Chính | Khóa và mở khóa nhân viên | Đăng nhập `admin`, có nhân viên đang hoạt động | `PUT /api/quan-tri/nhan-vien/{ma}/khoa` rồi `PUT .../mo-khoa` | Lần lượt 200, tài khoản `KHOA` rồi `HOAT_DONG` | | Chưa chạy |
| BE-QT-03 | Chính | Đổi vai trò | Đăng nhập `admin` | `PUT /api/quan-tri/nhan-vien/{ma}/vai-tro` sang KETOAN | 200. Đăng nhập lại thấy vai trò mới | | Chưa chạy |
| BE-QT-04 | Chính | Nhật ký hệ thống | Đã thực hiện BE-QT-01 | `GET /api/quan-tri/nhat-ky-he-thong` | 200. Có dòng ghi thao tác thêm tài khoản | | Chưa chạy |
| BE-QT-05 | Chính | Cập nhật năng lực HDV | Đăng nhập `manager01` | `PUT /api/dieu-hanh/nhan-vien/{ma}/nang-luc` | 200. `GET .../nang-luc` trả năng lực mới | | Chưa chạy |
| BE-SP-01 | Chính | Tạo tour mẫu | Đăng nhập `sanpham01` | `POST /api/san-pham/tour-mau` với thời lượng 3 ngày, giá sàn hợp lệ | 200. Tour mẫu mới có mã tự sinh | | Chưa chạy |
| BE-SP-02 | Chính | Thêm lịch trình | Có tour mẫu 3 ngày | `POST /api/san-pham/tour-mau/{id}/lich-trinh` cho ngày 1, 2, 3 | 200. Chi tiết tour mẫu có 3 ngày lịch trình | | Chưa chạy |
| BE-SP-03 | Chính | Sao chép tour mẫu | Có tour mẫu có lịch trình | `POST /api/san-pham/tour-mau/{id}/sao-chep` | 200. Tour mẫu mới có cùng lịch trình | | Chưa chạy |
| BE-SP-04 | Chính | Tạo tour thực tế | Có tour mẫu | `POST /api/dieu-hanh/tour-thuc-te` với giá không thấp hơn giá sàn, số khách tối thiểu ≤ tối đa | 200. Tour ở trạng thái `CHO_KICH_HOAT` | | Chưa chạy |
| BE-SP-05 | Chính | Quản lý dịch vụ thêm | Đăng nhập `sanpham01` | Thêm, sửa, xóa qua `/api/san-pham/dich-vu-them` | Mỗi bước 200, danh sách cập nhật đúng | | Chưa chạy |
| BE-SP-06 | Chính | Quản lý hành động xanh | Đăng nhập `sanpham01` | Thêm, sửa, xóa qua `/api/san-pham/hanh-dong-xanh` | Mỗi bước 200, điểm cộng không âm | | Chưa chạy |
| BE-SP-07 | Chính | Tra cứu tour công khai | Có tour `MO_BAN` | Không đăng nhập, gọi `GET /api/public/tour` và `GET /api/public/tour/{ma}` | 200. Chỉ thấy tour đang mở bán, chi tiết có lịch trình, dịch vụ, hành động xanh | | Chưa chạy |
| BE-DH-01 | Chính | Tìm HDV khả dụng | Tour `CHO_KICH_HOAT` | Đăng nhập `manager01`, `GET /api/dieu-hanh/hdv-kha-dung` theo khoảng ngày của tour | 200. Không có HDV đang bận trùng thời gian | | Chưa chạy |
| BE-DH-02 | Chính | Phân công HDV | Như BE-DH-01 | `POST /api/dieu-hanh/phan-cong` | 200. Phân công ở trạng thái chờ HDV phản hồi | | Chưa chạy |
| BE-DH-03 | Chính | HDV nhận phân công | Có phân công cho `hdv01` | Đăng nhập `hdv01`, `POST /api/huong-dan-vien/phan-cong/{ma}/dong-y` | 200. Tour đủ điều kiện mở bán | | Chưa chạy |
| BE-DH-04 | Chính | Mở bán tour | Đã có HDV nhận (BE-DH-03) | `PUT /api/dieu-hanh/tour-thuc-te/{id}` sang `MO_BAN` | 200. Tour xuất hiện ở API công khai | | Chưa chạy |
| BE-DH-05 | Chính | Tự chuyển trạng thái tour | Tour `MO_BAN` có ngày khởi hành là hôm nay | Chờ đến đầu giờ kế tiếp (`TourThucTeScheduler` chạy mỗi giờ) | Tour chuyển `DANG_DIEN_RA` | | Chưa chạy |
| BE-VH-01 | Chính | Xem đoàn khách | Tour `DANG_DIEN_RA` do `hdv01` phụ trách | `GET /api/huong-dan-vien/tour/{maTour}/doan` | 200. Danh sách khách và người đồng hành | | Chưa chạy |
| BE-VH-02 | Chính | Điểm danh | Như BE-VH-01 | `POST /api/huong-dan-vien/tour/{maTour}/diem-danh` | 200. Có bản ghi điểm danh | | Chưa chạy |
| BE-VH-03 | Chính | Ghi nhận hành động xanh | Như BE-VH-01 | `POST /api/huong-dan-vien/tour/{maTour}/hanh-dong-xanh` cho một khách | 200. Điểm xanh của khách tăng đúng điểm cộng | | Chưa chạy |
| BE-VH-04 | Chính | Báo cáo sự cố | Như BE-VH-01 | `POST /api/huong-dan-vien/tour/{maTour}/su-co` mức `SOS` | 200. Điều hành xem được qua `GET /api/dieu-hanh/tour/{maTour}/su-co` | | Chưa chạy |
| BE-VH-05 | Chính | Khai chi phí | Như BE-VH-01 | `POST /api/huong-dan-vien/tour/{maTour}/chi-phi` kèm chứng từ | 200. Chi phí chờ kế toán duyệt | | Chưa chạy |
| BE-KT-01 | Chính | Duyệt chi phí | Có chi phí ở BE-VH-05 | Đăng nhập `ketoan01`, `PUT /api/ke-toan/chi-phi/{ma}/duyet` | 200. Chi phí đã duyệt | | Chưa chạy |
| BE-KT-02 | Chính | Quyết toán tour | Tour `KET_THUC` | `GET /api/ke-toan/tinh-toan/{maTour}`, `POST /api/ke-toan/quyet-toan/{maTour}`, rồi `PUT /api/ke-toan/quyet-toan/{ma}/chot` | 200. Quyết toán chốt, tour `DA_QUYET_TOAN` | | Chưa chạy |
| BE-KT-03 | Chính | Xác nhận hoàn tiền | Có giao dịch hoàn tiền `CHO_THANH_TOAN` | `PUT /api/ke-toan/giao-dich-hoan/{ma}/xac-nhan` | 200. Giao dịch hoàn tất | | Chưa chạy |
| BE-KT-04 | Chính | Báo cáo doanh thu | Đăng nhập `ketoan01` | `GET /api/ke-toan/bao-cao/doanh-thu` theo khoảng ngày | 200. Số liệu khớp tổng các đơn đã xác nhận trong khoảng | | Chưa chạy |
| BE-KT-05 | Chính | Power BI | Đăng nhập `ketoan01` | `GET /api/ke-toan/power-bi/kho-du-lieu`, `GET .../ket-noi`, `POST .../xuat-du-lieu` | 200. Xuất được dữ liệu, nhật ký ghi `XUAT_DU_LIEU_POWERBI` | | Chưa chạy |
| BE-DG-01 | Chính | Khách đánh giá tour | `khach01` đã tham gia tour `KET_THUC` | `POST /api/khach-hang/danh-gia` | 200. Kinh doanh xem được qua `GET /api/kinh-doanh/danh-gia` | | Chưa chạy |

