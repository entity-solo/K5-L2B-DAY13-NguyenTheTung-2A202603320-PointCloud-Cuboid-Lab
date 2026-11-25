# Báo cáo Day 13 — Nguyễn Thế Tùng, 2A202603320

## 1. Kết luận trước
Mã số sinh viên: 2A202603320 · Task CVAT: `D13-2A202603320` · Tập 10 frame: `s01f28, s01f45, s02f02, s02f42, s03f10, s03f56, s04f17, s04f57, s05f00, s05f48`.
Môi trường thực hành: Hệ điều hành Windows 11, trình duyệt Google Chrome, phiên bản CVAT 2.83.1 (Community). Bạn cùng bàn đối kiểm: 2A202603318.
Tổng số cuboid đã gán nhãn: 56 cuboid trên toàn bộ 10 frame (frame 1 đạt 8 cuboid, 9 frame còn lại đạt từ 5–6 cuboid/frame). Các frame có 5 cuboid (mức tối thiểu yêu cầu) gồm `s01f45, s02f02, s02f42, s03f10, s03f56, s05f00` do mật độ phương tiện thực tế trong phạm vi quan sát rõ (<= 30 m) không nhiều hoặc các vật thể ở quá xa/quá thưa điểm không đủ cơ sở hình học để suy luận chắc chắn.
Công cụ `selfcheck` phát hiện 9 cuboid có cảnh báo (chủ yếu là cảnh báo kích thước chiều rộng/chiều dài của một số xe SUV, minivan hoặc xe tải).
Về độ tin cậy: Tự tin nhất ở góc hướng yaw (mũi tên đỏ +X dọc thân xe) và độ cao mặt đáy (z đáy tiếp xúc sát mặt đường ~ -1.7 m, không bị lơ lửng hay chìm) nhờ luôn đối chiếu đồng thời trên cả 3 khung nhìn trực giao Top/Side/Front. Phần còn yếu nhất là việc xác định chính xác chiều dài và vị trí tâm dọc thân xe của các vật thể ở khoảng cách xa (> 25 m) do mây điểm LiDAR bị thưa thớt, chỉ thấy được vài điểm phản xạ ở một mặt nên phải dựa nhiều vào kích thước chuẩn trong Bảng 4.3 và đối chiếu ảnh camera.

