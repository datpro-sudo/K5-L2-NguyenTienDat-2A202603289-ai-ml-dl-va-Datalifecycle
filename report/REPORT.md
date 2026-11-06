1. Phân loại ảnh
Mở file: day1_lab_outputs/classification_predictions.json và visuals/classification_top5.png.
Chỉ dùng ảnh sample_id = traffic, lấy dòng rank = 1.

    1.Chép record hạng 1: class_id=468 ; class_name=cab ; rank=1 ; score=0.510915 ; taxonomy_name=ImageNet-1K
    2.Record này nói về cả bức ảnh hay về một vật thể riêng trong ảnh? Vì sao bạn nghĩ vậy?
      - Record này nói về cả bức ảnh, không phải một vật thể riêng. Vì mô hình classification nhận toàn bộ ảnh làm đầu vào và  đưa ra các lớp dự đoán đại diện cho nội dung tổng thể của ảnh. Nó không cung cấp vị trí cụ thể của từng vật thể.
    3.Danh sách tên lớp (class list) do đâu mà có: model tự nghĩ ra, hay đi kèm taxonomy/checkpoint?
      - Các tên lớp đã có sẵn trong bộ dữ liệu phân loại đi kèm model, ở đây là ImageNet-1K. Model không tự tạo ra tên lớp mới mà dự đoán dựa trên danh sách lớp đã được sử dụng khi huấn luyện.
    4.Vì sao cần ghi cả class_id, class_name và taxonomy_name, không chỉ ghi mỗi tên lớp?
      - Cần ghi cả class_id, class_name và taxonomy_name để xác định lớp chính xác, tránh nhầm lẫn giữa các taxonomy có thể sử dụng tên hoặc cách đánh số lớp khác nhau. 
    5.Nếu trong ảnh có nhiều vật / nhiều chủ đề, guideline cần nói rõ điều gì để người gán nhãn chọn một lớp?
      -  Guideline cần quy định rõ cách chọn nội dung chính của bức ảnh. Ví dụ, nếu ảnh có nhiều phương tiện giao thông thì cần xác định sẽ chọn chủ đề chung của ảnh hay chọn một đối tượng nổi bật nhất. Nếu không có quy tắc rõ ràng, mỗi người có thể chọn một nhãn khác nhau cho cùng một bức ảnh.
    6.Vì sao không được lấy score của model làm ground truth?
      - Score chỉ thể hiện mức độ tự tin của model đối với kết quả dự đoán. Ví dụ, model dự đoán cab với score 0.510915, nhưng điều đó không có nghĩa chắc chắn ảnh thuộc lớp này.
      - Ground truth cần được con người xác định dựa trên hình ảnh và quy tắc gán nhãn, chứ không thể lấy trực tiếp từ kết quả model.
