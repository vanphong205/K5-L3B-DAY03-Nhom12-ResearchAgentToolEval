# Báo cáo cá nhân — Nguyễn Quốc Quang Anh


## 1. Thông tin thành viên

- Họ tên: Nguyễn Quốc Quang Anh
- MSSV: 2A202604120
- Email: nguyenquocquanganh28@gmail.com
- Vai trò: Thành viên 2 — Prompt Engineering & Tool Optimization

## 2. Phần việc phụ trách

Tôi phụ trách phân tích baseline của research agent, cải thiện system prompt và tool declarations qua các vòng v1, v2, v3, chạy evaluation/test, ghi version log và tổng hợp bằng chứng cho báo cáo.

## 3. Đóng góp cụ thể

### Phân tích baseline

Tôi kiểm tra code nhóm trên `origin/main` tại commit `d52e697` và đối chiếu nhánh `origin/HoangMinh`. Run cũ trên nhánh HoangMinh không khớp hash `tools.yaml` đang commit trên nhánh đó hoặc trên main, nên tôi không dùng run này làm baseline chính. Tôi chạy lại v0 trên đúng artifacts đã commit ở `origin/main`.

Baseline v0 đạt 13/20 case (case accuracy 0.65), tool routing 0.75, argument accuracy 0.65 và multi-turn accuracy 1.00; đủ 20 case được đo, không có provider error. Các lỗi gồm: query/topic lookup sai ở R03/R13; gọi `send` cho câu hỏi ngoài phạm vi ở R08/R14; và không hỏi/không xác nhận đúng khi thiếu handle, URL hoặc ý định gửi ở R10–R12.

### Các vòng tối ưu

- **v1 — system prompt:** thay hướng dẫn đoán thông tin, tự gửi và chỉ dùng một tool bằng quy tắc routing, hỏi `clarify` khi thiếu dữ kiện, xử lý yêu cầu ngoài phạm vi và xác nhận trước khi gửi. Case accuracy tăng từ 0.65 lên 0.85. Các lỗi còn lại là R10, R11 và R12 do thiếu `response_type`.
- **v2 — tool descriptions:** làm rõ điều kiện dùng/không dùng tool và quy ước tham số, nhất là `lookup` và `clarify.response_type`. Case accuracy tăng từ 0.85 lên 0.95; R10/R11 được sửa. R12 vẫn chọn `text` thay vì `yes_no`.
- **v3 — prompt refinement:** dựa trên lỗi R12 còn lại, yêu cầu xác nhận ý định gửi/đăng bằng yes/no trước, kể cả khi còn thiếu nội dung; sau đó vẫn cần xác nhận nội dung và đích gửi chính xác trước khi gọi `send`. Kết quả giữ ở 0.95; R12 vẫn chưa đạt. Tôi ghi nhận đây là lỗi còn tồn tại, không tính là đã sửa.

## 4. Kết quả evaluation

Tất cả run dùng provider OpenAI, model `gpt-4o-mini`, suite `base`; mỗi run đo đủ 20/20 case và có 0 provider error.

| Version | Case accuracy | Tool routing | Argument | Multi-turn | Pass | Case chưa đạt |
|---|---:|---:|---:|---:|---:|---|
| v0 | 0.65 | 0.75 | 0.65 | 1.00 | 13/20 | R03, R08, R10, R11, R12, R13, R14 |
| v1 | 0.85 | 1.00 | 0.85 | 1.00 | 17/20 | R10, R11, R12 |
| v2 | 0.95 | 1.00 | 0.95 | 1.00 | 19/20 | R12 |
| v3 | 0.95 | 1.00 | 0.95 | 1.00 | 19/20 | R12 |

## 5. Bằng chứng và kiểm thử

- Prompt: `starter_v0/artifacts/system_prompt.md`
- Tool declarations: `starter_v0/artifacts/tools.yaml`
- Version log có giả thuyết, metric, hash và run path: `starter_v0/artifacts/version_log.csv`
- Eval JSON:
  - v0: `starter_v0/runs/v0_B_base_openai_20261110T191849190666.json`
  - v1: `starter_v0/runs/v1_B_base_openai_20261110T192004170716.json`
  - v2: `starter_v0/runs/v2_B_base_openai_20261110T192122099163.json`
  - v3: `starter_v0/runs/v3_B_base_openai_20261110T192240878196.json`
- Contract test: `python -m pytest tests/test_tool_contract.py -q` — 65 passed.
- Toàn bộ test suite: `python -m pytest -v` — 93 passed, 0 skipped.
- Provider preflight OpenAI thành công với `gpt-4o-mini`.
- Cấu hình khóa được nạp từ file `.env` ngoài repository qua `DAY03_ENV_FILE`; không sao chép hoặc ghi giá trị khóa vào project.

## 6. Khó khăn, giải pháp và phối hợp

Khó khăn chính là run/version log có sẵn trên nhánh HoangMinh không khớp hash tool declaration của source đã commit. Tôi kiểm tra hash, giữ run cũ làm tham khảo và tạo chuỗi v0–v3 mới từ đúng source của `origin/main` để so sánh có kiểm soát.

R12 vẫn lỗi sau ba vòng; cần phối hợp nhóm để quyết định có tiếp tục tối ưu hay báo cáo là case chưa khắc phục. Báo cáo cá nhân được đặt tên theo MSSV và họ tên trong `TEAM.md`; R12 được ghi nhận là lỗi còn tồn tại sau v3.
