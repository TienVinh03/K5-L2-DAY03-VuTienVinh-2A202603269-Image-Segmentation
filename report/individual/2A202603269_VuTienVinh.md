# Báo cáo thực hành Day 3 — Image Segmentation

- MSSV: 2A202603269 — Vũ Tiến Vinh.
- Mã học viên theo lớp: 03269.
- Ngày cập nhật: 12/11/2026.
- Công cụ: CVAT, Ultralytics SAM 3 (`sam3.pt`), PyTorch, Python, NumPy, OpenCV, Pillow và pycocotools.
- Quy trình thực hiện: tự vẽ annotation cho hai task tier `easy_semantic` và `medium_instance` trên CVAT; có sử dụng mask gợi ý từ SAM 3, import ZIP vào CVAT để chỉnh sửa. Bước xử lý tiếp theo được thực hiện trên CVAT: đối chiếu groundtruth, sửa annotation, Save rồi export lại ZIP.

## 1. Bài đã nộp

Các file dưới đây hiện có trong `submissions/` và đã qua kiểm tra cấu trúc bằng `scripts/inspect_submissions.py`. Trong quá trình làm bài, tôi có tự vẽ trên CVAT, import ZIP gợi ý từ SAM để sửa và export lại. Số ảnh trong bảng là số ảnh có annotation trong ZIP, không phải số ảnh được vẽ hoàn toàn bằng tay. Chưa xác nhận việc nộp trên LMS trong đợt cập nhật này.

| Task | File ZIP | Số ảnh có annotation | Điểm tối đa |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Kết quả tự chấm riêng ba tier trên các phiên bản ZIP đã được scorer kiểm tra trước đó (có chỉnh bằng script). Đã export từ CVAT kết quả cuối vào báo cáo này:

| Task | Metric mới nhất | Điểm gốc | Sau lần chỉnh 1 | Sau lần chỉnh 2 |
| --- | ---: | ---: | ---: | ---: |
| Easy | mIoU = 0,8101 | 17,0 / 20 | 17,8 / 20 | 18,2 / 20 |
| Medium | Mean matched IoU × recall = 0,6166 | 14,3 / 32 | 14,9 / 32 | 15,4 / 32 |
| Hard | PQ = 0,4371 | 13,3 / 30 | 15,4 / 30 | 15,8 / 30 |
| **Tổng ba tier** | | **44,6 / 82** | **48,1 / 82** | **49,4 / 82** |

Chưa tự chấm sáu checkpoint vì không có reference tương ứng trong đợt đối chiếu này. Không cộng điểm checkpoint vào tổng 49,4/82.

Trong phiên bản ZIP đã chấm, Medium có **84 annotation RLE** trên ba ảnh; Hard có **85 annotation** trên hai ảnh, gồm 5 polygon và 80 RLE. Cần kiểm tra lại số lượng và cấu trúc nếu thay bằng bản export mới từ CVAT.

## 2. Một quyết định trước khi dùng gợi ý

Tôi đã tự vẽ annotation trên CVAT cho hai task `easy_semantic` và `medium_instance`. Hai ảnh Medium tôi đã tự vẽ là `000000181542.jpg` và `000000373353.jpg`. Object đầu tiên tôi tự vẽ trước khi xem gợi ý SAM thuộc class `person`.

Sau khi import ZIP gợi ý từ SAM vào CVAT, tôi giữ lại các mask đã tự vẽ trước đó. Tôi dùng gợi ý để đối chiếu, tự chỉnh sửa annotation trên CVAT rồi Save và export lại ZIP.

Các object và quy tắc biên dưới đây được mô tả từ nội dung hai ảnh:

| Ảnh | Object và vị trí | Class | Quy tắc chọn biên |
| --- | --- | --- | --- |
| `000000181542.jpg` | Người phụ nữ mặc áo dài sáng màu, đứng gần giữa ảnh, phía trước xe bán tải và giữa các xe máy | `person` | Bám theo phần cơ thể và trang phục nhìn thấy: tóc, đầu, thân áo, tay, tà áo, quần và chân. Loại nền đường và các xe máy chạy sát người; không nối kín khoảng nền giữa các bộ phận nếu có |
| `000000373353.jpg` | Người mặc áo khoác xám, đội mũ tối màu, đang đi ngang trước taxi vàng ở gần giữa phía dưới ảnh | `person` | Tách người khỏi taxi phía sau; bám theo mũ, đầu, thân áo, tay, quần và giày nhìn thấy. Không gộp mặt đường, bóng đổ hoặc thân taxi vào mask người |

Khi đối chiếu với gợi ý SAM, các điểm cần kiểm tra là ranh người–xe, các phần nhỏ như bàn tay và giày, và việc giữ từng người thành một instance riêng. Mask xe phía sau chỉ bao phần xe nhìn thấy, không lấp vùng bị người che.

