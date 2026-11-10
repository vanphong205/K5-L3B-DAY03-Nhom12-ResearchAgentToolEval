# Phân tích Group Eval v0 — Phạm Trấn Quốc (2A202604113)

## 1. Phạm vi và bằng chứng

- Bộ case: [eval_group.json](../data/eval_group.json), gồm đúng 10 case độc lập: 5 single turn (`query`) và 5 multi turn (`turns`). Mỗi case có `phase=B`, `failure_type`, `expect` và `metadata.what_it_tests`.
- Hai tool mới đã có trong checkout: `github_search` và `hf_search`. Các case kiểm tra chọn tool, truyền tham số, hỏi lại khi thiếu chủ đề, và phân biệt với `papers` / `policy`. G10 yêu cầu gọi đồng thời hai tool mới.
- Phiên bản: prompt v0 tại `artifacts/system_prompt.md`, declarations có cả hai tool tại `artifacts/tools.yaml`; nhãn `v0-group-quoc-two-tools`, provider OpenAI, model `gpt-4o-mini`.
- Run chính: [v0-group-quoc-two-tools_B_group_openai_20261110T183742208256.json](../runs/v0-group-quoc-two-tools_B_group_openai_20261110T183742208256.json). `artifact_version=v0-group-quoc-two-tools+peb1c8179815b+tee32b4055306`.
- Lệnh chạy từ `starter_v0/`: `python run_eval.py --provider openai --version v0-group-quoc-two-tools --suite group --eval-cases data/eval_group.json`.
- Kiểm tra contract của schema/tool registry sau khi tích hợp `hf_search`: `python -m pytest tests/test_tool_contract.py -q` → 65 passed.

Harness chấm tên tool và một tập con arguments trong `expect.tool_calls`. So sánh chuỗi dùng `strip()` và chuyển chữ thường, nhưng không hiểu hai cách diễn đạt đồng nghĩa. Vì vậy expected arguments chỉ chứa giá trị cần đo; riêng G03 chỉ chấm định tuyến, không ép một chuỗi query duy nhất.

## 2. Kết quả run chính

| Chỉ số trong `summary` | Kết quả |
|---|---:|
| `total_cases` / `measured_cases` | 10 / 10 |
| `provider_error_cases` | 0 |
| `passed_cases` / `case_accuracy` | 7 / 70% |
| `tool_routing_accuracy` | 90% |
| `argument_accuracy` | 70% |
| `multiturn_accuracy` | 80% (4/5) |

| Case | Lượt | Mục tiêu đo | Kết quả |
|---|---|---|---|
| G01 | Single | GitHub repo Python về RAG, 3 kết quả theo stars | PASS |
| G02 | Single | Dataset tiếng Việt trên Hugging Face, kind=datasets, sort=lastModified, limit=4 | PASS |
| G03 | Single | Chọn GitHub repo thay vì web/paper | PASS |
| G04 | Single | Hỏi lại khi thiếu chủ đề GitHub | FAIL |
| G05 | Single | Chọn `policy_area=data_privacy` cho câu hỏi API key | FAIL |
| G06 | Multi | Giữ chủ đề cũ, thêm Python và limit 4 | PASS |
| G07 | Multi | Đổi sort sang updated và limit 2 | PASS |
| G08 | Multi | Chuyển từ tìm repo sang tìm arXiv paper | PASS |
| G09 | Multi | Đổi JavaScript sang Python, giữ ý scraping bất đồng bộ | FAIL |
| G10 | Multi | Tìm cả GitHub repo và model Hugging Face | PASS |

## 3. Phân tích từng lỗi

| Case | Expected | Actual từ run JSON | Mismatch và hướng xử lý |
|---|---|---|---|
| G04 | `clarify(response_type=text)` | `github_search(query="AI", sort="stars", max_results=5)` | `missing_tool_call`: agent tự chọn một chủ đề không có trong yêu cầu. Prompt v0 yêu cầu đoán khi thiếu dữ kiện; nhóm phụ trách prompt cần đổi quy tắc sang hỏi lại khi thiếu chủ đề tìm kiếm. |
| G05 | `policy(policy_area="data_privacy")` | `policy(query="API key in prompt")` | `wrong_arg_value`: chọn đúng tool nhưng không truyền `policy_area`. Cần mô tả rõ trong `tools.yaml` rằng API key/secret thuộc `data_privacy` và kiểm tra lại ở version sau. |
| G09 | `github_search(query="asynchronous web scraping", language="python")` | `github_search(query="web scraping", language="python", sort="stars", max_results=5)` | `wrong_arg_value`: đã sửa đúng language nhưng bỏ tính chất “bất đồng bộ” ở lượt mới nhất. Cần rule giữ mọi điều kiện mới khi viết lại query. |

`failure_counts` của run là 1 `missing_info` và 2 `wrong_arg_value`. `observed_mismatch_counts` là 1 `missing_tool_call` và 2 `wrong_arg_value`. G05/G09 là lỗi args, dù tên tool đều đúng.

## 4. Giới hạn và lưu ý khi dùng số liệu

- Run chính có **1 tool error** từ GitHub Search API: G10 trả HTTP 403 do rate limit. Đây là lỗi khi thực thi tool, không phải `provider_error`; harness vẫn đo được tool call và args của model. PASS ở G10 chỉ chứng minh lựa chọn tool/args đúng theo bộ chấm, chưa chứng minh agent lấy được nội dung repo ở case đó.
- G02 gọi đúng `hf_search` nhưng API trả danh sách rỗng; điểm PASS chứng minh routing/arguments, không chứng minh tìm được dataset phù hợp.
- Cả hai tool mới đã được tích hợp trước run chính. G02 đo riêng `hf_search`; G10 đo phối hợp `github_search` và `hf_search`.
- Run này dùng prompt v0, chưa phải số đo v3. Sau khi nhóm tối ưu prompt và tool descriptions, chạy cùng `eval_group.json` với `--version v3` rồi đối chiếu từng case và metric. Không thay expected theo kết quả model.
- Run baseline `v0_B_base_openai_20261110T173412960047.json` chấm 20 case thuộc suite `base` với `prompt_hash` bắt đầu `f0c107a9d7` và `tools_hash` bắt đầu `41a2894f8a`. Run group này chấm 10 case thuộc suite `group` với hash `eb1c817981` / `ee32b40553`. Khác suite và khác artifacts, nên không lấy 70% group để kết luận agent giữ nguyên hoặc cải thiện so với baseline.
- Run group trước `...183156015784.json` được tạo trước khi tích hợp `hf_search` và đã được thay thế. Bảng kết quả và kết luận trong báo cáo này dùng duy nhất run chính `...183742208256.json`.

## 5. Điều chỉnh thiết kế case trước run chính

G03 ban đầu so khớp nguyên văn query, dễ báo sai khi hai truy vấn GitHub cùng nghĩa. Expected đã đổi thành chỉ kiểm tra `github_search`, đúng mục tiêu định tuyến của case. Sau khi `hf_search` được tích hợp, G02 chuyển sang tìm dataset Hugging Face và G10 chuyển sang tình huống cần đồng thời GitHub repo và Hugging Face model. Các thay đổi được thực hiện trước run chính.
