# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Hoài Thanh`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Không có (tuân thủ nghiêm ngặt định nghĩa xe bốn bánh; không gán xe máy, xe đạp, người đi bộ hay ảnh xe trong biển quảng cáo/gương chiếu hậu).

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                                           | Vì sao                                                                                                                                              |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại  | Giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps)                                                          | Xe chỉ tạm thời khuất tầm nhìn, quỹ đạo và vị trí sau khi xuất hiện lại liên tục và dự đoán được; giữ nguyên ID giúp bảo toàn tính nhất quán (IDF1/AssA). |
| Xe bị che lâu hơn ngưỡng trên    | Kết thúc track cũ (bật Outside ở frame cuối nhìn thấy) và tạo **track mới với ID mới** khi xe xuất hiện lại             | Quá 25 frame (~2 giây), độ bất định về vị trí quá lớn, nếu tiếp tục giữ ID cũ sẽ dễ gây gán nhầm sang xe khác hoặc tạo ID switch sai lệch.       |
| Xe rời khung hình rồi quay lại   | Mặc định: **track mới** (kết thúc track cũ bằng Outside ngay khi xe ra khỏi khung)                                      | Khi xe đã ra khỏi khung hình, chuỗi quan sát cục bộ kết thúc; khi vào lại được xem như một thực thể mới xuất hiện trong video.                     |
| Hai xe cắt nhau / chồng lên nhau | Mỗi xe **giữ nguyên ID của mình** (tuyệt đối không hoán đổi ID). Xe bị che khuất thì bbox chỉ ôm sát phần nhìn thấy được | Hai xe là hai thực thể độc lập có quỹ đạo riêng biệt; giữ đúng ID để tránh lỗi hoán đổi danh tính (ID Switch - IDSW).                               |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                                                                                                                                         |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                                                                                         |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                                                                                                                                                       |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: khi xe đạt kích thước tối thiểu khoảng 15–20 px và nhận diện được đặc trưng (bánh xe, đèn, khối thân xe), không gán đốm sáng mờ ảo. |
| Xe đang đỗ, không di chuyển            | Vẫn gán nhãn `vehicle` và duy trì track liên tục suốt thời gian xe trong khung hình; đặt keyframe cố định ôm sát thân xe, không để bbox bị xê dịch / rung lắc giữa các frame.                                       |
| Keyframe đặt dày ở đâu                 | Đặt dày (mỗi 2–5 frame) tại các đoạn xe vào cua, đổi hướng, tăng/giảm tốc độ hoặc bắt đầu/kết thúc bị che khuất; đặt thưa (10–20 frame) khi xe chạy thẳng đều để tận dụng tính năng nội suy (interpolation).       |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01` / frame 1–190 / `ID 1`
- Tình huống: Chiếc xe đỗ ở lề đường bên trái (x ~ 201, y ~ 244) hầu như đứng yên không di chuyển suốt toàn bộ 190 frame của clip. Đặc biệt ở các frame 117–125 (khi ID 5 đi qua) và 133–143 (khi ID 6 đi qua), xe bị che khuất một phần bởi xe khác.
- Quyết định: Vẫn gán nhãn `vehicle` và duy trì duy nhất một track ID 1 liên tục từ frame 1 đến 190. Đặt keyframe cố định bbox ôm sát xe lúc đứng yên; ở các frame bị che khuất một phần thì bbox chỉ ôm phần thân xe còn nhìn thấy được.
- Lý do: Tuân thủ quy tắc xe đang đỗ vẫn là `vehicle` và phải track suốt thời gian xuất hiện trong khung hình; không được ngắt quãng hay đổi ID.

### Ca 2

- Clip / frame / ID: `clip_01` / frame 6–12 / `ID 2`
- Tình huống: Xe di chuyển nhanh về phía góc dưới bên trái màn hình. Từ frame 6 đến frame 11, xe bị cắt dần bởi mép ảnh trái (`x = 0.0`), kích thước vùng nhìn thấy thu hẹp từ 156px xuống còn 36px. Đến frame 12, xe ra khỏi khung hình hoàn toàn.
- Quyết định: Từ frame 6 đến 11, bbox chạm đúng mép rìa ảnh (`x = 0.0`), không vẽ phỏng đoán phần thân xe đã ra ngoài khung hình. Ngay tại frame 12 (khi xe hoàn toàn biến mất), bấm thuộc tính `Outside` trong CVAT để kết thúc track.
- Lý do: Đúng luật bbox cắt khung hình (chỉ ôm phần nhìn thấy) và luật biên track (kết thúc ngay khi xe ra khỏi khung để tránh tạo bbox treo / false positive).

### Ca 3

- Clip / frame / ID: `clip_01` / frame 63–75 / `ID 4` và `ID 5`
- Tình huống: Tại frame 63, ID 5 xuất hiện bên cạnh ID 4. Cả hai xe di chuyển cùng chiều, chạy song song và che khuất nhau một phần (vùng bbox chồng lấn từ frame 63 đến frame 134, chỉ số IoU cao nhất đạt ~0.21 quanh frame 65–70).
- Quyết định: Duy trì độc lập 2 track riêng biệt cho từng xe, giữ nguyên ID 4 và ID 5 xuyên suốt, tuyệt đối không hoán đổi ID. Với xe bị che khuất một phần, bbox chỉ ôm phần nhìn thấy được và bật thuộc tính Occluded.
- Lý do: Hai xe là hai thực thể chuyển động độc lập; giữ đúng ID giúp tránh triệt để lỗi hoán đổi danh tính (ID Switch - IDSW), bảo đảm chỉ số IDF1 và AssA cao khi đánh giá với tập gold.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Làm rõ thời điểm bấm `Outside` khi xe rời khung hình:** Ban đầu chưa nêu rõ frame ngắt track. Viết lại rõ: Tại frame cuối cùng còn nhìn thấy xe (dù chỉ còn một phần nhỏ ở mép), bbox phải chạm sát viền ảnh (`x = 0.0` hoặc rìa ảnh); nhưng ngay ở frame kế tiếp khi xe hoàn toàn biến mất, bắt buộc phải bật thuộc tính `Outside` trong CVAT để ngắt track ngay lập tức, tránh để CVAT tự động nội suy kéo dài box treo (ghost box) gây lỗi False Positive (FP).
- **Quy định kiểm soát hiện tượng trôi hộp do nội suy (Interpolation drift):** Ban đầu để khoảng cách keyframe tự do dẫn đến việc xe đổi tốc độ hoặc đi đường hơi cong thì bbox nội suy tự động ở các frame giữa bị lệch khỏi thân xe. Viết lại rõ: Giữa hai keyframe không được cách nhau quá 10–15 frame; bắt buộc phải rà soát tua chậm từng cụm 5 frame để chèn thêm keyframe tinh chỉnh, đảm bảo bbox luôn bám khít thân xe nhằm đạt điểm cao ở tiêu chí hình học (`LocA` / `MOTP`).
- **Quy tắc bbox cho xe đỗ khi bị xe khác đi ngang qua che khuất:** Ban đầu chỉ quy định chung "xe đỗ thì giữ nguyên track". Viết lại rõ: Khi xe đỗ bị xe di chuyển khác che khuất một phần, bbox của xe đỗ không được giữ nguyên kích thước lớn đè lên xe kia, mà phải co lại chỉ ôm đúng phần nhìn thấy được của xe đỗ; chỉ kéo lại kích thước đầy đủ khi vật cản đã đi qua hoàn toàn.
