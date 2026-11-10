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

|Case ID|What It Tests|Expected Tool/Behavior|Result|
|-|-|-|-|
|||||

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

