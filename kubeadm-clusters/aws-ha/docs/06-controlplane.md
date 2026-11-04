# Khởi động control plane

Để tạo một control plane có tính sẵn sàng cao, chúng ta cài kubeadm trên node control plane đầu tiên gần giống hệt như với cluster chỉ có một control plane, sau đó *join* các node control plane còn lại theo cách tương tự như join worker node.

## controlplane01

1.  ssh tới `controlplane01`

    ```bash
    ssh controlplane01
    ```

1. Chuyển sang root
    ```bash
    sudo -i
    ```

1.  Đặt biến shell cho dải CIDR của mạng Pod.

    ```bash
    POD_CIDR=192.168.0.0/16
    ```

    > 💡 **Bổ sung:** Ở đây dùng được `192.168.0.0/16` (giá trị mặc định trong `custom-resources.yaml` của Calico) vì default VPC của AWS dùng dải `172.31.0.0/16`, không trùng với dải Pod. Khác với bản VirtualBox/Apple Silicon, nơi dải này dễ trùng với mạng gia đình.

1. Khởi động control plane đầu tiên, dùng địa chỉ IP của load balancer làm control plane endpoint

    Đặt một biến shell bằng IP của loadbalancer

    ```bash
    LOADBALANCER=$(dig +short loadbalancer)
    ```

    ...rồi cài Kubernetes với `--control-plane-endpoint` và `--upload-certs`, hai tham số này báo cho kubeadm biết chúng ta đang dựng cluster có nhiều node control plane.

    ```bash
    kubeadm init --pod-network-cidr $POD_CIDR \
        --control-plane-endpoint ${LOADBALANCER}:6443 \
        --upload-certs
    ```

    Copy cả hai lệnh join được in ra vào notepad để dùng trên các node control plane khác và các worker node.

    > 💡 **Bổ sung:**
    > * `--control-plane-endpoint` đặt địa chỉ chung (ở đây là load balancer) mà mọi kubeconfig và mọi node sẽ dùng để gọi API server, thay vì IP của riêng `controlplane01`. Tham số này phải được đặt ngay từ đầu — rất khó chuyển một cluster một control plane sang HA nếu ban đầu không có nó.
    > * `--upload-certs` tải các certificate dùng chung của control plane (CA, front-proxy CA, khóa ký ServiceAccount, CA của etcd) lên Secret `kubeadm-certs` trong namespace `kube-system`, được mã hóa bằng một certificate key. Các control plane khác join với `--control-plane --certificate-key <key>` sẽ tự tải về các certificate này. Secret và key này tự bị xóa sau 2 giờ; nếu quá hạn, tạo lại bằng `kubeadm init phase upload-certs --upload-certs` trên `controlplane01`.
    > * Lệnh join cho control plane khác lệnh join cho worker ở chỗ có thêm `--control-plane --certificate-key ...`.

1. Cài network plugin (calico). Weave không hoạt động tốt lắm với cluster HA.
    ```bash
    kubectl --kubeconfig /etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/tigera-operator.yaml
    kubectl --kubeconfig /etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/custom-resources.yaml
    ```

1.  Kiểm tra xem mọi thứ đã chạy chưa

    ```bash
    kubectl --kubeconfig /etc/kubernetes/admin.conf get pods -n kube-system
    ```

1.  Thoát khỏi root shell

    ```bash
    exit
    ```

1.  Chuẩn bị file kubeconfig để copy sang node `student-node`, nơi chúng ta sẽ chạy các lệnh `kubectl` từ nay về sau.

    ```bash
    {
      sudo cp /etc/kubernetes/admin.conf .
      sudo chmod 666 admin.conf
    }
    ```

1.  Thoát về `student-node`

    ```bash
    exit
    ```

1. Trên `student-node`, copy kubeconfig về để có thể chạy lệnh kubectl từ `student-node`

    ```bash
    {
        mkdir ~/.kube
        scp controlplane01:~/admin.conf ~/.kube/config
    }
    ```

## controlplane02 và controlplane03

Đứng ở `student-node`

Với từng node `controlplane02` và `controlplane03`

1.  SSH tới `controlplane02`
1.  Chuyển sang root

    ```bash
    sudo -i
    ```
1.  Dán lệnh join dành cho các node *control plane* mà `kubeadm init` đã in ra trên `controlplane01`
1.  Thoát về `student-node`
    ```bash
    exit
    exit
    ```
1. Lặp lại các bước 2, 3 và 4 trên `controlplane03`

> 💡 **Bổ sung:** Sau khi join xong, bạn có thể kiểm tra etcd đã có đủ 3 thành viên bằng `kubectl get pods -n kube-system -l component=etcd -o wide` (mỗi control plane có một Pod `etcd-controlplane0X`).

Tiếp theo: [Thiết lập worker](./07-workers.md)</br>
Trước đó: [Chuẩn bị node](./05-node-setup.md)