Cách dùng CVAT:
- Đổi góc nhìn: Giữ chuột phải hoặc dùng các phím điều hướng (U, I, O, J, K, L) để xoay phối cảnh 3D; lăn con trỏ chuột giữa để phóng to/thu nhỏ; kéo chuột trái để di chuyển góc nhìn (pan); bấm phím nhìn từ trên xuống (Top-down view) để căn chỉnh hướng và kích thước dài/rộng.
- Vẽ và chỉnh một cuboid: Chọn chế độ Shape -> nháy đúp hoặc kéo chuột để tạo cuboid 3D mới; quan sát và tinh chỉnh đồng thời trên cả 3 khung nhìn trực giao (Top, Side, Front) ở góc dưới màn hình để kéo 6 mặt hộp ôm sát mây điểm của vật.
- Bật mũi tên hướng và kiểm tra: Tích chọn "Cuboid orientation" ở bảng Appearance để hiển thị trục tọa độ (+X đỏ ngắn dọc thân xe); định kỳ chạy `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera nhằm kiểm tra trực quan độ khớp thực tế.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f28` và `s01f45`. Ảnh minh họa: `t1-s01f28-3d.png`, `t1-s01f28-top.png`, `t1-s01f45-3d.png`, `t1-s01f45-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f28 | Phía trước xe ở cạnh dưới (-y); trục +x chỉ sang bên trái xe (cạnh trái); trục +y chỉ về phía sau xe (cạnh trên) | Phẳng, hơi dốc nhẹ về phía trước; mặt đường thấp hơn tâm LiDAR khoảng 1.70 m | ~35.5 m (xe con ở làn ngược chiều phía xa) | Car (sedan, SUV), Truck (xe tải/bán tải) | Phía sau có xe con cùng chiều bám đuôi (~18 m); bên trái có hàng xe chạy ngược chiều; bên phải có dải phân cách và cây xanh | Dải mây điểm rải rác ở khoảng cách >45 m bên lề đường không rõ là xe đỗ hay chướng ngại vật tĩnh |
| s01f45 | Phía trước xe ở cạnh dưới (-y); trục +x chỉ sang trái (cạnh trái); trục +y chỉ ra sau (cạnh trên) | Bằng phẳng; mặt đường thấp hơn tâm LiDAR khoảng 1.72 m | ~25.0 m (các xe con di chuyển cùng chiều phía trước) | Car, Cyclist (xe máy có người lái), Pedestrian (người đi bộ) | Phía trước-trái có người đi bộ (~14.1 m) và người lái xe máy (~10.3 m); phía trước có 3 xe con cùng chiều; phía sau thoáng | Cụm điểm phản xạ rải rác ngoài mép vỉa hè phía trước-trái do cây cối che khuất một phần |

Ba điều chỉ camera cho biết:
1. Màu sắc xe, thông tin biển số và đèn tín hiệu giao thông (LiDAR chỉ cung cấp tọa độ không gian và cường độ phản xạ dạng số).
2. Chi tiết nhận dạng class khi điểm thưa (ví dụ phân biệt rõ người đi bộ dắt xe đạp so với người đi bộ thông thường, hoặc kiểu dáng xe sedan vs hatchback).
3. Hướng mặt thực tế của người đi bộ hoặc nhận diện chi tiết đầu/đuôi xe khi xe bị che khuất một phần chỉ lộ bề mặt phẳng.

Ba điều chỉ LiDAR cho biết:
1. Khoảng cách hình học 3D chính xác từng centimet từ xe thu dữ liệu đến mọi điểm trên bề mặt vật thể (camera đơn mắt khó ước lượng độ sâu chính xác tuyệt đối).
2. Kích thước vật lý thực tế 3D (dài, rộng, cao tính bằng mét) độc lập hoàn toàn với hiệu ứng phối cảnh quang học và góc chụp của ống kính camera.
3. Vị trí chính xác và bao quát toàn cảnh 360° xung quanh xe kể cả ở góc mù giữa các camera hoặc trong điều kiện ánh sáng chói chang, bóng râm hay ban đêm.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f02 | Cạnh dưới (-y) | Có một xe buýt (Bus) cỡ lớn vượt sát bên trái xe, mật độ điểm phản xạ cực kỳ dày đặc (>16.000 điểm) |
| s02f42 | Cạnh dưới (-y) | Đoạn đường hơi cong nhẹ, các phương tiện ở xa (30–40 m) mật độ điểm rất thưa |
| s03f10 | Cạnh dưới (-y) | Mật độ giao thông đông, nhiều xe con bám sát nhau cả phía trước lẫn phía sau trong phạm vi 10 m |
| s03f56 | Cạnh dưới (-y) | Có xe buýt lớn ở làn đối diện cách ~30 m và người đi xe máy (Cyclist) ở làn bên cạnh |
| s04f17 | Cạnh dưới (-y) | Tuyến đường nhiều làn xe, nhiều ô tô con chạy song song cùng chiều |
| s04f57 | Cạnh dưới (-y) | Xuất hiện xe chạy ở khoảng cách xa (>35 m) ở làn đối diện |
| s05f00 | Cạnh dưới (-y) | Có xe tải (Truck) lớn chạy phía sau đuôi xe (~10 m) phản xạ mây điểm rất rõ nét |
| s05f48 | Cạnh dưới (-y) | Khu vực giao lộ rộng, có xe con chạy chuyển làn và xe tải lớn ở góc xa phía sau (~47 m) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh họa: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f28, idx 1 | Car | Đúng (yaw = 270.0°, mũi đỏ dọc thân xe hướng về trước) | Chạm đường (z đáy = -1.80 m, sát mặt đường -1.7 m) | Đã kiểm tra đầy đủ cả 3 view và ảnh camera | L: 5.05 m, W: 2.09 m, H: 1.66 m (phù hợp SUV/sedan cỡ lớn) |
| (b) xe buýt hoặc xe tải | s01f28, idx 4 | Truck | Đúng (yaw = 90.2°, ngược chiều) | Chạm đường (z đáy = -2.06 m) | Đã kiểm tra đầy đủ cả 3 view và ảnh camera | L: 5.98 m, W: 2.41 m, H: 2.42 m (phù hợp xe tải nhẹ/thùng) |
| (c) người đi bộ hoặc xe hai bánh | s01f45, idx 4 | Cyclist | Đúng (yaw = 280.1°, cùng chiều) | Chạm đường (z đáy = -1.35 m, trên mặt đường) | Đã kiểm tra đầy đủ cả 3 view và ảnh camera | L: 1.66 m, W: 0.61 m, H: 1.73 m (phù hợp xe hai bánh có người lái) |

Vòng hiệu chuẩn:
Qua buổi trao đổi hiệu chuẩn cùng lớp và coach, bài học quan trọng nhất là việc xác định hướng mũi tên trục tọa độ của cuboid trong CVAT. Mũi tên đỏ ngắn (`+X` cuboid) luôn phải hướng theo hướng di chuyển của phương tiện (hoặc hướng đầu xe khi đứng yên), không được để nằm ngang thân xe. Đối với xe di chuyển cùng chiều phía trước xe thu dữ liệu, góc `yaw_deg` phải xoay quanh mốc 270°; đối với xe chạy ngược chiều đối diện, góc `yaw_deg` phải xoay quanh mốc 90°. Mặt đáy của cuboid cần được hạ sát mặt đường (z đáy xấp xỉ -1.7 m), không để lơ lửng hoặc chìm sâu dưới lòng đất.
Về các vật bỏ qua vì không đủ cơ sở: Tại frame `s02f42`, có một cụm mây điểm nhỏ khoảng 8–10 điểm ở lề đường bên phải cách khoảng 48 m (thuộc dạng nghi vấn: `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`); do số lượng điểm quá ít (dưới 15% thể tích xe) và ảnh camera bị mờ ở cự ly xa không thể xác định chắc chắn là ô tô hay vật thể tĩnh ven đường nên đã bỏ qua theo đúng quy tắc guideline để tránh tạo nhãn sai.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f28 / `t4-1-bong-che.png` / Task cá nhân | Chùm tia laser LiDAR bị thân xe đi trước cản lại, tạo ra một vùng bóng tối (shadow) hình dẻ quạt phía sau khiến xe phía sau hoặc phần trước của chính xe đó bị mất điểm phản xạ | Chỉ quan sát được các điểm phản xạ ở mặt sau và một cạnh sườn xe; mặt trước và phần lòng trong của xe hoàn toàn rỗng; sau xe có vệt khuyết điểm kéo dài | Bật thuộc tính `occluded` khi vật bị che khuất một phần bởi vật khác; áp dụng kích thước chuẩn trong Bảng 4.3 để nội suy bao trọn kích thước thực của xe thay vì chỉ bọc phần điểm nhìn thấy |
| 2. Thưa ở xa | s02f42 / `t4-2-thua-o-xa.png` / Task cá nhân | Các tia laser phát ra theo góc mở rẻ quạt cố định, khoảng cách càng xa thì khoảng cách giữa các tia càng giãn rộng, số tia bắn trúng bề mặt vật thể giảm tỉ lệ nghịch với bình phương khoảng cách | Vật thể ở khoảng cách >30 m chỉ nhận được 2–3 đường quét (ring), tổng số điểm chỉ còn vài chục điểm rải rác thay vì hàng nghìn điểm như ở cự ly gần | Áp dụng quy tắc 15–20% thể tích; nếu nhận diện được hình dạng cơ bản thì dựng cuboid theo kích thước tham chiếu Bảng 4.3; nếu quá thưa và không đủ bằng chứng đối chiếu camera thì không vẽ |
| 3. Phản xạ thấp / ít điểm bất thường | s01f45 / `t4-3-phan-xa-thap.png` / Task cá nhân | Kính chắn gió ô tô, bề mặt sơn đen bóng hoặc vũng nước có tính chất phản xạ gương hoặc hấp thụ bước sóng laser 905 nm, làm tia bị tán xạ lệch hướng không phản xạ ngược lại đầu thu LiDAR | Phần kính chắn gió trước/sau và nóc kính của xe ô tô bị "thủng lỗ" hoặc không có điểm nào; mặt đường nhẵn bị khuyết điểm | Nhận biết đó là hiện tượng vật lý bình thường, không co hẹp cuboid lại theo các lỗ thủng này mà phải bao trọn chiều cao xe (tính cả phần kính và mui xe) |
| 4. Điểm lạ | s05f48 / `t4-4-diem-la.png` / Task cá nhân | Do bụi đường, hơi ẩm, hoặc tia laser phản xạ nhiều lần (multipath) qua mặt đường gồ ghề / gầm xe phản xạ ngược lại cảm biến | Xuất hiện một số điểm phản xạ trôi nổi sâu dưới mặt đường (z < -2.2 m) hoặc lơ lửng giữa khoảng trống giữa các xe | Không bao các điểm này vào cuboid; loại bỏ các điểm nhiễu lơ lửng để tránh làm biến dạng kích thước chiều cao hoặc làm chìm đáy cuboid xuống dưới lòng đất |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất từ `submission/D13-2A202603320-bangA.csv` và `submission/D13-2A202603320-tukiem.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f28, idx 1, Car | -0.971 m | 1.660 m | -1.801 m | 6.89 m | 6.96 m | 270.0° | Chạm đường (z đáy -1.80 m rất sát mặt đường chuẩn -1.7 m) |
| s01f45, idx 1, Car | -1.064 m | 1.751 m | -1.939 m | 10.36 m | 10.41 m | 268.3° | Chạm đường (lốp xe tiếp xúc tốt với mặt đường) |
| s01f45, idx 5, Pedestrian | -0.306 m | 1.570 m | -1.091 m | 14.13 m | 14.13 m | 258.3° | Chạm đường (người đi bộ đi trên vỉa hè cao hơn lòng đường ~0.6 m) |

