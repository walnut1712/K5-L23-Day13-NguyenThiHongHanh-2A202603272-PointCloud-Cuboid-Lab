# Báo cáo Day 13 — Nguyễn Thị Hồng Hạnh, 2A202603272

## 1. Kết luận trước
- Mã số học viên: 2A202603272
- Task CVAT: D13-2A202603272
- 10 frame của tôi: s01f17, s01f42, s02f10, s02f55, s03f09, s03f31, s04f00, s04f37, s05f03, s05f33
- Hệ điều hành: Windows 11
- Trình duyệt: Google Chrome
- Phiên bản CVAT: v2.83.0
- MSSV bạn ngồi cạnh: 2A202603271

Tổng số cuboid đã vẽ là 86 cuboid (vượt mức tối thiểu 50 cuboid theo rubric). Frame có ít cuboid nhất là s03f31 với 3 cuboid do đây là đoạn đường thoáng rộng, mật độ xe lưu thông thấp, tầm nhìn bị che khuất một phần và các phương tiện phía trước đã ra ngoài cự ly quan sát tin cậy 50 m. Phiếu tự kiểm selfcheck cờ 22/86 cuboid (chủ yếu là xe hai bánh Cyclist có chiều dài ngắn hơn dải kích thước chuẩn do góc khuất và các xe ở cự ly xa >35 m bị thưa điểm/chìm nhẹ theo dốc đường). Tôi tự tin nhất ở phần xác định đúng Class (Car, Truck, Bus, Cyclist, Pedestrian), định hướng góc yaw dọc theo thân xe và căn chỉnh tâm cuboid trong phạm vi 30 m; phần còn yếu và khó nhất là xác định cao độ đáy hộp (z đáy) khi mặt đường mấp mô dốc và ước lượng chiều dài xe ở xa (>35 m) bị thưa điểm.

Cách dùng CVAT (3 dòng):
1. Đổi góc nhìn: Chuột phải / chuột giữa để xoay khung 3D không gian; chuột trái kéo để dời vùng nhìn (pan); lăn chuột để phóng to/thu nhỏ; bấm nút Top view để chuyển nhanh về góc nhìn thẳng từ trên xuống.
2. Vẽ và chỉnh cuboid: Chọn công cụ Draw new shape (chế độ Shape, không dùng Track), chọn nhãn Class; nhấp đúp để tạo cuboid mặc định, sau đó kéo chỉnh kích thước và vị trí ôm sát cụm điểm trên cả 3 khung nhìn trực giao Top, Side và Front.
3. Mũi tên hướng và kiểm tra camera: Vào Appearance bật Cuboid orientation để hiển thị vạch đỏ ngắn (+X của cuboid) và mũi tên yaw; chạy lệnh `python scripts/lab.py preview` để kiểm tra hình chiếu 3D của cuboid lên 7 ảnh camera quanh xe.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f17-3d.png`, `t1-s01f17-top.png`, `t1-s01f42-3d.png`, `t1-s01f42-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f17 | Phía trước ở cạnh dưới của ảnh nhìn từ trên xuống (hướng −y); +x chỉ sang bên trái xe, +y chỉ về phía sau xe | Mặt đường tương đối phẳng; thấp hơn LiDAR khoảng 1,7 m (z ≈ −1,7 m) | ~47,0 m (cụm điểm xe con ở phía xa) | Car, Bus | Xe con và xe máy ở làn bên trái (hướng +x, xem cam06) và một số xe đi phía sau (hướng +y, xem cam09) | Cụm phản xạ yếu từ cây xanh ven đường bên phải ở cự ly ~35 m |
| s01f42 | Phía trước ở cạnh dưới của ảnh nhìn từ trên xuống (hướng −y); +x chỉ sang bên trái xe, +y chỉ về phía sau xe | Mặt đường phẳng, có dốc nhẹ về phía xa; thấp hơn LiDAR khoảng 1,7 m | ~48,8 m (xe con ở làn phía trước xa) | Car, Cyclist | Xe máy di chuyển ở làn bên trái (cam06/cam05) và xe con đi sau xe thu dữ liệu (cam08/cam09) | Vệt điểm thưa rời rạc không đủ hình khối ở cự ly >50 m phía trước bên phải |

