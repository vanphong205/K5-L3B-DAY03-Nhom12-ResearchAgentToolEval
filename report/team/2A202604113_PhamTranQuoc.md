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

| Chỉ số | Kết quả |
|---|---:|
| Tổng case / case được đo | 10 / 10 |
| Case đạt | 7/10 (70%) |
| Tool routing accuracy | 90% |
| Argument accuracy | 70% |
| Multi-turn accuracy | 80% (4/5) |
| Provider error | 0 |
| Single-turn đạt | 3/5 |
| Multi-turn đạt | 4/5 |

Ba case chưa đạt cho thấy ba dạng lỗi khác nhau:

| Case | Expected | Actual trong run | Phân tích |
|---|---|---|---|
| G04 — thiếu chủ đề | Hỏi lại bằng `clarify` | Gọi `github_search` với query tự đoán `AI research` | Agent tiếp tục theo hướng dẫn đoán khi thiếu thông tin trong prompt hiện hành. Đây là lỗi boundary/missing-info; hướng cải thiện là hỏi chủ đề trước khi tìm. |
| G05 — policy về rate limit | `policy_area=tool_usage` | Chọn `policy` nhưng đặt `policy_area=all` | Routing đúng, nhưng arguments quá rộng. Nên mô tả rõ trong `tools.yaml` cách ánh xạ rate limit sang vùng `tool_usage`. |
| G09 — sửa query multi-turn | Query chứa ý “asynchronous web scraping”, language Python | Gọi đúng `github_search` và đặt Python, nhưng query chỉ còn `web scraping` | Agent tiếp nhận phần đổi ngôn ngữ nhưng bỏ mất điều kiện nội dung mới ở lượt sau. Cần giữ đầy đủ các ràng buộc khi viết lại query. |

Run cũng ghi nhận lỗi runtime của GitHub Search API: G01, G03 và G04 trả HTTP 403 do rate limit. G02 gọi đúng `hf_search` và đúng các arguments được chấm, nhưng API trả danh sách rỗng. Vì vậy PASS/FAIL trong summary xác nhận hành vi tool-call theo harness; các trường hợp trên không xác nhận rằng API đã cung cấp dữ liệu hữu ích.

## 5. Kiểm tra cấu hình và giới hạn khi ghi kết quả

Run lưu `prompt_hash=eb1c8179815bd79d34de7d326420bb99b3072e6e8ae96464c02d4411f905fc68` và `tools_hash=ee32b4055306d7588dc158083f6d07c069ef4f766b0e57a8aeeb3fa8c3403b41`; `artifact_version` là `v3+peb1c8179815b+tee32b4055306`. Nhãn `version` trong run được đặt là `v3`, nhưng khi đối chiếu với dòng v3 trong `starter_v0/artifacts/version_log.csv`, hash prompt/tool của run group này không khớp: version log ghi prompt `c800280bc6ca…` và tools `0f6be0ee4fef…`. Do đó, báo cáo này xem đây là run Group Eval được gọi với nhãn v3 và cấu hình/hashes nêu trên; không dùng nó làm bằng chứng rằng đúng artifacts v3 của CP2 đã được kiểm tra. Cần chạy lại Group Eval khi đúng file prompt và tool v3 được đặt trong checkout để xác nhận ảnh hưởng của v3.

Điểm 70% của bộ group cũng không được so trực tiếp với điểm base: hai run dùng bộ case khác nhau. Group Eval phục vụ đo các case nhóm tự thiết kế, trong khi run base dùng 20 case cố định. Kết quả hữu ích nhất ở đây là các lỗi cụ thể và số đo trong cùng một bộ group khi chạy lại với đúng artifacts.

## 6. Bằng chứng và đóng góp vào repo

| Hạng mục | Bằng chứng |
|---|---|
| Bộ case | [eval_group.json](../../starter_v0/data/eval_group.json) |
| Run Group Eval | [v3_B_group_openai_20261110T192353230210.json](../../starter_v0/runs/v3_B_group_openai_20261110T192353230210.json) |
| Kết quả B3 và phân tích lỗi | [group_report.md](group_report.md) |
| Kiểm tra registry/schema | `python -m pytest tests/test_tool_contract.py -q` — 65 passed sau khi tích hợp `hf_search` |
| Commit cá nhân trên nhánh nộp | `d067aa9`, `6cb1a4d`, `c1047e4`, `c07d09c`; các thay đổi được merge vào `main` |

Các file trong phạm vi đóng góp của tôi gồm `starter_v0/data/eval_group.json`, run JSON của Group Eval, mục B3 của báo cáo nhóm, `TEAM.md` và báo cáo cá nhân này. Tool implementations của `github_search` và `hf_search` do phần việc Tool Development cung cấp; tôi sử dụng chúng làm đối tượng kiểm tra và không nhận là tác giả implementation.

**Công cụ hỗ trợ và tự kiểm tra:** Tôi dùng Codex để rà yêu cầu rubric, hỗ trợ biên tập case và tổng hợp phần mô tả. Số liệu được lấy từ `run_eval.py` và run JSON do provider thật tạo ra. Tôi đối chiếu số lượng 5+5, ID case, summary, expected/actual tool calls, lỗi API và hash artifacts. Codex không được dùng để tạo run hay metric thay cho kết quả thực tế.

## 7. Tự đánh giá và việc tiếp theo

Phần việc của tôi đã hoàn thành các deliverable cấu trúc của CP4: bộ eval có đúng 10 case theo tỷ lệ 5+5; có case cho cả hai tool mới; run group đo đủ case; B3 liệt kê kết quả và ba lỗi; báo cáo cá nhân ghi MSSV, vai trò, bằng chứng và phần tự đánh giá. Contract test cho registry/schema đạt 65 passed.

Điểm còn cần làm để có đánh giá cuối đáng tin cậy hơn là chạy lại bộ group bằng đúng artifacts v3 đã kiểm chứng qua hash, sau khi chúng được tích hợp vào checkout/nhánh nộp. Nên chạy khi GitHub Search API hết rate limit để quan sát dữ liệu tool thực tế. Các giả thuyết tiếp theo là: hỏi lại khi chủ đề tìm kiếm bị thiếu; mô tả cụ thể `policy_area=tool_usage`; và giữ mọi thuộc tính mới khi sửa query multi-turn. Khi có run mới, cần cập nhật bảng B3 và thay đường dẫn/hash trong báo cáo này.
