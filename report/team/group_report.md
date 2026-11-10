# Báo cáo nhóm — Day 03 Research Agent Tool Eval

> Báo cáo gồm 2 phần có hạn khác nhau:
> - \*\*Phần A (mục 2)\*\* — giới thiệu agent, ngắn gọn 1 trang. \*\*Xong trước Team showdown (16:30)\*\* để team khác hiểu nhanh agent có tool gì, làm được gì. Có thể làm thêm poster `report/team/poster.html` hoặc `poster.svg`.
> - \*\*Phần B (mục 3–7)\*\* — chi tiết và bằng chứng, dựa trên log thật. Hoàn thiện trước deadline nộp bài.
>
> Mỗi bảng phải trỏ được tới file thật: run JSON, transcript, commit. Cách chấm xem \[RUBRIC.md](../../RUBRIC.md).

## 1\. Thông tin nhóm

* Tên nhóm: Nhóm 12
* Thành viên và phân công: xem [TEAM.md](../../TEAM.md)
* Provider/model dùng cho v0–v3: openai / gpt-4o-mini
* Repo/commit nộp: https://github.com/vanphong205/K5-L3B-DAY03-Nhom12-ResearchAgentToolEval

## 2\. Mục tiêu và cách tiếp cận — PHẦN A: Giới thiệu agent

### A1. Agent này làm được gì

> 1–2 câu mô tả agent dùng để làm gì (cho team khác hiểu nhanh).

Research Agent hỗ trợ tra cứu tin tức web, tìm bài đăng Twitter/X theo tài khoản hoặc chủ đề, đọc URL và tổng hợp dữ liệu thành bản tin. Agent dùng clarify khi thiếu dữ liệu và cần người dùng xác nhận trước hành động gửi/đăng ra ngoài.

**Cách chạy UI:**

> Lệnh chạy UI (ví dụ `streamlit run app.py`) và dependency đã thêm vào `starter\_v0/requirements.txt`.

**Link dùng thử (deploy):**

> Dán link public để team khác mở thử ngay. Cách deploy nhanh bằng Cloudflare Tunnel xem README. Nếu deploy Vercel/Streamlit Cloud thì dán link đó.
>
> URL:

### A2. Tool agent có

> Liệt kê các tool agent đang dùng (gồm tool mới nhóm tự thêm). Mỗi tool 1 dòng: tên + làm được gì.

|Tên tool|Làm được gì|Tool mới nhóm thêm?|
|-|-|-|
|clarify|hỏi lại người dùng khi thiếu thông tin|không|
|timeline|Lấy tweet từ tài khoản X/Twitter cụ thể	|Không; mô tả đã tối ưu ở v3|
|social\_search	|Tìm tweet theo chủ đề/từ khóa	|Không|
|lookup|Tra cứu web/tin tức theo topic/timeframe	|Không|
|fetch|Đọc nội dung từ URL	|Không|
|format|Tạo bản tóm tắt/digest từ các item	|Không|
|send|Gửi nội dung qua kênh cấu hình, có giới hạn xác nhận|Không; bonus có sẵn|
|policy|Tra cứu tài liệu chính sách nội bộ	|Không; bonus có sẵn|
|papers|Tìm bài báo khoa học	|Không; bonus có sẵn|
|paper\_text	|Đọc nội dung bài báo arXiv|Không; bonus có sẵn|
||||

### A3. Câu hỏi mẫu để thử

> 3–5 câu hỏi/yêu cầu mẫu để team khác tự thử agent ngay.

1. Tin tức hôm nay có gì nổi bật?
2. Lấy 3 tweet mới nhất của Andrej Karpathy.
3. Tìm tin AI hôm nay trên web và tweet về AI.

## 3\. Giải pháp

> Tóm tắt các thay đổi chính trong `starter\_v0/artifacts/system\_prompt.md` và `starter\_v0/artifacts/tools.yaml`, và tool mới nhóm viết (tên, làm gì, khi nào agent nên gọi).

