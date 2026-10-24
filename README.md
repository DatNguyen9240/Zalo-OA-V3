# Zalo Automation with n8n (v3)

Dự án tích hợp và tự động hóa tài khoản Zalo cá nhân trên nền tảng **n8n** thông qua Docker và Community Node `n8n-nodes-zalo-user-v3`.

Hệ thống hỗ trợ tự động lắng nghe tin nhắn, phản hồi tự động, gửi sticker, lấy thông tin người gửi và chuyển đổi văn bản thành giọng nói AI (Text-To-Speech - TTS) để gửi file âm thanh `.mp3` qua Zalo.

---

## 📁 Cấu trúc dự án

```text
├── docker-compose.yml   # File cấu hình triển khai n8n qua Docker
├── zalo.json            # Workflow chatbot tự động phản hồi tin nhắn Zalo
├── tts.json             # Workflow test chuyển văn bản thành giọng đọc AI (TTS)
├── .gitignore           # Danh sách file/thư mục bỏ qua khi commit Git
└── README.md            # Tài liệu hướng dẫn sử dụng
```

---

## 🚀 Hướng dẫn cài đặt & Triển khai

### 1. Khởi chạy n8n bằng Docker

Đảm bảo máy tính của bạn đã cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop/).

Mở Terminal tại thư mục dự án và chạy lệnh:

```bash
docker compose up -d
```

Sau khi khởi chạy thành công, truy cập giao diện n8n tại: **[http://localhost:5678](http://localhost:5678)** và hoàn thành bước tạo tài khoản Admin ban đầu.

---

### 2. Cài đặt Community Node Zalo

1. Tại giao diện n8n, chọn biểu tượng bánh răng **Settings** ở góc dưới bên trái.
2. Chọn mục **Community nodes** $\rightarrow$ bấm **Install a community node**.
3. Tại ô **npm Package Name**, nhập:
   ```text
   n8n-nodes-zalo-user-v3
   ```
4. Đánh dấu tích đồng ý với cảnh báo rủi ro $\rightarrow$ Bấm **Install**.

---

### 3. Cấu hình n8n API Key

Node đăng nhập Zalo qua mã QR cần quyền gọi API n8n để tự động lưu thông tin xác thực (*Credential*):

1. Vào **Settings** $\rightarrow$ chọn mục **n8n API**.
2. Bấm **Create an API key** $\rightarrow$ Đặt tên bất kỳ $\rightarrow$ Copy mã **API Key**.
3. Tạo Credential n8n API:
   - Vào mục **Credentials** (menu bên trái) $\rightarrow$ chọn **Add Credential** $\rightarrow$ tìm **n8n API**.
   - Dán mã **API Key** vừa tạo vào.
   - Mục **Base URL** (nếu có): điền `http://127.0.0.1:5678/api/v1` $\rightarrow$ Bấm **Save**.

---

### 4. Import Workflow & Đăng nhập Zalo

1. Tạo một Workflow mới $\rightarrow$ bấm vào biểu tượng menu `...` ở góc trên bên phải $\rightarrow$ chọn **Import from File**.
2. Chọn file [zalo.json](file:///d:/Dat/MIAI_Zalo_N8N/zalo.json) từ thư mục dự án.
3. **Đăng nhập Zalo bằng mã QR:**
   - Click đúp vào node **Zalo Login Via QR Code**.
   - Ở mục **Credential for n8n API**, chọn tài khoản `n8n API` đã tạo ở Bước 3.
   - Bấm nút **Test step** $\rightarrow$ chuyển sang tab **Binary** ở khung Output để xem ảnh mã QR.
   - Mở ứng dụng **Zalo trên điện thoại** $\rightarrow$ Quét mã QR trên màn hình $\rightarrow$ Bấm **Xác nhận đăng nhập**.
   - Hệ thống sẽ tự động lưu thông tin phiên đăng nhập vào n8n.
4. **Gán Credential:**
   - Mở node **Zalo User Trigger** và các node **Zalo User Interact**, chọn tài khoản Zalo vừa đăng nhập thành công.
5. **Kích hoạt tự động:**
   - Gạt nút công tắc ở góc trên bên phải từ **Inactive** sang **Active** để bot chạy ngầm liên tục 24/7.

---

## 🛠️ Chi tiết các Workflow

### 1. `zalo.json` (Chatbot tự động phản hồi)
* **Trigger:** Lắng nghe tin nhắn gửi đến tài khoản Zalo cá nhân.
* **Quy trình xử lý:**
  1. Gửi tin nhắn phản hồi: *"Cảm ơn bạn rất nhiều. Có phải nội dung bạn gửi là [Nội dung tin]..."*
  2. Gửi kèm một sticker cảm xúc.
  3. Lấy thông tin tài khoản người gửi.
  4. Tạo file âm thanh đọc bằng AI (TTS) với nội dung tùy chỉnh.
  5. Gửi file âm thanh `.mp3` qua Zalo cho người nhắn.

### 2. `tts.json` (Test tính năng Text-To-Speech)
* **Trigger:** Kích hoạt bằng tay (*Manual Trigger*).
* **Quy trình xử lý:**
  1. Tạo đường dẫn file âm thanh tạm thời trên ổ đĩa.
  2. Chuyển đoạn văn bản chỉ định thành giọng nói AI.
  3. Ghi ra file `.mp3` và gửi tới một User/Thread ID chỉ định.
  4. Tự động xoá file tạm để dọn dẹp bộ nhớ.

---

## ⚠️ Lưu ý quan trọng

> [!WARNING]
> * **Không bấm chạy lại node QR sau khi đã đăng nhập:** Node **Zalo Login Via QR Code** chỉ cần chạy **duy nhất 1 lần** để lấy phiên. Nếu bạn bấm chạy lại node này, Zalo sẽ lập tức hủy phiên đăng nhập cũ trên máy chủ và bạn sẽ phải quét mã lại từ đầu.
> * **Không test sticker bằng Cloud của tôi:** Máy chủ Zalo chặn tính năng gửi sticker tới chính mình (`toid = uid của bạn`). Hãy dùng một tài khoản Zalo khác nhắn tin tới để kiểm tra đầy đủ tính năng.
> * **Tránh spam:** Sử dụng tài khoản Zalo cá nhân để tự động hóa cần duy trì tần suất hợp lý, tránh gửi tin nhắn hàng loạt để không bị hạn chế tính năng hoặc khóa tài khoản.

---

## 🐳 Các lệnh quản lý Docker

| Thao tác | Lệnh thực thi |
| :--- | :--- |
| Khởi động container | `docker compose up -d` |
| Tạm dừng container | `docker compose stop` |
| Bật lại container | `docker compose start` |
| Khởi động lại container | `docker compose restart` |
| Xem logs container | `docker compose logs -f` |
| Tắt và xóa container | `docker compose down` |

---

## 👤 Tác giả
* **Nguyễn Thành Đạt**
* **Repository:** [https://github.com/DatNguyen9240/Zalo-OA-V3](https://github.com/DatNguyen9240/Zalo-OA-V3)
