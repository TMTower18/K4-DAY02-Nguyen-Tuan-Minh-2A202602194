# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Nguyễn Tuấn Minh<br>
**MSSV:** 2A202602194<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33
- Bốn mã ảnh: drive_022, drive_033, drive_038, drive_008
- Số vật thể thực tế: 136
- Mã SHA-256 của gói YOLO của bạn: 9ee60fa9a9f528d1a19aba3e579cda268e89b7b94e85dcf596c5e0f965e8ec3c
- Mã SHA-256 của gói CVAT gốc của bạn: f154cc0850f627a43480626584af3215020eb94b78e213611041368e5a74ac1e
- Nguồn đối chiếu: bộ nhãn đối chiếu do Lab Coach cấp (teaching_reference)
- Mã SHA-256 của gói đối chiếu: c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: Lần nhận bộ tham chiếu sau khi hoàn thành gán nhãn + huấn luyện thử (ngày 14/09/2026)

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:
Tôi đã gán nhãn toàn bộ 4 ảnh trong CVAT một cách độc lập, xuất cả 2 gói (YOLO + CVAT for images 1.1) từ cùng một job, tự kiểm tra thuộc tính và hình học trước khi nhận bất kỳ nguồn đối chiếu nào. Chỉ sau khi hoàn thành kiểm tra audit, huấn luyện thử và tạo detect_result.jpg mới tải bộ nhãn teaching_reference lên để đối chiếu.

## 2. Quyết định phân lớp

| Ảnh/vật thể                          | Lớp   | Dấu hiệu nhìn thấy                                      | Quy tắc áp dụng                                                                 |
|--------------------------------------|-------|---------------------------------------------------------|---------------------------------------------------------------------------------|
| drive_008.jpg – xe tải lớn phía trước | truck | Khung xe lớn, thùng hàng/cabin cao, bánh lớn            | Có thùng chứa hàng rõ ràng + kích thước lớn hơn xe con → truck                 |
| drive_008.jpg – xe khách dài          | bus   | Thân xe dài, nhiều cửa sổ liên tiếp, cao hơn xe con     | Hình dạng dài + nhiều cửa sổ hành khách → bus                                  |
| drive_008.jpg – xe hộp vừa            | van   | Hình hộp chữ nhật, cabin liền thân, kích thước trung bình | Hình dạng hộp, không có thùng hàng riêng, nhỏ hơn bus → van                    |
| drive_008.jpg – nhiều xe con          | car   | Kích thước nhỏ, cabin hành khách, 4 bánh rõ             | Xe 4 chỗ thông thường, không thùng hàng lớn → car                              |
| drive_022.jpg – xe tải                | truck | Cabin + thùng chứa, kích thước lớn                      | Có thùng chứa hàng + kích thước lớn → truck                                    |
| drive_022.jpg – xe buýt               | bus   | Thân dài, nhiều cửa sổ                                  | Dài + cửa sổ hành khách nhiều → bus                                            |
| drive_033.jpg – nhiều xe con nhỏ      | car   | Hình dạng compact, cabin thấp                           | Kích thước nhỏ, không thùng hàng → car                                         |
| drive_038.jpg – xe tải                | truck | Cabin cao + phần chứa hàng                              | Có thùng hàng rõ + lớn hơn xe con → truck                                      |
| drive_038.jpg – xe buýt               | bus   | Thân dài, nhiều cửa sổ                                  | Dài + nhiều cửa sổ → bus                                                       |
| drive_038.jpg – xe van                | van   | Hình hộp, kích thước trung bình                         | Hình hộp liền thân, không phải bus dài → van                                   |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Chiếc xe (ID: 9) trong ảnh drive_008.jpg được gán lớp = car (vì nó là xe con 4 chỗ, kích thước nhỏ, cabin hành khách).

Cùng chiếc xe đó lại có thuộc tính:

visibility = occluded (bị che khuất một phần)
boundary = inside (nằm hoàn toàn trong khung hình)
review_state = confident

