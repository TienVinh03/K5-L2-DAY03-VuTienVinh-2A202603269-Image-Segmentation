# Kiểm tra trước nộp — 12/11/2026

Đối chiếu `SUBMISSION.md` và `docs/SUBMISSION.md` với workspace hiện tại.

| Nội dung | Kết quả |
| --- | --- |
| 9 ZIP đúng tên, đủ ảnh và đọc được | Đạt kiểm tra cấu trúc bằng `scripts/inspect_submissions.py`, không có lỗi |
| Báo cáo có bốn mục và ba ca cân nhắc | Có; mục 2 đã bổ sung class đầu tiên là `person`, giữ mask tự vẽ và chỉnh trên CVAT sau gợi ý. Chưa xác định object đầu tiên thuộc ảnh nào trong hai ảnh đã nêu |
| Điểm ba tier | 49,4/82; không phải điểm chính thức trên 100 |
| Tên repo | `K5-L2B-DAY03-VuTienVinh-03269-Image-Segmentation`, khác mẫu `K5-L2-DAY03-...` ở phần `L2B` |
| Danh tính trong tên file và repo | Đã xác nhận MSSV là `2A202603269`; tên report đúng MSSV. Repo vẫn dùng mã lớp `03269`, cần đổi phần MSSV theo mẫu đề |
| Đưa file lên GitHub | ZIP và báo cáo đang untracked tại lúc kiểm tra; chưa đủ bằng chứng đã có trên fork |
| Coach đọc được repo và nộp LMS | Chưa kiểm tra được từ workspace |
| Hạn theo đề | 12/11/2026 lúc 12:00 (UTC+7); cần kiểm tra trạng thái nộp thực tế |

Báo cáo hiện có hai mô tả chưa thống nhất: một đoạn ghi kết quả cuối đã export từ CVAT, đoạn khác ghi số liệu chưa phải kết quả chấm bản export CVAT mới. Cần đối chiếu mô tả với đúng ZIP thực nộp. Kiểm tra cấu trúc không xác nhận nguồn gốc export hoặc thay thế kiểm tra phủ vùng/chồng lấn trong CVAT.

## Dọn script

Đã chuyển ba file không cần cho việc kiểm tra/nộp sang `data/_cache/archived_scripts/` (local, Git bỏ qua):

- `refine_tiers_reference.py`: script thử nghiệm chọn tham số theo reference.
- `report_reference_errors.py`: script sinh báo cáo lỗi, có thể ghi đè báo cáo đã sửa tay khi chạy lại.
- `sam3-inference-ultralytics.py`: ví dụ inference ảnh/video dùng ảnh mẫu `dogs.png`, không thuộc pipeline bài nộp.

Giữ các script tạo annotation SAM, chuyển label, kiểm tra ZIP, chấm điểm và đóng gói. Không đổi nội dung ZIP trong đợt dọn dẹp này.

## Việc còn lại

1. Mục 2 đã có class đầu tiên `person` và cách xử lý sau gợi ý; còn xác định object đầu tiên thuộc ảnh nào trong hai ảnh và chi tiết biên đã sửa.
2. MSSV đã xác nhận là `2A202603269`; tên repo theo mẫu đề là `K5-L2-DAY03-VuTienVinh-2A202603269-Image-Segmentation`. Chưa đổi tên repo remote.
3. Đối chiếu ZIP cuối với mô tả export trong báo cáo; kiểm tra Hard trên CVAT.
4. Commit/push ZIP và report lên fork, mở lại kiểm tra file và quyền đọc của coach.
5. Nộp URL fork cá nhân trên LMS và kiểm tra trạng thái nộp.
