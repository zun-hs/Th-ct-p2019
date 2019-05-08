# 1. Giới thiệu

Cân bằng tải là một phương pháp phân phối khối lượng truy cập trên nhiều máy chủ nhằm tối ưu hóa tài nguyên hiện có đồng thời tối đa hóa thông lượng, giảm thời gian đáp ứng và tránh tình trạng quá tải cho một máy chủ.

# 2. Cấu hình HAProxy làm Load balancing cho Apache

## 2.1. Thiết lập ban đầu tại mỗi node loadbalancer và web

Thiết lập hostname, cập nhật hệ thống

`hostnamectl set-hostname hostname`

`yum update -y`

Tắt Firewall và SELinux

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux`

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

`systemctl stop firewalld`

`systemctl disable firewalld`

Cấu hình Host file

`echo "192.168.100.191 loadbalancer1" >> /etc/hosts`

`echo "192.168.100.192 loadbalancer2" >> /etc/hosts`

`echo "192.168.100.196 web1" >> /etc/hosts`

`echo "192.168.100.198 web2" >> /etc/hosts`

Khởi động lại hệ thống

`init 6`

## 2.2. Cài đặt

**Tại node loadbalancer**

`sudo yum install wget socat -y`

`wget http://cbs.centos.org/kojifiles/packages/haproxy/1.8.1/5.el7/x86_64/haproxy18-1.8.1-5.el7.x86_64.rpm`

`yum install haproxy18-1.8.1-5.el7.x86_64.rpm -y`

Chỉnh sửa cấu hình HAProxy

`vi /etc/haproxy/haproxy.cfg`

```
log 127.0.0.1 local2
chroot	/var/lib/haproxy
pidfil	/var/run/haproxy.pid
maxconn	4000
user	haproxy
group	haproxy
daemon
stats socket	/var/lib/haproxy/stats
defaults
mode	http
log	global
option	httplog
option	dontlognull
option http-server-close
option forwardfor	except 127.0.0.0/8
option	redispatch
retries		3
timeout http-request	10s
timeout queue	1m
timeout connect	10s
timeout client	1m
timeout server	1m
timeout http-keep-alive	10s
timeout check	10s
maxconn	3000

listen	stats
bind: 8080
mode	http
stats	enable
stats uri	/stats
stats	realm HAProxy\ Statistics

listen webcluster
bind:	80
balance	roundrobin
mode	http
option	forwardfor
server web1 192.168.100.196:80 check
server web2 192.168.100.198:80 check
```

Khởi động lại dịch vụ HAProxy

`sudo systemctl enable haproxy`

`sudo systemctl restart haproxy`

**Tại node web**

Tạo một file index.html

`vi /var/www/html/index.html`

**Kiểm tra HAProxy status page**

Truy cập địa chỉ *http://192.168.100.191:8080/stats*

**Cấu hình Log cho HAProxy**

Chỉnh sửa cấu hình *rsyslog.conf*

`sed -i "s/#\$ModLoad imudp/\$ModLoad imudp/g" /etc/rsyslog.conf`

`sed -i "s/#\$UDPServerRun 514/\$UDPServerRun 514/g" /etc/rsyslog.conf`

`echo '$UDPServerAddress 127.0.0.1' >> /etc/rsyslog.conf`

Tạo file cấu hình rsyslog cho HAProxy

`echo 'local2.*    /var/log/haproxy.log' > /etc/rsyslog.d/haproxy.conf`

Khởi động lại Rsyslog và HAProxy

`systemctl restart rsyslog`

`systemctl restart haproxy`

# 3. Cài đặt Keepalived tạo và quản lý Vitual IP

**Trên node loadbalance**

`yum install -y keepalived`

Trên lb1 cấu hình cho nó mode active (master)

`vi /etc/keepalived/keepalived.conf`

```
vrrp_script chk_haproxy {
	script “killall -0 haproxy”
	interval 2
	weight 2 
}

vrrp_instance VI_1 {
	interface eth0
	state MASTER
	virtual_router_id 51
	priority 101   # 101 on master, 100 on backup
	virtual_ipaddress {
	192.168.100.123
	}
	track_script {
		chk_haproxy
	}
}
```

**Cấu hình chia sẻ Virtual IP (Làm cả trên lb1 và lb2)**

Thêm vào file *sysctl.conf*

`echo "net.ipv4.ip_nonlocal_bind=1" >> /etc/sysctl.conf`

Chạy lệnh để hệ thống nhận file cấu hình và kiểm tra lại xem đã có dòng vừa thêm chưa:

`sysctl -p`

Cài thêm gói psmisc để thêm tiện ích killall (Cài trên cả 2 node)

`yum install -y psmisc`

Sau đó khởi động keepalived ở lb1

`/etc/init.d/keepalived start`

`chkconfig keepalived on`

**Tương tự ở lb2**

Sau khi cấu hình xong, truy cập *http:// 192.168.100.123* thành công






