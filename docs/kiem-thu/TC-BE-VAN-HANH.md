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

## B. Dữ liệu sai, trạng thái sai và phân quyền

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| BE-QT-06 | Lỗi | Tự khóa chính mình | Đăng nhập `admin` | Khóa tài khoản của chính `admin` | 400 "Không thể tự khóa chính tài khoản của bạn." | | Chưa chạy |
| BE-QT-07 | Lỗi | Khóa tài khoản đã khóa | Tài khoản đang `KHOA` | Gọi khóa lần nữa | 400 "Tài khoản đã ở trạng thái KHOA." | | Chưa chạy |
| BE-QT-08 | Quyền | Không phải admin xem nhật ký | Đăng nhập `manager01` | `GET /api/quan-tri/nhat-ky-he-thong` | 403 | | Chưa chạy |
| BE-SP-08 | Lỗi | Lịch trình vượt thời lượng | Tour mẫu 3 ngày | Thêm lịch trình ngày thứ 4 | 400 "Ngay thu khong duoc vuot qua thoi luong tour (…)" | | Chưa chạy |
| BE-SP-09 | Lỗi | Xóa tour mẫu đang dùng | Tour mẫu đã có tour thực tế | `DELETE /api/san-pham/tour-mau/{id}` | 400 "Không thể xóa tour mẫu vì đã có tour thực tế liên kết" | | Chưa chạy |
| BE-SP-10 | Lỗi | Giá thấp hơn giá sàn | Có tour mẫu | Tạo tour thực tế với giá nhỏ hơn giá sàn | 400 "Gia hien hanh khong duoc thap hon gia san cua tour mau (…)" | | Chưa chạy |
| BE-SP-11 | Lỗi | Số khách sai | – | Tạo tour thực tế với số khách tối thiểu lớn hơn tối đa | 400 "Số khách tối thiểu không được lớn hơn số khách tối đa" | | Chưa chạy |
| BE-SP-12 | Lỗi | Xóa tour đã có đơn | Tour thực tế đã có đơn đặt | `DELETE /api/dieu-hanh/tour-thuc-te/{id}` | 400 "Không thể xóa tour thực tế đã phát sinh đơn đặt tour" | | Chưa chạy |
| BE-SP-13 | Quyền | Kinh doanh tạo tour mẫu | Đăng nhập `sales01` | `POST /api/san-pham/tour-mau` | 403 | | Chưa chạy |
| BE-DH-06 | Lỗi | Mở bán khi chưa có HDV | Tour `CHO_KICH_HOAT` chưa có HDV nhận | Cập nhật sang `MO_BAN` | 400 "Tour chỉ được mở bán khi có HDV đã xác nhận phân công." | | Chưa chạy |
| BE-DH-07 | Lỗi | HDV trùng lịch | HDV đã được phân công tour khác cùng thời gian | Phân công HDV đó cho tour này | 400 "HDV đã bị phân công cho tour khác trong cùng thời gian." | | Chưa chạy |
| BE-DH-08 | Lỗi | Phản hồi phân công hai lần | HDV đã đồng ý | Gọi `dong-y` hoặc `tu-choi` lần nữa | 400 "Phân công đã được phản hồi trước đó." | | Chưa chạy |
| BE-DH-09 | Quyền | HDV phản hồi phân công của người khác | Phân công thuộc `hdv02` | Đăng nhập `hdv01`, gọi `dong-y` | 403 "Bạn không có quyền phản hồi phân công này" | | Chưa chạy |
| BE-VH-06 | Lỗi | Mức độ sự cố sai | Như BE-VH-04 | Báo sự cố với mức độ `CAO` | 400 "Mức độ sự cố không hợp lệ. Chỉ chấp nhận: THAP, SOS" | | Chưa chạy |
| BE-VH-07 | Lỗi | Ghi hành động xanh trùng | Đã ghi nhận ở BE-VH-03 | Ghi lại cùng hành động cho cùng khách | 400 "Hành động xanh này đã được ghi nhận cho khách hàng trong tour." | | Chưa chạy |
| BE-VH-08 | Quyền | HDV vận hành tour không phụ trách | Tour do `hdv02` phụ trách | Đăng nhập `hdv01`, điểm danh tour đó | 403 "Bạn không có quyền vận hành tour này" | | Chưa chạy |
| BE-KT-06 | Lỗi | Quyết toán tour chưa kết thúc | Tour `DANG_DIEN_RA` | `POST /api/ke-toan/quyet-toan/{maTour}` | 400 "Tour chưa kết thúc, không thể quyết toán." | | Chưa chạy |
| BE-KT-07 | Lỗi | Sửa quyết toán đã chốt | Quyết toán đã chốt | Gọi `yeu-cau-bo-sung` | 400 "Quyết toán đã bị chốt, không thể yêu cầu bổ sung." | | Chưa chạy |
| BE-KT-08 | Lỗi | Duyệt chi phí đã xử lý | Chi phí đã duyệt | Gọi `duyet` lần nữa | 400 "Chi phí đã được xử lý rồi." | | Chưa chạy |
| BE-KT-09 | Quyền | Kinh doanh xem báo cáo | Đăng nhập `sales01` | `GET /api/ke-toan/bao-cao/doanh-thu` | 403 | | Chưa chạy |
| BE-DG-02 | Lỗi | Đánh giá tour chưa kết thúc | Tour `DANG_DIEN_RA` | `POST /api/khach-hang/danh-gia` | 400 "Chỉ có thể đánh giá tour đã kết thúc" | | Chưa chạy |
| BE-DG-03 | Lỗi | Đánh giá hai lần | Đã đánh giá ở BE-DG-01 | Gửi đánh giá lần nữa | 400 "Bạn đã đánh giá tour này rồi" | | Chưa chạy |
