# Chuẩn bị node (Node Setup)

**LƯU Ý**<br/>
Trong phần này và các phần tiếp theo, bạn sẽ thấy một số nhóm lệnh shell được bao trong cặp `{ }`. Mục đích là để bạn có thể copy cả khối lệnh từ github bằng nút copy của github rồi dán vào terminal của node. Nếu không có cặp ngoặc này, khi dán một nhóm nhiều lệnh có `sudo`, shell sẽ dừng lại sau lệnh đầu tiên và nhiều khả năng bạn không để ý, dẫn tới các bước sau không hoạt động.

---

Trong phần này, chúng ta sẽ cấu hình các node và cài đặt các thành phần cần thiết như container runtime (`containerd`).

Thực hiện tất cả các bước dưới đây trên từng node `controlplane01`, `controlplane02`, `controlplane03`, `node01` và `node02`.

[//]: # (host:controlplane-node01-node02)

1. Cập nhật danh mục gói apt và cài các gói cần thiết để sử dụng apt repository của Kubernetes:
    ```bash
    {
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl
    }
    ```

1. Nạp các kernel module cần thiết và cấu hình để chúng được nạp lại sau mỗi lần khởi động
    ```bash
    {
        cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

        sudo modprobe overlay
        sudo modprobe br_netfilter
    }
    ```

    > 💡 **Bổ sung:** `overlay` là module cho filesystem OverlayFS mà containerd dùng để xếp chồng các layer của image thành filesystem của container. `br_netfilter` cho phép traffic đi qua Linux bridge (nơi các Pod được nối vào) được xử lý bởi iptables — thiếu module này thì các rule iptables do kube-proxy tạo cho Service sẽ không áp dụng với traffic giữa các Pod trên cùng node. File trong `/etc/modules-load.d/` giúp module được nạp tự động khi reboot, còn `modprobe` nạp ngay lập tức.

1.  Thiết lập các tham số kernel cần thiết và lưu cố định
    ```bash
    {
        cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

        sudo sysctl --system
    }
    ```

    > 💡 **Bổ sung:** `net.bridge.bridge-nf-call-iptables`/`ip6tables = 1` bật việc chuyển traffic của bridge qua iptables (chỉ có hiệu lực khi đã nạp `br_netfilter`). `net.ipv4.ip_forward = 1` cho phép node chuyển tiếp (route) gói tin giữa các network interface — bắt buộc để Pod trên node này nói chuyện được với Pod trên node khác và với Service. Nếu thiếu `ip_forward`, `kubeadm init`/`kubeadm join` sẽ báo lỗi ở bước preflight check. `sysctl --system` nạp lại toàn bộ cấu hình trong `/etc/sysctl.d/` mà không cần reboot.

1. Cài đặt container runtime
    ```bash
    sudo apt-get install -y containerd
    ```

1.  Cấu hình container runtime sử dụng systemd Cgroups. Đây là bước rất nhiều học viên bỏ sót, và nếu không làm thì control plane sẽ khởi động lên được, nhưng sau đó tất cả các Pod bắt đầu bị crashloop. `kubectl` cũng sẽ lỗi với thông báo kiểu `The connection to the server x.x.x.x:6443 was refused - did you specify the right host or port?`

    > 💡 **Bổ sung:** cgroup là cơ chế của Linux để giới hạn tài nguyên (CPU, memory) cho tiến trình. Trên hệ điều hành dùng systemd, kubelet (từ v1.22 kubeadm mặc định cấu hình kubelet dùng `cgroupDriver: systemd`) và container runtime **phải dùng cùng một cgroup driver**. Nếu containerd vẫn dùng driver mặc định `cgroupfs` trong khi kubelet dùng `systemd`, sẽ có hai trình quản lý cgroup cùng lúc, gây xung đột tài nguyên — các static Pod của control plane (kube-apiserver, etcd...) liên tục bị khởi động lại. Lệnh `sed` dưới đây đổi `SystemdCgroup = false` thành `true` trong cấu hình runc của containerd.

    1. Tạo file cấu hình mặc định

        ```bash
        {
            sudo mkdir -p /etc/containerd
            containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
        }
        ```

    1.  Khởi động lại containerd

        ```bash
        sudo systemctl restart containerd
        ```

1.  Xác định phiên bản Kubernetes mới nhất và lưu vào một biến shell

    ```bash
    KUBE_LATEST=$(curl -L -s https://dl.k8s.io/release/stable.txt | awk 'BEGIN { FS="." } { printf "%s.%s", $1, $2 }')
    ```

1. Tải signing key công khai của Kubernetes
    ```bash
    {
        sudo mkdir -p /etc/apt/keyrings
        curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    }
    ```

1. Thêm apt repository của Kubernetes
    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

    > 💡 **Bổ sung:** Repository `pkgs.k8s.io` được tổ chức theo từng minor version (vd `v1.31`). Repo cũ `apt.kubernetes.io`/`packages.cloud.google.com` đã ngừng hoạt động từ năm 2024. Vì mỗi repo chỉ chứa một minor version, khi upgrade cluster lên minor version mới (vd 1.31 → 1.32), bạn phải sửa file `/etc/apt/sources.list.d/kubernetes.list` sang repo của version mới trước rồi mới `apt-get update` — đây là điểm hay gặp trong bài upgrade của đề CKA.

1. Cập nhật danh mục gói apt, cài kubelet, kubeadm và kubectl, rồi ghim (pin) phiên bản của chúng
    ```bash
    {
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
    }
    ```

    > 💡 **Bổ sung:** `apt-mark hold` ngăn các gói này bị nâng cấp ngoài ý muốn khi chạy `apt-get upgrade`. Việc upgrade Kubernetes phải được làm có kiểm soát theo đúng trình tự (control plane trước, worker sau), nên khi cần upgrade ta sẽ `apt-mark unhold`, cài version mới rồi `hold` lại.

    > 💡 **Bổ sung — về swap:** Hướng dẫn này không có bước tắt swap vì các image VM dùng trong lab mặc định không bật swap. Theo truyền thống, kubelet không khởi động nếu node đang bật swap (preflight check của kubeadm sẽ báo lỗi), vì swap làm sai lệch việc tính toán memory requests/limits và hành vi OOM của Pod. Từ v1.28 kubelet đã hỗ trợ swap ở mức beta nhưng vẫn cần cấu hình riêng. Nếu môi trường của bạn có swap, hãy tắt nó trước khi chạy kubeadm: `sudo swapoff -a` và comment dòng swap trong `/etc/fstab` để không bị bật lại sau khi reboot. Kiểm tra bằng `free -h` hoặc `swapon --show`.

1.  Cấu hình `crictl` để dùng khi cần kiểm tra các container đang chạy
    ```bash
    sudo crictl config \
        --set runtime-endpoint=unix:///run/containerd/containerd.sock \
        --set image-endpoint=unix:///run/containerd/containerd.sock
    ```

1.  Chuẩn bị tham số bổ sung cho kubelet để khi khởi động, nó lắng nghe trên địa chỉ mạng chính của VM chứ không phải một địa chỉ NAT có thể tồn tại. Bước này dùng biến môi trường `PRIMARY_IP` đã được định nghĩa sẵn như đã nói ở phần trước.

    Dán đoạn sau vào terminal

    ```bash
    cat <<EOF | sudo tee /etc/default/kubelet
    KUBELET_EXTRA_ARGS='--node-ip ${PRIMARY_IP}'
    EOF
    ```

    > 💡 **Bổ sung:** Trong một số môi trường (đặc biệt VirtualBox/Multipass), VM có nhiều network interface (vd một interface NAT để ra Internet có cùng IP trên mọi VM). Nếu kubelet tự chọn nhầm interface đó, `kubectl get nodes -o wide` sẽ hiện INTERNAL-IP sai và `kubectl logs`/`exec` tới Pod trên node đó sẽ lỗi. Vì vậy ta chỉ định rõ `--node-ip`.

Tiếp theo: [Thiết lập Control Plane](./06-controlplane.md)</br>
Trước đó: [Thiết lập Load Balancer](./04-loadbalancer.md)