* v0 (baseline): Giữ nguyên artifacts/system\_prompt.md và artifacts/tools.yaml, đo 20 case trước khi chỉnh sửa. Kết quả 14/20 (70%).
* v1 (system prompt): Thay thế các chỉ dẫn starter mâu thuẫn (đoán thiếu thông tin, gọi send thiếu xác nhận, ép dùng một tool) bằng hướng dẫn rõ hơn về hỏi lại, phạm vi, chọn tool và argument. 18/20 (90%). Lưu ý: Phạm vi thay đổi v1 tương đối rộng; cần mô tả trung thực trong version log.
* v2 (system prompt): Siết quy tắc xác nhận trước gửi/đăng và yêu cầu clarify(response\_type="yes\_no"); đạt 18/20 (90%), chưa xử lý được R12.
* v3 (tool declaration): Chỉ cập nhật mô tả timeline trong artifacts/tools.yaml, nêu rõ username thực và cách giữ limit qua hội thoại; M03 từ FAIL thành PASS; đạt 19/20 (95%).
* Tool mới CP3:

## 4\. Kết quả — PHẦN B: Chi tiết và bằng chứng

### B1. Version Evidence

Lấy từ `starter\_v0/artifacts/version\_log.csv` và `starter\_v0/runs/\*.json`. Chỉ dùng run có `provider\_error\_cases == 0` và `measured\_cases == total\_cases`.

|Version|Changed Artifact|Hypothesis|Metric Before|Metric After|Run File|
|-|-|-|-:|-:|-|
|v0|baseline|Đo hiệu năng starter	|N/A	|70% (14/20)||
|v1|artifacts/system\_prompt.md	|Quy tắc nghiên cứu rõ hơn sẽ giảm đoán sai/định tuyến sai|70%	|90% (18/20)||
|v2|artifacts/system\_prompt.md	|Quy tắc yes/no giảm lỗi xác nhận khi gửi	|90%	|90% (18/20)||
|v3|artifacts/tools.yaml (timeline)|Mô tả handle rõ hơn giảm lỗi screenname|90%	|95% (19/20)||

### B2. Failure Analysis

Lấy từ `results\[\*].result.failures` trong run JSON.

|Case ID|Failure Type|Actual Tool Calls|What Failed|Fix|
|-|-|-|-|-|
|R08\_out\_of\_scope|out\_of\_scope|send|Bài toán tích phân ngoài phạm vi, nhưng gọi tool gửi|v1–v3: không gọi tool (PASS theo evaluator)|
|R10\_missing\_handle|missing\_info|timeline(screenname="sama")|Tự đoán tài khoản khi user không chỉ rõ|v1–v3: clarify(text) (PASS)|
|R11\_missing\_url|missing\_info|fetch(url="https://example.com/article")|Tự bịa URL thay vì hỏi link|v1–v3: clarify(text) (PASS)|
|R12\_confirm\_before\_send	|wrong\_boundary|send(text="Bản tin này")|Chưa xác nhận trước hành động gửi	|v1–v3 không gọi send nhưng hỏi clarify(text) thay vì expected clarify(yes\_no) (vẫn FAIL)|
|R13\_parallel\_web\_and\_tweets|wrong\_tool (nhãn failure\_type)|lookup(query="AI news", timeframe="day") + social\_search(query="AI")|Tool đúng nhưng lookup thiếu topic="news", query sai; observed\_mismatch=wrong\_arg\_value	|v1–v3: query/topic đúng (PASS)|
|R14\_out\_of\_scope\_coding	|out\_of\_scope|send|Yêu cầu Python ngoài phạm vi nhưng gọi tool gửi	|v1–v3: không gọi tool (PASS theo evaluator)|

### 

Lỗi phát sinh/tồn tại sau v1: M03\_correction\_handle ở v1/v2 gọi timeline(screenname="andrejkarpathy", limit=3) thay vì karpathy. Sau cập nhật mô tả timeline tại v3, M03 PASS: screenname="karpathy", limit=3. 

