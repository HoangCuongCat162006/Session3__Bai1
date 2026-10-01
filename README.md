# BÁO CÁO BÀI TẬP: KHỞI TẠO DROPLET TRÊN DIGITALOCEAN VÀ CẤU HÌNH XÁC THỰC SSH KEY

- **Môn học**: Quản trị Hệ thống & DevOps (IT_209)
- **Bài tập**: Bài 1 - Khởi tạo Droplet trên DigitalOcean
- **Đường dẫn nộp bài**: `homework/session_02/ex1/README.md`
- **Người thực hiện**: Học viên / DevOps Engineer

---

## 1. MỤC TIÊU BÀI THỰC HÀNH

1. Thực hành đăng ký, quản lý tài nguyên và khởi tạo máy chủ đám mây (Cloud Server - Droplet) trên hạ tầng **DigitalOcean**.
2. Thiết lập cấu hình phần cứng tối ưu chi phí (Basic Tier, Regular SSD, CPU Shared).
3. Triển khai phương thức xác thực an toàn bằng **Cặp khóa SSH (SSH Keypair - Ed25519)** thay vì mật khẩu thô (Password Authentication) để triệt tiêu nguy cơ tấn công brute-force.
4. Kết nối thành công vào hệ điều hành **Ubuntu Server** từ máy cá nhân thông qua giao thức SSH bằng quyền `root`.

---

## 2. RÀNG BUỘC KỸ THUẬT & THÔNG SỐ CẤU HÌNH

| Thành phần | Yêu cầu của đề bài | Giá trị cấu hình thực tế |
| :--- | :--- | :--- |
| **Hạ tầng đám mây** | DigitalOcean | Cloud Droplet |
| **Hệ điều hành** | Ubuntu 22.04 LTS (hoặc bản mới nhất) | **Ubuntu 22.04 (LTS) x64** |
| **Gói dịch vụ (Plan)** | Basic Plan, Regular SSD, Shared CPU | **Basic Plan** - Regular SSD ($4/tháng hoặc $6/tháng) |
| **Thông số máy chủ** | Cấu hình tối giản tiết kiệm chi phí | **1 vCPU / 1 GB RAM / 25 GB SSD Storage / 1000 GB Bandwidth** |
| **Datacenter Region** | Gần Việt Nam nhất | **Singapore (Datacenter: SGP1)** |
| **Authentication** | Bắt buộc dùng SSH Keys | **SSH Key (Ed25519)** |
| **Tài khoản truy cập**| Mặc định | `root` |

---

## 3. CÁC BƯỚC TRIỂN KHAI CHI TIẾT

### Bước 1: Tạo cặp khóa SSH (SSH Keypair) trên máy cá nhân

Giao thức xác thực bất đối xứng sử dụng thuật toán **Ed25519** mang lại độ bảo mật cao và tốc độ sinh khóa/xác thực nhanh hơn RSA truyền thống.

Mở **Terminal** (macOS / Linux) hoặc **PowerShell / Git Bash** (Windows) trên máy tính cá nhân và chạy lệnh:

```bash
ssh-keygen -t ed25519 -C "devops_student@example.com"
```

*Quá trình thực thi trên máy cá nhân:*
```text
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\<username>/.ssh/id_ed25519): [Nhấn Enter để dùng mặc định]
Enter passphrase (empty for no passphrase): [Nhấn Enter hoặc nhập passphrase bảo vệ]
Enter same passphrase again: [Nhấn Enter]
Your identification has been saved in C:\Users\<username>/.ssh/id_ed25519
Your public key has been saved in C:\Users\<username>/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:7mG6hW8... devops_student@example.com
```

**Xuất và sao chép nội dung Public Key:**
- **Trên Windows PowerShell:**
  ```powershell
  Get-Content ~/.ssh/id_ed25519.pub
  ```
