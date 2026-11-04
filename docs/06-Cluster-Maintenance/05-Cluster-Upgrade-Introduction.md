# Giới thiệu về Upgrade Cluster
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/cluster-upgrade-introduction/)
  
#### Có bắt buộc tất cả các thành phần Kubernetes phải cùng phiên bản không?
- Không, các thành phần có thể ở các phiên bản khác nhau.

> 💡 **Bổ sung – Version skew policy (chính sách chênh lệch phiên bản):** kube-apiserver là thành phần "trung tâm", các thành phần khác không được mới hơn nó. Với kube-apiserver ở phiên bản `1.X`:
> - kube-apiserver (trong cluster HA): các instance chênh nhau tối đa 1 minor version.
> - kube-controller-manager, kube-scheduler, cloud-controller-manager: `1.X` hoặc `1.X-1`.
> - kubelet và kube-proxy: từ `1.X-3` đến `1.X` (từ v1.28; trước đó là tối đa `1.X-2`).
> - kubectl: `1.X+1`, `1.X` hoặc `1.X-1`.
>
> Hệ quả: khi upgrade, luôn upgrade control plane (kube-apiserver) **trước**, sau đó mới tới kubelet trên các node.
  
#### Tại một thời điểm, Kubernetes chỉ hỗ trợ 3 minor version gần nhất
- Cách khuyến nghị là upgrade lần lượt từng minor version một (vd 1.10 → 1.11 → 1.12, không nhảy thẳng 1.10 → 1.12).
  
  ![up2](../../images/up2.PNG)
  
#### Các lựa chọn để upgrade cluster k8s
 
  ![opt](../../images/opt.PNG)

> 💡 **Bổ sung:** Với dịch vụ managed (GKE, EKS, AKS), nhà cung cấp cloud thực hiện upgrade control plane chỉ với vài thao tác. Với cluster tự dựng bằng kubeadm, dùng `kubeadm upgrade`. Với cluster dựng "the hard way" (từ binary), phải tự upgrade từng thành phần thủ công.
  
## Upgrade một Cluster
- Upgrade cluster gồm 2 bước chính: upgrade master node (control plane) trước, sau đó upgrade các worker node.
- Trong khi master node đang được upgrade, các thành phần control plane (kube-apiserver, scheduler, controller-manager) tạm thời không khả dụng. Điều này không làm ảnh hưởng ứng dụng đang chạy trên worker node, nhưng bạn không thể dùng `kubectl` để truy cập cluster, không thể deploy ứng dụng mới, và nếu Pod bị lỗi thì sẽ không được tạo lại cho đến khi control plane hoạt động trở lại.
  
#### Có nhiều chiến lược khác nhau để upgrade các worker node
- Cách thứ nhất là upgrade tất cả cùng lúc. Nhưng khi đó các Pod sẽ bị down và người dùng không thể truy cập ứng dụng (có downtime).
  ![stg1](../../images/stg1.PNG)
- Cách thứ hai là upgrade lần lượt từng node một (workload được dời sang các node khác trong lúc upgrade).
  ![stg2](../../images/stg2.PNG)
- Cách thứ ba là thêm các node mới (đã có phiên bản mới) vào cluster, dời workload sang, rồi gỡ node cũ. Cách này đặc biệt thuận tiện trên môi trường cloud, nơi có thể dễ dàng tạo node mới.
  ![stg3](../../images/stg3.PNG)
  
## kubeadm - Upgrade master node
- kubeadm có lệnh upgrade giúp upgrade cluster. `kubeadm upgrade plan` cho biết phiên bản hiện tại, phiên bản kubeadm, phiên bản stable mới nhất, và các thành phần sẽ được upgrade.
  ```
  $ kubeadm upgrade plan
  ```
  ![kube1](../../images/kube1.png)

> 💡 **Lưu ý:** kubeadm không cài đặt hay upgrade kubelet. Sau khi upgrade control plane, bạn phải tự upgrade kubelet trên từng node. Ngoài ra kubeadm cũng tuân theo phiên bản của chính nó, nên cần upgrade kubeadm **trước** rồi mới upgrade cluster.
  
- Upgrade kubeadm từ v1.11 lên v1.12
  ```
  $ apt-get upgrade -y kubeadm=1.12.0-00
  ```
- Upgrade cluster
  ```
  $ kubeadm upgrade apply v1.12.0
  ```
- Nếu chạy lệnh 'kubectl get nodes', bạn vẫn thấy phiên bản cũ. Lý do là output của lệnh này hiển thị phiên bản của kubelet trên từng node đã đăng ký với API Server, chứ không phải phiên bản của chính API Server.
  ```
  $ kubectl get nodes
  ```
  
  ![kubeu](../../images/kubeu.PNG)
  
