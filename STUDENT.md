# Thông tin học viên

- Họ và tên: Trần Đức Kỳ
- MSHV: 2A202603277
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: [STSGx5](https://github.com/STSGx5)
- Repo bài nộp (URL): [Repo](https://github.com/STSGx5/K5-L23-Day08-2A202603277-TranDucKy-AI-Assisted-Labeling.git)
- Commit chốt bài (hash): [CẦN BẠN ĐIỀN sau khi commit: `git rev-parse --short HEAD`]
- Máy: Windows, không có GPU (chạy CPU) [CẦN BẠN XÁC NHẬN]
- CVAT (Docker): [CẦN BẠN ĐIỀN: phiên bản, ví dụ v2.81.0]

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 390 / 7 / 29 (mẫu số: 434 box máy); added: 15 (mẫu số: 412 đối tượng)
- So với gold: precision / recall của **máy**: 40,0% (24/60) / 96,0% (24/25) (mẫu số: 25 đối tượng gold)
- Bắt bẫy: 11/12 = 91,7% (lọt: img_017 loose; vòng duyệt đầu 6/12)
- Thời gian: gán tay [CẦN BẠN ĐIỀN] phút/ảnh; duyệt có pre-label ≈ 10,2 phút/ảnh (244 phút / 24 ảnh, từ `time_log.csv`)
- Lỗi QC: 60,0% theo ảnh (3/5), 5,7% theo đối tượng (5/88)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 3/5 = 60% (> 2%) và lỗi `thua` lặp lại ở box conf thấp