Quyết định giữ các mask tự vẽ trước đó giúp giữ lại phần annotation đã làm thủ công khi sử dụng thêm gợi ý SAM. Tôi có tự chỉnh sửa sau khi xem gợi ý; báo cáo chưa ghi chi tiết từng phần biên đã chỉnh và chưa xác định object đầu tiên nằm ở ảnh nào trong hai ảnh trên.

## 3. Một lỗi tìm thấy và sửa sau khi xem groundtruth

Tôi đối chiếu với groundtruth để chỉnh sửa annotation trên CVAT, sau đó Save và export lại ZIP. Các bảng định lượng dưới đây được giữ làm số liệu tham chiếu từ đợt chạy script.

**Medium — ảnh `000000458325.jpg`:** Sau khi chỉnh, số instance ghép đúng cùng class ở IoU ≥ 0,5 tăng **17 → 18**, FP giảm **9 → 8**, FN giảm **8 → 7** so với ZIP gốc. Script thử co/giãn và closing/opening theo class; closing khép một số khe hở trong mask. Mỗi class dùng chung tham số trên các ảnh, chỉ giữ thay đổi làm tăng metric tổng.

| Chỉ số Medium | ZIP gốc | Lần chỉnh 1 | Lần chỉnh 2 |
| --- | ---: | ---: | ---: |
| TP | 55 | 55 | 56 |
| FP | 29 | 29 | 28 |
| FN | 16 | 16 | 15 |
| Mean matched IoU | 0,7751 | 0,7866 | 0,7818 |
| Recall | 0,7746 | 0,7746 | 0,7887 |
| Mean matched IoU × recall | 0,6004 | 0,6094 | 0,6166 |

Mean matched IoU lần 2 thấp hơn lần 1, nhưng số match và recall tăng nên metric tổng vẫn tăng. FP/FN là mask không ghép được theo class và ngưỡng IoU; một mask sai biên có thể tạo đồng thời FP và FN, không thể coi mọi FP là vật không tồn tại. Metric Medium không phạt trực tiếp FP.

**Easy:** `sidewalk` vẫn là lớp yếu nhất, IoU tăng **0,4804 → 0,5189**; `vegetation` tăng **0,6695 → 0,7229** so với bản gốc. Script giãn mask vào vùng chưa gán nhãn, thử closing để khép khe hở và opening để loại phần nhô nhỏ. mIoU tăng **0,7820 → 0,8101**, nhưng lớp vỉa hè vẫn cần kiểm tra thêm.

**Hard:** `traffic light` giảm FP **4 → 2**, giữ TP = 3 và tăng PQ **0,358 → 0,448**. Tuy nhiên, `car` tăng PQ **0,546 → 0,615** trong khi TP giảm **25 → 21** và FN tăng **1 → 5**. Lọc mask nhỏ đã loại cả một số dự đoán khớp đúng; điểm tổng tăng không đồng nghĩa mọi object đều tốt hơn. `sidewalk` và `bicycle` vẫn chưa có match đúng.

Ba ZIP thuộc đợt chạy script đã qua kiểm tra cấu trúc và được chấm bằng scorer gốc. Không sửa scorer hoặc groundtruth trong đợt chạy đó. ZIP gốc và bản lần 1 được giữ local trong `data/_cache/`. Đây là kết quả sau xem reference trên chính các ảnh dùng để chọn tham số. Đây là kết qua export trên CVAT về.

Chi tiết: [scorecard](../../reports/tiers/SCORECARD.md), [phân tích lỗi theo class và từng ảnh](../../reports/tiers/ERROR_ANALYSIS.md), [tham số lần chỉnh 2](../../reports/tiers/reference_refinement_round2.json).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vùng hoặc class | Hai cách xử lý cân nhắc | Bằng chứng hiện có | Quyết định hoặc việc cần kiểm tra |
| --- | --- | --- | --- |
| Medium, `000000458325.jpg`, các instance trong ảnh | Giữ mask gốc hoặc khép khe hở bằng closing | Match đúng tăng 17 → 18; FP giảm 9 → 8, FN giảm 8 → 7 | Giữ bản chỉnh vì metric tổng tăng; vẫn còn 7 FN và 8 FP cần đối chiếu trực quan |
| Hard, các mask `car` nhỏ trên hai ảnh | Loại mask nhỏ để giảm FP hoặc giữ để tránh mất xe nhỏ | So với gốc, FP giảm 22 → 8 nhưng TP giảm 25 → 21, FN tăng 1 → 5 | Bản hiện tại giữ cấu hình cho PQ tổng cao hơn; cần kiểm tra lại các xe nhỏ đã bị loại |
| Hard, `sidewalk` trên hai ảnh | Tiếp tục chỉnh biên hoặc xem lại vùng/class và khả năng gộp vùng | Có 2 FP, 2 FN, TP = 0; PQ vẫn bằng 0 sau hậu xử lý | Chưa xác định nguyên nhân chỉ từ metric; cần xem trực tiếp mask và reference trước khi kết luận |
