# Bảng tính ngày / Personal date calculation kit

Đây là mẫu công thức có thể dùng ngay trong Excel, kèm mã tính mốc cho n8n. Đây là tài liệu văn bản, không phải tệp XLSX đính kèm. / This is a usable Excel template plus n8n milestone code, supplied as text rather than an attached XLSX workbook.

## Giả định / Assumptions

- “Số lượng” is assumed to mean calendar days. The quotation does not establish whether the requester means days, records or product quantities. Formulas for each interpretation appear below.
- Use date-only values, without times. Elapsed days exclude the start date. A relationship starting today has elapsed zero days. “100 days” means start + 100, not the 100th day under inclusive counting (start + 99).
- Six months means six calendar months; one year means twelve calendar months. Missing target days clamp to the target month's last day. February 29, 2024 + one year = February 28, 2025.
- End/reference date must be on or after the start. Future starts are rejected in this template. Inputs must be actual Excel date values, not ambiguous text. Supported planning dates: 1901–9998, with milestone years at most 9999.
- Excel formulas below use English function names and commas. If your Excel expects semicolons, replace argument commas with semicolons. Set calculation to Automatic. TODAY updates on recalculation, not continuously while the workbook is closed.

## 1. Mẫu Excel dùng ngay / Ready-to-enter Excel template

Create sheets `VI` and `EN`. On each sheet, put the corresponding language label in column A and the value or formula in column B. Both sheets use identical cell addresses and formulas; each has independent inputs. Format B2:B3, B5 and B10:B12 as `yyyy-mm-dd`; format B6:B9 as numbers with zero decimals. Give B2:B3 a pale yellow fill to mark the inputs. No macros or external connections are needed.

Tạo hai trang `VI` và `EN`, dùng cùng địa chỉ ô và công thức. Cột A là nhãn theo ngôn ngữ; cột B là dữ liệu/công thức. Ô B2 và B3 là ô nhập. Định dạng các ô ngày là `yyyy-mm-dd`, các ô số ngày là số nguyên.

| Cell | Nhãn VI (column A) | EN label (column A) | Enter in column B |
|---|---|---|---|
| B2 | Ngày bắt đầu | Start date | `=DATE(2026,7,11)` |
| B3 | Ngày tham chiếu / kết thúc | Reference / end date | `=DATE(2026,10,19)` |
| B4 | Kiểm tra dữ liệu | Input check | Formula below |
| B5 | Ngày cuối tháng | Month-end date | `=IF(B4<>"OK","",EOMONTH(B3,0))` |
| B6 | Số ngày còn lại (không tính ngày tham chiếu) | Days left (excluding reference date) | `=IF(B4<>"OK","",B5-B3)` |
| B7 | Số ngày còn lại (tính cả ngày tham chiếu) | Days left (including reference date) | `=IF(B4<>"OK","",B5-B3+1)` |
| B8 | Số ngày đã qua | Elapsed days | `=IF(B4<>"OK","",B3-B2)` |
| B9 | Số ngày của khoảng (tính cả hai đầu) | Days in range (both dates included) | `=IF(B4<>"OK","",B3-B2+1)` |
| B10 | Mốc 100 ngày | 100-day milestone | `=IF(B4<>"OK","",B2+100)` |
| B11 | Mốc 6 tháng | Six-month milestone | `=IF(B4<>"OK","",EDATE(B2,6))` |
| B12 | Mốc 1 năm | One-year milestone | `=IF(B4<>"OK","",EDATE(B2,12))` |

B4, on both sheets:

```excel
=IFERROR(IF(AND(ISNUMBER(B2),ISNUMBER(B3),B2=INT(B2),B3=INT(B3),B2>=DATE(1901,1,1),B3>=B2,B2<=DATE(9998,12,31),B3<=DATE(9998,12,31)),"OK","CHECK"),"CHECK")
```

Add these instructions to A15:A18 on the appropriate sheet:

