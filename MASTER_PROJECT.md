MASTER PROJECT CONTEXT — AI E-COMMERCE CHATBOT

Mục đích của file: Đây là tài liệu nguồn ngữ cảnh chính (Source of Truth) của dự án. Khi mở lại dự án ở cuộc trò chuyện khác, chỉ cần cung cấp file này và yêu cầu “đọc MASTER_PROJECT.md và tiếp tục từ CURRENT STATE”. Không cần giải thích lại toàn bộ kiến trúc.

1. PROJECT OVERVIEW
   Tên dự án

Xây dựng hệ thống AI Chatbot hỗ trợ bán hàng và chăm sóc khách hàng cho cửa hàng giày dép trên Facebook Messenger.

Mục tiêu

Xây dựng chatbot thương mại điện tử có khả năng:

Tự động tư vấn khách hàng.
Tìm kiếm sản phẩm.
Kiểm tra tồn kho thực tế từ PostgreSQL.
Cá nhân hóa tư vấn dựa trên hồ sơ khách hàng.
Tự động lưu thông tin khách hàng.
Nhận diện và tìm kiếm sản phẩm từ hình ảnh.
Hỗ trợ tạo đơn hàng.
Hỗ trợ thanh toán bằng VietQR.
Trả lời FAQ nhanh mà không cần gọi AI.
Có Human-in-the-Loop khi AI không nên tiếp tục xử lý.
Có Guardrail chống Prompt Injection.
Có Output Validation.
Có AI Monitoring/Logging.
Có RAG cho FAQ và chính sách.
Có kiến trúc Multi-Agent bằng n8n + Gemini API.
Có khả năng mở rộng thành hệ thống AI Commerce hoàn chỉnh.

2.  TECHNOLOGY STACK
    Core
    n8n
    PostgreSQL
    Gemini API
    Facebook Messenger / Messenger Platform
    AI
    Gemini API
    Gemini Vision
    LangChain / AI Agent nodes trong n8n
    Postgres Chat Memory
    Data
    PostgreSQL
    Có thể mở rộng
    RAG / Vector Store
    VietQR
    Telegram/Zalo hoặc kênh thông báo nhân viên
    Looker Studio / Power BI 3. KIẾN TRÚC TỔNG THỂ
    FACEBOOK MESSENGER
    │
    ▼
    ┌─────────────┐
    │ WEBHOOK │
    └──────┬──────┘
    │
    ▼
    ┌──────────────────────┐
    │ DATA PROCESSOR │
    │ │
    │ Verify Webhook │
    │ Extract IDs │
    │ Normalize Data │
    │ Generate Session ID │
    │ Timestamp │
    └──────────┬───────────┘
    │
    ▼
    ┌──────────────────┐
    │ MESSAGE ROUTER │
    └────────┬─────────┘
    │
    ┌─────────────┼─────────────┐
    │ │ │
    TEXT IMAGE OTHER
    │ │ │
    ▼ ▼ ▼
    INPUT GUARDRAIL GEMINI VISION HANDLE
    │ │
    ▼ ▼
    QUICK REPLY IMAGE ANALYSIS
    │ │
    MISS │
    └──────┬──────┘
    ▼
    DEBOUNCE
    │
    ▼
    CUSTOMER RESOLVER
    │
    ▼
    HANDOFF STATUS CHECK
    │ │
    ACTIVE NORMAL
    │ │
    ▼ ▼
    STOP AI CHAT MEMORY
    │ │
    ALERT STAFF ▼
    SUPERVISOR
    │
    ┌────────────┼────────────┐
    ▼ ▼ ▼
    SALES CRM ORDER
    AGENT AGENT AGENT
    │ │ │
    ▼ ▼ ▼
    TOOLS TOOLS TOOLS
    │ │ │
    └────────────┼────────────┘
    │
    ▼
    AI RESPONSE
    │
    ▼
    OUTPUT VALIDATOR
    │ │
    FAIL PASS
    │ │
    RETRY ▼
    SEND
    │
    ▼
    MESSENGER 4. NGUYÊN TẮC KIẾN TRÚC

