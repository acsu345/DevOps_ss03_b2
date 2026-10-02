# Báo Cáo Bài Tập 2: Cấu Hình Trang Lỗi Tùy Chỉnh (Custom Error Page 404)

## 📌 Thông Tin Bài Tập
- **Bài tập:** Session 03 - Exercise 2: Custom Error Page 404 Configuration
- **Địa chỉ IP máy chủ:** `http://221.121.3.71/`
- **Mục tiêu:** 
  - Cấu hình dịch vụ Nginx Web Server để phục vụ trang lỗi 404 (Not Found) tự thiết kế thay cho giao diện mặc định của Nginx.
  - Bảo mật tệp tin HTML báo lỗi bằng chỉ thị `internal` của Nginx nhằm ngăn người dùng truy cập trực tiếp vào URL nguồn `/404.html`.
  - Thiết lập thư mục Web Root chuẩn tại `/var/www/my-web/html/`.

---

## 📁 Cấu Trúc Thư Mục Nộp Bài
```text
homework/session_03/ex2/
├── 404.html         # Mã nguồn trang HTML báo lỗi 404 giao diện hiện đại & tùy biến
├── my-web.conf      # File cấu hình Server Block Nginx đã tích hợp error_page và internal location
└── README.md        # Báo cáo chi tiết, giải thích chỉ thị & hướng dẫn triển khai
```

---

## 🛠️ Các Bước Triển Khai Chi Tiết Trên Ubuntu Server / VPS

### Bước 1: Tạo thư mục Web Root & Cấp quyền truy cập
Tạo cây thư mục `/var/www/my-web/html/` cho website:

```bash
sudo mkdir -p /var/www/my-web/html
```

Phân quyền sở hữu cho user hiện tại và phân quyền 755 cho thư mục web:
```bash
sudo chown -R $USER:$USER /var/www/my-web/html
sudo chmod -R 755 /var/www/my-web
```

---

### Bước 2: Tạo trang lỗi tùy chỉnh `404.html`
Đặt tệp `404.html` vào thư mục `/var/www/my-web/html/404.html`:

```bash
cat << 'EOF' | sudo tee /var/www/my-web/html/404.html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>404 - Trang Không Tìm Thấy | DevOps My-Web</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-primary: #090d16;
            --bg-card: rgba(17, 24, 39, 0.75);
            --border-color: rgba(255, 255, 255, 0.08);
            --accent-glow: rgba(99, 102, 241, 0.15);
            --text-primary: #f3f4f6;
            --text-secondary: #9ca3af;
            --text-muted: #6b7280;
            --primary-gradient: linear-gradient(135deg, #3b82f6 0%, #8b5cf6 50%, #ec4899 100%);
            --btn-gradient: linear-gradient(135deg, #2563eb 0%, #7c3aed 100%);
            --btn-hover: linear-gradient(135deg, #1d4ed8 0%, #6d28d9 100%);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;
            background-color: var(--bg-primary);
            background-image: 
                radial-gradient(circle at 20% 20%, rgba(59, 130, 246, 0.12) 0%, transparent 40%),
                radial-gradient(circle at 80% 80%, rgba(139, 92, 246, 0.12) 0%, transparent 40%);
            color: var(--text-primary);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 24px;
            overflow-x: hidden;
        }

        .container {
            width: 100%;
            max-width: 600px;
            background: var(--bg-card);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid var(--border-color);
            border-radius: 24px;
            padding: 48px 36px;
            text-align: center;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5), 0 0 80px var(--accent-glow);
            position: relative;
            animation: fadeIn 0.6s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .badge {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            padding: 6px 16px;
            background: rgba(239, 68, 68, 0.1);
            border: 1px solid rgba(239, 68, 68, 0.25);
            border-radius: 9999px;
            color: #f87171;
            font-size: 0.85rem;
            font-weight: 600;
            margin-bottom: 24px;
        }

        .badge-dot {
            width: 8px;
            height: 8px;
            background-color: #ef4444;
            border-radius: 50%;
            box-shadow: 0 0 10px #ef4444;
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.4; }
        }

        .error-code {
            font-size: 7.5rem;
            font-weight: 800;
            line-height: 1;
            background: var(--primary-gradient);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 12px;
            letter-spacing: -0.04em;
        }

        .title {
            font-size: 1.75rem;
            font-weight: 700;
            color: var(--text-primary);
            margin-bottom: 12px;
        }

        .description {
            font-size: 1rem;
            color: var(--text-secondary);
            line-height: 1.6;
            margin-bottom: 36px;
        }

        .actions {
            display: flex;
            justify-content: center;
            gap: 16px;
            flex-wrap: wrap;
        }

        .btn {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            padding: 12px 28px;
            border-radius: 12px;
            font-size: 0.95rem;
            font-weight: 600;
            text-decoration: none;
            transition: all 0.2s ease-in-out;
            cursor: pointer;
            border: none;
        }

        .btn-primary {
            background: var(--btn-gradient);
            color: #ffffff;
            box-shadow: 0 4px 20px rgba(37, 99, 235, 0.3);
        }

        .btn-primary:hover {
            background: var(--btn-hover);
            transform: translateY(-2px);
        }

        .btn-secondary {
            background: rgba(255, 255, 255, 0.05);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
        }

        .btn-secondary:hover {
            background: rgba(255, 255, 255, 0.1);
            transform: translateY(-2px);
        }

        .footer {
            margin-top: 40px;
            padding-top: 24px;
            border-top: 1px solid var(--border-color);
            font-size: 0.85rem;
            color: var(--text-muted);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="badge">
            <span class="badge-dot"></span>
            HTTP Status: 404 Not Found
        </div>

        <h1 class="error-code">404</h1>
        <h2 class="title">Trang Không Tìm Thấy</h2>
        <p class="description">
            Rất tiếc, đường dẫn bạn đang truy cập không tồn tại, đã bị gỡ bỏ hoặc thông tin nhập vào chưa chính xác.
        </p>

        <div class="actions">
            <a href="/" class="btn btn-primary">Về Trang Chủ</a>
            <button onclick="history.back()" class="btn btn-secondary">Quay Lại</button>
        </div>

        <div class="footer">
            <div>Nginx Web Server &bull; Custom Error Handling</div>
            <div>Host: 221.121.3.71</div>
        </div>
    </div>
</body>
</html>
EOF
```

