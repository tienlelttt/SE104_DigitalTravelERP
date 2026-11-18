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