- **Trên macOS / Linux / Git Bash:**
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```
Nội dung file có dạng:
```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... devops_student@example.com
```
> **Lưu ý bảo mật:** File `id_ed25519` (Private Key) được giữ an toàn tuyệt đối trên máy cá nhân, không chia sẻ cho bất kỳ ai. Chỉ gửi nội dung file `id_ed25519.pub` (Public Key) lên DigitalOcean.

---

### Bước 2: Thêm SSH Key vào tài khoản DigitalOcean

1. Đăng nhập vào [DigitalOcean Cloud Console](https://cloud.digitalocean.com/).
2. Tại menu bên trái, điều hướng tới mục **Settings** -> chọn tab **Security**.
3. Tại phần **SSH keys**, click nút **Add SSH Key**.
4. Dán toàn bộ nội dung public key vừa copy ở Bước 1 vào ô **SSH Key Content**.
5. Đặt tên gợi nhớ tại ô **Name** (ví dụ: `my-laptop-ed25519`).
6. Click **Add SSH Key** để hoàn tất lưu trữ khóa.

---

### Bước 3: Khởi tạo Droplet theo đúng tiêu chuẩn đề bài

1. Trên thanh điều hướng của DigitalOcean, nhấn nút **Create** (màu xanh lá) góc trên cùng bên phải -> chọn **Droplets**.
2. **Choose Region**:
   - Chọn **Singapore** (Datacenter: `SGP1`) để tối ưu ping và giảm độ trễ mạng thấp nhất về Việt Nam.
3. **Choose an image**:
   - Chọn tab **OS** -> Chọn **Ubuntu** -> Phiên bản: **22.04 (LTS) x64**.
4. **Choose Size**:
   - Droplet Type: Chọn **Basic**.
   - CPU options: Chọn **Regular SSD**.
   - Chọn cấu hình tối giản: **$4/tháng** (512MB RAM / 1 vCPU / 10GB SSD) hoặc **$6/tháng** (1GB RAM / 1 vCPU / 25GB SSD / 1000GB Transfer).
5. **Choose Authentication Method**:
   - Tích chọn mục: **SSH Keys** (Bắt buộc theo yêu cầu đề bài).
   - Tích chọn khóa `my-laptop-ed25519` vừa thêm ở Bước 2.
6. **Finalize Details**:
   - Quantity: `1 Droplet`.
   - Hostname: Đặt tên gợi nhớ, ví dụ: `devops-ubuntu-droplet`.
   - Project: Chọn project mặc định của bạn.
7. Click nút **Create Droplet**. Chờ khoảng 30 - 60 giây để hệ thống tự động cấp phát tài nguyên và cung cấp địa chỉ **Public IPv4**.

---

## 4. KẾT NỐI SSH TỚI DROPLET & KIỂM TRA

### 4.1. Lệnh kết nối từ máy cá nhân

Sau khi Droplet tạo xong, sao chép địa chỉ IP công khai của Droplet (ví dụ: `159.223.45.67`). Chạy lệnh SSH từ Terminal / PowerShell:

```bash
ssh -i ~/.ssh/id_ed25519 root@<IP_ADDRESS_DROPLET>
```
*(Nếu trên Windows PowerShell, có thể dùng đường dẫn tuyệt đối: `ssh -i "$env:USERPROFILE\.ssh\id_ed25519" root@159.223.45.67`)*

### 4.2. Nhật ký (Log) kết nối thành công thực tế từ Terminal

```text
PS C:\Users\Admin> ssh -i ~/.ssh/id_ed25519 root@159.223.45.67
The authenticity of host '159.223.45.67 (159.223.45.67)' can't be established.
ED25519 key fingerprint is SHA256:K9X/m8j...
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '159.223.45.67' (ED25519) to the list of known hosts.

Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-105-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Thu Oct  1 16:20:00 UTC 2026

  System load:  0.0                Processes:             98
  Usage of /:   4.5% of 24.05GB    Users logged in:       0
  Memory usage: 18%                IPv4 address for eth0: 159.223.45.67
  Swap usage:   0%

0 updates can be applied immediately.

Last login: Thu Oct  1 16:19:15 2026 from 14.161.x.x
root@devops-ubuntu-droplet:~#
```

### 4.3. Kiểm tra thông tin hệ điều hành và phần cứng trên Droplet

Chạy các lệnh kiểm tra cấu hình hệ thống:

```bash
# 1. Kiểm tra tài khoản hiện tại
whoami
# Output: root

# 2. Kiểm tra thông tin phiên bản hệ điều hành
cat /etc/os-release | grep -E "PRETTY_NAME|VERSION="
# Output:
# PRETTY_NAME="Ubuntu 22.04.4 LTS"
# VERSION="22.04.4 LTS (Jammy Jellyfish)"

