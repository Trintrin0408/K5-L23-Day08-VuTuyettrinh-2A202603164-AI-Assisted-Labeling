# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 621; đối tượng sau duyệt: 609; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 596 | 596/621 = 96.0% | 621 box máy |
| edited (sửa) | 6 | 6/621 = 1.0% | 621 box máy |
| deleted (xoá) | 17 | 17/621 = 2.7% | 621 box máy |
| added (vẽ thêm) | 7 | 7/609 = 1.1% | 609 đối tượng sau duyệt |

Trong số deleted + added, **3 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 4.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 267 | 1 | 2 | 0 |
| bicycle | 7 | 0 | 1 | 1 |
| motorbike | 230 | 2 | 6 | 2 |
| car | 54 | 0 | 0 | 0 |
| bus | 2 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 1 |
| three_wheeler | 30 | 3 | 8 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 62 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 24 | 14 | 66.7% (KTC 95%: 55–76%) | 77.4% (KTC 95%: 66–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 1/12 = 8.3% (KTC 95%: 1–35%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 1/3
  - extra: 0/3
  - missing: 0/3
  - wrong_class: 0/3
  - lọt: img_027.jpg (loose), img_025.jpg (extra), img_023.jpg (missing), img_022.jpg (wrong_class), img_013.jpg (wrong_class), img_005.jpg (missing), img_009.jpg (extra), img_033.jpg (loose), img_018.jpg (extra), img_026.jpg (wrong_class), img_024.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