2. Phát hiện vật thể
Mở file: day1_lab_outputs/detection_predictions.json và visuals/detection_predictions.png.
Chỉ dùng ảnh sample_id = kitchen. Chọn một record.
Số lượng prediction theo ngưỡng: xem output ô so sánh 0.20 / 0.35 / 0.60 trong notebook.

    1.Chép một record: class_name=person; score=0.912625; bbox_xyxy = [385.33, 69.24, 498.92, 348.92]; bbox_width=113.58; bbox_height=279.68
    2.Giải thích box bằng lời thường: góc trên-trái ở đâu, góc dưới-phải ở đâu, rộng/cao bao nhiêu pixel?
      - Góc trên-trái ở (385.33, 69.24): cách mép trái khoảng 385 pixel và cách mép trên khoảng 69 pixel. Đó là khoảng đỉnh đầu của người đàn ông mặc áo trắng đeo dây treo quần.
      - Góc dưới-phải ở (498.92, 348.92): cách mép trái khoảng 499 pixel và cách mép trên khoảng 349 pixel. Đó là khoảng gót chân.
      - Hộp rộng khoảng 113.6 pixel (498.92 − 385.33) và cao khoảng 279.7 pixel (348.92 − 69.24). Hộp cao gần gấp 2.5 lần chiều rộng, hợp với dáng người đứng.
    3.Chọn hai ngưỡng (vd. 0.20 và 0.60). Mỗi ngưỡng có bao nhiêu prediction?

      - Ngưỡng 0.60: Có 6 prediction.
      - Ngưỡng 0.35: Có 11 prediction.

    Khi giảm ngưỡng confidence, model có thể giữ lại nhiều dự đoán hơn. Ví dụ, ở ngưỡng 0.35 có 11 prediction, trong khi ở ngưỡng 0.60 chỉ có 6 prediction.Điều này cho thấy ngưỡng confidence ảnh hưởng đến số lượng vật thể mà model đưa ra kết quả dự đoán. Tuy nhiên, nhiều prediction hơn không có nghĩa là tất cả đều chính xác.
    4.Khi hạ ngưỡng (giữ nhiều hộp hơn), người reviewer phải làm thêm việc gì?
      - Phải xem nhiều hộp hơn. Từ 0.60 xuống 0.35, thêm 5 hộp (bowl 0.50, bowl 0.46, cup 0.45, cup 0.38, bowl 0.38).
      - Với mỗi hộp mới, phải kiểm tra đúng/sai: đúng loại chưa, có phải vật thật không, hộp có ôm sát không.
      - Phải xóa hộp sai hoặc trùng, sửa hộp lệch, và đôi khi đổi nhãn.
      - Đổi lại, ít sót vật hơn (recall tăng), nhưng nhiều hộp sai hơn (precision giảm), tốn thời gian và công sức hơn.

    5.Viết một quy tắc box để người gán nhãn làm theo (áp dụng được cho nhiều ảnh, không chỉ ảnh này).
      - Mỗi vật thể thuộc danh mục cần được vẽ một Bounding Box riêng, bao quanh sát phần vật thể nhìn thấy được. Box không nên chừa quá nhiều phần nền xung quanh hoặc cắt mất phần vật thể nhìn thấy. Nếu có hai vật thể riêng biệt thì cần vẽ hai box riêng, không gộp chung thành một box.
    6.Trên ảnh kitchen, chỉ ra một vật bị che khuất hoặc cắt mép. Bạn sẽ viết thành quy tắc trong guideline, hay đưa lên hỏi Lab Coach (escalation)? Vì sao?
      - Khi quan sát ảnh kitchen, tôi thấy một trường hợp đáng chú ý là đối tượng person có score 0.61, với Bounding Box:
        [0.12, 263.18, 61.55, 310.78]
      - Đối tượng này nằm sát mép trái ảnh và chỉ nhìn thấy một phần cánh tay hoặc bàn tay. Ngoài ra, một số đối tượng bowl và oven ở phía bên trái cũng bị cắt một phần bởi mép ảnh.
      - Trường hợp này cần xử lý theo hai hướng:
         Thứ nhất, đưa vào guideline: Đối với vật thể bị cắt bởi mép ảnh, có thể quy định chỉ vẽ Bounding Box quanh phần vật thể nhìn thấy được và để cạnh box chạm vào mép ảnh nếu cần thiết.

         Thứ hai, hỏi Lab Coach (escalation): Tôi chưa chắc chắn trường hợp chỉ nhìn thấy một phần bàn tay hoặc cẳng tay thì có được gán nhãn person hay không. Ngoài ra, đối tượng được model nhận diện là oven ở bên trái ảnh cũng cần được kiểm tra lại để xác định có phù hợp với định nghĩa lớp hay không.

      - Em nghĩ những trường hợp chưa có quy định rõ ràng nên được hỏi Lab Coach trước, thay vì mỗi người tự đưa ra cách gán nhãn khác nhau. Điều này giúp dữ liệu được gán nhãn thống nhất hơn.

