# Practice Test - DaemonSets (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-daemonsets/)
  
Lời giải cho bài practice test DaemonSets
- Chạy lệnh kubectl get daemonsets --all-namespaces (đếm số DaemonSet trong tất cả namespace)
  
  <details>

  ```
  $ kubectl get daemonsets --all-namespaces
  ```
  </details>

- Chạy lệnh kubectl get daemonsets --all-namespaces (xác định namespace chứa các DaemonSet)

  <details>

  ```
  $ kubectl get daemonsets --all-namespaces
  ```
  </details>

- Chạy lệnh kubectl get all --all-namespaces và xác định loại của các đối tượng

  <details>

  ```
  $ kubectl get all --all-namespaces
  ```
  </details>

- Chạy lệnh kubectl describe daemonset kube-proxy --namespace=kube-system (xem số Pod đang chạy/được lập lịch trên các node)

  <details>

  ```
  $ kubectl describe daemonset kube-proxy --namespace=kube-system
  ```
  </details>

- Chạy lệnh kubectl describe daemonset kube-flannel-ds-amd64 --namespace=kube-system (xem image mà DaemonSet đang dùng). Lưu ý: ở các phiên bản lab mới, DaemonSet của Flannel có thể tên là `kube-flannel-ds` và nằm trong namespace `kube-flannel`.

  <details>

  ```
  $ kubectl describe daemonset kube-flannel-ds-amd64 --namespace=kube-system
  ```
  </details>
    
- Tạo DaemonSet

  <details>

  ```
  $ vi ds.yaml
  ```

  ```
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: elasticsearch
    namespace: kube-system
    labels:
      k8s-app: fluentd-logging
  spec:
    selector:
      matchLabels:
        name: elasticsearch
    template:
      metadata:
        labels:
          name: elasticsearch
      spec:
        tolerations:
        # toleration này cho phép DaemonSet chạy trên master node
        # xóa đi nếu master node của bạn không được chạy Pod
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        containers:
        - name: elasticsearch
          image: k8s.gcr.io/fluentd-elasticsearch:1.20
  ```

  > 💡 **Cập nhật:** Từ Kubernetes v1.24, taint của control plane là `node-role.kubernetes.io/control-plane` (key `node-role.kubernetes.io/master` không còn được dùng), nên toleration nên đổi thành `key: node-role.kubernetes.io/control-plane`. Registry `k8s.gcr.io` đã bị đóng băng, image chính thức hiện được phân phối qua `registry.k8s.io`. Luôn dùng đúng image mà đề bài yêu cầu (lab thường dùng `registry.k8s.io/fluentd-elasticsearch:1.20`).
  </details>

- Tạo DaemonSet, sau đó liệt kê các DaemonSet và Pod

  <details>

  ```
  $ kubectl create -f ds.yaml
  $ kubectl get ds -n kube-system
  $ kubectl get pod -n kube-system|grep elasticsearch
  ```
  </details>




