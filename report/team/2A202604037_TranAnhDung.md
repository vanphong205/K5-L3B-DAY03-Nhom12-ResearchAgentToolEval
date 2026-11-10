# Báo cáo cá nhân — Trần Anh Dũng (2A202604037)

## 1. Thông tin thành viên

- **Họ tên:** Trần Anh Dũng
- **MSSV:** 2A202604037
- **Email:** dungkidx01@gmail.com
- **GitHub:** `tad-4963`
- **Vai trò:** **Testing** (CP1, CP5)

---

## 2. Phần việc phụ trách

Theo phân công trong nhóm, tôi phụ trách mảng **Testing & Setup**:

1. **Cài đặt môi trường & file `.env` (CP1):** Khởi tạo môi trường ảo Python, cài đặt dependencies, cấu hình file `.env` với các API keys cho OpenAI và các tool nghiên cứu, kiểm tra kết nối preflight.
2. **Chạy kiểm thử Baseline `v0` (CP1):** Thực hiện chạy đánh giá baseline `v0` trên bộ 20 case chuẩn bằng OpenAI `gpt-4o-mini`, thống kê kết quả ban đầu và phân loại các lỗi của starter code.
3. **Kiểm thử Live Chat tương tác trực tiếp (Step 5 / CP5):** Vận hành tương tác nhiều lượt trực tiếp với Agent qua `chat.py`, ghi nhận biên bản hội thoại transcript JSON thực tế.
4. **Kiểm thử tích hợp hệ thống (Integration Testing):** Kiểm tra toàn bộ test suite của hệ thống với pytest, xác nhận tính tương thích và bảo đảm không còn test nào bị skip hay fail.

---

## 3. Đóng góp cụ thể

### 3.1. Thiết lập môi trường và cấu hình `.env`

- Tạo môi trường ảo Python 3.12 (`.venv`), cài đặt toàn bộ gói phụ thuộc trong `starter_v0/requirements.txt` (`openai`, `requests`, `PyYAML`, `pypdf`, `pytest`).
- Thiết lập file `starter_v0/.env` cấu hình các khóa API cần thiết:
  - `OPENAI_API_KEY`: dùng để gọi model `gpt-4o-mini`.
  - Khóa API cho các tool nghiên cứu: `TAVILY_API_KEY`, `RAPIDAPI_KEY` (Twitter timeline/search), `FIRECRAWL_API_KEY`.
- Kiểm tra kết nối provider và model, đảm bảo hệ thống sẵn sàng hoạt động mà không gặp lỗi xác thực hoặc mạng.

### 3.2. Chạy đánh giá Baseline `v0` và phân loại lỗi (CP1)

- Chạy lệnh eval baseline ban đầu:
  ```powershell
  python run_eval.py --provider openai --version v0 --suite base --eval-cases data/eval_base.json
  ```
- Kết quả run JSON đo đủ 20/20 case, 0 provider error, đạt **13/20 case (65%)**:
  - `case_accuracy`: **0.65**
  - `tool_routing_accuracy`: **0.75**
  - `argument_accuracy`: **0.65**
  - `multiturn_accuracy`: **1.00**
- Phân tích `failures` trong file run log `starter_v0/runs/v0_B_base_openai_20261110T191849190666.json`:
  - `wrong_tool` (R03, R13): Sai query hoặc chọn sai topic khi tìm kiếm tin tức web.
  - `out_of_scope` (R08, R14): Câu hỏi ngoài phạm vi nghiên cứu nhưng agent vẫn cố gọi tool.
  - `missing_info` (R10, R11): Thiếu thông tin bắt buộc nhưng starter prompt tự đoán mò thay vì hỏi lại (`clarify`).
  - `wrong_boundary` (R12): Gọi thẳng tool `send` mà không yêu cầu xác nhận.
- Bàn giao dữ liệu phân loại lỗi cho các thành viên trong nhóm làm căn cứ tối ưu prompt.

### 3.3. Kiểm thử Live Chat tương tác thực tế (Step 5 / CP5)

- Khởi chạy phiên Live Chat thực tế qua giao diện dòng lệnh với model OpenAI `gpt-4o-mini`:
  ```powershell
  python chat.py --provider openai --version v3
  ```