- Upgrade 'kubelet' trên master node (với cluster kubeadm, các thành phần control plane chạy dưới dạng static Pod trên master node nên master node cũng có kubelet)
  ```
  $ apt-get upgrade kubelet=1.12.0-00
  ```
- Restart kubelet
  ```
  $ systemctl restart kubelet
  ```
- Chạy 'kubectl get nodes' để kiểm tra
  ```
  $ kubectl get nodes
  ```
  
  ![kubeu1](../../images/kubeu1.PNG)
 
## kubeadm - Upgrade worker node
  
- Từ master node, chạy lệnh 'kubectl drain' để dời workload sang các node khác
  ```
  $ kubectl drain node-1
  ```
- Upgrade các gói kubeadm và kubelet
  ```
  $ apt-get upgrade -y kubeadm=1.12.0-00
  $ apt-get upgrade -y kubelet=1.12.0-00
  ```
- Cập nhật cấu hình node cho phiên bản kubelet mới
  ```
  $ kubeadm upgrade node config --kubelet-version v1.12.0
  ```
- Restart dịch vụ kubelet
  ```
  $ systemctl restart kubelet
  ```
- Đánh dấu node có thể schedule trở lại
  ```
  $ kubectl uncordon node-1
  ```
  
  ![kubeu2](../../images/kubeu2.PNG)
  
- Upgrade tất cả các worker node còn lại theo cách tương tự

  ![kubeu3](../../images/kubeu3.PNG)

> 💡 **Bổ sung – Quy trình upgrade kubeadm hiện tại (v1.30+, dùng cho kỳ thi):** Các lệnh trong video (`apt-get upgrade kubeadm=1.12.0-00`, `kubeadm upgrade node config --kubelet-version`) đã lỗi thời.
> - Repo cũ `apt.kubernetes.io` / `packages.cloud.google.com` **đã ngừng hoạt động** (bị đóng băng từ 9/2023 và gỡ bỏ vào 2024). Hiện nay dùng repo cộng đồng **`pkgs.k8s.io`**, và **mỗi minor version có một repo riêng**. Vì vậy muốn upgrade lên minor version mới phải đổi repo trước.
> - Định dạng phiên bản gói mới là `1.31.x-1.1` (không còn `-00`).
> - `kubeadm upgrade node config --kubelet-version` đã bị loại bỏ; trên worker node (và các control plane node phụ) chỉ cần `kubeadm upgrade node`.
>
> **Trên control plane node** (ví dụ upgrade từ 1.30 lên 1.31):
> ```
> # 1. Đổi repo sang minor version mới
> vim /etc/apt/sources.list.d/kubernetes.list
> # deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /
>
> # 2. Xem các phiên bản có sẵn
> apt-get update
> apt-cache madison kubeadm
>
> # 3. Upgrade kubeadm
> apt-mark unhold kubeadm
> apt-get install -y kubeadm='1.31.x-*'
> apt-mark hold kubeadm
> kubeadm version
>
> # 4. Xem kế hoạch và áp dụng upgrade
> kubeadm upgrade plan
> kubeadm upgrade apply v1.31.x
>
> # 5. Drain node, upgrade kubelet và kubectl
> kubectl drain <controlplane-node> --ignore-daemonsets
> apt-mark unhold kubelet kubectl
> apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*'
> apt-mark hold kubelet kubectl
> systemctl daemon-reload
> systemctl restart kubelet
> kubectl uncordon <controlplane-node>
> ```
> (Thay `x` bằng patch version cụ thể, vd `1.31.2`. Nếu có nhiều control plane node, các node còn lại dùng `kubeadm upgrade node` thay vì `kubeadm upgrade apply`.)
>
> **Trên từng worker node:**
> ```
> # Trên worker node: đổi repo, upgrade kubeadm như trên rồi
> kubeadm upgrade node
>
> # Từ máy có kubectl (control plane): drain node
> kubectl drain <worker-node> --ignore-daemonsets
>
> # Trên worker node: upgrade kubelet, kubectl
> apt-mark unhold kubelet kubectl
> apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*'
> apt-mark hold kubelet kubectl
> systemctl daemon-reload
> systemctl restart kubelet
>
> # Từ control plane: uncordon
> kubectl uncordon <worker-node>
> ```
> Mẹo thi: luôn kiểm tra kỹ bạn đang SSH vào đúng node nào; lệnh `kubectl drain/uncordon` thường chạy từ control plane; xác nhận kết quả bằng `kubectl get nodes`.
  

#### Video demo về [Upgrade Cluster](https://kodekloud.com/topic/demo-cluster-upgrade/)

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/
