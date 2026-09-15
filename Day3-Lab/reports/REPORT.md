# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Nguyễn Hoàng Tùng - 2A202602314

Ngày: 15/09/2026.

## 1. Quá trình gán nhãn

| Mục                               | Giá trị                                                    |
| :-------------------------------- | :--------------------------------------------------------- |
| Công cụ gán nhãn                  | CVAT                                                       |
| Thời gian gán `clip_02` (warm-up) | 10m                                                        |
| Thời gian gán `clip_01`           | 30m                                                        |
| Số track trong nhãn `clip_01`     | 8 track, 617 bbox                                          |
| Số keyframe trung bình mỗi track  | Không suy ra được từ file MOT — cần bổ sung từ công cụ gán |
| Dữ liệu clip chính                | 190 frame, kích thước 960 × 540                            |
| Nhãn đang sử dụng                 | `annotations/clip_01/gt/gt.txt`                            |

Ba tình huống cần chú ý được xác định qua kiểm chéo và đối chiếu kết quả:

| Tình huống                    | Frame / ID nhãn người                         | Kết quả và hướng xử lý                                                                                                   |
| :---------------------------- | :-------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------- |
| Kết thúc track khi xe rời ảnh | ID 1, 2, 4, 5, 6, 8 trong bản kiểm chéo cũ    | Bản cập nhật đã hết sáu đoạn kéo dài được kiểm chéo cảnh báo. Cần kiểm tra `outside` ở frame đầu xe không còn trong ảnh. |
| Xe bị che một phần            | Frame 80 / ID 5                               | Nhãn người khớp gold, model bỏ sót. Giữ ID và khoanh phần nhìn thấy; kiểm tra dày hơn quanh đoạn che khuất.              |
| Xe đứng yên đã có từ đầu clip | Frame 1 / ID 3 của nhãn người ở các frame sau | Xe trắng đang đỗ có trong gold nhưng thiếu nhãn người ở frame 1. Cần kiểm tra cả xe đỗ ngay từ đầu clip.                 |

Các hướng xử lý trên là bài học và đề xuất; không khẳng định đã sửa các lỗi phát hiện sau khi chạy model.

## 2. Tự kiểm và kiểm chéo

Chưa có nhật ký ba lượt tua bằng mắt. Kết quả kiểm tra dữ liệu tương ứng với từng lượt như sau:

| Lượt kiểm          | Bằng chứng hiện có                         | Việc cần chú ý                                            |
| :----------------- | :----------------------------------------- | :-------------------------------------------------------- |
| 1 — nhìn ID        | Nhãn người có 0 ID switch so với gold      | Vẫn kiểm tra tính liên tục khi xe bị che hoặc cắt nhau    |
| 2 — frame đầu/cuối | Kiểm chéo còn khác biệt ID 5 ở frame 74–78 | Thống nhất thời điểm bắt đầu và kết thúc track theo ảnh   |
| 3 — frame giữa     | Kiểm chéo có 18 cặp frame/ID với bbox lệch | Ưu tiên ID 6, frame 90–98; kiểm tra nội suy giữa keyframe |

Đối chiếu với file `annotations/clip_01/gt_dinhvt.txt`; chưa xác nhận họ tên đầy đủ người gán. Chi tiết tại [review_partner.md](review_partner.md), số liệu hiện tại tại [eval_peer.json](../outputs/eval_peer.json).

| Chỉ số kiểm chéo                 | Giá trị         |
| :------------------------------- | --------------: |
| HOTA                             |          0.8199 |
| IDF1                             |          0.9528 |
| MOTA                             |          0.9051 |
| Bất đồng biên track được liệt kê |          1 đoạn |
| Bbox lệch được liệt kê           | 18 cặp frame/ID |

Chưa quy các bất đồng thành số lỗi của từng người vì bản bạn cùng nhóm không phải gold. Luật còn cần làm rõ là ngưỡng bắt đầu track cho xe nhỏ/mờ, bbox phần nhìn thấy và thời điểm `outside`.

