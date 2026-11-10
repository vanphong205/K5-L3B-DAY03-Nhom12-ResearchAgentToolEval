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

Người phụ trách: **Phạm Trấn Quốc**. Bộ case: [eval_group.json](../../starter_v0/data/eval_group.json). Chạy bằng `python run_eval.py --provider openai --version v3 --suite group --eval-cases data/eval_group.json` trên `gpt-4o-mini`, với cả `github_search` và `hf_search` đã được đăng ký. [Run JSON](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json) có 10/10 case được đo, 0 provider error; 7/10 case đạt (70%), routing 90%, arguments 70%, multi turn 80%.

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| G01 (single) | Repo Python về RAG; language, sort, limit | `github_search` (`python`, `stars`, 3) | PASS |
| G02 (single) | Dataset tiếng Việt trên Hugging Face; sort theo cập nhật | `hf_search` (`datasets`, `lastModified`, 4) | PASS |
| G03 (single) | Phân biệt repository với paper và web | `github_search` | PASS |
| G04 (single) | Thiếu chủ đề tìm repo thì hỏi lại | `clarify` (`response_type=text`) | FAIL: đoán `AI research` và gọi `github_search` |
| G05 (single) | Tra policy nội bộ về API miễn phí bị rate limit | `policy` (`policy_area=tool_usage`) | FAIL: dùng `policy_area=all` |
| G06 (multi) | Giữ chủ đề cũ, lấy language và limit ở lượt mới | `github_search` (vector database, `python`, `stars`, 4) | PASS |
| G07 (multi) | Ưu tiên yêu cầu sửa sort và limit mới nhất | `github_search` (agent framework, `python`, `updated`, 2) | PASS |
| G08 (multi) | Chuyển từ tìm repo sang paper arXiv | `papers` | PASS |
| G09 (multi) | Sửa language và chủ đề thành scraping bất đồng bộ | `github_search` (`asynchronous web scraping`, `python`) | FAIL: query bỏ mất “bất đồng bộ” |
| G10 (multi) | Giữ yêu cầu tìm repo và thêm model Hugging Face | `github_search` + `hf_search` | PASS |

**Phân tích ba case FAIL** (đọc từ `results[*].result` trong run JSON):

| Case | Expected | Actual | Nguyên nhân và hướng xử lý |
|---|---|---|---|
| G04 | `clarify(response_type=text)` | `github_search(query="AI research")` | Agent đoán chủ đề khi người dùng chưa nêu. Prompt v0 đang khuyến khích đoán; cần yêu cầu hỏi lại khi thiếu chủ đề tìm kiếm. |
| G05 | `policy(policy_area="tool_usage")` | `policy(query="API nghiên cứu miễn phí bị rate limit", policy_area="all")` | Chọn đúng tool nhưng dùng vùng chung; cần mô tả rõ quy định rate limit thuộc `tool_usage` trong `tools.yaml`. |
| G09 | `github_search(query="asynchronous web scraping", language="python")` | `github_search(query="web scraping", language="python")` | Đổi đúng ngôn ngữ nhưng bỏ điều kiện “bất đồng bộ” ở lượt mới; cần giữ mọi điều kiện mới khi viết lại query. |

**Cách chấm và giới hạn:** Harness chấm tên tool và tập con arguments ghi trong `expect`; G03 chỉ đo routing để tránh ép một cách viết query duy nhất. GitHub API trả HTTP 403 do rate limit ở 3 lần gọi GitHub (G01, G03, G04); G02 gọi đúng `hf_search` nhưng kết quả dataset rỗng. Điểm PASS xác nhận lựa chọn tool/arguments theo harness, không xác nhận đã lấy được nội dung hữu ích từ API. Lệnh chạy group mang nhãn `v3`, nhưng `prompt_hash=eb1c8179815b` / `tools_hash=ee32b4055306` không khớp hash v3 ở `version_log.csv` (`c800280bc6ca` / `0f6be0ee4fef`); vì vậy run này chưa chứng minh group eval đã được đo với đúng artifacts v3. Run base dùng bộ 20 case khác, nên không so trực tiếp 70% group với điểm base.

### B4. Live Chat Evidence

Lấy từ `starter\_v0/transcripts/\*.transcript.json`.

|Turn|User Request|Tool Calls|Version Evidence|Outcome|
|-|-|-|-|-|
||||||

### B5a. Required Evidence — Tool mới, unit test, UI

|Item|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Tool mới (`tools/<tên>/`)||||
|Unit test tool mới (≥ 3 test/tool)|`pytest` output (0 failed)|||
|UI (lệnh chạy + link dùng thử)||||

Tool unit tests: số test đã viết, và một ví dụ lỗi mà unit test bắt được nhưng eval không bắt được.

### B5. Bonus Evidence

Chỉ điền nếu nhóm làm bonus (ghi riêng, tối đa 10 điểm). Tool có sẵn (`send`, `policy`, `papers`, `paper\_text`) không tự được bonus.

|Bonus|Evidence File|What Worked|Risk / Guardrail|
|-|-|-|-|
|Small Local Model|xem B7|||
|Hơn 3 tool mới (≥ 4)||||
|`send` gửi thật lên Telegram sau xác nhận|transcript|||
|Test challenge (0 skipped / regression test)|`pytest` output|||

Test challenge: số bug tìm được trong tool có sẵn (nếu có), cách sửa, và test regression tương ứng.

### B7. Small Local Model Challenge (bonus)

Chỉ điền nếu nhóm làm bonus này.

|Version|Model (provider)|Prompt/Tools|case\_accuracy|tool\_routing|argument|multiturn|Run File|
|-|-|-|-|-|-|-|-|
|v3|(cloud)|v3||||||
|v3-<model>|(ollama)|v3||||||
|v3-<model>|(ollama)|v3||||||
|v4-local||||||||
|v5-local||||||||

Lỗi đặc thù của model nhỏ (đếm từ run JSON):

|Failure Pattern|Count|Example Case|Fix Tried|Helped?|
|-|-|-|-|-|
|wrong / hallucinated tool|||||
|invalid JSON args (`\_invalid\_json\_arguments`)|||||
|no tool call / answered directly|||||
|missed `clarify`|||||
|repeated / unnecessary calls|||||

* Cái gì giúp model nhỏ gọi tool đúng hơn nhất? Bằng chứng?
* Bản prompt tối ưu cho local chạy trên model cloud thì sao?
* Khi nào nhóm sẽ chọn model local thay vì cloud cho agent này?

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
