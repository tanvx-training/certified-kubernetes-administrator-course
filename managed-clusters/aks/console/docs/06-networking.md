# Networking

Bây giờ chuyển sang phần networking

![image](../images/06-networking.png)

Ở đây chúng ta phải đặt một DNS prefix. DNS prefix phải là duy nhất trên toàn cầu, và bạn không thể biết liệu có ai khác đang làm theo hướng dẫn này cùng lúc với bạn hay không! Vì vậy chúng ta không thể dùng một tên cố định ở đây.

Chúng ta có thể tạo một tên đủ độc nhất bằng cách dùng `kodekloud-demo-` theo sau là vài chữ số ngẫu nhiên.

![image](../images/06a-networking.png)

Giữ nguyên mặc định cho tất cả các thiết lập khác.

> 💡 **Bổ sung:** DNS prefix được dùng để tạo FQDN (tên miền đầy đủ) cho API server của cluster, ví dụ `kodekloud-demo-12345-xxxx.hcp.eastus.azmk8s.io`. Đây chính là địa chỉ `server:` sẽ xuất hiện trong file kubeconfig của bạn.

Tiếp theo: [Monitoring](./07-monitoring.md)
