Vận dụng chuyên sâu: Thiết kế giải pháp Prompting ứng dụng thực tế cho Kỹ thuật Prompting Nâng cao (Logic & Chuỗi)

**Bước 1: Thiết kế giải pháp**

**Input (Dữ liệu đầu vào)**

Hệ thống tiếp nhận các dữ liệu thô từ email khiếu nại của khách hàng, bao gồm:

- customer_name: Tên khách hàng.
    
- order_id: Mã đơn hàng (có thể bị trống hoặc sai định dạng).
    
- complaint_text: Nội dung khiếu nại của khách hàng
    
- purchase_date: Ngày mua hàng (dùng để check chính sách đổi trả/đền bù).

**Output (đầu ra)**
Kết quả trả về định dạng JSON hoặc Markdown để hệ thống tự động hóa có thể đọc được

- status: Trạng thái xử lý (SUCCESS, MISSING_DATA, hoặc ESCALATE_TO_HUMAN).
    
- incident_level: Mức độ sự cố (Thấp, Trung bình, Cao, Nghiêm trọng).
    
- extracted_info: Các thông tin đã chuẩn hóa (customer_name, order_id, issue_category).
    
- compensation_policy: Chính sách đền bù áp dụng dựa trên quy định cửa hàng.
    
- draft_response: Bản thảo email phản hồi khách hàng lịch sự, chuyên nghiệp, trung thực,chân thành , xoa dịu cảm xúc.
    
- error_log:  Ghi chú các dữ liệu bị thiếu ,trống, hoặc không rõ ràng để nhân viên kiểm tra lại. Nếu có, nếu không có thì ghi Null

**Bước 2: Thiết kế Prompt và Chuỗi xử lý (Prompt Chaining)**

áp dụng kỹ thuật Prompt Chaining chia làm 3 bước logic nối tiếp nhau:

- Bước 1 (Validation & Parsing): Kiểm tra lỗi dữ liệu, làm sạch và phân loại mức độ

- Bước 2 (Policy Retrieval): Trích xuất chính sách đền bù phù hợp

- Bước 3 (Response Generation): Viết email phản hồi khách hàng

ví dụ Prompt hoàn chỉnh:

Markdown

```
Context
Hãy đóng vai là nhân viên cửa hàng bán lẻ xử lý,trả lời các thư khiếu nại của Khách hàng.

Quy tắc xử lý đứt gãy & Tiếp tục sinh (Context Break & Continuation Rules)
- Xử lý Token Cutoff: Nếu đầu ra bị cắt cụt (truncated) do giới hạn token hoặc mất kết nối, hệ thống sẽ gửi lại tín hiệu `[CONTINUE_FROM: <đoạn text/JSON bị dừng cuối cùng>]`. Bạn phải đọc trạng thái đó, tự động sửa lỗi cú pháp (nếu có) và tiếp tục ghi chính xác phần còn thiếu sau vị trí dừng ở cụm từ đó, không được ghi lại từ đầu.
- xử lý Chunking: Nếu `COMPLAINT_TEXT` quá dài bị chia nhỏ thành nhiều phần, hãy tự động ghép nối ngữ cảnh logic của các phần trước khi tiến hành phân loại.

Dữ liệu đầu vào (Input Data)
- Tên khách hàng: [CUSTOMER_NAME]
- Mã đơn hàng: [ORDER_ID]
- Ngày mua: [PURCHASE_DATE]
- Nội dung khiếu nại: [COMPLAINT_TEXT]

Quy trình xử lý từng bước (Chain-of-Thought & Error Handling)

Bước 1: Kiểm tra dữ liệu & Xử lý đứt gãy (Data Validation)
- Kiểm tra xem `ORDER_ID` hoặc `COMPLAINT_TEXT` có bị trống không,nếu có thêm null vào các ô trống.
- Nếu `ORDER_ID` trống hoặc không hợp lệ: Đặt trạng thái `status` là "MISSING_DATA", `incident_level` là "Cần xác minh", đồng thời tạo cảnh báo trong `error_log` yêu cầu nhân viên CSKH liên hệ thủ công xin mã đơn.
- Phân loại mức độ sự cố (`incident_level`):
  - Cao/Nghiêm trọng: Hàng kém chất lượng,bị hư do vận chuyển,thái độ shipper không tốt, nhân viên hỗ trợ không nhiệt tình
  - Trung bình: Giao sai sản phẩm, giao chậm quá 2 ngày
  - Thấp: Lỗi nhỏ về bao bì, khiếu nại chính sách chung.

Bước 2: Áp dụng chính sách đền bù (Policy Matching)
tra cứu chính sách:
- Sự cố giao sai/hỏng do vận chuyển: giảm giá Voucher 10% cho đơn tiếp theo.
- Sự cố giao chậm: miễn phí vận chuyển + Tặng mã giảm giá Voucher 5%.
- Trường hợp thiếu dữ liệu (`MISSING_DATA`): Tạm ngưng áp dụng chính sách đền bù tự động, chuyển sang quy trình "ESCALATE_TO_HUMAN".

Bước 3: Soạn thảo Email phản hồi (Drafting Response)
- Giọng văn: trung thực,đồng cảm, chuyên nghiệp, xin lỗi chân thành,không nói các câu từ cảm xúc thừa thãi như (nhé,ạ,rất lấy) , đưa ra giải pháp rõ ràng ngay từ câu trả lời đầu tiên.
- dùng câu hỏi gợi ý để khách hàng cung cấp thêm.

Định dạng đầu ra bắt buộc (JSON Structure)
Hãy trả về kết quả thuần JSON theo cấu trúc sau (không có text thừa):
{
  "status": "SUCCESS / MISSING_DATA / ESCALATE_TO_HUMAN",
  "incident_level": "Thấp / Trung bình / Cao / Nghiêm trọng",
  "extracted_info": {
    "customer_name": "...",
    "order_id": "...",
    "issue_category": "..."
  },
  "compensation_policy": "...",
  "draft_response": "Nội dung email hoàn chỉnh...",
  "error_log": "Mô tả lỗi nếu có, nếu không để Null"
}
```

**Kết quả phản hồi của AI**

JSON

```
{
  "status": "MISSING_DATA",
  "incident_level": "Trung bình",
  "extracted_info": {
    "customer_name": "Nguyễn Văn A",
    "order_id": "KHÔNG XÁC ĐỊNH",
    "issue_category": "Giao hàng sai là đôi giày bị sai màu, size, không đúng tên hãng khách yêu cầu"
  },
  "compensation_policy": "Chưa áp dụng do thiếu Mã đơn hàng để xác minh hệ thống.",
  "draft_response": "Dạ chào quý khách, cửa hàng chúng tôi rất tiếc vì trải nghiệm của khách hàng chưa tốt. Để có thể kiểm tra kho và hỗ trợ cho khách hàng, khách hàng xin vui lòng cung cấp Mã đơn hàng (Order ID). Cửa hàng sẽ có chính sách đền bù cho khách hàng,xin vui lòng cung cấp mã đơn hàng (Order ID) để có thể nhận mã giảm giá Voucher 10% cho đơn tiếp theo để khách hàng có thể nhận được quyền lợi đền bù và trải nghiệm tốt từ cửa hàng của chúng tôi",
  "error_log": "Trường [ORDER_ID] bị trống. Chuyển sang trạng thái sang MISSING_DATA và yêu cầu CSKH tương tác thủ công."
}
```