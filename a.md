# MINI PROJECT: HỆ THỐNG QUẢN TRỊ HẠ TẦNG VÀ NHẬT KÝ KỸ THUẬT GREENTECH
## 2. PHÂN TÍCH HỆ THỐNG THEO IPO VÀ DIKW
### 2.1 IPO
| Thành phần | Áp dụng tại Green Tech |
|---|---|
| Input | Log máy chủ, danh mục cấu hình phần cứng, mã linh kiện tại 100 trạm |
| Process | Lọc log theo dòng; chuẩn hóa mã bằng TRIM; đối soát khóa bằng VLOOKUP chính xác; kiểm tra mã trùng hoặc không tồn tại |
| Output | Báo cáo sự cố được nối vào backup, bảng trạm có tên thiết bị và thông số, tài liệu hướng dẫn vận hành |

### 1.2. Mô hình DIKW

- **Data — Dữ liệu:** một dòng log ghi `ERROR 404`, một mã cấu hình có khoảng trắng, một giá trị RAM.
- **Information — Thông tin:** danh sách các dòng sự cố kèm thời điểm và mã trạm; bảng mã đã được làm sạch và tra cứu.
- **Knowledge — Kiến thức:** khi nhiều TIMEOUT cùng xuất hiện ở một tuyến mạng, kỹ thuật viên đối chiếu thời gian và trạm để hình thành giả thuyết về đường truyền. Một dòng log riêng lẻ chưa đủ chứng minh nguyên nhân. 
- **Wisdom — Quyết định:** ưu tiên kiểm tra tuyến mạng ảnh hưởng nhiều trạm, xác nhận nguyên nhân trước khi thay linh kiện, giữ báo cáo làm bằng chứng vận hành.

## 2. Cây thư mục ba cấp và bốn trụ cột

Ba cấp của dữ liệu chính là **gốc dự án → nhóm chức năng → tệp**. Tên thư mục dự án `greentech-system` 

```text
greentech-system/
├── docs/
│   ├── README.md
│   ├── srs-he-thong.docx
│   └── kiem-tra-nghiem-thu.md
├── scripts/
│   ├── auto_report.bat
│   └── auto-report.sh
├── data/
│   ├── log-he-thong.log
│   └── quan-ly-thiet-bi.xlsx
├── backup/
│   └── bao-cao-su-co.txt
├── .env
├── .gitignore
├── .gitattributes
└── README.md
```
Các tệp tạo dữ liệu và kiểm thử bổ sung nằm trong `scripts/`. Nếu có cấu hình CI, `.`.github/workflows/` tuân theo vị trí bắt buộc của GitHub và không thay thế bốn trụ cột dữ liệu chính.

| Trụ cột | Trách nhiệm | Quy tắc |
|---|---|---|
| docs | Đặc tả và hướng dẫn | Cập nhật cùng phiên bản script và workbook |
| scripts | Tự động hóa | Audit trước khi chạy; không có lệnh xóa dữ liệu |
| data | Đầu vào gốc và sổ thiết bị | Không sửa log đầu vào trong khi tạo báo cáo |
| backup | Báo cáo phục vụ truy vết | Ghi nối từng đợt; giữ lại báo cáo trước đó |

`auto_report.bat` giữ nguyên tên theo yêu cầu đề bài. Tên repository nộp bài là `HCM-KS26-CNTT1_NhapMonCNTT_MiniProject_GreenTech`; khi clone có thể chỉ định thư mục đích `greentech-system` để tuân thủ kebab-case tại máy local.

### 1.1.

Bash trên macOS hoặc Linux:

```bash
mkdir -p greentech-system/{docs,scripts,data,backup}
```

## Phần 2. Module 2 — Robot lọc log tự động

### 2.1. Quy tắc lọc

Giữ lại các dòng có chứa **CRITICAL** và **ERROR** từ file log 500 MB.

Sử dụng lệnh grep trên macOS:

```bash
grep -E "CRITICAL|ERROR" log-he-thong.log
```

### 2.2. Chức năng

Tự động quét tệp log-he-thong.log (có thể lên tới 500MB), lọc ra các dòng nghiêm trọng và ghi nối kết quả vào báo cáo, mà không cần mở toàn bộ tệp trong RAM.

### 2.3. Quy trình xử lý

Dùng lệnh echo để in tiêu đề báo cáo (kèm ngày giờ chạy).

Dùng findstr (Windows) hoặc grep (Linux/macOS) để tìm các dòng chứa "CRITICAL" hoặc "ERROR".

Kết hợp toán tử Pipe (|) để dòng dữ liệu chảy trực tiếp từ lệnh đọc file sang lệnh lọc.

Dùng toán tử >> để nối kết quả vào cuối tệp bao-cao-su-co.txt, tránh ghi đè báo cáo cũ.

Lưu tệp báo cáo vào thư mục backup/.

### 2.4. Script mẫu (auto_report.bat)

```bat
@echo off
echo ===== BAO CAO SU CO HE THONG ===== >> backup\bao-cao-su-co.txt
echo Thoi gian chay: %date% %time% >> backup\bao-cao-su-co.txt

