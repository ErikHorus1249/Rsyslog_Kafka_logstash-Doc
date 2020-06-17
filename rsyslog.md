# Cài đặt và cấu hình Rsyslog

## I. Chuẩn bị 

-	Rsyslog server : Ubuntu server 20.04 ( hiện tại t cấu hình trên Kali linux) 
	-	chức năng là lấy log từ client
- Rsyslog client : Ubuntu server 20.04 
	- Chức năng là sinh log
	
Khởi chạy hai server và xác định IP : 

  
> `$ ifconfig`

Hiện tại hai server của mình có IP như sau :
1. Kali linux (Rsyslog Server) : `192.168.1.105`
2. Ubuntu (Rsyslog Server) : `192.168.1.108`

## II. Cài đặt và cấu hình
### 1. Tiến hành trên cả hai máy 
Tiến hành cài đặt Rsyslog trên **[cả hai máy](h)**  sư dụng lệnh sau 

  
> $ sudo apt-get install rsyslog -y
> 

Sau khi cài đặt kiểm tra và khởi động Rsyslog
- Kiểm tra : 
> `$ rsyslog -v`
- Khởi động 
> $ sudo systemctl status rsyslog

? rsyslog.service - System Logging Service
   Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: **active** (**[running](h)**) since Tue 2019-10-22 04:28:55 UTC; 1min 31s ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 724 (rsyslogd)
    Tasks: 4 (limit: 1114)
   CGroup: /system.slice/rsyslog.service
           ??724 /usr/sbin/rsyslogd -n

Oct 22 04:28:53 ubuntu1804 systemd[1]: Starting System Logging Service...
Oct 22 04:28:54 ubuntu1804 rsyslogd[724]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.32.0]
Oct 22 04:28:54 ubuntu1804 rsyslogd[724]: rsyslogd's groupid changed to 106
Oct 22 04:28:54 ubuntu1804 rsyslogd[724]: rsyslogd's userid changed to 102
Oct 22 04:28:54 ubuntu1804 rsyslogd[724]:  [origin software="rsyslogd" swVersion="8.32.0" x-pid="724" x-info="http://www.rsyslog.com"] start
Oct 22 04:28:55 ubuntu1804 systemd[1]: Started System Logging Service.
### 2. Cấu hình trên Rsyslog server
- Sử dụng Nano để sửa file config của Rsyslog

> `$ nano /etc/rsyslog.conf`

Tùy vào việc chọn giao thức để truyền thì có thể sử dụng cả TCP và UDP hoặc chọn 1 trong 2. Sợ bị lỗi thì chọn cả hai .

    $ModLoad imudp 
    $UDPServerRun 514
    
    
    $ModLoad imtcp
    $InputTCPServerRun 514

![](https://i.imgur.com/BsW2W4T.png)

Sau đó thêm vào cuối file : 

>     $AllowedSender TCP, 127.0.0.1, 192.168.0.0/24, *.example.com
>     $AllowedSender UDP, 127.0.0.1, 192.168.0.0/24, *.example.com

    $template remote-incoming-logs, "/var/log/%HOSTNAME%/%PROGRAMNAME%.log" 
    *.* ?remote-incoming-logs
![](https://i.imgur.com/nQHiKFy.png)

Ctrl + X ---> Y để lưu lại.

Sử dụng lệnh sau để kiểm tra lỗi cú pháp sau khi sửa file config

> `$ rsyslogd -f /etc/rsyslog.conf -N1`



Thông báo như sau là ổn : 

    rsyslogd: version 8.32.0, config validation run (level 1), master config /etc/rsyslog.conf
    rsyslogd: End of config validation run. Bye.

Khởi động lại dịch vụ của Rsyslog 

> `$ sudo systemctl restart rsyslog`

Kiểm tra xem cổng 514 trên localhost có đang mở , nếu thấy còn **Listen** là ok

> `$ netstat -ltnp`

![](https://i.imgur.com/vD3tBaM.png)

### 3. Cấu hình Rsyslog client
- Sử dụng Nano để sửa file config của Rsyslog
```
$ nano /etc/rsyslog.conf
```
Thêm vào cuối file 

    ##Enable sending of logs over UDP add the following line:
    
    *.* @192.168.1.105:514
    
    
    ##Enable sending of logs over TCP add the following line:
    
    *.* @@192.168.1.105:514
    
    ##Set disk queue when rsyslog server will be down:
    
    $ActionQueueFileName queue
    $ActionQueueMaxDiskSpace 1g
    $ActionQueueSaveOnShutdown on
    $ActionQueueType LinkedList
    $ActionResumeRetryCount -1
#### <> Tại IP ở hai dòng trên thay bằng IP của máy Server đã lấy từ ban đầu. Thì việc định hướng cho giao thức TCP hay UDP mới được thực hiện. Không sửa IP sẽ gây lỗi.
![](https://i.imgur.com/42hZSEb.png)
![](https://i.imgur.com/6YgorOk.png)


Lưu lại và restart rsyslog.
```
$ sudo systemtcl restart rsyslog
```

## IV. Kết quả
Kết quả thu được là log của máy **client** tức  Ubuntu server sẽ được lưu trữ trong `/var/log/` của **Rsyslog Server** 

Máy client của mình có hostname là erik
Sau khi vào trong /var/log thì xuất hiện folder tên là **[erik](h)** cùng tên **hostname** của máy client là ok.
Trong `/erik` thấy có đủ các file log của máy **client**.
Trong trường hợp gặp lỗi có thể tham khảo hai file rsyslog.conf của client và server của mình [**tại đây**](h)

![](https://i.imgur.com/zvIwm6w.png)

![](https://i.imgur.com/dEvQ5AD.png)

### Xong.
## Preference
- [howtoforge](https://www.howtoforge.com/how-to-setup-rsyslog-server-on-ubuntu-1804/)