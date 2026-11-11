# Team

## Thông tin nhóm

* Tên nhóm: Nhóm 12
* Lớp: L3B
* Repo nhóm: K5-L3B-DAY03-Nhom12-ResearchAgentToolEval
* URL repo, nhánh nộp, commit chốt: https://github.com/vanphong205/K5-L3B-DAY03-Nhom12-ResearchAgentToolEval

## Thành viên

|STT|Họ tên|MSSV|Email|GitHub|Vai trò|
|-:|-|-|-|-|-|
|1|Hoàng Minh|2A202604103|minh03forwork@gmail.com|https://github.com/WorkOfMinh|Leader|
|2|Nguyễn Quốc Quang Anh|2A202604120|nguyenquocquanganh28@gmail.com|https://github.com/QuangAnh28|Thành viên|
|3|Phạm Trấn Quốc	|2A202604113|phamquoc896@gmail.com|https://github.com/Quoosc|Thành viên|
|4|Nguyễn Anh Tú	|2A202604088|comtrang378@gmail.com|https://github.com/comtrang378-ship-it|Thành viên|
|5|Trương Văn Phong	|2A202604091|phongtruongvan2005@gmail.com|https://github.com/vanphong205|Thành viên|
|6|Trần Anh Dũng|2A202604037|dungkidx01@gmail.com|https://github.com/tad-4963|Testing|

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV\_HoTen.md
```

Ví dụ: `report/team/20241234\_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## HOANGMINH - 2A202604103

### Phần việc phụ trách

* CP2 — Prompt \& Tool Optimization: thực hiện các vòng đánh giá và cải tiến Research Agent ở phiên bản v1, v2, v3 dựa trên baseline v0.
* Phụ trách chỉnh sửa starter\_v0/artifacts/system\_prompt.md và phần khai báo tool timeline trong starter\_v0/artifacts/tools.yaml.
* Kiểm tra kết quả bằng bộ eval data/eval\_base.json và kiểm thử hợp đồng tool.

### Đóng góp cụ thể

* v1 — System prompt: chỉnh sửa hướng dẫn về thông tin thiếu, lựa chọn tool, phạm vi yêu cầu và tham số. Kết quả: 18/20 case, accuracy 90%, so với baseline v0 14/20, 70%.
* v2 — Xác nhận trước khi gửi: bổ sung/làm rõ quy tắc hỏi xác nhận trước hành động gửi/đăng. Kết quả: 18/20 case, accuracy 90%. Case R12\_confirm\_before\_send vẫn chưa đạt do agent chọn clarify(response\_type="text") thay vì "yes\_no"; ghi nhận đây là hạn chế còn tồn tại.
* v3 — Tool declaration: cải thiện mô tả timeline về điều kiện sử dụng, định dạng screenname, mapping tài khoản (bao gồm karpathy) và giữ đúng limit qua nhiều lượt. Case M03\_correction\_handle chuyển từ FAIL sang PASS; kết quả 19/20 case, accuracy 95%.
* Chạy pytest tests/test\_tool\_contract.py: 55 passed; các bản eval v1–v3 đều đo đủ 20/20 case và có 0 provider errors.

### Bằng chứng

* v1: starter\_v0/runs/v1\_B\_base\_openai\_20261110T175822514538.json — case\_accuracy=0.90; artifact v1+pdc7ce7efdf3c+t41a2894f8ad1.
* v2: starter\_v0/runs/v2\_B\_base\_openai\_20261110T180448519846.json — case\_accuracy=0.90; artifact v2+pc800280bc6ca+t41a2894f8ad1.
* v3: starter\_v0/runs/v3\_B\_base\_openai\_20261110T181231483180.json — case\_accuracy=0.95; artifact v3+pc800280bc6ca+t0f6be0ee4fef.
* Các file thay đổi: starter\_v0/artifacts/system\_prompt.md, starter\_v0/artifacts/tools.yaml; kết quả thử nghiệm ghi trong starter\_v0/artifacts/version\_log.csv (sau khi cập nhật vào repo).
* Báo cáo cá nhân: report/team/2A202604103\_HoangMinh.md

