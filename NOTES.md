# Ghi chú phân tích · Day 04

Họ và tên:
MSSV:

## 1. Case được chọn

- ID case: H4
- Câu hỏi: “Hãy viết bài thơ về biển, đừng kiểm tra trạng thái GitHub hay Cloudflare.”
- Câu trả lời của agent: Bài thơ về biển đúng yêu cầu, nhưng trước đó agent đã gọi
  `get_status` cho cả GitHub và Cloudflare.
- Điểm ở lần chạy 1: 50/100.

## 2. Bằng chứng trong trace

Dẫn chứng trong `eval_before.json`: trace có hai lần gọi tool dù người dùng đã
yêu cầu rõ không kiểm tra status.

```text
step 1: get_status(provider="github")
step 2: get_status(provider="cloudflare")
step 3: final poem
```

Nguyên nhân có thể xảy ra: system prompt cũ nhấn mạnh “dùng tools trước khi
khẳng định về trạng thái”, nhưng chưa có quy tắc ưu tiên yêu cầu ngoài luồng;
model đã gọi tools theo thói quen dù câu hỏi không cần dữ liệu status.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: thêm vào `SYSTEM_PROMPT` trong `agent.py` quy tắc: câu hỏi
  không liên quan status/incident phải được trả lời trực tiếp, không gọi tools.
- Vì sao chọn thay đổi này: tránh side effect, giảm model calls và tôn trọng
  ranh giới “đừng kiểm tra” trong H4.
- Ngoài ra, agent đã có evidence gate: lưu URL từ incident thành công; nếu câu
  trả lời thiếu URL thì ghi `validation_missing_source` và yêu cầu sửa trong
  số bước còn lại. Tool thất bại không kích hoạt gate.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 92.68/100 | 97.09/100 |
| Điểm case đã chọn | 50/100 | 100/100 |
| Số model calls | 3 | 1 |
| Token, chi phí nếu có | 2.311 tokens cho H4; chi phí khoảng $0.0000847 | 451 tokens cho H4; chi phí Khoảng $0.0002875 |

- Lỗi đã giải quyết: H4 không còn gọi status tools, giữ nguyên bài thơ; số tool
  calls giảm từ 2 xuống 0 và điểm tăng từ 50 lên 100.
- Lỗi còn lại hoặc mới xuất hiện: các case chưa đạt 100 gồm E1, E2, E4, E5,
  A1, A3, A5, A6, H1, H5, H7 và H8; judge vẫn chỉ là ước lượng.

## 5. Giới hạn

Điểm judge là ước lượng; snapshot không phản ánh trạng thái live; mỗi phiên bản
chỉ chạy một lần nên chưa chứng minh độ ổn định. Cả hai lần đều chấm đủ 22/22
case, không có case `unscored`.

## 6. Kết quả bonus

Đã chạy đủ 10/10 case bonus:

| Case | Điểm |
|---|---:|
| B1 | 100 |
| B2 | 99 |
| B3 | 100 |
| B4 | 99 |
| B5 | 100 |
| B6 | 96 |
| B7 | 100 |
| B8 | 93 |
| B9 | 98 |
| B10 | 92 |

Điểm bonus tham khảo: **14.65/15** (97.7/100), không có case `unscored`.
Hai case cần xem thêm nếu muốn tối ưu là B8 và B10; đọc `grade.reason` trong
`eval_bonus.json` để biết thiếu sót cụ thể về scope hoặc tổng hợp nhiều provider.
