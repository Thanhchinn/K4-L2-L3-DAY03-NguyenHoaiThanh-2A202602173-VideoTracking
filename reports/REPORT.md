# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Hoài Thanh - 2A202602173`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị                            |
| --------------------------------- | ---------------------------------- |
| Công cụ                           | CVAT (chế độ Rectangle Track Mode) |
| Thời gian gán `clip_02` (warm-up) | `25` phút                          |
| Thời gian gán `clip_01`           | `65` phút                          |
| Số track đã vẽ trong `clip_01`    | `8` track (ID 1 đến 8)             |
| Số keyframe trung bình mỗi track  | `~6` keyframe / track              |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đang đỗ đứng yên ven đường (Track 1) suốt clip và bị các xe khác đi ngang qua che khuất (frame 117–125 và 133–143):**
   - _Cách xử lý:_ Giữ nguyên một ID 1 duy nhất liên tục từ frame 1 đến 190. Đặt keyframe cố định vị trí bbox để tránh trôi hộp nội suy (interpolation drift). Ở các frame bị xe khác che khuất một phần, bbox chỉ ôm sát phần thân xe còn nhìn thấy được.
2. **Xe chạy nhanh chạm rìa ảnh và rời khỏi khung hình ở góc dưới bên trái (Track 2 từ frame 1 đến 11):**
   - _Cách xử lý:_ Khi xe tiến dần ra rìa ảnh, bbox kéo chạm đúng mép ảnh (`x = 0.0`), không vẽ phỏng đoán phần xe nằm ngoài màn hình. Ngay tại frame 12 khi xe hoàn toàn ra khỏi màn hình, lập tức kích hoạt thuộc tính `Outside` trong CVAT để ngắt track, tránh tạo bbox treo gây lỗi False Positive (FP).
3. **Hai xe chạy cùng chiều song song và che khuất nhau một phần (Track 4 và Track 5, frame 63–134):**
   - _Cách xử lý:_ Quản lý độc lập 2 track riêng biệt, giữ nguyên ID của từng xe (không tráo đổi ID) để tránh triệt để lỗi ID Switch (IDSW). Với xe bị che khuất một phần, bbox chỉ ôm phần nhìn thấy được và bật thuộc tính `Occluded`. Đồng thời bổ sung keyframe dày hơn (cách 3–5 frame) quanh vùng giao cắt để hạn chế lệch hộp do nội suy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (Nhìn số ID): Phát nhanh toàn bộ clip ở tốc độ cao, tập trung quan sát số ID hiển thị trên từng bbox. Kiểm tra thấy 8 ID duy trì liên tục, không có hiện tượng nhấp nháy, mất track giữa chừng hay nhảy số ID (đặc biệt là ID 4 và ID 5 khi chạy song song qua nhau quanh frame 63–75 vẫn giữ nguyên ID riêng biệt).
- Lượt 2 (Frame đầu và frame cuối): Rà soát chính xác frame bắt đầu và frame kết thúc của từng track. Phát hiện và kiểm tra kỹ Track 2 khi thoát ra ở mép trái ảnh: đã kích hoạt thuộc tính `Outside` ngay tại frame 12 (frame đầu tiên xe biến mất hoàn toàn), đảm bảo không còn bbox treo lơ lửng (ghost box).
- Lượt 3 (Frame giữa các khoảng dài): Nhảy vào các frame nằm giữa hai keyframe cách xa nhau (ở Track 1, Track 4, Track 7). Phát hiện ở Track 4 quanh frame 95 do xe hơi chuyển hướng lái nên bbox nội suy tự động hơi bị trễ so với mũi xe; đã bổ sung thêm 1 keyframe ở giữa để nắn lại bbox ôm khít thân xe, tối ưu điểm `LocA`.

