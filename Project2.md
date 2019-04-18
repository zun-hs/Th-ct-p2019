# 1. Giới thiệu

NFS viết tắt của từ Network File System, là một giao thức dùng để chia sẻ dữ liệu giữa các máy sử dụng *unix với nhau thông qua môi trường mạng được phát triển bởi SUN.

•	Dịch vụ NFS hoạt động theo giao thức TCP với mô hình Client-Server

•	Cho phép các client kết nối đến phân vùng chia sẻ và sử dụng như một phân vùng cục bộ

•	Hiện tại NFS có 4 phiên bản. NFSv4 là phiên bản đang được sử dụng nhiều nhất và hỗ trợ phát huy tối đa của giao thức NFS

•	Một số vấn đề với NFS

	Không bảo mật, mã hóa dữ liệu

	Hiệu suất hoạt động trung bình ở mức khá, nhưng không ổn định

	Dữ liệu phân tán có thể bị phá vỡ nếu có nhiều phiên sử dụng đồng thời

# 2. Thành phần

Server

•	Là nơi chứa các thư mục, các file chia sẻ

•	Đóng vai trò là một filesystem

Client

•	Yêu cầu thông tin về các file, thư mục được chia sẻ

File handle

•	Cách để truy cập một file mà không cần biết tên file

•	Tương tác giống như đang làm việc ở trên một filesystem cục bộ

# 3. Giao thức NFS
•	NFS là giao thức hoạt động theo cơ chế RPC (Thủ tục gọi hàm từ xa thông qua một cổng mạng bất kỳ nào đó)

•	Sử dụng như tài nguyên cục bộ

•	Hoạt động theo kiểu stateless

•	Được thiết kế dành riêng cho hệ thống Linux

•	NFS được xây dựng từ 4 giao thức riêng biệt

	Nfs: Đọc ghi, truy cập các file. Thống kê và xác thực

	Mountd: Dùng để mount folder

	Nsm: Giám sát trạng thái của mạng ở client và server

	Nlm: Quản lý các Lock mạng, cho phép tương tác đồng thời

# 4. NFS Server

## 4.1. File cấu hình, dịch vụ

Có ba tập tin cấu hình chính, bạn sẽ cần phải chỉnh sửa để thiết lập một máy chủ NFS: /etc/exports; /etc/hosts.allow; /etc/hosts.deny

a)	File /etc/exports

`dir host1(options) host2(options) hostN(options)`

•	dir: thư mục hoặc file system muốn chia sẻ.

•	host: một hoặc nhiều host được cho phép mount dir. Có thể được định nghĩa là một tên, một nhóm sử dụng ký tự, * hoặc một nhóm sử dụng 1 dải địa chỉ mạng/subnetmask...

•	options: định nghĩa 1 hoặc nhiều options khi mount.

Các options:

•	ro: thư mục được chia sẻ chỉ đọc được; client không thể ghi lên nó.

•	rw: client có thể đọc và ghi trên thư mục.

•	no_root_squash: Mặc định, bất kỳ file truy vấn được tạo bởi người dùng (root) máy trạm đều được xử lý tương tự nếu nó được tạo bởi user nobody. Nếu no_root_squash được chọn, user root trên client sẽ giống như root trên server.

•	no_subtree_check: Nếu chỉ 1 phần của ổ đĩa được chia sẻ, 1 đoạn chương trình gọi là “thẩm tra lại việc kiểm tra cây con” được yêu cầu từ phía client (nó là 1 file trong phân vùng được chia sẻ). Nếu toàn bộ ổ đĩa được chia sẻ, việc vô hiệu hóa sự kiểm tra này sẽ tăng tốc độ truyền tải.

•	sync: thông báo cho client biết 1 file đã được ghi xong- tức là nó đã được ghi để lưu trữ an toàn-khi mà NFS hoàn thành việc kiểm soát ghi lên các file hệ thống. cách xử lí này có thể là nguyên nhân làm sai lệch dữ liệu nếu server khởi động lại.

Ví dụ 1 file cấu hình mẫu

`/usr/local *.123.vn(ro)`

`/home 192.168.1.0/255.255.255.0(rw)`

`/var/tmp 192.168.1.1(rw) 192.168.1.3(rw)`

•	Dòng thứ nhất: Cho phép tất cả các host với tên miền định dạng “somehost”.123.vn được mount thư mục /usr/local với quyền chỉ đọc.

•	Dòng thứ hai: Cho phép bất kỳ host nào có địa chỉ IP thuộc subnet 192.168.1.0/24 được mount thư mục /home với quyền đọc và ghi.

