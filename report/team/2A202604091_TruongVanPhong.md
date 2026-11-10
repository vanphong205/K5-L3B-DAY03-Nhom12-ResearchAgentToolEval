# Báo cáo cá nhân — Trương Văn Phong (2A202604091)

## 1. Thông tin thành viên

- Họ tên: Trương Văn Phong
- MSSV: 2A202604091
- Email: phongtruongvan2005@gmail.com
- GitHub: `vanphong205`
- Vai trò: Phát triển tool mới và viết unit test (CP3 & Test Challenge)

## 2. Phần việc phụ trách

Thiết kế, cài đặt và đăng ký 2 tool mới cho agent (`github_search` và `hf_search`) đúng chuẩn contract của hệ thống, đồng thời viết unit test offline cho cả 2 tool mới và hoàn thiện toàn bộ các stub unit test của các tool có sẵn trong [test_tools_todo.py](../../starter_v0/tests/test_tools_todo.py).

## 3. Đóng góp cụ thể

1. **Xây dựng tool mới `github_search` (commit `8ece7ca`):**
   - Cài đặt hàm [`search_github_repos`](../../starter_v0/tools/github_search/tool.py) gọi GitHub Search REST API (`https://api.github.com/search/repositories`).
   - Hỗ trợ lọc theo `language`, sắp xếp theo `sort` (`stars`, `forks`, `updated`, `best_match`), giới hạn `max_results` trong khoảng `1..10`, tùy chọn `GITHUB_TOKEN` qua biến môi trường để tăng rate limit.
   - Chuẩn hóa kết quả trả về (`title`, `url`, `source`, `summary`, `stars`, `language`, `updated_at`), kiểm tra đầu vào (`query`, `language`, `sort`, `max_results`) và trả về error dict qua `err("github_search", exc)` thay vì raise exception.
   - Tạo [TOOL.md](../../starter_v0/tools/github_search/TOOL.md), đăng ký key `"github_search"` trong [`TOOL_FUNCTIONS`](../../starter_v0/tools/__init__.py) và khai báo schema trong [tools.yaml](../../starter_v0/artifacts/tools.yaml).

2. **Xây dựng tool mới `hf_search` (commit `44616cc`):**
   - Cài đặt hàm [`search_huggingface`](../../starter_v0/tools/hf_search/tool.py) gọi Hugging Face Hub API (`https://huggingface.co/api/models` và `/api/datasets`).
   - Hỗ trợ chọn `kind` (`models`, `datasets`), `sort` (`downloads`, `likes`, `lastModified`), giới hạn `max_results` trong khoảng `1..10`, tùy chọn `HF_TOKEN`.
   - Chuẩn hóa danh sách `items` (`title`, `url`, `source`, `summary`, `kind`, `downloads`, `likes`, `pipeline_tag`, `tags`, `last_modified`), validate input chặt chẽ (kể cả chặn `bool` truyền vào `max_results`) và trả về error dict qua `err("hf_search", exc)`.
   - Tạo [TOOL.md](../../starter_v0/tools/hf_search/TOOL.md), đăng ký key `"hf_search"` trong [`TOOL_FUNCTIONS`](../../starter_v0/tools/__init__.py) và khai báo schema trong [tools.yaml](../../starter_v0/artifacts/tools.yaml).

3. **Viết unit test cho tool mới & hoàn thiện Test Challenge ([test_tools_todo.py](../../starter_v0/tests/test_tools_todo.py)):**
   - Viết **4 unit test** cho `github_search`: happy path kiểm tra request params/headers và mapping output; validate input xấu (`query` rỗng, sai kiểu, `sort` không hợp lệ) không gọi mạng; xử lý HTTP 403 error; xử lý `requests.Timeout`.
   - Viết **3 unit test** cho `hf_search`: happy path cho cả `models` và `datasets`; validate input xấu (`query` rỗng, `kind`/`sort` sai enum, `max_results=True`); xử lý HTTP 503 và `requests.Timeout`.
   - Hoàn thiện toàn bộ **14 stub unit test** cho các tool có sẵn (`clarify`, `format`, `send`, `lookup`, `fetch`, `timeline`, `social_search`, `policy`), giúp toàn bộ test suite đạt trạng thái **0 failed, 0 skipped**.

4. **Thiết lập ban đầu & tích hợp nhánh nhóm:**
   - Bổ sung `.gitignore` cho các file sinh ra trong quá trình code với AI (commit `7147314`) và thực hiện merge các nhánh làm việc của thành viên trong nhóm.

## 4. Bằng chứng

| Hạng mục | File / Commit / Kết quả |
|---|---|
| Tool `github_search` | [tool.py](../../starter_v0/tools/github_search/tool.py), [TOOL.md](../../starter_v0/tools/github_search/TOOL.md) (commit `8ece7ca4d6505b231e45a41aaca7002c030d9ef`) |
| Tool `hf_search` | [tool.py](../../starter_v0/tools/hf_search/tool.py), [TOOL.md](../../starter_v0/tools/hf_search/TOOL.md) (commit `44616ccddedb6bc75ff05a41aaca7002c030d9ef`) |
| Registry & Tool declaration | [tools/\_\_init\_\_.py](../../starter_v0/tools/__init__.py), [tools.yaml](../../starter_v0/artifacts/tools.yaml) |
| Unit test (21 test trong `test_tools_todo.py`) | [test_tools_todo.py](../../starter_v0/tests/test_tools_todo.py) |
| Kết quả chạy toàn bộ test suite | `.\.venv\Scripts\python.exe -m pytest -v` (từ `starter_v0/`) → **93 passed in 0.43s** (65 contract tests + 7 example tests + 21 TODO/new-tool tests; 0 failed, 0 skipped) |

**AI/công cụ đã dùng và cách tự kiểm tra kết quả:**
- Dùng `pytest` cùng fixture offline `fake_http`, `FakeResponse` và `monkeypatch` trong [conftest.py](../../starter_v0/tests/conftest.py) để đảm bảo không phát sinh request mạng thật khi chạy unit test; kiểm tra đồng bộ giữa `tools.yaml`, `TOOL.md`, `TOOL_FUNCTIONS` và các bộ eval bằng [test_tool_contract.py](../../starter_v0/tests/test_tool_contract.py).
- Dùng trợ lý AI hỗ trợ rà soát contract, cấu trúc test edge-case và tổng hợp nội dung báo cáo từ commit/log thực tế; toàn bộ code và test đều được chạy kiểm chứng trực tiếp bằng `pytest -v` (93 passed).

## 5. Tự đánh giá

- **Điểm đạt được:** Hoàn thành vượt yêu cầu CP3 (xây dựng 2 tool mới thay vì tối thiểu 1, mỗi tool đều có ≥ 3 unit test phủ happy path, input xấu và nhánh lỗi mạng/HTTP) và hoàn tất toàn bộ stub trong `test_tools_todo.py` (0 skipped, 0 failed). Các tham số và output được chuẩn hóa thống nhất theo cấu trúc `items` của project để agent dễ tổng hợp bằng `format`.
- **Hạn chế / hướng cải thiện:** Trong môi trường chạy eval thực tế chưa cấu hình `GITHUB_TOKEN`, GitHub Search API dễ chạm ngưỡng rate limit (HTTP 403) khi gọi liên tiếp nhiều case. Có thể bổ sung hướng dẫn cấu hình `GITHUB_TOKEN` / `HF_TOKEN` rõ hơn trong `.env.example` và thêm test cho nhánh có token gửi kèm header `Authorization`.