Kiểm chéo với: Bạn cùng nhóm (Peer review). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `2` lỗi (1 lỗi quên bấm `Outside` khiến bbox bị treo 2 frame sau khi xe ra khỏi khung hình; 1 lỗi lệch bbox do khoảng cách keyframe quá xa).
Số lỗi bạn ấy tìm được trong bản của bạn: `1` lỗi (ở Track 1 xe đỗ, có vài frame bị xe khác đi ngang qua che khuất nhưng bbox chưa thu gọn ôm sát phần nhìn thấy được).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- Hai người có sự khác biệt ở tình huống: **Xử lý bbox của xe đỗ (Track 1) khi bị xe khác (Track 5, Track 6) đi ngang qua che khuất một phần.** Ban đầu một người giữ nguyên kích thước bbox ban đầu của xe đỗ vì cho rằng xe đứng yên; người kia thu nhỏ bbox lại chỉ ôm phần nhìn thấy được theo luật occlusion.
- Luật còn thiếu trong `GUIDELINE_MINI.md`: Lúc đầu chưa quy định cụ thể trường hợp xe đỗ bị che khuất một phần. Sau khi thảo luận kiểm chéo, nhóm đã thống nhất và bổ sung vào Mục 5 của `GUIDELINE_MINI.md`: dù xe đỗ đứng yên nhưng khi bị che khuất thì bbox vẫn phải co lại chỉ ôm phần nhìn thấy được, tránh tạo bbox đè lên xe đang di chuyển.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `8222bfaf04065524f38f82677cc4a52dbd77a5479e5c5816ca388fcea2bf2f1a` |
| Thời điểm khóa                                       | `20:34:10 15/09/2026 (2026-09-15T13:34:10 UTC)`                    |
| Số row / frame / track trước khi mở reference        | `631` rows / `190` frames / `8` tracks                             |

|              |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------ | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| Bản pre-gold | 0.658 | 0.629 | 0.703 | 0.765 | 0.945 | 0.885 | 0.717 |  62 |   4 |    0 |
| Sau rework   | 0.717 | 0.690 | 0.758 | 0.765 | 0.993 | 0.986 | 0.717 |   4 |   4 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **Có** (ĐẠT cả 3 chỉ số ngay từ bản pre-gold và nâng cao hơn sau rework; giữ vững 0 ID Switch).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi                          | Frame   | ID  | Đã sửa thế nào                                                                                        |
| --------------------------------- | ------- | --- | ----------------------------------------------------------------------------------------------------- |
| Bbox thừa (FP) do gán quá sớm     | 63–78   | 5   | Cắt bỏ 16 bbox vẽ quá sớm khi xe còn nhỏ/mờ ở xa; bắt đầu track từ frame 79 khi xe nhận diện rõ ràng. |
| Bbox thừa (FP) do gán quá sớm     | 79–100  | 6   | Cắt bỏ 22 bbox vẽ sớm trước khi xe xuất hiện rõ nét; bắt đầu track từ frame 101 theo chuẩn của gold.  |
| Bbox treo (FP) do bấm Outside trễ | 149–151 | 4   | Bấm `Outside` tại frame 149 ngay khi xe đã rời khung hình, loại bỏ 3 bbox treo ngoài rìa ảnh.         |
| Bbox treo (FP) do bấm Outside trễ | 169–171 | 8   | Bấm `Outside` tại frame 169 ngay khi xe vừa biến mất, loại bỏ 3 bbox treo dư thừa.                    |
| Bbox thừa (FP) do gán biên        | 103–105 | 7   | Bỏ 3 frame đầu còn mờ sát mép, bắt đầu track chuẩn từ frame 106 khi thấy rõ thân xe.                  |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                                                     |
| ---------------------------------- | --------------------------------------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13`                           |
| weights / hai tracker              | `yolo26n.pt` / `bytetrack.yaml` (control) & `botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes       | `0.25` / `0.7` / `960` / `[2, 5, 7]` (car, bus, truck)                      |
| device                             | `0` (GPU)                                                                   |

