# Đối chiếu lỗi với groundtruth — 12/11/2026

Các số liệu dưới đây do scorer tính từ mask, không phải ghi nhận thao tác vẽ tay. Ba ZIP được chỉnh trực tiếp bằng code sau khi xem groundtruth; chưa Save/export lại trong CVAT. Tham số được chọn trên chính tập ảnh chấm nên điểm này là kết quả hiệu chỉnh trên tập đã biết.

| Task | ZIP gốc | Lần 1 | Lần 2 |
| --- | ---: | ---: | ---: |
| easy_semantic | 17.0 | 17.8 | 18.2 |
| medium_instance | 14.3 | 14.9 | 15.4 |
| hard_panoptic | 13.3 | 15.4 | 15.8 |

## Easy: thiếu phủ vùng và sai ranh giới theo class

| Class | IoU gốc | IoU lần 1 | IoU lần 2 |
| --- | ---: | ---: | ---: |
| road | 0.9483 | 0.9818 | 0.9872 |
| sidewalk | 0.4804 | 0.5064 | 0.5189 |
| building | 0.8429 | 0.8429 | 0.8471 |
| vegetation | 0.6695 | 0.6972 | 0.7229 |
| sky | 0.9688 | 0.9742 | 0.9742 |

Giãn mask chỉ vào vùng chưa gán nhãn, kết hợp closing để nối khe hở và opening để loại phần nhô nhỏ. Đây là điều chỉnh hình thái học, không bổ sung khả năng nhận diện vật mới.

## Medium: số vật thiếu/thừa và biên chưa khớp

| Phiên bản | TP | FP | FN | Mean matched IoU | Recall |
| --- | ---: | ---: | ---: | ---: | ---: |
| Gốc | 55 | 29 | 16 | 0.7751 | 0.7746 |
| Lần 1 | 55 | 29 | 16 | 0.7866 | 0.7746 |
| Lần 2 | 56 | 28 | 15 | 0.7818 | 0.7887 |

FP/FN ở đây là số mask không ghép được ở IoU ≥ 0,5 cùng class; một mask sai biên cũng có thể tạo đồng thời FP và FN. Không thể kết luận mọi FP là vật không tồn tại. Scorer Medium dùng mean matched IoU × recall, nên FP không bị phạt trực tiếp trong metric này.

### Vị trí cần kiểm tra tiếp theo từng ảnh

| Ảnh | TP gốc → mới | FP gốc → mới | FN gốc → mới |
| --- | ---: | ---: | ---: |
| 000000181542.jpg | 17 → 17 | 9 → 9 | 3 → 3 |
| 000000373353.jpg | 21 → 21 | 11 → 11 | 5 → 5 |
| 000000458325.jpg | 17 → 18 | 9 → 8 | 8 → 7 |

## Hard: lỗi nhận diện và chất lượng ranh giới

| Class | PQ gốc → mới | TP gốc → mới | FP gốc → mới | FN gốc → mới |
| --- | ---: | ---: | ---: | ---: |
| bicycle | 0.000 → 0.000 | 0 → 0 | 0 → 0 | 1 → 1 |
| building | 0.642 → 0.654 | 2 → 2 | 0 → 0 | 0 → 0 |
| bus | 0.364 → 0.455 | 1 → 1 | 1 → 0 | 2 → 2 |
| car | 0.546 → 0.615 | 25 → 21 | 22 → 8 | 1 → 5 |
| motorcycle | 0.339 → 0.339 | 1 → 1 | 0 → 0 | 3 → 3 |
| person | 0.180 → 0.194 | 2 → 2 | 3 → 2 | 7 → 7 |
| road | 0.399 → 0.433 | 1 → 1 | 1 → 1 | 1 → 1 |
| sidewalk | 0.000 → 0.000 | 0 → 0 | 2 → 2 | 2 → 2 |
| sky | 0.939 → 0.959 | 2 → 2 | 0 → 0 | 0 → 0 |
| traffic light | 0.358 → 0.448 | 3 → 3 | 4 → 2 | 1 → 1 |
| truck | 0.413 → 0.472 | 2 → 2 | 2 → 1 | 2 → 2 |
| vegetation | 0.618 → 0.677 | 2 → 2 | 0 → 0 | 0 → 0 |

Đã thử lọc mask nhỏ theo class và co/giãn, closing/opening. Chỉ giữ phép biến đổi khi metric tổng tăng. Các lớp còn FN hoặc PQ thấp vẫn cần bổ sung/tách instance và kiểm tra ranh giới; hậu xử lý chưa giải quyết được vật bị mô hình bỏ sót hoàn toàn.

## Công cụ và bằng chứng

- Chỉnh sửa: `refine_tiers_reference.py` (đã lưu local trong `data/_cache/archived_scripts/`); giữ nguyên scorer và groundtruth.
- Chi tiết phép biến đổi đã chọn: `reference_refinement_round2.json`.
- Kết quả cuối: `SCORECARD.md`. Các ảnh preview đã được dọn vì không bắt buộc trong bài nộp.
- ZIP gốc và bản lần 1 được giữ local trong `data/_cache/`.
- COCO dùng cả polygon và RLE; kiểm tra cấu trúc không chứng minh đã import thành công vào CVAT.
