# Báo cáo thực hành Day 3 — Segmentation

- Mã học viên theo lớp: 2A202603141
- Ngày / CVAT local: 2026-11-11 · CVAT local (tạo task → Job editor → Export job dataset)
- Công cụ đã dùng: Brush/Polygon trong CVAT local (không dùng SAM)

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Ba tier và bốn checkpoint (cp3–cp6) đã export đúng format quy định (semantic: Segmentation mask 1.1; instance/panoptic: COCO 1.0) và đặt trong `submissions/` với đúng tên. Cả chín trạm đã có ZIP; cp1_holes còn **lỗi mask trùng** được liệt kê ở mục 3, cần sửa trong CVAT và export lại nếu còn kịp.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg` — một **person** bên trái khung hình (tâm mask ~39% chiều ngang, ~49% chiều cao của ảnh, vùng nhìn thấy ~23 nghìn pixel). Đây là object đầu tiên tôi tô bằng Brush trước khi xử lý các vật còn lại trong ảnh.
- Class và quy tắc tôi dùng để chọn biên: class `person` (đúng chữ trong `classes.json` của Medium). Biên đi theo **phần nhìn thấy** của cơ thể: dừng ở ranh người giao với nền/xe bên cạnh, không mở rộng vào bóng đổ dưới chân, và phần thân bị xe che thì không tự đoán vẽ tiếp.
- Nếu dùng gợi ý sau đó: không dùng gợi ý tự động cho object này (lần lượt các vật còn lại cũng tô tay để kiểm class và số object trực tiếp trong danh sách Objects).
- Nếu không dùng gợi ý: một quyết định gán nhãn của tôi là với **người ngồi trên xe máy** trong cùng ảnh — người là mask `person` riêng, xe là mask `motorcycle` riêng, không gộp người vào mask xe dù hai vùng đè lên nhau.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `hard_panoptic` · `000000460147.jpg` · cụm truck ở giữa ảnh.
- Lỗi thuộc loại: **thừa vật (mask trùng)** — cùng một chiếc truck bị vẽ thành nhiều mask `truck` nằm đè nhau.
- Bằng chứng tôi nhìn thấy: chạy tự kiểm decode RLE trên ZIP export thấy 3 mask `truck` (annotation id 34, 41, 42) có IoU cặp nhau 0.81–0.94 (chồng lấn 14.112–16.621 pixel); ngoài ra mask truck này còn đè lên mask `car` ~4 nghìn pixel và tràn nhẹ sang `road`/`vegetation`. Panoptic yêu cầu mỗi thing chỉ một mask và các mask không chồng lấn, nên đây là lỗi thật, không phải sai số biên.
- Quy tắc và hành động sửa: theo quy tắc panoptic, xoá các mask `truck` dư trong CVAT, giữ đúng một mask cho chiếc xe đó, kiểm lại danh sách Objects rồi Save. Tôi **không sửa JSON bên trong ZIP bằng tay** vì repo cấm sửa export thủ công; phải sửa trong CVAT và export lại.
- Sau sửa đã Save và export lại chưa? **Chưa** — lỗi được phát hiện ở bước tự kiểm sau khi export; cần quay lại CVAT sửa mask, Save và export `hard_panoptic.zip` lần nữa, rồi thay file trong `submissions/`. Cần coach hỗ trợ xác nhận cách xử lý phần truck tràn sang `road`/`vegetation` khi vẽ lại. Bản `hard_panoptic.zip` hiện trong `submissions/` vẫn là bản trước khi sửa, chỉ việc thay khi có export mới.

Về tự đánh giá: chưa chạy scorer / chưa có điểm tự đánh giá (ground truth chưa dùng để chấm; không đưa đáp án vào fork).

**Lỗi thứ hai phát hiện bằng tự kiểm decode RLE (cùng trạng thái "chưa sửa"):** `cp1_holes` · `000000144300.jpg` — mask bị vẽ trùng: id 3 (`truck`) và id 6 (`bus`) là cùng một mask trên caravan (45.527 px, cùng bbox); id 5 (`bus`) và id 9 (`truck`) là cùng một mask trên xe van (11.093 px); vùng van bị ba mask đè nhau (id 4/5/8); hai mask `person` id 2/10 chồng gần trọn. Hành động cần làm: trong CVAT xoá các mask dư để mỗi xe/person còn đúng một mask, chọn một class cho từng xe rồi Save và export lại. Bản `cp1_holes.zip` hiện tại vẫn là bản chưa sửa.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `000000181542.jpg` (Medium) — người ngồi trên xe máy, mask `motorcycle` và `bicycle`/`person` đè nhau ~3.317 px | (1) Phần người và xe vẽ chồng nhau là chấp nhận được vì mỗi vật vẫn là instance riêng; (2) phải cắt đôi biên để hai mask không chạm nhau | Mỗi vật đếm được là một mask riêng; vẽ sát phần nhìn thấy của từng vật | Đã vẽ người là `person` riêng và xe là `motorcycle`/`bicycle` riêng, giữ chỗ giao nhau cho vật gần camera; câu hỏi: chồng lấn nhỏ giữa hai instance có bị trừ điểm biên không |
| `000000460147.jpg` (Hard) — ranh `road`–`sidewalk` ở mép đường, hai vùng màu ảnh gần giống nhau | (1) Cắt ranh theo đổi màu sáng/tối trên ảnh; (2) cắt theo bó vỉa/chức năng đi bộ | Ranh road–sidewalk xác định theo **chức năng và bó vỉa**, không chỉ theo màu | Đã cắt theo bó vỉa nhìn thấy được, kể cả chỗ màu không đổi rõ; câu hỏi cho coach: đoạn vỉa hè bị xe đè che bó vỉa thì cắt theo đường nối chức năng hay dừng tại phần nhìn thấy |
| `000000373353.jpg` + `000000181542.jpg` (Medium) — người ở rất xa, mask chỉ ~50–85 px | (1) Vẽ đủ mọi person nhìn thấy kể cả rất nhỏ; (2) chỉ vẽ người nhận diện chắc chắn để tránh nhiễu mask | "Đủ vật" theo phần nhìn thấy; không bịa nhãn cho đủ coverage | Đã vẽ cả các person nhỏ này (mỗi người một mask `person` riêng); câu hỏi: mask dưới ~100 px có nên giữ hay bỏ để tránh thành nhiễu ở bước chấm |
