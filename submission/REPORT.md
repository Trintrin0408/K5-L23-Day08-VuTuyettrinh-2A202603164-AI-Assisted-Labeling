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
- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  Kết quả: 726 box cho 28 ảnh (24 batch + 4 gold): person 311, motorbike 274, car 65, three_wheeler 52, bicycle 13, truck 7, bus 4. Box YOLO26 bị loại vì chồng lên box YOLOE ưu tiên cao hơn (IoU ≥ 0.6): car 11, truck 6, motorbike 2, bus 2, bicycle 1 (đúng kiểu xe ba bánh bị YOLO26 gọi car/truck ở Part 2).
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`) — bản bài khuyến nghị, chạy được trên CPU (4.45 s/ảnh, `r05`), máy RAM 15.5 GB đủ chạy; không dùng SAM 3 vì cần quyền truy cập và GPU. Mask từ box máy có lỗi (dính người ngồi trên xe, lệch/thủng ở tuk-tuk img_013, box sai thì mask sai theo) nên Part 5 dùng click điểm.
- Lỗi **không sửa được bằng đổi ngưỡng**:
  - Sai lớp xe ba bánh: img_032 xích lô bị YOLO26 gọi bicycle; img_013 tuk-tuk bị gọi car; nâng ngưỡng 0.25 → 0.5 (`r02` → `r03`) thì img_032 vẫn sai lớp, img_007 xích lô chỉ đổi từ sai lớp (motorbike) thành mất box. Sửa bằng model khác: YOLOE cho three_wheeler.
  - Bỏ sót: person tối đa 20/28 dù hạ ngưỡng; img_007 xích lô bị cả YOLO26 (ở 0.5) và YOLOE (0 box) bỏ sót. Chỉ sửa được bằng người review tự quét ảnh và vẽ thêm.
  - YOLOE nhận nhầm xe 4 bánh thành three_wheeler (img_011, img_002: 3/8 box pilot, `r04`) → kiểm khi review.
- Box **điểm cao mà sai**: img_011 xe xanh (theo tôi là car) bị YOLO26 gọi truck với conf 0.77. Điểm cao không có nghĩa là đúng.
- Giới hạn: ngưỡng được chọn và chấm trên chính 4 ảnh gold, nên precision / recall của máy trên gold ở ngưỡng đã chọn là **lạc quan**. Mẫu nhỏ: three_wheeler 7 vật, car 9, motorbike 15 → ngưỡng chỉ là ước lượng thô; bicycle / truck / bus không đủ vật để chọn.

## 2. AI error profile (từ `qc/review_stats.md`)

Số của nhãn cuối (`qc/review_stats.md`; đã loại 24 đối tượng quanh bẫy khỏi bảng):

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 593 | 593/621 = 95.5% | 621 box máy |
| edited | 6 | 6/621 = 1.0% | 621 box máy |
| deleted | 20 | 20/621 = 3.2% | 621 box máy |
| added | 8 | 8/607 = 1.3% | 607 đối tượng sau duyệt |

Trong deleted + added có 4 ca đổi lớp (box đúng chỗ, sai lớp), nên số vật máy sót thật khoảng 8 − 4 = 4. Số của lượt duyệt đầu (`qc/review_stats_round1.md`) gần như giống hệt: accepted 596/621 = 96.0%, deleted 17/621, added 7/609.

- **Lớp máy tệ nhất: `three_wheeler`.** Có 41 box máy, bị xoá 9/41 = 22% và sửa 3/41; trong khi `motorbike` chỉ bị xoá 7/238 = 2.9% và `person` 2/270 = 0.7%. Lỗi theo hai chiều:
  - YOLOE gán `three_wheeler` thừa hoặc sai lên xe máy, xe 4 bánh (img_009, img_010, img_014: xoá 5 box; img_009 đổi 1 `three_wheeler` → `motorbike`; img_027, img_035 cũng có sửa box `three_wheeler`).
  - YOLOE bỏ sót xe ba bánh thật (img_004, img_017: tôi vẽ thêm mỗi ảnh 1), và box lỏng không ôm hết tuk-tuk (img_013).
  - Khớp với `r04` ở Part 2 (3/8 box pilot là xe 4 bánh): prompt "tuk-tuk / cyclo / cargo tricycle" ở ngưỡng 0.15 đổi recall lấy precision.
- **So với gold:** máy đạt precision 48/72 = 66.7% (KTC 95%: 55–76%) và recall 48/62 = 77.4% (KTC 95%: 66–86%). Mẫu số recall là 62 đối tượng gold.
  - **Mâu thuẫn quan trọng:** recall 77% nghĩa là máy sót khoảng 1/4 vật, nhưng trên batch tôi chỉ vẽ thêm 8/607 = 1.3%. Lô batch khó có thể tốt hơn gold nhiều đến vậy. Phần lớn chênh lệch là **vật máy sót mà tôi cũng sót**: `added` thấp vì tôi duyệt theo box có sẵn, không phải vì máy không sót. Số gold lại lạc quan (ngưỡng chọn trên chính 4 ảnh này), nên khoảng cách thật có thể còn lớn hơn.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy lượt đầu: 1/12 = 8.3%** (KTC 95%: 1–35%; ngưỡng đạt 10/12). Đây là số đo việc duyệt của tôi.
  - Chỉ bắt được 1 bẫy: img_021, box `car` lỏng.
  - **missing lọt 3/3** (img_005 person, img_023 three_wheeler, img_024 bus).
  - **extra lọt 3/3** (img_009 truck, img_018 motorbike, img_025 truck).
  - **wrong_class lọt 3/3** (img_013, img_022, img_026, cả ba đều là box bị đổi thành `bicycle`).
  - **loose lọt 2/3** (img_027, img_033).
- **Bước checklist đã bỏ:**
  - **Bước 0 (ẩn nhãn máy, quét trước):** chỉ làm ở 5/24 ảnh (003, 004, 005, 009, 013). Ở 19 ảnh còn lại tôi nhìn nhãn máy ngay, nên không thể thấy vật không có box. Cả 3 bẫy missing đều lọt, kể cả img_023 nơi vật bị bỏ là một chiếc tuk-tuk chiếm gần nửa ảnh. Ở img_005 tôi có quét trước nhưng không ghi số đếm, nên vẫn không có gì để đối chiếu.
  - **Bước 3 (sai lớp):** tôi chỉ soi lớp của xe ba bánh, lỗi đã biết từ Part 2, mà không đọc nhãn từng box. Cả 3 box `bicycle` sai đều lọt, kể cả ở img_013, ảnh tôi đã quay lại kiểm riêng chiếc tuk-tuk.
  - **Bước 2 (thừa):** các box `truck` ma to nằm trên nền (img_009, img_025) bị bỏ qua. Mắt tôi chỉ tìm lỗi ở chỗ có xe, không hỏi "box này có vật không".
  - **Mệt mỏi:** làm liền 24 ảnh, 0 lần nghỉ. 8/24 ảnh tôi không sửa box nào (003, 005, 008, 023, 025, 026, 028, 033), và 5 trong 8 ảnh đó có bẫy lọt.
- **Accepted cao mà bắt bẫy thấp:** accepted 96% không chứng minh máy tốt. Tôi giữ nguyên gần như mọi box, kể cả 11/12 lỗi đã biết chắc là sai. Đây đúng là thiên kiến tự động hóa: nhãn máy trông hợp lý nên tôi "làm theo" (lọt wrong_class, extra) và "bỏ qua" chỗ không có box (lọt missing).
- **Sau lượt đầu:** sau khi export, tôi mở `traps_key.json` (đúng thời điểm hướng dẫn cho phép) để sửa lại các bẫy lọt trong nhãn cuối. `review_stats.md` hiện ghi 8/12, nhưng số này **không đo người duyệt**, vì đã biết ảnh và loại lỗi; số đo việc duyệt là 1/12 ở trên.
  - **img_023:** tôi vẽ 2 box `three_wheeler` cho 2 chiếc tuk-tuk riêng. Box gốc của máy (215,112)–(799,646) gộp 2 xe thành một, tức box gốc sai. Script vẫn chấm "lọt" vì so với box gốc (IoU 0.40 / 0.28); tôi giữ nhãn của mình vì đúng guideline hơn.
  - **Nhãn cuối còn 3 lỗi đã biết, chưa sửa:** thiếu 1 `person` (img_005), thiếu 1 `bus` (img_024), thừa 1 `motorbike` ma (img_018). Ba lỗi này được tính vào quyết định ở mục 7.

## 4. Máy có đáng dùng không?

- **Thời gian (`time_log.csv`):** gán tay 34 phút / 4 ảnh = 8.5 phút/ảnh; duyệt 90 phút / 24 ảnh = 3.75 phút/ảnh, tức nhanh hơn khoảng 56%. Cả hai số đều tính cả thời gian hỏi đáp trong lúc làm.
  - **Thiên lệch:** gold được gán trước nên khi duyệt tôi đã quen kiểu cảnh và bộ nhãn.
  - **Thiên lệch lớn hơn:** 3.75 phút/ảnh là tốc độ của một lượt duyệt **bỏ Bước 0 ở 19/24 ảnh và lọt 11/12 bẫy**. Phần "tiết kiệm" có một phần đến từ việc bỏ kiểm tra, không phải từ máy. Thêm khoảng 15 phút sửa lại thì vẫn khoảng 4.4 phút/ảnh, còn nhanh hơn gán tay khoảng 48%.
- **Kết luận: có, kèm điều kiện.**
  - Dùng pre-label cho `person`, `motorbike`, `car`: accepted cao, deleted thấp (person 2/270, motorbike 7/238, car 0/54).
  - `three_wheeler` và lớp hiếm (`bicycle`, `truck`, `bus`) phải kiểm từng box: three_wheeler bị xoá 22%, và các bẫy sai lớp đều rơi vào `bicycle`.
  - Máy chỉ đáng dùng nếu người duyệt giữ được Bước 0: với recall máy 77% trên gold, bỏ Bước 0 nghĩa là chấp nhận sót khoảng 1/4 số vật.
- **Ba quy tắc giảm thiên kiến của tôi:**
  1. **Đếm trước ở 24/24 ảnh:** ẩn nhãn máy và ghi số đếm theo lớp vào sổ ghi chú **trước** khi hiện box, ở 100% ảnh (lượt này chỉ 5/24). Kiểm tra: số dòng trong sổ có số đếm = số ảnh của lô.
  2. **Đọc nhãn từng box ở lớp hiếm:** ở mọi ảnh, dùng Filters lọc riêng `bicycle`, `truck`, `bus`, `three_wheeler`, bấm từng box và đối chiếu lớp với vật thật (lượt này lọt 3/3 wrong_class, đều là `bicycle`, và 2 box `truck` ma). Kiểm tra: 24/24 dòng trong sổ có ghi "đã lọc lớp hiếm".
  3. **Ảnh không sửa gì phải quét lại:** ảnh nào không sửa box nào thì quét lại lần hai theo 4 góc phần tư trước khi sang ảnh sau (lượt này 8/24 ảnh không sửa gì, 5 trong đó có bẫy lọt). Kiểm tra: mọi ảnh "0 sửa" trong sổ có ghi "đã quét lại 4 góc".

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk, có người lái) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương: thân `400,480`, mui `600,110`; 1 âm: người lái `720,300`) | **Có, một phần.** Click âm chỉ bỏ được vùng áo quanh điểm click; chân, quần và tay người lái vẫn bị tô | Mask phủ đủ mui, thân, bánh, không tràn ra đường. Nhưng **tô cả nền nhìn xuyên qua khung xe** (người ngồi phía sau, ghế), là các lỗ lớn mà guideline 4.2.2 nói phải bỏ ra | `tuktuk_img013.jpg` |
| img_032 (xích lô phủ chăn hoa) | SAM 2.1 base | 3 (2 dương: ghế `450,340`, khung trước `380,450`; 1 âm: người đứng cạnh `585,330`) | Không dính người đứng cạnh; xe không có người ngồi | Mask bám viền tốt nhất trong 3 vật (cả chăn, 2 bánh trước, khung). **Không đủ xe** vì đuôi xe (yên người đạp, bánh sau) bị người đứng che: đây là do khuất, không phải lỗi SAM, và đúng 4.2.1 (chỉ phần nhìn thấy) | `xichlo_img032.jpg` |
| img_023 (tuk-tuk giữa, có khách) | SAM 2.1 base | 2 (2 dương: mui `620,215`, thân `560,400`; **không** click âm, cố ý) | **Có: tô luôn người khách ngồi trong xe** | Tách đúng chiếc ở giữa, **không tràn sang 2 tuk-tuk đứng sát** (chỗ máy từng gộp 2 xe làm 1 box, xem mục 3). Phần thân bị người đứng che không được tô | `tuktuk_img023.jpg` |

- **Đối chiếu guideline mục 4.2:**
  - **Quy tắc giúp được:** 4.2.1 (bám phần nhìn thấy) giải thích vì sao xích lô img_032 và tuk-tuk img_023 "thiếu" đúng phần bị che. 4.2.4 (hai vật chạm nhau thì mỗi vật một polygon) được SAM làm đúng ở img_023.
  - **Quy tắc thiếu:** 4.2 **không nói mask xe có gồm người lái / khách / hàng hoá hay không**. Mặc định SAM coi "xe + người trên xe" là một vật (img_023, 2 click dương thì lấy cả khách). Muốn loại người phải thêm click âm, mà 1 click âm vẫn chưa đủ (img_013). Đề xuất đưa vào v1.1: "mask xe **không** gồm người; người có mask riêng; vùng chồng nhau thuộc người (ở phía trước)". Ca này vào `calibration_decisions.csv`.
  - 4.2.2 (lỗ lớn bỏ ra) không được SAM tự làm: img_013 tô cả nền xuyên qua khung xe, phải sửa tay hoặc thêm click âm vào từng lỗ.
- **Quy tắc / lỗi rút ra:**
  1. Mỗi người ngồi trên xe cần ít nhất 1 click âm (thân và chân là 2 vùng riêng); kiểm lại mask trước khi nhận.
  2. Với xe có khung hở (tuk-tuk, xích lô), thêm click âm vào các lỗ lớn nhìn xuyên ra nền.
  3. Mask đẹp không sửa được lớp sai: lớp vẫn phải kiểm riêng (ở `r05`, mask tuk-tuk img_013 từ box YOLO26 mang lớp `car`). Ở đây lớp do tôi chọn khi click.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử (chỉ dùng `sam2_points.py` offline).

## 6. Calibration (Part 6)

- **Số quyết định:** 6 (`calibration_decisions.csv`): 2 `loi_may`, 2 `cau_hoi_mo`, 1 `bay_lot`, 1 `can_review`.
- **Quan trọng nhất: CAL-03** (mask / box xe có gồm người không). Part 5 cho thấy SAM 2.1 mặc định tô cả người lái, khách vào mask xe (img_013, img_023), mà v1.0 mục 4.2 không có quy tắc nào. Không chốt thì hai người làm mask sẽ ra hai kiểu khác nhau.
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:**
  - 4.2.5: mask xe không gồm người.
  - 4.1.8: xe đỗ sát thì mỗi xe một box (CAL-02, img_023).
  - Mục 2: minivan ≤ 9 chỗ là `car`, minibus 12–16 chỗ là `bus` (CAL-04, img_024).
  - 4.1.6: ngoại lệ vật nhỏ riêng lẻ nhận rõ lớp (CAL-05, img_020) và ngưỡng `ignore_region` theo chiều cao vật (CAL-06, img_005 / img_028).
  - 4.3: bước kiểm box `three_wheeler` của máy (CAL-01).
  - Câu hỏi mở 1, 3, 5, 6, 7 **chưa chốt** vì lô này không có ca ví dụ rõ.
- **Nguyên nhân gốc, tách hai loại:**
  - **Guideline mơ hồ** (phải sửa chữ): CAL-03, 04, 05, 06. Ví dụ v1.0 ghi `car` gồm "minivan" nhưng `bus` gồm "minibus", nên xe 16 chỗ ở img_024 rơi vào giữa hai lớp.
  - **Máy sai** (guideline đã rõ, chỉ thêm bước kiểm): CAL-01 (YOLOE gán `three_wheeler` thừa, 9/41 bị xoá), CAL-02 (gộp 2 tuk-tuk làm 1 box).
  - **Lỗi của tôi** (không đổi guideline): bẫy lọt ở mục 3 và box lệch ở mục 7 là do tôi không theo checklist, không phải do quy tắc thiếu.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh (img_003, img_006, img_014, img_031, img_035), 120 đối tượng. Gồm 0 ảnh `can_review` (batch không có box nào được tick) và 5 ảnh ngẫu nhiên, seed 42.
  - Người kiểm là tôi (**hạn chế**: tôi vừa là người duyệt, nên dễ lặp lại đúng điểm mù của mình).
  - 3 ảnh mang lỗi đã biết (img_005, img_018, img_024; mục 3) **không rơi vào mẫu**, nên mẫu không phản ánh chúng.
- **Lỗi theo đối tượng:** 13/120 = 10.8%. **Lỗi theo ảnh:** 5/5 = 100% (ngưỡng Release ≤ 2%).
  - Với các ô tôi ghi "2 trở lên" (img_006 bỏ sót và thừa, img_014 và img_035 lệch), `qc_review.csv` chỉ tính 2, nên **13 là cận dưới**.
- **Top lỗi:**
  - **Sai vị trí / kích thước: 7/13** (img_003, img_006, img_014, img_031, img_035). Đây là lỗi hệ thống, có mặt ở cả 5 ảnh. Khớp với mục 3: lúc review tôi gần như không làm bước 4 (lọt 2/3 bẫy `loose`; img_013 chỉ phát hiện box tuk-tuk lỏng khi được hỏi lại).
  - **Bỏ sót: 4/13** (img_003, img_006). img_003 là ảnh tôi giữ nguyên 27/27 box khi review.
  - **Thừa: 2/13** (img_006).
  - Không thấy sai lớp, sai thuộc tính, sai mask trong mẫu.
- **Giới hạn mẫu nhỏ:**
  - 5/5 ảnh lỗi cho cận trên 95% (Wilson) là 100%; cận dưới cũng khoảng 57%. Kết luận "lô không đạt 2%" là chắc chắn, dù mẫu nhỏ.
  - Ngược lại, kể cả khi mẫu 5 ảnh ra 0 lỗi thì cũng **không chứng minh được** ≤ 2%: theo quy tắc 3/n, lỗi thật vẫn có thể tới khoảng 60%.
  - Mẫu cũng chỉ là 5/24 ảnh, do chính người duyệt kiểm.
- **Script kiểm được / không kiểm được:**
  - **Kiểm được:** định dạng, lớp hợp lệ, đủ 24 ảnh, các số accepted / added, bắt bẫy, và đếm lỗi theo phiếu kiểm.
  - **Không kiểm được:**
    - Box có ôm sát không: 7 lỗi lệch chỉ thấy bằng mắt.
    - Vật mà cả máy lẫn tôi cùng sót: recall máy trên gold 77% nhưng added chỉ 1.3%.
    - Tôi có theo checklist thật không: chỉ biết từ sổ ghi chú, Bước 0 ở 5/24 ảnh.
  - Số bắt bẫy trong `qc_report_auto.md` (8/12) là số **sau khi sửa lại**; số đo việc duyệt là 1/12 (mục 3).

**Quyết định: Rework**, ngày 2026-11-18. Lý do, đi theo cây Hold → Rework → Release:
1. **Không Hold.** Guideline đã chốt các ca gặp trong lô (v1.1, CAL-01…06); 5 câu hỏi còn mở không có ca nào trong 24 ảnh; dữ liệu không hỏng.
2. **Rework**, vì 3 điều kiện cùng đúng:
   - ảnh lỗi 5/5 = 100% > 2%;
   - lỗi hệ thống "sai vị trí / kích thước" chiếm 7/13 lỗi;
   - bắt bẫy lượt duyệt đầu 1/12 = 8.3% < 80%, nên lô duyệt chưa đáng tin.
3. **Không thể Release.**

**Việc cần làm tiếp:**
1. Duyệt lại cả 24 ảnh theo 3 quy tắc ở mục 4: đếm trước 24/24, lọc từng lớp hiếm, quét lại ảnh 0 sửa. Thêm **bước 4 bắt buộc**: bấm từng box xem 4 cạnh.
2. Sửa 3 lỗi đã biết: thêm `person` img_005, thêm `bus` img_024, xoá `motorbike` ma img_018.
3. Export `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, QC vòng 2 với mẫu lớn hơn (≥ 10 ảnh). Nếu được, nhờ người khác kiểm.

