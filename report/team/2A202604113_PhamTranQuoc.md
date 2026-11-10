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