| So sánh                   |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| bạn vs gold               | 0.717 | 0.690 | 0.758 | 0.765 | 0.993 | 0.986 | 0.717 |   4 |   4 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 |  88 |  54 |    2 |
| BoT-SORT + ReID vs gold   | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 |  91 |  26 |    2 |
| ReID vs bạn               | 0.654 | 0.587 | 0.746 | 0.780 | 0.887 | 0.763 | 0.734 | 100 |  35 |    1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả của mình, **IDF1 cao hơn MOTA** (ở bản pre-gold: `IDF1 = 0.945` > `MOTA = 0.885`; sau rework: `IDF1 = 0.993` > `MOTA = 0.986`). Cả hai chỉ số đều đạt mức rất cao, thể hiện nhãn vừa phát hiện chính xác từng frame vừa giữ ID cực kỳ ổn định.
- Nếu một hệ thống có **MOTA cao mà IDF1 thấp**: Điều đó chứng tỏ detector tìm vật thể ở từng frame rất tốt (ít FP, ít FN, tọa độ khớp), nhưng **tính nhất quán của danh tính theo thời gian lại rất kém** — tracker liên tục bị nhảy số ID (ID Switch) hoặc track bị chia cắt thành nhiều mẩu ngắn.
- **Vì sao MOTA không phạt nặng lỗi ID?** Trong công thức tính MOTA:
  $$\text{MOTA} = 1 - \frac{\sum (\text{FN} + \text{FP} + \text{IDSW})}{\sum \text{GT}}$$
  Mỗi lần hoán đổi ID (ID Switch), MOTA chỉ phạt đúng 1 lần tại frame xảy ra sự cố. Trong hàng chục hay hàng trăm frame tiếp theo, dù đối tượng đó đang mang ID sai, MOTA vẫn tính đó là detection đúng và không trừ thêm điểm. Ngược lại, **IDF1** đo lường tỷ lệ các detection được gán đúng danh tính thực tế (Identity F1-score) trên toàn bộ quãng đời của track. Nếu một track dài 100 frame bị đổi ID ở frame 50, toàn bộ 50 frame phía sau sẽ bị phạt là IDFP/IDFN, làm IDF1 tụt dốc nghiêm trọng. Do đó, MOTA thiên về chất lượng detection từng frame, còn IDF1 mới thực sự phản ánh chất lượng duy trì danh tính lâu dài.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **So sánh số liệu:**
  - `IDF1`: BoT-SORT + ReID (`0.900`) cao hơn ByteTrack (`0.875`).
  - `AssA`: BoT-SORT + ReID (`0.820`) cao hơn ByteTrack (`0.776`).
  - `IDSW`: Cả hai tracker đều ghi nhận `2` lần nhảy ID.
  - `FN`: BoT-SORT + ReID giảm mạnh số lượng bỏ sót detection từ `54` xuống `26` (giảm hơn một nửa).
- **Dẫn chứng frame sequence:**
  - _Treatment tốt hơn:_ Ở **track gold 4** (quanh frame 54–60 khi xe mới vào khung): ByteTrack bị detector giật dẫn đến nhảy ID tại frame 59 (chuyển từ ID 14 sang 15), làm track bị phân mảnh. BoT-SORT + ReID nhờ có đặc trưng ngoại hình (appearance embedding) kết hợp với vị trí đã liên kết thành công và duy trì 1 track duy nhất xuyên suốt 95 frame.
  - _Treatment gặp khó khăn:_ Ở **track gold 5** (xe bị che một phần quanh frame 85–95): BoT-SORT + ReID bị đổi ID ở frame 87 (từ ID 17 sang 18) do vùng crop ảnh dính một phần xe bên cạnh làm biến dạng embedding ngoại hình.
- **Lưu ý phương pháp luận:** Thí nghiệm này là một **system comparison**, **KHÔNG cô lập được hiệu ứng nhân quả (causal effect) của riêng ReID**. ByteTrack và BoT-SORT là hai implementation association khác nhau (khác biệt về thuật toán matching tầng 2, ma trận hiệp phương sai Kalman Filter và hàm chi phí). Để cô lập riêng ReID, cần thực hiện ablation study so sánh BoT-SORT (`with_reid: true`) vs BoT-SORT (`with_reid: false`) trên cùng một tập siêu tham số.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- `DetA` tăng từ `0.649` (ByteTrack) lên `0.711` (BoT-SORT + ReID).
- `FP` tăng nhẹ từ `88` lên `91`.
- `FN` giảm mạnh từ `54` xuống `26` (giúp độ bao phủ tốt hơn đáng kể).
- **Lỗi còn lại chủ yếu là do DETECTOR (YOLO)**:
  - Lượng `FP` (~91 box) khá cao là do YOLO phát hiện nhầm các đối tượng tĩnh hoặc xe đỗ ngoài phạm vi quan tâm (ví dụ ghost track ID 7/10 bị vẽ liên tục 42–43 frame từ frame 16–116).
  - Lượng `FN` (26 box) xảy ra ở các frame xe ở xa có kích thước quá nhỏ hoặc bị che khuất nặng khiến YOLO không sinh ra bbox. Khi detector không phát hiện được vật thể, tracker (kể cả có ReID) cũng không có bất kỳ crop ảnh hay bounding box nào để thực hiện liên kết (association).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Clip / Frame / ID:** `clip_01` / frame 87–88 / `ID 5 trong gold` (trong model ReID là `ID 17` nhảy sang `ID 18`).
