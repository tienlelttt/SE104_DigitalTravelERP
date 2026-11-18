# Kịch bản kiểm thử – Giao diện cổng Khách hàng và cổng HDV

- **Người viết:** Lê Thị Thanh Tiền
- **Phạm vi:**
  - `Frontend/Khachhang`: trang chủ, chi tiết tour, giới thiệu, hộ chiếu số, luồng đặt tour nhiều bước, thanh toán.
  - `Frontend/HDV`: đăng nhập, bảng điều khiển, lịch trình, điểm danh, điểm xanh, chi phí, sự cố, hồ sơ.
- **Môi trường:** backend chạy với dữ liệu mẫu, `npm run dev` cho từng cổng, trình duyệt Chrome. Có thêm một lượt test cho Khách hàng trên màn hình điện thoại (375px).

## A. Luồng chính

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| FK-TC-01 | Chính | Trang chủ | Có tour đang mở bán | Mở `/` khi chưa đăng nhập | Hiện danh sách tour mở bán có ảnh, giá, ngày khởi hành | | Chưa chạy |
| FK-TC-02 | Chính | Tìm và lọc tour | Ở trang chủ | Tìm theo điểm đến, lọc theo khoảng giá | Danh sách chỉ còn các tour khớp điều kiện | | Chưa chạy |
| FK-CT-01 | Chính | Chi tiết tour | Ở trang chủ | Bấm một tour | Mở `/tour/:tourId` với lịch trình từng ngày, dịch vụ thêm, hành động xanh, đánh giá | | Chưa chạy |
| FK-XT-01 | Chính | Đăng ký và đăng nhập | Chưa đăng nhập | Mở hộp đăng nhập, đăng ký tài khoản mới, rồi đăng nhập | Header hiện tên khách hàng | | Chưa chạy |
| FK-DT-01 | Chính | Đặt tour – chọn loại | Đăng nhập `khach01`, ở chi tiết tour | Bấm Đặt tour, chọn loại đặt | Mở cửa sổ đặt tour ở bước nhập hành khách | | Chưa chạy |
| FK-DT-02 | Chính | Đặt tour – hành khách | Như FK-DT-01 | Nhập thông tin người đặt và 1 người đồng hành | Chuyển sang bước dịch vụ | | Chưa chạy |
| FK-DT-03 | Chính | Đặt tour – dịch vụ và hành động xanh | Như FK-DT-02 | Chọn 1 dịch vụ thêm và 1 hành động xanh | Tổng tiền cập nhật ngay theo dịch vụ | | Chưa chạy |
| FK-DT-04 | Chính | Đặt tour – tổng kết và voucher | Như FK-DT-03, ví có voucher còn hạn | Ở bước tổng kết, áp voucher | Tổng tiền giảm đúng, hiện dòng giảm giá | | Chưa chạy |
| FK-TT-01 | Chính | Thanh toán QR | Như FK-DT-04 | Chọn chuyển khoản QR, bấm "Tôi đã chuyển khoản" trong 5 phút | Hiện màn hình chờ xác nhận, sau khi Kinh doanh xác nhận thì hiện màn hình đặt tour thành công | | Chưa chạy |
| FK-HC-01 | Chính | Hộ chiếu số | Đăng nhập `khach01` | Mở `/passport` | Hiện hạng thành viên, điểm xanh, lịch sử tour, ví voucher | | Chưa chạy |
| FK-HC-02 | Chính | Cập nhật hồ sơ | Như FK-HC-01 | Sửa số điện thoại, Lưu | Thông báo thành công, tải lại vẫn thấy số mới | | Chưa chạy |
| FK-HC-03 | Chính | Đổi điểm lấy voucher | Đủ điểm xanh | Ở hộ chiếu số, chọn voucher đổi được, xác nhận | Điểm xanh giảm, voucher xuất hiện trong ví | | Chưa chạy |
| FK-HC-04 | Chính | Yêu cầu hủy tour | Có đơn đã xác nhận, còn hơn 2 ngày đến ngày khởi hành | Ở lịch sử đặt tour, gửi yêu cầu hủy | Đơn hiện trạng thái chờ hủy | | Chưa chạy |
| FH-DN-01 | Chính | HDV đăng nhập | Ở cổng HDV | Đăng nhập `hdv01` / `password` | Vào bảng điều khiển, hiện tên HDV | | Chưa chạy |
| FH-BD-01 | Chính | Nhận phân công | Có phân công chờ phản hồi | Ở bảng điều khiển, bấm Nhận | Phân công chuyển đã nhận, tour vào lịch trình | | Chưa chạy |
| FH-LT-01 | Chính | Lịch trình | Có tour đã nhận | Mở tab Lịch trình | Hiện các tour theo ngày, mở được lịch trình chi tiết | | Chưa chạy |
| FH-DD-01 | Chính | Điểm danh | Tour đang diễn ra | Mở tab Điểm danh, đánh dấu có mặt cho khách, Lưu | Lưu thành công, tải lại vẫn giữ trạng thái | | Chưa chạy |
| FH-DX-01 | Chính | Điểm xanh | Tour đang diễn ra | Mở tab Điểm xanh, ghi nhận hành động xanh cho một khách | Thông báo thành công, khách được cộng điểm | | Chưa chạy |
| FH-CP-01 | Chính | Khai chi phí | Tour đang diễn ra | Mở tab Chi phí, nhập khoản chi, tải chứng từ, Gửi | Chi phí ở trạng thái chờ duyệt | | Chưa chạy |
| FH-SC-01 | Chính | Báo sự cố | Tour đang diễn ra | Mở tab Sự cố, chọn mức SOS, nhập mô tả, Gửi | Sự cố hiện trong danh sách | | Chưa chạy |
| FH-HS-01 | Chính | Hồ sơ HDV | Đã đăng nhập | Mở tab Hồ sơ | Hiện thông tin cá nhân và năng lực | | Chưa chạy |

