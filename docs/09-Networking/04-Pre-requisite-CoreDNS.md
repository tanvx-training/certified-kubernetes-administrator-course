# Kiến thức nền tảng: CoreDNS

  - Đi tới [Lecture](https://kodekloud.com/topic/prerequisite-coredns/)

Trong phần này, chúng ta sẽ tìm hiểu **CoreDNS**

> 💡 **Bổ sung:** CoreDNS là DNS server mã nguồn mở viết bằng Go, hoạt động theo kiến trúc plugin (mỗi tính năng như `hosts`, `forward`, `cache`, `kubernetes`... là một plugin). CoreDNS là dự án graduated của CNCF và là DNS server mặc định của Kubernetes (thay thế kube-dns từ v1.13). Phần dưới đây cài CoreDNS như một DNS server độc lập trên Linux để hiểu cách nó hoạt động.

## Cài đặt CoreDNS

```
$ wget https://github.com/coredns/coredns/releases/download/v1.7.0/coredns_1.7.0_linux_amd64.tgz
coredns_1.7.0_linux_amd64.tgz

```

## Giải nén file tar

```
$ tar -xzvf coredns_1.7.0_linux_amd64.tgz
coredns
```

## Chạy file thực thi

- Chạy file thực thi để khởi động DNS server. Mặc định nó lắng nghe trên port 53 — port mặc định của DNS server.

```
$ ./coredns

```

## Cấu hình file hosts

- Thêm các entry vào file `/etc/hosts`.
- CoreDNS sẽ lấy danh sách IP và tên từ file `/etc/hosts` trên server.

```
$ cat > /etc/hosts
192.168.1.10    web
192.168.1.11    db
192.168.1.15    web-1
192.168.1.16    db-1
192.168.1.21    web-2
192.168.1.22    db-2
```

## Thêm cấu hình vào Corefile

```
$ cat > Corefile
. {
	hosts   /etc/hosts
}

```

> 💡 **Bổ sung:** `Corefile` là file cấu hình của CoreDNS. Khối `. { ... }` áp dụng cho mọi zone (root `.`), và plugin `hosts` chỉ định CoreDNS trả lời dựa trên file `/etc/hosts`. Trong thực tế bạn thường thêm `forward . 8.8.8.8` để chuyển các truy vấn không có trong file ra DNS server bên ngoài, và `cache` để cache kết quả. Trong Kubernetes, Corefile được lưu trong ConfigMap `coredns` ở namespace `kube-system` và dùng plugin `kubernetes` (sẽ học ở phần CoreDNS trong Kubernetes).

## Chạy lại file thực thi

```
$ ./coredns

```


#### Tài liệu tham khảo

- https://github.com/kubernetes/dns/blob/master/docs/specification.md
- https://coredns.io/plugins/kubernetes/
- https://github.com/coredns/coredns/releases
