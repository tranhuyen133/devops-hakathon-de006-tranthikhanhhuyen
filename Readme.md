# Báo Cáo & Hướng Dẫn Thực Hiện - DevOps Hackathon (Đề Số 06)

**Học viên:** Trần Thị Khánh Huyền  
**Tài khoản Linux:** `tranthikhanhuyen`  
**Mã đề thi:** Đề 06 (DE06)  
**Thư mục mã nguồn triển khai:** `/var/www/devops-hakthon-de006-tranthikhanhuyen`  

---

## 📁 Cấu Trúc Thư Mục Dự Án

```text
de06_hakathon/
├── src/
│   └── index.html                 # Trang web hiển thị thông tin bài thi & sinh viên
├── nginx/
│   └── tranthikhanhuyen.conf      # File cấu hình Nginx Server Block mẫu
├── ngnix/
│   └── tranthikhanhuyen.conf      # Thư mục dự phòng theo đúng đề bài yêu cầu
├── screenshot/                    # Thư mục lưu trữ hình ảnh minh chứng kết quả
│   └── README.md
├── .gitignore                     # Bỏ qua tệp hệ điều hành, IDE, logs
└── Readme.md                      # Tài liệu hướng dẫn & giải trình bài làm
```

---

## 🛠 PHẦN 1: Chuẩn Bị Môi Trường Linux

Mục tiêu: Khởi tạo tài khoản hệ thống mới chuẩn bảo mật với thư mục home riêng, bash shell mặc định và quyền sudo.

### Bước 1: Tạo tài khoản Linux `tranthikhanhuyen`
Chạy lệnh tạo user có thư mục home (`-m`), shell mặc định `/bin/bash` (`-s /bin/bash`):
```bash
sudo useradd -m -s /bin/bash tranthikhanhuyen
```

Đặt mật khẩu cho tài khoản:
```bash
sudo passwd tranthikhanhuyen
```
*(Nhập mật khẩu an toàn và xác nhận lại mật khẩu)*

### Bước 2: Thiết lập Group cho tài khoản
Kiểm tra primary group mặc định (Linux tự gán primary group trùng tên user `tranthikhanhuyen`). Thêm user vào secondary group `sudo`:
```bash
sudo usermod -aG sudo tranthikhanhuyen
```

Kiểm tra lại thông tin định danh của user:
```bash
id tranthikhanhuyen
```
**Kết quả mong đợi:**
```text
uid=1001(tranthikhanhuyen) gid=1001(tranthikhanhuyen) groups=1001(tranthikhanhuyen),27(sudo)
```
*(Gid chính là `tranthikhanhuyen`, secondary group bao gồm `sudo`)*.

### Bước 3: Đăng nhập vào tài khoản mới
Chuyển phiên làm việc sang tài khoản `tranthikhanhuyen`:
```bash
su - tranthikhanhuyen
```
Kiểm tra đường dẫn hiện tại và quyền hạn:
```bash
pwd        # Kết quả: /home/tranthikhanhuyen
whoami     # Kết quả: tranthikhanhuyen
sudo whoami # Nhập mật khẩu của tranthikhanhuyen -> Kết quả: root
```
> **Lưu ý:** Tất cả các thao tác từ đây trở về sau đều được thực hiện từ tài khoản `tranthikhanhuyen` (sử dụng `sudo` khi cần quyền quản trị).

---

## 📦 PHẦN 2: Cài Đặt Phần Mềm & Xây Dựng Mã Nguồn

### 1. Cài đặt các gói phần mềm cần thiết
Cập nhật danh sách gói và cài đặt `nginx`, `git`, `ufw`, `curl`:
```bash
sudo apt update
sudo apt install -y nginx git ufw curl
```

### 2. Đảm bảo Nginx đang chạy và tự khởi động cùng hệ điều hành
Kích hoạt service tự chạy khi boot:
```bash
sudo systemctl enable nginx
```
Khởi động service Nginx (nếu chưa chạy):
```bash
sudo systemctl start nginx
```
Kiểm tra trạng thái Nginx:
```bash
systemctl status nginx
```
Kiểm tra nhanh:
```bash
systemctl is-active nginx    # Kết quả: active
systemctl is-enabled nginx   # Kết quả: enabled
```

---