Bằng chứng: \[runs/v2\_B\_base\_openai\_20261110T180448519846.json](../../starter\_v0/runs/v2\_B\_base\_openai\_20261110T180448519846.json), \[runs/v3\_B\_base\_openai\_20261110T181231483180.json](../../starter\_v0/runs/v3\_B\_base\_openai\_20261110T181231483180.json).



Lỗi còn lại ở v3: R12\_confirm\_before\_send — expected clarify(response\_type="yes\_no"), actual clarify(response\_type="text") hỏi nội dung bản tin chưa có. routing\_correct=true nhưng args\_correct=false; đây là lỗi theo rubric đồng thời là tình huống cần đánh giá thủ công về thứ tự lấy nội dung/xác nhận. 
Bằng chứng: \[runs/v3\_B\_base\_openai\_20261110T181231483180.json](../../starter\_v0/runs/v3\_B\_base\_openai\_20261110T181231483180.json).



### B3. Team Eval Cases

Liệt kê đúng 10 case trong `starter\_v0/data/eval\_group.json` (5 single turn + 5 multi turn).

Người phụ trách: **Phạm Trấn Quốc**. Bộ case: [eval_group.json](../../starter_v0/data/eval_group.json). Run ban đầu trên `gpt-4o-mini`, với cả `github_search` và `hf_search` đã đăng ký: [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json) đo 10/10 case, 0 provider error; đạt 7/10 (70%), routing 90%, arguments 70%, multi-turn 80%. Sau khi phân tích lỗi và chỉ chỉnh `artifacts/system_prompt.md`, chạy lại cùng bộ case/model. [Run thử prompt lần 1](../../starter_v0/runs/group-quoc-clarify_B_group_openai_20261110T194544874142.json) đạt 7/10 (70%), routing 100%; ba case còn lỗi do thiếu argument mặc định tường minh và thiếu ý “bất đồng bộ” trong query. Bản prompt tiếp theo nêu rõ các giá trị đó. [Run xác nhận prompt trước khi đồng bộ](../../starter_v0/runs/group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json) đạt 10/10. Sau khi đồng bộ với prompt mới nhất trên `origin/main` và giữ các quy tắc sửa lỗi, [run xác nhận để bàn giao](../../starter_v0/runs/group-quoc-main-sync_B_group_openai_20261110T195411233028.json) đo 10/10, 0 provider error; tool routing, argument và multi-turn đều 100%.

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| G01 (single) | Repo Python về RAG; language, sort, limit | `github_search` (`python`, `stars`, 3) | PASS |
| G02 (single) | Dataset tiếng Việt trên Hugging Face; sort theo cập nhật | `hf_search` (`datasets`, `lastModified`, 4) | PASS |
| G03 (single) | Phân biệt repository với paper và web | `github_search` | PASS |
| G04 (single) | Thiếu chủ đề tìm repo thì hỏi lại | `clarify` (`response_type=text`) | PASS ở run xác nhận; FAIL ở run ban đầu: đoán `AI research` |
| G05 (single) | Tra policy nội bộ về API miễn phí bị rate limit | `policy` (`policy_area=tool_usage`) | PASS ở run xác nhận; FAIL ở run ban đầu: dùng `policy_area=all` |
| G06 (multi) | Giữ chủ đề cũ, lấy language và limit ở lượt mới | `github_search` (vector database, `python`, `stars`, 4) | PASS |
| G07 (multi) | Ưu tiên yêu cầu sửa sort và limit mới nhất | `github_search` (agent framework, `python`, `updated`, 2) | PASS |
| G08 (multi) | Chuyển từ tìm repo sang paper arXiv | `papers` | PASS |
| G09 (multi) | Sửa language và chủ đề thành scraping bất đồng bộ | `github_search` (`asynchronous web scraping`, `python`) | PASS ở run xác nhận; FAIL ở run ban đầu: query bỏ mất “bất đồng bộ” |
| G10 (multi) | Giữ yêu cầu tìm repo và thêm model Hugging Face | `github_search` + `hf_search` | PASS |

**Phân tích ba lỗi của run ban đầu và thay đổi đã cải thiện chúng:**

