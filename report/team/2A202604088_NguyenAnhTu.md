# Báo cáo cá nhân: Nguyễn Anh Tú

## 1. Thông tin thành viên

| Thông tin | Chi tiết |
|---|---|
| Họ tên | Nguyễn Anh Tú |
| MSSV | 2A202604088 |
| Email | comtrang378@gmail.com |
| Vai trò | Phát triển giao diện người dùng (UI) |

## 2. Phần việc phụ trách

Phụ trách thiết kế và phát triển giao diện người dùng (UI) cho hệ thống Research Agent bằng Streamlit. Xây dựng giao diện tương tác giữa người dùng và AI Agent, hỗ trợ cấu hình kết nối với các nhà cung cấp mô hình ngôn ngữ lớn (LLM), gửi câu hỏi và hiển thị phản hồi từ Agent.

Bên cạnh đó, phụ trách hiển thị lịch sử hội thoại, thông tin về quá trình sử dụng công cụ (Tool Calling), quản lý trạng thái phiên làm việc và hỗ trợ tạo giao diện demo cho sản phẩm.

## 3. Đóng góp cụ thể

| STT | Công việc | Nội dung thực hiện |
|---:|---|---|
| 1 | Thiết kế giao diện | Sử dụng Streamlit xây dựng giao diện Research Agent với bố cục rộng, khu vực chat và thanh sidebar cấu hình. |
| 2 | Kết nối LLM Provider | Xây dựng chức năng lựa chọn và kết nối các Provider: OpenRouter, OpenAI, Anthropic, Gemini và Ollama. |
| 3 | Xây dựng giao diện Chat | Sử dụng `st.chat_input()` và `st.chat_message()` để tiếp nhận câu hỏi và hiển thị nội dung hội thoại giữa người dùng với Agent. |
| 4 | Quản lý hội thoại | Sử dụng `st.session_state` để lưu trữ lịch sử tin nhắn, trạng thái kết nối và thông tin phiên hội thoại. |
| 5 | Hiển thị Tool Calling | Xây dựng khu vực mở rộng bằng `st.expander()` để hiển thị chi tiết các công cụ mà Agent đã sử dụng. |
| 6 | Cấu hình Agent | Cho phép người dùng lựa chọn phiên bản prompt, điều chỉnh số lượt hội thoại được giữ trong ngữ cảnh và số vòng gọi Tool tối đa. |
| 7 | Hỗ trợ Demo | Bổ sung khu vực hướng dẫn truy cập ứng dụng thông qua localhost và hướng dẫn triển khai lên Streamlit Community Cloud. |

Ngoài ra, giao diện được tích hợp với các module xử lý phía Agent thông qua `run_model_tool_loop()`, giúp kết nối phần giao diện với quá trình xử lý yêu cầu và gọi công cụ của hệ thống.

## 4. Bằng chứng

### File triển khai chính

- `app.py`, 

### B5a. Required Evidence — Tool mới, unit test, UI