- Thực hiện chuỗi **4 lượt hội thoại** kiểm chứng thực tế các tính năng của Agent:
  1. **Lượt 1 — Tra cứu nghiên cứu với tool mới:**
     - *User:* `"Tìm giúp mình 3 model Qwen2.5 phổ biến nhất trên Hugging Face và tóm tắt ngắn gọn."`
     - *Agent:* Gọi tool mới `hf_search(query="qwen2.5", max_results=3)` lấy danh sách 3 model từ Hugging Face Hub, sau đó gọi `format(template="bullets")` để định dạng kết quả kèm link nguồn.
  2. **Lượt 2 — Xử lý câu hỏi thiếu thông tin:**
     - *User:* `"Cho mình xem 3 tweet mới nhất nhé."`
     - *Agent:* Nhận diện yêu cầu thiếu đối tượng/tài khoản.
  3. **Lượt 3 — Bổ sung thông tin ở lượt tiếp theo:**
     - *User:* `"Ý mình là của Andrej Karpathy nhé."`
     - *Agent:* Kế thừa ngữ cảnh `limit=3` từ lượt 2, ánh xạ chính xác tên Andrej Karpathy sang username `karpathy` và gọi `timeline(screenname="karpathy", limit=3)`.
  4. **Lượt 4 — Yêu cầu gửi ra ngoài & kiểm tra ranh giới an toàn (Guardrail):**
     - *User:* `"Gửi danh sách 3 model Qwen2.5 ở trên lên Telegram giúp mình."`
     - *Agent:* Ở round 1, tool `send` kích hoạt cơ chế bảo vệ chặn lại (`status="needs_confirmation"`).
- Toàn bộ phiên tương tác được tự động lưu vào transcript JSON thực tế:
  `starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json`.

### 3.4. Kiểm thử tích hợp hệ thống (Integration Testing)

- Thực hiện kiểm thử tích hợp toàn bộ test suite từ thư mục `starter_v0/`:
  ```powershell
  python -m pytest tests/ -v
  ```
- Kết quả kiểm thử: **93 passed, 0 failed, 0 skipped** (~1.4s):
  - `tests/test_tool_contract.py`: 65 passed (kiểm tra contract, schema của toàn bộ tool).
  - `tests/test_tools_examples.py`: 7 passed.
  - `tests/test_tools_todo.py`: 21 passed (xác nhận tất cả unit test chức năng đều pass, không còn test nào bị skip).

---

## 4. Bằng chứng bàn giao

| Hạng mục | File / Bằng chứng thực tế | Chi tiết |
|---|---|---|
| **Cấu hình môi trường** | `starter_v0/.env`, `starter_v0/requirements.txt` | File môi trường và thư viện cài đặt |
| **Run Baseline `v0`** | [`starter_v0/runs/v0_B_base_openai_20261110T191849190666.json`](../../starter_v0/runs/v0_B_base_openai_20261110T191849190666.json) | Đo 20 case, 0 lỗi provider, accuracy 65% |
| **Live Chat Transcript (CP5)** | [`starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json`](../../starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json) | File transcript ghi nhận 4 lượt hội thoại thật |
| **Bằng chứng Live Chat nhóm** | [`group_report.md`](group_report.md) (mục B4) | Bảng tổng hợp 4 lượt hội thoại kiểm thử thực tế |
| **Kiểm thử tích hợp (Pytest)** | `starter_v0/tests/` | Kết quả chạy pytest: **93 passed, 0 skipped, 0 failed** |
| **Bảng phân công nhóm** | [`TEAM.md`](../../TEAM.md) | Khai báo vai trò Testing trong nhóm |

---

## 5. Sử dụng AI và tính trung thực

- Sử dụng trợ lý AI hỗ trợ định dạng tài liệu Markdown.
- Toàn bộ kết quả chạy `v0`, transcript JSON và kết quả pytest đều được chạy thực tế trên máy với OpenAI API và Python, không làm giả số liệu.

---

## 6. Tự đánh giá

- **Mức độ hoàn thành:** Hoàn thành tốt các phần việc kỹ thuật được giao (setup môi trường, kiểm thử baseline v0, thực hiện live chat đa lượt và kiểm thử tích hợp hệ thống).
- **Điểm nổi bật:** Phiên Live Chat thực hiện đủ 4 lượt bao phủ trọn vẹn kịch bản rubric (tool mới, thiếu thông tin, ngữ cảnh đa lượt và guardrail an toàn).
