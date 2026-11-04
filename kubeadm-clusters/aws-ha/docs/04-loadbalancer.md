# Cấu hình load balancer

Bây giờ chúng ta sẽ cài load balancer đóng vai trò endpoint để kết nối tới API server. Nó sẽ phân phối các request tới API server lần lượt (round-robin) giữa các node control plane. Để làm việc này, chúng ta dùng [HAProxy](https://haproxy.org/) ở chế độ cân bằng tải TCP. Ở chế độ này, nó chỉ đơn giản chuyển tiếp toàn bộ traffic tới các back end (các control plane) mà không thay đổi gì, ví dụ không thực hiện SSL termination.

> 💡 **Bổ sung:** Phải dùng chế độ TCP (layer 4) vì kubectl, kubelet và các thành phần khác xác thực với kube-apiserver bằng client certificate (mutual TLS). Nếu load balancer giải mã TLS (SSL termination) thì client certificate sẽ không tới được API server. Ngoài ra, certificate của API server phải chứa địa chỉ của load balancer trong SAN — `kubeadm init` với `--control-plane-endpoint` sẽ tự lo việc này.

Trước tiên, hãy đăng nhập vào `student-node` như hướng dẫn ở trên.

1.  Đăng nhập vào load balancer

    ```
    ssh loadbalancer
    ```

1. Chuyển sang root (đỡ phải gõ `sudo` trước mỗi lệnh)

    ```bash
    sudo -i
    ```

1. Cập nhật danh mục gói apt và cài các gói cần cho HAProxy:

    ```bash
    apt-get update
    apt-get install -y haproxy
    ```

1.  Dùng lệnh [dig](https://linux.die.net/man/1/dig) — một lựa chọn thay thế cho `nslookup` và tiện hơn khi viết script — chúng ta có thể lấy địa chỉ IP *private* của loadbalancer và 3 control plane.

    ```bash
    dig +short loadbalancer
    dig +short controlplane01
    dig +short controlplane02
    dig +short controlplane03
    ```

    **Thuật ngữ**

    Về cơ bản, EC2 instance của AWS có hai địa chỉ IP:
    1. Private IP - Đây là IP trên subnet AWS nơi instance được khởi chạy, và được dùng để các EC2 instance nói chuyện với nhau.
    1. Public IP - Không phải lúc nào cũng được gán, đây là IP dùng để truy cập EC2 instance từ thế giới bên ngoài (tức là từ trình duyệt của bạn). Chúng ta sẽ thấy nó ở phần kiểm thử sau.

1.  Tạo file cấu hình HAProxy

    ```bash
    cat <<EOF > /etc/haproxy/haproxy.cfg
    frontend kubernetes
        bind $(dig +short loadbalancer):6443
        option tcplog
        mode tcp
        default_backend kubernetes-control-nodes

    backend kubernetes-control-nodes
        mode tcp
        balance roundrobin
        option tcp-check
        server controlplane01 $(dig +short controlplane01):6443 check fall 3 rise 2
        server controlplane02 $(dig +short controlplane02):6443 check fall 3 rise 2
        server controlplane03 $(dig +short controlplane03):6443 check fall 3 rise 2
    EOF
    ```

    > 💡 **Bổ sung:** `check fall 3 rise 2` nghĩa là HAProxy liên tục kiểm tra kết nối TCP tới port 6443 của từng control plane; một backend bị coi là chết sau 3 lần kiểm tra thất bại liên tiếp và được đưa trở lại sau 2 lần thành công. Nhờ vậy khi một control plane gặp sự cố, traffic tự động chuyển sang các node còn lại.

1.  Khởi động lại và kiểm tra haproxy

    ```bash
    systemctl restart haproxy
    systemctl status haproxy
    ```

    Nó sẽ cảnh báo rằng không có backend nào khả dụng — điều này đúng vì chúng ta chưa cài Kubernetes!

1.  Thoát khỏi `sudo` rồi quay về `student-node`

    ```bash
    exit
    exit
    ```

Tiếp theo: [Chuẩn bị node](./05-node-setup.md)<br>
Trước đó: [Kết nối](./03-connectivity.md)