### 2.1 Xây Dựng Website (`src/index.html`)
Mã nguồn tệp `src/index.html` được thiết kế chuẩn HTML5, sử dụng CSS hiện đại, bố cục rõ ràng bao gồm:
- **Cấu trúc chuẩn**: Doctype html, thẻ meta UTF-8, viewport responsive.
- **Thẻ Semantic**: `<header>`, `<section>`, `<main>`, `<footer>`, `<ul>`, `<li>`.
- **Tiêu đề chính**: "BÀI THI DEVOPS HACKATHON - Đề số 06".
- **Khối thông tin sinh viên**: Họ tên (Trần Thị Khánh Huyền), User (`tranthikhanhuyen`), Email, GitHub.
- **Khối thông tin bài thi**: Tên kỳ thi, Mã đề 06, Thư mục web, Cổng dịch vụ 80.
- **Đoạn giới thiệu ngắn**: Mục đích bài thi, quy chuẩn phân quyền hệ thống và kiến trúc Nginx + UFW.

---

### 2.2 Quản Lý Mã Nguồn Với Git & GitHub

#### 1. Khởi tạo Git repository và cấu hình `.gitignore`
Tại thư mục bài thi trên máy:
```bash
git init -b main
```
Tạo tệp `.gitignore` bỏ qua `.DS_Store`, `.idea/`, `*.log`, v.v.

#### 2. Thực hiện tối thiểu 4 commits có ý nghĩa
```bash
# Commit 1: Khởi tạo cấu trúc và .gitignore
git add .gitignore screenshot/
git commit -m "chore: khoi tao cau truc thu muc va .gitignore"

# Commit 2: Thêm mã nguồn website index.html
git add src/
git commit -m "feat: xay dung giao dien website index.html day du thong tin"

# Commit 3: Thêm cấu hình Nginx Server Block
git add nginx/ ngnix/
git commit -m "feat: cau hinh nginx server block cho tranthikhanhuyen"

# Commit 4: Cập nhật tài liệu Readme.md và hướng dẫn triển khai
git add Readme.md README.md
git commit -m "docs: hoan thien Readme.md huong dan chi tiet cac buoc trien khai va ufw"
```

Kiểm tra lịch sử commit:
```bash
git log --oneline
```

#### 3. Tạo Repo Public trên GitHub và Push Toàn Bộ Mã Nguồn
- Tạo repository mới trên GitHub ở chế độ **Public**, ví dụ: `devops-hackathon-de06-tranthikhanhuyen`.
- Liên kết remote và đẩy code:
```bash
git remote add origin https://github.com/tranhuyen133/devops-hackathon-de06-tranthikhanhuyen.git
git push -u origin main
```

---

## 🚀 PHẦN 3: Triển Khai Website & Tường Lửa UFW

### 3.1 Deploy mã nguồn từ Git lên máy chủ

#### Bước 1: Clone Git Repo về thư mục chỉ định
Trên máy chủ Linux (chạy với user `tranthikhanhuyen`):
```bash
sudo git clone https://github.com/tranhuyen133/devops-hackathon-de06-tranthikhanhuyen.git /var/www/devops-hakthon-de006-tranthikhanhuyen
```

#### Bước 2: Đổi quyền sở hữu để `git pull` không cần `sudo`
Gán quyền sở hữu toàn bộ thư mục cho user và group `tranthikhanhuyen`:
```bash
sudo chown -R tranthikhanhuyen:tranthikhanhuyen /var/www/devops-hakthon-de006-tranthikhanhuyen
```
*Từ đây, khi có cập nhật code mới, chỉ cần chạy `cd /var/www/devops-hakthon-de006-tranthikhanhuyen && git pull` mà không cần quyền root.*

#### Bước 3: Phân quyền bảo mật (Thư mục 755, File 644 - Tuyệt đối không dùng 777)
```bash
# Phân quyền 755 (rwxr-xr-x) cho tất cả thư mục
find /var/www/devops-hakthon-de006-tranthikhanhuyen -type d -exec chmod 755 {} \;

# Phân quyền 644 (rw-r--r--) cho tất cả tệp tin
find /var/www/devops-hakthon-de006-tranthikhanhuyen -type f -exec chmod 644 {} \;
```
Kiểm tra lại phân quyền:
```bash
ls -ld /var/www/devops-hakthon-de006-tranthikhanhuyen
ls -l /var/www/devops-hakthon-de006-tranthikhanhuyen/src
```

---

### 3.2 Cấu hình Server Block cho Nginx

#### Bước 1: Kích hoạt cấu hình từ repo vào Nginx
Sao chép tệp cấu hình từ thư mục repo vào `/etc/nginx/sites-available/`:
```bash
sudo cp /var/www/devops-hakthon-de006-tranthikhanhuyen/nginx/tranthikhanhuyen.conf /etc/nginx/sites-available/tranthikhanhuyen.conf
```
Tạo symbolic link sang `/etc/nginx/sites-enabled/`:
```bash
sudo ln -s /etc/nginx/sites-available/tranthikhanhuyen.conf /etc/nginx/sites-enabled/
```