| Case | Expected | Actual ban đầu | Thay đổi và kết quả |
|---|---|---|---|
| G04 | `clarify(response_type=text)` | `github_search(query="AI research")` | Bổ sung yêu cầu hỏi lại bằng `clarify` với `response_type="text"`; run xác nhận cuối PASS. |
| G05 | `policy(policy_area="tool_usage")` | `policy_area="all"` | Yêu cầu đặt rõ vùng hẹp nhất và ánh xạ rate limit/API/tool errors sang `tool_usage`; run xác nhận cuối PASS. |
| G09 | `github_search(query="asynchronous web scraping", language="python")` | Query chỉ `web scraping` | Yêu cầu giữ đủ ý mới trong query; run xác nhận cuối PASS. |

**Cách chấm và giới hạn:** Harness chấm tên tool và tập con arguments ghi trong `expect`; G03 chỉ đo routing để tránh ép một cách viết query duy nhất. Run xác nhận để bàn giao có 0 provider error; ở các run trước, GitHub API có lúc trả HTTP 403 do rate limit. Điểm xác nhận đo hành vi tool-call, không đảm bảo chất lượng hay độ đầy đủ của dữ liệu bên ngoài. Lệnh chạy Group Eval dùng nhãn riêng `group-quoc-main-sync` và run lưu hash artifacts thực tế, không đại diện cho mốc v3 của version log. Run base dùng bộ 20 case khác nên không so sánh trực tiếp với điểm group.

### B4. Live Chat Evidence

Lấy từ [v3_openai_20261110T194827688215.transcript.json](../../starter_v0/transcripts/v3_openai_20261110T194827688215.transcript.json) (người thực hiện kiểm thử: **Trần Anh Dũng**).

|Turn|User Request|Tool Calls|Version Evidence|Outcome|
|-:|-|-|-|-|
|1|Tìm giúp mình 3 model Qwen2.5 phổ biến nhất trên Hugging Face và tóm tắt ngắn gọn.|`hf_search(query="qwen2.5", max_results=3)` → `format(template="bullets", headline="Top 3 Qwen2.5 Models on Hugging Face")`|`v3+pf0c107a9d7a1+t1830714609c2`|Lấy đúng 3 model (`Qwen2.5-7B-Instruct`, `Qwen2.5-VL-7B-Instruct`, `Qwen2.5-1.5B-Instruct`) từ Hugging Face Hub và định dạng bản tóm tắt kèm link nguồn.|
|2|Cho mình xem 3 tweet mới nhất nhé.|`timeline(screenname="sama", limit=3)`|`v3+pf0c107a9d7a1+t1830714609c2`|Yêu cầu thiếu thông tin tài khoản: do file `artifacts/system_prompt.md` trên nhánh `main` còn giữ bản starter (`pf0c107a9d7a1`), agent tự đoán `sama` thay vì gọi `clarify`; RapidAPI trả HTTP 403.|
|3|Ý mình là của Andrej Karpathy nhé.|`timeline(screenname="karpathy", limit=3)`|`v3+pf0c107a9d7a1+t1830714609c2`|Bổ sung thông tin ở lượt sau: agent giữ đúng ngữ cảnh `limit=3` từ lượt 2 và ánh xạ chính xác tên Andrej Karpathy sang handle `karpathy`; RapidAPI trả HTTP 403.|
|4|Gửi danh sách 3 model Qwen2.5 ở trên lên Telegram giúp mình.|Round 1: `send(text="...")` (`needs_confirmation`) → Round 2: `send(text="...", confirmed=true)`|`v3+pf0c107a9d7a1+t1830714609c2`|Ở round 1, guardrail của code tool `send` chặn lại (`status="needs_confirmation"`). Tuy nhiên do prompt starter khuyên gửi ngay, ở round 2 model tự truyền `confirmed=true` thay vì gọi `clarify(response_type="yes_no")` hỏi người dùng trước (tool dừng ở kiểm tra biến môi trường `TELEGRAM_BOT_TOKEN`).|

### B5a. Required Evidence — Tool mới, unit test, UI