| VI | EN |
|---|---|
| Chỉ sửa B2 và B3. Dùng DATE(năm,tháng,ngày) để tránh nhầm định dạng. | Edit B2 and B3 only. Use DATE(year,month,day) to avoid ambiguous formats. |
| Để tính từ hôm nay, thay B3 bằng =TODAY(). | To calculate from today, replace B3 with =TODAY(). |
| CHECK: nhập ngày hợp lệ, không có giờ; ngày kết thúc không trước ngày bắt đầu; năm từ 1901 đến 9998. | CHECK: enter valid dates without times; end must not precede start; years must be 1901–9998. |
| Không gửi thông báo. Ngày hôm nay chỉ cập nhật khi bảng tính tính lại. | No notifications. Today updates when the workbook recalculates. |

For the fixed example, expect B5 = 2026-10-31, B6 = 12, B7 = 13, B8 = 100, B9 = 101, B10 = 2026-10-19, B11 = 2027-01-11 and B12 = 2027-07-11. Excel DATE normalizes impossible dates, so enter a real calendar date: DATE(2026,2,30) becomes March 2 and cannot be distinguished afterward.

### Trả lời trực tiếp câu hỏi cuối tháng / Direct month-end solution

Không tính hôm nay / Exclude today:
```excel
=EOMONTH(TODAY(),0)-TODAY()
```
Tính cả hôm nay / Include today:
```excel
=EOMONTH(TODAY(),0)-TODAY()+1
```

Nếu “số lượng” là số dòng có ngày ở A2:A1000, dùng công thức sau. Điều kiện nhỏ hơn ngày đầu tháng sau cũng tính đúng các ô có giờ trong ngày cuối tháng. / If quantity means rows dated this month from today onward, use:
```excel
=COUNTIFS(A2:A1000,">="&TODAY(),A2:A1000,"<"&EOMONTH(TODAY(),0)+1)
```

Nếu cần cộng số lượng ở B2:B1000 theo ngày ở A2:A1000 / To sum quantities instead:
```excel
=SUMIFS(B2:B1000,A2:A1000,">="&TODAY(),A2:A1000,"<"&EOMONTH(TODAY(),0)+1)
```

These alternatives include today, require real Excel dates, and assume columns A/B contain source records on a separate data sheet, not the calculator layout above. Working days, holidays and shifts require different rules.

## 2. n8n: relationship milestone calculation

This code supplies the calculation portion of the quoted workflow. A Code node alone does not schedule or deliver notifications. Use a Code node in **Run Once for All Items** mode after mapping a form or webhook body to these top-level fields:

```json
{"relationshipId":"demo-1","startDate":"2026-07-11","today":"2026-10-19"}
```

`today` must be the current calendar date in the recipient's chosen timezone, supplied upstream as `YYYY-MM-DD`. UTC below is only used to represent calendar dates and avoid daylight-saving arithmetic errors. No personal timezone is inferred. Each input produces either calculated output or an error without stopping other records.