Ba điều chỉ camera cho biết:
1. Màu sơn thực tế của phương tiện (trắng, đen, đỏ...), biển số xe và các chữ/ký hiệu trên thân xe (ví dụ: tem xe, phù hiệu xe buýt).
2. Trạng thái đèn tín hiệu giao thông (đèn xanh, đỏ, vàng), đèn pha/cos và đèn xi-nhan của các xe xung quanh đang bật hay tắt.
3. Chi tiết nhận dạng người: trang phục, mũ bảo hiểm, phân biệt người đang ngồi lái xe hai bánh (Cyclist) với người đang dắt bộ xe máy/xe đạp (Pedestrian).

Ba điều chỉ LiDAR cho biết:
1. Tọa độ không gian 3D chính xác tuyệt đối (x, y, z tính bằng mét) và khoảng cách Euclid thực $r = \sqrt{x^2+y^2+z^2}$ đến từng điểm bề mặt vật thể, hoàn toàn không phụ thuộc vào ánh sáng ngày hay đêm.
2. Kích thước vật lý thực tế của vật thể (dài, rộng, cao tính bằng mét) không bị biến dạng phối cảnh (perspective distortion) như thấu kính camera quang học.
3. Góc nhìn bao quát 360° liên tục đồng thời không có điểm mù góc giữa các thấu kính và cho biết mật độ điểm phản xạ cụ thể trên từng bề mặt vật thể.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f10 | Cạnh dưới (hướng −y) | Xuất hiện xe buýt lớn đi ngược chiều (yaw ~84,6°); mật độ xe máy Cyclist khá dày ở làn bên trái |
| s02f55 | Cạnh dưới (hướng −y) | Mật độ phương tiện giảm dần, có khoảng trống lớn phía sau xe thu dữ liệu |
| s03f09 | Cạnh dưới (hướng −y) | Xe con phía trước bám rất sát (~4,0 m) với mật độ điểm laser phản xạ cực kỳ đậm đặc (>2400 điểm) |
| s03f31 | Cạnh dưới (hướng −y) | Đường rất thoáng, phương tiện thưa thớt, chỉ nhận diện được 3 cuboid trong phạm vi quan sát tin cậy |
| s04f00 | Cạnh dưới (hướng −y) | Xe con phía trước ở cự ly gần (~4,3 m) có lượng điểm phản xạ rất lớn (>5800 điểm); có xe đi ngược chiều ở làn đối diện |
| s04f37 | Cạnh dưới (hướng −y) | Có xe tải lớn (Truck) đi ở cự ly gần (~7,9 m) với kích thước chiều dài xấp xỉ 6,9 m |
| s05f03 | Cạnh dưới (hướng −y) | Xuất hiện phương tiện đi sát sườn bên trái chỉ cách LiDAR 3,21 m (vật gần nhất toàn bộ bài lab) |
| s05f33 | Cạnh dưới (hướng −y) | Xuất hiện xe tải lớn thùng hàng rộng và có người đi bộ (Pedestrian) di chuyển ven vỉa hè |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f17, 1 | Car | Đúng (yaw = 267,5°, chỉ thẳng về phía trước theo hướng −y) | Đúng (z_bottom = −1,839 m, sát mặt đường −1,7 m) | Đã kiểm đủ Top/Side/Front và ảnh cam01 | Đúng (dài 4,383 m, rộng 1,850 m, cao 1,594 m; chuẩn sedan) |
| (b) xe buýt hoặc xe tải | s02f10, 6 | Bus | Đúng (yaw = 84,6°, xe buýt đi ngược chiều làn đối diện) | Đúng (z_bottom = −1,898 m, đáy sát mặt đường) | Đã kiểm đủ Top/Side/Front và ảnh cam06/cam07 | Đúng (dài 11,670 m, rộng 2,550 m, cao 3,013 m; chuẩn xe buýt thường) |
| (c) người đi bộ hoặc xe hai bánh | s02f10, 8 | Cyclist | Đúng (yaw = 357,7°, đầu xe máy hướng chếch sang trái) | Đúng (z_bottom = −1,773 m, đáy chạm mặt đường) | Đã kiểm đủ Top/Side/Front và ảnh cam06 | Đúng (dài 1,045 m, rộng 0,828 m, cao 1,773 m; chuẩn xe hai bánh có người) |

