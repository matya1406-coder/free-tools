# Bộ mẫu Excel: tính đến cuối tháng và sao chép bảng có ô gộp

Đây là tài liệu dùng ngay để dựng mẫu trong Excel, kèm công thức, dữ liệu giả và quy trình thao tác. Tài liệu này không phải tệp XLSX hoặc PDF đã xuất.

## Giả định và giới hạn

- “Số lượng” trong câu hỏi chưa có định nghĩa. Mẫu tách riêng số ngày và tổng số lượng, giả định số lượng mỗi ngày cố định do bạn nhập. Không suy ra doanh thu, tồn kho hoặc số giao dịch thực tế.
- Đếm ngày lịch, gồm cuối tuần và ngày lễ; tính cả ngày cuối tháng. Có lựa chọn tính hôm nay. Không tính giờ lẻ.
- Công thức dành cho Excel có EOMONTH (Excel 2007 trở lên). Ví dụ dùng tên hàm tiếng Anh và dấu phẩy. Nếu máy dùng dấu chấm phẩy làm dấu phân cách tham số, thay dấu phẩy bằng dấu chấm phẩy.
- Dữ liệu bên dưới đều giả. Thử trên sổ mới hoặc bản sao; không cần gửi dữ liệu cá nhân. Tên menu có thể khác giữa các bản Excel.

## 1. Tạo trang tính Ngay

Tạo trang tính tên `Ngay`. Nhập từng ô theo bảng. Cột A là nhãn; cột B là giá trị hoặc công thức. Định dạng B2 và B6 là ngày `dd/mm/yyyy`; B3, B7 là số nguyên; B4, B8 là số thường.

| Ô | Nội dung cột A tương ứng | Nhập vào cột B |
|---|---|---|
| B2 | Ngày tham chiếu | `=TODAY()` |
| B3 | Tính cả ngày tham chiếu? 1 = có, 0 = không | `1` |
| B4 | Số lượng mỗi ngày (cố định) | `4` |
| B5 | Đơn vị | `đơn vị` |
| B6 | Ngày cuối tháng | `=EOMONTH(B2,0)` |
| B7 | Số ngày được tính | `=MAX(0,B6-B2+B3)` |
| B8 | Tổng số lượng | `=B7*B4` |

Trong Data → Data Validation, đặt B2 là Date trong khoảng 01/01/1900–31/12/9998; B3 là Whole number từ 0 đến 1; B4 là Decimal lớn hơn hoặc bằng 0. Bật Error Alert kiểu Stop. Nhập ngày bằng bộ chọn ngày hoặc hàm DATE để tránh nhầm ngày/tháng. Kiểm tra lại đầu vào nếu dán dữ liệu vì thao tác dán có thể vượt qua xác thực.

Để thử với ngày cố định, thay B2 bằng `=DATE(2026,10,19)`. Sau khi thử, trả B2 về `=TODAY()`. TODAY dùng ngày trên máy và cập nhật khi Excel tính lại; bật Formulas → Calculation Options → Automatic nếu cần. B2 chỉ nên chứa ngày không kèm giờ. B4 phải là số, không nhập chuỗi như “4 sản phẩm”; ghi đơn vị ở B5.

### Ví dụ kiểm tra bằng tay

Mỗi dòng là một lần thay B2, B3, B4. Các kết quả dưới đây được đối chiếu bằng phép đếm ngày lịch, chưa chạy kiểm thử trong ứng dụng Excel tại môi trường soạn tài liệu.