3. Phân đoạn theo từng đối tượng
Mở file: day1_lab_outputs/segmentation_predictions.json và visuals/segmentation_prediction.png.
Chỉ dùng ảnh sample_id = kitchen. Chọn một record.

    1.Chép một record: instance_id=kitchen-001 ; class_name=person ; score=0.899318 ; số điểm polygon=polygon_point_count=348 ; 3–5 điểm đầu của polygon_xy= [446, 70], [445, 71], [444, 71], [443, 72], [442, 72]
    2.So với box, polygon cho thêm thông tin gì về hình dạng vật thể?
      - Box là hình chữ nhật rộng khoảng 112.6 pixel và cao khoảng 282.1 pixel, nên gồm cả nền ở bốn góc (tường, dụng cụ treo, sàn).
      - Polygon đi theo dáng người: đầu tròn ở đỉnh (y ≈ 70), vai và hai tay chống hông ở hai bên, rồi thu hẹp xuống eo.
      - Phía dưới, polygon có một chỗ khuyết ở khoảng (432, 252) đến (435, 252). Đó là khe giữa hai chân, nơi box vẫn tính là “người”.
      - Nhờ vậy ta tính được diện tích thật của người và tách được người khỏi nền phía sau.
    3.instance_id dùng để làm gì? Nó không phải class ID và không phải tracking ID — bạn hiểu câu này thế nào?
      - Là tên riêng của từng vật thể cụ thể trong một ảnh. Ở đây kitchen-001 là người đàn ông mặc áo trắng. Reviewer dùng nó để chỉ đúng vật khi sửa hoặc báo lỗi.
      - Không phải class ID: class ID (person = 0) cho biết vật thuộc loại gì, nên mọi người đều có số 0. instance_id cho biết vật nào. Trong ảnh này có hai person (kitchen-001 và kitchen-009), hai instance khác nhau nhưng cùng lớp.
      - Không phải tracking ID: tracking ID theo dõi cùng một vật qua nhiều khung hình video. instance_id chỉ có nghĩa trong một ảnh tĩnh, sang ảnh khác thì không liên quan.
    4.Viết một quy tắc vẽ biên mask để người gán nhãn làm theo.
      - Vẽ polygon bám sát đường viền ngoài của phần nhìn thấy của vật, mỗi vật một mask riêng. Biên đi đúng trên mép vật, không lấn ra nền và không cắt vào trong vật. Khoảng trống thật giữa các phần của vật (ví dụ giữa hai chân, giữa tay và thân) thì không tô vào mask. Vật bị che thì chỉ vẽ phần thấy được, không tự đoán phần khuất.
    5.Trên ảnh kitchen, chỉ ra một chỗ biên mờ / hai vật chạm nhau / bị che. Bạn dùng guideline hay escalation? Vì sao?
      - Một số dụng cụ như cán gỗ, muôi chạm sát vào người nên khó phân biệt ranh giới. Phần quần và giày tối màu cũng khó xác định đường viền với nền. Ngoài ra, một phần cơ thể và hai bàn tay bị che khuất do người trong ảnh quay lưng.
      - Dùng guideline hay escalation? Nên dùng guideline cho các trường hợp đã có quy tắc rõ ràng như bám theo phần nhìn thấy, không tô khoảng trống giữa hai chân và tách riêng mask của từng vật thể. Với những trường hợp chưa rõ như biên quá mờ hoặc vật dụng nằm sát người có thuộc mask person hay không, cần hỏi Lab Coach để thống nhất cách xử lý.
      - Lý do: Việc thống nhất quy tắc giúp mọi người gán nhãn nhất quán và hạn chế sai lệch trong bộ dữ liệu.
4. Vòng đời và kiểm tra chất lượng
Lab hôm nay chỉ xem prediction. Chuỗi dưới đây là vòng đời production để bạn nối các khái niệm:

ảnh thô → guideline → ground truth → huấn luyện → prediction → QC/rework

Điền bảng (viết ngắn, có dẫn chứng từ JSON/PNG khi nói về lỗi):

### Bảng vòng đời production (viết lại, có dẫn chứng phân loại)