(a) Cuboid gần nhất và xa nhất:
- Cuboid gần nhất: `s02f02, idx 1, Bus` ở khoảng cách `dist_m = 3.48 m`, `r = 3.53 m` (hoặc xe con gần nhất là `s03f10, idx 3, Car` ở `dist_m = 4.24 m`, `r = 4.36 m`).
- Cuboid xa nhất: `s05f48, idx 4, Car` ở khoảng cách `dist_m = 77.42 m`, `r = 77.43 m`. Trong phạm vi chấm điểm (<= 50 m), cuboid xa nhất là `s05f48, idx 6, Truck` ở `dist_m = 46.96 m`, `r = 46.96 m`.

(b) Kích thước l, w, h so với Bảng 4.3:
- Người đi bộ `s01f45, idx 5, Pedestrian`: L = 0.50 m, W = 0.58 m, H = 1.57 m. Cả 3 chiều nằm hoàn toàn trong khoảng chuẩn của Bảng 4.3 (Dài: 0.4–0.9 m, Rộng: 0.5–0.9 m, Cao: 1.50–1.95 m).
- Xe con `s01f28, idx 1, Car`: L = 5.05 m, W = 2.09 m, H = 1.66 m. Chiều dài L (5.05 m) và chiều rộng W (2.09 m) nằm ở ngưỡng trên của dải chuẩn SUV/minivan do xe thực tế là loại SUV 7 chỗ cỡ lớn và kích thước hộp bao gồm cả gương chiếu hậu hai bên.

