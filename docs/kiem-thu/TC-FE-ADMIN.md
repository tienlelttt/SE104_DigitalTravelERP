# Kịch bản kiểm thử – Giao diện cổng Quản trị

- **Người viết:** Đoàn Thị Thùy Trang
- **Phạm vi:** `Frontend/Admin` (React + Vite), gồm đăng nhập, phân quyền menu (`config/rolePermissions.ts`, `hooks/useAccessGuard.ts`) và các trang nghiệp vụ.
- **Môi trường:** backend chạy với dữ liệu mẫu, `npm run dev` trong `Frontend/Admin`, trình duyệt Chrome mới nhất.

## A. Luồng chính

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| FA-DN-01 | Chính | Đăng nhập | Ở trang `/login` | Nhập `admin` / `password`, bấm Đăng nhập | Chuyển sang `/dashboard`, thanh trên hiện tên và vai trò "Quản trị viên" | | Chưa chạy |
| FA-DN-02 | Chính | Giữ phiên khi tải lại | Đã đăng nhập | Bấm F5 ở `/dashboard` | Vẫn ở trang tổng quan, không bị đưa về `/login` | | Chưa chạy |
| FA-DN-03 | Chính | Đăng xuất | Đã đăng nhập | Bấm Đăng xuất | Về `/login`, token bị xóa khỏi localStorage | | Chưa chạy |
| FA-PQ-01 | Chính | Menu của ADMIN | Đăng nhập `admin` | Xem sidebar | Có đủ 12 mục: tổng quan, tour mẫu, tour thực tế, đơn hàng, khách hàng, khiếu nại, khuyến mãi, điều phối, tài chính, tài khoản, nhân sự, nhật ký | | Chưa chạy |
| FA-PQ-02 | Chính | Menu của SANPHAM | Đăng nhập `sanpham01` | Xem sidebar | Chỉ có tổng quan, tour mẫu, tour thực tế | | Chưa chạy |
| FA-PQ-03 | Chính | Menu của KINHDOANH | Đăng nhập `sales01` | Xem sidebar | Chỉ có tổng quan, đơn hàng, khách hàng, khiếu nại, khuyến mãi | | Chưa chạy |
| FA-PQ-04 | Chính | Menu của DIEUHANH | Đăng nhập `manager01` | Xem sidebar | Chỉ có tổng quan, tour thực tế, điều phối, nhân sự | | Chưa chạy |
| FA-PQ-05 | Chính | Menu của KETOAN | Đăng nhập `ketoan01` | Xem sidebar | Chỉ có tổng quan, đơn hàng, tài chính | | Chưa chạy |
| FA-HT-01 | Chính | Danh sách tài khoản | Đăng nhập `admin` | Vào `/system/accounts`, tìm theo tên, chuyển trang | Danh sách lọc đúng, phân trang hoạt động | | Chưa chạy |
| FA-HT-02 | Chính | Tạo tài khoản nhân viên | Như FA-HT-01 | Bấm Thêm, nhập đủ thông tin, Lưu | Thông báo thành công, tài khoản mới có trong danh sách | | Chưa chạy |
| FA-HT-03 | Chính | Phân quyền | Như FA-HT-01 | Mở hộp phân quyền của một nhân viên, đổi vai trò, Lưu | Vai trò mới hiển thị trong danh sách | | Chưa chạy |
| FA-HT-04 | Chính | Nhân sự và năng lực | Đăng nhập `manager01` | Vào `/system/hr`, mở hồ sơ một HDV, sửa năng lực | Lưu thành công, hồ sơ hiện năng lực mới | | Chưa chạy |
| FA-HT-05 | Chính | Nhật ký hệ thống | Đăng nhập `admin` | Vào `/system/logs` | Hiển thị nhật ký, có thao tác vừa làm ở FA-HT-02 | | Chưa chạy |
| FA-SP-01 | Chính | Tour mẫu | Đăng nhập `sanpham01` | Vào `/tour-template`, tạo tour mẫu, mở chi tiết, thêm lịch trình và dịch vụ | Lưu thành công, chi tiết hiện đủ lịch trình | | Chưa chạy |
| FA-SP-02 | Chính | Tour thực tế | Đăng nhập `sanpham01` | Vào `/tour-instance`, dùng trình tạo tour thực tế từ tour mẫu | Tour mới có trạng thái chờ kích hoạt | | Chưa chạy |
| FA-SP-03 | Chính | Dịch vụ và hành động xanh | Đăng nhập `admin` | Vào `/services` và `/green-actions`, thêm, sửa, xóa một mục | Danh sách cập nhật đúng sau mỗi thao tác | | Chưa chạy |
| FA-DH-01 | Chính | Phân công HDV | Đăng nhập `manager01`, có tour chờ kích hoạt | Vào `/dispatch/assign`, chọn tour, chọn HDV khả dụng, xác nhận | Phân công tạo thành công, tour hiện HDV chờ phản hồi | | Chưa chạy |
| FA-DH-02 | Chính | Danh sách HDV | Đăng nhập `manager01` | Vào `/dispatch/list`, mở hồ sơ HDV | Hiện thông tin, năng lực và lịch công tác | | Chưa chạy |
| FA-KD-01 | Chính | Đơn hàng | Đăng nhập `sales01` | Vào `/orders`, lọc theo trạng thái, mở chi tiết một đơn | Lọc đúng, chi tiết có hành khách, dịch vụ, thanh toán | | Chưa chạy |
| FA-KD-02 | Chính | Xác nhận thanh toán | Có đơn khách đã báo chuyển khoản | Mở chi tiết đơn, bấm Xác nhận | Đơn chuyển "Đã xác nhận" | | Chưa chạy |
| FA-KD-03 | Chính | Khách hàng | Đăng nhập `sales01` | Vào `/customers`, mở chi tiết khách | Hiện hồ sơ, hộ chiếu số, lịch sử đặt tour | | Chưa chạy |
| FA-KD-04 | Chính | Voucher | Đăng nhập `sales01` | Vào `/promotions`, tạo voucher, phân phối cho khách | Voucher mới có trong danh sách, khách được phân bổ | | Chưa chạy |
| FA-KD-05 | Chính | Khiếu nại | Có yêu cầu hỗ trợ | Vào `/complaints`, mở một khiếu nại, cập nhật xử lý | Trạng thái khiếu nại được cập nhật | | Chưa chạy |
| FA-KT-01 | Chính | Duyệt chi phí | Đăng nhập `ketoan01`, có chi phí chờ duyệt | Vào `/finance/cost-management`, duyệt một chi phí | Chi phí chuyển đã duyệt | | Chưa chạy |
| FA-KT-02 | Chính | Quyết toán | Có tour đã kết thúc | Vào `/finance/settlement`, tạo và chốt quyết toán | Quyết toán chốt thành công | | Chưa chạy |
| FA-KT-03 | Chính | Hoàn tiền | Có giao dịch hoàn tiền chờ | Vào `/finance/refund`, xử lý hoàn tiền | Giao dịch chuyển đã hoàn | | Chưa chạy |
| FA-KT-04 | Chính | Tổng quan và Power BI | Đăng nhập `ketoan01` | Vào `/dashboard`, mở hộp kết nối Power BI | Hiện số liệu tổng quan và thông tin kết nối | | Chưa chạy |

