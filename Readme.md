# Báo Cáo & Hướng Dẫn Thực Hiện - DevOps Hackathon (Đề Số 06)

**Học viên:** Trần Thị Khánh Huyền  
**Lớp:** K24CNTT3  
**Tài khoản Linux:** `tranthikhanhhuyen-k24cntt3` (hoặc `tranthikhanhuyen`)  
**Mã đề thi:** Đề 06 (DE06)  
**Thư mục mã nguồn triển khai:** `/var/www/devops-hakthon-de006-tranthikhanhuyen`  
**GitHub Repo:** [tranhuyen133/devops-hackathon-de06-tranthikhanhuyen](https://github.com/tranhuyen133/devops-hackathon-de06-tranthikhanhuyen)  

---

## 📁 Cấu Trúc Thư Mục Dự Án

```text
de06_hakathon/
├── src/
│   └── index.html                           # Trang web hiển thị thông tin bài thi & sinh viên
├── nginx/
│   ├── tranthikhanhhuyen-k24cntt3.conf      # File cấu hình Nginx Server Block mẫu (theo tên user máy chủ)
│   └── tranthikhanhuyen.conf                # File cấu hình Nginx Server Block mẫu (theo tên ngắn)
├── ngnix/
│   ├── tranthikhanhhuyen-k24cntt3.conf      # Thư mục dự phòng theo đúng chính tả đề bài
│   └── tranthikhanhuyen.conf
├── screenshot/                              # Thư mục lưu trữ hình ảnh minh chứng kết quả
│   ├── 01-user.png                          # Ảnh chụp lệnh id, whoami
│   └── README.md
├── .gitignore                               # Bỏ qua tệp hệ điều hành, IDE, logs
└── Readme.md                                # Tài liệu hướng dẫn & giải trình bài làm
```

---

## 🛠 PHẦN 1: Chuẩn Bị Môi Trường Linux

Mục tiêu: Khởi tạo tài khoản hệ thống mới chuẩn bảo mật với thư mục home riêng, shell mặc định `/bin/bash` và quyền sudo.

### Bước 1: Tạo tài khoản Linux
> Giả sử tên tài khoản của bạn là `tranthikhanhhuyen-k24cntt3` (hoặc `tranthikhanhuyen`):

Chạy lệnh tạo user có thư mục home (`-m`), shell mặc định `/bin/bash` (`-s /bin/bash`):
```bash
sudo useradd -m -s /bin/bash tranthikhanhhuyen-k24cntt3
```

Đặt mật khẩu cho tài khoản:
```bash
sudo passwd tranthikhanhhuyen-k24cntt3
```
*(Nhập mật khẩu an toàn và xác nhận lại mật khẩu)*

### Bước 2: Thiết lập Primary group trùng với tên user, Secondary group là `sudo`
Primary group mặc định của Linux khi tạo bằng `useradd` sẽ tự động trùng với tên tài khoản.  
Để thêm secondary group `sudo`, chạy lệnh:
```bash
sudo usermod -aG sudo tranthikhanhhuyen-k24cntt3
```

Kiểm tra lại thông tin nhóm:
```bash
id tranthikhanhhuyen-k24cntt3
```
**Kết quả mong đợi:**
```text
uid=1004(tranthikhanhhuyen-k24cntt3) gid=1005(tranthikhanhhuyen-k24cntt3) groups=1005(tranthikhanhhuyen-k24cntt3),27(sudo)
```
> ⚠️ **LƯU Ý QUAN TRỌNG:** Trong ảnh `screenshot/01-user.png`, nếu lệnh `id` chỉ mới hiển thị `groups=1005(tranthikhanhhuyen-k24cntt3)` mà **chưa có `sudo`**, hãy chạy lệnh `sudo usermod -aG sudo tranthikhanhhuyen-k24cntt3`, sau đó chạy `su - tranthikhanhhuyen-k24cntt3` (hoặc thoát ra đăng nhập lại) để nhóm `sudo` có hiệu lực. Khi gõ `id` sẽ thấy xuất hiện group `sudo`. Hãy chụp lại ảnh này để nộp bài đạt điểm tối đa!

### Bước 3: Đăng nhập vào tài khoản mới
Chuyển phiên làm việc sang tài khoản mới:
```bash
su - tranthikhanhhuyen-k24cntt3
```
Kiểm tra đường dẫn hiện tại và quyền hạn:
```bash
pwd          # Kết quả: /home/tranthikhanhhuyen-k24cntt3
whoami       # Kết quả: tranthikhanhhuyen-k24cntt3
sudo whoami  # Nhập mật khẩu user -> Kết quả: root (chứng minh quyền sudo đã hoạt động)
```
> **Nguyên tắc:** Mọi thao tác từ đây trở về sau đều thực hiện bằng tài khoản này, dùng `sudo` khi cần quyền root.

---

## 📦 PHẦN 2: Cài Đặt Phần Mềm & Xây Dựng Mã Nguồn

### 1. Cài đặt các gói phần mềm bằng apt
Cập nhật danh sách gói và cài đặt `nginx`, `git`, `ufw`, `curl`:
```bash
sudo apt update
sudo apt install -y nginx git ufw curl
```

### 2. Đảm bảo Nginx đang chạy (active) và bật tự khởi động (enable)
Bật tự khởi động cùng hệ điều hành:
```bash
sudo systemctl enable nginx
```
Khởi động service Nginx:
```bash
sudo systemctl start nginx
```
Kiểm tra trạng thái Nginx:
```bash
systemctl status nginx
```
Hoặc kiểm tra nhanh:
```bash
systemctl is-active nginx    # Kết quả: active
systemctl is-enabled nginx   # Kết quả: enabled
```

---

### 2.1 Xây Dựng Giao Diện Website (`src/index.html`)
Mã nguồn tệp `src/index.html` được thiết kế chuẩn HTML5, sử dụng CSS hiện đại, bố cục rõ ràng bao gồm:
1. **Cấu trúc chuẩn**: Doctype html, thẻ meta charset UTF-8, viewport responsive.
2. **Thẻ Semantic**: `<header>`, `<main>`, `<section>`, `<footer>`, `<ul>`, `<li>`.
3. **Tiêu đề chính**: "BÀI THI DEVOPS HACKATHON - Đề số 06".
4. **Khối thông tin sinh viên**:
   - Họ và tên: Trần Thị Khánh Huyền
   - Lớp: K24CNTT3
   - Tài khoản Linux: `tranthikhanhhuyen-k24cntt3`
   - Nhóm: `tranthikhanhhuyen-k24cntt3 (sudo)`
   - Email: `tkhuyen1303@gmail.com`
   - GitHub: `tranhuyen133`
5. **Khối thông tin bài thi**:
   - Kỳ thi / Môn học: DevOps Hackathon
   - Mã đề thi: Đề 06 (DE06)
   - Thư mục web: `/var/www/devops-hakthon-de006-tranthikhanhuyen`
   - Cổng dịch vụ: Port 80 (HTTP)
6. **Đoạn giới thiệu ngắn**: Giới thiệu mục tiêu bài thi, nguyên tắc phân quyền và quy trình vận hành web server bảo mật.

---

### 2.2 Quản Lý Mã Nguồn Với Git & GitHub

#### 1. Khởi tạo Git repository và cấu hình `.gitignore`
Tại thư mục gốc của dự án:
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
git commit -m "feat: xay dung giao dien website index.html day du thong tin sinh vien va de thi"

# Commit 3: Thêm cấu hình Nginx Server Block
git add nginx/ ngnix/
git commit -m "feat: cau hinh nginx server block cho tranthikhanhhuyen"

# Commit 4: Cập nhật tài liệu Readme.md và hướng dẫn triển khai
git add Readme.md
git commit -m "docs: hoan thien Readme.md huong dan chi tiet cac buoc trien khai va ufw"
```

Kiểm tra lịch sử commit:
```bash
git log --oneline
```

#### 3. Tạo Repo Public trên GitHub và Push Toàn Bộ Mã Nguồn
1. Truy cập [github.com/new](https://github.com/new), tạo repository mới tên `devops-hackathon-de06-tranthikhanhuyen`, chọn chế độ **Public**.
2. Đẩy toàn bộ mã nguồn lên GitHub:
```bash
git remote add origin https://github.com/tranhuyen133/devops-hackathon-de06-tranthikhanhuyen.git
git branch -M main
git push -u origin main
```

---

## 🚀 PHẦN 3: Triển Khai Website & Tường Lửa UFW

### 3.1 Deploy mã nguồn từ Git lên máy chủ

#### Bước 1: Clone Git Repo về thư mục chỉ định
Trên máy chủ Linux (thực hiện bằng tài khoản của bạn):
```bash
sudo git clone https://github.com/tranhuyen133/devops-hackathon-de06-tranthikhanhuyen.git /var/www/devops-hakthon-de006-tranthikhanhuyen
```

#### Bước 2: Đổi quyền sở hữu để `git pull` sau này không cần `sudo`
Gán quyền sở hữu toàn bộ thư mục cho tài khoản của bạn:
```bash
sudo chown -R tranthikhanhhuyen-k24cntt3:tranthikhanhhuyen-k24cntt3 /var/www/devops-hakthon-de006-tranthikhanhuyen
```
*(Hoặc dùng `tranthikhanhuyen:tranthikhanhuyen` tùy theo tên tài khoản của bạn)*.

Từ bước này, mỗi lần cần cập nhật mã nguồn, bạn chỉ cần:
```bash
cd /var/www/devops-hakthon-de006-tranthikhanhuyen
git pull
```
hoàn toàn không cần gõ `sudo`!

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

### 3.2 Cấu hình Server Block cho Nginx từ template

#### Bước 1: Kích hoạt cấu hình từ repo vào Nginx
Sao chép cấu hình vào thư mục `sites-available`:
```bash
# Chọn file tương ứng với tên tài khoản của bạn:
sudo cp /var/www/devops-hakthon-de006-tranthikhanhuyen/nginx/tranthikhanhhuyen-k24cntt3.conf /etc/nginx/sites-available/tranthikhanhhuyen-k24cntt3.conf
```
Tạo symbolic link sang thư mục `sites-enabled`:
```bash
sudo ln -s /etc/nginx/sites-available/tranthikhanhhuyen-k24cntt3.conf /etc/nginx/sites-enabled/
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

Sau khi cú pháp ok, thực hiện reload Nginx:
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
> Nếu bạn đang thao tác trên máy chủ từ xa (VPS / Cloud VM qua SSH), **BẮT BUỘC** phải cho phép cổng SSH (Port 22) **TRƯỚC KHI** kích hoạt UFW (`ufw enable`). Nếu kích hoạt UFW trước khi mở cổng 22, bạn sẽ bị ngắt kết nối SSH và bị khóa hoàn toàn khỏi máy chủ!

#### Các bước thực hiện:

#### 1. Đặt chính sách mặc định (Default Policies)
Chính sách bảo mật chuẩn: Chặn toàn bộ kết nối đi vào (incoming) và cho phép toàn bộ kết nối đi ra (outgoing):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### 2. Cho phép kết nối SSH (Port 22)
```bash
sudo ufw allow 22/tcp
# Hoặc: sudo ufw allow OpenSSH
```
*Mục đích: Đảm bảo kết nối SSH của bạn không bị mất khi bật tường lửa.*

#### 3. Cho phép dịch vụ Web HTTP (Port 80) và HTTPS (Port 443)
```bash
# Mở cổng HTTP để người dùng truy cập website Nginx
sudo ufw allow 80/tcp

# (Khuyến nghị) Mở cổng HTTPS cho SSL sau này
sudo ufw allow 443/tcp

# Hoặc dùng profile có sẵn của Nginx:
# sudo ufw allow 'Nginx Full'
```

#### 4. Kích hoạt tường lửa UFW
```bash
sudo ufw enable
```
Hệ thống sẽ hỏi xác nhận:  
`Command may disrupt existing ssh connections. Proceed with operation (y|n)?`  
Nhập `y` và nhấn **Enter**.

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

Xem danh sách kèm số thứ tự (hữu ích khi muốn xóa rule):
```bash
sudo ufw status numbered
```

#### 6. Các lệnh quản trị UFW hữu ích:
- **Tắt UFW khi cần debug:** `sudo ufw disable`
- **Xóa một rule:** `sudo ufw delete allow 443/tcp` (hoặc `sudo ufw delete <số-thứ-tự>`)
- **Khởi động lại UFW:** `sudo ufw reload`
- **Xem log UFW:** `sudo tail -f /var/log/ufw.log`

---

## 📸 Danh Sách Ảnh Chụp Màn Hình Minh Chứng (`screenshot/`)

| STT | Tên tệp ảnh | Nội dung minh chứng |
| :---: | :--- | :--- |
| 1 | `01-user.png` | Lệnh `sudo usermod -aG sudo ...` và kết quả lệnh `id`, `whoami` (phải thấy group `sudo`) |
| 2 | `02-install-software.png` | Cài đặt `nginx`, `git`, `ufw`, `curl` và kiểm tra `systemctl status nginx` (active & enabled) |
| 3 | `03-git-commits.png` | Lịch sử commit trên máy local / server: `git log --oneline` (tối thiểu 4 commits) |
| 4 | `04-clone-permission.png` | Clone repo về `/var/www/devops-hakthon-de006-tranthikhanhuyen`, lệnh `chown` và phân quyền `755`/`644` |
| 5 | `05-nginx-test.png` | Cấu hình server block, `sudo nginx -t` (syntax ok) và `curl -I http://localhost` (HTTP 200 OK) |
| 6 | `06-ufw-status.png` | Cấu hình tường lửa và kết quả `sudo ufw status verbose` (active, mở cổng 22, 80) |
| 7 | `07-browser-result.png` | Giao diện website hiển thị trên trình duyệt qua IP máy chủ (`http://<IP-may-chu>`) |

---
*Hoàn thành bởi: Trần Thị Khánh Huyền - Lớp K24CNTT3*
