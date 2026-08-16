# Workflow-n8n-marketing

Bộ workflow n8n phục vụ marketing của TMICORP — nhà sản xuất và xuất khẩu
nông sản chế biến Việt Nam.

Mọi credential, API key, ID tài liệu Google và địa chỉ email đã được thay bằng
placeholder trước khi đưa lên đây. Xem [Cấu hình trước khi chạy](#cấu-hình-trước-khi-chạy).

## Danh sách workflow

| File | Tên | Nodes | Chức năng |
|---|---|---:|---|
| [`send-email.json`](send-email.json) | send email | 14 | Đọc Google Sheet, dựng email HTML kèm ảnh từ Drive, gửi qua Gmail, cập nhật trạng thái dòng |
| [`topic.json`](topic.json) | TOPIC | 16 | Sinh từ khoá bằng Google Autosuggest, gom cụm theo search intent bằng LLM, ghi lại vào Sheet |
| [`seo.json`](seo.json) | SEO | 58 | Pipeline sản xuất nội dung SEO: nghiên cứu (Tavily, USDA FoodData), nhiều AI Agent, xuất ra Sheet/Drive |
| [`email.json`](email.json) | email | 80 | Pipeline email mở rộng: sinh ảnh qua kie.ai, phân nhánh theo loại nội dung, thông báo Telegram |
| [`social-media.json`](social-media.json) | social media | 89 | Đăng bài đa kênh: Facebook Graph API, LinkedIn, Instagram; sinh ảnh qua kie.ai; điều phối qua Sheet |

### Dịch vụ ngoài được dùng

Google Sheets · Google Drive · Gmail · Facebook Graph API · LinkedIn · Instagram
Graph · Telegram · OpenRouter · Tavily · kie.ai · USDA FoodData Central ·
Google Autosuggest

## Cách import

1. Mở n8n → **Workflows** → **Import from File**
2. Chọn file `.json` cần import
3. Gán lại credential cho từng node (xem bên dưới)
4. Thay các placeholder `YOUR_*` bằng giá trị thật

## Cấu hình trước khi chạy

### Credential

Phần credential đã bị loại bỏ khỏi file JSON. Sau khi import, tạo và gán lại
trong n8n cho các node: Google Sheets, Google Drive, Gmail, Facebook Graph API,
Telegram, OpenRouter, Tavily.

### Placeholder cần thay

| Placeholder | Ý nghĩa | Xuất hiện ở |
|---|---|---|
| `YOUR_GOOGLE_SHEET_ID` | ID của Google Sheet điều phối (bảng "Marketing plan") | tất cả |
| `YOUR_GOOGLE_FILE_ID` | ID file/thư mục Drive (ảnh sản phẩm, catalogue, thư mục upload) | `send-email`, `email`, `social-media` |
| `YOUR_KIE_AI_API_KEY` | API key kie.ai, đặt ở header `Authorization: Bearer` | `email`, `social-media` |
| `YOUR_LINKEDIN_ACCESS_TOKEN` | OAuth access token LinkedIn | `social-media` |
| `YOUR_USDA_API_KEY` | API key USDA FoodData Central | `seo` |
| `YOUR_ALIBABA_SHOP` | Subdomain gian hàng Alibaba trustpass | `social-media` |
| `your-email@example.com` | Email người gửi / người nhận | `send-email`, `email`, `social-media` |

### Webhook

Node webhook đã được đặt lại `webhookId` và `path` về giá trị rỗng
(`00000000-0000-0000-0000-000000000000`). n8n sẽ sinh ID mới khi import.

Các webhook này mặc định **không có xác thực** — bất kỳ ai biết URL đều kích hoạt
được workflow. Nên bật Header Auth cho node webhook trước khi dùng ở môi trường thật.

### Cấu trúc Google Sheet

Workflow đọc/ghi các sheet sau trong cùng một spreadsheet:

- **Email** — `day`, `subject`, `subject email`, `paragraph1..3`, `product images`, `prompt`, `images in email`, `attachments`, `status`
- **KEY WORD SEO** — `KW_ID`, `Pillar Topic`, `Seed`, `Geolocation`, `Tavily`, `Search Intent`, `Buyer`, `Title`, `primary`, `secondary`, `Meta`, `link`, `status`
- **Topic** — `Topic`, `List_keyword`, `Pillar Topic`, `Seed`

Cột `status` đóng vai trò hàng đợi: workflow lọc theo `Send` / `Processing`
rồi cập nhật thành `Sent` / `Finished` sau khi xử lý xong.