•	Dòng thứ ba: Cho phép 2 host được mount thư mục /var/tmp với quyền đọc và ghi.

b)	 File /etc/hosts.allow và /etc/hosts.deny

Hai tập tin đặc biệt này giúp xác định các máy tính trên mạng có thể sử dụng các dịch vụ trên máy của bạn. Mỗi dòng trong nội dung file chứa duy nhất 1 danh sách gồm 1 dịch vụ và 1 nhóm các máy tính. Khi server nhận được yêu cầu từ client, các công việc sau sẽ được thực thi:

•	Kiểm tra file host.allow – nếu client phù hợp với 1 quy tắc được liệt kê tại đây thì nó có quyền truy cập.

•	Nếu client không phù hợp với 1 mục trong host.allow server chuyển sang kiểm tra trong host.deny để xem thử client có phù hợp với 1 quy tắc được liệt kê trong đó hay không (host.deny). Nếu phù hợp thì client bị từ chối truy cập.

•	Nếu client phù hợp với các quy tắc không được liệt kê trong cả 2 file thì nó sẽ được quyền truy cập.

Ví dụ: Muốn chặn hoặc cho phép một host hoặc network thì thêm vào file deny hoặc allow.

`portmap: 10.10.10.5, 10.10.10.0/24`

## 4.2. Các dịch vụ có liên quan

Để sử dụng dịch vụ NFS, cần có các daemon (dịch vụ chạy ngầm trên hệ thống) sau:

•	Portmap: Quản lý các kết nối, dịch vụ chạy trên port 2049 và 111 ở cả server và client.

•	NFS: Khởi động các tiến trình RPC (Remote Procedure Call) khi được yêu cầu để phục vụ cho chia sẻ file, dịch vụ chỉ chạy trên server.

•	NFS lock: Sử dụng cho client khóa các file trên NFS server thông qua RPC.

a)	Khởi động portmapper

NFS phụ thuộc vào tiến trình ngầm quản lý các kết nối (portmap hoặc rpc.portmap), chúng cần phải được khởi động trước.

Nó nên được đặt tại /sbin nhưng đôi khi trong /usr/sbin. Hầu hết các bản phân phối linux gần đây đều khởi động dịch vụ này trong „kịch bản khởi động‟ (boot scripts –tự khởi động khi server khởi động) nhưng vẩn phải đảm bảo nó được khởi động đầu tiên trước khi bạn làm việc với NFS (chỉ cần gõ lệnh netstat -anp |grep portmap để kiểm tra).

b)	Các tiến trình ngầm

Dịch vụ NFS được hỗ trợ bởi 5 tiến trình ngầm:

•	rpc.nfsd: thực hiện hầu hết mọi công việc.

•	rpc.lockd and rpc.statd: quản lý việc khóa các file.

•	rpc.mountd: quản lý các yêu cầu gắn kết lúc ban đầu.

•	rpc.rquotad: quản lý các hạn mức truy cập file của người sử dụng trên server được truy xuất.

•	lockd: được gọi theo yêu cầu của nfsd. Vì thế bạn cũng không cần quan tâm lắm tới việc khởi động nó.

•	statd: thì cần phải được khởi động riêng.

Tuy nhiên trong các bản phân phối linux gần đây đều có kịch bản khởi động cho các tiến trình trên. Tất cả các tiến trình này đều nằm trong gói nfs-utils, nó có thể được lưu giữ trong /sbin hoặc /usr/sbin. Nếu bản phân phối của bạn không tích hợp chúng trong kịch bản khởi động, thì bạn nên tự thêm chúng vào, cấu hình theo thứ tự sau đây:

`rpc.portmap`

`rpc.mountd `

`rpc.nfsd`

## 4.3. Xác minh các dịch vụ của NFS	đang chạy

Để làm điều này, ta truy vấn portmapper với lệnh *rpcinfo để tìm ra dịch vụ nào đang được cung cấp.

## 4.4. Cập nhật thay đổi cho /etc/exports

Nếu thay đổi trong /etc/exports, các thay đổi đó có thể chưa có hiệu lực ngay lập tức, bạn phải thực thi lệnh **exportfs -ra** để bắt nfst cập nhật lại nội dung file /etx/exports.

Nếu không tìm thấy lệnh exportfs thì bạn có thể kết thúc nfsd với lệnh HUD.