(c) Góc yaw của xe cùng chiều và ngược chiều:
- Xe cùng chiều phía trước (`s01f45, idx 2, Car`): `yaw_deg = 269.6°`, rất sát với mốc lý thuyết 270.0°.
- Xe ngược chiều đối diện (`s01f28, idx 4, Truck`): `yaw_deg = 90.2°`, rất sát với mốc lý thuyết 90.0°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f28, idx 1, Car | 6.89 m | 2130 |
| s01f28, idx 3, Car | 13.97 m | 1004 |
| s01f28, idx 7, Car | 21.24 m | 359 |
| s01f28, idx 8, Car | 35.65 m | 120 |

Quy luật mật độ và tự kiểm:
- Quy luật mật độ: Khi khoảng cách tăng gấp đôi (từ 6.89 m lên 13.97 m), số điểm LiDAR rơi trên thân xe giảm hơn một nửa (từ 2130 điểm xuống 1004 điểm); khi khoảng cách tăng gấp 5 lần (~35.65 m), số điểm suy giảm mạnh chỉ còn khoảng 1/18 (~120 điểm). Kết quả này hoàn toàn tương thích với đường cong lý thuyết suy giảm mật độ theo bình phương khoảng cách đã học trên lớp (xe con: ~1968 điểm ở 0–10 m, ~718 điểm ở 10–20 m, ~214 điểm ở 20–30 m, ~72 điểm ở 30–40 m).
- Tự kiểm và chỉnh sửa:
  1. Đã sửa cuboid `Car` ở `s01f28`: Ban đầu bị xoay nhầm trục 90° dẫn đến L = 2.07 m < W = 3.39 m. Sau khi phát hiện qua `selfcheck`, đã xoay lại yaw về 270° để vạch đỏ +X dọc theo thân xe (kích thước sau sửa: L = 5.05 m, W = 2.09 m).
  2. Đã sửa cuboid `Truck` ở `s05f48, idx 6`: Ban đầu vẽ hộp quá rộng (W = 4.46 m), sau khi kiểm tra lại trên view Front/Top đã thu hẹp lại về W = 3.48 m ôm sát thành xe.
  3. Các cảnh báo giữ nguyên: Các cảnh báo về chiều rộng của một số xe con (`s02f02, idx 4` W = 2.44 m; `s02f42, idx 1` W = 2.33 m; `s05f00, idx 3` W = 2.39 m) được giữ nguyên vì sau khi đối chiếu ảnh camera trước và bên hông, đây là các dòng xe SUV/bán tải lớn có gương chiếu hậu nhô rộng sang hai bên.

