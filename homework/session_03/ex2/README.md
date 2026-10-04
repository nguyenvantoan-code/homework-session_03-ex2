# Bài 2: Cấu hình trang lỗi tùy chỉnh (Custom Error Page 404)

## 📌 Mục tiêu
- Cấu hình Nginx Web Server để hiển thị trang báo lỗi **404 Not Found** tùy biến (giao diện thân thiện, hiện đại) thay cho trang báo lỗi mặc định đơn điệu của hệ thống.
- Sử dụng chỉ thị `internal` của Nginx để bảo vệ tệp `404.html`, ngăn chặn người dùng ngoài Internet truy cập trực tiếp vào tệp nguồn bằng URL `http://<IP>/404.html`.

---

## 📁 Cấu trúc nộp bài trên GitHub
Đường dẫn thư mục nộp bài: `homework/session_03/ex2/`

```text
homework/session_03/ex2/
├── 404.html           # Mã nguồn HTML trang lỗi 404 tùy biến với giao diện Modern Dark Glassmorphism
├── my-web.conf        # File cấu hình Server Block Nginx đã tích hợp error_page và location internal
└── README.md          # Tài liệu hướng dẫn các bước thực hiện và kiểm tra chi tiết
```

---

## 🛠️ Hướng dẫn thực hiện từng bước trên Server / Droplet

### Bước 1: Tạo tệp `404.html` trong thư mục Web Root
Tạo hoặc chép tệp `404.html` vào thư mục `/var/www/my-web/html/`:

```bash
sudo nano /var/www/my-web/html/404.html
```

*Dán toàn bộ nội dung HTML tùy biến từ tệp `404.html` vào và lưu lại.*

Phân quyền truy cập phù hợp cho tệp `404.html`:
```bash
sudo chown www-data:www-data /var/www/my-web/html/404.html
sudo chmod 644 /var/www/my-web/html/404.html
```

---

### Bước 2: Cấu hình Server Block trong Nginx
Mở file cấu hình Server Block của trang web `my-web` (thường nằm tại `/etc/nginx/sites-available/my-web` hoặc `/etc/nginx/conf.d/my-web.conf`):

```bash
sudo nano /etc/nginx/sites-available/my-web
```

Thêm/chỉnh sửa chỉ thị `error_page` và khối `location = /404.html` như sau:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/my-web/html;
    index index.html index.htm;

    # 1. Khai báo chỉ thị error_page cho mã 404
    error_page 404 /404.html;

    # 2. Định nghĩa location block bảo vệ file 404.html
    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### Bước 3: Kiểm tra cú pháp cấu hình Nginx
Trước khi áp dụng thay đổi, luôn kiểm tra xem file cấu hình có bị lỗi cú pháp không:

```bash
sudo nginx -t
```

**Kết quả mong đợi:**
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

### Bước 4: Reload lại dịch vụ Nginx
Sau khi kiểm tra thành công, reload dịch vụ Nginx để áp dụng cấu hình mới:

```bash
sudo systemctl reload nginx
```

---

## 🧪 Kiểm tra kết quả (Verification)

### 1. Truy cập một đường dẫn không tồn tại
Sử dụng `curl` để gửi yêu cầu đến đường dẫn không tồn tại trên hệ thống:

```bash
curl -I http://<IP_ADDRESS_DROPLET>/invalid-path-demo
```

**Kết quả mong đợi:**
- HTTP Status Code: `HTTP/1.1 404 Not Found`
- Khi truy cập bằng trình duyệt web, giao diện sẽ hiển thị trang `404.html` tùy biến với thông điệp *"Trang không tồn tại"*.

```bash
curl -s http://<IP_ADDRESS_DROPLET>/invalid-path-demo | grep -i "Trang không tồn tại"
```

---

### 2. Truy cập trực tiếp đường dẫn `/404.html`
Sử dụng `curl` để kiểm tra truy cập trực tiếp tệp `/404.html`:

```bash
curl -I http://<IP_ADDRESS_DROPLET>/404.html
```

**Kết quả mong đợi:**
- HTTP Status Code: `HTTP/1.1 404 Not Found` (thay vì 200 OK).
- **Giải thích:** Nhờ chỉ thị `internal` trong block `location = /404.html`, Nginx cấm người dùng ngoài internet gọi trực tiếp URL này. Nếu cố tình truy cập, Nginx sẽ phản hồi ngay lỗi `404 Not Found`.

---

## 💡 Giải thích cơ chế hoạt động

| Chỉ thị / Block | Ý nghĩa & Cơ chế hoạt động |
| :--- | :--- |
| `error_page 404 /404.html;` | Hướng dẫn Nginx khi gặp lỗi 404 (do không tìm thấy tài nguyên), hãy thực hiện chuyển hướng nội bộ (internal redirect) gọi URI `/404.html`. |
| `location = /404.html` | Sử dụng khớp chính xác (`=`) URI `/404.html` để tối ưu hiệu năng và định cấu hình riêng cho tệp lỗi này. |
| `internal;` | Chỉ cho phép Nginx truy cập location này thông qua các yêu cầu nội bộ (internal request/redirect) do Nginx tự tạo ra. Nếu người dùng gọi trực tiếp từ bên ngoài qua browser/curl, Nginx sẽ chặn lại và trả về 404. |