Nếu các việc đó không hoạt động, đừng quên kiểm tra lại hosts.allow để đảm bảo rằng bạn không quên việc khai báo danh sách các máy con trong đấy. Ngoài ra cũng nên kiểm tra danh sách các máy chủ trên bất kỳ hệ thống tường lửa nào mà bạn đã thiết lập.

## 4.5. Chức năng mount thự động khi boot

Chỉnh sửa file fstab bên phía client. Để đảm bảo cho hoạt động nên dùng hard,intr cho nfs.

## 4.6. Các trường hợp dùng NFS

•	Ứng dụng hỗ trợ: VDI, Oracle, VMware ESXi, SAS Grid, SAP HANA, TIBCO, OpenStack, Docker, etc

•	Các khách hàng lớn.

•	Đơn giản, dễ quản lý.

•	Không cần client OS file system.

•	Dễ dàng mở rộng, thu hồi.

•	Dễ dàng di chuyển các storage.

•	Chạy trên Ethernet.

•	Hiệu suất lớn, độ trễ thấp. Hiệu suất tốt hơn iSCSl trong vài trường hợp.

## 4.7. Chú ý

•	Có thể cấu hình tùy chọn kích thước gói dữ liệu để tối ưu khi truyền với options rsize và wsize

•	Không đặt rsize và wsize lớn hơn MTU của mạng truyền.

•	Dùng nfsstat để có thêm thông tin về hoạt động của NFS.

•	Thay đổi port cho statd: statd -p 32765 -o 32766

•	Thay đổi port mountd: mountd -p 32767

NFS có 2 chế độ mount:

•	Mount cứng là ghi trực tiếp vào file /etc/fstab

•	Mount mềm là mount bằng lệnh thông thường và bị mất khi máy tính được khởi động lại

# 5. Demo NFS

## 5.1. Mô hình hệ thống

NFS Server: 192.168.1.5

NFS Client: 192.168.1.3 và 192.168.1.4

## 5.2. Cài đặt và cấu hình

**Cài đặt NFS Server**

Đăng nhập tài khoản root và sử dụng lệnh

`yum install nfs-utils`

Khởi động NFS Server

`systemctl start rpcbind nfs-server`

Đặt NFS Server khởi động cùng server

`systemctl enable rpcbind nfs-server`

Kiểm tra port sử dụng bởi NFS

`rpcinfo -p`

Cấu hình Firewall để cho phép truy cập

`firewall-cmd --permanent --add-service=nfs`

`firewall-cmd --permanent --add-service=mountd`

`firewall-cmd --permanent --add-service=rpc-bind`

`firewall-cmd --permanent --add-port=2049/tcp`

`firewall-cmd --permanent --add-port=2049/udp`

`firewall-cmd --reload`

Tạo thư mục chia sẻ trên Server

`mkdir /nfs_share`

Sửa file /etc/exports để tạo mountpoint export

`vi /etc/exports`

Thêm nội dung sau:

```
/nfs_share 192.168.1.3(rw,no_root_squash) 192.168.1.4(rw,no_root_squash)
```
Mỗi khi chỉnh sửa **/etc/exports** chúng ta phải thực hiện lệnh dưới thì các thay đổi mới có hiệu quả

`exportfs -a`

Kiểm tra mount point trên server

`showmount -e localhost`

**Trên máy Client**

Thực hiện cài đặt NFS Client bằng việc cài 2 gói nfs-utils và nfs-utils-lib

`yum install nfs-utils nfs-utils-lib`

Để kiểm tra mount point trên NFS Server từ client, sử dụng command “showmount -e <NFS_Server_IP>”

`showmount -e 192.168.1.5`

Tạo và mount thư mục để mount tới NFS Server

`mkdir /nfs_mount`

`mount 192.168.1.5:/nfs_share /nfs_mount`

Để kiểm tra thông tin đã mount trên client sử dụng command sau

`nfsstat -m`

Hiện thị thư mục chia sẻ vừa gán

`df -h`

và lệnh

`mount`

**Kiểm tra:**

Trên thành phần client (192.168.1.3), thử tạo file thử nghiệm trên NFS chia sẻ

`touch /nfs_mount/test.txt`

Trên Server: ls –l /nfs_share để kiểm tra

Trên Client (192.168.1.4): ls –l /nfs_mount cũng thấy file test.txt đã tạo

Để tự động mount tới NFS Server khi máy chủ reboot, thì bạn sửa file /etc/fstab, thêm dòng sau vào cuối file

`192.168.1.5:/nfs_share /nfs_mount nfs rw, sync, hard, intr 0 0`