| B2 (nhập bằng DATE) | B3 | B4 | B6 kỳ vọng | B7 kỳ vọng | B8 kỳ vọng |
|---|---:|---:|---|---:|---:|
| `=DATE(2026,10,19)` | 1 | 4 | 31/10/2026 | 13 | 52 |
| `=DATE(2026,10,19)` | 0 | 4 | 31/10/2026 | 12 | 48 |
| `=DATE(2026,10,31)` | 1 | 4 | 31/10/2026 | 1 | 4 |
| `=DATE(2026,10,31)` | 0 | 4 | 31/10/2026 | 0 | 0 |
| `=DATE(2028,2,28)` | 1 | 2 | 29/02/2028 | 2 | 4 |
| `=DATE(2027,2,28)` | 0 | 2 | 28/02/2027 | 0 | 0 |
| `=DATE(2026,12,1)` | 1 | 0 | 31/12/2026 | 31 | 0 |

Nếu chỉ cần số ngày, dùng B7 và bỏ qua B4/B8. Nếu số lượng khác nhau từng ngày, cần bảng ngày–số lượng riêng; phép nhân trong B8 không giải quyết trường hợp đó. Nếu chỉ đếm ngày làm việc, cần xác định ngày nghỉ tuần và danh sách ngày lễ trước khi dùng công thức khác.

## 2. Dựng bảng thử có ô gộp, màu và định dạng

Tạo trang `Mau` trống; nhập:

| Ô | Giá trị |
|---|---|
| A1 | BẢNG MINH HỌA |
| A2 | Nhóm |
| B2 | Mặt hàng |
| C2 | Số lượng |
| D2 | Ghi chú |
| A3 | Văn phòng |
| B3 | Bút |
| C3 | 4 |
| D3 | Dữ liệu giả |
| B4 | Giấy |
| C4 | 2 |
| D4 | Dữ liệu giả |
| A5 | Tổng |
| C5 | `=SUM(C3:C4)` |

Để B1:D1, A4, B5 và D5 trống. Sau đó:

1. Gộp A1:D1; gộp A3:A4; gộp A5:B5. Chỉ ô trên cùng bên trái của mỗi vùng có nội dung; Excel sẽ bỏ nội dung ở các ô còn lại nếu gộp ô đang có dữ liệu.
2. Tô A1:D1 xanh đậm, chữ trắng, đậm; A2:D2 xanh nhạt; A5:D5 vàng nhạt. Kẻ viền A1:D5; bật Wrap Text và căn giữa theo chiều dọc.
3. Đặt độ rộng cột A/B/C/D lần lượt 16/22/12/28. Đặt chiều cao hàng 1 = 30, hàng 2 = 24, hàng 3–4 = 32, hàng 5 = 24. Định dạng C3:C5 là số nguyên.

Sơ đồ ô gộp (không phải dữ liệu để dán):

```text
A1────────────────────────D1  tiêu đề gộp ngang
A2       B2       C2       D2
A3       B3       C3       D3
│        B4       C4       D4  A3:A4 gộp dọc
A5───────B5       C5       D5
```

### Sao chép bằng menu, không cần macro

1. Tạo trang `BanSao` mới, chưa gộp ô. Chọn toàn bộ `Mau!A1:D5`, bảo đảm không cắt ngang vùng gộp; nhấn Ctrl+C.
2. Chọn **chỉ ô A1** của BanSao rồi dùng Home → Paste → Keep Source Formatting (hoặc Paste All). Không dùng chỉ Values nếu muốn giữ màu, viền và ô gộp.
3. Khi vùng nguồn vẫn đang được sao chép, dùng Paste Special → Column widths tại A1 để giữ độ rộng cột. Nếu có lựa chọn Keep Source Column Widths, có thể dùng lựa chọn đó.
4. Chiều cao hàng có thể không đi theo khi sao chép một vùng. Đặt lại chiều cao hàng 1–5 theo các số ở trên. Độ rộng cột/chiều cao hàng áp dụng cho cả cột/hàng, nên dùng trang mới để tránh ảnh hưởng bảng khác.
5. Kiểm tra A1:D1, A3:A4, A5:B5 vẫn gộp; màu/viền đúng; C5 bằng 6; không có chữ bị che. Nếu chỉ cần bản sao toàn trang trên Excel máy tính, nhấp phải thẻ trang → Move or Copy → Create a copy để giữ bố cục trang thuận tiện hơn.