Không xây toàn bộ hệ thống thành một workflow n8n khổng lồ.

Hệ thống được chia thành các workflow/module:

WF01 — Messenger Ingestion
WF02 — Text AI Processing
WF03 — Visual Search
WF04 — Customer Intelligence
WF05 — Sales Agent
WF06 — Order & Payment
WF07 — Human Handoff
WF08 — AI Monitoring

Các workflow có thể gọi nhau bằng sub-workflow/webhook phù hợp.

5. DATABASE

PostgreSQL là database trung tâm.

Bảng chính
customers
customer_preferences
customer_events

products
product_variants
inventory

conversations

orders
order_items
payments

faq

human_handoffs

ai_logs 6. CUSTOMER PROFILE

Customer Profile là dữ liệu nghiệp vụ, khác với Chat Memory.

customers

Lưu thông tin khách hàng:

customer_id
messenger_id
full_name
phone
address
created_at
updated_at
customer_preferences

Lưu thông tin phục vụ cá nhân hóa:

preference_id
customer_id
shoe_size
foot_length
budget_min
budget_max
favorite_brands
favorite_colors
favorite_styles
notes
updated_at

Thông tin cần hỗ trợ lưu tự động:

Tên
Số điện thoại
Địa chỉ
Size giày
Chiều dài bàn chân
Ngân sách
Thương hiệu yêu thích
Màu yêu thích
Kiểu dáng yêu thích
customer_events

Theo dõi hành vi:

event_id
customer_id
event_type
product_id
event_data
created_at

Các event ví dụ:

VIEW_PRODUCT
SEARCH_PRODUCT
ASK_PRICE
ASK_SIZE
ADD_TO_CART
PURCHASE 7. CHAT MEMORY

Postgres Chat Memory chỉ có nhiệm vụ lưu lịch sử hội thoại.

AI Agent
↓
Postgres Chat Memory
↓
session_id

Không dùng Connected Chat Trigger Node.

Messenger sử dụng Webhook nên Session ID phải được tự tạo.

Session ID
{{ $json.messenger_id + '_' + $json.page_id }}

Ví dụ:

27855554264111350_1247306088468500 8. CURRENT STATE — TRẠNG THÁI HIỆN TẠI
Đã hoàn thành
Database
PostgreSQL database đã được tạo.
Các bảng nghiệp vụ chính đã được tạo theo kiến trúc dự án.
Database hướng đến hệ thống bán giày dép.
Database không chỉ lưu sản phẩm mà còn có Customer, Order, Payment, Human Handoff và AI Logs.
Messenger
Facebook Messenger Webhook đã hoạt động.
Messenger gửi được dữ liệu vào n8n.
Production Webhook đã được xác nhận.
Đã nhận được message thực tế từ Messenger.

Dữ liệu Webhook thực tế đã xác nhận dạng:

body
└── entry
└── messaging
├── sender.id
├── recipient.id
├── timestamp
└── message
└── text

Ví dụ message text được lấy bằng:

{{ $('Webhook').item.json.body.entry[0].messaging[0].message.text }}
Workflow cũ

Workflow demo ban đầu có:

Webhook
↓
IF
↓
Text
↓
AI
↓
Response

Trong đó IF dùng để phân loại:

TEXT → xử lý
NOT TEXT → bỏ qua

Kiến trúc này không còn là kiến trúc mục tiêu.

AI
Đã kết nối Gemini API.
Đã sử dụng AI Agent trong n8n.
Đã thử sử dụng Postgres Chat Memory.
Memory

Đã gặp lỗi:

Key parameter is empty

và:

Provide a key to use as session ID
or use the Connected Chat Trigger Node

Nguyên nhân: Messenger dùng Webhook chứ không phải Chat Trigger.

Giải pháp đã thống nhất:

messenger_id + page_id
↓
session_id
↓
Postgres Chat Memory 9. ĐANG TRIỂN KHAI

Đang xây dựng lại workflow từ đầu theo kiến trúc hoàn chỉnh.

Ưu tiên hiện tại:

Data Processor
↓
Message Router
↓
Customer Resolver
↓
Handoff Status Check
↓
Chat Memory
↓
Supervisor

