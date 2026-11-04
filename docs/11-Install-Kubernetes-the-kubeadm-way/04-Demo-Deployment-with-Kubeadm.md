# Demo - Triển khai cluster bằng công cụ Kubeadm

- Xem [Bài giảng](https://kodekloud.com/topic/demo-deployment-with-kubeadm/)

# Apple Silicon

Nếu bạn dùng máy Apple M1 hoặc M2 (Apple Silicon), hãy làm theo hướng dẫn riêng tại [đây](../../apple-silicon/README.md).

Với tất cả các hệ điều hành/phần cứng khác, bao gồm cả máy Apple chip Intel, hãy tiếp tục theo trang này...

# VirtualBox/Vagrant

Trong lab này, bạn sẽ tự dựng một Kubernetes cluster gồm 3 node bằng `kubeadm`, sử dụng `VirtualBox` và `Vagrant`.

## Lưu ý quan trọng

Hiện tại không thể thực hiện lab này trên Apple M1/M2. Chúng tôi đang chờ VirtualBox phát hành phiên bản tương thích hoàn toàn với Apple Silicon.

## Thay đổi so với bài giảng

Kể từ khi video được ghi hình, Kubernetes và Ubuntu đã có nhiều thay đổi.

Hiện chúng ta dùng bản phát hành ổn định mới nhất của Ubuntu là 22.04.

Docker không còn được hỗ trợ làm container driver. Thay vào đó, chúng ta sẽ cài driver `containerd` và các công cụ đi kèm.

> 💡 **Bổ sung:** Cụ thể, dockershim (thành phần cho phép kubelet dùng Docker Engine) đã bị gỡ khỏi Kubernetes từ v1.24. Image build bằng Docker vẫn chạy bình thường trên containerd vì đều theo chuẩn OCI. Nếu muốn tiếp tục dùng Docker Engine thì phải cài thêm adapter `cri-dockerd`.
>
> Ngoài ra, repository cũ `apt.kubernetes.io` / `packages.cloud.google.com` dùng trong các lệnh bên dưới đã bị **ngừng hoạt động** (bị đóng băng từ tháng 9/2023 và gỡ bỏ từ tháng 3/2024). Hãy dùng repository cộng đồng `pkgs.k8s.io`. Mỗi minor version có một repository riêng, vd với v1.31:
>
> - `sudo mkdir -p -m 755 /etc/apt/keyrings`
> - `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg`
> - `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list`
> - `sudo apt-get update`
> - `sudo apt-get install -y kubelet kubeadm kubectl`
> - `sudo apt-mark hold kubelet kubeadm kubectl`
>
> Xem hướng dẫn cập nhật nhất tại [Kubeadm Clusters](../../kubeadm-clusters/).

# Hướng dẫn thực hành lab

1. Cài đặt phần mềm cần thiết

    <details>

    * VirtualBox: https://www.virtualbox.org/
    * Vagrant: https://developer.hashicorp.com/vagrant/downloads

    </details>

1. Clone repository này

    <details>

    ```
    git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
    cd certified-kubernetes-administrator-course
    ```

    </details>

1. Khởi động các máy ảo (VM)

    <details>

    ```
    vagrant up
    ```

    Lệnh này sẽ khởi động 3 VM có tên:

    * `kubemaster` - nơi chúng ta sẽ cài control plane
    * `kubenode01`
    * `kubenode02`

    </details>

1. Kiểm tra xem bạn có SSH được vào từng VM không

    <details>

    Lưu ý: Để thoát khỏi phiên ssh của VM, gõ `exit`

    ```
    vagrant ssh kubemaster
    ```

    ```
    vagrant ssh kubenode01
    ```

    ```
    vagrant ssh kubenode02
    ```

    </details>

1. Khởi tạo các node.

    Các bước sau phải được thực hiện trên cả ba node: `ssh` vào `kubemaster` và chạy các bước, sau đó làm tương tự với `kubenode01`, rồi `kubenode02`.

      1. Cấu hình các tham số kernel

            ```
            {
            cat <<EOF | sudo tee /etc/modules-load.d/11-k8s.conf
            br_netfilter
            EOF

            sudo modprobe br_netfilter

            cat <<EOF | sudo tee /etc/sysctl.d/11-k8s.conf
            net.bridge.bridge-nf-call-ip6tables = 1
            net.bridge.bridge-nf-call-iptables = 1
            net.ipv4.ip_forward = 1
            EOF

            sudo sysctl --system
            }
            ```

            > 💡 **Bổ sung:** Module `br_netfilter` cho phép traffic đi qua Linux bridge được iptables xử lý, nhờ đó kube-proxy và CNI hoạt động đúng. `net.ipv4.ip_forward = 1` cho phép node chuyển tiếp (forward) gói tin giữa các interface, điều bắt buộc để Pod trên các node khác nhau giao tiếp được với nhau. Ngoài ra, containerd cần module `overlay` cho overlayfs (có thể thêm `overlay` vào file module và chạy `sudo modprobe overlay`). Lệnh `sysctl --system` nạp lại tất cả cấu hình sysctl mà không cần khởi động lại máy.

    1. Cài container driver `containerd` và các công cụ đi kèm

        <details>

        ```bash
        {
            sudo apt update
            sudo apt install -y apt-transport-https ca-certificates curl
            sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
            echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
            sudo apt-get install -y containerd
            #sudo mkdir -p /opt/cni/bin
            #wget -q --https-only \
            #  https://github.com/containernetworking/plugins/releases/download/v0.8.#6/cni-plugins-linux-amd64-v0.8.6.tgz \
            #  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.25.0/#crictl-v1.25.0-linux-amd64.tar.gz
            #sudo tar -xzf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin
            #sudo tar -xzf crictl-v1.25.0-linux-amd64.tar.gz -C /usr/local/bin
        }
        ```

        > 💡 **Bổ sung:** Khi dùng kubelet với cgroup driver `systemd` (như trong `KubeletConfiguration` bên dưới), containerd cũng phải dùng `systemd` cgroup. Nếu không đồng nhất, các thành phần control plane có thể bị khởi động lại liên tục. Cách cấu hình phổ biến:
        >
        > - `sudo mkdir -p /etc/containerd`
        > - `containerd config default | sudo tee /etc/containerd/config.toml`
        > - `sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml`
        > - `sudo systemctl restart containerd`

        </details>

    1. Cài phần mềm Kubernetes

        <details>
        Bước này sẽ cài phiên bản mới nhất

        ```bash
        {
        sudo  curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

        echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.

        sudo apt update

        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl

        # Cấu hình crictl để không in ra các cảnh báo khó chịu
        sudo crictl config \
            --set runtime-endpoint=unix:///run/containerd/containerd.sock \
            --set image-endpoint=unix:///run/containerd/containerd.sock
        }
        ```

        > 💡 **Bổ sung:** Lệnh `echo ... | sudo tee /etc/apt/sources.` ở trên bị cắt cụt trong bản gốc (đúng ra là `/etc/apt/sources.list.d/kubernetes.list`). Hơn nữa, repository `apt.kubernetes.io` đã ngừng hoạt động, nên hãy dùng các lệnh với `pkgs.k8s.io` ở phần "Thay đổi so với bài giảng" phía trên. `apt-mark hold` giữ nguyên phiên bản để `apt upgrade` không vô tình nâng cấp các gói này. Việc upgrade cluster phải làm có kiểm soát bằng `kubeadm upgrade`.

        </details>

  1. Khởi tạo controlplane node

     <details>

      1. Lấy địa chỉ IP của adapter `eth0` trên controlplane

         ```
         ip addr show dev enp0s8
         ```

         Lấy giá trị in ra ở mục `inet` trong output. Giá trị đó phải là:

         > 192.168.56.11

         > 💡 **Lưu ý:** Trong VM VirtualBox, interface có tên `enp0s8` (adapter thứ hai, mạng host-only/bridge) chứ không phải `eth0`. Đó là lý do lệnh dùng `enp0s8`.

      1. Tạo file cấu hình để `kubeadm` đọc các thiết lập

          ```yaml
          kind: ClusterConfiguration
          apiVersion: kubeadm.k8s.io/v1beta3
          kubernetesVersion: v1.25.4          # <- Phiên bản tại thời điểm viết. Hãy đổi cho phù hợp
          controlPlaneEndpoint: 192.168.56.11:6443
          networking:
            serviceSubnet: "10.96.0.0/16"
            podSubnet: "10.244.0.0/16"
            dnsDomain: "cluster.local"
          controllerManager:
            extraArgs:
              "node-cidr-mask-size": "24"
          apiServer:
            extraArgs:
              authorization-mode: "Node,RBAC"
            certSANs:
              - "192.168.56.11"
              - "kubemaster"
              - "kubernetes"
              
          ---
          kind: KubeletConfiguration
          apiVersion: kubelet.config.k8s.io/v1beta1
          cgroupDriver: systemd
          serverTLSBootstrap: true
          ```

          > 💡 **Bổ sung:** Để dùng file này, lưu nó lại (vd `kubeadm-config.yaml`) rồi chạy `sudo kubeadm init --config kubeadm-config.yaml` thay cho lệnh dùng flag ở bước tiếp theo. Bạn chỉ cần chọn một trong hai cách. Một số lưu ý:
          >
          > - Từ kubeadm v1.31 có thêm API `kubeadm.k8s.io/v1beta4`, trong đó `extraArgs` chuyển sang dạng danh sách `name`/`value`. Có thể chuyển đổi file cấu hình cũ bằng `kubeadm config migrate`.
          > - `kubeadm config print init-defaults` in ra cấu hình mặc định để tham khảo.
          > - Với `serverTLSBootstrap: true`, kubelet sẽ gửi CertificateSigningRequest (CSR) cho serving certificate. Bạn cần duyệt các CSR này (`kubectl get csr`, `kubectl certificate approve <tên-csr>`), nếu không thì `kubectl logs`/`kubectl exec` sẽ báo lỗi TLS.

      1. Chạy `kubeadm init`, dùng địa chỉ IP đã lấy ở trên cho `--apiserver-advertise-address`

         ```
         sudo kubeadm init \
            --apiserver-cert-extra-sans=kubemaster01 \
            --apiserver-advertise-address 192.168.56.11 \
            --pod-network-cidr=10.244.0.0/16
         ```

         Ghi lại lệnh `kubeadm join` được in ra ở cuối quá trình này. Bạn sẽ cần nó ở bước `Khởi tạo các worker node` bên dưới.

         > 💡 **Bổ sung:** Ý nghĩa các flag:
         >
         > - `--apiserver-advertise-address`: IP mà kube-apiserver quảng bá (advertise) và lắng nghe.
         > - `--apiserver-cert-extra-sans`: thêm tên/IP vào trường SAN của certificate kube-apiserver.
         > - `--pod-network-cidr`: dải IP cho Pod. Dải này phải khớp với cấu hình của CNI plugin, vd Flannel mặc định dùng `10.244.0.0/16`.
         >
         > Nếu `kubeadm init` lỗi giữa chừng, chạy `sudo kubeadm reset` để dọn dẹp trước khi thử lại.

      1. Thiết lập file kubeconfig mặc định

         ```
         {
         mkdir ~/.kube
         sudo cp /etc/kubernetes/admin.conf ~/.kube/config
         sudo chown vagrant:vagrant ~/.kube/config
         }
         ```

         > 💡 **Bổ sung:** Sau bước này, các node sẽ ở trạng thái `NotReady` cho đến khi bạn cài một CNI plugin, vd Flannel: `kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`.

     </details>

1. Khởi tạo các worker node

    <details>

    Các bước sau phải được thực hiện trên cả hai worker node: `ssh` vào `kubenode01` và chạy các bước, sau đó làm tương tự với `kubenode02`.

    * Dán lệnh `kubeadm join` từ bước trên vào command prompt rồi nhấn Enter (nhớ chạy với `sudo`).

    > 💡 **Bổ sung:** Nếu làm mất lệnh join hoặc token đã hết hạn (TTL mặc định là 24 giờ), hãy chạy `kubeadm token create --print-join-command` trên controlplane để tạo lại lệnh. Sau khi join xong, kiểm tra bằng `kubectl get nodes` trên controlplane.

    </details>