## NGUYENQUOCQUANGANH - 2A202604120

## Phần việc phụ trách

- CP2 — Prompt Engineering & Tool Optimization: phân tích baseline v0 và thực hiện ba vòng cải tiến v1, v2, v3 cho Research Agent.
- Phụ trách chỉnh sửa `starter_v0/artifacts/system_prompt.md` và mô tả tool, điều kiện sử dụng cùng tham số trong `starter_v0/artifacts/tools.yaml`.
- Chạy base evaluation với `data/eval_base.json`, kiểm tra contract/unit tests và cập nhật `starter_v0/artifacts/version_log.csv` cùng báo cáo cá nhân.

## Đóng góp cụ thể

- **Baseline v0:** chạy trên artifacts đã commit ở `origin/main`. Kết quả 13/20 case, case accuracy 65%, tool routing accuracy 75%, argument accuracy 65%, multi-turn accuracy 100%. Các lỗi chính gồm chọn sai tham số lookup, gọi tool cho yêu cầu ngoài phạm vi và không hỏi lại khi thiếu thông tin.
- **v1 — System prompt:** bổ sung quy tắc chọn tool, hỏi lại khi thiếu thông tin, xử lý yêu cầu ngoài phạm vi và xác nhận trước khi gửi. Kết quả tăng lên 17/20 case, case accuracy 85%, tool routing accuracy 100%, argument accuracy 85%. Các lỗi còn lại là R10, R11 và R12 do thiếu `response_type`.
- **v2 — Tool declarations:** làm rõ điều kiện dùng/không dùng tool và mô tả tham số, trong đó có cách chọn `response_type` cho `clarify`. Kết quả tăng lên 19/20 case, case accuracy 95%, tool routing accuracy 100%, argument accuracy 95%. R10 và R11 được sửa; R12 vẫn chọn `text` thay vì `yes_no`.
- **v3 — Tinh chỉnh prompt:** dựa trên lỗi R12, yêu cầu xác nhận ý định gửi/đăng bằng yes/no trước, kể cả khi còn thiếu nội dung, đồng thời vẫn yêu cầu xác nhận chính xác nội dung trước khi gửi. Kết quả giữ ở 19/20 case, case accuracy 95%, multi-turn accuracy 100%. R12 tiếp tục chưa đạt và được ghi nhận là lỗi còn tồn tại.
- **Kiểm thử:** contract test đạt 65 passed. Toàn bộ test suite đạt 93 passed, 0 skipped. Provider preflight OpenAI thành công. Các run eval đều đo đủ 20/20 case và có 0 provider errors.

## Bằng chứng

- **v0:** `starter_v0/runs/v0_B_base_openai_20261110T191849190666.json` — case accuracy 0.65.
- **v1:** `starter_v0/runs/v1_B_base_openai_20261110T192004170716.json` — case accuracy 0.85.
- **v2:** `starter_v0/runs/v2_B_base_openai_20261110T192122099163.json` — case accuracy 0.95.
- **v3:** `starter_v0/runs/v3_B_base_openai_20261110T192240878196.json` — case accuracy 0.95; R12 vẫn chưa đạt.
- Các file đã chỉnh sửa gồm `starter_v0/artifacts/system_prompt.md`, `starter_v0/artifacts/tools.yaml` và `starter_v0/artifacts/version_log.csv`.
- Báo cáo cá nhân: `report/team/2A202604120_NguyenQuocQuangAnh.md`.

## Trương Văn Phong - 2A202604091

### Phần việc phụ trách

* Phát triển và đăng ký 2 tool mới (`github_search`, `hf_search`) đúng chuẩn contract của hệ thống (CP3).
* Viết unit test offline cho các tool mới và hoàn thiện toàn bộ các stub unit test trong `starter_v0/tests/test_tools_todo.py` (Test Challenge).