Sau đó mới lần lượt triển khai các Agent và module nâng cao.

10. CHƯA TRIỂN KHAI HOÀN CHỈNH

Các module sau chưa hoàn thiện:

□ Data Processor hoàn chỉnh
□ Message Router hoàn chỉnh
□ Text Processing hoàn chỉnh
□ Image Processing
□ Gemini Vision
□ Visual Product Search

□ Customer Resolver hoàn chỉnh
□ Customer Profile Auto Extraction
□ Customer Preference Auto Update
□ Customer Event Tracking

□ Handoff Status Check
□ Handoff Detector
□ Human Handoff
□ Staff Notification

□ Quick Reply / Fast Track
□ Debounce
□ Input Guardrail

□ Supervisor Agent
□ Sales Agent
□ CRM Agent
□ Order Agent

□ Product Tools
□ Variant Tools
□ Inventory Tools
□ Customer Tools
□ Order Tools
□ Payment Tools

□ RAG
□ VietQR

□ Output Validator
□ Retry Logic

□ AI Logs
□ Token Tracking
□ Latency Tracking
□ Cost Tracking
□ Error Tracking

□ Dashboard / Monitoring
□ End-to-End Testing 11. MESSAGE PROCESSING

Message Router phải phân loại ít nhất:

TEXT
IMAGE
OTHER

Không được dùng logic cũ:

TEXT → xử lý
OTHER → bỏ
TEXT
TEXT
↓
Input Guardrail
↓
Quick Reply
↓
Debounce
↓
Customer Resolver
↓
Handoff Status Check
↓
Chat Memory
↓
Supervisor
IMAGE
IMAGE
↓
Download Image
↓
Gemini Vision
↓
Structured Attributes
↓
Product Search
↓
Variant Search
↓
Inventory
↓
Sales Agent
OTHER

Có thể xử lý fallback tùy loại message.

12. VISUAL SEARCH

Gemini Vision dùng để phân tích ảnh giày.

Có thể trích xuất:

category
brand
color
style
features

Ví dụ:

{
"category": "sneaker",
"brand": "Nike",
"color": "black",
"style": "running",
"features": [
"low-top",
"white sole"
]
}

Không được coi kết quả Vision là bằng chứng sản phẩm chính xác.

Vision chỉ tạo thuộc tính.

Sau đó:

Vision
↓
Structured Attributes
↓
PostgreSQL Product Search
↓
Database Verification 13. CUSTOMER INTELLIGENCE

Đây là module quan trọng.

AI phải có khả năng tự phát hiện thông tin khách hàng từ hội thoại.

Ví dụ:

“Mình tên Hoàng, số điện thoại 090xxxxxxx, ở quận 7, chân size 42, ngân sách khoảng 2 triệu.”

AI trích xuất:

{
"full_name": "Hoàng",
"phone": "090xxxxxxx",
"address": "Quận 7",
"shoe_size": 42,
"budget_max": 2000000
}

Sau đó CRM Agent cập nhật PostgreSQL.

Nếu khách chỉ nói:

“Mình thích Nike.”

Chỉ lưu:

favorite_brands = Nike

Không tự suy ra:

shoe_size
budget
address

AI không được tự tạo dữ liệu khách hàng chưa được cung cấp.

Nếu khách cập nhật:

Size 42 → Size 43

thì cập nhật profile hiện tại thay vì tạo khách hàng mới.

14. SUPERVISOR + MULTI-AGENT

Kiến trúc:

SUPERVISOR
│
├── SALES AGENT
├── CRM AGENT
└── ORDER AGENT

Không bắt buộc mỗi Agent phải dùng một model khác nhau.

Có thể dùng:

Gemini API

cho toàn bộ hệ thống.

Agent khác nhau ở:

System Prompt
Tools
Responsibilities
Permissions 15. TOOLS

Tool phải được gắn trực tiếp vào Agent dưới dạng tool/sub-node phù hợp với n8n LangChain.

Sales Agent
Product Search Tool
Variant Search Tool
Inventory Tool
RAG Tool
CRM Agent
Customer Lookup
Update Customer
Update Preferences
Customer Event
Order Agent
Create Order
Order Lookup
Inventory Update
Payment
VietQR

