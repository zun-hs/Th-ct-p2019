# 1. Cài đặt Apache

Cập nhật trước khi cài update

`sudo apt update`

Cài apache2

`sudo apt install apache2`

Khởi động lại apache2

`sudo systemctl start apache2.service`

Kiểm tra phiên bản apache2

`apache2 –v`

# 2. Cài đặt MySQL

Cài MySQL

`sudo apt-get install mysql-server`

Tạo cơ sở dữ liệu

`sudo mysql –u root –p`

`CREATE DATABASE <tên cơ sở dữ liệu>;`

Tạo người dùng cho cơ sở dữ liệu

`CREATE USER ‘tên user’@’localhost’ IDENTIFIED BY ‘mật khẩu’;`

Gán quyền cho người dùng cơ sở dữ liệu

`GRANT ALL ON tên cơ sở dữ liệu.* TO ‘tên người dùng’@’localhost’ IDENTIFIED BY ‘mật khẩu người dùng’ WITH GRANT OPTION;`

`FLUSH PRIVILEGES;`

`EXIT;`

# 3. Cài đặt PHP

Cài đặt PHP

`sudo apt-get –y install php7.2`

Tạo tệp để kiểm tra PHP có cài thành công không

`sudo nano /var/www/html/info.php`

Thêm câu lệnh sau vào tệp trên:

```
<?php
phpinfo();
```

# 4. Cài đặt Wordpress

Tải và giải nén wordpress

`cd /tmp && wget https://wordpress.org/latest.tar.gz`

`tar –zxvf latest.tar.gz`

`sudo mv wordpress /var/www/html/wordpress`

Đặt quyền chính xác cho Wordpress hoạt động

`sudo chown –R www-data:www-data /var/www/html/wordpress/`

`sudo chmod –R 755 /var/www/html/wordpress/`

Cấu hình máy chủ HTTP Apache2

`sudo nano /etc/apache2/sites-available/wordpress.conf`

Thêm các câu lệnh sau vào tệp trên:

```
<VirtualHost *:80>
  ServerAdmin admin@example.com
  DocumentRoot /var/www/html/wordpress/
  ServerName example.com
  ServerAlias www.example.com
  
  <Directory /var/www/html/wordpress/>
    Option +FollowSymlinks
    AllowOverride All
    Require all granted
  </Directory>
  
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtuaHost>
```

Kích hoạt Mô-đun Wordpress

`sudo a2ensite wordpress.conf`

`sudo a2enmod rewrite`

`sudo systemctl restart apache2.service`

Tạo tệp wp-config.php của Wordpress

`sudo mv /var/www/html/wordpress/wp-config-sample.php /var/www/html//wordpress/wp-config-sample.php`

`sudo nano /var/www/html//wordpress/wp-config-sample.php`

Sửa lại tên cơ sở dữ liệu và người dùng phù hợp:

```
define('BD_NAME','tên cơ sở dữ liệu');
define('BD_USER','user');
define('BD_PASSWORD','password');
```
 
Tích hợp PHP và MySQL

`sudo apt-get –y install php7.2-mysql`

`sudo apt-get install php-pear php7.2-curl php7.2-dev php7.2-gd php7.2-mbstring php7.2-zip php7.2-mysql php7.2-xml`

Khởi động lại apache2

`sudo service apache2 restart`

# 5. Cài đặt Wordpress Multisite

Để kích hoạt multisite Network, cần sửa tập tin wp-config.php như sau:

Chèn mã sau vào dưới hàm **<?php**

`define( 'WP_ALLOW_MULTISITE', true );`

Sau đó, restart PHP-FPM để xóa Opcache:

`sudo service php-fpm restart`

Truy cập trang quản trị */wp-admin/* mục *Tools – Network Setup* để bắt đầu cài đặt:

Tiếp đó, sửa file *wp-config.php* bằng cách thêm yêu cầu sau vào sau phần /* That’s all, stop editing! Happy blogging. */

```
define('MULTISITE',true);
define('SUBDOMAIN_INSTALL',true);
define('DOMAIN_CURRENT_SITE','hocvps.com');
define('PATH_CURRENT_SITE','/');
define('SITE_ID_CURRENT_SITE',1);
define('BLOG_ID_CURRENT_SITE',1);
```

Thiết lập thành công, truy cập lại hệ thống qua */wp-admin/*

# 6. Set Basic_Authen

Tạo tệp tin mật khẩu

`sudo htpasswd -c /etc/apache2/.htpasswd tên người dùng`

Cấu hình xác thực mật khẩu Apache

`sudo nano /etc/apache2/sites-enabled/000-default.conf`

```
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
  
  <Directory "/var/www/html">
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
  </Directory>
</VirtuaHost>
```

Định cấu hình kiểm soát truy cập với tệp .htaccess

`sudo nano /etc/apache2/apache2.conf`

```
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
 </Directory>
```

`sudo nano /var/www/html/.htaccess`

```
AuthType Basic
AuthName "Restricted Content"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

Khởi động lại máy chủ web xác minh thành công

`sudo systemctl restart apache2`

`sudo systemctl status apache2`