Nếu gặp thông báo các ô gộp phải cùng kích thước: hủy dán, kiểm tra nguồn đã chứa trọn mọi vùng gộp, rồi dán vào trang mới chưa gộp ô. Không gộp sẵn vùng đích. Nếu buộc dùng vùng đích cũ, sao lưu trước, kiểm tra toàn bộ vùng gộp giao với vùng đích và bỏ gộp chúng bằng Unmerge Cells; thao tác này có thể thay đổi cả phần nằm ngoài vùng dự định dán. Không bỏ gộp hoặc ghi đè vùng có dữ liệu cần giữ. Gỡ bảo vệ trang chỉ khi bạn có quyền.

### Nếu đang viết VBA để copy “mảng dữ liệu”

Gán `.Value` hoặc `.Value2` chỉ chuyển giá trị, không chuyển định dạng hoặc cấu trúc ô gộp. Nếu mục tiêu là sao chép nguyên bảng, dùng `Range.Copy`. Macro tùy chọn dưới đây dùng Excel máy tính có VBA, không chạy trên Excel web. Chỉ chạy trong sổ thử có trang `Mau` theo ví dụ; macro tạo trang mới để không ghi đè dữ liệu đích. Không cần macro để thực hiện cách bằng menu ở trên.

```vb
Option Explicit

Sub SaoChepBangMau()
    Dim src As Range, dst As Worksheet
    Dim i As Long
    ' Dat ma trong chinh so thu nghiem co trang Mau.
    Set src = ThisWorkbook.Worksheets("Mau").Range("A1:D5")
    Set dst = ThisWorkbook.Worksheets.Add( _
        After:=ThisWorkbook.Worksheets(ThisWorkbook.Worksheets.Count))

    src.Copy Destination:=dst.Range("A1")
    For i = 1 To src.Columns.Count
        dst.Columns(i).ColumnWidth = src.Columns(i).ColumnWidth
    Next i
    For i = 1 To src.Rows.Count
        dst.Rows(i).RowHeight = src.Rows(i).RowHeight
    Next i
    Application.CutCopyMode = False
End Sub
```

Macro là ví dụ độc lập, chưa được chạy trong Excel ở môi trường soạn tài liệu; không phải bản sửa cho hàm gốc vì chưa có mã nguồn của người hỏi. Mẫu XLSX dùng công thức và thao tác menu không cần lưu macro; nếu muốn lưu mã VBA thì dùng bản riêng định dạng XLSM.

Copy thông thường giữ công thức và Excel điều chỉnh tham chiếu tương đối theo vị trí đích. Tham chiếu có tên trang, tên vùng hoặc liên kết sổ khác cần kiểm tra riêng. Nếu chỉ muốn giữ số liệu hiện tại cùng định dạng, trước hết copy cả bảng vào trang mới, sau đó thử Paste Special → Values trên đúng vùng với cùng cấu trúc gộp; nếu Excel báo lỗi, dừng và chỉ chuyển các ô dữ liệu không gộp, không bỏ gộp hàng loạt. Không áp dụng hướng dẫn này cho đối tượng nổi, ảnh, biểu đồ hay bảng lọc phức tạp mà chưa thử riêng.

## Tiêu chí hoàn thành

- Các dòng kiểm tra ngày cho kết quả đúng; đổi B3 giữa 0 và 1 làm số ngày thay đổi đúng một ngày.
- Bảng sao chép có đủ chữ, số, công thức, ba vùng gộp, màu, viền và kích thước hàng/cột như nguồn.
- Tự kiểm tra trên phiên bản Excel sẽ dùng. Dùng mẫu cho tổ chức cá nhân có rủi ro thấp; không dựa vào ví dụ để ra quyết định tài chính hoặc nghiệp vụ quan trọng.

---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_4), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_4) من غير تسجيل.
