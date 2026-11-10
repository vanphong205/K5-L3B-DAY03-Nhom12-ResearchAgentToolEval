# Team

## Thông tin nhóm

* Tên nhóm:
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
|6|Trần Anh Dũng|2A202604037|dungkidx01@gmail.com|https://github.com/tad-4963|Thành viên|

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

### Bằng chứng

* `starter_v0/runs/v3_B_group_openai_20261110T192353230210.json`
  * `starter_v0/runs/group-quoc-clarify_B_group_openai_20261110T194544874142.json`
  * `starter_v0/runs/group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json`
  * `starter_v0/runs/group-quoc-main-sync_B_group_openai_20261110T195411233028.json` (run xác nhận cuối trên prompt đã đồng bộ với `origin/main`)
  * `starter_v0/artifacts/system_prompt.md` (quy tắc cải thiện cho Group Eval)
* `report/team/group_report.md` (mục B3)
* `report/team/2A202604113_PhamTranQuoc.md`