Không thiết kế thành một khối Tools chạy tuần tự sau Agent.

Agent phải có khả năng tự quyết định tool nào cần sử dụng.

16. SALES AGENT

Sales Agent có quyền truy cập:

Products
Product Variants
Inventory
Customer Profile
RAG

Ví dụ:

“Tìm cho tôi giày chạy Nike dưới 2 triệu, size 42.”

Logic:

Customer Profile
↓
Product Search
↓
Category = Running
↓
Brand = Nike
↓
Variant = Size 42
↓
Stock > 0
↓
Price < 2M
↓
Recommendation

AI không được tự bịa tồn kho.

Tồn kho phải lấy từ PostgreSQL.

17. ORDER AGENT

Luồng:

Customer
↓
Order Agent
↓
Create Order
↓
Order Items
↓
Reserve / Deduct Stock
↓
Payment
↓
VietQR
↓
Messenger

Ví dụ:

Order ID: ORD-1024
Amount: 1.290.000đ
Transfer Content: ORD-1024

Thông tin Order và Payment phải được lưu database.

18. QUICK REPLY / FAST TRACK

Các câu hỏi cố định không nên gọi Gemini nếu không cần.

Ví dụ:

Shop ở đâu?
Phí ship?
Đổi trả?
Giờ mở cửa?

Luồng:

Message
↓
Quick Reply Matcher
↓
HIT → Instant Response
MISS → AI

Mục tiêu:

Giảm latency.
Giảm token.
Giảm chi phí API.
Tăng độ ổn định. 19. DEBOUNCE

Nếu khách gửi liên tục:

shop ơi
còn Nike không
size 42 ấy

Không gọi Gemini ba lần.

Hệ thống cần có cơ chế debounce/gom message để xử lý thành một lượt khi phù hợp.

20. HUMAN HANDOFF

Có hai lớp kiểm tra.

Lớp 1 — Handoff Status Check

Đặt trước AI.

Customer Resolver
↓
Handoff Status Check
↓
is_human_active?

Nếu:

TRUE

→ STOP AI.

Không gọi Gemini.

Không gọi Specialist Agent.

Không để AI trả lời đè nhân viên.

Có thể thông báo/ghi nhận trạng thái cho nhân viên.

Lớp 2 — Handoff Detector

Đặt sau AI hoặc trong quá trình xử lý phù hợp.

Kiểm tra:

Keyword
Intent
Sentiment
Confidence

Ví dụ:

“Cho tôi gặp nhân viên.”
“Khiếu nại.”
“Hoàn tiền.”

→ tạo human_handoffs.

human_handoffs
↓
status = active
↓
Notify Staff
↓
AI Paused 21. INPUT GUARDRAIL

Input Guardrail nằm sau Message Router đối với TEXT.

Không đặt một text guardrail trước Router để xử lý mọi loại dữ liệu.

Kiến trúc:

Message Router
│
├── TEXT
│ ↓
│ Input Guardrail
│
└── IMAGE
↓
Vision Pipeline

Guardrail có nhiệm vụ phát hiện yêu cầu bất thường như Prompt Injection/Jailbreak hoặc yêu cầu trái với quy tắc hệ thống.

22. OUTPUT VALIDATOR

AI không được gửi response ngay lập tức.

Agent
↓
Output Validator
↓
PASS → SEND
FAIL → RETRY / FIX

Có thể kiểm tra:

Giá
Tồn kho
Thông tin sản phẩm
Khuyến mãi
Chính sách

Mục tiêu chính là giảm hallucination.

23. RAG

RAG chứa kiến thức không nên hard-code trực tiếp vào prompt.

Ví dụ:

FAQ
Chính sách đổi trả
Chính sách vận chuyển
Bảo hành
Thông tin thương hiệu
Hướng dẫn mua hàng

RAG là Tool của Agent, không phải một node bắt buộc chạy tuần tự sau Agent.

24. AI MONITORING

Mọi AI execution quan trọng cần log.

ai_logs:

