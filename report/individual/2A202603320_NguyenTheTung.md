# Báo cáo Day 13 — Nguyễn Thế Tùng, 2A202603320

## 1. Kết luận trước

Task CVAT `D13-2A202603320` gồm các frame `s01f28, s01f45, s02f02, s02f42, s03f10, s03f56, s04f17, s04f57, s05f00, s05f48`. Môi trường thực hành là Windows 11, Google Chrome và CVAT Community 2.83.1. Bạn đổi kiểm có MSSV 2A202603318.

Tôi đã gán **56 cuboid dạng Shape**. Frame đầu có 8 cuboid. Các frame `s01f45`, `s02f02`, `s02f42`, `s03f10`, `s03f56` và `s05f00` có 5 cuboid mỗi frame. Ba frame còn lại có 6 cuboid mỗi frame. Trong số này, 55 cuboid có tâm cách LiDAR không quá 50 m theo khoảng cách ngang. Phiếu tự kiểm có 9 cuboid bị gắn cờ, chủ yếu do kích thước.

Tôi tự tin nhất khi xác định đầu xe và kiểm đáy hộp ở các vật gần có cụm điểm rõ. Với vật xa, tôi còn chưa chắc về chiều dài và vị trí tâm dọc thân xe vì thường chỉ thấy một phần bề mặt. Trong CVAT, tôi xoay khung 3D bằng cách kéo chuột trái, dời góc nhìn bằng chuột phải và zoom bằng con lăn. Tôi bật **Cuboid orientation**, kiểm lần lượt Top, Side, Front rồi đối chiếu ảnh camera để nhận loại vật và hướng đầu xe.

## 2. Đọc frame (Bước 2)

Tâm LiDAR là gốc của hệ VF. Trong hệ tọa độ gắn với xe, `+x` hướng sang trái xe, `+y` hướng về phía sau và `+z` hướng lên. Cả hai ảnh nhìn từ trên được xoay để đầu xe nằm ở cạnh dưới. Vì vậy **trên ảnh**, `−y` hướng xuống, `+y` hướng lên và `+x` hướng sang phải. Cam01 nhìn về phía trước, theo `−y`.

| Ảnh | Cách đọc |
|---|---|
| `t1-s01f28-3d.png` | Góc nghiêng cho thấy các dải điểm mặt đường ở dưới các cụm điểm thân xe. Dùng góc này để xem độ cao của cụm điểm so với mặt đường. |
| `t1-s01f28-top.png` | Nhìn từ trên giúp tách các cụm xe trước, sau và hai bên theo hướng của ảnh. Cụm xe ở xa và cụm ven đường thưa hơn các cụm gần LiDAR. |
| `t1-s01f45-3d.png` | Góc nghiêng giúp phân biệt mặt đường và các cụm đứng cao hơn, như thân xe và người. |
| `t1-s01f45-top.png` | Nhìn từ trên cho thấy vị trí tương đối của các xe phía trước và các vật lệch về một bên. Người đi bộ và xe hai bánh có `x < 0`, ở phía trước bên phải xe thu dữ liệu. |

| Frame | Mặt đường | Vật xa nhất còn đọc được | Loại vật và vị trí | Cụm chưa đọc được |
|---|---|---|---|---|
| `s01f28` | Gần phẳng, có vẻ hơi dốc về phía trước, thấp hơn LiDAR khoảng 1,70 m. | Xe con ở xa khoảng 35,5 m. | Car và Truck. Phía sau có xe con cùng chiều khoảng 18 m. Bên trái xe thu dữ liệu có các xe ngược chiều. | Cụm điểm ven đường ngoài khoảng 45 m, chưa đủ để phân biệt xe đỗ với vật tĩnh. |
| `s01f45` | Gần phẳng, thấp hơn LiDAR khoảng 1,72 m. | Xe phía trước khoảng 25 m theo quan sát khi đọc cảnh. | Car, Cyclist và Pedestrian. Người đi bộ cách khoảng 14,1 m và xe hai bánh cách khoảng 10,3 m, đều ở trước-phải. | Cụm điểm rải rác ngoài mép vỉa hè phía trước, chưa phân biệt rõ vật. |

Ảnh camera giúp tôi nhận màu và đèn xe, phân biệt dáng xe hoặc người đang lái với người dắt xe, và nhận đầu xe khi cụm điểm chỉ hiện một mặt. LiDAR cho phép đo khoảng cách tới các bề mặt có điểm trả về, ước lượng kích thước theo mét và đặt các cụm điểm hai bên hoặc phía sau trong cùng hệ tọa độ 360°. Đối với vật xa hoặc bị che, phần thể tích thiếu điểm vẫn phải suy luận từ hình dạng và ảnh camera.

