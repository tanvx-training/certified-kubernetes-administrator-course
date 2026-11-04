# DaemonSets
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/daemonsets/)

Trong phần này, chúng ta sẽ tìm hiểu về DaemonSets.

#### DaemonSet giống ReplicaSet ở chỗ nó giúp triển khai nhiều instance của Pod. Nhưng DaemonSet chạy đúng một bản sao của Pod trên mỗi node trong cluster.
- Khi có node mới được thêm vào cluster, một bản sao của Pod sẽ tự động được tạo trên node đó. Khi node bị gỡ khỏi cluster, Pod cũng tự động bị xóa. DaemonSet đảm bảo luôn có một bản sao của Pod trên mọi node.
  
  ![ds](../../images/ds.PNG)
  
## DaemonSets - Các trường hợp sử dụng
- Triển khai monitoring agent hoặc log collector trên mỗi node (ví dụ node-exporter, Fluentd/Fluent Bit).
- kube-proxy: một thành phần cần chạy trên mọi node — thường được triển khai dưới dạng DaemonSet.
- Giải pháp mạng (CNI) như Weave Net, Flannel, Calico cũng cần một agent trên mỗi node và thường được triển khai bằng DaemonSet.

  ![ds-uc](../../images/ds-uc.PNG)
  
  ![ds-uc-kp](../../images/ds-uc-kp.PNG)
  
  ![ds-ucn](../../images/ds-ucn.PNG)
  
## DaemonSets - Định nghĩa
- Cách tạo DaemonSet tương tự như tạo ReplicaSet.
- Với DaemonSet, ta vẫn có apiVersion, kind là **`DaemonSet`** thay vì **`ReplicaSet`**, metadata và spec. Phần spec gồm `selector` và `template` y hệt ReplicaSet (chỉ khác là không có `replicas`). Hãy đảm bảo label trong `selector` khớp với label trong `template`.
  ```
  apiVersion: apps/v1
  kind: Replicaset
  metadata:
    name: monitoring-daemon
    labels:
      app: nginx
  spec:
    selector:
      matchLabels:
        app: monitoring-agent
    template:
      metadata:
       labels:
         app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
  ```
  
  ```
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: monitoring-daemon
    labels:
      app: nginx
  spec:
    selector:
      matchLabels:
        app: monitoring-agent
    template:
      metadata:
       labels:
         app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
  ```
  ![dsd](../../images/dsd.PNG)
  
- Tạo DaemonSet từ file định nghĩa
  ```
  $ kubectl create -f daemon-set-definition.yaml
  ```

## Xem DaemonSets
- Liệt kê các DaemonSet
  ```
  $ kubectl get daemonsets
  ```
- Xem chi tiết DaemonSet
  ```
  $ kubectl describe daemonsets monitoring-daemon
  ```
  ![ds1](../../images/ds1.PNG)
  
## DaemonSets hoạt động như thế nào
- Cách đơn giản nhất để chạy Pod trên mỗi node là đặt `nodeName` cho từng Pod (đây là cách DaemonSet làm cho đến Kubernetes v1.12).
- Từ v1.12 trở đi, DaemonSet dùng default scheduler kết hợp với node affinity (DaemonSet controller tự thêm node affinity trỏ tới từng node cụ thể vào mỗi Pod).

  ![ds2](../../images/ds2.PNG)

> 💡 **Bổ sung:**
> - Lưu ý: trong file mẫu ReplicaSet ở trên, `kind: Replicaset` viết sai chính tả — đúng phải là `ReplicaSet` (phân biệt hoa thường).
> - Không có lệnh `kubectl create daemonset`. Mẹo thi: tạo YAML từ Deployment rồi sửa lại:
>   ```
>   kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > ds.yaml
>   ```
>   Sau đó đổi `kind: Deployment` thành `kind: DaemonSet`, xóa các trường `replicas`, `strategy` và `status`.
> - DaemonSet controller tự động thêm một số toleration cho Pod (ví dụ `node.kubernetes.io/not-ready`, `node.kubernetes.io/unreachable`, `node.kubernetes.io/unschedulable`...), nên Pod của DaemonSet vẫn chạy được trên node đã bị `cordon`. Tuy nhiên, để chạy trên control plane node vẫn phải tự thêm toleration `node-role.kubernetes.io/control-plane:NoSchedule`.
> - Có thể giới hạn DaemonSet chỉ chạy trên một số node bằng `nodeSelector` hoặc node affinity trong `template.spec`.
> - Khi `kubectl drain` một node, cần thêm `--ignore-daemonsets` vì Pod của DaemonSet sẽ không bị xóa khỏi node.
> - DaemonSet hỗ trợ rolling update (`spec.updateStrategy.type: RollingUpdate`, mặc định) hoặc `OnDelete`; có thể dùng `kubectl rollout status/undo daemonset <tên>`.

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec
