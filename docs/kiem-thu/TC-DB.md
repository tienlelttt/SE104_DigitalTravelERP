# Kịch bản kiểm thử – Cơ sở dữ liệu Oracle

- **Người viết:** Nguyễn Trần Thủy Tiên
- **Phạm vi:** script `database/ddl/KhoiTaoBang.sql` (31 bảng, các ràng buộc, chỉ mục, trigger), dữ liệu mẫu trong `database/seed`, các script `database/checks`.
- **Công cụ:** SQL Developer hoặc SQL*Plus, chạy trên một schema Oracle trống.

## A. Khởi tạo schema và dữ liệu mẫu

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| DB-KT-01 | Chính | Tạo bảng | Schema trống | Chạy `database/ddl/KhoiTaoBang.sql` | Chạy hết không lỗi. `SELECT COUNT(*) FROM user_tables` trả 31 | | Chưa chạy |
| DB-KT-02 | Chính | Đối tượng hợp lệ | Đã chạy DB-KT-01 | Chạy truy vấn số 1 trong `checks/kiem_tra_du_lieu.sql` | Không có trigger, function, procedure, view nào ở trạng thái INVALID | | Chưa chạy |
| DB-KT-03 | Chính | Chỉ mục | Đã chạy DB-KT-01 | `SELECT index_name FROM user_indexes WHERE index_name LIKE 'IDX_%' OR index_name LIKE 'UQ_%'` | Có đủ các chỉ mục `IDX_DONDATTOUR_*`, `IDX_GIAODICH_DATTOUR`, `UQ_CTDT_DATTOUR_KHACHHANG`, `UQ_CTDT_DATTOUR_NDH`… như trong script | | Chưa chạy |
| DB-KT-04 | Chính | Nạp dữ liệu mẫu | Đã chạy DB-KT-01 | Chạy `seed/data_v1.sql` rồi `seed/data_lien_ket.sql` | Chạy hết không lỗi khóa ngoại. `COMMIT` thành công | | Chưa chạy |
| DB-KT-05 | Chính | Số dòng sau khi nạp | Đã chạy DB-KT-04 | Chạy truy vấn số 2 trong `checks/kiem_tra_du_lieu.sql` | Các bảng `VAITRO`, `TAIKHOAN`, `NHANVIEN`, `TOURMAU`, `TOURTHUCTE`, `DONDATTOUR` đều có dữ liệu (số dòng > 0) | | Chưa chạy |
| DB-KT-06 | Chính | Tài khoản theo vai trò | Đã chạy DB-KT-04 | `SELECT VaiTro, COUNT(*) FROM TAIKHOAN GROUP BY VaiTro` | Có đủ 7 vai trò: ADMIN, DIEUHANH, SANPHAM, KINHDOANH, KETOAN, HDV, KHACHHANG | | Chưa chạy |
| DB-KT-07 | Chính | Reset schema | Schema đã có dữ liệu | Chạy `checks/reset_schema_sach.sql`, sau đó chạy lại DB-KT-01 và DB-KT-04 | Schema được xóa sạch rồi tạo lại thành công, kết quả giống lần đầu | | Chưa chạy |
| DB-KT-08 | Chính | Kiểm tra trùng khóa ngoại mồ côi | Đã chạy DB-KT-04 | `SELECT COUNT(*) FROM DONDATTOUR d WHERE NOT EXISTS (SELECT 1 FROM TOURTHUCTE t WHERE t.MaTourThucTe = d.MaTourThucTe)` | Trả 0 | | Chưa chạy |

## B. Ràng buộc toàn vẹn

Mỗi test case chạy một câu lệnh vi phạm ràng buộc. Kết quả mong đợi là Oracle báo lỗi và dữ liệu không đổi. Chạy `ROLLBACK` sau mỗi test case.

