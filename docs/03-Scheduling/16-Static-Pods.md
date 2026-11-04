# Static Pods 
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/static-pods/)
  
Trong phần này, chúng ta sẽ tìm hiểu về Static Pods

#### Làm thế nào để cung cấp file định nghĩa Pod cho kubelet khi không có kube-apiserver?
- kubelet có thể tự quản lý một node một cách độc lập, kể cả khi không có master (không có kube-apiserver, scheduler, controller hay etcd).
- Bạn có thể cấu hình kubelet đọc các file định nghĩa Pod từ một thư mục trên server, nơi được chỉ định để lưu thông tin về Pod. kubelet định kỳ kiểm tra thư mục này: tạo Pod khi có file mới, cập nhật (tạo lại) Pod khi file thay đổi, và xóa Pod khi file bị xóa. Kubelet cũng đảm bảo Pod luôn chạy, nếu ứng dụng crash thì kubelet sẽ khởi động lại.
- Những Pod được kubelet tự tạo như vậy, không thông qua kube-apiserver hay các thành phần khác của control plane, được gọi là **static Pod**. Lưu ý: cách này chỉ tạo được Pod, không tạo được ReplicaSet, Deployment hay Service — các đối tượng đó cần controller trên control plane.

## Cấu hình Static Pod
- Thư mục được chỉ định có thể là bất kỳ thư mục nào trên host, và vị trí của thư mục được truyền cho kubelet dưới dạng một option khi chạy service.
  - Option này tên là **`--pod-manifest-path`**.
  
  ![sp](../../images/sp.PNG)
  
## Một cách khác để cấu hình static Pod 
- Thay vì khai báo option trực tiếp trong file **`kubelet.service`**, bạn có thể chỉ tới một file cấu hình khác bằng option `--config`, và khai báo đường dẫn thư mục bằng trường `staticPodPath` trong file đó. Cluster dựng bằng kubeadm dùng cách này.

  ![sp1](../../images/sp1.PNG)

## Xem các static Pod
- Để xem các static Pod (khi không có kube-apiserver, `kubectl` không dùng được)
  ```
  $ docker ps
  ```
  ![sp2](../../images/sp2.PNG)

#### kubelet có thể tạo đồng thời cả hai loại Pod - static Pod và Pod nhận từ kube-apiserver.
- Khi node là một phần của cluster, với mỗi static Pod, kubelet sẽ tạo một **mirror Pod** (bản sao chỉ đọc) trên kube-apiserver. Vì vậy `kubectl get pods` vẫn thấy static Pod, nhưng không thể sửa hay xóa nó qua kubectl — chỉ có thể sửa/xóa file manifest trên node. Tên của mirror Pod được thêm hậu tố là tên node, ví dụ `kube-apiserver-controlplane`.

  ![sp3](../../images/sp3.PNG)

## Static Pods - Trường hợp sử dụng
- Vì static Pod không phụ thuộc vào control plane, có thể dùng chúng để triển khai chính các thành phần của control plane dưới dạng Pod trên một node: cài kubelet trên master node, rồi đặt file định nghĩa Pod của `kube-apiserver`, `kube-controller-manager`, `etcd`... vào thư mục manifest. kubelet sẽ tự chạy và khởi động lại chúng nếu bị crash. Đây chính là cách kubeadm dựng cluster.

  ![sp4](../../images/sp4.PNG)
  
  ![sp5](../../images/sp5.PNG)
  
## Static Pods so với DaemonSets
- Static Pod: do kubelet tạo trực tiếp, không cần kube-apiserver; dùng để triển khai các thành phần control plane.
- DaemonSet: do DaemonSet controller (thông qua kube-apiserver) tạo, đảm bảo mỗi node có một bản sao của Pod; dùng cho monitoring agent, logging agent...
- Theo slide, cả hai đều **bị kube-scheduler bỏ qua**. Chính xác hơn: static Pod hoàn toàn không đi qua scheduler; còn Pod của DaemonSet, từ v1.12, vẫn được default scheduler gán node, nhưng node đích đã được DaemonSet controller xác định trước bằng node affinity, nên scheduler không thực sự "chọn" node.

   ![spvsds](../../images/spvsds.PNG)
  
> 💡 **Bổ sung (cập nhật cho Kubernetes hiện tại):**
> - Từ Kubernetes v1.24, dockershim đã bị gỡ bỏ, các node thường dùng containerd hoặc CRI-O nên lệnh `docker ps` không còn dùng được. Thay vào đó dùng `crictl ps` (hoặc `nerdctl ps` với containerd).
> - Thư mục static Pod mặc định trên cluster kubeadm: `/etc/kubernetes/manifests`. Tìm đường dẫn thực tế:
>   ```
>   ps -aux | grep kubelet            # tìm giá trị --config (thường là /var/lib/kubelet/config.yaml)
>   grep -i staticPodPath /var/lib/kubelet/config.yaml
>   ```
> - Nhận biết static Pod qua kubectl: tên có hậu tố `-<tên-node>`, và trong `kubectl get pod <tên> -o yaml` phần `ownerReferences` có `kind: Node` (thay vì `ReplicaSet`/`DaemonSet`...).
> - Tạo nhanh file manifest cho static Pod khi thi:
>   ```
>   kubectl run static-busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
>   ```
> - Nếu static Pod nằm trên worker node, phải SSH vào node đó để sửa/xóa file manifest. Sau khi sửa `staticPodPath`, cần `systemctl restart kubelet`.
> - Static Pod không thể tham chiếu tới ServiceAccount, ConfigMap, Secret... (các đối tượng API khác).

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