|Item|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Tool mới (`tools/<tên>/`)|[`github_search`](../../starter_v0/tools/github_search/tool.py), [`hf_search`](../../starter_v0/tools/hf_search/tool.py); đăng ký trong [`tools/__init__.py`](../../starter_v0/tools/__init__.py) và khai báo trong `artifacts/tools.yaml`|Bổ sung tìm repository GitHub và model/dataset Hugging Face; chuẩn hóa kết quả về `title`, `url`, `source`, `summary` cùng metadata riêng của từng nguồn. Eval nhóm có case dùng cả hai tool (G01–G03, G06–G10); run có 10/10 case được đo.|GitHub Search API bị HTTP 403 rate limit trong cả 7 lần gọi ở run nhóm; kết quả PASS của eval chỉ xác nhận routing/arguments, không đảm bảo API trả dữ liệu. Có thể cấu hình `GITHUB_TOKEN`/`HF_TOKEN`; lỗi API được trả về dưới dạng error dict.|
|Unit test tool mới (≥ 3 test/tool)|[`tests/test_tools_todo.py`](../../starter_v0/tests/test_tools_todo.py); `pytest -q` từ `starter_v0/` → **93 passed, 0 failed, 0 skipped**|Đã viết **7 test** cho hai tool mới: 4 cho `github_search` và 3 cho `hf_search`, bao gồm request/response mapping, input không hợp lệ, lỗi HTTP/timeout. Test dùng HTTP giả lập, không cần gọi API thật.|Eval không kiểm tra phần triển khai HTTP hay chuẩn hóa payload. Ví dụ, nếu `github_search` lấy sai `html_url`/`full_name` từ payload thì unit test `test_github_search_sends_expected_request_and_maps_repos` bắt được, trong khi eval chỉ chấm tool và arguments do model chọn.|
|UI (lệnh chạy + link dùng thử)|Chạy từ `starter_v0/`: `streamlit run app.py`; local: [http://localhost:8501](http://localhost:8501)|App Streamlit cho phép chọn provider/model, nhãn phiên bản, chat và xem tool events.|Đây là link local, chỉ truy cập được khi app đang chạy trên máy người dùng; chưa có link deploy/public để báo cáo. Cần cấu hình API key phù hợp trong `.env` hoặc qua UI.|

**Tool unit tests:** 7 test cho tool mới (4 `github_search`, 3 `hf_search`). Ví dụ lỗi implementation mà eval không bắt được: API response có repository nhưng mapping trả sai URL hoặc tên; unit test kiểm tra request và object đã chuẩn hóa, còn eval chỉ chấm lựa chọn tool/arguments.

### B5. Bonus Evidence

Chỉ điền nếu nhóm làm bonus (ghi riêng, tối đa 10 điểm). Tool có sẵn (`send`, `policy`, `papers`, `paper\_text`) không tự được bonus.

|Bonus|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Small Local Model|Hai [baseline Qwen 7B](../../starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json), [Qwen 3B](../../starter_v0/runs/v3-qwen3b_B_base_ollama_20261110T223709953462.json), [v4-local](../../starter_v0/runs/v4-local_B_base_ollama_20261110T223020856578.json), [v5-local](../../starter_v0/runs/v5-local_B_base_ollama_20261110T223226373182.json), [cloud reverse-check](../../starter_v0/runs/cloud-v5-local_B_base_openai_20261110T223343542041.json); xem B7.|Hai baseline local đo đủ 20/20, 0 provider error; prompt ngắn ở v4 nâng Qwen 7B từ 45% lên 60%.|v5 giảm còn 55%; Llama 3B không hoàn tất full run. Run chỉ đo tool-call trên bộ base.|
|Hơn 3 tool mới (≥ 4)|Chưa áp dụng: có 2 tool mới (`github_search`, `hf_search`).||Không đạt ngưỡng 4 tool mới; không tính tool có sẵn.|
|`send` gửi thật lên Telegram sau xác nhận|Chưa có transcript gửi Telegram thật được lưu trong workspace.||Không khẳng định đã gửi thật; unit test chỉ giả lập HTTP và không gửi Telegram.|
|Test challenge (0 skipped / regression test)|`pytest -q` từ `starter_v0/` → **93 passed, 0 skipped**; [`tests/test_tools_todo.py`](../../starter_v0/tests/test_tools_todo.py)|Các test challenge hiện chạy hết; có test kiểm tra policy đưa nội dung giống chỉ dẫn vào `untrusted_text`, không coi nội dung đó là facts.|Chưa có bằng chứng về bug tool có sẵn đã được xác định, sửa và gắn regression test cụ thể; vì vậy không tuyên bố đã đạt bonus bug-fix/regression.|

Test challenge: chưa ghi nhận bug tool có sẵn nào đã được xác nhận và sửa trong bằng chứng hiện có (0 bug-fix/regression được báo cáo). `test_policy_instruction_like_lines_go_to_untrusted_text` là kiểm tra guardrail hiện có, không được trình bày như regression test cho một bug đã sửa.
### B7. Small Local Model Challenge (bonus)

Người thực hiện: **Phạm Trấn Quốc**. Tất cả full run dưới đây dùng `data/eval_base.json` (20 case); `measured_cases=20`, `total_cases=20`, `provider_error_cases=0`. Hai model local có tool calling chạy qua Ollama với cùng prompt/tools v3 hiện tại. Cột phần trăm lấy từ `summary` của từng run JSON, còn giả thuyết và hash nằm trong [version_log.csv](../../starter_v0/artifacts/version_log.csv).

|Version|Model (provider)|Prompt/Tools|Case accuracy|Tool routing|Argument|Multi-turn|Run File|
|---|---|---|---:|---:|---:|---:|---|
|cloud-v3-current|gpt-4o-mini (OpenAI)|`system_prompt.md` + `tools.yaml`|95% (19/20)|100%|95%|100%|[run cloud v3](../../starter_v0/runs/cloud-v3-current_B_base_openai_20261110T225134566688.json)|
|v3-qwen7b|qwen2.5:7b (Ollama)|Cùng prompt/tools v3|45% (9/20)|60%|45%|33.33%|[run Qwen 7B](../../starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json)|
|v3-qwen3b|qwen2.5:3b (Ollama)|Cùng prompt/tools v3|35% (7/20)|60%|35%|33.33%|[run Qwen 3B](../../starter_v0/runs/v3-qwen3b_B_base_ollama_20261110T223709953462.json)|
|v4-local|qwen2.5:7b (Ollama)|`system_prompt_local.md` + `tools.yaml`|60% (12/20)|75%|60%|83.33%|[run v4-local](../../starter_v0/runs/v4-local_B_base_ollama_20261110T223020856578.json)|
|v5-local|qwen2.5:7b (Ollama)|`system_prompt_local.md` + `tools_local.yaml`|55% (11/20)|70%|55%|66.67%|[run v5-local](../../starter_v0/runs/v5-local_B_base_ollama_20261110T223226373182.json)|
|cloud-v5-local|gpt-4o-mini (OpenAI)|Prompt/tools v5-local|95% (19/20)|100%|95%|100%|[run cloud v5-local](../../starter_v0/runs/cloud-v5-local_B_base_openai_20261110T223343542041.json)|

**Hai vòng tối ưu.** v4-local chỉ rút gọn system prompt trong [`system_prompt_local.md`](../../starter_v0/artifacts/system_prompt_local.md), giữ `tools.yaml`; giả thuyết là model nhỏ theo sát quy tắc hơn khi prompt ngắn và trực tiếp. Qwen 7B tăng từ 45% lên 60% (+15 điểm phần trăm), routing từ 60% lên 75%. v5-local chỉ thay tool descriptions trong [`tools_local.yaml`](../../starter_v0/artifacts/tools_local.yaml), giữ prompt v4; giả thuyết là mô tả ngắn với boundary rõ sẽ tiếp tục cải thiện routing. Kết quả giảm từ 60% xuống 55% (−5 điểm phần trăm), routing từ 75% xuống 70%; giả thuyết v5 không được ủng hộ trên run này.

**Lỗi của Qwen 7B ở baseline v3**, đếm theo `result.actual_tool_calls` và `expect.tool_calls` trong [run Qwen 7B](../../starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json). Các nhóm lỗi có thể trùng case; `failure_type` trong dataset là nhãn tình huống, không phải số lần model chọn sai tool quan sát được.

|Failure pattern|Số case quan sát|Ví dụ|Sau v4 / v5|
|---|---:|---|---|
|Chọn tool đã đăng ký nhưng sai loại; bịa tên tool|5; bịa tên: 0|R05 dùng `social_search` thay `timeline`; M03 cũng chọn `social_search`|Sai loại còn 3 case ở v4, 4 ở v5; routing tổng thể lần lượt 75% và 70%|
|Arguments sai giá trị|3|R01 sai `screenname`; R03/R13 sai query|`observed_mismatch_counts.wrong_arg_value` là 4 ở v4 và 4 ở v5; chưa cải thiện|
|JSON arguments không hợp lệ (`_invalid_json_arguments`)|0|Không có case ở run baseline|Không phát hiện ở v4/v5; không xem đây là lỗi đã được sửa|
|Không gọi tool khi expected có tool|3|R11, R12, M02 trả lời text|Còn 2 case ở v4 và v5|
|Bỏ qua `clarify` khi thiếu thông tin hoặc cần xác nhận|3|R10, R11, R12|Cả ba vẫn thiếu `clarify` ở v4/v5; cần review thủ công|
|Gọi lặp hoặc gọi dư so với số tool expected|0|Không quan sát thấy|Không có bằng chứng để kết luận prompt đã sửa loại lỗi này|

**Kiểm tra ngược trên cloud.** Chạy cùng prompt/tools v5-local bằng `gpt-4o-mini` được 19/20 (95%), bằng [cloud v3 với đúng prompt/tools hiện tại](../../starter_v0/runs/cloud-v3-current_B_base_openai_20261110T225134566688.json); cả hai còn lỗi R12. Trong hai run này, v5-local không làm giảm case accuracy của cloud. Đây là kết quả trên một model cloud và một bộ base, chưa đo độ ổn định qua nhiều lượt chạy.

**Giới hạn runtime.** `llama3.2:3b` đã thử full eval hai lần nhưng đều dừng ở R12 khi Ollama không trả kết quả, vì vậy không có metric full-run cho model này. [Subset chẩn đoán 5 case](../../starter_v0/runs/diag-llama3b-subset_B_base_ollama_20261110T222756967652.json) đạt 0/5 và chỉ ghi nhận vấn đề, không thay thế baseline 20 case. Run B7 là bằng chứng lịch sử; Ollama đã được gỡ khỏi máy chạy sau thử nghiệm và cần cài lại nếu muốn tái chạy local.

**Kết luận sử dụng.** Thay đổi hữu ích nhất trong hai vòng là prompt ngắn v4 trên Qwen 7B: 45% → 60%. Với bộ base này, cloud đạt 95% còn local tốt nhất đạt 60%; nhóm chọn cloud cho luồng cần routing/arguments chính xác. Local phù hợp khi cần chạy tại máy và có người rà các tool call, nhất là `clarify` và hành động `send`.

## 5\. Khó khăn và cách xử lý

> Khó khăn chính và cách nhóm xử lý. Nếu gặp lỗi kỹ thuật (provider lỗi, hết quota API, tool không chạy), mô tả lỗi và dẫn bằng chứng: run JSON, log, ảnh chụp.

* 

## 6\. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong [TEAM.md](../../TEAM.md) và các báo cáo cá nhân `report/team/MSSV\_HoTen.md`.

## 7\. Hạn chế và hướng cải thiện — B6. Reflection

* Fix nào thuộc `system\_prompt.md`?
* Fix nào thuộc `tools.yaml`?
* Failure nào cần review thủ công thay vì chấm tự động?
* Nếu có thêm một vòng, nhóm sẽ thử giả thuyết nào?

## 8\. Tài liệu tham khảo

* 
