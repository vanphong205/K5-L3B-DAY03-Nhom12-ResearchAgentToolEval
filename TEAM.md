# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3A / L3B
- Repo nhóm: `K5-L3A-DAY03-TenNhom-ResearchAgentToolEval` hoặc `K5-L3B-DAY03-TenNhom-ResearchAgentToolEval` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Trấn Quốc | 2A202604113 | chưa cung cấp | chưa cung cấp | Evaluation |
|   2 |        |      |       |        |         |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## Phạm Trấn Quốc — 2A202604113

### Phần việc phụ trách

- Group Evaluation (CP4): thiết kế 10 case, chạy eval và phân tích lỗi.

### Đóng góp cụ thể

- Viết `starter_v0/data/eval_group.json` (5 single turn, 5 multi turn), đo lựa chọn hai tool mới `github_search`, `hf_search` và các tình huống phân biệt với `papers`/`policy`.
- Chạy Group Eval với OpenAI `gpt-4o-mini`, phân tích ba case chưa đạt và giới hạn do GitHub API rate limit.
- Điền mục B3 của `report/team/group_report.md` và viết báo cáo cá nhân.

### Bằng chứng

- `starter_v0/runs/v0-group-quoc-two-tools_B_group_openai_20261110T183742208256.json`
- `starter_v0/analysis/group_eval_v0.md`
- `report/team/2A202604113_PhamTranQuoc.md`
