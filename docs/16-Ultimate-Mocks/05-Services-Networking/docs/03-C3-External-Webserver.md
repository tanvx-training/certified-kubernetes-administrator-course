# Cluster 3 - External Webserver

LƯU Ý: Câu hỏi này cũng có trong Ultimate CKAD Mocks. Ở đó tên Service là `external-webserver-ckad01-svcn`, nhưng lời giải hoàn toàn giống nhau. Nếu bạn đang làm phiên bản CKAD, hãy thay `external-webserver-cka03-svcn` bằng `external-webserver-ckad01-svcn` ở mọi chỗ.

Với câu hỏi này, hãy chuyển context sang cluster3 bằng lệnh:

```
kubectl config use-context cluster3
```


Chúng ta có một webserver **bên ngoài** cluster đang chạy trên `student-node` và được expose ở port `9999`. Chúng ta đã tạo một Service tên là `external-webserver-cka03-svcn` để kết nối tới webserver này từ bên trong cluster3 của Kubernetes, nhưng hiện tại nó không hoạt động như mong đợi.

Hãy sửa lỗi để các Pod khác trong cluster3 có thể dùng Service `external-webserver-cka03-svcn` để truy cập webserver.

---

Đề bài cho biết ta cần nối Service tới một web server đang chạy trên `student-node` ở port `9999`. Hãy kiểm tra điều này. Trên student node, chạy lệnh sau

```
curl localhost:9999
```

> Output

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Đúng rồi, nó đang chạy!

Điều quan trọng cần lưu ý là web server này nằm *bên ngoài* cluster, do đó Service sẽ phải kết nối tới một IP không thuộc cluster – thực chất là địa chỉ IP chính của `student-node`. Hãy tìm IP đó bằng cách chạy lệnh sau trên `student-node`:

```
ifconfig
```

> Output (lưu ý: giá trị bạn nhận được cho từng interface gần như chắc chắn sẽ khác)

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 192.37.66.3  netmask 255.255.255.0  broadcast 192.37.66.255
        ether 02:42:c0:25:42:03  txqueuelen 0  (Ethernet)
        RX packets 2179  bytes 713082 (713.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2361  bytes 391620 (391.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.25.0.103  netmask 255.255.255.0  broadcast 172.25.0.255
        ether 02:42:ac:19:00:67  txqueuelen 0  (Ethernet)
        RX packets 36  bytes 8457 (8.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14  bytes 1593 (1.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 161  bytes 14395 (14.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 161  bytes 14395 (14.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Interface chính là `eth0`. Ghi lại giá trị `inet` của interface này, trong ví dụ là `192.37.66.3`. Đây là IP mà Service cần kết nối tới.

> 💡 **Bổ sung:** Nếu máy không có `ifconfig`, dùng `ip addr show eth0` (hoặc `hostname -I`) để thay thế.

Nếu bây giờ chạy `kubectl get service` cho `external-webserver-cka03-svcn` (hoặc `kubectl describe service`), ta thấy nó không có pod selector và do đó không có endpoint nào. Vì vậy nó "không hoạt động như mong đợi" do không có endpoint.

Để nối Service tới IP bên ngoài, ta phải tự tạo endpoint cho Service. Lưu ý rằng tên của Endpoints (`metadata.name`) *phải khớp chính xác* với tên Service mà bạn muốn gắn vào. Dưới đây là manifest Endpoints, kèm comment giải thích từng phần.

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: external-webserver-cka03-svcn  # <- Phải trùng tên với Service cần gắn vào
  namespace: default
subsets:
  - addresses:
      - ip: 192.37.66.3  # <- Lấy từ ifconfig
    ports:
      - port: 9999       # <- Cho trong đề bài
```

Lưu nội dung này vào một file rồi `kubectl apply` nó.

> 💡 **Bổ sung:**
> - Nếu port trong Service có đặt tên (`spec.ports[].name`), thì port trong Endpoints cũng phải có `name` giống hệt, nếu không kube-proxy sẽ không ghép được. Kiểm tra bằng `kubectl get svc external-webserver-cka03-svcn -o yaml`.
> - Service phải **không có** selector; nếu có selector, endpoints controller sẽ quản lý và ghi đè Endpoints bạn tạo thủ công.
> - Từ Kubernetes v1.33, API `Endpoints` bị đánh dấu deprecated (vẫn hoạt động, chỉ hiện cảnh báo) để khuyến khích dùng `EndpointSlice` (`discovery.k8s.io/v1`). Với EndpointSlice, tên tùy ý nhưng phải gắn label `kubernetes.io/service-name: external-webserver-cka03-svcn` và khai báo `addressType: IPv4`, `endpoints[].addresses`, `ports[]`. Trong đề thi, cách nào cũng được trừ khi đề yêu cầu cụ thể.
> - Kiểm tra: `kubectl get endpoints external-webserver-cka03-svcn` phải hiển thị `192.37.66.3:9999`.
> - Một cách khác khi đích bên ngoài có tên DNS (không phải IP) là dùng Service kiểu `ExternalName`.

Giờ hãy kiểm tra bằng một Pod `wbitt/network-multitool` có sẵn curl để gọi tới Service.<br/> MẸO - hãy nhớ image này! Nó chứa nhiều công cụ mạng và DNS phổ biến, rất hữu ích khi troubleshooting - và đúng vậy, có thể dùng nó trong kỳ thi thật.

```
k run test-pod --image wbitt/network-multitool --restart Never -it -- curl external-webserver-cka03-svcn.default.svc
```

> Output

```
The directory /usr/share/nginx/html is not mounted.
Therefore, over-writing the default index.html file with some useful information:
WBITT Network MultiTool (with NGINX) - test-pod - 10.42.0.13 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Đã nhận được phản hồi - THÀNH CÔNG!

Như vậy, ta đã cấu hình một Service ClusterIP cho phép các Pod *bên trong* cluster nói chuyện với một dịch vụ *bên ngoài* cluster, thông qua một endpoint được khai báo thủ công trỏ tới một IP bên ngoài. `kube-proxy` lo phần định tuyến cho chúng ta.

> 💡 **Bổ sung:** Lệnh test ở trên thiếu `--rm` nên Pod `test-pod` sẽ còn lại sau khi chạy. Nên thêm `--rm` (`k run test-pod --image wbitt/network-multitool --restart Never --rm -it -- curl ...`) hoặc xóa Pod sau khi test để không để lại resource thừa trong cluster.