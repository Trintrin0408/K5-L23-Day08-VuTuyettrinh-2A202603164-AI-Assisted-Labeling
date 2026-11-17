# Báo cáo Day 08 — 2A202603164 - Vũ Tuyết Trinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| motorbike | YOLO26m | **0.1** | `threshold_sweep`: ở 0.1 bắt 14/15 xe máy gold (recall 93%, precision 70%); lên 0.3 chỉ còn 10/15. Tôi chấp nhận 6 box thừa phải xoá để không sót xe máy. Pilot (`r02`): xe máy thật phần lớn nhận đúng |
| person | YOLO26m | 0.3 (mặc định) | Không ngưỡng nào ≤ 0.5 đạt recall 90%: tối đa 20/28 = 71% ở 0.1–0.15. Hạ từ 0.3 xuống 0.15 chỉ thêm 2 người (18→20/28) mà thêm 4 box thừa (8→12), nên giữ 0.3; người bị sót phải tự vẽ khi review |
| car | YOLO26m | 0.3 (mặc định) | Recall đứng yên 8/9 = 89% từ 0.1 đến 0.5 (chưa đạt 90%), chỉ 9 vật nên không chọn ngưỡng riêng |
| bicycle / truck / bus | YOLO26m | 0.3 (mặc định) | Quá ít vật trong gold (2 / 1 / 0, < 5) để chọn ngưỡng riêng. Pilot (`r02`): xe đạp, xe tải, xe buýt thật nhận đúng nhưng YOLO26 gọi xích lô là bicycle (img_032) và gọi xe xanh img_011 (theo tôi là car) là truck → kiểm kỹ khi review |
| three_wheeler | YOLOE-26s-seg, prompt: "tuk-tuk", "cyclo", "cargo tricycle" | **0.15** | `threshold_sweep`: ở 0.15 bắt 7/7 xe ba bánh gold (recall 100%, precision 78%); lên 0.3 sót 2/7. YOLO26 không có lớp này, không lần nào ra three_wheeler (`r02`); YOLOE pilot đúng 5/8 box (`r04`) |

- Chuỗi ngưỡng dùng ở Part 4: `--thr "motorbike=0.1,three_wheeler=0.15"` (lớp khác dùng `--conf 0.3`)
- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`) — bản bài khuyến nghị, chạy được trên CPU (4.45 s/ảnh, `r05`), máy RAM 15.5 GB đủ chạy; không dùng SAM 3 vì cần quyền truy cập và GPU. Mask từ box máy có lỗi (dính người ngồi trên xe, lệch/thủng ở tuk-tuk img_013, box sai thì mask sai theo) nên Part 5 dùng click điểm.
- Lỗi **không sửa được bằng đổi ngưỡng**:
  - Sai lớp xe ba bánh: img_032 xích lô bị YOLO26 gọi bicycle; img_013 tuk-tuk bị gọi car; nâng ngưỡng 0.25 → 0.5 (`r02` → `r03`) thì img_032 vẫn sai lớp, img_007 xích lô chỉ đổi từ sai lớp (motorbike) thành mất box. Sửa bằng model khác: YOLOE cho three_wheeler.
  - Bỏ sót: person tối đa 20/28 dù hạ ngưỡng; img_007 xích lô bị cả YOLO26 (ở 0.5) và YOLOE (0 box) bỏ sót. Chỉ sửa được bằng người review tự quét ảnh và vẽ thêm.
  - YOLOE nhận nhầm xe 4 bánh thành three_wheeler (img_011, img_002: 3/8 box pilot, `r04`) → kiểm khi review.
- Box **điểm cao mà sai**: img_011 xe xanh (theo tôi là car) bị YOLO26 gọi truck với conf 0.77. Điểm cao không có nghĩa là đúng.
- Giới hạn: ngưỡng được chọn và chấm trên chính 4 ảnh gold, nên precision / recall của máy trên gold ở ngưỡng đã chọn là **lạc quan**. Mẫu nhỏ: three_wheeler 7 vật, car 9, motorbike 15 → ngưỡng chỉ là ước lượng thô; bicycle / truck / bus không đủ vật để chọn.

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
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Ca khó khi gán gold** (box tick `can_review` trong `labels/gold_manual.json`, id theo file export):
  - `img_020` box #55, #56 (`person`, cao 13.7 px và 14.4 px): người nhỏ hơn 20 px, chỉ lộ phần đầu nhưng nhìn thấy rõ. Phân vân giữa quy tắc 4.1.6 (cao < 20 px thì không vẽ, chỉ dùng `ignore_region` khi cụm ≥ 5 vật) và việc vật vẫn nhận ra rõ ràng là người; chỉ có 2 người nên không đủ thành cụm.
  - `img_001` box #12 (`person`, cao 25 px): không chắc là người, chỉ thấy một bóng đen.

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
