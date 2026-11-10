# Báo cáo cá nhân — Phạm Trấn Quốc (2A202604113)

## 1. Thông tin thành viên

- Họ tên: Phạm Trấn Quốc
- MSSV: 2A202604113
- Email: chưa cung cấp
- Vai trò: Evaluation

## 2. Phần việc phụ trách

Thiết kế bộ Group Eval gồm 10 case mới (5 single turn, 5 multi turn), chạy eval với model thật, đọc run JSON để xác định lỗi chọn tool và truyền arguments, rồi đưa kết quả vào mục B3 của báo cáo nhóm. Bộ case được viết trong [eval_group.json](../../starter_v0/data/eval_group.json) và tập trung vào tool mới `github_search` cùng các tình huống dễ nhầm với `papers` và `policy`.

## 3. Đóng góp cụ thể

1. Rà `README.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`, mẫu `group_report.md`, danh sách tool và schema của `run_eval.py` để xác định đúng yêu cầu CP4.
2. Viết 10 case `G01`–`G10`, mỗi case có `phase=B`, `failure_type`, `expect`, `metadata.what_it_tests`. Năm case đầu dùng `query`; năm case sau dùng `turns`. Có case chọn `github_search`, sửa arguments qua nhiều lượt, đổi từ GitHub sang arXiv, và gọi kết hợp hai tool.
3. Điều chỉnh G03 để chỉ đo routing, vì phép so chuỗi query nguyên văn có thể báo sai với truy vấn cùng nghĩa. Điều chỉnh G10 thành tình huống cần cả GitHub và arXiv. Các thay đổi này được thực hiện trước run chính và nêu rõ trong phân tích.
4. Chạy Group Eval bằng OpenAI `gpt-4o-mini`, nhãn `v0-group-quoc`. Run chính đo đủ 10/10 case, 0 provider error, đạt 7/10 case; routing 90%, arguments 70%, multi turn 80%.
5. Đọc `actual_tool_calls`, `failures`, `observed_mismatch` và `tool_results` để phân tích ba case FAIL (G04, G05, G09). Phát hiện thêm GitHub Search API trả 403 rate limit ở G07, G09, G10, nên tách lỗi thực thi tool khỏi lỗi chọn tool/arguments của model.
6. Ghi [phân tích chi tiết](../../starter_v0/analysis/group_eval_v0.md) và điền bảng B3 trong [báo cáo nhóm](group_report.md).

## 4. Bằng chứng

| Hạng mục | File / kết quả |
|---|---|
| Bộ Group Eval | [eval_group.json](../../starter_v0/data/eval_group.json) |
| Run chính | [v0-group-quoc_B_group_openai_20261110T183156015784.json](../../starter_v0/runs/v0-group-quoc_B_group_openai_20261110T183156015784.json) |
| Phân tích lỗi và giới hạn | [group_eval_v0.md](../../starter_v0/analysis/group_eval_v0.md) |
| Bảng B3 của nhóm | [group_report.md](group_report.md) |
| Kiểm tra contract | `python -m pytest tests/test_tool_contract.py -q` → 60 passed |

**AI/công cụ đã dùng:** Codex hỗ trợ rà yêu cầu, soạn case và tổng hợp báo cáo; Python `run_eval.py` gọi provider OpenAI và ghi run JSON thật. Tôi kiểm tra lại số lượng 5 single + 5 multi, `summary` và từng `results[*].result` trong run, đối chiếu expected/actual với file case. Không dùng AI để tạo số liệu hoặc run giả.

**Lệnh tái hiện:** chạy từ `starter_v0/` sau khi cấu hình `.env` và cài `requirements.txt`:

```powershell
python run_eval.py --provider openai --version v0-group-quoc --suite group --eval-cases data/eval_group.json
```

## 5. Tự đánh giá

Phần Evaluation đạt yêu cầu cấu trúc CP4: đúng 10 case, đủ 5 single + 5 multi, có case cho tool mới, run đủ 10/10 không gặp provider error, bảng B3 có dẫn chứng. Ba case FAIL là kết quả thực tế để nhóm cải thiện prompt/tool declaration. Run hiện dùng prompt v0; cần chạy lại cùng bộ case trên v3 khi nhóm hoàn tất tối ưu. GitHub rate limit làm một số tool call không lấy được dữ liệu repo dù harness vẫn đo được routing/arguments. Trong checkout hiện tại mới có một tool bổ sung `github_search`; nếu tool thứ hai được merge, cần thay một case phù hợp để kiểm tra tool đó mà vẫn giữ tổng 10 case.
