# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1_b | 1 dương + 1 âm + box | Không gồm người lái (áo xanh vẫn nguyên màu); chưa có hàng hoá để xét | Mask 270.591 px (38,7% ảnh; khoảng 51% box), 1 polygon. Lẹm sang đầu xe máy bên phải (chắn bùn, bánh trước); viền bánh trước dưới cùng lởm chởm; vài lỗ nhỏ quanh chân người lái | `tuktuk_img013_sam21_box.jpg` |
| img_013 | MobileSAM | 1 dương + 1 âm + box | **Có** gồm người lái và cả xe máy bên phải; dính thêm một phần người ngồi phía sau | Mask 324.620 px (46,5% ảnh; khoảng 61% box), 1 polygon: lớn hơn SAM 2.1 khoảng 20% nhưng phần lớn là phần thừa; viền bánh trước lởm chởm | `tuktuk_img013_mobile_box.jpg` |
| img_004 | SAM 2.1_b | 1 dương + box | Xe thứ hai bị xe xanh phía trước che một phần | Mask 11.432 px (1,6% ảnh; khoảng 66% box), 1 polygon, phủ gần hết phần xe nhìn thấy trong box; còn một vệt trắng nhỏ không được phủ ở mép trái | `tuktuk_img004_sam21_box.jpg` |
| img_017 | SAM 2.1_b | 1 dương | Không thấy người lái trong ảnh (bị mái che) | Mask 85.882 px (12,3% ảnh), 1 polygon, phủ cả mái, thân, bánh của xe bên phải và không tràn sang xe xanh bên trái; có vài lỗ nhỏ trong cabin | `tuktuk_img017_sam21.jpg` |

**Các lần chạy hỏng (giữ lại làm bằng chứng):**
- `img_013`, không box: click dương `520,330` rơi lên một vật nhỏ màu hồng (túi) trên xe, nên mask chỉ có 4.003 px (SAM 2.1_b) và 4.600 px (MobileSAM), dưới 1% ảnh. Click âm `130,300` nằm sát mép trái phía sau xe chứ **không** nằm trên người lái, nên chấm đỏ chưa làm đúng việc tách người lái. Files: `tuktuk_img013_sam21.jpg`, `tuktuk_img013_mobile.jpg`.
- `img_004`, một click `229,325`: mask 134 px (0,0% ảnh), điểm rơi vào chi tiết nhỏ của xe bị che. Đổi click sang `200,300` cho 7.348 px nhưng tách thành 2 polygon; chỉ thêm `--box` mới ra mask liền một mảng. Files: `tuktuk_img004_sam21.jpg`, `tuktuk_img004_sam21_b.jpg`.
- Bài học: với vật lớn, một click đơn không đủ; cần thêm box. Chất lượng mask phụ thuộc vị trí click nhiều hơn phụ thuộc phiên bản SAM.

**Tốc độ (CPU, gồm thời gian nạp model, đo bằng `Measure-Command`):** SAM 2.1_b khoảng 7,6 đến 10,9 giây/lần; MobileSAM khoảng 4,7 đến 5,3 giây/lần (nhanh hơn khoảng 1,6 lần); con số dao động giữa các lần chạy.

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42) trong 24 ảnh batch, 88 đối tượng (img_003, 006, 014, 031, 035; nguồn `qc/qc_sample.csv`); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt). Phiếu `qc_review.csv` được soạn nháp cùng Claude từ ảnh có in id, rồi tôi đối chiếu lại.
- Lỗi theo **đối tượng**: 5/88 = 5,7% · theo **ảnh**: 3/5 = 60,0% (ngưỡng Release ≤ 2%)
- Top lỗi: `thua` 3 lỗi (img_003: #20 three_wheeler conf 0.20 và #10 truck conf 0.25; img_006: #68 car conf 0.55) và `sai_vi_tri_kich_thuoc` 2 lỗi (img_006: một box motorbike bao nhiều xe máy đỗ sát nhau; img_031: #367 person, box dừng ở eo, thiếu phần chân). Các loại `bo_sot`, `sai_lop`, `sai_thuoc_tinh`, `sai_mask`: 0 lỗi ghi nhận, nhưng `bo_sot` chưa được kiểm kỹ nên con số 0 không có nghĩa là không có.
- **Giới hạn mẫu nhỏ** (`qc_report_auto.md`): 3 ảnh lỗi trong 5 ảnh, cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là 88%. Với mẫu nhỏ như vậy không thể chứng minh ngưỡng 2%; chiều ngược lại, 3/5 vẫn đủ để kết luận lô chưa đạt.
- Script kiểm được / không kiểm được: script chỉ đếm theo phiếu kiểm bằng mắt, tính tỉ lệ và khoảng Wilson. Nó không tự biết box có ôm sát không, cũng không phát hiện vật mà cả máy và tôi cùng bỏ sót; ở các ảnh đông (img_006, img_035) nhiều box chồng nhau nên tôi chưa kiểm hết từng vật.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đủ rõ để quyết các ca đã gặp và dữ liệu không hỏng; Rework vì ảnh lỗi 3/5 = 60% > 2%, lỗi `thua` lặp lại (3/5 lỗi, quy tắc cho box conf thấp chưa đủ chặt), và bắt bẫy mù ở vòng duyệt đầu chỉ 6/12 = 50% < 80%. Chưa thể Release.
Nếu Rework / Hold, việc cần làm tiếp: sửa các box lỗi ở img_003, img_006, img_031 trong CVAT rồi export `labels/qc_fixed.json`, chạy lại `build_release.py`; thêm quy tắc bỏ box `three_wheeler`/`truck`/`car` có conf thấp khi không thấy vật rõ vào guideline v1.1; chốt câu mở 2 (xe máy đỗ dày: từng xe hay `ignore_region`); lấy mẫu QC mới (seed khác, có ca `can_review`) sau khi sửa.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
