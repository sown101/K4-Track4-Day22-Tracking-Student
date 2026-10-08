# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** son **Thành viên:** Nguyễn Hoàng Sơn - 2A202602457

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

Cách thử: mỗi video chạy ByteTrack và BoT-SORT ở conf=0.30, iou=0.50 (150 frame). Sau đó, với tracker được chọn, quét conf 0.15/0.30/0.50 (giữ iou=0.50) và iou 0.40/0.50/0.70 (giữ conf=0.30); mỗi lần chỉ đổi một số. Quan sát dựa trên frame 1/50/100/150 của video thử có vẽ ID. Bản nộp chạy đủ frame.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | bytetrack | 0.30 | 0.50 | ByteTrack giữ ID 2 cho người áo đỏ phía trước suốt các frame đã xem; BoT-SORT đổi người này từ ID 2 sang ID 29 ở frame 100/150. Trong 150 frame, ByteTrack sinh 10 ID, BoT-SORT 19 ID. conf=0.50 thiếu hộp người nhỏ phía sau ở frame 50. | BoT-SORT conf=0.30 (đổi ID người áo đỏ); ByteTrack conf=0.50 (bỏ người nhỏ phía sau) |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.15 | 0.50 | BoT-SORT có ID cho người áo trắng phía trên giữa ngay frame 1 và giữ đến frame 150; ByteTrack bỏ người này ở frame 1/50. conf=0.15 thêm hộp cho người tối màu ở giữa phía trên tại frame 50. Đám đông ở xa vẫn còn nhiều người chưa có ID. | ByteTrack conf=0.30 (bỏ người áo trắng lúc đầu); BoT-SORT conf=0.50 (bỏ nhiều người trong cảnh tối) |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.15 | 0.50 | Cả hai tracker giữ ID 1 cho người áo sọc. BoT-SORT có thêm hộp cho người phía xa ở frame 100 và theo được người áo xám lớn bên trái đến frame 150. Camera di chuyển làm kích thước hộp đổi nhanh. | ByteTrack conf=0.30 (ít người được theo hơn); BoT-SORT conf=0.50 (mất hộp người áo xám bên trái ở frame 100/150) |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.30 | 0.50 | BoT-SORT có ID 4 cho người áo đỏ từ frame 1, ByteTrack thì chưa. Người xa cạnh cột kính giữ ID 17 ở frame 100/150 với BoT-SORT, còn ByteTrack đổi từ ID 19 sang ID 37. Người áo trắng giữ ID 6 ở cả hai. | ByteTrack conf=0.30 (đổi ID người cạnh cột kính); BoT-SORT conf=0.50 (mất người áo đỏ ở frame 50); conf=0.15 (người cạnh cột có dấu hiệu đổi ID, dễ bắt phản chiếu) |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.15 | 0.50 | BoT-SORT có thêm hộp cho người trên vỉa hè phải và trong vùng tối bên trái ở frame 1/50; ID 22 giữ qua frame 50/100. Xe tiến tới nên nhiều người đi ra khỏi ảnh; người xa vẫn bị bỏ sót. | ByteTrack conf=0.30 (ít hộp ở vùng tối); BoT-SORT conf=0.50 (bỏ người nhỏ và người trong bóng tối) |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
(dán output ở đây)
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

**video_1 — ByteTrack (camera tĩnh, ban ngày, mật độ vừa).** Ở video này ByteTrack giữ ID tốt hơn. Người áo đỏ phía trước giữ ID 2 suốt các frame đã xem, còn BoT-SORT đổi sang ID 29. Trong 150 frame thử, ByteTrack chỉ sinh 10 ID với độ dài track trung vị khoảng 54 frame; BoT-SORT sinh 19 ID, trung vị khoảng 25 frame. Camera đứng yên, ánh sáng tốt và người đi khá đều nên mô hình chuyển động (Kalman + IoU) dự đoán đủ chính xác; Re-ID không thêm lợi ích mà còn tách track. BoT-SORT có ưu điểm là phủ thêm vài người ở xa, nhưng ổn định danh tính quan trọng hơn cho IDF1/HOTA.

**video_4 — BoT-SORT (trong nhà, camera di chuyển, kính phản chiếu).** Ở video này BoT-SORT giữ ID tốt hơn. Người xa cạnh cột kính giữ ID 17 ở frame 100/150, còn ByteTrack đổi từ ID 19 sang ID 37. BoT-SORT cũng bắt được người áo đỏ ngay từ frame 1. Khi camera tiến tới, vị trí và kích thước hộp đổi nhanh nên chỉ dựa vào chuyển động dễ ghép sai. BoT-SORT có bù chuyển động camera và dùng ngoại hình (Re-ID), nên nối lại được người sau khi hộp nhảy. Giữ conf=0.30 để hạn chế hộp giả từ phản chiếu trên kính.

**video_2 — BoT-SORT (camera tĩnh, ban đêm, rất đông).** Ban đêm detector cho điểm thấp với người tối màu. Hạ conf xuống 0.15 cộng với Re-ID giúp giữ được người áo trắng phía trên và người tối màu ở giữa, trong khi ByteTrack bỏ họ ở các frame đầu. Đổi lại, đám đông xa vẫn còn nhiều người chưa có ID. Đây là giới hạn của detector nano ở ảnh 640 px, đổi tracker không khắc phục được.

## 4. Nếu có thêm thời gian

Đổi `--iou` 0.40/0.50/0.70 cho kết quả giống hệt nhau ở cả 5 video, có thể vì YOLO26 không dùng NMS (chạy end-to-end). Nên dành thời gian quét conf mịn hơn (0.20/0.25) thay vì quét iou. Ngoài ra, nên thử OC-SORT cho video_3/video_5 (camera rung, ít fps), và xem kỹ các frame hai người cắt nhau trong bản đủ frame, không chỉ 4 frame mẫu.
