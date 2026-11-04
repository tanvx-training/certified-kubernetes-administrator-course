# ETCD trong Kubernetes
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/etcd-in-kubernetes/)

Trong phần này, chúng ta sẽ tìm hiểu vai trò của ETCD trong Kubernetes

## Kho dữ liệu ETCD
- Kho dữ liệu ETCD lưu thông tin về cluster như **`Nodes`**, **`PODS`**, **`Configs`**, **`Secrets`**, **`Accounts`**, **`Roles`**, **`Bindings`** và **`Others`** (các đối tượng khác).
- Mọi thông tin bạn thấy khi chạy lệnh **`kubectl get`** đều đến từ **`ETCD Server`**.

> 💡 **Bổ sung:** Mọi thay đổi bạn thực hiện trên cluster (thêm node, tạo Pod, Deployment, ReplicaSet...) đều được ghi vào ETCD server; chỉ khi đã được ghi vào ETCD thì thay đổi mới được coi là hoàn tất.

## Cài đặt - Thủ công
- Nếu bạn tự dựng cluster từ đầu (from scratch), bạn sẽ tự triển khai **`ETCD`** bằng cách tải binary ETCD về
- Tự cài binary và cấu hình ETCD chạy như một service trên master node.
  ```
  $ wget -q --https-only "https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz"
  ```

  ![etcd](../../images/etcd.PNG)

> 💡 **Bổ sung:** Trong cấu hình service, tham số quan trọng cần chú ý là `--advertise-client-urls`: địa chỉ mà ETCD công bố cho client kết nối tới — gồm IP của server và port `2379` (còn `--listen-client-urls` là địa chỉ etcd thực sự lắng nghe). Đây chính là URL cần cấu hình cho kube-apiserver khi nó cần kết nối tới ETCD server.
  
## Cài đặt - Kubeadm
- Nếu bạn dựng cluster bằng **`kubeadm`** thì kubeadm sẽ triển khai ETCD server cho bạn dưới dạng một Pod trong namespace **`kube-system`**.
  ```
  $ kubectl get pods -n kube-system
  ```
  ![etcd1](../../images/etcd1.PNG)

> 💡 **Bổ sung:** Với kubeadm, etcd chạy dưới dạng **static Pod** có tên `etcd-<tên-node>` (ví dụ `etcd-controlplane`). Manifest nằm ở `/etc/kubernetes/manifests/etcd.yaml`, chứa các thông tin hay được hỏi khi thi: đường dẫn dữ liệu `--data-dir` (mặc định `/var/lib/etcd`), các file certificate (`--cert-file`, `--key-file`, `--trusted-ca-file` dưới `/etc/kubernetes/pki/etcd/`) và `--listen-client-urls`. Mô hình etcd chạy ngay trên các control plane node như vậy gọi là **stacked etcd**; ngược lại, etcd chạy trên các máy riêng gọi là **external etcd**.

## Khám phá ETCD
- Để liệt kê toàn bộ key do Kubernetes lưu, chạy lệnh dưới đây
  ```
  $ kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt get / --prefix --keys-only"
  ```
- Kubernetes lưu dữ liệu theo một cấu trúc thư mục cụ thể: thư mục gốc là **`registry`**, bên dưới là các đối tượng Kubernetes như **`minions`**, **`nodes`**, **`pods`**, **`replicasets`**, **`deployments`**, **`roles`**, **`secrets`** và **`Others`**.
  
  ![etcdctl1](../../images/etcdctl1.PNG)

> 💡 **Bổ sung:** Thay `etcd-master` bằng tên Pod etcd thực tế trong cluster của bạn (xem bằng `kubectl get pods -n kube-system`). Vì etcd giao tiếp qua TLS, mọi lệnh `etcdctl` đều phải kèm `--cacert`, `--cert`, `--key` (và `--endpoints` nếu chạy từ ngoài Pod). Lưu ý dữ liệu trong etcd (kể cả Secret) mặc định **không được mã hóa** — đó là lý do có tính năng encryption at rest (`EncryptionConfiguration`).

## ETCD trong môi trường HA
   - Trong môi trường high availability (sẵn sàng cao), cluster sẽ có nhiều master node, với nhiều instance ETCD trải đều trên các master node đó.
   - Đảm bảo các instance etcd biết nhau bằng cách thiết lập đúng tham số trong cấu hình **`etcd.service`**. Đó là tùy chọn **`--initial-cluster`**, nơi bạn khai báo các instance khác nhau của service etcd.
     ![etcd-ha](../../images/etcd-ha.PNG)

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#stacked-control-plane-and-etcd-nodes
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#external-etcd-nodes