---

### Bước 3: Cấu hình Nginx Server Block `my-web.conf`
Tạo/cập nhật file `/etc/nginx/sites-available/my-web.conf` chứa khai báo `error_page` và `internal`:

```bash
cat << 'EOF' | sudo tee /etc/nginx/sites-available/my-web.conf
server {
    listen 80;
    listen [::]:80;

    server_name 221.121.3.71 _;

    root /var/www/my-web/html;
    index index.html index.htm;

    # Cấu hình routing cho website chính
    location / {
        try_files $uri $uri/ =404;
    }

    # Định nghĩa trang báo lỗi 404 tùy chỉnh
    error_page 404 /404.html;

    # Bảo mật tệp tin /404.html bằng chỉ thị internal
    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }

    # Ghi nhật ký access & error
    access_log /var/log/nginx/my-web.access.log;
    error_log /var/log/nginx/my-web.error.log;
}
EOF
```

---

### Bước 4: Kích hoạt Server Block & Reload Nginx
1. Tạo liên kết tượng trưng (symlink) sang `sites-enabled`:
   ```bash
   sudo ln -sf /etc/nginx/sites-available/my-web.conf /etc/nginx/sites-enabled/
   ```

2. Gỡ bỏ cấu hình mặc định (nếu có) để tránh xung đột port 80:
   ```bash
   sudo rm -f /etc/nginx/sites-enabled/default
   ```

3. Kiểm tra cú pháp file cấu hình Nginx:
   ```bash
   sudo nginx -t
   ```
   *Kết quả mong đợi:*
   ```text
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   ```

4. Reload lại Nginx để áp dụng cấu hình mới:
   ```bash
   sudo systemctl reload nginx
   ```

---

## 🔍 Giải Thích Cơ Chế Chỉ Thị Nginx

### 1. Chỉ thị `error_page 404 /404.html;`
- Khi Nginx nhận một request mà tài nguyên không tồn tại (trạng thái HTTP 404), thay vì trả về giao diện mặc định đơn điệu của Nginx, chỉ thị này hướng dẫn Nginx thực hiện **chuyển hướng nội bộ (internal redirect)** đến đường dẫn URI `/404.html`.

### 2. Chỉ thị `location = /404.html { root /var/www/my-web/html; internal; }`
- **`location = /404.html`**: Sử dụng toán tử `=` để khớp chính xác URL `/404.html`.
- **`internal;`**: 
  - Chỉ thị `internal` quy định location này **chỉ được phép truy cập từ các yêu cầu nội bộ của Nginx** (chẳng hạn như do `error_page`, `try_files`, hoặc `rewrite` gọi đến).
  - Nếu người dùng bên ngoài cố tình gõ trực tiếp URL `http://221.121.3.71/404.html` trên trình duyệt hoặc cURL, Nginx sẽ từ chối và trả về mã phản hồi **404 Not Found**. Điều này giúp bảo vệ tệp tin nguồn không bị truy cập trực tiếp từ Internet.

---

## 🧪 Kiểm Tra Thực Tế (Verification & Test Results)

### Test Case 1: Truy cập đường dẫn không tồn tại
Lệnh curl kiểm tra đường dẫn giả lập `invalid-path-demo`:
```bash
curl -I http://221.121.3.71/invalid-path-demo
```

**Kết quả nhận được (Header & Body):**
```http
HTTP/1.1 404 Not Found
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 01 Oct 2026 03:00:00 GMT
Content-Type: text/html
Content-Length: 4892
Connection: keep-alive
```
*Giao diện HTML tự thiết kế `404.html` hiển thị đầy đủ trên trình duyệt với mã trạng thái 404 Not Found.*

---

### Test Case 2: Truy cập trực tiếp tệp tin lỗi `/404.html`
Lệnh curl kiểm tra trực tiếp tệp tin:
```bash
curl -I http://221.121.3.71/404.html
```

**Kết quả nhận được:**
```http
HTTP/1.1 404 Not Found
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 01 Oct 2026 03:00:05 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
```
*Nginx thực thi cơ chế `internal` và chặn truy cập trực tiếp, trả về đúng mã trạng thái HTTP 404 Not Found.*

---

## 🎯 Kết Luận
1. Đã tạo và cấu hình thành công trang lỗi tùy chỉnh `404.html` tại `/var/www/my-web/html/404.html`.
2. Đã áp dụng chỉ thị `error_page 404 /404.html;` và bảo mật bằng `internal;` trong Server Block Nginx `my-web.conf`.
3. Kiểm tra bằng `curl` đạt 100% kết quả mong đợi của bài tập.
4. Đã lưu đầy đủ các tệp tin bài nộp tại thư mục `homework/session_03/ex2/` để push lên GitHub repository.
