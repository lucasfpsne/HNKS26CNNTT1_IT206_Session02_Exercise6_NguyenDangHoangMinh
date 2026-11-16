# BÁO CÁO PHÂN TÍCH LỖI VÀ TỐI ƯU PROMPT STRUCTURED OUTPUT

## 1. Phân tích nguyên nhân lỗi

### 1.1. Hiện trạng hệ thống

Hệ thống ERP của TechShop sử dụng API AI để trích xuất thông tin từ biên bản tiếp nhận bảo hành và chuyển đổi thành dữ liệu JSON phục vụ việc lưu trữ vào Database.

Tuy nhiên, hệ thống backend gặp thông báo lỗi:

`JSON.parse Error: Unexpected token 'D' in position 0`

### 1.2. Nguyên nhân AI tự động sinh lời chào và lời chúc

- Các mô hình ngôn ngữ lớn (LLM) được huấn luyện trên nhiều dữ liệu hội thoại của con người.
- AI được tối ưu để giao tiếp tự nhiên, lịch sự và thân thiện với người dùng.
- AI thường bổ sung lời chào, lời giải thích và lời kết luận để phản hồi có tính hội thoại.
- Prompt ban đầu chưa quy định nghiêm ngặt định dạng đầu ra.
- Thiếu kỹ thuật Negative Constraint để ngăn AI sinh nội dung ngoài yêu cầu.

### 1.3. Nguyên nhân kỹ thuật của lỗi JSON.parse

Hàm `JSON.parse()` yêu cầu chuỗi đầu vào phải tuân thủ cú pháp JSON hợp lệ.

Tuy nhiên, kết quả AI trả về bắt đầu bằng chữ "D" trong từ "Dạ", thay vì ký tự `{` của đối tượng JSON cần trích xuất.

Ngoài ra, lời chúc ở cuối phản hồi cũng là nội dung không hợp lệ đối với JSON.

Điều này khiến backend không thể phân tích dữ liệu và làm gián đoạn quá trình lưu trữ vào Database.

### 1.4. Hậu quả

- Hệ thống backend không thể xử lý dữ liệu.
- Quy trình tự động hóa bị gián đoạn.
- Dữ liệu bảo hành không được lưu vào Database.
- Tăng thời gian và chi phí xử lý lỗi thủ công.

## 2. Hoàn thiện Prompt ép cấu trúc

### 2.1. Kỹ thuật áp dụng

- **Structured Output:** Quy định cấu trúc dữ liệu đầu ra.
- **Negative Constraint:** Thiết lập các điều kiện cấm đối với AI.
- **Delimiters:** Phân tách dữ liệu đầu vào với câu lệnh.
- **JSON Schema:** Xác định các trường dữ liệu và kiểu dữ liệu mong muốn.

### 2.2. Prompt hoàn chỉnh

Bạn là một hệ thống trích xuất dữ liệu tự động cho hệ thống ERP của TechShop.

Nhiệm vụ của bạn là đọc biên bản tiếp nhận bảo hành và trích xuất chính xác ba trường thông tin:

- customer_name: Tên khách hàng.
- device: Tên thiết bị.
- issue: Lỗi hoặc vấn đề của thiết bị.

Yêu cầu định dạng đầu ra:

- Chỉ trả về một đối tượng JSON hợp lệ.
- Sử dụng đúng ba trường: customer_name, device và issue.
- Giá trị của các trường phải có kiểu chuỗi hoặc null.
- Nếu thiếu thông tin, sử dụng giá trị null.
- Không tự ý suy đoán hoặc bổ sung dữ liệu.

Negative Constraint (Ràng buộc cấm):

- Không thêm lời chào hỏi.
- Không thêm lời cảm ơn hoặc lời chúc.
- Không giải thích kết quả.
- Không sử dụng Markdown Code Block.
- Không thêm trường dữ liệu ngoài yêu cầu.
- Không thêm bất kỳ ký tự hoặc văn bản nào bên ngoài đối tượng JSON.
- Không sử dụng dấu phẩy thừa ở cuối đối tượng.

Chỉ trả về chuỗi JSON thuần túy, không dùng Markdown Block, không giải thích, bắt đầu bằng ký tự { và kết thúc bằng ký tự }.

Dữ liệu đầu vào:

"""
Khách hàng: Trần Văn A
Thiết bị: Laptop Dell XPS 15
Tình trạng bảo hành: Liệt phím Space
"""

## 3. Kết quả chạy thử Prompt

{
  "customer_name": "Trần Văn A",
  "device": "Laptop Dell XPS 15",
  "issue": "Liệt phím Space"
}

## 4. Đánh giá kết quả

| Tiêu chí | Trước tối ưu | Sau tối ưu |
|---|---|---|
| Có lời chào hỏi | Có | Không |
| Có lời chúc kết luận | Có | Không |
| Có văn bản thừa | Có | Không |
| Đúng cú pháp JSON | Không | Có |
| Đầy đủ trường dữ liệu | Có | Có |
| Sử dụng Markdown Block | Không | Không |
| Bắt đầu bằng ký tự { | Không | Có |
| Kết thúc bằng ký tự } | Không | Có |
| Khả năng xử lý bằng JSON.parse() | Thất bại | Thành công với mẫu này |
| Khả năng tích hợp vào ERP | Hạn chế | Tốt |

## 5. Giải pháp đảm bảo độ ổn định

Để tăng độ tin cậy của hệ thống trong môi trường thực tế, cần kết hợp:

- Sử dụng Structured Outputs với JSON Schema ở cấp độ API nếu mô hình hỗ trợ.
- Kiểm tra tính hợp lệ của JSON trước khi lưu vào Database.
- Kiểm tra kiểu dữ liệu và các trường bắt buộc.
- Sử dụng try...catch để xử lý ngoại lệ.
- Thiết lập cơ chế thử lại khi AI trả về dữ liệu không hợp lệ.

Lưu ý: Chỉ sử dụng Prompt không thể đảm bảo tỷ lệ parse thành công 100% trong mọi trường hợp. Kết quả chạy thử trên là kết quả minh họa trực tiếp, chưa được kiểm thử qua API hoặc hệ thống ERP thực tế.

## 6. Kết luận

Nguyên nhân chính của lỗi là Prompt chưa thiết lập các ràng buộc nghiêm ngặt về định dạng đầu ra, khiến AI tự động bổ sung nội dung mang tính hội thoại.

Việc kết hợp Negative Constraint, Structured Output và Delimiters giúp hạn chế văn bản thừa và tạo ra dữ liệu JSON đúng cấu trúc.

Kết hợp thêm JSON Schema ở cấp độ API và cơ chế kiểm tra dữ liệu tại backend sẽ giúp nâng cao độ ổn định của hệ thống ERP TechShop, giảm lỗi và tối ưu hóa quy trình tự động hóa.
