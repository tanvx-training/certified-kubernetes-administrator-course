# Các phương pháp Backup và Restore
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/backup-and-restore-methods/)
  
Trong phần này, chúng ta sẽ tìm hiểu các phương pháp backup và restore.

## Những thứ cần backup
 
 ![bc](../../images/bc.PNG)

Gồm ba nhóm chính: file cấu hình resource (resource configuration), cluster etcd (nơi lưu toàn bộ trạng thái cluster), và persistent volume (dữ liệu ứng dụng).
 
## Cấu hình resource (Resource Configuration)
- Cách imperative (dùng lệnh trực tiếp)
  
  ![rci](../../images/rci.PNG)

- Cách declarative (cách được khuyến nghị) – định nghĩa resource trong file rồi `kubectl apply`
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
      type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  ```
 ![rcd](../../images/rcd.PNG)
 
- Một thói quen tốt là lưu các file cấu hình resource trên kho mã nguồn như GitHub. Khi đó, kể cả mất toàn bộ cluster, bạn vẫn có thể deploy lại ứng dụng từ các file này.

  ![rcd1](../../images/rcd1.PNG)

## Backup - Resource Configs
Nếu không chắc mọi resource đều được tạo theo cách declarative (có người tạo bằng lệnh imperative), cách tốt hơn là truy vấn kube-apiserver để lấy cấu hình của tất cả resource:

  ```
  $ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml (only for few resource groups)
  ```

- Lệnh trên chỉ lấy một số nhóm resource (`get all` không bao gồm ConfigMap, Secret, PVC, Ingress, RBAC...). Còn rất nhiều nhóm resource khác cần xem xét. Có những công cụ như **`ARK`**, nay gọi là **`Velero`** (ban đầu của Heptio), có thể làm việc này cho bạn.

  ![brc](../../images/brc.PNG)
  
## Backup - ETCD
- Thay vì backup từng resource như trên, bạn có thể chọn backup chính cluster etcd. etcd lưu thông tin về trạng thái của cluster, bao gồm các node và mọi resource được tạo bên trong cluster.
- Khi cấu hình etcd, bạn chỉ định thư mục lưu toàn bộ dữ liệu (flag `--data-dir`). Bạn có thể dùng công cụ backup để sao lưu thư mục này.
  
  ![be](../../images/be.PNG)
  
- Ngoài ra, bạn có thể tạo snapshot cơ sở dữ liệu etcd bằng lệnh snapshot save của công cụ **`etcdctl`**.
  ```
  $ ETCDCTL_API=3 etcdctl snapshot save snapshot.db
  ```
  ```
  $  ETCDCTL_API=3 etcdctl snapshot status snapshot.db
  ```
  ![be1](../../images/be1.PNG)
  
## Restore - ETCD
- Để restore etcd từ bản backup sau này: trước tiên dừng dịch vụ kube-apiserver (vì quá trình restore etcd yêu cầu restart etcd cluster, và kube-apiserver phụ thuộc vào nó)
  ```
  $ service kube-apiserver stop
  ```
- Chạy lệnh etcdctl snapshot restore, chỉ định `--data-dir` mới. Khi restore, etcd khởi tạo cấu hình cluster mới và cấu hình các member như member mới của một cluster mới, để tránh member mới vô tình tham gia vào cluster cũ.
- Cập nhật cấu hình dịch vụ etcd để dùng data directory mới
- Reload cấu hình hệ thống
  ```
  $ systemctl daemon-reload
  ```
- Restart etcd
  ```
  $ service etcd restart
  ```
  
  ![er](../../images/er.PNG)
  
- Khởi động lại kube-apiserver
  ```
  $ service kube-apiserver start
  ```
#### Với mọi lệnh etcdctl, cần chỉ định cert, key, cacert và endpoint để xác thực.
```
$ ETCDCTL_API=3 etcdctl \
  snapshot save /tmp/snapshot.db \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/etcd-server.crt \
  --key=/etc/kubernetes/pki/etcd/etcd-server.key
```

  ![erest](../../images/erest.PNG)

> 💡 **Bổ sung – Lưu ý thực tế và khi thi:**
> - Endpoint nên viết `https://127.0.0.1:2379` (dấu ngoặc vuông `[...]` chỉ dùng cho địa chỉ IPv6). Nếu etcd chạy cùng node (stacked etcd) và lắng nghe trên `127.0.0.1:2379` thì có thể bỏ qua `--endpoints` vì đây là giá trị mặc định.
> - Trên cluster kubeadm, chứng chỉ server của etcd là `/etc/kubernetes/pki/etcd/server.crt` và `server.key` (tên `etcd-server.crt` trong ví dụ trên là của cluster dựng thủ công). Hãy luôn kiểm tra đường dẫn thực tế:
>   ```
>   grep -E 'cert|key|listen-client' /etc/kubernetes/manifests/etcd.yaml
>   ```
> - Các lệnh `snapshot status` và `snapshot restore` không cần kết nối tới etcd nên không cần cert/endpoint. Từ etcd v3.5, chúng được khuyến nghị chạy bằng `etcdutl`:
>   ```
>   etcdutl snapshot status /tmp/snapshot.db -w table
>   etcdutl snapshot restore /tmp/snapshot.db --data-dir /var/lib/etcd-from-backup
>   ```
> - **Restore trên cluster kubeadm (stacked etcd)**: etcd và kube-apiserver chạy dưới dạng static Pod, không phải systemd service, nên không có `service kube-apiserver stop`. Quy trình là: restore snapshot vào thư mục mới → sửa `/etc/kubernetes/manifests/etcd.yaml`, đổi `hostPath.path` của volume `etcd-data` (và/hoặc `--data-dir`) sang thư mục mới → kubelet tự tạo lại Pod etcd. Xem chi tiết ở [Practice Test - Backup and Restore Methods](09-Practice-Test-Backup-and-Restore-Methods.md).
> - **Restore với external etcd** (chạy như systemd service): restore vào thư mục mới, `chown -R etcd:etcd <thư-mục>`, sửa `--data-dir` trong unit file, rồi `systemctl daemon-reload && systemctl restart etcd`. Xem [Practice Test - Backup and Restore Methods 2](10-Practice-Test-Backup-and-Restore-Methods-2.md).
> - Với managed Kubernetes (EKS, GKE, AKS), bạn không truy cập được etcd, nên backup bằng cách truy vấn API (vd Velero) là lựa chọn duy nhất.
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/


 