### Đóng góp cụ thể

* Cài đặt tool `github_search` (`starter_v0/tools/github_search/tool.py`, `TOOL.md`) gọi GitHub Search REST API (lọc `language`, sắp xếp `stars`/`forks`/`updated`/`best_match`, giới hạn `max_results` 1–10, trả error dict qua `err()` khi gặp lỗi), đồng bộ khai báo vào `starter_v0/tools/__init__.py` (`TOOL_FUNCTIONS`) và `starter_v0/artifacts/tools.yaml`.
* Cài đặt tool `hf_search` (`starter_v0/tools/hf_search/tool.py`, `TOOL.md`) gọi Hugging Face Hub API cho `models` và `datasets` (hỗ trợ `kind`, `sort`, `max_results`, validate input và trả error dict qua `err()`), đồng bộ khai báo vào `starter_v0/tools/__init__.py` và `starter_v0/artifacts/tools.yaml`.
* Viết 4 unit test cho `github_search`, 3 unit test cho `hf_search`, và hoàn thiện toàn bộ 14 stub unit test của các tool có sẵn (`clarify`, `format`, `send`, `lookup`, `fetch`, `timeline`, `social_search`, `policy`) trong `starter_v0/tests/test_tools_todo.py` dùng `fake_http` và `monkeypatch`.
* Kiểm tra toàn bộ suite `pytest -v` từ `starter_v0/`: đạt 93 passed, 0 failed, 0 skipped (65 contract tests, 7 example tests, 21 tests trong `test_tools_todo.py`).

### Bằng chứng

* Commits: `8ece7ca` (thêm `github_search` và unit tests), `44616cc` (thêm `hf_search` và hoàn thiện toàn bộ test stubs), `7147314` (`.gitignore`).
* Files: `starter_v0/tools/github_search/`, `starter_v0/tools/hf_search/`, `starter_v0/tools/__init__.py`, `starter_v0/artifacts/tools.yaml`, `starter_v0/tests/test_tools_todo.py`.
* Báo cáo cá nhân: `report/team/2A202604091_TruongVanPhong.md`

## PHẠM TRẤN QUỐC - 2A202604113

### Phần việc phụ trách

* C5/CP4 — Team Evaluation: thiết kế 10 eval case, gồm 5 single-turn và 5 multi-turn, chạy group eval và phân tích lỗi.

### Đóng góp cụ thể

  * Thiết kế `starter_v0/data/eval_group.json` gồm đúng 10 case (5 single-turn, 5 multi-turn), bao phủ routing `github_search`/`hf_search`, thiếu thông tin, policy, chuyển nguồn và cập nhật truy vấn qua nhiều lượt.
  * Chạy baseline Group Eval với OpenAI `gpt-4o-mini` (7/10, 70%), đọc expected/actual để phân tích lỗi G04, G05, G09; ghi nhận riêng GitHub API rate limit và không quy lỗi API thành lỗi provider.
  * Cải thiện `starter_v0/artifacts/system_prompt.md`: yêu cầu hỏi lại bằng `clarify(response_type="text")` khi thiếu chủ đề; đặt `policy_area="tool_usage"` rõ ràng cho vấn đề API/tool; giữ khái niệm mới khi sửa query multi-turn. Chạy hai lượt xác nhận, lưu JSON riêng; lượt cuối đạt 10/10, routing/argument/multi-turn đều 100%, provider error 0.
  * Cập nhật mục B3 trong `report/team/group_report.md` và báo cáo cá nhân gồm baseline, lần thử trung gian, run xác nhận, phân tích lỗi, hash/artifact version và giới hạn diễn giải. Không thay đổi dataset hoặc expected để tác động kết quả.
  * Thực hiện bonus B7: chạy hai baseline Ollama (`qwen2.5:7b`, `qwen2.5:3b`) đủ 20 case, thử v4-local/v5-local trên Qwen 7B và kiểm tra ngược trên OpenAI. Ghi metric, run và giới hạn Llama 3B trong `version_log.csv`, B7 báo cáo nhóm và báo cáo cá nhân.

