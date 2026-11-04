# Practice Test - Cluster Upgrade Process (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-cluster-upgrade-process/)
  
Lời giải cho bài practice test - quy trình upgrade cluster
- Phiên bản hiện tại của cluster là gì?
  
  <details>
  ```
  $ kubectl get nodes
  ```
  </details>
  
- Có bao nhiêu node thuộc cluster này?
  
  <details>
  ```
  $ kubctl get nodes
  ```
  </details>

  > Lưu ý: lệnh đúng là `kubectl get nodes` (bản gốc gõ nhầm `kubctl`).
  
- Kiểm tra xem các Pod đang được host trên những node nào.
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
  
- Đếm số lượng Deployment
  
  <details>
  ```
  $ kubectl get deploy
  ```
  </details>
  
- Chạy lệnh kubectl get pods -o wide
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
  
- Bạn được giao nhiệm vụ upgrade cluster. Người dùng đang truy cập ứng dụng không được bị ảnh hưởng, và bạn không thể tạo thêm VM mới. Bạn sẽ dùng chiến lược nào để upgrade cluster?
  
  <details>
  ```
  Upgrade one node at a time while moving the workloads to the other
  ```
  </details>

  > 💡 **Giải thích:** Upgrade lần lượt từng node, đồng thời dời workload sang node khác. Upgrade tất cả cùng lúc sẽ gây downtime, còn thêm node mới thì không được phép vì không thể tạo VM mới.
  
- Chạy lệnh kubeadm upgrade plan
  
  <details>
  ```
  $ kubeadm upgrade plan
  ```
  </details>
  
- Chạy kubectl drain master --ignore-daemonsets
  
  <details>
  ```
  $ kubectl drain master --ignore-daemonsets
  ```
  </details>
  
- Chạy lệnh apt install kubeadm=1.18.0-00, sau đó kubeadm upgrade apply v1.18.0, rồi apt install kubelet=1.18.0-00 để upgrade kubelet trên master node
  
  <details>
  ```
  $ apt install kubeadm=1.18.0-00
  $ kubeadm upgrade apply v1.18.0 
  $ apt install kubelet=1.18.0-00
  ```
  </details>

  > 💡 **Bổ sung:** Sau khi cài kubelet mới, nên chạy `systemctl daemon-reload && systemctl restart kubelet` để kubelet chạy phiên bản mới. Với lab hiện tại (v1.30+), phiên bản gói có dạng `1.31.0-1.1` và phải đổi repo `pkgs.k8s.io` sang minor version mới trước (xem chi tiết ở [Giới thiệu về Upgrade Cluster](05-Cluster-Upgrade-Introduction.md)). Ví dụ:
  > ```
  > apt-get update
  > apt-cache madison kubeadm
  > apt-get install -y kubeadm=1.31.0-1.1
  > kubeadm upgrade plan
  > kubeadm upgrade apply v1.31.0
  > apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1
  > systemctl daemon-reload
  > systemctl restart kubelet
  > ```
  
- Chạy lệnh kubectl uncordon master
  
  <details>
  ```
  $ kubectl uncordon master 
  ```
  </details>
  
- Chạy lệnh kubectl drain node01 --ignore-daemonsets
  
  <details>
  ```
  $ kubectl drain node01 --ignore-daemonsets
  ```
  </details>
  
- Chạy các lệnh: apt install kubeadm=1.18.0-00, sau đó kubeadm upgrade node. Cuối cùng chạy apt install kubelet=1.18.0-00.
  
  <details>
  ```
  $ apt install kubeadm=1.18.0-00
  $ kubeadm upgrade node
  $ apt install kubelet=1.18.0-00
  ```
  </details>

  > 💡 **Lưu ý:** Các lệnh này phải chạy **trên node01** (SSH vào: `ssh node01`), không phải trên master. Tương tự, nhớ `systemctl daemon-reload && systemctl restart kubelet` sau khi cài kubelet, rồi `exit` để quay lại control plane trước khi chạy `kubectl uncordon node01`.
  
- Chạy lệnh kubectl uncordon node01
  
  <details>
  ```
  $ kubectl uncordon node01
  ```
  </details>

