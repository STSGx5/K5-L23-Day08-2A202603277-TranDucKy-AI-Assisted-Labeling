# Label contract — Phố châu Á — BỔ SUNG v1.1 — <<2A202603277- Trần Đức Kỳ>>

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Xe còn thấy rõ từ 20% trở lên: vẽ từng xe. Không tách được từng xe: một `ignore_region` bao cả cụm. Không dùng một box `motorbike` cho nhiều xe | img_006 | CAL-03 |
| 5 | Kính chắn gió thuộc mask xe không? | Có, kính thuộc mask xe. Người lái và hành khách nhìn qua kính hoặc khung xe là `person` riêng, không thuộc mask xe; khi dùng SAM đặt điểm âm trên người lái, vật lớn thì thêm box | img_013, img_017 | CAL-04 |

Chưa chốt ở v1.1: câu 1, 3, 4, 6, 7.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_006 | Box conf thấp mà không thấy vật rõ | YOLOE giữ `three_wheeler` 0.20; YOLO26m giữ `truck` 0.25 và `car` 0.55 | Box máy `three_wheeler`, `truck`, `car` có conf dưới 0.6 chỉ giữ khi nhìn rõ thân xe hoặc đếm được bánh; không thì xóa (CAL-01) | Mục 3 (conf) |
| img_013, img_018, img_005 | Vật không phải xe ba bánh bị gọi `three_wheeler` | YOLOE gán cho bus, truck, tủ điện | `three_wheeler` chỉ khi đếm được 3 bánh hoặc đúng dạng tuk-tuk, xích lô, ba gác; xe có buồng lái 4 bánh trở lên là `truck` hoặc `bus`; tủ điện không phải xe, xóa không vẽ ignore (CAL-02) | Mục 2 (cột Không gồm) và 4.3 |
| img_031 | Người bán hàng sau xe đạp chở hàng, bị che từng đoạn | YOLO26m ra box `person` dừng ở eo | Box `person` kéo tới hết phần thân dưới còn nhìn thấy (CAL-05) | Mục 4.1.2, 4.1.4 |
| img_017, img_009 | Box `three_wheeler` bị lỏng lọt qua lượt duyệt đầu | Không phải lỗi máy: bẫy | So từng cạnh box với điểm ngoài cùng của xe, sai quá 2 px thì kéo lại; quét ảnh trước khi nhìn nhãn máy (CAL-06) | Mục 4.1.1 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu mở 2 và 5; thêm điều kiện giữ box conf thấp, quy tắc `three_wheeler` so với xe khác, box `person` bị che từng đoạn, nhắc quy trình kiểm box lỏng | CAL-01 đến CAL-06 |