| Tác vụ | Ground truth trông như thế nào? (đơn vị nhãn) | Lỗi hoặc chỗ khó quyết định bạn thấy trên ảnh/JSON | Annotator làm gì? | Reviewer xem gì? |
|---|---|---|---|---|
| **Phân loại ảnh (Image Classification)** | Một nhãn (`class_id`, `class_name`, `taxonomy_name`) cho cả ảnh, ví dụ `468 / cab / ImageNet-1K`. Đơn vị nhãn = **1 ảnh**, không có vị trí. | Ảnh `traffic` có nhiều bus (`class 5`) và car (`class 2`) trong file detection, nhưng prediction phân loại chỉ ra `cab` với score `0.510915`, không nói xe nào, ở đâu. Score thấp, các hạng 2 đến 5 cạnh tranh. Chỗ khó: ảnh nhiều chủ đề thì chọn lớp theo vật nào. | Chọn **một** lớp mô tả chủ đề chính theo guideline (vật lớn nhất hoặc nổi bật nhất), ghi đủ ID, tên, taxonomy. Không lấy top-1 của model làm đáp án. Ảnh mơ hồ thì hỏi Lab Coach. | Nhãn có khớp chủ đề chính không, có nhất quán với các ảnh tương tự không, có đúng taxonomy không (`468` chỉ đúng nghĩa trong ImageNet-1K). Dự đoán sai hay score thấp của model được dùng để tìm ảnh khó. |
| **Phát hiện vật thể (Object Detection)** | Danh sách hộp `bbox_xyxy` kèm `class_name`. Đơn vị nhãn = **1 hộp quanh 1 vật** (kitchen có 11 prediction ở ngưỡng 0.35, taxonomy COCO-80). | Person `0.61` `[0.12, 263.18, 61.55, 310.78]` chỉ là một phần tay ở mép trái, bị cắt mép. Oven `0.63` `[489.09, 200.73, 617.81, 344.88]` là bồn rửa và tủ đen, oven `0.69` bên trái là bếp lò kiểu cũ: ranh giới lớp `oven` không rõ. Bowl `0.38` `[0.32, 305.02, 87.69, 371.32]` chạm mép ảnh. | Vẽ một hộp ôm sát phần nhìn thấy, đúng lớp. Vật bị cắt mép thì hộp chạm mép ảnh. Câu hỏi định nghĩa lớp (bếp lò cũ có là `oven` không?) thì đưa lên Lab Coach. | Có sót vật không (recall), có hộp thừa hoặc sai lớp không (precision), hộp có sát vật không. Xem kỹ hộp điểm thấp như cup `0.45`, bowl `0.38`. |
| **Phân đoạn từng đối tượng (Instance Segmentation)** | Mỗi vật một **mask/polygon** (`polygon_xy`) kèm `class_name` và `instance_id`. Đơn vị nhãn = **1 mask của 1 vật**. | Dining table `0.58` (kitchen-005) có 598 điểm, mask phủ cả bát, chai, bột trên bàn và có điểm lộn xộn. Oven `0.49` (kitchen-007) bị bàn, bowl và người chồng lên, biên dưới không rõ. Bowl `0.53` (kitchen-006), bowl `0.70` (kitchen-003) và person `0.43` (kitchen-009) nằm sát nhau ở góc dưới trái. | Vẽ polygon bám đường viền phần nhìn thấy, mỗi vật một mask, không chồng, không để hở khe giữa hai vật chạm nhau. Câu hỏi định nghĩa (mask bàn có khoét đồ trên bàn không?) thì escalation. | Mask có lấn ra nền hoặc cắt vào vật không, vật chạm nhau có bị gộp hoặc chồng mask không, `instance_id` có tách đúng từng vật không, mask có theo guideline không. |

5. An toàn dữ liệu
  1.Quy tắc bảo vệ dữ liệu của bạn cho bài lab này là gì? (viết 1 câu)
    - Chỉ dùng ảnh và file lab được giao (traffic, kitchen, dining) cho đúng mục đích bài học, không sao chép, chia sẻ hay đưa ra ngoài môi trường lab, và không nhập dữ liệu nhạy cảm hay dữ liệu cá nhân thật vào công cụ nào khác.
  2.Nếu gặp ảnh hoặc dữ liệu không đúng phạm vi, bạn sẽ dừng và báo cho ai?
    - Dừng ngay: không gán nhãn, không chạy tiếp, không tải lên hay chia sẻ, giữ nguyên hiện trạng để người phụ trách kiểm tra.
    - Báo Lab Coach (người hướng dẫn lab, cũng là người nhận escalation ở các phần trước). Nếu lab có quy định kênh báo riêng, hoặc người phụ trách an toàn dữ liệu, thì báo theo kênh đó.
    - Nội dung báo ngắn gọn: tên file hoặc sample_id, điều gì khiến bạn thấy sai phạm vi (ví dụ ảnh có mặt người rõ nét, biển số, thông tin cá nhân, hoặc không thuộc bộ ảnh lab), và bạn đã dừng ở bước nào.
  
6. Checklist trước khi nộp link repo
Kiểm tra trên GitHub trong thư mục report/:

 REPORT.md (file này đã điền)
 day1_lab_outputs/classification_predictions.json
 day1_lab_outputs/detection_predictions.json
 day1_lab_outputs/segmentation_predictions.json
 day1_lab_outputs/IMAGE_ATTRIBUTION.md
 day1_lab_outputs/visuals/classification_top5.png
 day1_lab_outputs/visuals/detection_predictions.png
 day1_lab_outputs/visuals/segmentation_prediction.png
 Ô validation cuối notebook báo PASS
 Không có họ tên, MSSV hoặc dữ liệu nhạy cảm trong báo cáo/output
 Đã nộp link repository trên LMS

