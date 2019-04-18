# 1. Cài đặt Apache
Cập nhật trước khi cài update
*sudo apt update*
Cài apache2
*sudo apt install apache2*
Khởi động lại apache2
*sudo systemctl start apache2.service*
Kiểm tra phiên bản apache2
*apache2 –v*
# 2. Cài đặt MySQL
Cài MySQL
*sudo apt-get install mysql-server*
Tạo cơ sở dữ liệu
* sudo mysql –u root –p *
CREATE DATABASE <tên cơ sở dữ liệu>;
Tạo người dùng cho cơ sở dữ liệu
CREATE USER ‘tên user’@’localhost’ IDENTIFIED BY ‘mật khẩu’;
Gán quyền cho người dùng cơ sở dữ liệu
GRANT ALL ON tên cơ sở dữ liệu.* TO ‘tên người dùng’@’localhost’ IDENTIFIED BY ‘mật khẩu người dùng’ WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
