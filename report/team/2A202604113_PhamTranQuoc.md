# Báo cáo cá nhân — Phạm Trấn Quốc (2A202604113)

## 1. Thông tin thành viên

- Họ tên: Phạm Trấn Quốc
- MSSV: 2A202604113
- Email: phamquoc896@gmail.com
- GitHub: [Quoosc](https://github.com/Quoosc)
- Nhóm/lớp: Nhóm 12, L3B
- Vai trò: Evaluation

## 2. Vai trò và mục tiêu công việc

Tôi phụ trách **CP4 — Team Evaluation**, tương ứng với **C5 — 10 case eval nhóm** trong rubric. Mục tiêu của phần này là bổ sung một bộ kiểm tra do nhóm tự thiết kế để đo các tình huống mà bộ base chưa tập trung, đặc biệt là việc chọn đúng tool mới, truyền đúng tham số, giữ và sửa thông tin qua nhiều lượt, cũng như chuyển giữa các nguồn tra cứu.

Deliverable của CP4 gồm ba phần liên kết với nhau: bộ 10 case trong `starter_v0/data/eval_group.json`, một run được tạo bởi `run_eval.py --suite group`, và phần mô tả kết quả trong bảng B3 của `report/team/group_report.md`. Rubric C5 yêu cầu đúng 5 case single-turn và 5 case multi-turn, mỗi case có expected behavior cùng `metadata.what_it_tests`, và phải có case kiểm tra route tới tool mới.

Group Eval đánh giá quyết định của model trước khi nội dung tool được dùng: tool name, số lần gọi và các arguments được nêu trong `expect`. Đây là phần khác với unit test của tool, vốn kiểm tra implementation Python như xử lý input xấu hoặc lỗi HTTP. Vì vậy, tôi giữ nội dung đánh giá ở mức agent behavior và ghi riêng khi API trả lỗi để không nhầm lỗi dịch vụ với lỗi routing.

## 3. Công việc đã thực hiện

### 3.1. Rà yêu cầu và contract của harness

Trước khi tạo case, tôi rà `README.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`, mẫu group report, `starter_v0/run_eval.py`, tool declarations trong `artifacts/tools.yaml` và registry trong `tools/__init__.py`. Việc này giúp xác định schema mà harness thực sự đọc: `phase`, `query` hoặc `turns`, `failure_type`, `expect` và `metadata.what_it_tests`.

Tôi cũng kiểm tra cách chấm trong `run_eval.py`. Harness so tên tool và một tập con arguments được khai báo trong `expect`; chuỗi được chuẩn hóa chữ hoa/thường và khoảng trắng, nhưng không đánh giá ngữ nghĩa của hai câu query tương đương. Do đó, G03 chỉ kiểm tra routing sang `github_search`, không khóa expected vào một cách viết query duy nhất.

### 3.2. Thiết kế bộ 10 case

Bộ case có đúng 10 ID duy nhất: G01–G05 là single-turn dùng `query`; G06–G10 là multi-turn dùng `turns`. Nội dung được viết mới cho Group Eval và rà lại để tránh lặp nguyên văn các câu hỏi trong `eval_base.json` hoặc `eval_research_extension.json`.

| Case | Hành vi được kiểm tra | Expected chính |
|---|---|---|
| G01 | Tìm GitHub repository Python về RAG, trích ngôn ngữ, sort và số lượng | `github_search` với `python`, `stars`, limit 3 |
| G02 | Tìm dataset tiếng Việt trên Hugging Face, chọn loại dataset và sort theo cập nhật | `hf_search` với `kind=datasets`, `sort=lastModified`, limit 4 |
| G03 | Phân biệt yêu cầu tìm repository mã nguồn với tìm paper hoặc tra web | `github_search` |
| G04 | Nhận ra thiếu chủ đề tìm repository và hỏi lại thay vì tự đoán | `clarify(response_type=text)` |
| G05 | Tra chính sách nội bộ về cách ghi nhận API rate limit | `policy(policy_area=tool_usage)` |
| G06 | Giữ chủ đề vector database từ lượt trước, nhận ngôn ngữ và limit ở lượt mới | `github_search` với `python`, `stars`, limit 4 |
| G07 | Áp dụng yêu cầu đổi sort và limit trong lượt sau, giữ query/language cũ | `github_search` với `updated`, limit 2 |
| G08 | Chuyển từ yêu cầu tìm repository sang tìm paper arXiv | `papers` |
| G09 | Cập nhật ngôn ngữ và chủ đề query trong hội thoại nhiều lượt | `github_search` với Python và query giữ ý “asynchronous” |
| G10 | Kết hợp yêu cầu tìm repository GitHub trước đó với model trên Hugging Face | Gọi cả `github_search` và `hf_search` |

G02 và G10 kiểm tra hai tool mới đang có trong checkout: `hf_search` và `github_search`. G10 kiểm tra routing tới cả hai tool trong cùng lượt. G04 kiểm tra hành vi khi thiếu thông tin; G05 kiểm tra lựa chọn vùng policy; G06, G07 và G09 tập trung vào carry-over/correction trong hội thoại; G08 kiểm tra chuyển đổi loại nguồn.

Trong quá trình rà, tôi đổi G05 từ một câu hỏi về API key vì nội dung đó quá gần case data-privacy có sẵn trong extension. Case cuối chuyển sang quy tắc xử lý rate limit thuộc `tool_usage`. Tôi cũng điều chỉnh G10 để đo tình huống kết hợp GitHub với Hugging Face. Các quyết định này được ghi lại trong lịch sử thay đổi của bộ eval.

### 3.3. Chạy eval và phân tích run JSON

Lệnh dùng để tạo run chính, chạy từ thư mục `starter_v0/`:

```powershell
python run_eval.py --provider openai --version v3 --suite group --eval-cases data/eval_group.json
```

Run được lưu tại [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json). Harness đo đủ 10 case và không có `provider_error_cases`. Tôi kiểm tra `summary`, từng `results[*].result`, expected/actual tool calls và `tool_results`, sau đó ghi kết quả cùng giới hạn runtime vào B3.

## 4. Kết quả

| Chỉ số | Run ban đầu | Run xác nhận sau cải thiện |
|---|---:|---:|
| Tổng case / case được đo | 10 / 10 | 10 / 10 |
| Case đạt | 7/10 (70%) | 10/10 (100%) |
| Tool routing accuracy | 90% | 100% |
| Argument accuracy | 70% | 100% |
| Multi-turn accuracy | 80% (4/5) | 100% (5/5) |
| Provider error | 0 | 0 |
| Single-turn đạt | 3/5 | 5/5 |
| Multi-turn đạt | 4/5 | 5/5 |

Tôi xem từng lỗi của run gốc, cập nhật `starter_v0/artifacts/system_prompt.md`, rồi chạy lại nguyên bộ `eval_group.json` với model `gpt-4o-mini`. Lần chạy thử đầu đã nâng routing lên 100%, nhưng vẫn 7/10 vì ba arguments chưa khớp hoàn toàn. Tôi làm rõ tiếp cách gọi `clarify` và `policy`, đồng thời yêu cầu giữ cụm khái niệm mới khi sửa query. Run xác nhận đạt 10/10. Dataset, expected và cách chấm được giữ nguyên.

| Case | Lỗi ban đầu | Điều chỉnh hành vi | Kết quả cuối |
|---|---|---|---|
| G04 — thiếu chủ đề | Agent tự đoán `AI research`; lần chạy thử sau hỏi lại nhưng bỏ `response_type` | Hỏi bằng `clarify` và đặt rõ `response_type="text"` | PASS |
| G05 — policy về rate limit | Chọn `policy` nhưng đặt `policy_area=all` | Chọn rõ `policy_area="tool_usage"` cho API limits và xử lý lỗi tool | PASS |
| G09 — sửa query multi-turn | Đổi language sang Python nhưng bỏ ý “bất đồng bộ” | Bao gồm khái niệm mới trong query hoàn chỉnh `asynchronous web scraping` | PASS |

Run ban đầu có các phản hồi HTTP 403 từ GitHub Search API ở G01, G03 và G04; lần chạy xác nhận có 0 provider error. Metric đánh giá lựa chọn tool và arguments theo harness, không tự nó xác nhận chất lượng dữ liệu từ API ngoài. Hai lần thử prompt được lưu riêng để giữ lại lịch sử: lần thử đầu đạt 7/10, lần xác nhận đạt 10/10.

## 5. Thay đổi và phương pháp xác nhận

Prompt hiện nêu rõ: (1) không đoán chủ đề thiết yếu còn thiếu và hỏi lại qua `clarify` với loại phản hồi text; (2) với policy nội bộ, phải đặt vùng hẹp nhất và ánh xạ câu hỏi về rate limit/API/tool sang `tool_usage`; (3) ở hội thoại nhiều lượt, giữ ràng buộc còn hiệu lực và đưa đủ khái niệm mới vào query. Không sửa dataset hoặc expected. Lần chạy đầu sau thay đổi đạt 7/10; sau khi thêm các hướng dẫn argument/query cụ thể, run xác nhận đạt 10/10 trên đúng 10 case hiện hữu.

Run xác nhận ban đầu: [group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json](../../starter_v0/runs/group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json). Run cuối dùng prompt đã đồng bộ với `origin/main`: [group-quoc-main-sync_B_group_openai_20261110T195411233028.json](../../starter_v0/runs/group-quoc-main-sync_B_group_openai_20261110T195411233028.json) — 10/10 case, 0 provider error, các metric routing/argument/multi-turn đều 100%. Thử nghiệm trung gian: [group-quoc-clarify_B_group_openai_20261110T194544874142.json](../../starter_v0/runs/group-quoc-clarify_B_group_openai_20261110T194544874142.json). Baseline: [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json). Các run dùng cùng model và dataset; mỗi run lưu artifact version cùng prompt/tools hashes thực tế, không gắn nhầm vào mốc v3 của version log.

Điểm 70% của bộ group cũng không được so trực tiếp với điểm base: hai run dùng bộ case khác nhau. Group Eval phục vụ đo các case nhóm tự thiết kế, trong khi run base dùng 20 case cố định. Kết quả hữu ích nhất ở đây là các lỗi cụ thể và số đo trong cùng một bộ group khi chạy lại với đúng artifacts.

## 6. Bằng chứng và đóng góp vào repo

| Hạng mục | Bằng chứng |
|---|---|
| Bộ case | [eval_group.json](../../starter_v0/data/eval_group.json) |
| Run Group Eval ban đầu | [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json) |
| Run xác nhận sau cải thiện | [group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json](../../starter_v0/runs/group-quoc-clarify-v2_B_group_openai_20261110T194638090929.json) |
| Run xác nhận sau đồng bộ `origin/main` | [group-quoc-main-sync_B_group_openai_20261110T195411233028.json](../../starter_v0/runs/group-quoc-main-sync_B_group_openai_20261110T195411233028.json) |
| Kết quả B3 và phân tích lỗi | [group_report.md](group_report.md) |
| Kiểm tra registry/schema | `python -m pytest tests/test_tool_contract.py -q` — 65 passed sau khi tích hợp `hf_search` |
| Commit cá nhân trên nhánh nộp | `d067aa9`, `6cb1a4d`, `c1047e4`, `c07d09c`; các thay đổi được merge vào `main` |

Các file trong phạm vi đóng góp của tôi gồm `starter_v0/data/eval_group.json`, run JSON của Group Eval, mục B3 của báo cáo nhóm, `TEAM.md` và báo cáo cá nhân này. Trong lượt cải thiện sau đó, tôi cũng chỉnh `starter_v0/artifacts/system_prompt.md` dựa trên lỗi quan sát được trong suite group. Tool implementations của `github_search` và `hf_search` do phần việc Tool Development cung cấp; tôi sử dụng chúng làm đối tượng kiểm tra và không nhận là tác giả implementation.

### Chi tiết các phần việc Evaluation

1. **Thiết kế bộ kiểm tra:** Xây dựng 10 case trong `eval_group.json`, chia đúng 5 single-turn và 5 multi-turn. Các case bao phủ định tuyến GitHub repository (G01, G03), Hugging Face dataset/model (G02, G10), yêu cầu thiếu thông tin (G04), policy nội bộ về lỗi API (G05), giữ và cập nhật ràng buộc qua hội thoại (G06, G07, G09), và chuyển loại nguồn từ repository sang paper (G08). Expected tập trung vào tool-call và các argument quan trọng; G03 chỉ chấm routing để không ép một cách viết truy vấn duy nhất.
2. **Thiết lập và chạy đánh giá:** Chạy baseline Group Eval với `gpt-4o-mini` trên đủ 10 case và lưu nguyên kết quả thành run JSON. Sau khi sửa prompt, chạy thêm một lượt chẩn đoán trung gian rồi lượt xác nhận cuối với cùng dataset/model. Mỗi lượt được lưu thành file riêng để giữ lịch sử và tránh thay thế số liệu cũ.
3. **Phân tích lỗi theo bằng chứng:** Đối chiếu expected với actual tool calls của G04, G05 và G09. Phân loại cụ thể lỗi thiếu thông tin, chọn policy area quá rộng, và bỏ sót ý mới trong query nhiều lượt. Không đổi nội dung case hoặc tiêu chí chấm để làm tăng điểm.
4. **Cải thiện và xác nhận:** Cập nhật system prompt để agent hỏi lại qua `clarify` với `response_type="text"` khi thiếu chủ đề thiết yếu; đặt rõ `policy_area="tool_usage"` cho câu hỏi về rate limit/API/tool; và đưa khái niệm mới vào query đã cập nhật. Lượt chẩn đoán đầu đạt 7/10 và chỉ ra thiếu argument cụ thể; sau khi làm rõ các argument đó, lượt xác nhận đạt 10/10, routing 100%, argument 100%, multi-turn 100%, provider error 0.
5. **Bàn giao báo cáo:** Cập nhật B3 trong báo cáo nhóm và bảng đóng góp này để ghi rõ baseline, lượt chẩn đoán, run xác nhận, nguyên nhân lỗi, hướng khắc phục và giới hạn diễn giải. Kết quả 100% gắn với đúng artifact version/hash trong run xác nhận; không ghi nhầm thành số đo của CP2 v3. Các lỗi HTTP 403 của GitHub ở lần chạy cũ cũng được nêu tách biệt khỏi provider error và điểm hành vi tool-call.

Phạm vi đóng góp tập trung vào thiết kế eval, chạy harness, phân tích hành vi và ghi nhận kết quả. Tôi không nhận quyền tác giả đối với implementation của hai tool mới, và không coi điểm tool-call là bằng chứng rằng API luôn trả dữ liệu đầy đủ hoặc hữu ích.

**Công cụ hỗ trợ và tự kiểm tra:** Tôi dùng Codex để rà yêu cầu rubric, hỗ trợ biên tập case và tổng hợp phần mô tả. Số liệu được lấy từ `run_eval.py` và run JSON do provider thật tạo ra. Tôi đối chiếu số lượng 5+5, ID case, summary, expected/actual tool calls, lỗi API và hash artifacts. Codex không được dùng để tạo run hay metric thay cho kết quả thực tế.

## 7. Tự đánh giá và việc tiếp theo

Phần việc của tôi đã hoàn thành các deliverable cấu trúc của CP4: bộ eval có đúng 10 case theo tỷ lệ 5+5; có case cho cả hai tool mới; run group đo đủ case; B3 liệt kê kết quả và ba lỗi; báo cáo cá nhân ghi MSSV, vai trò, bằng chứng và phần tự đánh giá. Contract test cho registry/schema đạt 65 passed.

Run xác nhận Group Eval hiện đạt 10/10 trên bộ 10 case với prompt hash và tool hash được lưu trong JSON. Kết quả này áp dụng cho đúng biến thể prompt đã chạy; không ghi thành kết quả CP2 v3 vì artifact version khác. GitHub API có giới hạn truy cập theo thời điểm, vì vậy điểm tool-call không thay thế đánh giá chất lượng dữ liệu trả về.

### Bổ sung B7 — Small Local Model Challenge

Tôi chạy base eval 20 case bằng hai model local có tool capability và cùng prompt/tools v3: `qwen2.5:7b` đạt 9/20 (45%), còn `qwen2.5:3b` đạt 7/20 (35%); cả hai run đo đủ 20/20 và có 0 provider error. Llama 3.2 3B có sẵn nhưng hai lần full-run dừng ở R12, nên không được tính là baseline; tôi lưu riêng subset chẩn đoán 5 case (0/5) để ghi nhận giới hạn.

Trên Qwen 7B, v4-local chỉ rút gọn system prompt và đạt 12/20 (60%), tăng 15 điểm phần trăm so với v3 local. v5-local chỉ thay tool descriptions bằng nội dung ngắn, rõ boundary hơn; điểm giảm còn 11/20 (55%), nên giả thuyết đó không được ủng hộ cho local model. Kiểm tra ngược v5-local bằng OpenAI `gpt-4o-mini` đạt 19/20 (95%), bằng cloud v3; cả hai còn cùng lỗi R12. Hash, giả thuyết và run path được ghi trong `starter_v0/artifacts/version_log.csv` và mục B7 của báo cáo nhóm.

Các run bằng chứng: [cloud v3 current-prompt baseline](../../starter_v0/runs/cloud-v3-current_B_base_openai_20261110T225134566688.json), [Qwen 7B](../../starter_v0/runs/v3-qwen7b_B_base_ollama_20261110T220201663566.json), [Qwen 3B](../../starter_v0/runs/v3-qwen3b_B_base_ollama_20261110T223709953462.json), [v4-local](../../starter_v0/runs/v4-local_B_base_ollama_20261110T223020856578.json), [v5-local](../../starter_v0/runs/v5-local_B_base_ollama_20261110T223226373182.json), [cloud-v5 check](../../starter_v0/runs/cloud-v5-local_B_base_openai_20261110T223343542041.json), [Llama diagnostic subset](../../starter_v0/runs/diag-llama3b-subset_B_base_ollama_20261110T222756967652.json). Live chat evidence bổ sung: [transcript OpenAI](../../starter_v0/transcripts/v3-live-evidence_openai_20261110T223250499529.transcript.json).

## 8. Phân tích chi tiết từng case của CP4

### 8.1. Nhóm single-turn G01–G05

**G01 — GitHub repository Python về RAG.**
Case này được thiết kế để kiểm tra một yêu cầu tìm mã nguồn có đủ ba loại ràng buộc: chủ đề RAG, ngôn ngữ Python và tiêu chí xếp hạng theo số sao. Agent cần route sang `github_search`, không được chuyển sang `hf_search`, `papers` hoặc `lookup`, vì người dùng nói rõ “repository trên GitHub”. Ngoài việc chọn đúng tool, expected còn kiểm tra `language="python"`, `sort="stars"` và `max_results=3`. Đây là một case quan trọng vì nó đo khả năng tách thông tin tự nhiên trong câu hỏi thành các tham số có cấu trúc.

**G02 — Dataset tiếng Việt trên Hugging Face.**
Case này kiểm tra sự khác nhau giữa tài nguyên trên Hugging Face và repository trên GitHub. Agent phải chọn `hf_search`, đặt `kind="datasets"`, sắp xếp theo `lastModified` và giới hạn kết quả ở 4. Tôi cố ý đưa cụm “bộ dữ liệu tiếng Việt để đánh giá RAG” vào câu hỏi nhưng không ép expected phải chứa một câu query duy nhất. Điều cần đo là loại tài nguyên, sort và giới hạn kết quả; việc này giúp tránh đánh giá quá cứng một cách diễn đạt truy vấn mà harness không có khả năng đo ngữ nghĩa đầy đủ.

**G03 — Phân biệt repository với paper và web.**
G03 kiểm tra boundary giữa ba nguồn thường dễ bị nhầm: repository mã nguồn, paper khoa học và kết quả web tổng quát. Người dùng nói rõ “repository trên GitHub” và “đừng tìm bài báo khoa học”, vì vậy expected yêu cầu `github_search`. Tôi để arguments là tập rỗng trong expected để tập trung vào routing, bởi có nhiều query hợp lệ cho “benchmark đánh giá LLM agent”. Cách thiết kế này thể hiện nguyên tắc không nên biến một yêu cầu kiểm tra tool thành một bài chấm văn bản query duy nhất.

**G04 — Thiếu chủ đề tìm repository.**
G04 là case missing-information. Người dùng yêu cầu một repository phù hợp nhưng không nói repository về chủ đề nào. Nếu agent tự điền một chủ đề như `AI research` hoặc `RAG`, nó sẽ tạo ra một kết quả có vẻ hợp lệ nhưng không có cơ sở từ yêu cầu. Vì vậy expected là gọi `clarify` với `response_type="text"`. Case này kiểm tra nguyên tắc hỏi lại khi thiếu thông tin thiết yếu, đồng thời phân biệt câu hỏi xin thông tin nội dung với câu hỏi xác nhận yes/no.

**G05 — Policy và rate limit.**
G05 không phải là yêu cầu tìm kiếm bên ngoài. Người dùng hỏi về cách ghi nhận lỗi rate limit của API nghiên cứu miễn phí trong run JSON, nên agent phải route đến `policy` với vùng `tool_usage`. Lỗi ban đầu là agent chọn đúng `policy` nhưng dùng `policy_area="all"`, một giá trị quá rộng so với chủ đề. Tôi đưa lỗi này vào phân tích vì nó cho thấy routing đúng chưa đủ; argument cũng phải phản ánh phạm vi nhỏ nhất phù hợp.

### 8.2. Nhóm multi-turn G06–G10

**G06 — Giữ chủ đề và bổ sung tham số.**
Ở lượt đầu người dùng chỉ nêu chủ đề “vector database”. Lượt sau mới cung cấp Python, sort theo stars và giới hạn 4. Agent cần kết hợp context cũ với constraints mới nhất, tạo một lần gọi `github_search` có query, language, sort và max_results đầy đủ. Case này không chỉ kiểm tra memory của hội thoại mà còn kiểm tra khả năng không làm mất chủ đề cũ khi lượt sau chỉ bổ sung bộ lọc.

**G07 — Sửa sort và limit.**
G07 bắt đầu bằng yêu cầu repository Python về agent framework, sort theo stars. Ở lượt sau người dùng sửa sort thành `updated` và giới hạn còn 2. Expected yêu cầu giữ query/language cũ nhưng thay sort/limit mới. Đây là tình huống correction rất phổ biến trong chat thực tế: agent phải ưu tiên sửa đổi gần nhất, không ghép cả giá trị cũ và mới, cũng không coi câu trả lời trung gian của assistant là một yêu cầu mới.

**G08 — Chuyển từ GitHub sang paper.**
G08 kiểm tra sự chuyển đổi nguồn trong multi-turn. Lượt đầu yêu cầu repository GitHub, nhưng lượt sau nói “thay vào đó” và yêu cầu paper arXiv về RAG evaluation. Agent phải bỏ route `github_search` và chuyển sang `papers` với query mới. Tôi đưa case này vào để kiểm tra liệu agent có bám cứng tool đã chọn ở lượt trước hay có hiểu rằng người dùng đã thay đổi mục tiêu.

**G09 — Sửa ngôn ngữ và bổ sung ý nghĩa query.**
G09 là case có lỗi rõ nhất ở lần chạy đầu. Người dùng đổi JavaScript thành Python và bổ sung ý “scraping bất đồng bộ”. Agent ban đầu chỉ giữ `web scraping`, làm mất khái niệm mới. Sau khi cập nhật prompt, expected được đáp ứng bằng query `asynchronous web scraping` và `language="python"`. Bài học ở đây là khi người dùng nói “sửa thành” hoặc “tập trung vào”, agent cần tạo lại query từ toàn bộ trạng thái mới, không chỉ thay một argument riêng lẻ.

**G10 — Gọi đồng thời GitHub và Hugging Face.**
G10 kiểm tra yêu cầu kết hợp. Người dùng muốn giữ yêu cầu tìm repository GitHub từ lượt trước và thêm model embedding tiếng Việt trên Hugging Face. Agent phải gọi cả `github_search` và `hf_search`; chỉ gọi tool thứ hai sẽ làm mất yêu cầu trước đó. Case này có giá trị vì nhiều agent có xu hướng chỉ xử lý mệnh đề cuối cùng trong lượt mới nhất. Expected của GitHub để trống arguments nhằm tập trung vào việc không bỏ tool cũ, trong khi Hugging Face vẫn kiểm tra `kind="models"`.

## 9. Quy trình kiểm soát tính hợp lệ của kết quả

Tôi không xem một run có điểm cao là hợp lệ ngay lập tức. Mỗi run được kiểm tra theo nhiều lớp. Trước hết, tôi xác nhận file JSON có đúng `suite="group"`, đúng `provider`, đúng model và đúng dataset. Sau đó kiểm tra `summary.total_cases`, `summary.measured_cases` và `summary.provider_error_cases`. Điều kiện tối thiểu để dùng làm evidence là 10 case được đo đủ và không có provider error. Nếu một case thất bại vì API bên ngoài trả 403, tôi không tự động kết luận agent chọn sai tool; tôi tách lỗi dịch vụ khỏi lỗi routing/argument trong kết quả.

Lớp kiểm tra thứ hai là đối chiếu expected và actual cho từng case. Tôi đọc tên tool, số lần gọi, arguments và trạng thái failure trong `results`. Với G04, tôi kiểm tra riêng `response_type`; với G05, kiểm tra `policy_area`; với G09, kiểm tra cả query và language. Lớp thứ ba là kiểm tra artifact identity: prompt hash, tools hash và `artifact_version` trong run phải khớp với biến thể được mô tả trong báo cáo. Vì vậy, run `group-quoc-main-sync` được ghi là run xác nhận đã đồng bộ, chứ không gán nhầm thành mốc v3 của CP2.

Tôi cũng phân biệt ba loại bằng chứng:

1. **Evidence cấu trúc:** `eval_group.json` chứng minh số lượng, ID, phase, turns/query, expect và metadata.
2. **Evidence thực nghiệm:** run JSON chứng minh model đã được chạy thật, số case đo được và actual tool calls.
3. **Evidence giải thích:** report, version log và bảng phân tích lỗi giải thích vì sao thay đổi được thực hiện và giới hạn của metric.

Ba lớp này bổ trợ cho nhau. Chỉ có dataset thì chưa chứng minh agent đã chạy; chỉ có run điểm cao thì chưa chứng minh case được thiết kế đúng rubric; chỉ có mô tả bằng văn bản thì không đủ để tái kiểm tra. Đây là lý do tôi giữ các run baseline, intermediate và confirmation riêng thay vì ghi đè lên cùng một file.

## 10. Các quyết định thiết kế và lý do không xóa run trung gian

Trong quá trình thực hiện có nhiều file run mang tên gần giống nhau. Tôi đã kiểm tra chúng theo tác giả, timestamp, suite, metric và hash. Các run `group-quoc-clarify`, `group-quoc-clarify-v2` và `group-quoc-main-sync` không phải bản sao vô nghĩa: chúng lần lượt thể hiện lần thử prompt đầu, lần xác nhận đạt 10/10 và lần chạy lại sau khi đồng bộ artifacts. Giữ các file này giúp người chấm thấy điểm 7/10 không bị che giấu và điểm 10/10 là kết quả của một thay đổi có kiểm soát.

Tương tự, các run v0–v3 của các thành viên khác có timestamp và artifact hash khác nhau. Dù tên version lặp lại, chúng thuộc những series đánh giá khác nhau. Tôi không xóa run của thành viên khác và không nhận các run đó là evidence cá nhân của mình. Trong phạm vi cá nhân, các run được dùng trực tiếp gồm baseline group, các vòng prompt của CP4 và chuỗi local-model B7. Cách phân loại này bảo vệ tính trung thực của báo cáo và tránh làm mất dữ liệu có thể cần cho việc đối chiếu sau này.

## 11. B7 — phân tích bổ sung về thử thách local model

B7 được thực hiện sau CP4 như một phần bonus, nhưng có liên hệ trực tiếp với công việc evaluation vì tôi phải giữ nguyên dataset base, ghi lại artifact hash và phân tích lỗi theo cùng một cách. Hai baseline local đầy đủ là Qwen2.5 7B và Qwen2.5 3B chạy qua Ollama. Qwen 7B đạt 9/20, tương đương 45%; Qwen 3B đạt 7/20, tương đương 35%. Cả hai đều đo đủ 20 case và có 0 provider error, nên có thể dùng để so sánh trong cùng bộ base.

Vòng v4-local chỉ thay system prompt bằng bản rút gọn dành cho local model. Qwen 7B tăng từ 45% lên 60%, routing tăng từ 60% lên 75%, argument tăng từ 45% lên 60% và multi-turn tăng từ 33.33% lên 83.33%. Kết quả này ủng hộ giả thuyết rằng model nhỏ có thể hưởng lợi từ prompt ngắn, ít nhiễu và nêu trực tiếp các boundary quan trọng.

Vòng v5-local chỉ thay tool descriptions bằng bản ngắn hơn. Kết quả giảm còn 55%, routing 70%, argument 55% và multi-turn 66.67%. Vì vậy, giả thuyết “description ngắn hơn sẽ tiếp tục cải thiện Qwen 7B” không được ủng hộ trong thực nghiệm này. Tôi không cố giải thích quá mức rằng description ngắn luôn xấu; kết luận đúng hơn là thay đổi cụ thể này, trên model và bộ case này, không cải thiện.

Để tách ảnh hưởng của model khỏi ảnh hưởng của artifact, tôi chạy ngược prompt/tools v5-local trên OpenAI. Cloud đạt 19/20, 95%, routing 100%, argument 95% và multi-turn 100%, bằng cloud-v3-current. Điều này cho thấy phần giảm điểm ở v5-local không thể đơn giản kết luận là tool schema không hợp lệ; nó có thể liên quan đến khả năng diễn giải instruction của Qwen 7B. Tuy vậy, đây vẫn là một thí nghiệm giới hạn vì chỉ dùng một model cloud và một bộ base.

Llama 3.2 3B được thử hai lần full-run nhưng đều dừng tại R12 khi Ollama không trả kết quả. Tôi không dùng một run chưa hoàn tất làm baseline và không dùng subset 5 case để thay thế full eval. Subset chẩn đoán được lưu riêng với nhãn `diagnostic_only`, đạt 0/5 và chỉ dùng để ghi nhận giới hạn runtime/routing. Đây là quyết định quan trọng về tính trung thực của evaluation: thiếu dữ liệu không được biến thành một metric đầy đủ.

## 12. Hạn chế của phép đo và các rủi ro còn lại

Group Eval chủ yếu đo quyết định tool-call, không đo toàn diện chất lượng câu trả lời cuối. Một tool có thể được chọn đúng nhưng API trả dữ liệu rỗng, bị rate limit hoặc trả kết quả không phù hợp. Ngược lại, query có thể khác expected về câu chữ nhưng vẫn có ý nghĩa tương đương; harness hiện không đánh giá semantic equivalence đầy đủ. Vì vậy, điểm 10/10 của Group Eval nên được diễn giải là agent đáp ứng các expected tool-call/argument đã định nghĩa, không phải bằng chứng rằng mọi kết quả nghiên cứu đều chính xác.

Rủi ro thứ hai là phụ thuộc provider và thời điểm chạy. GitHub Search có thể trả HTTP 403, RapidAPI có thể bị giới hạn, Ollama có thể treo tại một lượt cụ thể. Tôi ghi rõ provider error và failure pattern thay vì gộp tất cả thành một accuracy duy nhất. Rủi ro thứ ba là confirmation boundary: R12 trong base vẫn là case cần review thủ công. B7 cũng cho thấy model nhỏ dễ bỏ qua `clarify` hoặc gọi nhầm `timeline` và `social_search`.

Hướng cải thiện hợp lý là bổ sung evaluator cho nội dung nguồn, thêm test riêng cho thứ tự hỏi xác nhận trước `send`, và chạy lặp nhiều lần để đo độ ổn định thay vì chỉ một lần. Với local model, cần khảo sát thêm prompt vừa đủ chi tiết thay vì chỉ rút gọn tối đa; đồng thời cần timeout và cơ chế dừng rõ ràng để một case không làm mất cả full run.

## 13. Phối hợp với các thành viên và ranh giới đóng góp

Phần CP4 của tôi phụ thuộc vào hai tool mới `github_search` và `hf_search`, nhưng tôi không nhận mình là tác giả implementation của chúng. Phần Tool Development cung cấp logic Python, `TOOL.md`, registry và unit test; tôi sử dụng contract của hai tool để thiết kế expected trong eval. Ranh giới này được giữ rõ để tránh ghi nhầm công sức hoặc quy lỗi implementation cho evaluation.

Tôi phối hợp với phần Prompt Engineering bằng cách cung cấp lỗi cụ thể từ G04, G05 và G09. Thay vì đề xuất sửa prompt theo cảm tính, tôi đưa ra failure có thể tái hiện, argument bị thiếu/sai và kết quả sau mỗi giả thuyết. Sau khi prompt được cập nhật, tôi chạy lại cùng dataset và giữ nguyên expected. Cách phối hợp này giúp thay đổi prompt có mục tiêu và cho phép người khác kiểm tra lại.

Tôi cũng cập nhật B3, phần đóng góp trong `TEAM.md`, version log và báo cáo cá nhân để các artifact có cùng câu chuyện evidence. Khi ghi B7, tôi tách rõ phần bonus local model khỏi kết quả CP2 v0–v3 và không dùng diagnostic subset làm full baseline. Việc ghi rõ ranh giới giúp báo cáo cá nhân dài hơn nhưng vẫn có thể kiểm toán: mỗi tuyên bố quan trọng đều gắn với một file hoặc một run cụ thể.

## 14. Tự đánh giá chi tiết

Điểm mạnh lớn nhất của phần việc là biến yêu cầu rubric thành một bộ đo có cấu trúc, sau đó dùng lỗi thực tế để điều chỉnh prompt. Tôi không chỉ tạo đủ 10 case mà còn chia chúng theo các năng lực khác nhau: routing nguồn, mapping argument, clarification, policy boundary, carry-over, correction và multi-tool orchestration. Tôi cũng giữ lại run thất bại và run trung gian để thể hiện quá trình cải tiến, thay vì chỉ trình bày kết quả cuối.

Điểm còn hạn chế là bộ Group Eval vẫn chủ yếu đánh giá một lần chạy với một model cloud và chưa có chấm semantic cho nội dung trả về. Một số case có thể được mở rộng bằng expected về provenance, chất lượng tóm tắt hoặc xử lý API error. B7 cũng mới khảo sát Qwen 7B/3B và một subset Llama 3B; chưa đủ để kết luận chung cho mọi local model. Những hạn chế này được ghi vào báo cáo thay vì che giấu.

Nếu có vòng tiếp theo, tôi sẽ ưu tiên ba việc. Thứ nhất, tạo một regression suite nhỏ chỉ cho confirmation boundary, trong đó tách rõ “xin nội dung”, “xin ý định gửi” và “xác nhận nội dung/đích gửi”. Thứ hai, bổ sung evaluator kiểm tra source URL, trạng thái lỗi và độ đầy đủ của kết quả tool. Thứ ba, chạy repeated trials cho các case khó như G04, G09, G10, R10, R11 và R12 để đo variance của model. Mục tiêu không chỉ là đạt một điểm cao mà là làm cho hành vi ổn định và có thể giải thích.

## 15. Kết luận cá nhân

CP4/C5 đã hoàn thành đầy đủ về mặt cấu trúc và evidence: đúng 10 case với tỷ lệ 5 single-turn/5 multi-turn, có metadata, có route tới hai tool mới, có baseline, có các vòng cải thiện, có run xác nhận 10/10 và có phân tích giới hạn. B7 bổ sung thêm một chuỗi đánh giá local model với hai baseline full, hai vòng tối ưu, cloud reverse-check và chẩn đoán Llama được gắn nhãn đúng.

Đóng góp của tôi không chỉ là tạo một file eval mà là xây dựng một chu trình evaluation có thể kiểm tra: đọc rubric và harness, thiết kế case, chạy baseline, phân tích actual calls, đưa ra giả thuyết, thay đổi có kiểm soát, chạy xác nhận, lưu hash/run và ghi nhận hạn chế. Kết quả cuối cùng được trình bày theo hướng có thể tái lập và phân biệt rõ giữa số liệu thực tế, diễn giải hợp lý và những điều chưa thể kết luận.
