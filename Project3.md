# 1. Giới thiệu

Nagios Core là hệ thống giám sát Opensource khá mạnh mẽ, đi kèm với nó là những plugin có thể theo dõi tài nguyên hệ thống và các dịch vụ như HTTP, FTP, SSH, SMTP, v.v...

## 1.1. Môi trường cài đặt

Để có thể cài đặt và sử dụng được Nagios Core, phải cài đặt một số thư viện và các gói dịch vụ đi kèm.

`sudo apt-get install autoconf gcc libc6 build-essential bc gawk dc gettext \
libmcrypt-dev libssl-dev make unzip apache2 apache2-utils php libgd2-xpm-dev \
libapache2-mod-php7.0 php7.0-mysql php7.0-curl php7.0-json`

## 1.2. Tạo Nagios User và Group

Tạo user nagios và thiết lập password cho nó

`sudo useradd -m -s /bin/bash nagios`

•	m: Tạo thư mục home cho user

•	s: User sử dụng Bash Shell mặc định

Tạo group nagcmd cho phép sử dụng thư mục Web UI, thêm nagios và www-data (user của apache):

`sudo groupadd nagcmd`

`sudo usermod -a -G nagcmd nagios`

`sudo usermod -a -G nagcmd www-data`

# 2. Cài đặt Nagios Core

Tải Nagios Core và Plugin

`mkdir ~/nagios`

`cd ~/nagios`

`wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-4.3.1.tar.gz`

`wget https://nagios-plugins.org/download/nagios-plugins-2.2.0.tar.gz`

`curl -L -O http://downloads.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz`

## 2.1. Cài đặt Nagios Core

Sau khi tải xong, giải nén và bắt đầu phần biên dịch Nagios Core

`cd ~/nagios`

`tar xzf nagios-4.3.1.tar.gz`

Sau đó thay đổi thư mục được trích xuất:

`cd nagios-4.3.1`

Trước khi xây dựng Nagios, phải định dạng nó. Nếu muốn định dạng nó để sử dụng postfix (có thể cài đặt bằng apt-get), hãy thêm *--with-mail=/usr/sbin/sendmail vào lệnh sau:*

`./configure --with-nagios-group=nagios --with-command-group=nagcmd`

Biên dịch Nagios với lệnh này:

`make all` 

Bây giờ có thể chạy các lệnh này để cài đặt Nagios, init script và các tệp cấu hình mẫu:

`sudo make install`

`sudo make install-commandmode`

`sudo make install-init`

`sudo make install-config`

`sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf`

Cho phép Nagios khởi động cùng với hệ thống:

`sudo update-rc.d nagios defaults`

Bật các tính năng để Nagios Web UI có thể chạy

`sudo make install-webconf`

`sudo a2enmod rewrite`

`sudo a2enmod cgi`

Cài đặt password cho nagiosadmin, khi đăng nhập Web:

`sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin`

Nếu sử dụng tường lửa UFW, hãy thêm rule để mở port cho Web UI.

`sudo ufw allow Apache`

`sudo ufw reload`

## 2.2.	Cài đặt Nagios Plugins

Giải nén Nagios Plugins lưu trữ với lệnh này:

`cd ~/nagios`

`tar xzf nagios-plugins-2.2.0.tar.gz`

`cd nagios-plugins-2.2.0`

Trước khi xây dựng Nagios Plugins, phải định dạng nó

`./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl `

Bây giờ biên dịch Nagios Plugins và cài đặt

`make `

`sudo make install`

# 3. Cài đặt NRPE

NRPE - (Nagios Remote Plugin Executor) là một công cụ đi kèm để theo dõi tài nguyên hệ thống, nó còn được biết như một Agent để theo dõi các host từ xa (Remote hosts)

Sau khi tải xong, giải nén và bắt đầu phần cấu hình NRPE

`cd ~/nagios`

`tar xf nrpe-*.tar.gz`

Sau đó thay đổi thư mục được trích xuất:

` ~/nagios/nrpe-*`

Định cấu hình NRPE bằng các lệnh sau:

`./configure --enable-command-args --with-nagios-user=nagios --with-nagios-group=nagios --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu`

Bây giờ hãy xây dựng và cài đặt NRPE và tập lệnh khởi động xinetd của nó bằng các lệnh sau:

`make all `

`sudo make install`

`sudo make install-xinetd`

`sudo make install-daemon-config`

Mở tập lệnh khởi động xinetd trong trình chỉnh sửa:

`sudo vi /etc/xinetd.d/nrpe`

Sửa đổi dòng only_from bằng cách thêm địa chỉ IP riêng của Nagios server vào cuối
```
...
only_from = 127.0.0.1 192.168.64.100
...
```
Khởi động lại dịch vụ xinetd để khởi động NRPE:

`sudo service xinetd restart `

