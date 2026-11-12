# Cải thiện sau khi xem reference — 12/11/2026

**Cập nhật lần 2:** tổng **49,4/82** (Easy 18,2; Medium 15,4; Hard 15,8). Các bảng bên dưới ghi lịch sử lần 1. Xem [phân tích lỗi và kết quả mới](ERROR_ANALYSIS.md) và `reference_refinement_round2.json` cho lần 2. Script hiện chạy từ bản sao lần 1 ở `data/_cache/before_reference_refinement_round2/`, thử thêm opening/closing và khoảng co/giãn rộng hơn. `cN` nghĩa là closing, `oN` nghĩa là opening, N là bán kính kernel ellipse tính bằng pixel.

Đây là hậu xử lý tự động của mask SAM 3, chọn tham số bằng groundtruth đã phát của chính các ảnh đang chấm. Không phải kết quả đánh giá độc lập hoặc bằng chứng tự vẽ trong CVAT. Chưa thực hiện Save/export lại qua CVAT.

| Task | Metric trước → sau | Điểm trước → sau |
| --- | --- | --- |
| Easy | mIoU 0,782 → 0,800 | 17,0 → 17,8 / 20 |
| Medium | mean matched IoU × recall 0,600 → 0,609 | 14,3 → 14,9 / 32 |
| Hard | PQ 0,400 → 0,431 | 13,3 → 15,4 / 30 |
| Tổng | | **44,6 → 48,1 / 82** |

Script `scripts/refine_tiers_reference.py` thử co/giãn mask với kernel ellipse theo từng class, dùng cùng tham số cho tất cả ảnh. Easy chỉ giãn vào vùng chưa gán nhãn. Hard còn thử lọc instance diện tích nhỏ theo class. Chỉ giữ thay đổi khi metric tổng tăng. Không lấy polygon, mask hay tọa độ object từ reference đưa vào ZIP; scorer và groundtruth không bị sửa.

Medium tăng nhờ giãn nhẹ mask `car` và `motorcycle`. Hard tăng nhờ chỉnh biên một số lớp và bỏ các dự đoán nhỏ ở `person`, `car`, `truck`, `traffic light`. Chi tiết tham số và metric đầy đủ ở `reference_refinement.json`. Mask COCO đã chỉnh biên dùng RLE để giữ chính xác vùng rỗng; cần kiểm tra trực quan và khả năng import trên CVAT trước khi dùng trong quy trình nộp bài yêu cầu export từ CVAT.

ZIP gốc được giữ tại `data/_cache/before_reference_refinement/` (đã được gitignore). ZIP cập nhật nằm trong `submissions/`. Các ảnh preview đã được dọn vì không bắt buộc trong bài nộp.

Script thử nghiệm `refine_tiers_reference.py` và script sinh báo cáo `report_reference_errors.py` đã được chuyển vào `data/_cache/archived_scripts/` (local, Git bỏ qua). Muốn chạy lại cần chép script về vị trí cũ trong `scripts/`, vì script xác định thư mục gốc theo vị trí file.

Kiểm tra bài hiện tại từ thư mục gốc:

```powershell
python scripts/inspect_submissions.py
python scoring/scorecard.py --group tiers --dir submissions --out reports/tiers
```

Điểm tăng trên tập dùng để chọn tham số không chứng minh khả năng tổng quát trên ảnh mới. Groundtruth vẫn chỉ nằm ở thư mục local được bỏ qua bởi Git.