## 3. Chấm với gold — trước và sau rework

Nguồn: [eval_vs_gold.json](../outputs/eval_vs_gold.json), đối chiếu với `gold/gold/clip01/gt.txt`.

Chỉ có một kết quả chấm gold được lưu; không có đủ hai lần chấm gold để điền số trước/sau rework. Không sử dụng điểm kiểm chéo làm điểm gold trước sửa.

| Chỉ số | Kết quả gold hiện tại |
| :----- | --------------------: |
| HOTA   |                0.7974 |
| DetA   |                0.7663 |
| AssA   |                0.8376 |
| LocA   |                0.8674 |
| IDF1   |                0.9597 |
| MOTA   |                0.9162 |
| MOTP   |                0.8477 |
| FP     |                    46 |
| FN     |                     2 |
| IDSW   |                     0 |

**Qua cổng: có.** Kết quả này đã được xác nhận trước khi chạy model.

| Điều kiện | Ngưỡng | Kết quả | Đánh giá |
| :-------- | -----: | ------: | :------: |
| IDF1      | ≥ 0.80 |  0.9597 |   Đạt    |
| MOTA      | ≥ 0.75 |  0.9162 |   Đạt    |
| MOTP      | ≥ 0.70 |  0.8477 |   Đạt    |

Một số vị trí còn cần xem lại sau khi đọc chẩn đoán gold:

| Loại khác biệt         | Frame   | ID nhãn người     | Hướng xử lý đề xuất                            |
| :--------------------- | :-----: | ----------------: | :--------------------------------------------- |
| Bắt đầu sớm hơn gold   |  80–100 |                 6 | Xem lại ngưỡng bắt đầu và phần xe đủ nhận diện |
| Bắt đầu sớm hơn gold   |  74–78  |                 5 | Xác định frame đầu nhìn rõ xe bốn bánh         |
| Kết thúc muộn hơn gold | 149–152 |                 4 | Kiểm tra rìa ảnh và `outside`                  |
| Kết thúc muộn hơn gold |  44–46  |                 2 | Kiểm tra xe còn xuất hiện hay chỉ còn bbox     |
| Bỏ sót so với gold     |    1    | 3 ở các frame sau | Kiểm tra bổ sung xe trắng đang đỗ từ frame đầu |

Chưa có bản nhãn sau các đề xuất này; nhãn dùng chấm ba chiều được giữ nguyên sau khi chạy model.

## 4. Kết quả model và so sánh ba chiều

Pipeline: **YOLO26n + ByteTrack**, dùng cùng hàm `track_clip` trong script và notebook. Notebook đã lưu kết quả; thí nghiệm đổi tham số được bỏ qua với `RUN_EXPERIMENT=False`.

| Tham số                  | Giá trị                   |
| :----------------------- | :------------------------ |
| Model                    | `yolo26n.pt`              |
| Tracker                  | `bytetrack.yaml`          |
| Confidence               | 0.25                      |
| NMS IoU                  | 0.7                       |
| `imgsz`                  | 960                       |
| Lớp COCO                 | 2, 5, 7 — car, bus, truck |
| Trạng thái qua các frame | `persist=True`            |
| Kết quả model            | 607 bbox, 16 track        |

Bảng tổng hợp được trình bày theo chiều dọc để dễ đọc, giữ đủ chỉ số của notebook:

| Chỉ số   | Bạn vs gold | Model vs gold | Model vs bạn |
| :------- | ----------: | ------------: | -----------: |
| **HOTA** |  **0.7974** |    **0.7085** |   **0.6625** |
| DetA     |      0.7663 |        0.6487 |       0.6038 |
| AssA     |      0.8376 |        0.7761 |       0.7289 |
| LocA     |      0.8674 |        0.8463 |       0.8442 |
| **IDF1** |  **0.9597** |    **0.8746** |   **0.8415** |
| **MOTA** |  **0.9162** |    **0.7487** |   **0.6937** |
| MOTP     |      0.8477 |        0.8226 |       0.8182 |
| FP       |          46 |            88 |           88 |
| FN       |           2 |            54 |           98 |
| IDSW     |           0 |             2 |            3 |