| Mã TC | Loại | Chức năng | Tiền điều kiện | Các bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|---|---|
| DB-RB-01 | Lỗi | Trạng thái tài khoản | Có dữ liệu mẫu | `UPDATE TAIKHOAN SET TrangThai = 'XOA' WHERE TenDangNhap = 'khach01'` | Lỗi ORA-02290 vi phạm `CK_TAIKHOAN_TTHAI` | | Chưa chạy |
| DB-RB-02 | Lỗi | Điểm xanh không âm | Có dữ liệu mẫu | Cập nhật `HOCHIEUSO.DiemXanh = -1` cho một khách hàng | Lỗi vi phạm `CK_HCS_DiemXanh` | | Chưa chạy |
| DB-RB-03 | Lỗi | Hạng thành viên | Có dữ liệu mẫu | Cập nhật `HOCHIEUSO.HangThanhVien = 'BACH_KIM'` | Lỗi vi phạm `CK_HCS_HangTV` | | Chưa chạy |
| DB-RB-04 | Lỗi | Thời lượng tour mẫu | Có dữ liệu mẫu | Thêm `TOURMAU` với `ThoiLuong = 0` | Lỗi vi phạm `CK_TOURMAU_ThoiLuong` | | Chưa chạy |
| DB-RB-05 | Lỗi | Số khách tour thực tế | Có dữ liệu mẫu | Cập nhật một `TOURTHUCTE` có `SoKhachToiThieu` lớn hơn `SoKhachToiDa` | Lỗi vi phạm `CK_TTT_SoKhach` | | Chưa chạy |
| DB-RB-06 | Lỗi | Chỗ còn lại | Có dữ liệu mẫu | Cập nhật `ChoConLai` lớn hơn `SoKhachToiDa` | Lỗi vi phạm `CK_TTT_ChoConLai` | | Chưa chạy |
| DB-RB-07 | Lỗi | Trạng thái đơn đặt tour | Có dữ liệu mẫu | Cập nhật `DONDATTOUR.TrangThai = 'HOAN_THANH'` | Lỗi vi phạm `CK_DDT_TrangThai` | | Chưa chạy |
| DB-RB-08 | Lỗi | Thành tiền dịch vụ | Có dữ liệu mẫu | Thêm `CHITIETDICHVU` với `SoLuong = 2`, `DonGia = 100000`, `ThanhTien = 150000` | Lỗi vi phạm `CK_CTDV_ThanhTien_Tinh` | | Chưa chạy |
| DB-RB-09 | Lỗi | Voucher phần trăm | Có dữ liệu mẫu | Thêm `VOUCHER` loại `PHAN_TRAM` với `GiaTriGiam = 120` | Lỗi vi phạm `CK_VOUCHER_PhanTram` | | Chưa chạy |
| DB-RB-10 | Lỗi | Ngày hiệu lực voucher | Có dữ liệu mẫu | Thêm `VOUCHER` có `NgayHieuLuc` sau `NgayHetHan` | Lỗi vi phạm `CK_VOUCHER_NgayHL` | | Chưa chạy |
| DB-RB-11 | Lỗi | Lượt dùng voucher | Có dữ liệu mẫu | Cập nhật `SoLuotDaDung` lớn hơn `SoLuotPhatHanh` | Lỗi vi phạm `CK_VOUCHER_DaDung` | | Chưa chạy |
| DB-RB-12 | Lỗi | Khách trùng trong một đơn | Có một đơn có chi tiết khách hàng | Thêm lại `CHITIETDATTOUR` cùng đơn và cùng mã khách hàng | Lỗi trùng chỉ mục duy nhất `UQ_CTDT_DATTOUR_KHACHHANG` | | Chưa chạy |
| DB-RB-13 | Lỗi | Khóa ngoại đơn đặt tour | Có dữ liệu mẫu | Thêm `DONDATTOUR` với `MaTourThucTe` không tồn tại | Lỗi ORA-02291 vi phạm khóa ngoại | | Chưa chạy |
| DB-RB-14 | Lỗi | Mở bán tour khi chưa có HDV | Một tour `CHO_KICH_HOAT` chưa có phân công HDV được nhận | Cập nhật `TOURTHUCTE.TrangThai = 'MO_BAN'` | Trigger `TRG_TTT_OPEN_REQUIRE_HDV` chặn và báo lỗi | | Chưa chạy |
| DB-RB-15 | Lỗi | Đánh giá năng lực | Có dữ liệu mẫu | Cập nhật `NANGLUCNHANVIEN.DanhGia = 6` | Lỗi vi phạm `CK_NLNV_DanhGia` | | Chưa chạy |
