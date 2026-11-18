# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603164 - Vũ Tuyết Trinh

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khó có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.
> Phân loại nguyên nhân: **guideline mơ hồ** (phải sửa chữ: CAL-03, CAL-04, CAL-05, CAL-06) khác **máy sai** (guideline đã rõ, chỉ thêm bước kiểm khi review: CAL-01, CAL-02).

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Ngưỡng là **chiều cao vật**, không phải độ dày: vật ≥ 20 px thì gán từng xe, kể cả khi đứng sát nhau. `ignore_region` chỉ dùng cho cụm ≥ 5 vật cao < 20 px | img_005, img_028 (gán từng xe, không ignore); img_018, img_021 (ignore cho cụm nhỏ ở xa) | CAL-06 |
| 4 | Người ngồi trong xe ba bánh có mui: ngưỡng "còn thấy ≥ 20%" có hợp lý không? | Giữ 20% nhưng **ước lượng theo phần cơ thể** để hai người đánh giá giống nhau: thấy đầu + thân ≈ 50% thì gán; chỉ thấy đầu ≈ 15% thì không gán. Không chắc thì gán và tick `can_review` | img_023 (khách trong tuk-tuk giữa), img_013 | CAL-03 |

**Chưa chốt:** câu 1 (xe đạp điện), 3 (hàng nhô ra), 5 (kính xe), 6 (box chồng nhau ban đêm), 7 (`conf` tối thiểu). Lô này không có ca nào đủ rõ để làm ảnh ví dụ, nên giữ mở cho vòng sau, không đoán.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_009, img_010, img_014 | Box `three_wheeler` nằm trên xe máy, xe 4 bánh hoặc chỗ trống | YOLOE-26s (prompt tuk-tuk / cyclo / cargo tricycle, ngưỡng 0.15) gán thừa: 9/41 box bị xoá | Xoá box thừa; đổi sang `motorbike` nếu đúng là xe máy | **4.3 (thêm):** khi review, mọi box `three_wheeler` của máy phải đếm được 3 bánh hoặc thấy rõ mui / thùng của tuk-tuk, xích lô, ba gác; không thì đổi lớp hoặc xoá. *(Máy sai, không đổi định nghĩa lớp.)* — CAL-01 |
| img_023 | Hai tuk-tuk đỗ sát nhau | YOLOE vẽ **một** box gộp cả hai xe | Hai box `three_wheeler` riêng | **4.1.8 (mới):** xe đỗ sát nhau thì mỗi xe một box; box rộng bất thường (dài hơn 1 xe) phải tách. — CAL-02 |
| img_013, img_023 | Mask tuk-tuk có người lái / khách | SAM 2.1 base (click điểm) tô cả người vào mask xe; 1 click âm chưa loại hết người lái | Mask xe **không** gồm người | **4.2.5 (mới):** mask xe không gồm người lái, khách, hàng rời; người có mask riêng; vùng chồng nhau thuộc vật ở phía trước (theo 4.2.4). Khi dùng SAM: mỗi người trên xe cần ≥ 1 click âm, kiểm lại mask trước khi nhận. — CAL-03 |
| img_024 | Xe khách 16 chỗ dáng Hiace: v1.0 ghi `car` gồm "minivan", `bus` gồm "minibus" | Máy gọi `bus` | `bus` | **Mục 2 (sửa dòng `car` / `bus`):** ≤ 9 chỗ (minivan gia đình) là `car`; 12–16 chỗ, thân dài, nhiều hàng cửa sổ là `bus`. — CAL-04 |
| img_020, img_001 (gold) | img_020: 2 người cao 13.7 / 14.4 px nhìn rõ, không đủ cụm 5 để ignore. img_001: bóng đen 25 px không chắc là người | (gán tay, không có máy) | img_020: giữ 2 box `person` + `can_review`. img_001: gán `person` (lớp tin nhất) + `can_review` | **4.1.6 (thêm ngoại lệ):** vật < 20 px đứng riêng lẻ (< 5 vật, không thành cụm) mà nhận rõ lớp thì vẫn vẽ box và tick `can_review`; không rõ lớp thì theo 4.3. — CAL-05 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2 (ngưỡng ignore theo chiều cao vật) và 4 (ước lượng 20% theo phần cơ thể). Sửa mục 2 (phân biệt minivan `car` / minibus `bus`). Thêm 4.1.6 ngoại lệ vật nhỏ riêng lẻ, 4.1.8 xe đỗ sát mỗi xe một box, 4.2.5 mask xe không gồm người, 4.3 bước kiểm box `three_wheeler` của máy. Câu 1, 3, 5, 6, 7 còn mở | CAL-01 … CAL-06 |