```javascript
const DAY = 86400000;
function parseDate(value) {
  if (typeof value !== 'string' || !/^\d{4}-\d{2}-\d{2}$/.test(value))
    throw new Error('Use YYYY-MM-DD');
  const [y, m, d] = value.split('-').map(Number);
  if (y < 1901 || y > 9998) throw new Error('Year must be 1901–9998');
  const date = new Date(Date.UTC(y, m - 1, d));
  if (date.getUTCFullYear() !== y || date.getUTCMonth() !== m - 1 || date.getUTCDate() !== d)
    throw new Error('Invalid calendar date');
  return date;
}
const iso = date => date.toISOString().slice(0, 10);
function addMonths(date, count) {
  const first = new Date(Date.UTC(date.getUTCFullYear(), date.getUTCMonth() + count, 1));
  const lastDay = new Date(Date.UTC(first.getUTCFullYear(), first.getUTCMonth() + 1, 0)).getUTCDate();
  first.setUTCDate(Math.min(date.getUTCDate(), lastDay));
  return first;
}
return $input.all().map((item, index) => {
  try {
    const { relationshipId, startDate, today } = item.json;
    if (typeof relationshipId !== 'string' || !relationshipId.trim())
      throw new Error('A stable relationshipId is required');
    const start = parseDate(startDate), now = parseDate(today);
    if (start > now) throw new Error('Start date is in the future');
    const monthEnd = new Date(Date.UTC(now.getUTCFullYear(), now.getUTCMonth() + 1, 0));
    const milestones = [
      ['100-days', new Date(start.getTime() + 100 * DAY)],
      ['6-months', addMonths(start, 6)],
      ['1-year', addMonths(start, 12)]
    ].map(([name, date]) => ({
      name, date: iso(date), daysUntil: (date - now) / DAY,
      dueToday: iso(date) === today,
      eventKey: JSON.stringify([relationshipId, startDate, name, iso(date)])
    }));
    return { json: { relationshipId, startDate, today,
      elapsedDays: (now - start) / DAY,
      daysLeftInMonth: (monthEnd - now) / DAY,
      daysLeftIncludingToday: (monthEnd - now) / DAY + 1,
      milestones, dueToday: milestones.filter(m => m.dueToday)
    }, pairedItem: { item: index } };
  } catch (error) {
    return { json: { error: error.message }, pairedItem: { item: index } };
  }
});
```

Example result: elapsedDays = 100, daysLeftInMonth = 12; the only dueToday event is `100-days` on 2026-10-19. The other milestones are 2027-01-11 and 2027-07-11.

### Connecting notifications (separate from the workbook)

1. Form/Webhook → map and validate fields → store the stable relationship ID, start date, recipient timezone and explicitly chosen notification destination. Do not rely on the original webhook firing again months later.
2. Schedule Trigger → read saved records daily → calculate each recipient's local `today` → run the Code node above. A single-timezone pilot can run daily at 09:00 in that explicitly configured timezone.
3. Route error records for correction. Split `dueToday` into individual events; look up each `eventKey` in persistent delivery storage before sending through the selected notification node.
4. Store successful deliveries by eventKey. Use serialized processing and provider idempotency keys where supported. A crash after sending but before saving can otherwise cause duplicates; this simple design does not promise exactly-once delivery.
5. Exact-day mode misses milestones during downtime. For catch-up, store the last successful check date per relationship, select milestone dates in `(lastCheckedDate, today]`, and suppress already delivered keys. On first activation, set the baseline to the previous day to avoid sending all historical milestones. Advance the checkpoint only after processing succeeds.

Notifications require persistent storage, a running n8n instance and a configured delivery provider; provider costs and credentials are outside the free spreadsheet's scope. These are one-time 100-day, six-month and one-year events, not recurring anniversaries.

## 3. Acceptance cases / Các trường hợp kiểm tra

Use fixed inputs instead of TODAY for repeatable checks. These are expected results, not a claim that Excel or a live n8n installation was tested.

| Start | Reference | Elapsed | Month days left (exclusive / inclusive) | +100 days | +6 months | +1 year |
|---|---|---:|---|---|---|---|
| 2026-07-11 | 2026-10-19 | 100 | 12 / 13 | 2026-10-19 | 2027-01-11 | 2027-07-11 |
| 2024-02-29 | 2024-02-29 | 0 | 0 / 1 | 2024-06-08 | 2024-08-29 | 2025-02-28 |
| 2025-08-31 | 2025-08-31 | 0 | 0 / 1 | 2025-12-09 | 2026-02-28 | 2026-08-31 |
| 2025-12-31 | 2026-01-01 | 1 | 30 / 31 | 2026-04-10 | 2026-06-30 | 2026-12-31 |

Also check: blank or text Excel inputs → CHECK and blank outputs; end before start → CHECK / code error; timestamp input → CHECK / code error; n8n `2026-02-30` → error. Month-end on 2025-02-28 gives 0 / 1; on 2024-02-28 gives 1 / 2. Re-running a notification event should be suppressed by persistent eventKey storage, not by this calculation code alone.

---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_7), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_7) من غير تسجيل.
