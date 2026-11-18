# Báo cáo Day 08 — 2A202603277 - Trần Đức Kỳ

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person 0.5; motorbike 0.35; bicycle, car, bus, truck dùng mặc định 0.3 (từ `qc/threshold_sweep.md`) | r03/r04: conf 0.5 chung làm mất khoảng 35% box (215 xuống 139) nên chọn ngưỡng theo lớp; r07: chạy conf 0.05 trên 36 ảnh để quét ngưỡng; person và motorbike đạt recall 100% trên gold. car tối đa 83% (6 vật), bicycle/truck 0 vật và bus 1 vật trong gold nên không chọn được ngưỡng |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | r05/r08: YOLO26 không có lớp này; YOLOE nhanh nhất (0.32 giây/ảnh), recall 100% và precision 78% trên 7 vật gold ở ngưỡng 0.15 |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (YOLOE đứng trước để `three_wheeler` thắng `car`/`truck` khi IoU ≥ 0.6; ngưỡng lấy từ `threshold_sweep.md`; [CẦN BẠN ĐỐI CHIẾU với lệnh thật trong lịch sử terminal]). Sau đó chạy `inject_traps.py` với MSHV 2A202603277.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1_b (`sam2.1_b.pt`). Với cùng prompt trên img_013, SAM 2.1_b tách được người lái ra khỏi mask xe (270.591 px) còn MobileSAM gộp cả người lái và xe máy bên cạnh (324.620 px, thừa khoảng 20%). Đổi lại SAM 2.1_b chậm hơn MobileSAM khoảng 1,6 lần (8,38 so với 5,35 giây/ảnh trên CPU, gồm nạp model), chấp nhận được vì chỉ chạy 3 vật (r09, r10). Chạy được trên CPU, không cần quyền truy cập đặc biệt.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLOE gọi bus, truck và tủ điện là `three_wheeler` (img_013, img_018 bus; img_003, img_034 truck; img_005 tủ điện): 19/38 box `three_wheeler` bị xoá, và ngưỡng 0.15 còn làm tệ thêm vì phải thấp mới đủ recall. Sửa bằng quy tắc duyệt tay: chỉ giữ khi đếm được 3 bánh hoặc đúng dạng tuk-tuk, xích lô, ba gác (CAL-02), và đặt YOLOE trước YOLO26 khi merge.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_006 box `motorbike` conf 0.70 bao nhiều xe đỗ sát nhau; img_031 box `person` #367 conf 0.79 dừng ở eo, thiếu phần chân; img_006 `car` 0.55 không thấy ô tô. Điểm thấp mà đúng: `three_wheeler` phải hạ ngưỡng xuống 0.15 mới đạt recall 100% trên gold, tức có vật thật chỉ có conf thấp [CẦN BẠN GHI thêm ảnh cụ thể nếu nhớ].
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: gold chỉ có 25 vật (person 5, motorbike 6, car 6, three_wheeler 7, bus 1, bicycle 0, truck 0). Precision của máy 40% (KTC 95%: 29–53%) và recall 96% (KTC 80–99%) đều có khoảng tin cậy rộng. Ngưỡng của bicycle, bus, truck không có cơ sở số liệu, car chỉ đạt recall tối đa 83%.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 390 | 89,9% | 434 box máy |
| edited | 7 | 1,6% | 434 box máy |
| deleted | 29 | 6,7% | 434 box máy |
| added | 15 | 3,6% | 412 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: trong 38 box máy có 14 giữ nguyên, 5 sửa, 19 xoá (50%), do nhầm bus, truck, tủ điện (img_013, img_018, img_003, img_034, img_005). `truck` cũng kém: 1 giữ, 5 xoá, thêm tay 4. Ngược lại `person` (184 giữ / 187 box máy) và `motorbike` (125/125) gần như không cần sửa. Số liệu nằm ở `qc/review_stats.md`; 8 trường hợp trong deleted + added chỉ là đổi lớp, máy sót thật khoảng 7 vật.
- So với gold: precision / recall của **máy** 40,0% (24 TP / 60 box) và 96,0% (24/25) (mẫu số: 25 đối tượng gold trên 4 ảnh). Precision thấp vì ngưỡng được chọn theo recall 90% nên giữ nhiều box thừa; số này lạc quan vì chọn ngưỡng trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91,7% (KTC 95%: 65–99%; `qc/review_stats.md`, số tính trên nhãn sau duyệt); vòng duyệt đầu chỉ được 6/12 = 50% (xem mục 7), sau đó duyệt lại mới lên 11/12. Loại lọt: `loose` ở img_017 (box `three_wheeler` rộng hơn xe nhiều); các bẫy missing, extra, wrong_class đều bắt đủ 3/3. Vì sao: bỏ bước so từng cạnh box với điểm ngoài cùng của xe, và nhìn nhãn máy trước khi quét ảnh nên bỏ qua người thiếu ở img_029, img_022 ở vòng đầu (CAL-06).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 89,9% nhưng vòng đầu bắt bẫy chỉ 50%: tôi có xu hướng tin nhãn máy và lướt nhanh (thiên kiến tự động hóa). Accepted cao chỉ tốt khi bắt bẫy cũng cao; sau khi duyệt lại theo checklist thì bắt bẫy lên 91,7%, và vẫn lọt 1 box lỏng nên chưa thể tin tuyệt đối vào con số 89,9%.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay [CẦN BẠN ĐIỀN: phút/ảnh = tổng phút gold_manual / 4] vs duyệt 244 phút / 24 ảnh ≈ 10,2 phút/ảnh (cộng từ `time_log_detail_by_step.csv`, gồm 5 bước checklist mỗi ảnh). Thiên lệch: gán gold trước nên đã quen cảnh và quen lớp vật, làm duyệt batch nhanh hơn thực tế; ngược lại ảnh batch đông vật hơn (trung bình 18 vật/ảnh) so với gold (khoảng 6 vật/ảnh).
- Kết luận: chỉ cho lớp nào: dùng máy cho `person` và `motorbike` (accepted 184/187 và 125/125, recall gold 100%); `car` dùng có duyệt (57/59 giữ); `three_wheeler` và `truck` cần duyệt kỹ từng box vì 50% và 83% box máy bị xoá (19/38, 5/6). Dựa trên `qc/review_stats.md`.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét cả ảnh tìm vật thiếu **trước** khi nhìn nhãn máy. Kiểm bằng số: bẫy `missing` bắt đủ 3/3.
  2. Box `three_wheeler`, `truck`, `car` có conf dưới 0.6 chỉ giữ khi nhìn rõ thân xe hoặc đếm được bánh (CAL-01, CAL-02). Kiểm bằng số: lỗi `thua` trong lần QC mẫu mới bằng 0.
  3. So từng cạnh box với điểm ngoài cùng của vật, sai quá 2 px thì kéo lại (CAL-06). Kiểm bằng số: bẫy `loose` bắt đủ 3/3 (hiện 2/3).

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

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Giúp: 4.2.1 (bám biên phần nhìn thấy) và 4.2.4 (hai vật chạm nhau không chồng): mask img_013 lẹm sang xe máy bên phải vi phạm 4.2.4; 4.2.2 (lỗ nhỏ dưới 5% không vẽ): các lỗ nhỏ quanh chân người lái và trong cabin img_017 thuộc loại này. Thiếu: quy tắc loại người lái và hàng hoá khỏi mask xe, và cách đặt điểm âm; đã bổ sung ở v1.1 (CAL-04).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không dùng; chỉ làm offline bằng `sam2_points.py` [CẦN BẠN XÁC NHẬN]. Offline cho điều khiển chính xác điểm dương, điểm âm, box và ghi lại lệnh để lặp lại được.

