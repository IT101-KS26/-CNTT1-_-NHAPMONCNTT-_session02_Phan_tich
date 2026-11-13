# Bài tập về nhà


---

## Bước 1: Đề xuất 2 prompt

### Giải pháp 1: Đơn giản, ngắn gọn

````markdown
Bạn là nhân viên nhập đơn hàng. Hãy trích xuất tên khách, SĐT, địa chỉ, món, số lượng
và tổng tiền từ các tin nhắn dưới đây, trả về JSON.

Menu: Cà phê sữa đá 29000, Trà đào cam sả 35000, Bánh mì que 15000.

Ví dụ:
Input: "Anh Tuấn 0987654321, 5 Hai Bà Trưng, 1 bánh mì que"
Output: {"customer_name":"Anh Tuấn","phone":"0987654321","address":"5 Hai Bà Trưng","items":[{"product":"Bánh mì que","qty":1}],"total":15000}

Tin nhắn:
1. "Chị Lan, 0912345678, 12 Lê Lợi Q1, lấy 2 cà phê sữa đá và 1 trà đào cam sả"
2. "Anh Minh đặt 3 bánh mì que giao quận 3"
3. "Em Hoa 09123 ship 5 trà sữa trân châu tới 45 Nguyễn Huệ"
````

### Giải pháp 2: Chi tiết, nhiều ràng buộc (khung CREATE)

````markdown
# C – CHARACTER
Bạn là nhân viên nhập liệu đơn hàng của cửa hàng "Sài Gòn Xưa". Chính xác, không suy đoán, không bịa dữ liệu.

# R – REQUEST
Đọc tin nhắn trong <INPUT>, trích xuất từng đơn, đối chiếu <MENU>, tính tiền, kiểm tra lỗi theo <RULES>.

<MENU>
- Cà phê sữa đá: 29000
- Trà đào cam sả: 35000
- Bánh mì que: 15000
</MENU>

<RULES>
1. Thiếu tên/SĐT/địa chỉ → null, status = "NEED_INFO", ghi vào issues.
2. SĐT hợp lệ = đúng 10 chữ số, bắt đầu bằng 0. Sai → phone = null, ghi lỗi.
3. Địa chỉ phải có số nhà + tên đường; chỉ có quận/phường → coi là thiếu.
4. Món không có trong MENU → không đưa vào items, ghi lỗi, KHÔNG tự đặt giá.
5. Số lượng thiếu/không hợp lệ → ghi lỗi, không đoán.
6. Không có món hợp lệ → status = "ERROR", total = 0.
7. Đầy đủ và hợp lệ → status = "VALID".
8. subtotal = qty × unit_price; total = tổng subtotal.
</RULES>

# E – EXAMPLE (Few-shot)
Input: "Chị Mai lấy 2 sinh tố bơ, sđt 123"
Output: {"order_id":"DH00","customer_name":"Chị Mai","phone":null,"address":null,"items":[],"total":0,"status":"ERROR","issues":["Món 'sinh tố bơ' không có trong menu","SĐT '123' không hợp lệ","Thiếu địa chỉ"]}

# A – ADJUSTMENT
Chỉ dùng dữ liệu trong tin nhắn và MENU. Không chào hỏi, không giải thích dài. Tiền là số nguyên VNĐ.

# T – TYPE OF OUTPUT
1. Mảng JSON hợp lệ trong khối ```json (các trường: order_id, customer_name, phone, address, items[product, qty, unit_price, subtotal], total, status, issues).
2. Bảng Markdown: | Mã đơn | Khách | SĐT | Tổng tiền | Trạng thái | Vấn đề |

# E – EXTRA
Rà lại từng điều kiện trong RULES trước khi trả kết quả.

<INPUT>
1. "Chị Lan, 0912345678, 12 Lê Lợi Q1, lấy 2 cà phê sữa đá và 1 trà đào cam sả"
2. "Anh Minh đặt 3 bánh mì que giao quận 3"
3. "Em Hoa 09123 ship 5 trà sữa trân châu tới 45 Nguyễn Huệ"
</INPUT>
````

---

## Bước 2: Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
|---|---|---|
| Độ dễ hiểu / dễ viết | Cao | Trung bình |
| Tốn token | Ít | Nhiều hơn |
| Chặn lỗi biên (thiếu/sai dữ liệu) | Trung bình: AI dễ tự đoán | Cực tốt: có quy tắc cho từng bẫy |
| Tính nhất quán của output | Thấp: không ép `status`, `issues` | Cao: cấu trúc cố định, có JSON + bảng |
| Rủi ro bịa dữ liệu (giá, địa chỉ) | Cao: không cấm rõ | Thấp: có luật "không tự đặt giá" |
| Phù hợp tích hợp hệ thống | Kém: phải tự kiểm tra thêm | Tốt: dữ liệu sẵn sàng nhập |

**Điểm yếu cụ thể của Giải pháp 1 với dữ liệu thử:** không có luật về món ngoài menu nên AI có thể tự đặt giá cho "trà sữa trân châu"; không định nghĩa địa chỉ hợp lệ nên có thể chấp nhận "quận 3"; không có `status`/`issues` nên người nhập không biết đơn nào cần gọi lại khách.

---

## Bước 3: Lựa chọn và chạy thử

**Chọn: Giải pháp 2.**

**Lý do:** Đây là bài toán nhập liệu thật, sai một đơn là mất tiền hoặc giao nhầm. Chi phí token tăng thêm là nhỏ so với rủi ro của dữ liệu sai, còn prompt có thể dùng lại nhiều lần nên chỉ cần viết một lần.

**Kết quả chạy thử Giải pháp 2:**

```json
[
  {
    "order_id": "DH01",
    "customer_name": "Chị Lan",
    "phone": "0912345678",
    "address": "12 Lê Lợi, Q1",
    "items": [
      {"product": "Cà phê sữa đá", "qty": 2, "unit_price": 29000, "subtotal": 58000},
      {"product": "Trà đào cam sả", "qty": 1, "unit_price": 35000, "subtotal": 35000}
    ],
    "total": 93000,
    "status": "VALID",
    "issues": []
  },
  {
    "order_id": "DH02",
    "customer_name": "Anh Minh",
    "phone": null,
    "address": null,
    "items": [
      {"product": "Bánh mì que", "qty": 3, "unit_price": 15000, "subtotal": 45000}
    ],
    "total": 45000,
    "status": "NEED_INFO",
    "issues": ["Thiếu SĐT", "Địa chỉ 'quận 3' thiếu số nhà và tên đường"]
  },
  {
    "order_id": "DH03",
    "customer_name": "Em Hoa",
    "phone": null,
    "address": "45 Nguyễn Huệ",
    "items": [],
    "total": 0,
    "status": "ERROR",
    "issues": ["SĐT '09123' không hợp lệ", "Món 'trà sữa trân châu' không có trong menu"]
  }
]
```

| Mã đơn | Khách | SĐT | Tổng tiền | Trạng thái | Vấn đề |
|---|---|---|---|---|---|
| DH01 | Chị Lan | 0912345678 | 93.000đ | VALID | Không |
| DH02 | Anh Minh | (thiếu) | 45.000đ | NEED_INFO | Thiếu SĐT, địa chỉ chưa đủ |
| DH03 | Em Hoa | (sai) | 0đ | ERROR | SĐT sai, món ngoài menu |