| Frame lướt | Phía trước trong góc nhìn từ trên đã căn | Quan sát |
|---|---|---|
| `s02f02` | Cạnh dưới (`−y`) | Xe buýt bên trái ở rất gần, có hơn 16.000 điểm trong cuboid. |
| `s02f42` | Cạnh dưới (`−y`) | Các xe khoảng 30–40 m có cụm điểm thưa. |
| `s03f10` | Cạnh dưới (`−y`) | Nhiều xe phía trước và phía sau, một số trong khoảng 10 m. |
| `s03f56` | Cạnh dưới (`−y`) | Có xe buýt ở làn đối diện khoảng 30 m và xe hai bánh ở làn bên cạnh. |
| `s04f17` | Cạnh dưới (`−y`) | Nhiều xe con chạy song song cùng chiều. |
| `s04f57` | Cạnh dưới (`−y`) | Xe ở xa trên làn đối diện, còn ít điểm. |
| `s05f00` | Cạnh dưới (`−y`) | Xe tải phía sau khoảng 10 m có cụm điểm rõ. |
| `s05f48` | Cạnh dưới (`−y`) | Cảnh giao lộ, có xe chuyển làn và một xe tải phía sau khoảng 47 m. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Trên ba ảnh, khung phối cảnh cho biết hộp đang bao vật nào. **Top** dùng để kiểm chiều dài, chiều rộng và hướng mũi tên. **Side** dùng để xem hai đầu hộp và cao độ đáy. **Front** dùng để kiểm bề ngang, chiều cao và đáy so với các điểm mặt đường. Ảnh camera bên cạnh dùng để đối chiếu loại vật và đầu, đuôi xe. Vạch đỏ ngắn `+X` xoay theo từng cuboid.

| Ảnh và cuboid | Hướng và đáy | Đối chiếu Bảng 4.3 |
|---|---|---|
| `t2-a-car.png` · `s01f28`, idx 1, **Car** | Yaw 270,0°, dọc thân xe và cùng chiều với xe thu dữ liệu. Kiểm hai đầu hộp ở Top, rồi nhìn mép đáy ở Side và Front. Đáy −1,801 m, thấp hơn mặt đường ước lượng quanh hộp 0,08 m. | Dài × rộng × cao là 5,05 × 2,09 × 1,66 m, phù hợp dải SUV/minivan. |
| `t2-b-bus-truck.png` · `s01f28`, idx 4, **Truck** | Yaw 90,2°, gần hướng ngược chiều. Side cho thấy chiều dài thân/thùng. Front cho thấy bề ngang và đáy −2,058 m, cao hơn mặt đường ước lượng tại chỗ khoảng 0,01 m. | 5,98 × 2,41 × 2,42 m, nằm trong dải xe tải hộp/xe tải nhẹ. |
| `t2-c-ped-cyclist.png` · `s01f45`, idx 4, **Cyclist** | Yaw 280,1°, gần hướng cùng chiều. Top dùng để kiểm cả chiều dài xe và người trong một hộp. Side và Front dùng để kiểm khoảng từ điểm tiếp xúc bánh xe đến đỉnh đầu người. Đáy −1,354 m cao hơn mức đường ước lượng quanh hộp 0,32 m, nên còn cần đối chiếu với nền ngay dưới bánh. | 1,66 × 0,61 × 1,73 m. Nằm trong dải xe đạp có người. Chiều dài và chiều rộng thấp hơn dải tham chiếu xe máy, nên kiểu xe và độ khít ở hai phương này còn chưa chắc. |

Qua vòng hiệu chuẩn, tôi dùng mốc gần 270° cho xe chạy thẳng cùng chiều và gần 90° cho xe chạy ngược chiều trên đoạn đường này. Tôi đối chiếu hướng với ảnh camera và kiểm đáy theo mặt đường tại vị trí của vật.

