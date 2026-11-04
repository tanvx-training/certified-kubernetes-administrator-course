# Kiến thức nền tảng: DNS

  - Đi tới [Lecture](https://kodekloud.com/topic/prerequsite-dns/)

Trong phần này, chúng ta sẽ tìm hiểu **DNS trên Linux**

## Phân giải tên (Name Resolution)

- Dùng lệnh `ping` để kiểm tra một địa chỉ IP trong mạng có liên lạc được không.

```
$ ping 172.17.0.64
PING 172.17.0.64 (172.17.0.64) 56(84) bytes of data.
64 bytes from 172.17.0.64: icmp_seq=1 ttl=64 time=0.384 ms
64 bytes from 172.17.0.64: icmp_seq=2 ttl=64 time=0.415 ms

```
- Thử ping bằng hostname

```
$ ping web
ping: unknown host web

```
- Thêm entry vào file `/etc/hosts` để có thể phân giải theo hostname.

```
$ cat >> /etc/hosts
172.17.0.64  web


# Ctrl + c to exit
```
- Lúc này hệ thống sẽ tra cứu trong file `/etc/hosts`.

```
$ ping web
PING web (172.17.0.64) 56(84) bytes of data.
64 bytes from web (172.17.0.64): icmp_seq=1 ttl=64 time=0.491 ms
64 bytes from web (172.17.0.64): icmp_seq=2 ttl=64 time=0.636 ms

$ ssh web

$ curl http://web
```

> 💡 **Bổ sung:** Host tin tưởng hoàn toàn nội dung `/etc/hosts` mà không kiểm chứng — bạn có thể ánh xạ bất kỳ tên nào tới bất kỳ IP nào. Cách này chỉ phù hợp với vài máy; khi số máy tăng lên, việc cập nhật `/etc/hosts` trên từng host trở nên khó quản lý, nên người ta chuyển sang dùng một DNS server tập trung.

## DNS

- Mỗi host có một file cấu hình phân giải DNS tại `/etc/resolv.conf`.

```
$ cat /etc/resolv.conf
nameserver 127.0.0.53
options edns0
```

- Để thay đổi thứ tự phân giải tên (file `/etc/hosts` trước hay DNS trước), cần chỉnh file `/etc/nsswitch.conf`.

```
$ cat /etc/nsswitch.conf

hosts:          files dns
networks:       files

```

> 💡 **Bổ sung:** `hosts: files dns` nghĩa là tra `/etc/hosts` (files) trước, nếu không thấy mới hỏi DNS server. Vì vậy nếu một tên có ở cả hai nơi, giá trị trong `/etc/hosts` sẽ thắng. `nameserver 127.0.0.53` là stub resolver cục bộ của `systemd-resolved` trên Ubuntu; xem DNS server thực sự đang dùng bằng `resolvectl status`.

- Nếu phân giải thất bại trong một số trường hợp (ví dụ chưa có DNS server nào biết tên đó).

```
$ ping wwww.github.com
ping: www.github.com: Temporary failure in name resolution

```

- Thêm một nameserver công cộng phổ biến vào file `/etc/resolv.conf`.

```
$ cat /etc/resolv.conf
nameserver   127.0.0.53
nameserver   8.8.8.8
options edns0
``` 
```
$ ping www.github.com
PING github.com (140.82.121.3) 56(84) bytes of data.
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=1 ttl=57 time=7.07 ms
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=2 ttl=57 time=5.42 ms

```

> 💡 **Bổ sung:** Thông thường bạn cấu hình DNS server nội bộ để chuyển tiếp (forward) các tên không biết ra nameserver công cộng như `8.8.8.8`, thay vì thêm vào từng host. Trên các hệ thống dùng `systemd-resolved` hoặc NetworkManager, `/etc/resolv.conf` thường được tự sinh ra và có thể bị ghi đè — hãy cấu hình qua công cụ tương ứng.

## Tên miền (Domain Names)

![net-8](../../images/net8.PNG)

> 💡 **Bổ sung:** Tên miền có cấu trúc phân cấp, đọc từ phải sang trái: `.` (root) → top-level domain (`.com`, `.net`, `.io`...) → tên miền (`google`) → subdomain (`www`, `mail`, `maps`...). Khi phân giải, DNS server nội bộ sẽ hỏi lần lượt root server → server của `.com` → server của `google.com`, rồi cache kết quả trong một khoảng thời gian (TTL).

## Search Domain

![net-9](../../images/net9.PNG)

> 💡 **Bổ sung:** Thêm dòng `search mycompany.com` vào `/etc/resolv.conf` cho phép bạn gõ tên ngắn `web` và hệ thống tự thử `web.mycompany.com`. Kubernetes dùng chính cơ chế này: `/etc/resolv.conf` trong Pod có dòng `search <namespace>.svc.cluster.local svc.cluster.local cluster.local`, nhờ vậy bạn có thể gọi Service chỉ bằng tên ngắn.

## Các loại Record (Record Types)

![net-10](../../images/net10.PNG)

> 💡 **Bổ sung:** `A` ánh xạ tên → IPv4, `AAAA` ánh xạ tên → IPv6, `CNAME` ánh xạ tên → một tên khác (alias). Ngoài ra còn hay gặp `PTR` (tra ngược IP → tên), `SRV` (dùng cho service discovery, Kubernetes có tạo SRV record cho các named port của Service), `MX`, `TXT`, `NS`.

## Công cụ mạng (Networking Tools)

- Một số công cụ hữu ích để kiểm tra phân giải tên DNS.

> 💡 **Bổ sung:** `nslookup` và `dig` **chỉ truy vấn DNS server**, chúng không đọc file `/etc/hosts`. Vì vậy một tên có trong `/etc/hosts` có thể `ping` được nhưng `nslookup` lại không tìm thấy. Nếu muốn kiểm tra theo đúng thứ tự trong `nsswitch.conf`, dùng `getent hosts <tên>`.

#### nslookup 

```
$ nslookup www.google.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   www.google.com
Address: 172.217.18.4
Name:   www.google.com
```

#### dig

```
$ dig www.google.com

; <<>> DiG 9.11.3-1 ...
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8738
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; ANSWER SECTION:
www.google.com.         63      IN      A       216.58.206.4

;; Query time: 6 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
```
