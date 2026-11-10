# Báo cáo cá nhân — Phạm Trấn Quốc (2A202604113)

## 1. Thông tin thành viên

- Họ tên: Phạm Trấn Quốc
- MSSV: 2A202604113
- Email: chưa cung cấp
- Vai trò: Evaluation

## 2. Phần việc phụ trách

Tôi phụ trách toàn bộ C5/CP4 — Team Evaluation. Công việc gồm xác định yêu cầu rubric, thiết kế bộ Group Eval gồm đúng 10 case mới, kiểm tra schema và tính đa dạng của case, tích hợp các case với hai tool mới `github_search` và `hf_search`, chạy group eval bằng provider thật, phân tích expected/actual trong run JSON, cập nhật bảng B3 và ghi nhận các giới hạn của kết quả.

Phạm vi cá nhân của tôi tập trung vào hành vi của model (routing, arguments, boundary và multi-turn context), không thay thế unit test của tool. Unit test kiểm tra implementation Python; Group Eval kiểm tra model có chọn đúng tool và truyền đúng arguments hay không.

## 3. Đóng góp cụ thể

1. Rà `README.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`, mẫu `group_report.md`, danh sách tool và schema của `run_eval.py` để xác định đúng yêu cầu CP4.
2. Thiết kế 5 case single-turn và 5 case multi-turn. Năm case đầu dùng `query`; năm case sau dùng `turns`, để kiểm tra cả routing trong một lượt và khả năng giữ/cập nhật context qua nhiều lượt.
3. Viết 10 case `G01`–`G10`; mỗi case có `phase="B"`, `suite="group"`, `failure_type`, `expect` và `metadata.what_it_tests`. Các case bao phủ GitHub repository, Hugging Face dataset/model, policy nội bộ, clarify khi thiếu thông tin, chuyển từ repository sang paper, chỉnh sửa query/language/sort/limit và gọi đồng thời hai tool.
4. Kiểm tra tính độc lập của bộ case: G01–G03 và G06–G10 tập trung vào routing/arguments/multi-turn riêng, G04 kiểm tra missing information, G05 kiểm tra policy area, G08 kiểm tra chuyển tool, G10 kiểm tra multi-tool call. G03 chỉ yêu cầu routing để không phạt các cách diễn đạt query tương đương.
5. Sau khi `hf_search` được tích hợp, cập nhật G02 và G10 để bộ eval có bằng chứng route tới cả hai tool mới. Điều chỉnh G05 sang tình huống rate limit để phân biệt `policy(tool_usage)` với web search và tránh trùng bộ extension.
6. Chạy Group Eval bằng OpenAI `gpt-4o-mini` với nhãn cuối `v3`, đủ 10/10 case được đo và 0 provider error. Kết quả cuối: 7/10 case pass, `case_accuracy=0.70`, `tool_routing_accuracy=0.90`, `argument_accuracy=0.70`, `multiturn_accuracy=0.80`.
7. Đọc `actual_tool_calls`, `failures`, `observed_mismatch` và `tool_results` trong run JSON để phân tích ba case FAIL: G04 model đoán query thay vì clarify; G05 chọn đúng policy nhưng dùng `policy_area=all`; G09 bỏ mất điều kiện “asynchronous” khi cập nhật query.
8. Đối chiếu lỗi model với lỗi dịch vụ: GitHub Search API trả HTTP 403 do rate limit ở các lần gọi GitHub. Vì vậy tôi ghi rõ rằng harness vẫn đo routing/arguments, nhưng dữ liệu repository thực tế không được xác nhận.
9. Điền và cập nhật mục B3 trong báo cáo nhóm, gồm danh sách 10 case, expected behavior, PASS/FAIL, phân tích ba lỗi, metric, run hợp lệ và giới hạn của API.

## 4. Bằng chứng

| Hạng mục | File / kết quả |
|---|---|
| Bộ Group Eval | [eval_group.json](../../starter_v0/data/eval_group.json) |
| Run chính | [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json) |
| Bảng B3, phân tích lỗi và giới hạn | [group_report.md](group_report.md) |
| Registry và tool mới | [tools/__init__.py](../../starter_v0/tools/__init__.py), [tools/github_search/](../../starter_v0/tools/github_search/), [tools/hf_search/](../../starter_v0/tools/hf_search/) |
| Khai báo tool | [tools.yaml](../../starter_v0/artifacts/tools.yaml) |
| Kiểm tra contract | `python -m pytest tests/test_tool_contract.py -q` → 65 passed sau khi tích hợp `hf_search` |

**AI/công cụ đã dùng:** Codex hỗ trợ rà yêu cầu, soạn case và tổng hợp báo cáo; Python `run_eval.py` gọi provider OpenAI và ghi run JSON thật. Tôi kiểm tra lại số lượng 5 single + 5 multi, `summary` và từng `results[*].result` trong run, đối chiếu expected/actual với file case. Không dùng AI để tạo số liệu hoặc run giả.

**Lệnh tái hiện:** chạy từ `starter_v0/` sau khi cấu hình `.env` và cài `requirements.txt`:

```powershell
python run_eval.py --provider openai --version v3 --suite group --eval-cases data/eval_group.json
```

## 5. Tự đánh giá

Phần Evaluation đạt đầy đủ yêu cầu cấu trúc CP4: đúng 10 case tự thiết kế, đủ 5 single-turn và 5 multi-turn, mọi case có `expect` và `metadata.what_it_tests`, có case route tới cả `github_search` và `hf_search`, có run `--suite group` hợp lệ và bảng B3 có dẫn chứng.

Run cuối có `total_cases=10`, `measured_cases=10`, `provider_error_cases=0`, nên đủ điều kiện dùng làm evidence. Kết quả 7/10 case pass (70%) không phải yêu cầu pass rate tối thiểu của C5; ba case fail được giữ lại vì chúng là bằng chứng thực tế để nhóm cải thiện prompt/tool declaration. Routing đạt 90%, nhưng argument accuracy đạt 70% và multi-turn accuracy đạt 80%, cho thấy bộ case đã phát hiện được các lỗi mà chỉ nhìn câu trả lời cuối sẽ khó nhận ra.

Giới hạn cần ghi rõ: GitHub API bị HTTP 403 do rate limit, nên tool result của các case GitHub không chứng minh dữ liệu repository hữu ích. Tuy nhiên run vẫn có thể đánh giá hành vi model vì harness đối chiếu tên tool và subset arguments trước khi phụ thuộc vào nội dung API. G02 route đúng `hf_search` nhưng kết quả dataset rỗng; đây là giới hạn dữ liệu/runtime, không tự động được quy thành lỗi routing.

Qua nhiệm vụ này, tôi học được cách biến yêu cầu tự nhiên thành testable expectation, thiết kế case để phân biệt lỗi routing với lỗi argument, và dùng run JSON làm bằng chứng thay vì đánh giá bằng cảm nhận. Nếu có vòng tiếp theo, tôi sẽ sửa prompt cho missing query, mô tả rõ `policy_area=tool_usage` trong `tools.yaml`, và bổ sung quy tắc giữ toàn bộ điều kiện mới khi rewrite query multi-turn; sau đó chạy lại group eval để kiểm tra đúng các giả thuyết này.