Trường hợp bỏ qua là cụm điểm ven đường ở `s02f42`, phía bên phải, khoảng 48 m. Tôi ước lượng chỉ có khoảng 8–10 điểm, còn ảnh camera ở cự ly đó chưa đủ rõ để phân biệt xe với vật tĩnh. Tôi ghi trường hợp này là `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Ảnh, frame và nguồn | Cách đọc ảnh và giả thuyết | Hệ quả khi gán |
|---|---|---|---|
| Bóng che | `t4-1-bong-che.png` · `s01f28` · task cá nhân | Nhìn từ trên, vùng theo tia từ LiDAR đi qua Car idx 1 có ít điểm mặt đường hơn vùng sát bên. Vùng khuyết nằm xa tâm LiDAR hơn xe, về phía sau-phải của xe thu dữ liệu. Giả thuyết là thân xe chặn tia đi sâu hơn theo hướng đó. | Vùng khuyết là vùng chưa quan sát được. Nếu một vật khác bị che một phần, đánh dấu `occluded` và bao phần thể tích có thể suy luận từ cụm điểm, ảnh camera và kích thước tham chiếu. |
| Thưa ở xa | `t4-2-thua-o-xa.png` · `s02f42` · task cá nhân | Nhìn cụm Car ở giữa khung nhìn, idx 5, cách 39,78 m và có 72 điểm trong hộp. Một Car gần hơn trong cùng frame, idx 1 ở 24,76 m, có 286 điểm. Giả thuyết là với cùng bước góc quét, các tia cách nhau xa hơn trên bề mặt vật ở xa. | Nếu thấy khoảng 15–20% thể tích hoặc có đủ căn cứ từ hình dạng và camera, có thể vẽ cả hộp. Trường hợp quá thưa như cụm khoảng 48 m ở mục 3 được bỏ qua và ghi `LOW_VISIBILITY`. |
| Phản xạ thấp hoặc ít điểm bất thường | `t4-3-phan-xa-thap.png` · `s01f45` · task cá nhân | Trong một phần tư chiều cao trên cùng của cuboid, Car idx 1 ở 10,36 m có 314 điểm và Car idx 2 ở 9,36 m có 475 điểm. Khoảng cách, góc bề mặt hoặc vật liệu có thể góp phần tạo chênh lệch. Chỉ từ số điểm và ảnh này chưa xác định được nguyên nhân. | Giữ kích thước theo toàn bộ thân và mui xe khi có đủ cơ sở từ hình dạng và camera. Ghi phần ít điểm như một quan sát cần lưu ý khi kiểm. |
| Điểm lạ | `t4-4-diem-la.png` · `s02f42` · task cá nhân | Cụm nhỏ nằm tách phía dưới dải điểm mặt đường, hơi lệch phải so với tâm ảnh. Cụm có **12 điểm** ở phía trước-phải, cách LiDAR khoảng **20 m** theo mặt phẳng ngang. Tâm cụm có `z ≈ −9,31 m`, thấp hơn mặt đường ước quanh cụm (`z ≈ −1,79 m`) khoảng **7,5 m**. Phản xạ nhiều lần là một giả thuyết. | Cụm nằm tách hẳn dưới mặt đường. Không gán cụm thành vật và không kéo đáy cuboid xuống để bao các điểm đó. |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu lấy từ `submission/D13-2A202603320-bangA.csv` và `submission/D13-2A202603320-tukiem.csv`. `ground_gap_m` là chênh giữa đáy hộp và mặt đường ước lượng quanh hộp. Giá trị âm nghĩa là đáy thấp hơn mức ước lượng, giá trị dương nghĩa là đáy cao hơn.

| Cuboid | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r (m) | Yaw (°) | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| `s01f28`, idx 1, Car | −0,971 | 1,660 | −1,801 | 6,89 | 6,96 | 270,0 | Gần mặt đường quanh hộp, chênh −0,08 m. |
| `s01f45`, idx 1, Car | −1,064 | 1,751 | −1,939 | 10,36 | 10,41 | 268,3 | Đáy hơi thấp hơn mặt đường ước lượng, chênh −0,16 m. |
| `s01f45`, idx 5, Pedestrian | −0,306 | 1,570 | −1,091 | 14,13 | 14,13 | 258,3 | Đáy cao hơn mức ước lượng quanh hộp 0,18 m. Vỉa hè cao hơn lòng đường có thể giải thích một phần chênh lệch. |

**(a) Gần nhất và xa nhất.** `dist_m = √(x² + y²)` là khoảng cách ngang và `r = √(x² + y² + z²)` là khoảng cách 3D, đều đo đến tâm hộp.

- Gần nhất là Bus `s02f02`, idx 1, với `dist_m = 3,43 m` và `r = 3,43 m` sau khi làm tròn.
- Xa nhất trong bảng A là Car `s05f48`, idx 4, với `dist_m = 77,42 m` và `r = 77,43 m`. Cuboid này nằm ngoài vùng chấm 50 m.
- Xa nhất trong phạm vi 50 m là Truck `s05f48`, idx 6, với `dist_m = 46,91 m` và `r = 46,92 m`.

**(b) So kích thước với Bảng 4.3.** Pedestrian `s01f45`, idx 5, có kích thước 0,50 × 0,58 × 1,57 m, trong dải người lớn 0,40–0,90 × 0,50–0,90 × 1,50–1,95 m. Car `s01f28`, idx 1, có kích thước 5,05 × 2,09 × 1,66 m, trong dải SUV/minivan 4,40–5,30 × 1,80–2,10 × 1,65–2,10 m. Bề rộng ở gần cận trên của dải này.

**(c) Yaw cùng chiều và ngược chiều.** Car `s01f45`, idx 2, có yaw 269,6°, gần mốc cùng chiều 270°. Truck `s01f28`, idx 4, có yaw 90,2°, gần mốc ngược chiều 90°.

| Cuboid (đều là Car ở `s01f28`) | dist_m (m) | `points_in` (điểm) |
|---|---:|---:|
| idx 1 | 6,89 | 2.130 |
| idx 3 | 13,78 | 1.004 |
| idx 7 | 21,24 | 359 |
| idx 8 | 35,46 | 120 |

Khi khoảng cách tăng từ 6,89 m đến 35,46 m, số điểm trong hộp giảm từ 2.130 xuống 120. Xu hướng phù hợp với các mốc trung vị trên slide ở bốn dải tương ứng, lần lượt khoảng 1.968, 718, 214 và 72 điểm. Góc quan sát, phần bị che và bề mặt vật có thể làm từng xe lệch khỏi xu hướng chung.

Tôi đã chỉnh hướng hộp của một Car và thu chiều rộng hộp của một Truck sau khi tự kiểm.

1. Car `s01f28`, idx 1. Hộp ban đầu xoay lệch khoảng 90°. Tôi căn lại vạch đỏ theo thân xe, được yaw 270,0°, chiều dài 5,05 m và chiều rộng 2,09 m.
2. Truck `s05f48`, idx 6. Tôi thu chiều rộng từ khoảng 4,46 m xuống 3,48 m sau khi đối chiếu Front và Top. Phiếu tự kiểm vẫn gắn cờ chiều rộng. Cụm điểm ở gần 47 m, nên bề ngang chính xác còn chưa chắc.

Tôi giữ các cờ chiều rộng của Car `s02f02` idx 4 (2,44 m), `s02f42` idx 1 (2,33 m) và `s05f00` idx 3 (2,39 m). Khi đối chiếu camera, tôi đã tính đến phần gương nhô ra. Đây là các trường hợp còn cần kiểm lại ranh giới thân xe và class.

Tôi đổi kiểm với bạn MSSV 2A202603318. Hai số đã trao đổi là số cuboid ở `s01f28` (mỗi người có 8) và đáy cuboid xe con gần nhất (bạn −1,78 m, tôi −1,80 m, chênh khoảng 0,02 m).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

**(a) Ý nghĩa với mô hình tự hành.** Cuboid dùng mét để mô hình học vị trí và kích thước trong không gian xe vận hành, từ đó ước lượng khoảng cách an toàn. Với vật thưa điểm, hộp chỉ ôm phần bề mặt có điểm sẽ làm kích thước học được lệch khỏi kích thước thực. Quy tắc về mức nhìn thấy và kích thước tham chiếu giúp giữ nhãn nhất quán khi có đủ căn cứ suy luận.

**(b) Cách dùng AI.**

- Tôi dùng Google Antigravity và Codex.
- Tôi hỏi về hệ trục VF, cách chạy `lab.py sync` và `check_submission.py`, ý nghĩa cờ tự kiểm và cách chụp ảnh CVAT.
- Tôi tự gán nhãn trong CVAT. Antigravity hỗ trợ giải thích lệnh và hệ trục. Codex hỗ trợ lưu ảnh từ CVAT trên máy, xác định hướng trục, diễn giải bộ ảnh và biên tập báo cáo dựa trên bài làm và số liệu đã có.

**(c) Điều còn chưa chắc.** Ở `s04f57` và `s05f48`, một số xe ở xa chỉ có ít điểm trên một mặt, nên tâm hộp có thể lệch dọc thân xe khoảng vài chục centimét. Đây là ước lượng từ thao tác gán nhãn. Mức nền ngay dưới Cyclist `s01f45` idx 4 và nguyên nhân sinh ra cụm điểm dưới mặt đường ở `s02f42` vẫn cần kiểm thêm.
