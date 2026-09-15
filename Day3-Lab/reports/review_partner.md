# Báo cáo kiểm chéo — clip_01 (bản cập nhật)

Ngày cập nhật: 15/09/2026.

## 1. Dữ liệu và phạm vi

| Bản nhãn | File | Vai trò |
| :--- | :--- | :--- |
| A | `annotations/clip_01/gt_dinhvt.txt` | Tham chiếu của bạn cùng nhóm |
| B | `annotations/clip_01/gt/gt.txt` | Bản nhãn vừa cập nhật |

Clip có 190 frame, kích thước 960 × 540. Frame trong báo cáo bắt đầu từ 1 theo MOT. Chưa xác nhận họ tên đầy đủ người gán từ tên file `dinhvt`.

Đã đọc `outputs/eval_peer.json`, nhưng file này vẫn chứa kết quả cũ (1080 bbox ở B). Vì vậy đã chạy lại trên nhãn hiện tại và đọc [eval_peer_updated.json](../outputs/eval_peer_updated.json) để lập báo cáo. Kết quả cũ được giữ làm mốc so sánh.

```powershell
python tools/evaluate_tracking.py --pred annotations/clip_01/gt/gt.txt --gt annotations/clip_01/gt_dinhvt.txt --seqinfo data/clips/clip_01/seqinfo.ini --mode peer --output outputs/eval_peer_updated.json
```

Đây là phép so hai người theo mục 4 của `GUIDE.md`; A không phải gold. Không dùng trường `gate` trong JSON để kết luận đạt/trượt bài.

## 2. Kết quả kiểm tra

### 2.1. Định dạng của hai bản hiện tại

Đã chạy lại `check_mot_labels.py` cho cả hai file.

| Nội dung             | Bản A | Bản B |
| :------------------- | ----: | ----: |
| Số bbox              | 611   | 617   |
| Số track             | 8     | 8     |
| ID                   | 1–8   | 1–8   |
| Frame có nhãn        | 1–190 | 1–190 |
| Lỗi định dạng        | 0     | 0     |
| Cảnh báo             | 1     | 1     |

Cảnh báo còn lại là bbox gần như đứng im ở ID 3: frame 1–15 trong A và 2–16 trong B. Cần xem ảnh để xác nhận xe đứng yên hay bbox treo; không xóa track chỉ vì cảnh báo này.

### 2.2. So sánh trước và sau cập nhật

| Chỉ số       | Kết quả cũ | Kết quả mới |
| :----------- | ---------: | ----------: |
| HOTA         | 0.5581     | 0.8199      |
| DetA         | 0.4694     | 0.8047      |
| AssA         | 0.6648     | 0.8383      |
| LocA         | 0.8692     | 0.8692      |
| IDF1         | 0.6919     | 0.9528      |
| MOTA         | 0.1473     | 0.9051      |
| MOTP         | 0.8604     | 0.8604      |
| FP           | 495        | 32          |
| FN           | 26         | 26          |
| ID switch    | 0          | 0           |
| Bbox của B   | 1080       | 617         |

Bản mới giảm 463 bbox và FP giảm tương ứng 463. Sáu đoạn bbox kéo dài sau khi track tham chiếu kết thúc (ID 1, 2, 4, 5, 6, 8) không còn xuất hiện trong chẩn đoán mới. Điều này cho thấy khác biệt về biên kết thúc đã giảm rõ rệt; không khẳng định thao tác cụ thể trong CVAT vì không có lịch sử chỉnh sửa.

IDF1 tăng 0.2609, MOTA tăng 0.7578. LocA và MOTP không đổi, còn danh sách 18 cặp bbox lệch vẫn giữ nguyên. MOTA hiện thấp hơn IDF1 0.0477; với IDSW = 0, phần giảm MOTA đến từ FP/FN trong phép đối chiếu.

FP = 32 là số bbox B không ghép được với A; FN = 26 là số bbox A không ghép được với B ở ngưỡng IoU 0.5. Không coi đây là số lỗi đã xác nhận của từng người.

## 3. Bất đồng còn lại và hướng xử lý

### 3.1. Thời điểm bắt đầu track

| ID A/B | Frame đầu A | Frame đầu B | Đoạn bất đồng | Số bbox |
| -----: | ----------: | ----------: | :-----------: | ------: |
| 5      | 79          | 74          | 74–78         | 5       |

**Vấn đề:** B gán xe sớm hơn A năm frame. Chưa thể kết luận bên nào sai nếu chưa xem ảnh.

**Cách xử lý:** xem frame 74–79, chọn frame đầu tiên xác định được xe bốn bánh. Nếu xe đã nhận diện được ở frame 74, bổ sung phần thiếu ở A; nếu chưa, điều chỉnh thời điểm bắt đầu của B. Ghi ngưỡng thống nhất vào `GUIDELINE_MINI.md`.