→ Lớp trả lời câu hỏi “Đây là loại gì?” (car / truck / bus / van).

→ Thuộc tính trả lời các câu hỏi bổ sung về trạng thái quan sát (rõ/mờ/che khuất, nằm trong hay bị cắt biên, độ tin cậy của annotation).
Hai loại thông tin độc lập: cùng một lớp car có thể có nhiều tổ hợp thuộc tính khác nhau tùy theo điều kiện nhìn thấy.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa                                      | Loại lỗi                        | Cách phát hiện                          | Sau khi sửa và quy tắc                                                                 |
|----------------------------------------------------|---------------------------------|-----------------------------------------|----------------------------------------------------------------------------------------|
| Xe van nhỏ ở góc được gán lớp `car`                | Lớp (class)                     | So sánh hình dạng hộp vs xe con        | Đổi thành `van`. Quy tắc: thân xe hình hộp liền, không có thùng hàng riêng → van      |
| Bounding box xe tải bị cắt mất phần thùng hàng     | Hình học (geometry)             | Quan sát bbox không bao hết đối tượng  | Mở rộng bbox bao trọn cabin + thùng. Quy tắc: bbox phải bao toàn bộ đối tượng nhìn thấy |
| Xe ở xa rất nhỏ gán `visibility=clear`             | Thuộc tính (attribute)          | Kiểm tra độ rõ của chi tiết            | Đổi thành `visibility=unclear`. Quy tắc: không nhìn rõ biển số/đèn/hình dáng → unclear |
| Xe bị che bởi xe khác nhưng gán `visibility=clear` | Thuộc tính (attribute)          | So sánh với phần bị che khuất          | Đổi thành `visibility=occluded`. Quy tắc: >30% diện tích bị che → occluded             |
| Xe chỉ lộ một phần sát mép ảnh gán `boundary=inside` | Thuộc tính (attribute)        | Kiểm tra tọa độ bbox sát biên khung hình | Đổi thành `boundary=truncated`. Quy tắc: bất kỳ cạnh nào chạm/cắt mép ảnh → truncated |
| Xe buýt dài bị gán nhầm thành `truck`              | Lớp (class)                     | Đếm số cửa sổ và hình dạng thân xe     | Đổi thành `bus`. Quy tắc: thân dài + nhiều cửa sổ hành khách liên tiếp → bus          |


- Số hộp `needs_review` trước và sau khi kiểm: 34 / 28
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ:
Ở ảnh drive_008.jpg có một đối tượng nhỏ ở phía xa (ID: 9) hình dạng hơi hộp.

Ban đầu không chắc là van hay car vì quá nhỏ, chi tiết không rõ (không thấy rõ cửa sổ hay thùng hàng).
→ Đánh dấu review_state = needs_review + visibility = unclear.

→ Xin hỗ trợ bằng cách: hỏi Lab Coach xem cùng khung hình để thống nhất quy tắc phân lớp cho đối tượng xa.

## 4. Một dòng nhãn YOLO

- Dòng class x_center y_center width height: [0, 0.516266, 0.527477, 0.109031, 0.073391]
- Tên lớp và tọa độ điểm ảnh xyxy: lớp = 0 (car), pixel xyxy ≈ [295.5, 314.1, 365.3, 361.1]
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Dòng trên đúng cú pháp YOLO (5 số, class_id nằm trong 0–3, tọa độ chuẩn hóa trong [0,1]), nhưng vẫn có thể sai vì:

Lớp: có thể nhầm car với van hoặc truck nếu chỉ dựa vào kích thước mà không nhìn kỹ hình dạng thùng/hộp.
Phạm vi: có thể gán nhầm vật thể không thuộc 4 lớp (xe máy, người…) hoặc bỏ sót/gộp nhiều xe vào một hộp.
Hình học: bbox có thể quá lỏng (chứa nhiều nền) hoặc quá chặt (cắt mất phần xe), hoặc ước lượng phần bị che.


## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: drive_022, drive_033, drive_038
- Mã ảnh thẩm định: drive_008
- Mô tả một dự đoán trong detect_result.jpg: Mô hình dự đoán khá ít hộp và độ tin cậy thấp trên ảnh val (drive_008), nhiều xe con và xe lớn bị bỏ sót hoặc bị nhầm lớp (mAP50 rất thấp ≈ 0.0086).
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Cần kiểm lại phạm vi gán nhãn (số hộp 136 vượt xa mục tiêu 40–60), đặc biệt các hộp rất nhỏ/xa ở chân trời và các trường hợp visibility=unclear. Có thể đang gán quá nhiều đối tượng mờ, khiến mô hình khó học đặc trưng rõ ràng.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu sau khi lọc bỏ các hộp unclear/quá nhỏ rồi huấn luyện lại mà mAP vẫn thấp, hoặc nếu bộ teaching_reference cũng có số lượng hộp tương tự và vẫn cho kết quả kém, thì vấn đề nằm ở độ khó của dữ liệu/khả năng của mô hình nhỏ hơn là do quy tắc gán nhãn.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Chỉ có 3 ảnh train + 1 ảnh val, dữ liệu cực kỳ nhỏ và không đại diện, early-stopping sau 4 epoch, mục đích chỉ để phản hồi lỗi dữ liệu/nhãn chứ không phải benchmark sản xuất.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: mean IoU = 0.8686, median IoU = 0.8951
- Mức đồng thuận lớp: 0.7292 (≈ 72.9%)
- Số hộp phía bạn không ghép được: 88
- Số hộp phía đối chiếu không ghép được: 2
- Một điểm khác biệt cụ thể: Tôi gán rất nhiều hộp nhỏ/xa (đặc biệt ở drive_038 và drive_033) trong khi bộ teaching_reference chỉ giữ khoảng 50 hộp rõ ràng hơn → phần lớn 88 hộp unmatched của tôi là các đối tượng unclear hoặc quá nhỏ.
- Quy tắc hoặc hành động sửa phát sinh: Áp dụng chặt hơn quy tắc “vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ → không đoán”. Xóa hoặc chuyển sang needs_review các hộp có diện tích rất nhỏ và visibility=unclear.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? IoU cao và class agreement chỉ cho thấy hai người (hoặc người với bộ tham chiếu) đồng ý với nhau trên các hộp đã ghép được. Cả hai có thể cùng sai (cùng bỏ sót, cùng nhầm lớp, cùng vẽ bbox không tối ưu). Đồng thuận ≠ đúng tuyệt đối.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng:
- Hai gói xuất (YOLO + CVAT for images 1.1) được kiểm tra chéo với minimum_cross_format_iou ≈ 0.9998 → cùng một trạng thái annotation, không bị lệch hình học hay lớp giữa hai định dạng.
- Kết quả đối chiếu với bộ teaching_reference: 48 hộp ghép được, mean IoU ≈ 0.869, median IoU ≈ 0.895, class agreement ≈ 72.9%. Điểm khác biệt rõ ràng nhất là số lượng hộp (136 vs 50), cho thấy tôi đã gán quá nhiều đối tượng nhỏ/mờ.
- Ảnh phủ hộp comparison_overlay.png và detect_result.jpg trực quan hóa sự khác biệt phạm vi và lỗi dự đoán.

Câu hỏi còn lại cho Lab Coach:

- Với mục tiêu 40–60 vật thể, khi nào nên giữ vs loại bỏ các xe rất nhỏ ở chân trời (visibility=unclear)? Có ngưỡng diện tích tối thiểu khuyến nghị không?
- Khi số hộp unmatched của học viên cao (88 hộp), có nên ưu tiên sửa theo hướng giảm số lượng về gần bộ tham chiếu, hay giữ nguyên nếu vẫn tuân thủ quy tắc “đủ bằng chứng phân lớp”?
