# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét ảnh gần trùng hoặc trường hợp model không dự đoán được box: 

1. `frame_0182.jpg` — score **0.9591** — hạng **1** — score cao nhất trong danh sách, nên được ưu tiên human review.
2. `frame_0369.jpg` — score **0.9324** — hạng **2** — score cao và thuộc lô 12 ảnh được chọn.
3. `frame_0380.jpg` — score **0.9170** — hạng **3** — score cao, model còn chưa chắc chắn.
4. `frame_0326.jpg` — score **0.9155** — hạng **4** — thuộc nhóm score cao nhất.
5. `frame_0331.jpg` — score **0.9154** — hạng **5** — thuộc nhóm score cao và được chọn để rà soát.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet: `frame_0182.jpg` (score **0.9591**, hạng 1), `frame_0369.jpg` (score **0.9324**, hạng 2) và `frame_0380.jpg` (score **0.9170**, hạng 3). Các frame này đều nằm trong `outputs/selection_round1.csv` và được đưa vào `outputs/selection_round1.jpg`.

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do: Một frame có điểm cao nhưng không chọn là `frame_0058.jpg` với score khoảng **0.8256**. Điểm này vẫn khá cao nhưng thấp hơn nhóm frame được ưu tiên và còn bị giới hạn bởi ngân sách 12 ảnh.

Điều phép chọn này chưa chứng minh về chất lượng mô hình: **score uncertainty cao chỉ cho biết model chưa chắc chắn, không chứng minh rằng việc gán nhãn frame đó chắc chắn làm model tốt hơn. Chất lượng cần được kiểm tra bằng human review và đánh giá lại trên tập test.**