## 6. Calibration (Part 6)

- Số quyết định: 6 (`calibration_decisions.csv`, CAL-01 đến CAL-06); quan trọng nhất: CAL-02 (`three_wheeler` nhầm bus, truck, tủ điện, 19/38 box bị xoá) và CAL-01 (box conf thấp không rõ vật, gây cả 3 lỗi `thua` ở QC mẫu).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu mở 2 (xe máy đỗ dày: vẽ từng xe nếu còn thấy ≥ 20%, nếu không thì một `ignore_region`) và câu mở 5 (kính thuộc mask xe, người lái là `person` riêng); thêm điều kiện giữ box conf thấp, quy tắc đếm bánh cho `three_wheeler`, box `person` bị che từng đoạn, và cách kiểm box lỏng.
- Nguyên nhân gốc: máy (YOLOE nhầm `three_wheeler`, giữ box conf thấp); guideline mơ hồ (câu mở 2 và 5 của v1.0); chưa nắm quy tắc ở vòng duyệt đầu (bẫy lọt, CAL-06).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42) trong 24 ảnh batch, 88 đối tượng (img_003, 006, 014, 031, 035; nguồn `qc/qc_sample.csv`); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt). Phiếu `qc_review.csv` được soạn nháp cùng Claude từ ảnh có in id, rồi tôi đối chiếu lại.
- Lỗi theo **đối tượng**: 5/88 = 5,7% · theo **ảnh**: 3/5 = 60,0% (ngưỡng Release ≤ 2%)
- Top lỗi: `thua` 3 lỗi (img_003: #20 three_wheeler conf 0.20 và #10 truck conf 0.25; img_006: #68 car conf 0.55) và `sai_vi_tri_kich_thuoc` 2 lỗi (img_006: một box motorbike bao nhiều xe máy đỗ sát nhau; img_031: #367 person, box dừng ở eo, thiếu phần chân). Các loại `bo_sot`, `sai_lop`, `sai_thuoc_tinh`, `sai_mask`: 0 lỗi ghi nhận, nhưng `bo_sot` chưa được kiểm kỹ nên con số 0 không có nghĩa là không có.
- **Giới hạn mẫu nhỏ** (`qc_report_auto.md`): 3 ảnh lỗi trong 5 ảnh, cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là 88%. Với mẫu nhỏ như vậy không thể chứng minh ngưỡng 2%; chiều ngược lại, 3/5 vẫn đủ để kết luận lô chưa đạt.
- Script kiểm được / không kiểm được: script chỉ đếm theo phiếu kiểm bằng mắt, tính tỉ lệ và khoảng Wilson. Nó không tự biết box có ôm sát không, cũng không phát hiện vật mà cả máy và tôi cùng bỏ sót; ở các ảnh đông (img_006, img_035) nhiều box chồng nhau nên tôi chưa kiểm hết từng vật.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đủ rõ để quyết các ca đã gặp và dữ liệu không hỏng; Rework vì ảnh lỗi 3/5 = 60% > 2%, lỗi `thua` lặp lại (3/5 lỗi, quy tắc cho box conf thấp chưa đủ chặt), và bắt bẫy mù ở vòng duyệt đầu chỉ 6/12 = 50% < 80%. Chưa thể Release.
Nếu Rework / Hold, việc cần làm tiếp: sửa các box lỗi ở img_003, img_006, img_031 trong CVAT rồi export `labels/qc_fixed.json`, chạy lại `build_release.py`; thêm quy tắc bỏ box `three_wheeler`/`truck`/`car` có conf thấp khi không thấy vật rõ vào guideline v1.1; chốt câu mở 2 (xe máy đỗ dày: từng xe hay `ignore_region`); lấy mẫu QC mới (seed khác, có ca `can_review`) sau khi sửa.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude: soạn nháp phiếu `qc_review.csv` từ ảnh có in id và hỗ trợ rà soát bài nộp; tôi đối chiếu lại với ảnh gốc [CẦN BẠN XÁC NHẬN và bổ sung nếu dùng AI việc khác].
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? [CẦN BẠN ĐIỀN: không / có, như sau ...]
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: chạy trên CPU, không GPU (thời gian trong `model_log.csv` đo bằng `Measure-Command` nên là Windows PowerShell). [CẦN BẠN ĐIỀN: phiên bản CVAT Docker và vướng mắc cài đặt nếu có]
- Dùng cache `data/cache/` thay vì tự chạy? [CẦN BẠN ĐIỀN: không / có, vì ...] (`model_log.csv` có r07, r08 chạy toàn bộ 36 ảnh với thời gian đo được).
- Lỗi / bug script đã gặp (nếu có): [CẦN BẠN ĐIỀN, hoặc ghi "không"]. Ghi chú: `time_log.csv` ban đầu sai định dạng, đã sửa thành hai dòng `gold_manual`, `batch_review`.
- Một điều bạn làm khác lần sau: quét ảnh tìm vật thiếu và so cạnh box trước khi nhìn nhãn máy, vì bẫy lọt và 6/12 ở vòng đầu đều do nhìn nhãn máy trước.