type data\log-he-thong.log | findstr /I "CRITICAL ERROR TIMEOUT" >> backup\bao-cao-su-co.txt

echo Hoan tat loc log.
```

### 2.5. Output mẫu (backup/bao-cao-su-co.txt)

```text
===== BAO CAO SU CO HE THONG =====
Thoi gian chay: Mon 08/03/2026 10:00:00
[10:00:12] ERROR 404 - tram-cam-bien-27 khong phan hoi
[10:00:45] CRITICAL - mat ket noi may chu trung tam
[10:01:03] TIMEOUT - tram-cam-bien-58 qua thoi gian cho
```

### 2.6. Lưu ý / Bẫy

Dùng nhầm toán tử > thay vì >> sẽ ghi đè và xóa mất báo cáo cũ.

Phải có bước Audit (thẩm định) trước khi chạy bất kỳ lệnh xóa nào do AI gợi ý, tuyệt đối tránh del /f /s /q khi chưa xác nhận đường dẫn.

## Phần 3. Module 3: Tài liệu hướng dẫn vận hành

### 3.1. Chức năng

Chuẩn hóa tài liệu nội bộ để nhân viên mới có thể vận hành hệ thống mà không cần hỏi lại kỹ sư phụ trách.

### 3.2. Quy trình xử lý — README.md

Đặt tên hệ thống ở cấp Heading 1 (#).

Viết hướng dẫn chạy script trong khối Code Block.

Tạo Checklist kiểm tra thiết bị bằng Task List (- [ ]).

Trình bày thông số phần cứng bằng bảng Markdown (Table).

### 3.3. README.md mẫu

```markdown
# He thong quan tri ha tang GreenTech K26

## Cach chay script loc log
cd scripts
auto_report.bat

## Checklist kiem tra thiet bi
- [ ] Kiem tra ket noi tram cam bien
- [ ] Kiem tra dung luong RAM con trong
- [ ] Sao luu bao cao vao backup/

| Tram | RAM | Ngay lap dat |
|------|-----|--------------|
| tram-01 | 4GB | 2026-01-10 |
```

### 3.4. Quy trình xử lý — SRS Document (Word)

Gán nhãn Heading Styles (1, 2, 3) cho toàn bộ tiêu đề mục lục.

Chèn mục lục tự động (Auto TOC) ngay sau trang bìa.

Kiểm tra TOC cập nhật đúng số trang trước khi xuất bản.

## Phần 4. Module 4: Đối soát tài nguyên (Excel)

### 4.1. Chức năng

Tự động điền tên thiết bị vào bảng trạng thái trạm dựa trên mã linh kiện, tránh nhập tay gây sai lệch dữ liệu.

### 4.2. Quy trình xử lý

Làm sạch dữ liệu: dùng hàm TRIM để loại bỏ khoảng trắng thừa trong cột ma_linh_kien.

Xác định vùng dữ liệu tra cứu (bảng danh-muc-linh-kien) và cố định vùng bằng dấu $.

Viết công thức VLOOKUP với tham số FALSE để chỉ nhận kết quả trùng khớp chính xác.

Kiểm tra lỗi #N/A — nếu có, quay lại bước làm sạch dữ liệu.

Áp dụng AutoFilter để kiểm tra nhanh theo từng trạm.

### 4.3. Công thức mẫu

```excel
=TRIM(A2)
=VLOOKUP(TRIM(B2), 'danh-muc-linh-kien'!$A$2:$D$100, 2, FALSE)
```

### 4.4. Lưu ý / Bẫy

Vùng dữ liệu chứa hàng trống hoặc ô Merge Cell sẽ làm AutoFilter hoạt động sai.

Không cố định vùng dò tìm bằng $ sẽ khiến công thức lệch khi sao chép xuống các dòng khác.

## Phần 5. Các bẫy dữ liệu cần xử lý

| Bẫy | Rủi ro nếu không xử lý | Cách phòng tránh |
|---|---|---|
| Tên thư mục có dấu / khoảng trắng | Script và đường dẫn chạy lỗi trên nhiều hệ điều hành | Luôn đặt tên theo kebab-case, không dấu |
| Lệnh xóa nguy hiểm (del /f /s /q) | Xóa nhầm toàn bộ dữ liệu dự án | Bắt buộc bước Audit trước khi thực thi |
| Hàng trống / Merge Cell trong Excel | AutoFilter và VLOOKUP trả kết quả sai | Chuẩn hóa vùng dữ liệu liền mạch |
| Khoảng trắng thừa trong mã linh kiện | VLOOKUP trả lỗi #N/A | Dùng hàm TRIM trước khi đối chiếu |
| Dùng > thay vì >> khi ghi log | Mất toàn bộ báo cáo cũ | Luôn dùng >> khi cần nối thêm dữ liệu |
| Tính sai đơn vị tốc độ mạng | Ước lượng sai thời gian tải log | Quy đổi Mbps sang MB/s theo hệ số 8 |
