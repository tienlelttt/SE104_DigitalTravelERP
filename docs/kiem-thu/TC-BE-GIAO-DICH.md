# Kịch bản kiểm thử – Backend: xác thực và giao dịch

- **Người viết:** Nguyễn Tuấn Vũ
- **Phạm vi:**
  - `AuthController`
  - `KhachHangController`
  - `KinhDoanhDatTourController`
  - `ThanhToanController`
  - `VoucherController`
  - `VoucherAdminController`
  - `UploadController`
  - Tác vụ nền `DatTourScheduler`
- **Công cụ:** Postman hoặc Swagger UI. Mọi API cần đăng nhập đều gửi header `Authorization: Bearer <token>`.

## A. Luồng chính

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| BE-AU-01 | Chính | Đăng ký khách hàng | Email, CCCD, tên đăng nhập chưa tồn tại | `POST /api/auth/dang-ky` với đủ thông tin, mật khẩu và xác nhận trùng nhau | 200. Tạo tài khoản vai trò KHACHHANG và hộ chiếu số ban đầu | | Chưa chạy |
| BE-AU-02 | Chính | Đăng nhập | Có tài khoản `khach01` | `POST /api/auth/dang-nhap` với đúng tên đăng nhập và mật khẩu | 200. Trả token JWT và vai trò KHACHHANG | | Chưa chạy |
| BE-AU-03 | Chính | Đổi mật khẩu | Đã đăng nhập | `POST /api/auth/doi-mat-khau` với mật khẩu cũ đúng, mật khẩu mới và xác nhận trùng nhau | 200. Đăng nhập lại bằng mật khẩu mới thành công | | Chưa chạy |
| BE-AU-04 | Chính | Quên và đặt lại mật khẩu | Có tài khoản với email hợp lệ | `POST /api/auth/quen-mat-khau`, lấy mã, rồi `POST /api/auth/dat-lai-mat-khau` | 200. Đăng nhập bằng mật khẩu mới thành công | | Chưa chạy |
| BE-AU-05 | Chính | Đăng xuất | Đã đăng nhập | `POST /api/auth/dang-xuat`, sau đó gọi lại một API cần đăng nhập bằng token cũ | Đăng xuất trả 200. Lần gọi sau trả 401 | | Chưa chạy |
| BE-KH-01 | Chính | Xem hồ sơ và hộ chiếu số | Đăng nhập `khach01` | `GET /api/khach-hang/ho-so` | 200. Có thông tin cá nhân, điểm xanh, hạng thành viên | | Chưa chạy |
| BE-KH-02 | Chính | Cập nhật hồ sơ | Đăng nhập `khach01` | `PUT /api/khach-hang/ho-so` với số điện thoại mới | 200. Gọi lại `GET /ho-so` thấy số mới | | Chưa chạy |
| BE-KH-03 | Chính | Lịch sử tour | `khach01` có tour đã kết thúc | `GET /api/khach-hang/lich-su-tour` | 200. Danh sách tour đã tham gia | | Chưa chạy |
| BE-DT-01 | Chính | Đặt tour | Tour `MO_BAN` còn chỗ | `POST /api/khach-hang/dat-tour` với 1 người đặt, 1 người đồng hành, 1 dịch vụ thêm | 200. Đơn ở trạng thái `CHO_XAC_NHAN`, `ChoConLai` của tour giảm 2, tổng tiền gồm giá tour và dịch vụ | | Chưa chạy |
| BE-DT-02 | Chính | Xem đơn của tôi | Đã có đơn ở BE-DT-01 | `GET /api/khach-hang/dat-tour` và `GET /api/khach-hang/dat-tour/{maDatTour}` | 200. Có đơn vừa tạo với đủ chi tiết hành khách và dịch vụ | | Chưa chạy |
| BE-DT-03 | Chính | Kinh doanh xem đơn | Đăng nhập `sales01` | `GET /api/kinh-doanh/dat-tour` | 200. Danh sách đơn có phân trang | | Chưa chạy |
| BE-TT-01 | Chính | Khởi tạo thanh toán QR | Đơn `CHO_XAC_NHAN` của `khach01` | `POST /api/thanh-toan/khoi-tao` | 200. Tạo giao dịch chờ thanh toán, trả thông tin QR | | Chưa chạy |
| BE-TT-02 | Chính | Báo đã chuyển khoản | Đã khởi tạo QR dưới 5 phút | `POST /api/thanh-toan/{maDatTour}/xac-nhan-chuyen-khoan` | 200. Giao dịch chuyển sang chờ đối soát | | Chưa chạy |
| BE-TT-03 | Chính | Kinh doanh xác nhận thanh toán | Khách đã báo chuyển khoản | Đăng nhập `sales01`, `PUT /api/kinh-doanh/dat-tour/{maDatTour}/xac-nhan` | 200. Đơn chuyển `DA_XAC_NHAN` | | Chưa chạy |
| BE-TT-04 | Chính | Xem kết quả thanh toán | Đã xong BE-TT-03 | `GET /api/thanh-toan/{maDatTour}/ket-qua` | 200. Trạng thái thanh toán thành công | | Chưa chạy |
| BE-TT-05 | Chính | Hết hạn QR | Đã khởi tạo QR quá 5 phút, chưa báo chuyển khoản | `POST /api/thanh-toan/{maDatTour}/het-han-qr` | 200. Đơn chuyển `HET_HAN_GIU_CHO`, chỗ được trả lại cho tour | | Chưa chạy |
| BE-TT-06 | Chính | Tác vụ nền hủy đơn quá hạn | Có đơn QR quá 5 phút | Chờ tối đa 1 phút (`DatTourScheduler` chạy mỗi 60 giây) | Đơn tự chuyển `HET_HAN_GIU_CHO` | | Chưa chạy |
| BE-HT-01 | Chính | Yêu cầu hủy tour | Đơn `DA_XAC_NHAN`, ngày khởi hành còn hơn 2 ngày | `POST /api/khach-hang/dat-tour/{maDatTour}/huy` với lý do | 200. Đơn chuyển `CHO_HUY`, tạo yêu cầu hủy chờ xử lý | | Chưa chạy |
| BE-HT-02 | Chính | Kinh doanh duyệt hủy | Có yêu cầu ở BE-HT-01 | Đăng nhập `sales01`, `POST /api/kinh-doanh/yeu-cau-huy/{maYeuCau}/duyet` | 200. Đơn `DA_HUY`, sinh giao dịch hoàn tiền chờ kế toán | | Chưa chạy |
| BE-VC-01 | Chính | Tạo voucher | Đăng nhập `sales01` | `POST /api/kinh-doanh/voucher` loại `SO_TIEN`, ngày hiệu lực trước ngày hết hạn | 200. Voucher ở trạng thái `SAN_SANG` | | Chưa chạy |
| BE-VC-02 | Chính | Phát hành voucher | Có voucher ở BE-VC-01 | `POST /api/kinh-doanh/voucher/{maVoucher}/phat-hanh` cho `khach01` | 200. `khach01` thấy voucher trong `GET /api/khach-hang/vi-voucher` | | Chưa chạy |
| BE-VC-03 | Chính | Đổi điểm lấy voucher | `khach01` đủ điểm xanh, có voucher đổi được | `GET /api/khach-hang/voucher-co-the-doi` rồi `POST /api/khach-hang/doi-diem` | 200. Điểm xanh giảm, voucher vào ví, có dòng trong nhật ký đổi điểm | | Chưa chạy |
| BE-VC-04 | Chính | Áp voucher cho đơn | Đơn `CHO_XAC_NHAN`, voucher còn hiệu lực trong ví | `POST /api/khach-hang/ap-voucher` | 200. Tổng tiền đơn giảm đúng mức ưu đãi | | Chưa chạy |
| BE-VC-05 | Chính | Thu hồi voucher | Voucher đã phân bổ cho khách | `PUT /api/kinh-doanh/voucher/{maVoucher}/khach-hang/{maKhachHang}/thu-hoi` | 200. Voucher không còn dùng được trong ví khách | | Chưa chạy |
| BE-UP-01 | Chính | Tải lên chứng từ | Có file ảnh hợp lệ | `POST /api/public/upload` với file, rồi `GET /api/public/upload/files/{fileName}` | 200. Trả tên file, tải lại được đúng nội dung | | Chưa chạy |