- **Bạn đúng:** Bạn gán liên tục duy nhất 1 track ID 5 xuyên suốt 60 frame (từ frame 79 đến frame 138), không bị đứt đoạn hay đổi ID (0 ID Switch).
- **ReID sai:** Model BoT-SORT + ReID bị lỗi ID Switch tại frame 87 (xe đang mang ID 17 thì bị nhảy sang gán ID 18).
- **Vì sao:** Tại frame 87, xe ID 5 đi qua khu vực bị xe ID 4 che khuất một phần và góc chiếu sáng thay đổi; visual embedding của ReID bị nhiễu do crop chứa cả phần xe bên cạnh, dẫn đến độ tương đồng cosin giảm thấp và tracker mở nhầm track mới.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Clip / Frame / ID:** `clip_01` / frame 63–78 / `ID 5`.
- **Tình huống xem lại:** Ban đầu (ở bản pre-gold), bạn bắt đầu vẽ track ID 5 từ frame 63 (tổng cộng 78 frame). Nhưng cả model ReID lẫn tập gold đều chỉ bắt đầu nhận diện/track xe này từ khoảng frame 79 trở đi (khi xe tiến lại gần hơn và rõ nét).
- **Lý do evidence:** Từ frame 63 đến 78, xe ID 5 thực chất chỉ là một chấm nhỏ mờ ở xa phía cuối đường. Bạn bắt đầu gán quá sớm khi đối tượng chưa đạt ngưỡng kích thước nhận diện tối thiểu (~15-20px), tạo ra 16 frame FP. Nhờ quan sát thấy model ReID không bắt được và đối chiếu với gold, bạn đã rút kinh nghiệm và cắt bỏ đoạn gán sớm này trong đợt rework.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`:**
  1. _Định lượng rõ ràng kích thước tối thiểu để bắt đầu track:_ Quy định ngưỡng cứng (ví dụ: chiều rộng $\ge 20$ pixel hoặc thấy rõ khối đèn/bánh xe), tránh gán quá sớm khi xe còn là đốm mờ ở chân trời gây lỗi False Positive như trường hợp xe ID 5 ở frame 63–78.
  2. _Quy định chuẩn thời điểm ngắt track:_ Bắt buộc kích hoạt thuộc tính `Outside` ngay tại frame đầu tiên xe hoàn toàn ra khỏi khung hình để tránh để lại bbox treo (ghost box).
  3. _Hướng dẫn cụ thể về xe đỗ:_ Quy định rõ khi xe đỗ bị xe di chuyển khác che khuất, bbox của xe đỗ phải co lại chỉ ôm phần nhìn thấy được, không để bbox đè lên xe kia.
- **Đổi trong quy trình làm việc:**
  1. _Tuân thủ nghiêm ngặt 3 lượt tua trước khi export:_ Lượt 1 phát nhanh 2x nhìn ID để bắt lỗi nhấp nháy/nhảy số; Lượt 2 kiểm tra kỹ frame đầu và frame cuối của từng track; Lượt 3 nhảy vào giữa các khoảng cách keyframe xa để chống hiện tượng trôi hộp (interpolation drift).
  2. _Chạy script validator ngay sau mỗi clip:_ Chạy ngay `python tools/check_mot_labels.py` sau khi hoàn tất một clip để phát hiện sớm các lỗi định dạng hoặc track đứng im bất thường trước khi chuyển sang clip tiếp theo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
