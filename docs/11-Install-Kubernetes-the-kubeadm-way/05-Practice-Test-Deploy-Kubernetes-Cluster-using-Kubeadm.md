# Practice Test - Cài đặt Kubernetes cluster bằng công cụ kubeadm (Bài thực hành)

Nếu bạn muốn tự dựng cluster của riêng mình, hãy xem các hướng dẫn sau:

* [Kubeadm Clusters](../../kubeadm-clusters/)
* [Managed Clusters](../../managed-clusters/)

# Lời giải cho bài practice test của KodeKloud - Cài đặt bằng Kubeadm

- Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-deploy-a-kubernetes-cluster-using-kubeadm/)

Trong lab thực hành này, container runtime (containerd) đã được cài sẵn. Chúng ta chỉ cần tập trung vào phần cấu hình node còn lại, tức là thiết lập kernel module và tham số kernel, rồi cài đặt cluster bằng kubeadm.

Lưu ý rằng video trước lab này đã lỗi thời: video vẫn dùng container driver docker và một phiên bản Kubernetes cũ. Còn lab thì đã được cập nhật.

  1.  <details>
      <summary>Cài các gói kubeadm, kubelet và kubectl với đúng phiên bản 1.24.0 trên controlplane và node01.</summary>

      > 💡 **Lưu ý:** Phiên bản được yêu cầu có thể thay đổi theo từng lần cập nhật lab (lời giải bên dưới dùng 1.27.0). Hãy đọc kỹ đề và thay `v1.27` cũng như `1.27.0-2.1` cho đúng phiên bản được yêu cầu.

      Chạy hai bước sau trên cả `controlplane` và `node01` (dùng `ssh node01` để vào worker node).

      1. Cấu hình các tham số kernel

         ```
         cat <<EOF | tee /etc/modules-load.d/k8s.conf
         overlay
         br_netfilter
         EOF

         cat <<EOF | tee /etc/sysctl.d/k8s.conf
         net.bridge.bridge-nf-call-ip6tables = 1
         net.bridge.bridge-nf-call-iptables = 1
         net.ipv4.ip_forward = 1
         EOF

         sysctl --system
         ```

      2. Cài các binary của Kubernetes

         Lưu ý: vì bạn đăng nhập vào lab với quyền `root` nên không cần dùng `sudo`.

         ```
         apt-get update
         apt-get install -y apt-transport-https ca-certificates curl

         mkdir -m 755 /etc/apt/keyrings

         curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

         echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list

         apt-get update

         # Xem các nhãn phiên bản mới
         apt-cache madison kubeadm

         apt-get install -y kubelet=1.27.0-2.1 kubeadm=1.27.0-2.1 kubectl=1.27.0-2.1

         apt-mark hold kubelet kubeadm kubectl
         ```

         > 💡 **Bổ sung:** Repository `pkgs.k8s.io` tách riêng theo từng minor version (`v1.27`, `v1.28`...), nên URL phải khớp với phiên bản cần cài. Lệnh `apt-cache madison kubeadm` liệt kê các phiên bản có sẵn kèm hậu tố gói (vd `1.27.0-2.1`), và bạn phải dùng đúng chuỗi đó khi chạy `apt-get install`. Nếu thư mục `/etc/apt/keyrings` đã tồn tại thì lệnh `mkdir` sẽ báo lỗi, có thể bỏ qua lỗi này hoặc dùng `mkdir -p`.
      </details>

  1.  <details>
      <summary>Phiên bản kubelet đã được cài là bao nhiêu?</summary>

      ```
      kubelet version
      ```

      Bạn sẽ nhận được thông báo lỗi vì cluster chưa được cài đặt, nhưng thông báo đó vẫn cho biết phiên bản của kubelet.

      > 💡 **Bổ sung:** Cách chuẩn hơn là `kubelet --version`, lệnh này in trực tiếp phiên bản (vd `Kubernetes v1.27.0`) mà không báo lỗi.

      </details>

  1.  <details>
      <summary>Hiện có bao nhiêu node thuộc Kubernetes cluster?</summary>

      Bạn có chạy được `kubectl get nodes` không? Bạn đã chạy `kubeadm init` chưa?

      Chưa, nên chưa có node nào.

      > 0

      </details>

  1.  Chỉ mang tính thông tin

  1.  <details>
      <summary>Khởi tạo Control Plane Node (Master Node).</summary>

      Sử dụng các tùy chọn sau:

      * `apiserver-advertise-address` - Dùng địa chỉ IP được cấp cho eth0 trên controlplane node
      * `apiserver-cert-extra-sans` - Đặt giá trị là `controlplane`
      * `pod-network-cidr` - Đặt giá trị là `10.244.0.0/16`

      Sau khi xong, thiết lập file kubeconfig mặc định và chờ node trở thành một phần của cluster.

      1. Lấy địa chỉ IP của adapter `eth0` trên controlplane

         ```
         ifconfig eth0
         ```

         Lấy giá trị in ra ở mục `inet` trong output. Giá trị sẽ có dạng như bên dưới, nhưng có thể khác nhau mỗi lần bạn chạy lab.

         > 💡 **Bổ sung:** Nếu không có `ifconfig`, dùng `ip addr show eth0` hoặc `ip -4 addr show eth0`.

         > 10.13.26.9

      1. Chạy `kubeadm init`, dùng địa chỉ IP đã lấy ở trên cho `--apiserver-advertise-address`

         ```
         kubeadm init \
            --apiserver-cert-extra-sans=controlplane \
            --apiserver-advertise-address 10.13.26.9 \
            --pod-network-cidr=10.244.0.0/16
         ```

      1. Thiết lập file kubeconfig mặc định

         ```
         mkdir ~/.kube
         cp /etc/kubernetes/admin.conf ~/.kube/config
         ```

         > 💡 **Bổ sung:** Với quyền `root`, cũng có thể dùng `export KUBECONFIG=/etc/kubernetes/admin.conf` thay vì copy file. Kiểm tra bằng `kubectl get nodes`, lúc này controlplane sẽ ở trạng thái `NotReady` vì chưa có CNI.

      </details>

  1.  <details>
      <summary>Tạo một kubeadm join token</summary>

      Bạn có thể copy lệnh join do `kubeadm init` in ra, có dạng như sau

      ```
      kubeadm join 10.13.26.9:6443 --token cpwmot.ldhadf3cokvyyx60 \
        --discovery-token-ca-cert-hash sha256:ea3a622922315b14b289c6efd7b1a77cbf81d29f6ddaf03472c304b6d3228c06
      ```

      Lưu ý rằng lệnh này sẽ khác nhau mỗi lần bạn làm lab.

      > 💡 **Bổ sung:** Nếu không còn output của `kubeadm init`, chạy `kubeadm token create --print-join-command` trên controlplane để tạo token mới kèm đầy đủ lệnh join. `--discovery-token-ca-cert-hash` là hash SHA-256 của public key của CA, giúp worker node xác minh rằng nó đang kết nối đúng tới control plane.

      </details>

  1.  <details>
      <summary>Join node01 vào cluster bằng join token</summary>

      1. `ssh` vào `node01` và dán lệnh join ở trên
      1. Quay lại controlplane node
      1. Chạy `kubectl get nodes`. Lưu ý rằng cả hai node đều ở trạng thái `NotReady`. Điều này là bình thường vì chúng ta chưa cài networking.

      </details>

  1.  <details>
      <summary>Cài một Network Plugin</summary>

      1. Cài flannel

         Nhấn vào tab "Install Network Plugin" phía trên terminal, rồi tìm link tới Flannel trong trang hiện ra.

         ```
         kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
         ```

      2. Chờ khoảng 30 giây, sau đó chạy `kubectl get nodes`. Lúc này các node sẽ ở trạng thái Ready.

         > 💡 **Bổ sung:** Flannel mặc định dùng dải Pod `10.244.0.0/16`, đó là lý do dùng `--pod-network-cidr=10.244.0.0/16` khi chạy `kubeadm init`. Nếu dùng dải khác, phải sửa `net-conf.json` trong ConfigMap của manifest Flannel cho khớp. Có thể theo dõi Pod của Flannel bằng `kubectl get pods -n kube-flannel` và CoreDNS bằng `kubectl get pods -n kube-system`.

      </details>





