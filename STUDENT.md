# Thông tin học viên

- Họ và tên: Vũ Tuyết Trinh
- MSHV: 2A202603164
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Trintrin0408
- Repo bài nộp (URL): https://github.com/Trintrin0408/K5-L23-Day08-VuTuyettrinh-2A202603164-AI-Assisted-Labeling
- Commit chốt bài (hash): e5a551e
- Máy: Windows 11 Pro, RAM 15.5 GB, không có GPU (chạy CPU, torch 2.15.0+cpu)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 593/621 = 95.5% / 6/621 = 1.0% / 20/621 = 3.2% (mẫu số: 621 box máy); added: 8/607 = 1.3% (mẫu số: 607 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 48/72 = 66.7% / 48/62 = 77.4% (62 đối tượng gold, 4 ảnh; lạc quan vì chọn ngưỡng trên chính gold)
- Bắt bẫy: lượt duyệt đầu 1/12 = 8.3% (số đo người duyệt); sau khi export, mở `traps_key.json` và sửa lại thì còn 8/12 = 66.7% (không đo người duyệt, xem REPORT mục 3)
- Thời gian: gán tay 8.5 phút/ảnh (34 phút / 4 ảnh); duyệt có pre-label 3.75 phút/ảnh (90 phút / 24 ảnh)
- Lỗi QC: 100% theo ảnh (5/5), 10.8% theo đối tượng (13/120, cận dưới)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 5/5 > 2%, lỗi hệ thống box lệch 7/13, bắt bẫy lượt đầu 1/12 < 80%