## 8. Ghi chú cá nhân

- **Ca khó khi gán gold** (box tick `can_review` trong `labels/gold_manual.json`, id theo file export):
  - `img_020` box #55, #56 (`person`, cao 13.7 px và 14.4 px): người nhỏ hơn 20 px, chỉ lộ phần đầu nhưng nhìn thấy rõ. Phân vân giữa quy tắc 4.1.6 (cao < 20 px thì không vẽ, chỉ dùng `ignore_region` khi cụm ≥ 5 vật) và việc vật vẫn nhận ra rõ ràng là người; chỉ có 2 người nên không đủ thành cụm.
  - `img_001` box #12 (`person`, cao 25 px): không chắc là người, chỉ thấy một bóng đen.

- **Dùng AI để làm bài: có, Claude Code (Claude Opus 5.5) trong VS Code**, từ Part 4 trở đi.
  - **Việc AI làm:**
    - Hướng dẫn từng bước (lệnh chạy, thao tác CVAT) và hỏi tôi từng ảnh khi review; ghi sổ ghi chú review từ câu trả lời của tôi.
    - **Sau khi tôi export `batch_reviewed.json` lượt đầu:** đọc `traps_key.json` và vẽ vị trí các bẫy lọt lên ảnh để tôi sửa lại trong CVAT. Vì vậy số 8/12 không đo người duyệt (mục 3).
    - Chọn toạ độ click cho SAM ở Part 5.
    - Soạn nháp `calibration_decisions.csv`, `guideline_v1.1_calibrated.md` và REPORT mục 2–7 từ số liệu script và câu trả lời của tôi.
    - Ghi `qc_review.csv` theo lỗi tôi đếm khi nhìn ảnh QC.
  - **Việc tôi tự làm:**
    - Gán gold (Part 1), duyệt và sửa box trong CVAT (Part 4), nhìn mask (Part 5) và ảnh QC.
    - Chọn các quyết định calibration (CAL-04 xe van, CAL-05 người nhỏ), chọn 3 quy tắc giảm thiên kiến và kết luận mục 4.
  - **Cách kiểm tra:**
    - Mọi con số lấy từ output script (`review_stats`, `qc_report`), không sửa tay.
    - Tôi đọc lại REPORT và guideline đối chiếu với điều mình đã làm.
    - Mask và ảnh QC tôi tự nhìn; AI chỉ chép lại.
- **Gold có bị nhiễm không:** không. 4 ảnh gold gán tay ở Part 1, trước khi chạy model và trước khi dùng AI; tôi không xem pre-label hay hỏi AI trước khi gán xong.
- **Máy / CVAT:** Windows 11 Pro, RAM 15.5 GB, không GPU (torch CPU); CVAT v2.81.0 qua Docker. Không vướng cài đặt đáng kể; chỉ chậm trên CPU (SAM 2.1 khoảng 4.45 s/ảnh).
- **Dùng cache `data/cache/`?** Không; tôi tự chạy `prelabel.py --split all --conf 0.05` cho YOLO26 và YOLOE.
- **Lỗi / bug script:** không gặp.
- **Một điều làm khác lần sau:** kiểm **từng cạnh box**, tức bấm vào từng box và xem 4 cạnh có ôm sát không. Lý do: QC cho thấy 7/13 lỗi là box lệch / lỏng, có mặt ở cả 5 ảnh mẫu, và lúc review tôi gần như bỏ bước 4 (lọt 2/3 bẫy `loose`).