customer_id
session_id
agent_name
intent
input
output
tool_used
latency_ms
input_tokens
output_tokens
total_tokens
estimated_cost
validation_status
error
created_at

Mục tiêu:

Token
Latency
Cost
Error
Tool usage
Validation

Sau này có thể xây dashboard bằng Looker Studio/Power BI.

25. IMPORTANT RULES — QUY TẮC BẮT BUỘC
    Kiến trúc
    Messenger là nguồn input chính.
    Messenger đi vào hệ thống qua Webhook.
    Không dùng Chat Trigger để lấy Session ID.
    Không xây toàn bộ hệ thống thành một workflow n8n khổng lồ.
    Phải giữ kiến trúc module/workflow rõ ràng.
    Không quay lại kiến trúc IF: text → AI / non-text → bỏ.
    Session / Memory
    session_id phải được tạo từ messenger_id + page_id.
    Postgres Chat Memory dùng session_id.
    Customer Profile khác Chat Memory.
    Chat Memory lưu lịch sử hội thoại.
    Customer Profile lưu thông tin nghiệp vụ của khách.
    Message
    Phải hỗ trợ TEXT.
    Phải hỗ trợ IMAGE.
    Phải có pipeline riêng cho Vision.
    Không xử lý Image như Text.
    Không coi OTHER mặc định là bỏ qua nếu có thể mở rộng xử lý.
    Customer
    Phải tự động lưu thông tin khách khi khách cung cấp.
    Thông tin cần hỗ trợ gồm:
    Tên.
    Số điện thoại.
    Địa chỉ.
    Size giày.
    Chiều dài bàn chân.
    Budget.
    Thương hiệu yêu thích.
    Màu yêu thích.
    Kiểu dáng yêu thích.
    Không được tự bịa thông tin khách hàng.
    Khi khách thay đổi thông tin, phải cập nhật profile hiện tại.
    Hành vi khách hàng cần có khả năng ghi vào customer_events.
    Inventory
    AI không được tự bịa tồn kho.
    Tồn kho phải lấy từ database/tool.
    Product, Variant và Inventory phải được phân biệt.
    Recommendation phải kiểm tra stock trước khi khẳng định sản phẩm còn hàng.
    Agents
    Supervisor điều phối Specialist Agents.
    Sales Agent phụ trách bán hàng/tư vấn sản phẩm.
    CRM Agent phụ trách hồ sơ khách hàng.
    Order Agent phụ trách đơn hàng/thanh toán.
    Có thể dùng cùng Gemini API/model cho nhiều Agent.
    Agent khác nhau bằng prompt, nhiệm vụ và tools.
    Không cần dùng nhiều model AI nếu không có lý do thực tế.
    Tools
    Tool phải được gắn trực tiếp vào Agent dưới dạng tool/sub-node phù hợp với n8n LangChain.
    Agent tự quyết định khi nào cần gọi Tool.
    Không thiết kế Tools thành một chuỗi node bắt buộc chạy sau Agent.
    Sales Agent phải có Product/Variant/Inventory/RAG Tools.
    CRM Agent phải có Customer/Profile/Event Tools.
    Order Agent phải có Order/Inventory/Payment/VietQR Tools.
    Handoff
    Handoff Status Check phải nằm trước AI.
    Nếu is_human_active = TRUE, phải STOP AI.
    Không được để AI trả lời đè nhân viên.
    Handoff Detector dùng để phát hiện yêu cầu chuyển người trong quá trình AI xử lý.
    Có thể dựa vào Keyword + Intent + Sentiment + Confidence.
    Human Handoff phải được lưu database.
    Guardrail
    Input Guardrail nằm trong nhánh xử lý Text.
    Không áp dụng text guardrail trực tiếp cho Image.
    Phải có Output Validator.
    Không cho AI tự ý bỏ qua System Prompt/Business Rules.
    Không cho AI tự thay đổi giá/tồn kho/chính sách.
    Quick Reply
    FAQ cố định nên được xử lý trước Gemini.
    Không gọi AI khi câu hỏi có thể trả lời bằng Fast Track.
    Mục tiêu là giảm token, latency và chi phí.
    Vision
    Gemini Vision chỉ phân tích ảnh.
    Vision tạo Structured Attributes.
    Product Search phải đối chiếu database.
    Không để Vision tự khẳng định sản phẩm chính xác nếu database chưa xác nhận.
    RAG
    RAG là Tool của Agent.
    RAG dùng cho FAQ/chính sách/kiến thức sản phẩm.
    Không thiết kế RAG thành một bước bắt buộc chạy sau mọi Agent.
    Logging
    AI execution quan trọng phải được log.
    Phải theo dõi token.
    Phải theo dõi latency.
    Phải theo dõi estimated cost.
    Phải theo dõi lỗi.
    Phải theo dõi tool được sử dụng.
    Phải theo dõi validation status.