Nguồn: [bạn vs gold](../outputs/eval_vs_gold.json), [model vs gold](../outputs/eval_model_vs_gold.json), [model vs bạn](../outputs/eval_model_vs_me.json). Cổng annotation chỉ áp dụng cho nhãn người so với gold.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA = 0.9162 thấp hơn IDF1 = 0.9597 một khoảng **0.0435**. Với 0 ID switch, 46 FP và 2 FN, phần giảm MOTA của nhãn hiện tại đến từ bbox thừa/không khớp và bỏ sót.

MOTA tính theo `1 − (FP + FN + IDSW) / số bbox gold`. Một lần đổi ID chỉ cộng một sự kiện vào IDSW, dù ID mới tiếp tục tồn tại nhiều frame. IDF1 đánh giá sự nhất quán của ánh xạ ID trên toàn chuỗi, nên việc chia một xe thành nhiều ID có thể làm IDF1 giảm mạnh hơn. Vì vậy MOTA cao nhưng IDF1 thấp là dấu hiệu cần kiểm tra lỗi liên kết ID; trường hợp hiện tại không có dạng chênh lệch này.

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

So với gold, DetA = 0.6487 và AssA = 0.7761, chênh **0.1274**. DetA thấp hơn cho thấy khâu phát hiện/định vị là điểm yếu hơn: model có 88 FP và 54 FN. Tuy vậy, liên kết ID vẫn có lỗi:

| Frame | ID gold | ID model trước → sau | Chẩn đoán |
| ----: | ------: | :------------------: | :-------- |
|    59 |       4 |       14 → 15        | ID switch |
|    94 |       5 |       23 → 32        | ID switch |

Gold có 8 track, model có 16 track. Chẩn đoán cho thấy các gold ID 4, 5, 7 được phủ bởi nhiều ID model; số track tăng còn có đóng góp từ các track không khớp gold. Không thể quy toàn bộ 16 track thành lỗi tách track.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

**Frame 80, nhãn người ID 5 ↔ gold ID 5.** Ảnh cho thấy xe con ở phía bên trái đầu xe buýt, bị xe buýt che một phần. Bbox người đạt IoU **0.513** với gold, vượt ngưỡng ghép 0.5. Không bbox model nào đạt ngưỡng với xe này; IoU cao nhất chỉ khoảng **0.013**.

Đây là ví dụ model bỏ sót xe bị che theo gold. Người vẫn xác định được phần xe nhìn thấy. Bbox người còn có thể chỉnh khít hơn, nhưng đã phát hiện đúng vật thể ở ngưỡng đánh giá.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

**Frame 1, model ID 3 ↔ gold ID 1.** Xe trắng đang đỗ ở phía trên bên trái ảnh đã hiện rõ. Model khoanh xe với IoU **0.760** so với gold. Nhãn người tại frame 1 chỉ có ID 1 và 2 dành cho hai xe đang chạy, không có bbox cho xe trắng; xe này mang ID 3 trong nhãn người ở các frame sau.

Vì xe đỗ vẫn thuộc lớp `vehicle`, đây là một frame bị bỏ sót trong nhãn người. Cần kiểm tra frame đầu của mọi track, kể cả xe đứng yên. Các số ID giữa ba bộ nhãn không cần giống nhau; kết luận dựa trên vị trí bbox và ánh xạ theo vật thể.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

Theo hàm `disagreement_by_frame` trong notebook, tổng trên 190 frame là:

| Loại bất đồng                        | Số lượng |
| :----------------------------------- | -------: |
| **Chỉ nhãn người có bbox**           |   **98** |
| Chỉ model có bbox                    |       88 |
| Cặp khớp nhưng lệch ánh xạ ID ưu thế |        0 |

