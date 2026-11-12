# Ghi chú phân tích · Day 04

Họ và tên: Le Doan Duong
MSSV: 2A202603372

## 1. Case được chọn

- ID case: E2
- Câu trả lời của agent: Không có câu trả lời vì provider Gemini trả lỗi HTTP 429 ở lần gọi model đầu tiên.
- Điểm ở lần chạy 1: `null` (`unscored`)

`E2` được chọn làm case đại diện vì đây là case kiểm tra incident Cloudflare và phạm vi
FedRAMP, nhưng không thể đánh giá nội dung agent khi provider chưa trả về response.
Không có case nào có điểm thấp theo nghĩa thông thường: cả 22/22 case đều chưa được chấm.

## 2. Bằng chứng trong trace

Trace của E2:

```text
{
  "event": "error",
  "step": 1,
  "error": "RuntimeError: Provider trả HTTP 429; kiểm tra model, key và quota."
}
```

Nguyên nhân có thể xảy ra là API Gemini bị rate limit hoặc key/model đã hết quota.
Lỗi xảy ra trước khi agent gọi tool, nên đây là lỗi hạ tầng/cấu hình chứ chưa phải
bằng chứng agent trả lời sai. `usage` cũng rỗng và `tool_calls` bằng 0.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Hoàn thiện vòng lặp `run_agent` trong `agent.py`: giới hạn số
  model calls, xử lý nhiều tool calls, JSON arguments lỗi, lỗi tool/provider,
  trace, usage và kiểm tra URL nguồn incident.
- Vì sao chọn thay đổi này: Đây là contract bắt buộc của bài lab và đã được kiểm
  chứng bằng bộ test offline. Tuy nhiên, thay đổi này chưa thể đo bằng
  `eval_before.json` vì toàn bộ lần chạy bị Gemini HTTP 429.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | `null` (0/22 scored) | Chưa chạy |
| Điểm case đã chọn | `null` (unscored) | Chưa chạy |
| Số model calls | 22 lần gọi đầu tiên, đều lỗi HTTP 429 | Chưa chạy |
| Token, chi phí nếu có | Không có usage trong kết quả | Chưa chạy |

- Lỗi đã giải quyết: Các test offline của agent và bonus đều pass; lỗi HTTP 429
  chưa được giải quyết trong eval.
- Lỗi còn lại hoặc mới xuất hiện: Cần dùng key Gemini hợp lệ còn quota hoặc
  chờ hết rate limit, sau đó chạy lại eval trước khi so sánh chất lượng agent.

## 5. Giới hạn

- Vì cả 22 case đều `unscored`, chưa có điểm judge để kết luận agent tốt hay
  kém.
- Snapshot là dữ liệu cố định của bài lab, không đại diện cho trạng thái dịch vụ
  hiện tại.
- Eval gọi model thật và có thể bị ảnh hưởng bởi quota, rate limit, model ID và
  cấu hình API.
- Mỗi phiên bản chỉ chạy một lần nên ngay cả khi có điểm, kết quả vẫn chưa chứng
  minh được độ ổn định.
- Chưa có `eval_after.json`, nên chưa thể kết luận thay đổi đã cải thiện điểm.
