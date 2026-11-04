# Giám sát các thành phần của Cluster (Monitor Cluster Components)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/monitor-cluster-components/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách giám sát (monitoring) một Kubernetes cluster.

#### Làm thế nào để giám sát mức tiêu thụ tài nguyên trong Kubernetes? Hay quan trọng hơn, chúng ta muốn giám sát những gì?
  ![mon](../../images/mon.PNG)

Ở cấp độ node, ta muốn biết số lượng node trong cluster, bao nhiêu node đang khỏe (healthy), cùng các chỉ số hiệu năng như CPU, memory, network và disk. Ở cấp độ Pod, ta muốn biết số lượng Pod và mức tiêu thụ CPU/memory của từng Pod.

Kubernetes không đi kèm sẵn một giải pháp monitoring đầy đủ. Có nhiều giải pháp mã nguồn mở như Metrics Server, Prometheus, Elastic Stack, hoặc các giải pháp thương mại như Datadog, Dynatrace.
 
## Heapster và Metrics Server
- Heapster là dự án monitoring ban đầu của Kubernetes, hiện đã bị deprecated (ngừng phát triển) và được thay thế bằng một phiên bản tinh gọn hơn gọi là **`metrics server`**.

  ![hpms](../../images/hpms.PNG)
  
## Metrics Server

  ![ms1](../../images/ms1.PNG)

Mỗi Kubernetes cluster có thể chạy một Metrics Server. Metrics Server thu thập metrics từ từng node và Pod, tổng hợp lại và lưu **trong bộ nhớ (in-memory)**. Vì không lưu trên đĩa, bạn không thể xem dữ liệu hiệu năng trong quá khứ — muốn vậy cần dùng các giải pháp monitoring nâng cao (vd Prometheus).

#### Metrics của các Pod trên node được tạo ra như thế nào?

  ![ca](../../images/ca.PNG)

kubelet trên mỗi node có một thành phần con là **cAdvisor** (Container Advisor). cAdvisor chịu trách nhiệm thu thập các chỉ số hiệu năng từ Pod và cung cấp chúng qua API của kubelet, để Metrics Server có thể lấy về.
  
## Metrics Server - Bắt đầu sử dụng

  ![msg](../../images/msg.PNG)
  
- Clone metrics server từ repo GitHub
  ```
  $ git clone https://github.com/kubernetes-incubator/metrics-server.git
  ```
- Triển khai metrics server
  ```
  $ kubectl create -f metric-server/deploy/1.8+/
  ```

> 💡 **Bổ sung:** Repo `kubernetes-incubator/metrics-server` đã được chuyển sang `kubernetes-sigs/metrics-server`. Cách cài đặt hiện nay là áp dụng manifest từ bản release mới nhất: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`.
>
> Trên các cluster lab (kubeadm, kubelet dùng certificate tự ký) Metrics Server có thể không lấy được metrics do lỗi xác thực TLS; khi đó thường phải thêm tham số `--kubelet-insecure-tls` vào container `metrics-server` (chỉ dùng cho môi trường lab/thử nghiệm). Với Minikube: `minikube addons enable metrics-server`.
  
- Xem hiệu năng của các node trong cluster
  ```
  $ kubectl top node
  ```
- Xem metrics hiệu năng của các Pod
  ```
  $ kubectl top pod
  ```
  
  ![view](../../images/view.PNG)

> 💡 **Bổ sung:** Một số tùy chọn hữu ích của `kubectl top` khi thi:
> - `kubectl top pod -A` hoặc `kubectl top pod -n <namespace>`: xem Pod ở tất cả/một namespace.
> - `kubectl top pod --sort-by=cpu` hoặc `--sort-by=memory`: sắp xếp để tìm nhanh Pod tiêu thụ nhiều tài nguyên nhất.
> - `kubectl top pod --containers`: xem mức tiêu thụ theo từng container trong Pod.
> - `kubectl top node --sort-by=memory`.
>
> Metrics Server cũng là nguồn dữ liệu (Metrics API `metrics.k8s.io`) cho **HorizontalPodAutoscaler (HPA)** và **VerticalPodAutoscaler (VPA)** — xem thêm ở phần autoscaling.
  
  
