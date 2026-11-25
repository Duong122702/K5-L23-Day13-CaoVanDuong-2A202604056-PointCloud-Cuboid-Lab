# Báo cáo Day 13 — Cao Văn Dương, 2A202604056

## 1. Kết luận trước

Mã số: 2A202604056, task `D13-2A202604056`, 10 frame: s01f26, s01f50, s02f17, s02f46, s03f32, s03f49, s04f00, s04f33, s05f15, s05f48. Hệ điều hành: Windows 11. Trình duyệt: Microsoft Edge. Phiên bản CVAT: 2.83. MSSV bạn ngồi cạnh: 2A202604057.

Tổng cộng 53 cuboid (47 Car, 1 Bus, 1 Pedestrian, 2 Cyclist). Frame s01f26 nhiều nhất (8 cuboid) vì là frame đầu, nhiều xe gần rõ ràng; các frame còn lại mỗi frame 5. Selfcheck cờ 8/53 cuboid. Phần tôi tin nhất là phân loại class (Car, Bus, Pedestrian, Cyclist) vì đối chiếu được với ảnh camera. Phần yếu nhất là đáy và kích thước cuboid, đặc biệt với vật xa thưa điểm, vì khó căn chỉnh chính xác khi điểm ít.

Cách dùng CVAT: (1) Đổi góc nhìn: kéo chuột trái để xoay khung 3D, kéo chuột phải để dời, lăn con lăn để phóng to/thu nhỏ; xoay cho tới khi nhìn thẳng xuống để có góc nhìn từ trên xuống. (2) Vẽ cuboid: chọn nút vẽ cuboid ở thanh công cụ trái, chọn label, bấm Shape, nháy đúp vào cụm điểm để đặt hộp; chỉnh ở khung Top (dời tâm, kéo cạnh, xoay yaw), Side và Front (kéo xuống cho đáy chạm đường). (3) Bật Appearance → Cuboid orientation để hiện mũi tên hướng; đối chiếu ảnh camera để xác định đầu xe; chạy `python scripts/lab.py preview` để xem cuboid trên ảnh camera.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f26 | Phía trước (−y) ở cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới (ra sau) | Phẳng; thấp hơn LiDAR khoảng 1,7 m (z_bottom trung bình ≈ −1,7 m) | ~27 m (#7 Car ở 27,1 m) | Car (8 xe con) | Phía sau: 5 xe (y > 0) gồm #1 ở 6,9 m, #3 ở 18,4 m, #7 ở 27,1 m; bên trái xa: #5 ở x = 21 m, #8 ở x = 23,5 m — không thấy trong cam01 | Có cụm điểm thưa ở rìa xa (~25–30 m) khó xác định là xe hay vật cố định |
| s01f50 | Phía trước (−y) ở cạnh trên; +x chỉ sang trái, +y chỉ xuống dưới (ra sau) | Phẳng; thấp hơn LiDAR khoảng 1,7–1,8 m | ~16 m (#5 Car ở 16,1 m) | Car (5 xe con) | Phía sau: #1 ở 6,5 m, #2 ở 10,3 m (y > 0); phía trước: #3, #4, #5 đều ở hướng −y | Vài cụm điểm nhỏ bên phải xa (x < 0) không rõ là xe hay cây/cột |

3 điều **chỉ camera** cho biết: (1) Màu sắc xe — phân biệt xe trắng, đen, đỏ để hỗ trợ nhận dạng và giải thích sự khác biệt số điểm. (2) Đầu và đuôi xe — đèn pha, biển số, nắp ca-pô giúp xác định hướng yaw. (3) Vạch kẻ đường và biển báo — nhận ra cấu trúc đường, làn xe, giúp suy luận hướng di chuyển.

3 điều **chỉ LiDAR** cho biết: (1) Khoảng cách chính xác đến vật (mét) — camera không cho biết vật cách bao xa. (2) Hình dạng 3D thực tế — chiều cao, rộng, dài của vật trong không gian. (3) Vật ở mọi hướng 360° — LiDAR quét tròn, trong khi mỗi camera chỉ phủ một góc hẹp.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f17 | Cạnh trên (−y) | Hai xe ngược chiều (#1, #2, #3 yaw ≈ 90°) gần nhau ở khoảng 5–11 m |
| s02f46 | Cạnh trên (−y) | Xe #5 ở khá xa (38,1 m, 85 điểm), vẫn nhận ra hình dạng |
| s03f32 | Cạnh trên (−y) | Có Bus (#3) và Pedestrian (#4) — frame đa dạng class nhất; xe #5 ở 66,3 m chỉ 22 điểm |
| s03f49 | Cạnh trên (−y) | Có Cyclist (#5, xe máy có người) bên trái |
| s04f00 | Cạnh trên (−y) | Xe #1 rất gần (4,3 m, 5854 điểm), dày điểm nhất trong 10 frame |
| s04f33 | Cạnh trên (−y) | Có Cyclist (#5); hai xe xa #3, #4 ở ~26,5 m |
| s05f15 | Cạnh trên (−y) | Xe #2 rất gần (4,8 m, 5709 điểm); xe ở nhiều hướng khác nhau |
| s05f48 | Cạnh trên (−y) | Xe #5 ở 75,2 m chỉ 14 điểm — xa nhất trong toàn bộ bài; xe #1 gần (4,5 m) có 9720 điểm |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f26, #1 | Car | Có, vạch đỏ dọc thân xe chỉ về đầu xe | z_bottom = −1,68 m, chạm đường | Có | l=4,50 w=2,09 h=1,50; trong khoảng sedan/SUV |
| (b) xe buýt | s03f32, #3 | Bus | Có, vạch đỏ dọc thân xe chỉ về đầu xe | z_bottom = −1,68 m, chạm đường | Có | l=9,28 w=2,59 h=3,09; trong khoảng xe buýt thường |
| (c) xe hai bánh | s03f49, #5 | Cyclist | Có, vạch đỏ chỉ theo hướng di chuyển | z_bottom = −1,78 m, chạm đường | Có | l=1,74 w=0,81 h=1,68; trong khoảng xe máy có người |

Vòng hiệu chuẩn: cuboid của tôi lệch so với các bạn chủ yếu ở tâm và kích thước; cả lớp chốt cách hiểu: cuboid phải bao trọn các điểm LiDAR của vật, kéo cạnh sát điểm ngoài cùng, kiểm đủ cả 6 mặt hộp trên Top, Side, Front.

Vật bỏ qua vì không đủ cơ sở: có một số cụm điểm quá xa và quá thưa không đủ 15–20% thể tích để xác định hình dạng (LOW_VISIBILITY), và vài cụm không rõ là xe hay vật cố định (UNCERTAIN_CLASS).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f32 / `t4-1-bong-che.png` | Xe buýt (#3 Bus, 9,28 m dài, ở 34,7 m) chắn tia LiDAR → tia không quét được vùng phía sau xe buýt | Vùng phía sau xe buýt gần như không có điểm, trong khi hai bên vẫn dày điểm; vật ở phía xa cùng hướng (nếu có) bị mất một phần điểm | Bật occluded cho vật bị che; cuboid bao phần suy luận được từ ảnh camera và hình dạng nhìn thấy |
| 2. Thưa ở xa | s05f48 / `t4-2-thua-o-xa.png` | Tia LiDAR phân kỳ theo khoảng cách → càng xa thì khoảng cách giữa các tia càng lớn, ít tia trúng vật | #5 Car ở 75,2 m chỉ có 14 điểm, hình dạng xe gần như không nhận ra; so với #1 Car ở 4,5 m có 9720 điểm trong cùng frame | Vẽ nếu thấy khoảng 15–20% thể tích hoặc suy luận được; không đủ cơ sở thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s01f50 / `t4-3-phan-xa-thap.png` | Bề mặt xe tối màu hoặc kính hấp thụ nhiều tia laser hơn → ít tia phản hồi về cảm biến | #3 Car ở 9,6 m có 881 điểm, trong khi #4 Car ở 9,2 m (khoảng cách tương đương) có 1301 điểm — ít hơn khoảng 32%; kiểm tra ảnh camera thấy xe #3 có thể có màu tối hơn | Ghi giả thuyết về nguyên nhân, không khẳng định; cuboid vẫn bao trọn các điểm nhìn thấy |
| 4. Điểm lạ | Frame của tôi / `t4-4-diem-la.png` | Có thể do phản xạ đa đường (tia laser bật qua nhiều bề mặt như kính, kim loại bóng) hoặc lỗi đo khoảng cách của cảm biến | Khi xoay khung 3D nhìn nghiêng, thấy các điểm rải rác và vệt cong nằm trên cao so với mặt đường, không thuộc vật nào (không phải cây, cột hay tòa nhà); các điểm này lẻ loi, không tạo thành hình dạng vật thể nhận ra được | Không gán thành vật; bỏ qua khi vẽ cuboid |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202604056-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f26, #1, Car | −0,94 | 1,50 | −1,68 m | 6,9 m | 6,9 m | 268,7° | Chạm đường |
| s04f00, #1, Car | −0,80 | 1,92 | −1,76 m | 4,3 m | 4,4 m | 269,1° | Chạm đường |
| s03f32, #4, Pedestrian | −0,45 | 2,17 | −1,53 m | 12,8 m | 12,8 m | 290,4° | Hơi cao 0,17 m nhưng chấp nhận |

(a) Cuboid **gần nhất**: s04f00 #1 Car, dist_m = 4,3 m, r = √(3,65² + 2,36² + 0,80²) = 4,4 m. Cuboid **xa nhất**: s05f48 #5 Car, dist_m = 75,2 m, r = √(7,97² + 74,74² + 1,04²) = 75,2 m.

(b) Người đi bộ s03f32 #4 Pedestrian: l=0,55 w=0,53 h=2,17 m. So với bảng 4.3 (người lớn: l=0,4–0,9 w=0,5–0,9 h=1,50–1,95): dài và rộng trong khoảng; cao 2,17 m vượt khoảng (> 1,95 m), có thể do cuboid bao cả mũ/nón hoặc chưa hạ đáy sát đường. Xe con s01f26 #1 Car: l=4,50 w=2,09 h=1,50 m. So với bảng 4.3 (sedan: l=3,8–5,0 w=1,70–1,90 h=1,40–1,60): dài và cao trong khoảng; rộng 2,09 m vượt khoảng sedan nhưng nằm trong khoảng SUV (1,80–2,10).

(c) Xe cùng chiều phía trước: s01f26 #1, yaw_deg = 268,7°, gần 270°. Xe ngược chiều: s02f17 #1, yaw_deg = 91,2°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các dist_m khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (points_in) |
|---|---|---|
| s04f00, #1, Car | 4,3 m | 5854 |
| s01f26, #2, Car | 10,7 m | 1419 |
| s01f26, #3, Car | 18,4 m | 437 |
| s02f46, #5, Car | 38,1 m | 85 |

Quy luật: xa gấp khoảng 2 lần thì số điểm giảm khoảng 4–5 lần (tỷ lệ nghịch bình phương khoảng cách). So với slide bài giảng (xe con trung vị: 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m): số điểm của tôi cao hơn trung vị ở khoảng gần (5854 ở 4,3 m) và gần sát trung vị ở khoảng xa (85 ở 38,1 m), phù hợp xu hướng chung.

Hai cuboid đã **sửa** sau selfcheck: (1) s04f00 #3 Car — hộp rỗng 0 điểm, lơ lửng 5,23 m, kích thước 1×1×1: đã xóa và vẽ lại đúng vị trí cụm điểm. (2) s03f32 #5 Car — length < width, yaw sai: đã xoay yaw 90° cho vạch đỏ dọc thân xe.

Những cảnh báo **giữ nguyên** kèm lý do: s01f26 #5 Car width=1,31 m (hơi nhỏ hơn khoảng 1,4 m nhưng xe nhìn trên camera là xe nhỏ, chấp nhận); s04f33 #4 Car length=2,98 m (gần sát ngưỡng 3,0 m, lệch không đáng kể); s05f15 #5 Car height=1,17 m (vật ở gần, điểm rõ, kích thước phù hợp quan sát trên point cloud).

Đổi kiểm: đã kiểm chéo với 2A202604057. Hai con số đối chiếu: dist_m và yaw_deg của một cuboid trên bảng A. Một con số khớp, một con số lệch nhẹ — đã kiểm lại trên CVAT và giữ nguyên vì sai số trong khoảng cho phép.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Nhãn 3D đo bằng mét chứ không phải pixel vì model tự lái cần biết kích thước và khoảng cách thực tế của vật thể trong không gian 3D để đưa ra quyết định (phanh, rẽ, giữ làn). Pixel chỉ cho biết vị trí trên ảnh 2D, không cho biết vật cách bao xa hay lớn bao nhiêu. Vật thưa điểm cần quy tắc riêng vì ở xa, LiDAR trả về rất ít điểm khiến hình dạng vật không rõ ràng; nếu không có ngưỡng (15–20% thể tích), người gán nhãn sẽ không nhất quán khi quyết định vẽ hay bỏ qua, gây nhiễu cho tập huấn luyện.

(b) Có dùng AI. Công cụ: Claude Code (Anthropic). Câu hỏi chính: hỏi về hệ tọa độ VF, cách căn hướng yaw cuboid, phân biệt Shape và Track, cách đọc bảng A và phiếu tự kiểm, điền số liệu vào báo cáo. Cách dùng kết quả: dùng để hiểu guideline nhanh hơn, tính toán z đáy và r từ bảng A.

(c) Điều còn chưa chắc: kích thước cuboid của các vật ở xa (dist_m > 50 m) như s05f48 #5 (75,2 m) và s03f32 #5 (66,3 m) vì điểm quá thưa (chỉ 14–22 điểm), khó xác định chính xác biên của vật nên length và height có thể lệch so với thực tế.