26. THỨ TỰ TRIỂN KHAI CHÍNH THỨC

Không nhảy bước.

1.  PostgreSQL
2.  Messenger Webhook
3.  Data Processor
4.  Session ID
5.  Message Router
6.  Customer Resolver
7.  Handoff Status Check
8.  Postgres Chat Memory
9.  Supervisor Agent
10. Quick Reply
11. Debounce
12. Input Guardrail
13. Customer Intelligence
14. Product Database Tools
15. Variant Tools
16. Inventory Tools
17. Sales Agent
18. RAG
19. Gemini Vision
20. CRM Agent
21. Order Agent
22. Payment
23. VietQR
24. Output Validator
25. Handoff Detector
26. Human Handoff
27. AI Logs
28. Monitoring Dashboard
29. End-to-End Testing
30. Demo
31. TIÊU CHÍ “HOÀN THÀNH”

Dự án chỉ được coi là hoàn thiện khi có thể demo được chuỗi:

Khách nhắn tin
↓
Nhận diện Text/Image
↓
Xác định khách hàng
↓
Load Customer Profile
↓
Kiểm tra Human Handoff
↓
AI hiểu ngữ cảnh
↓
Agent phù hợp
↓
Tool phù hợp
↓
Database
↓
Recommendation / Order
↓
Validator
↓
Messenger Response
↓
AI Logs

Và đặc biệt phải demo được:

Demo 1 — Khách mới
Khách cung cấp tên + SĐT + địa chỉ
↓
Database tự lưu
Demo 2 — Cá nhân hóa
Khách có size 42 + budget 2M
↓
AI đề xuất đúng điều kiện
Demo 3 — Tồn kho
Khách hỏi size 42
↓
AI gọi Inventory Tool
↓
Trả số lượng thực tế
Demo 4 — Vision
Khách gửi ảnh giày
↓
Gemini Vision
↓
Product Search
↓
Tìm sản phẩm tương ứng/tương tự
Demo 5 — Human Handoff
Khách yêu cầu gặp nhân viên
↓
Handoff
↓
AI dừng
Demo 6 — Order
Khách xác nhận mua
↓
Order
↓
Stock
↓
Payment
↓
VietQR
Demo 7 — Safety
Prompt Injection
↓
Guardrail
↓
Không để AI phá Business Rules 28. CÁCH SỬ DỤNG FILE NÀY Ở CUỘC TRÒ CHUYỆN SAU

Khi mở cuộc trò chuyện mới, chỉ cần đưa file này và nói:

“Đọc MASTER_PROJECT.md. Đây là Source of Truth của dự án. Kiểm tra CURRENT STATE và tiếp tục triển khai từ bước chưa hoàn thành gần nhất. Không thay đổi kiến trúc hoặc IMPORTANT RULES nếu tôi chưa yêu cầu.”

Nếu có thay đổi trong quá trình làm, phải cập nhật:

CURRENT STATE

và các phần liên quan.

Ví dụ sau khi hoàn thành Vision:

Đã hoàn thành:

- Gemini Vision
- Image Download
- Visual Attribute Extraction
- Product Search

Đang triển khai:

- CRM Agent

Chưa triển khai:

- Order Agent
- VietQR
  ...

File này là bản “hợp đồng kỹ thuật” của dự án. Khi triển khai tiếp, mọi quyết định nên được đối chiếu với CURRENT STATE và IMPORTANT RULES trước.
