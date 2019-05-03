**Cập nhật hệ thống**

`yum –y update`

# 1. Cài đặt  InfluxDB, Telegraf và Grafana

## 1.1. InfluxDB

`wget https://dl.influxdata.com/influxdb/releases/influxdb-1.6.0.x86_64.rpm`

`yum -y localinstall influxdb-1.6.0.x86_64.rpm`

`systemctl start influxdb`

`systemctl enable influxdb`

## 1.2. Telegraf

`wget https://dl.influxdata.com/telegraf/releases/telegraf-1.7.2-1.x86_64.rpm`

`yum -y localinstall telegraf-1.7.2-1.x86_64.rpm`

`systemctl start telegraf`

`systemctl enable telegraf`

## 1.3. Grafana

`wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.2.2-1.x86_64.rpm`

`yum -y install initscripts fontconfig`

`yum -y localinstall grafana-5.2.2-1.x86_64.rpm`

`systemctl daemon-reload`

`systemctl enable grafana-server.service`

`systemctl start grafana-server.service`

Sau khi đã cài đặt xong, truy cập vào địa chỉ *IP:3000* để đăng nhập Grafana.

Tài khoản mặc định *admin/admin*

*Đối với các server con còn lại các bạn chỉ cần cài đặt InfluxDB và Telegraf*

Tiến hành cấu hình InfluxDB và Telegraf trên tất cả các server:

**– Thiết lập username & password trong InfluxDB**

`#influx`

`> show databases;`

`> use telegraf`

`> CREATE USER "influx" WITH PASSWORD 'influx_pass' WITH ALL PRIVILEGES`

`> exit`

`#systemctl restart influxdb`

**– Cấu hình Telegraf**

`cp /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.org`

`vi /etc/telegraf/telegraf.conf`

Tìm các phần sau và bỏ dấu *#* phía trước

```
# [[inputs.net]]
# [[inputs.netstat]]
```

Tìm đến [[outputs.influxdb]] và sửa theo dạng như sau

```
[[outputs.influxdb]]
...
database = "telegraf"
...
## HTTP Basic Auth
username = "influx"
password = "influx_pass"
```

Khởi động lại Telegraf

`systemctl restart telegraf`

## 1.4. Add data source

Vào Home Dashboard -> Add data source

Điền thông số cho “Data source” với username = “influx” & password = “influx_pass” tạo ở bước trên để grafana lấy được dữ liệu trên influxdb.

Chọn Save & Test

Nếu bị Network Error: Bad Gateway (502) các bạn cần kiểm tra lại port.

Đơn giản nhất đó chính là disbale firewall.

`systemctl stop firewalld`

`systemctl disable firewalld`
