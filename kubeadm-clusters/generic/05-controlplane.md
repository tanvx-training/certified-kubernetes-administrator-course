# Khởi động control plane

[//]: # (host:controlplane)



Thực hiện trên node `controlplane`

1.  Đặt các biến shell cho dải CIDR của Pod và Service. Địa chỉ advertise của API server dùng biến đã được định nghĩa sẵn như mô tả ở [phần trước](./04-node-setup.md). Lưu ý rằng với máy ảo dùng chế độ bridged (mặc định của bản cài đặt này), điều quan trọng là cả hai dải IP dưới đây đều không được trùng với mạng gia đình của bạn — với hầu hết cấu hình mặc định của router băng rộng thì mạng này là `192.168.0.0/24`.

    ```bash
    POD_CIDR=10.244.0.0/16
    SERVICE_CIDR=10.96.0.0/16
    ```

    > 💡 **Bổ sung:** Có ba dải địa chỉ phải tách biệt hoàn toàn: dải IP của các node (mạng VM), dải IP của Pod (`--pod-network-cidr`, do CNI cấp phát cho từng Pod) và dải IP của Service (`--service-cidr`, các ClusterIP ảo do kube-proxy xử lý). Nếu chúng chồng lấn nhau, việc định tuyến sẽ bị sai và cluster hoạt động chập chờn hoặc không chạy được.

1.  Khởi động control plane

    Ở đây chúng ta truyền tham số cho `kubeadm` để đảm bảo nó dùng đúng dải mạng và địa chỉ IP mong muốn, thay vì tự chọn giá trị mặc định có thể không đúng.

    ```bash
    sudo kubeadm init --pod-network-cidr $POD_CIDR --service-cidr $SERVICE_CIDR --apiserver-advertise-address $PRIMARY_IP
    ```

    > 💡 **Bổ sung:** `kubeadm init` sẽ chạy preflight check, sinh các certificate trong `/etc/kubernetes/pki`, tạo các file kubeconfig (`admin.conf`, `kubelet.conf`, `controller-manager.conf`, `scheduler.conf`), ghi manifest static Pod của kube-apiserver, kube-controller-manager, kube-scheduler và etcd vào `/etc/kubernetes/manifests`, rồi cài các addon CoreDNS và kube-proxy. Nếu init thất bại và muốn làm lại, chạy `sudo kubeadm reset` trước.

[//]: # (command:sleep 10)

1.  Copy lệnh `kubeadm join` được in ra vào notepad để dùng trên các worker node.

    > 💡 **Bổ sung:** Token trong lệnh join mặc định hết hạn sau 24 giờ. Nếu bị mất hoặc token đã hết hạn, bạn có thể tạo lệnh join mới trên control plane bằng `kubeadm token create --print-join-command`.

1.  Thiết lập kubeconfig để user đang đăng nhập có thể sử dụng

    1.  Tạo thư mục cho kubeconfig, copy kubeconfig admin làm kubeconfig mặc định cho user hiện tại (`vagrant` trên VirtualBox, `ubuntu` trên Multipass và AWS) và đặt quyền file cho đúng.</br>Lưu ý: trên Linux, lệnh `id -u` trả về user ID (UID) và `id -g` trả về group ID (GID) của bạn trên hệ thống.

        ```bash
        {
            mkdir ~/.kube
            sudo cp /etc/kubernetes/admin.conf ~/.kube/config
            sudo chown $(id -u):$(id -g) ~/.kube/config
            chmod 600 ~/.kube/config
        }
        ```

    1.  Kiểm tra

        ```bash
        kubectl get pods -n kube-system
        ```

        > 💡 **Bổ sung:** Ở thời điểm này các Pod `coredns` sẽ ở trạng thái `Pending` và node ở trạng thái `NotReady` — đây là bình thường vì chưa cài CNI. Chúng sẽ chuyển sang `Running`/`Ready` sau bước tiếp theo.

1.  Cài Calico CNI cho mạng của cluster

    Calico là một CNI có hỗ trợ Network Policy. Một CNI thay thế được nhắc tới trong khóa học là Flannel, tuy nhiên Flannel không hỗ trợ Network Policy, nó chỉ cung cấp mạng Pod cơ bản.

    1. Cài Tigera operator và các custom resource definition (CRD).

        ```bash
        kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.1/manifests/tigera-operator.yaml
        ```

    1. Cài Calico bằng cách tạo các custom resource cần thiết. Nếu máy ảo dùng mạng bridge (mặc định), chúng ta phải sửa các custom resource này cho khớp với `POD_CIDR` đã đặt ở trên, nếu không mạng Pod sẽ trùng với mạng gia đình của bạn và cluster sẽ không hoạt động.

        1. Tải manifest custom resource

            ```bash
            curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.30.1/manifests/custom-resources.yaml
            ```

        1. Dùng `sed` để đổi dải mạng ipPool trong manifest từ giá trị mặc định `192.168.0.0/16` sang giá trị `POD_CIDR` đã đặt ở trên. Bạn có thể sửa bằng `vi` nhưng dùng `sed` nhanh hơn.

            ```bash
            sed -i "s#192.168.0.0/16#$POD_CIDR#" custom-resources.yaml
            ```

        1. Apply manifest đã sửa

            ```bash
            kubectl apply -f custom-resources.yaml
            ```

    Sẽ mất một hai phút để tất cả các resource của calico khởi chạy xong. Bạn có thể theo dõi tiến trình bằng lệnh sau

    ```
    watch kubectl get tigerastatus
    ```

    Đợi đến khi tất cả các mục trong danh sách có trạng thái `True` ở cột available, sau đó nhấn `CTRL-C` để thoát lệnh `watch`.

    [Tài liệu cài đặt Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

[//]: # (command:sleep 120)

1.  Kiểm tra control plane

    ```bash
    kubectl get pods -n kube-system
    ```

[Link tài liệu Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

Tiếp theo: [Join các worker](./06-workers.md)</br>
Trước đó: [Chuẩn bị node](./04-node-setup.md)