#### Bước 2: Gỡ bỏ site mặc định (tránh chiếm cổng 80)
```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

#### Bước 3: Kiểm tra cú pháp và nạp lại cấu hình
```bash
sudo nginx -t
```
**Kết quả mong đợi:**
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Sau khi kiểm tra thành công, reload lại Nginx:
```bash
sudo systemctl reload nginx
```

#### Bước 4: Kiểm tra phản hồi HTTP
Kiểm tra trực tiếp tại máy chủ bằng lệnh curl:
```bash
curl -I http://localhost
```
**Kết quả mong đợi:**
```text
HTTP/1.1 200 OK
Server: nginx/...
Content-Type: text/html
...
```

---

### 3.3 Hướng Dẫn Cấu Hình Tường Lửa UFW Chi Tiết

> ⚠️ **CẢNH BÁO AN TOÀN QUAN TRỌNG:**  
> Nếu bạn đang thao tác trên máy chủ từ xa (VPS / Cloud VM qua SSH), **BẮT BUỘC** phải cho phép cổng SSH (Port 22) **TRƯỚC KHI** kích hoạt UFW (`ufw enable`). Nếu kích hoạt UFW trước khi mở cổng 22, bạn sẽ bị mất kết nối SSH và bị khóa hoàn toàn khỏi máy chủ!

#### Các bước thực hiện:

#### 1. Đặt chính sách mặc định (Default Policies)
Chính sách an toàn nhất là: Chặn toàn bộ kết nối đi vào (incoming) và cho phép toàn bộ kết nối đi ra (outgoing):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### 2. Cho phép kết nối SSH (Port 22)
```bash
sudo ufw allow 22/tcp
# Hoặc: sudo ufw allow ssh
```
*Giải thích: Đảm bảo quản trị viên luôn có thể kết nối vào máy chủ từ xa qua giao thức SSH.*

#### 3. Cho phép dịch vụ Web HTTP (Port 80) và HTTPS (Port 443)
```bash
# Mở cổng HTTP để người dùng truy cập website
sudo ufw allow 80/tcp

# (Khuyến nghị) Mở cổng HTTPS nếu sau này tích hợp SSL
sudo ufw allow 443/tcp

# Hoặc dùng profile có sẵn của Nginx:
# sudo ufw allow 'Nginx Full'
```

#### 4. Kích hoạt tường lửa UFW
```bash
sudo ufw enable
```
Hệ thống sẽ hiện cảnh báo:  
`Command may disrupt existing ssh connections. Proceed with operation (y|n)?`  
Nhập `y` và nhấn **Enter** (Vì ta đã mở cổng 22 ở trên nên hoàn toàn an toàn).

#### 5. Kiểm tra trạng thái tường lửa UFW
Xem trạng thái chi tiết:
```bash
sudo ufw status verbose
```
**Kết quả mong đợi:**
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
443/tcp                    ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443/tcp (v6)               ALLOW IN    Anywhere (v6)             
```

Xem danh sách kèm số thứ tự (hữu ích khi cần xóa rule):
```bash
sudo ufw status numbered
```

#### 6. Các lệnh hữu ích khác khi quản trị UFW:
- **Tắt UFW khi cần debug:** `sudo ufw disable`
- **Xóa một rule:** `sudo ufw delete allow 443/tcp` (hoặc `sudo ufw delete <số-thứ-tự>`)
- **Khởi động lại UFW:** `sudo ufw reload`
- **Xem nhật ký chặn kết nối:** `sudo less /var/log/ufw.log`

---

## 📸 Danh Sách Ảnh Chụp Màn Hình Minh Chứng (`screenshot/`)

| Tên tệp ảnh | Nội dung minh chứng |
| :--- | :--- |
| `01-create-user.png` | Lệnh tạo user `tranthikhanhuyen`, cấp group `sudo` và kết quả lệnh `id tranthikhanhuyen` |
| `02-install-software.png` | Lệnh `sudo apt install` và kiểm tra `systemctl status nginx` (active & enabled) |
| `03-git-commits.png` | Lịch sử commit trên git với tối thiểu 4 commit rõ ràng: `git log --oneline` |
| `04-clone-permission.png` | Thư mục `/var/www/devops-hakthon-de006-tranthikhanhuyen` được clone, chown và phân quyền 755/644 |
| `05-nginx-test.png` | Lệnh `sudo nginx -t` (ok) và lệnh `curl -I http://localhost` (HTTP 200 OK) |
| `06-ufw-status.png` | Trạng thái tường lửa: `sudo ufw status verbose` cho thấy cổng 22, 80 đang ALLOW |
| `07-browser-result.png` | Giao diện website hiển thị đầy đủ thông tin sinh viên và bài thi trên trình duyệt |

---
**Chúc bạn hoàn thành xuất sắc bài thi DevOps Hackathon Đề 06!**
