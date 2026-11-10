# Phân tích Group Eval v0 — Phạm Trấn Quốc (2A202604113)

## 1. Phạm vi và bằng chứng

- Bộ case: [eval_group.json](../data/eval_group.json), gồm đúng 10 case độc lập: 5 single turn (`query`) và 5 multi turn (`turns`). Mỗi case có `phase=B`, `failure_type`, `expect` và `metadata.what_it_tests`.
- Tool mới có trong checkout tại thời điểm chạy: `github_search`. Các case kiểm tra việc chọn tool này, truyền tham số, hỏi lại khi thiếu chủ đề, và phân biệt với `papers` / `policy`. G10 yêu cầu gọi đồng thời `github_search` và `papers`.
- Phiên bản: prompt v0 tại `artifacts/system_prompt.md`, declarations hiện tại tại `artifacts/tools.yaml`; nhãn `v0-group-quoc`, provider OpenAI, model `gpt-4o-mini`.
- Run chính: [v0-group-quoc_B_group_openai_20261110T183156015784.json](../runs/v0-group-quoc_B_group_openai_20261110T183156015784.json). `artifact_version=v0-group-quoc+peb1c8179815b+t3a079b68cf3e`.
- Lệnh chạy từ `starter_v0/`: `python run_eval.py --provider openai --version v0-group-quoc --suite group --eval-cases data/eval_group.json`.
- Kiểm tra contract của schema/tool registry: `python -m pytest tests/test_tool_contract.py -q` → 60 passed.

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
| G02 | Single | GitHub repo TypeScript về MCP, sort updated | PASS |
| G03 | Single | Chọn GitHub repo thay vì web/paper | PASS |
| G04 | Single | Hỏi lại khi thiếu chủ đề GitHub | FAIL |
| G05 | Single | Chọn `policy_area=data_privacy` cho câu hỏi API key | FAIL |
| G06 | Multi | Giữ chủ đề cũ, thêm Python và limit 4 | PASS |
| G07 | Multi | Đổi sort sang updated và limit 2 | PASS |
| G08 | Multi | Chuyển từ tìm repo sang tìm arXiv paper | PASS |
| G09 | Multi | Đổi JavaScript sang Python, giữ ý scraping bất đồng bộ | FAIL |
| G10 | Multi | Tìm cả GitHub repo và arXiv paper cùng chủ đề | PASS |

## 3. Phân tích từng lỗi

| Case | Expected | Actual từ run JSON | Mismatch và hướng xử lý |
|---|---|---|---|
| G04 | `clarify(response_type=text)` | `github_search(query="machine learning", sort="stars", max_results=5)` | `missing_tool_call`: agent tự chọn một chủ đề không có trong yêu cầu. Prompt v0 yêu cầu đoán khi thiếu dữ kiện; nhóm phụ trách prompt cần đổi quy tắc sang hỏi lại khi thiếu chủ đề tìm kiếm. |
| G05 | `policy(policy_area="data_privacy")` | `policy(query="API key in prompt", policy_area="all")` | `wrong_arg_value`: chọn đúng tool nhưng chọn vùng policy quá rộng. Cần mô tả rõ trong `tools.yaml` rằng API key/secret thuộc `data_privacy` và kiểm tra lại ở version sau. |
| G09 | `github_search(query="asynchronous web scraping", language="python")` | `github_search(query="web scraping", language="python", sort="stars", max_results=5)` | `wrong_arg_value`: đã sửa đúng language nhưng bỏ tính chất “bất đồng bộ” ở lượt mới nhất. Cần rule giữ mọi điều kiện mới khi viết lại query. |

`failure_counts` của run là 1 `missing_info` và 2 `wrong_arg_value`. `observed_mismatch_counts` là 1 `missing_tool_call` và 2 `wrong_arg_value`. G05/G09 là lỗi args, dù tên tool đều đúng.

## 4. Giới hạn và lưu ý khi dùng số liệu

- Run chính có **3 tool error** từ GitHub Search API: G07, G09 và G10 trả HTTP 403 do rate limit. Đây là lỗi khi thực thi tool, không phải `provider_error`; harness vẫn đo được tool call và args của model. PASS ở G07/G10 chỉ chứng minh lựa chọn tool/args đúng theo bộ chấm, chưa chứng minh agent lấy được nội dung repo ở các case đó.
- G09 còn có lỗi argument độc lập với 403: query bỏ mất “bất đồng bộ”. Vì vậy không quy lỗi G09 cho rate limit.
- Chỉ một tool do nhóm tự thêm đang có trong checkout này (`github_search`). Khi tool thứ hai được tích hợp, bộ eval 10 case cần được rà lại để có ít nhất một case kiểm tra tool đó; giữ tổng 5 single + 5 multi.
- Run này dùng prompt v0, chưa phải số đo v3. Sau khi nhóm tối ưu prompt và tool descriptions, chạy cùng `eval_group.json` với `--version v3` rồi đối chiếu từng case và metric. Không thay expected theo kết quả model.
- Hai run trước `...182840023108.json` và `...182931480574.json` là lượt thử khi hoàn thiện dataset. Bảng kết quả và kết luận trong báo cáo này dùng duy nhất run chính `...183156015784.json`.

## 5. Điều chỉnh thiết kế case trước run chính

G03 ban đầu so khớp nguyên văn query, dễ báo sai khi hai truy vấn GitHub cùng nghĩa. Expected đã đổi thành chỉ kiểm tra `github_search`, đúng mục tiêu định tuyến của case. G10 ban đầu kiểm tra một ranh giới không có action tool tương ứng trong agent; case cuối yêu cầu kết hợp GitHub và arXiv để kiểm tra multi-tool routing. Cả hai thay đổi được thực hiện trước run chính và được ghi lại để người đọc phân biệt run thử với số liệu bàn giao.