### Bằng chứng

* `starter_v0/runs/v3_B_group_openai_20261110T192353230210.json`
  * `starter_v0/runs/group-quoc-clarify_B_group_openai_20261110T194544874142.json`
  * `starter_v0/runs/group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json`
  * `starter_v0/runs/group-quoc-main-sync_B_group_openai_20261110T195411233028.json` (run xác nhận cuối trên prompt đã đồng bộ với `origin/main`)
  * `starter_v0/artifacts/system_prompt.md` (quy tắc cải thiện cho Group Eval)
* `report/team/group_report.md` (mục B3)
* `report/team/2A202604113_PhamTranQuoc.md`
* B7: `starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json`, `starter_v0/runs/v3-qwen3b_B_base_ollama_20261110T223709953462.json`, `starter_v0/runs/v4-local_B_base_ollama_20261110T223020856578.json`, `starter_v0/runs/v5-local_B_base_ollama_20261110T223226373182.json`, `starter_v0/runs/cloud-v5-local_B_base_openai_20261110T223343542041.json`.

## TRẦN ANH DŨNG - 2A202604037

### Phần việc phụ trách

* Vai trò: Testing (CP1, CP5) theo phân công nhóm.
* Cài đặt môi trường phát triển, thiết lập cấu hình biến môi trường (`.env`) và chạy kiểm thử baseline `v0` ban đầu (CP1).
* Kiểm thử tương tác trực tiếp nhiều lượt (Live Chat) qua `chat.py` với Agent (CP5 / Step 5), thu thập biên bản transcript JSON thực tế.
* Kiểm thử tích hợp (Integration Testing): chạy và kiểm tra toàn bộ suite test của hệ thống bằng pytest.

### Đóng góp cụ thể

* Cài đặt môi trường & Baseline v0: Thiết lập môi trường Python 3.12, file `.env` với các khóa API (`OPENAI_API_KEY`, `TAVILY_API_KEY`, `RAPIDAPI_KEY`, `FIRECRAWL_API_KEY`). Chạy kiểm thử baseline ban đầu `v0` trên bộ 20 case chuẩn `data/eval_base.json` (đạt 13/20 case, 65%, 0 provider error) lưu tại `starter_v0/runs/v0_B_base_openai_20261110T191849190666.json`, phân loại các lỗi sai ban đầu của starter prompt để bàn giao dữ liệu cho nhóm.
* Live Chat tương tác thực tế (Step 5): Thực hiện phiên hội thoại trực tiếp 4 lượt qua `chat.py` với model OpenAI `gpt-4o-mini`, lưu trữ transcript tại `starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json`. Kiểm chứng thực tế các hành vi: routing tool mới `hf_search`, xử lý yêu cầu thiếu thông tin, cập nhật context đa lượt và cơ chế guardrail chặn gửi tin của tool `send`.
* Kiểm thử tích hợp: Thực hiện kiểm thử toàn bộ test suite của hệ thống bằng `pytest starter_v0/tests`, xác nhận 93 passed, 0 failed, 0 skipped (65 tool contract tests, 7 example tests, 21 unit tests).

### Bằng chứng

* Môi trường & Baseline v0: `starter_v0/.env`, `starter_v0/runs/v0_B_base_openai_20261110T191849190666.json`.
* Live chat transcript: `starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json`.
* Live chat evidence nhóm: `report/team/group_report.md` (mục B4).
* Kiểm thử pytest: `starter_v0/tests/` → 93 passed, 0 skipped, 0 failed.
* Báo cáo cá nhân: `report/team/2A202604037_TranAnhDung.md`.