|Item|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Tool mới (`tools/<tên>/`)|[`github_search`](../../starter_v0/tools/github_search/tool.py), [`hf_search`](../../starter_v0/tools/hf_search/tool.py); đăng ký trong [`tools/__init__.py`](../../starter_v0/tools/__init__.py) và khai báo trong `artifacts/tools.yaml`|Bổ sung tìm repository GitHub và model/dataset Hugging Face; chuẩn hóa kết quả về `title`, `url`, `source`, `summary` cùng metadata riêng của từng nguồn. Eval nhóm có case dùng cả hai tool (G01–G03, G06–G10); run có 10/10 case được đo.|Run group ban đầu có HTTP 403 ở 3/7 lần gọi GitHub Search (G01, G03, G04); 4 lần còn lại không ghi lỗi HTTP trong `tool_results`. Điểm PASS chỉ xác nhận routing/arguments.|
|Unit test tool mới (≥ 3 test/tool)|[`tests/test_tools_todo.py`](../../starter_v0/tests/test_tools_todo.py); `pytest -q` từ `starter_v0/` → **93 passed, 0 failed, 0 skipped**|Đã viết **7 test** cho hai tool mới: 4 cho `github_search` và 3 cho `hf_search`, bao gồm request/response mapping, input không hợp lệ, lỗi HTTP/timeout. Test dùng HTTP giả lập, không cần gọi API thật.|Eval không kiểm tra phần triển khai HTTP hay chuẩn hóa payload. Ví dụ, nếu `github_search` lấy sai `html_url`/`full_name` từ payload thì unit test `test_github_search_sends_expected_request_and_maps_repos` bắt được, trong khi eval chỉ chấm tool và arguments do model chọn.|
|UI (lệnh chạy + link dùng thử)|Chạy từ `starter_v0/`: `streamlit run app.py`; local: [http://localhost:8501](http://localhost:8501)|App Streamlit cho phép chọn provider/model, nhãn phiên bản, chat và xem tool events.|Đây là link local, chỉ truy cập được khi app đang chạy trên máy người dùng; chưa có link deploy/public để báo cáo. Cần cấu hình API key phù hợp trong `.env` hoặc qua UI.|

**Tool unit tests:** 7 test cho tool mới (4 `github_search`, 3 `hf_search`). Ví dụ lỗi implementation mà eval không bắt được: API response có repository nhưng mapping trả sai URL hoặc tên; unit test kiểm tra request và object đã chuẩn hóa, còn eval chỉ chấm lựa chọn tool/arguments.

### B5. Bonus Evidence

Chỉ điền nếu nhóm làm bonus (ghi riêng, tối đa 10 điểm). Tool có sẵn (`send`, `policy`, `papers`, `paper\_text`) không tự được bonus.

|Bonus|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Small Local Model|[B7 báo cáo nhóm](group_report.md) và [run Qwen 7B](../../starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json)|Phạm Trấn Quốc đã chạy hai baseline local và hai vòng tối ưu có run JSON; kết quả chi tiết ở B7.|Llama 3B không hoàn tất full eval; không tính subset chẩn đoán là baseline.|
|Hơn 3 tool mới (≥ 4)|Chưa áp dụng: có 2 tool mới (`github_search`, `hf_search`).||Không đạt ngưỡng 4 tool mới; không tính tool có sẵn.|
|`send` gửi thật lên Telegram sau xác nhận|Chưa có transcript gửi Telegram thật được lưu trong workspace.||Không khẳng định đã gửi thật; unit test chỉ giả lập HTTP và không gửi Telegram.|
|Test challenge (0 skipped / regression test)|`pytest -q` từ `starter_v0/` → **93 passed, 0 skipped**; [`tests/test_tools_todo.py`](../../starter_v0/tests/test_tools_todo.py)|Các test challenge hiện chạy hết; có test kiểm tra policy đưa nội dung giống chỉ dẫn vào `untrusted_text`, không coi nội dung đó là facts.|Chưa có bằng chứng về bug tool có sẵn đã được xác định, sửa và gắn regression test cụ thể; vì vậy không tuyên bố đã đạt bonus bug-fix/regression.|

Test challenge: chưa ghi nhận bug tool có sẵn nào đã được xác nhận và sửa trong bằng chứng hiện có (0 bug-fix/regression được báo cáo). `test_policy_instruction_like_lines_go_to_untrusted_text` là kiểm tra guardrail hiện có, không được trình bày như regression test cho một bug đã sửa.

### AI/công cụ đã dùng và cách tự kiểm tra kết quả

- Python: Ngôn ngữ lập trình chính để phát triển ứng dụng.
- Streamlit: Xây dựng giao diện người dùng và các thành phần tương tác.
- LLM Providers: Hỗ trợ kết nối với OpenRouter, OpenAI, Anthropic, Gemini và Ollama.
- GitHub: Quản lý mã nguồn và tích hợp sản phẩm trong repository của nhóm.


## 5. Tự đánh giá

- Trong quá trình thực hiện, tôi đã hoàn thành phần phát triển giao diện người dùng cho hệ thống Research Agent, tập trung vào việc xây dựng giao diện trực quan, dễ sử dụng và hỗ trợ kết nối với các mô hình AI khác nhau.
- Thông qua công việc này, tôi có cơ hội củng cố kiến thức về Python, Streamlit, quản lý trạng thái ứng dụng và cách tích hợp giao diện với hệ thống AI Agent có sử dụng cơ chế Tool Calling.
- Tuy nhiên, phần triển khai giao diện vẫn có thể được cải thiện thêm về trải nghiệm người dùng, khả năng hiển thị trạng thái xử lý và tối ưu giao diện khi triển khai trên môi trường thực tế.
- Tôi đánh giá phần việc của mình đã đóng góp vào việc hoàn thiện sản phẩm ở khía cạnh giao diện và khả năng tương tác giữa người dùng với hệ thống Research Agent.