## B. Dữ liệu sai, phân quyền và giao diện

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| FK-XT-02 | Lỗi | Sai mật khẩu | Ở hộp đăng nhập | Nhập mật khẩu sai | Hiện thông báo lỗi, không đóng hộp thoại | | Chưa chạy |
| FK-XT-03 | Lỗi | Đăng ký trùng email | Email đã có | Đăng ký với email đó | Hiện "Email đã được sử dụng" | | Chưa chạy |
| FK-DT-05 | Quyền | Đặt tour khi chưa đăng nhập | Chưa đăng nhập, ở chi tiết tour | Bấm Đặt tour | Mở hộp đăng nhập thay vì cửa sổ đặt tour | | Chưa chạy |
| FK-DT-06 | Lỗi | Thiếu thông tin hành khách | Ở bước hành khách | Bỏ trống họ tên người đồng hành, bấm Tiếp | Báo lỗi tại trường, không sang bước sau | | Chưa chạy |
| FK-DT-07 | Lỗi | Tour hết chỗ | Tour còn 0 chỗ | Mở chi tiết tour | Nút Đặt tour bị khóa hoặc hiện "Tour đã hết chỗ" | | Chưa chạy |
| FK-TT-02 | Lỗi | QR hết hạn | Đang ở bước thanh toán QR | Chờ quá 5 phút, không bấm gì | Hiện thông báo mã QR hết hạn, đơn hết hạn giữ chỗ | | Chưa chạy |
| FK-HC-05 | Quyền | Vào hộ chiếu số khi chưa đăng nhập | Chưa đăng nhập | Gõ `/passport` | Yêu cầu đăng nhập, không hiện dữ liệu | | Chưa chạy |
| FK-GD-01 | Lỗi | Đường dẫn sai | – | Gõ `/xyz` | Hiện trang 404 | | Chưa chạy |
| FK-GD-02 | Chính | Giao diện điện thoại | Màn hình 375px | Mở trang chủ, chi tiết tour, cửa sổ đặt tour | Không tràn ngang, menu thu gọn hoạt động, nút bấm được | | Chưa chạy |
| FH-DN-02 | Quyền | Khách đăng nhập cổng HDV | Ở cổng HDV | Đăng nhập `khach01` | Bị từ chối, không vào bảng điều khiển | | Chưa chạy |
| FH-DN-03 | Quyền | Hết phiên | Đã đăng nhập cổng HDV | Sửa token cho sai, thao tác một tab | Ứng dụng đưa về màn hình đăng nhập | | Chưa chạy |
| FH-SC-02 | Lỗi | Sự cố thiếu mô tả | Ở tab Sự cố | Bỏ trống mô tả, Gửi | Hiện "Mô tả sự cố không được để trống" | | Chưa chạy |
| FH-DX-02 | Lỗi | Ghi trùng hành động xanh | Đã ghi ở FH-DX-01 | Ghi lại cùng hành động cho cùng khách | Hiện thông báo đã ghi nhận trước đó | | Chưa chạy |