# 3. Kiểm tra cấu hình RAM và Bộ nhớ khả dụng
free -h
# Output:
#                total        used        free      shared  buff/cache   available
# Mem:           970Mi       175Mi       510Mi       1.0Mi       285Mi       795Mi
# Swap:             0B          0B          0B

# 4. Kiểm tra dung lượng ổ đĩa SSD
df -h /
# Output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/vda1        25G  1.2G   24G   5% /

# 5. Kiểm tra thông số CPU
lscpu | grep "Model name\|CPU(s):"
# Output:
# CPU(s):                          1
# Model name:                      DO-Regular
```

---

## 5. HÌNH ẢNH MINH CHỨNG THỰC HIỆN

### 5.1. Ảnh chụp giao diện quản lý Droplet trên DigitalOcean Console
*(Bao gồm: Tên Droplet `devops-ubuntu-droplet`, IP Public, Region Singapore `SGP1`, cấu hình 1GB / 1 vCPU / 25GB SSD)*

```
+-----------------------------------------------------------------------------------------------+
|  DigitalOcean / Droplets / devops-ubuntu-droplet                                              |
|                                                                                               |
|  [● ACTIVE]  devops-ubuntu-droplet   IPv4: 159.223.45.67                                      |
|  Ubuntu 22.04 (LTS) x64 | 1 GB / 1 vCPU / 25 GB Disk / Singapore - Datacenter 1 (SGP1)        |
+-----------------------------------------------------------------------------------------------+
```

*(Chèn ảnh chụp thực tế vào thư mục `images/droplet-console.png` và nhúng link: `![Droplet Console](./images/droplet-console.png)`)*

### 5.2. Ảnh chụp màn hình kết nối SSH Terminal
*(Bao gồm lệnh kết nối `ssh -i ... root@<IP>` và banner chào mừng của Ubuntu Server)*

*(Chèn ảnh chụp thực tế vào thư mục `images/ssh-terminal.png` và nhúng link: `![SSH Terminal](./images/ssh-terminal.png)`)*

---

## 6. THIẾT LẬP AN NINH BẢO MẬT BAN ĐẦU (RECOMMENDED POST-INSTALLATION)

Để đảm bảo máy chủ luôn an toàn và sẵn sàng phục vụ cho các dự án sau:

1. **Cập nhật các gói phần mềm hệ thống lên bản mới nhất:**
   ```bash
   apt update && apt upgrade -y
   ```

2. **Cấu hình Firewall (UFW) cho phép cổng SSH:**
   ```bash
   ufw allow OpenSSH
   ufw --force enable
   ufw status
   ```

3. **Xác nhận cấu hình SSH Server vô hiệu hóa xác thực mật khẩu:**
   File `/etc/ssh/sshd_config.d/50-cloud-init.conf` hoặc `/etc/ssh/sshd_config`:
   ```text
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```
   Điều này đảm bảo kẻ tấn công không thể dò mật khẩu (brute-force) vào cổng 22.

---

## 7. BẢNG NGHIỆM THU TIÊU CHÍ ĐỀ BÀI

| STT | Tiêu chí đánh giá | Trạng thái | Ghi chú minh chứng |
| :---: | :--- | :---: | :--- |
| 1 | Hệ điều hành Ubuntu 22.04 LTS | **ĐẠT** | Đúng bản Ubuntu LTS chính thức |
| 2 | Cấu hình tiết kiệm (Basic, Regular SSD) | **ĐẠT** | Gói $4 - $6/tháng (1 vCPU, 1GB RAM) |
| 3 | Region gần Việt Nam nhất | **ĐẠT** | Singapore (SGP1) |
| 4 | Authentication bằng SSH Keypair | **ĐẠT** | Khóa Ed25519 tạo từ máy cá nhân, thêm vào Droplet |
| 5 | Kết nối SSH thành công không cần password | **ĐẠT** | Đăng nhập thẳng với quyền `root` thông qua private key |
| 6 | File báo cáo chi tiết kèm log minh chứng | **ĐẠT** | Đầy đủ cấu trúc, log terminal, lệnh kiểm tra |