Đổi kiểm:
Đã thực hiện đổi kiểm chéo bài làm với bạn có MSSV: 2A202603318.
Hai con số được đối chiếu với bảng A:
1. Số lượng cuboid ở frame đầu tiên `s01f28`: Bạn đạt 8 cuboid, tôi đạt 8 cuboid (hoàn toàn khớp nhau).
2. Tọa độ z đáy của xe con gần nhất ở frame `s01f28`: Bạn đạt -1.78 m, tôi đạt -1.80 m (chênh lệch chỉ 0.02 m, cả hai đều đạt yêu cầu tiếp xúc mặt đường chuẩn ~ -1.7 m).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ phía model tự hành:
- Vì sao nhãn 3D đo bằng mét chứ không phải pixel: Xe tự hành vận hành trong không gian vật lý thực tế 3D; các thuật toán lập kế hoạch quỹ đạo di chuyển (path planning), tính toán khoảng cách an toàn và phanh khẩn cấp tự động (AEB) bắt buộc phải dựa trên khoảng cách và kích thước vật lý chính xác tính bằng mét. Nhãn 2D theo pixel trên ảnh không thể hiện được độ sâu chính xác do phụ thuộc vào tiêu cự và góc phối cảnh quang học của camera.
- Vì sao vật thưa điểm cần quy tắc riêng: Ở khoảng cách xa (> 25 m), mây điểm LiDAR chỉ phản xạ được từ một góc hoặc bề mặt ngoài của vật thể; nếu người gán nhãn chỉ vẽ hộp bao khít các điểm quan sát được thì mô hình AI sẽ học sai kích thước xe (tưởng xe bị ngắn lại hoặc biến dạng), dẫn đến ước lượng sai kích thước và nguy cơ va chạm trong thực tế; do đó cần quy tắc dựa vào kích thước chuẩn để suy luận bao trọn toàn bộ thể tích vật lý của xe.

(b) Cách dùng AI:
- Công cụ: Sử dụng trợ lý AI Google Antigravity trong quá trình thực hành.
- Câu hỏi chính: Nhờ giải thích hệ tọa độ VF (+x trái, +y sau, +z lên, cam01 nhìn về hướng -y), cách sử dụng các lệnh script kiểm thử tự động (`lab.py sync`, `check_submission.py`) và phân tích nguyên nhân các cảnh báo kích thước của `selfcheck`.
- Cách dùng kết quả: Tự tay thực hiện 100% thao tác gán nhãn 56 cuboid trên công cụ CVAT; áp dụng các kiến thức về hệ tọa độ được giải thích để kiểm tra và tinh chỉnh lại các góc xoay yaw và mặt đáy cuboid.

(c) Điều còn chưa chắc:
Đối với một số phương tiện di chuyển ở khoảng cách xa (> 35 m) ở làn đường đối diện trong frame `s04f57` và `s05f48`, do mật độ mây điểm rất thưa (dưới 30 điểm phản xạ) và hình ảnh camera ở cự ly xa bị giảm độ nét, việc định vị chính xác vị trí đầu xe và đuôi xe chủ yếu dựa trên kích thước xe sedan trung bình trong Bảng 4.3; do đó vị trí tâm cuboid dọc theo trục xe có thể có độ dịch chuyển khoảng ±0.3 m so với nhãn chuẩn tuyệt đối.