Đây là số bbox/cặp qua các frame, không phải số xe. Nguồn lưu tại [model_disagreements.json](../outputs/model_disagreements.json).

Loại chỉ người có nhiều nhất, hơn loại chỉ model có 10 bbox. Frame 80 minh họa việc che khuất làm model bỏ sót. Tuy nhiên không thể kết luận cả 98 bbox đều là model sai: nhãn người cũng có 46 FP khi so với gold và có các đoạn bắt đầu sớm hơn gold. Cần xem từng bất đồng theo gold và ảnh.

Số 0 ở cột lệch ánh xạ không chứng minh model không đổi ID. Hàm notebook cho phép nhiều ID model cùng ánh xạ về một ID người, nên không bắt đầy đủ tách track. Phép CLEAR MOT vẫn ghi nhận **3 ID switch khi model so với người**, và **2 khi model so với gold**.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi đề xuất bổ sung các quy tắc sau vào `GUIDELINE_MINI.md`, kèm ảnh/frame làm ví dụ để người gán tiếp theo áp dụng thống nhất:

| Quy tắc cần làm rõ                   | Ca minh họa                         | Cách áp dụng                                                         |
| :----------------------------------- | :---------------------------------- | :------------------------------------------------------------------- |
| Xe đỗ phải được gán từ khi xuất hiện | Frame 1, gold ID 1                  | Kiểm tra toàn ảnh ở frame đầu; không chỉ tìm xe chuyển động          |
| Bắt đầu track khi đủ nhận diện       | ID người 5, frame 74–79             | Thống nhất frame đầu xác định được xe bốn bánh                       |
| Bbox ôm phần nhìn thấy               | ID người 5, frame 80                | Không đoán phần bị che; thêm keyframe quanh thời điểm che khuất      |
| Giữ ID qua che khuất ngắn            | Gold ID 4/5, model đổi ID tại 59/94 | Theo luật lab: giữ ID nếu che dưới 25 frame và xác định được cùng xe |
| Xe ra khỏi khung rồi quay lại        | Quy tắc chung của lab               | Kết thúc track cũ và tạo track mới                                   |
| Kết thúc đúng frame                  | ID người 4, frame 149–152           | Đặt `outside` khi xe không còn trong ảnh; rà frame đầu/cuối          |

Quy trình đề xuất: gán độc lập → ghi ca mơ hồ ngay lúc gán → kiểm ba lượt → kiểm định dạng → kiểm chéo → chấm gold → khóa bản dùng so sánh → chạy model. Sau khi xem model, ghi đề xuất sửa thành phiên bản riêng để giữ được bằng chứng trước/sau. Các quy tắc trên là đề xuất cho lần gán tiếp theo, chưa phải quyết định đã được cả nhóm xác nhận.

## 7. Tệp đã nộp

Checklist dưới đây phản ánh file đang có trong workspace, **chưa xác nhận đã commit/push hoặc nộp**.

- [x] Nhãn chính: `annotations/clip_01/gt/gt.txt` — khác đường dẫn chuẩn trong mẫu là `annotations/clip_01/gt.txt`.
- [x] Nhãn warm-up: `annotations/clip_02/gt/gt.txt` — khác đường dẫn chuẩn trong mẫu là `annotations/clip_02/gt.txt`.
- [ ] `GUIDELINE_MINI.md` đã điền — hiện còn nội dung mẫu; đề xuất bổ sung ở mục 6.
- [x] `outputs/eval_vs_gold.json`.
- [x] `outputs/model_clip_01.txt`.
- [x] `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json`.
- [x] `outputs/model_disagreements.json`.
- [x] `reports/review_partner.md`.
- [x] `reports/REPORT.md` (file này).

Thông tin cá nhân, thời gian gán nhãn và số keyframe cần người thực hiện bổ sung trước khi nộp. Các chỉ số và ví dụ frame/ID trong báo cáo lấy từ dữ liệu đã lưu, không ước lượng các thông tin còn thiếu.
