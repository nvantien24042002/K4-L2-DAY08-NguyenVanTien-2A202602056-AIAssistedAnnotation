# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyễn Văn Tiến

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Pool và test được chia theo trục thời gian thay vì ngẫu nhiên để tránh data leakage. Camera cố định nên các frame gần nhau có thể chứa cùng một chiếc xe hoặc gần như cùng một cảnh. Nếu chia ngẫu nhiên, một chiếc xe có thể xuất hiện ở cả tập train và test, làm AP50 cao hơn thực tế. Vùng đệm giúp tách test khỏi pool.

Theo dữ liệu của bài, test có 20 ảnh, buffer 112 ảnh và pool 268 ảnh. Khoảng cách tối thiểu từ pool gần test nhất là 4,4 giây.

## 2. Mô hình khởi đầu lạnh (cold start)

Cold start có AP50 = 0.771, Precision = 0.925, Recall = 0.489 và F1 = 0.640.

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
|---:|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | YOLOv8n cold start | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | YOLOv8n fine-tune | 12 | 283 | 0.741 | -0.030 | 1.000 | 0.184 | 0.310 | 0.000 | 0.176 | 0.537 |

Ở cold start, model phát hiện xe lớn và vừa tốt hơn xe nhỏ. Recall của xe nhỏ chỉ 0.182, thấp hơn nhiều so với xe vừa 0.547 và xe lớn 0.561. Các xe rất xa hoặc chỉ còn vài điểm sáng cần được xem xét cẩn thận vì nhãn tham chiếu cũng do model tạo và chưa được người kiểm tra từng box. Ngoài ra, 14 box tham chiếu cao dưới 16 px được bỏ qua khi chấm.

## 3. Chiến lược chọn mẫu

Điểm chọn mẫu được tính theo:

`score = W_U·U + W_A·A + W_D·D`

Trong đó U là độ bất định, A phản ánh độ khó khi rà nhãn và D khuyến khích sự đa dạng. Theo chiến lược của bài, các trọng số lần lượt ưu tiên U 0.5, A 0.3 và D 0.2. `MIN_GAP_S` dùng để tránh chọn nhiều frame quá gần nhau về thời gian.

Ba frame có điểm cao và được chọn là:

- `frame_0182.jpg`: score 0.9591
- `frame_0369.jpg`: score 0.9324
- `frame_0380.jpg`: score 0.9170

Ví dụ `frame_0369.jpg` được chọn nhưng các frame rất gần nó như `frame_0368.jpg` và `frame_0372.jpg` không được chọn để tránh lấy nhiều cảnh gần trùng. Một frame có score cao không được chọn cũng cho thấy uncertainty chỉ là tín hiệu ưu tiên, không chứng minh rằng sửa ảnh đó chắc chắn sẽ cải thiện model.

## 4. Các vòng học chủ động

Ở vòng 1, có 12 ảnh được rà nhãn. Model ban đầu đề xuất 169 box, sau khi sửa còn 283 box:

- 142 box giữ nguyên
- 11 box chỉnh sửa
- 16 box xoá
- 130 box thêm mới
- Accept rate: 84%

Sau fine-tune, AP50 giảm từ 0.771 xuống 0.741, giảm 0.030. Precision tăng từ 0.925 lên 1.000 nhưng Recall giảm mạnh từ 0.489 xuống 0.184. Recall xe nhỏ giảm từ 0.182 xuống 0, xe vừa giảm từ 0.547 xuống 0.176 và xe lớn giảm nhẹ từ 0.561 xuống 0.537.

Điều này cho thấy model sau vòng 1 ít false positive hơn nhưng bỏ sót nhiều xe hơn. Trong `compare_round1.jpg` có trường hợp TP giảm từ 11 xuống 3, FP giảm từ 2 xuống 0 nhưng FN tăng từ 7 lên 15.

Ba nguồn thông tin cần phân biệt: `BLIND_SCAN.md` ghi nhận quan sát trước khi xem pre-label; `REVIEW_LOG.csv` ghi các thao tác sửa nhãn trên CVAT; `round1_diff.md` thống kê thay đổi box; còn `metrics_round1.json` và `compare_round1.jpg` thể hiện kết quả model sau fine-tune.

Một trường hợp khó là xe rất xa hoặc sát đường chân trời, khi xe chỉ còn rất nhỏ hoặc chỉ thấy đèn. Những trường hợp này cần được đánh giá thận trọng thay vì mặc định xem mismatch là lỗi của model.

## 5. Kết luận và giới hạn

Sau vòng 1, AP50 giảm 0.030 từ 0.771 xuống 0.741. Precision tăng lên 1.000 nhưng Recall giảm còn 0.184, đặc biệt Recall xe nhỏ giảm xuống 0. Vì vậy chưa nên train thêm ngay. Trước tiên cần kiểm tra lại các box đã thêm, xoá và chỉnh sửa để tìm nguyên nhân model bỏ sót nhiều xe hơn.

Nếu có vòng tiếp theo, nên chú ý nhóm xe nhỏ/xa và xe kích thước trung bình, đồng thời tránh chọn nhiều frame gần trùng để giảm chi phí gán nhãn.

Kết quả có một số giới hạn: tập test chỉ có 20 ảnh, xe quá nhỏ bị bỏ qua khi chấm và nhãn test do model tạo chưa được người kiểm tra từng box. Vì vậy AP50 phản ánh mức khớp với bộ nhãn tham chiếu hiện tại, không phải độ chính xác tuyệt đối.

Nếu AP50 tiếp tục giảm, cần kiểm tra theo thứ tự: chất lượng nhãn vòng mới, các frame gần trùng và phân phối mẫu, failure cases trong ảnh so sánh, độ ổn định của test 20 ảnh; sau đó mới xem xét lại cấu hình train. Không nên sửa nhãn test để làm AP50 tăng.