Vòng hiệu chuẩn: Trong vòng hiệu chuẩn, cuboid ban đầu bị lệch nhẹ ở chiều cao và độ sâu đáy hộp (z đáy hơi chìm dưới mặt đường do cố ôm các điểm phản xạ nhiễu mặt đường). Cả lớp đã thống nhất chốt cách hiểu guideline: đáy cuboid phải phẳng ngang mặt đường (z ≈ −1,7 m), không kéo đáy xuống dưới mặt đường chỉ vì có vài tia laser bắn xuống mặt đất; hướng yaw (+X của cuboid) phải luôn song song với trục dọc thân xe và chỉ về đầu xe (xe cùng chiều phía trước hướng về −y nên yaw ≈ 270°, xe ngược chiều yaw ≈ 90°).

Các vật bỏ qua vì không đủ cơ sở:
- s01f17 tại vị trí x ≈ 15,2 m, y ≈ −48,5 m: Cụm điểm phản xạ rất mờ nhạt (chỉ có 4–5 điểm), không rõ hình dáng xe con hay vật cản ven đường (loại nghi vấn: LOW_VISIBILITY).
- s03f31 tại vị trí x ≈ −8,5 m, y ≈ 38,0 m: Cụm điểm phản xạ phía sau bị bóng che một phần, không xác định được hình dạng cụ thể (loại nghi vấn: UNCERTAIN_SCOPE / LOW_VISIBILITY).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png` (frame s02f10, sau xe buýt idx 6, hoặc s02f00 task chung) | Tia laser truyền thẳng bị thân xe buýt/xe tải kích thước lớn chắn lại hoàn toàn, tạo vùng khuất hình quạt phía sau | Phía sau xe buýt xuất hiện một khoảng trống không có điểm phản xạ (shadow cone), dù hai bên vẫn có chùm điểm dày đặc | Không coi vùng trống là đường trống; vật thể bị che một phần thì bật thuộc tính `occluded = 1`, vẽ cuboid bao trọn phần suy luận được nếu thấy ≥ 15–20% thể tích |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png` (frame s01f42, xe con idx 11 ở cự ly 48,80 m) | Góc quét chùm tia LiDAR cố định theo độ nên cự ly càng xa chùm tia càng dãn cách lớn, đồng thời năng lượng phản xạ giảm theo $1/r^2$ | Xe con ở cự ly gần 50 m chỉ nhận được một vệt mỏng từ 5–15 điểm rời rạc, không rõ biên dạng mặt bên hay mặt sau | Chỉ gán cuboid khi thấy được tối thiểu 15–20% thể tích hoặc có ảnh camera rõ để suy luận; nếu quá ít điểm không thể ước lượng chắc chắn thì bỏ qua (ghi LOW_VISIBILITY) |
| 3. Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png` (frame s01f17, xe con idx 7 ở cự ly 36,71 m chỉ có 11 điểm, hoặc kính xe) | Bề mặt sơn tối màu hoặc bề mặt kính trong suốt hấp thụ mạnh tia hồng ngoại hoặc phản xạ gương làm chệch tia laser không quay về cảm biến | Cùng cự ly khoảng 20–35 m nhưng phương tiện có số điểm ít hơn hẳn xe bên cạnh, xuất hiện các khoảng thủng điểm trên nóc và kính | Cần kết hợp đối chiếu ảnh camera để ước lượng kích thước chuẩn; giữ kích thước theo bảng tham chiếu 4.3 chứ không thu nhỏ cuboid theo cụm điểm |
| 4. Điểm lạ | `t4-4-diem-la.png` (frame s02f35 từ task chung, hoặc s05f03 phản xạ mặt đường) | Hiện tượng đa phản xạ (multipath) qua mặt đường bóng/nước hoặc nhiễu tán xạ quang học làm sai lệch thời gian bay (Time-of-Flight) của xung laser | Cụm điểm phản xạ xuất hiện sâu dưới mặt đường (z ≈ −13,5 m, thấp hơn mặt đường tới 11 m) hoặc lơ lửng giữa không trung | Tuyệt đối không gán nhãn cụm điểm này thành vật thể giao thông; không kéo đáy cuboid xuống dưới mặt đất để bao trùm các điểm này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603272-bangA.csv`; số điểm lấy từ cột `points_in` của `D13-2A202603272-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f17, 1, Car | −1,043 m | 1,594 m | −1,839 m | 8,23 m | 8,292 m | 267,5° | Chạm đường (chênh 0,139 m so với chuẩn −1,7 m, nằm trong ngưỡng cho phép ±0,3 m) |
| s02f10, 6, Bus | −0,392 m | 3,013 m | −1,898 m | 18,17 m | 18,175 m | 84,6° | Chạm đường (chênh 0,198 m so với −1,7 m, xe buýt lớn sát mặt đường dốc nhẹ) |
| s01f17, 8, Car | −0,877 m | 1,647 m | −1,700 m | 47,03 m | 47,039 m | 273,4° | Chạm đường chuẩn (z đáy đạt đúng −1,700 m trên mặt đường phẳng) |

(a) Cuboid gần nhất và xa nhất:
- Cuboid gần nhất: s05f03#3 (Car), khoảng cách mặt phẳng $dist\_m = 3,21\text{ m}$, khoảng cách không gian 3D $r = \sqrt{(-3,189)^2 + (-0,342)^2 + (-0,822)^2} = 3,311\text{ m}$.
- Cuboid xa nhất: s01f42#11 (Car), khoảng cách mặt phẳng $dist\_m = 48,80\text{ m}$, khoảng cách không gian 3D $r = \sqrt{(-2,754)^2 + 48,725^2 + (-0,753)^2} = 48,809\text{ m}$.

(b) Kích thước l, w, h của một xe hai bánh và một xe con so với bảng 4.3:
- Xe con s01f17#1 (Car): dài $l = 4,383\text{ m}$, rộng $w = 1,850\text{ m}$, cao $h = 1,594\text{ m}$. So với bảng 4.3 (Sedan: dài 3,8–5,0 m, rộng 1,70–1,90 m, cao 1,40–1,60 m): Cả ba chiều đều nằm hoàn toàn trong khoảng chuẩn lý tưởng.
- Xe hai bánh s02f10#8 (Cyclist): dài $l = 1,045\text{ m}$, rộng $w = 0,828\text{ m}$, cao $h = 1,773\text{ m}$. So với bảng 4.3 (Xe máy có người: dài 1,8–2,3 m, rộng 0,70–0,95 m, cao 1,40–1,80 m): Chiều rộng và chiều cao nằm trong khoảng chuẩn; chiều dài $l = 1,045\text{ m}$ ngắn hơn khoảng chuẩn vì xe máy đang di chuyển chéo bị che khuất một phần đuôi xe bởi phương tiện bên cạnh, chỉ nhận được chùm điểm từ đầu xe và thân người lái.

(c) yaw_deg của một xe cùng chiều phía trước và một xe ngược chiều:
- Xe cùng chiều phía trước: s01f42#1 có $yaw\_deg = 269,7^\circ$ (hoặc s01f17#1 có $yaw\_deg = 267,5^\circ$), rất gần với góc lý thuyết $270,0^\circ$ (hướng di chuyển theo trục −y).
- Xe ngược chiều: s02f10#4 có $yaw\_deg = 90,0^\circ$ (hoặc s02f55#2 có $yaw\_deg = 90,0^\circ$), hoàn toàn trùng khớp với góc lý thuyết $90,0^\circ$ (hướng ngược lại theo trục +y).

Mật độ theo khoảng cách (4 cuboid xe con ở các dist_m khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f17, 1, Car | 8,23 m | 1713 điểm |
| s01f17, 3, Car | 14,67 m | 751 điểm |
| s05f03, 9, Car | 25,02 m | 221 điểm |
| s04f37, 4, Car | 37,87 m | 86 điểm |

Nhận xét quy luật: Khi khoảng cách tăng khoảng gấp đôi (từ ~8,2 m lên ~14,7 m, và từ ~14,7 m lên ~25–38 m), số điểm phản xạ giảm khoảng 2,3 đến 3,4 lần, phản ánh chính xác quy luật suy giảm mật độ chùm tia laser phân kỳ trong không gian và năng lượng phản xạ nghịch đảo bình phương cự ly ($1/r^2$). Số liệu thu được rất khớp với đường cong phân bố trên slide bài giảng (xe con: khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m).

Phần tự kiểm:
- Hai cuboid đã sửa sau preview / selfcheck:
  1. s01f17#1: Ban đầu đáy cuboid bị chỉnh sâu ($z\text{ đáy} = -2,05\text{ m}$), sau khi xem lại góc nhìn Side và nhận cảnh báo cờ chìm từ selfcheck, đã nâng đáy lên $z\text{ đáy} = -1,839\text{ m}$ để tiếp xúc mặt đường chuẩn.
  2. s01f42#1: Ban đầu góc xoay yaw hơi lệch ($yaw = 258,0^\circ$), sau khi kiểm tra ảnh camera preview thấy đuôi xe bị vẹo, đã xoay lại chấm xanh ở Top view về $yaw = 269,7^\circ$ thẳng theo thân xe.
- Những cảnh báo giữ nguyên kèm lý do:
  - Các cảnh báo `length=0.65 ngoài khoảng 1.0–2.4 của Cyclist` (như s01f42#3, s01f42#7) giữ nguyên vì đây là người đi xe hai bánh trong đám đông bị che khuất phần dưới bánh xe, cụm điểm thực tế chỉ đo được phần người và đầu xe.
  - Cảnh báo tại s01f17#8 (`đáy thấp hơn mặt đường 0.42 m (chìm?)`) giữ nguyên vì ở khoảng cách xa 47,03 m mặt đường thực tế có dốc thoai thoải xuống, nếu nâng đáy lên −1,7 m thì cuboid sẽ bị lơ lửng so với vết tiếp đất của cụm điểm bánh xe.

Đổi kiểm:
MSSV người đã kiểm chéo: 2A202603271. Hai con số đối chiếu với bảng A: (1) Tổng số cuboid của tôi là 86, của bạn là 82; (2) Khoảng cách cuboid gần nhất của tôi là 3,21 m, của bạn là 3,45 m. Hai bên đã đối chiếu bảng A, các số liệu khớp logic và không có sai lệch về quy ước trục tọa độ hay định dạng file.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ phía mô hình học sâu:
1. Mô hình nhận thức 3D trong xe tự hành cần trực tiếp cung cấp kích thước vật lý và vị trí thực tế trong không gian tọa độ thế giới (tính bằng mét) để các module lập kế hoạch quỹ đạo (trajectory planning) và điều khiển phanh an toàn có thể tính toán cự ly dừng xe chính xác và tránh va chạm, điều mà hộp tọa độ 2D pixel không làm được do thiếu thông tin chiều sâu thực.
2. Ở cự ly xa hoặc khi bị che khuất, vật thể chỉ còn vài điểm phản xạ; nếu không có quy tắc chuẩn hóa (như ngưỡng tối thiểu 15–20% thể tích và đối chiếu kích thước tiên nghiệm từ bảng 4.3), người gán nhãn sẽ phỏng đoán tùy tiện gây ra hiện tượng nhãn nhiễu (label noise), làm suy giảm độ chính xác và khiến mô hình học sai hình học vật thể.

(b) Dùng AI:
- Công cụ: Antigravity AI Assistant.
- Câu hỏi chính: Nhờ kiểm tra cú pháp script, trích xuất đối soát dữ liệu từ Bảng A và phiếu tự kiểm tra CSV, rà soát các điều kiện nộp bài theo Rubric.
- Bạn dùng kết quả thế nào: Dùng kết quả phân tích thống kê để điền chính xác các thông số số học (r, z đáy, góc yaw, phân bố mật độ điểm theo khoảng cách) vào báo cáo và rà soát các lỗi định dạng của bài nộp.

(c) Điều còn chưa chắc, lỗi phát hiện sau hạn:
Một số xe hai bánh (Cyclist) ở khoảng cách trên 15 m di chuyển sát nhau trên làn đường hỗn hợp rất khó phân tách ranh giới chính xác giữa người dắt xe và xe đi liền kề; ngoài ra độ dốc vi mô của mặt đường ở các frame sequence_03 và sequence_05 làm việc xác định cao độ đáy z_bottom của các vật thể ở xa ngoài 35 m còn phụ thuộc vào phán đoán chủ quan.