Đây là đoạn bất đồng về biên duy nhất được script liệt kê; năm bbox này không đại diện cho toàn bộ FP = 32.

### 3.2. Bbox lệch nhau

Có 18 cặp frame/ID được chẩn đoán với IoU từ 0.5 đến dưới 0.6. Giá trị 0.600 trong bảng là số làm tròn của script.

| ID A/B | Frame | IoU   |
| -----: | ----: | ----: |
| 4      | 54    | 0.538 |
| 4      | 55    | 0.577 |
| 5      | 88    | 0.514 |
| 5      | 89    | 0.506 |
| 5      | 91    | 0.600 |
| 6      | 90    | 0.512 |
| 6      | 91    | 0.538 |
| 6      | 92    | 0.563 |
| 6      | 93    | 0.536 |
| 6      | 94    | 0.511 |
| 6      | 96    | 0.527 |
| 6      | 97    | 0.561 |
| 6      | 98    | 0.592 |
| 7      | 105   | 0.507 |
| 7      | 106   | 0.586 |
| 8      | 136   | 0.596 |
| 8      | 137   | 0.588 |
| 8      | 171   | 0.544 |

| ID | Cách kiểm tra và sửa |
| --: | :--- |
| 4 | Xem frame 54–55; kiểm tra bbox giữa các keyframe và chỉnh theo phần xe nhìn thấy. |
| 5 | Xem frame 88–91; thống nhất biên bbox khi xe bị che một phần. |
| 6 | Ưu tiên rà đoạn 90–98 vì có 8 cặp lệch; thêm keyframe nếu nội suy làm bbox trôi. |
| 7 | Xem frame 105–106; so phần xe nhìn thấy và chỉnh bản chưa phù hợp. |
| 8 | Xem frame 136–137 và 171; kiểm tra bbox ở cuối track và rìa ảnh. |

Các hướng xử lý là đề xuất, chưa phải những sửa đổi đã thực hiện. Script không ghi nhận ID switch, tách track, bỏ sót toàn bộ track hoặc track tham chiếu được phủ dưới 80%. Vẫn cần xem video để xác minh ID và hình học bbox.

## 4. Reviewer checklist

`[x]` = đã xác minh; `[ ]` = còn cần xem ảnh hoặc bổ sung bằng chứng.

- [ ] **Số track khớp số xe đếm bằng mắt:** hai file cùng có 8 track, chưa kiểm đếm độc lập toàn clip bằng mắt.
- [ ] **Frame đầu/cuối hợp lý:** sáu cảnh báo kéo dài đã hết; còn bất đồng ID 5 ở frame 74–78 và cần xác minh biên theo ảnh.
- [x] **Không có ID xuất hiện hai lần trong một frame:** cả hai file qua kiểm tra định dạng với 0 lỗi.
- [ ] **Xe bị che rồi hiện lại giữ nguyên ID:** script ghi nhận 0 ID switch; chưa xem toàn bộ tình huống che khuất.
- [ ] **MOT 1.1 và số ID khớp số track gốc:** hai file đọc được theo MOT, cùng có 8 ID; chưa đối chiếu với số track trong CVAT.
- [ ] **Mọi ca mơ hồ được ghi trong guideline:** cần cập nhật và thống nhất các ca ở mục 5.

## 5. Luật cần thống nhất trong GUIDELINE_MINI.md

| Ca cụ thể | Luật cần làm rõ |
| :--- | :--- |
| ID 5, frame 74–79 | Bắt đầu từ frame đầu xác định được xe bốn bánh; ghi rõ frame hai người thống nhất. |
| ID 6, frame 90–98 | Bbox ôm phần nhìn thấy; thêm keyframe khi phần nhìn thấy hoặc chuyển động thay đổi. |
| ID 8, frame 171 | Bbox sát phần xe còn trong ảnh, không suy đoán phần ngoài rìa. |
| ID 3, frame 1–16 | Xe đỗ vẫn cần track; phân biệt xe đứng yên thật với quên `outside`. |

Tiếp tục áp dụng quy tắc đặt `outside` ở frame đầu xe không còn xuất hiện để tránh lặp lại các đoạn kéo dài của bản cũ. Những đề xuất trên chưa được coi là quyết định đã thống nhất giữa hai thành viên.

## 6. Kết luận

Bản cập nhật có mức đồng thuận cao hơn rõ rệt: **HOTA 0.8199, IDF1 0.9528, MOTA 0.9051**, hai file đều có 0 lỗi định dạng. Sáu đoạn kéo dài track của bản cũ không còn trong chẩn đoán.

Còn **1 đoạn bất đồng về thời điểm bắt đầu** và **18 cặp frame/ID có bbox lệch** cần xem lại. Ưu tiên ID 5 ở frame 74–79 và ID 6 ở frame 90–98, sau đó cập nhật guideline và đánh giá lại. Chưa đủ bằng chứng để quy số lỗi cho từng người hoặc kết luận chất lượng so với gold.
