# Practice Test - OS Upgrades (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-os-upgrades/)
  
Lời giải cho bài practice test - OS Upgrades
- Hãy khám phá môi trường trước. Cluster có bao nhiêu node?
  
  <details>
  ```
  $ kubectl get nodes
  ```
  </details>
  
- Có bao nhiêu ứng dụng đang được host trên cluster?
  
  <details>
  ```
  $ kubectl get deploy
  ```
  </details>
  
- Chạy lệnh 'kubectl get pods -o wide' và xem danh sách các node mà Pod đang được đặt lên
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
  
- Chạy lệnh kubectl drain node01 --ignore-daemonsets
  
  <details>
  ```
  $ kubectl drain node01 --ignore-daemonsets
  ```
  </details>

  > 💡 **Bổ sung:** Cần `--ignore-daemonsets` vì trên node có các Pod của DaemonSet (vd kube-proxy, CNI như flannel/calico). Pod DaemonSet không thể "dời" đi đâu được nên drain sẽ báo lỗi nếu thiếu flag này.
  
- Chạy lệnh 'kubectl get pods -o wide' và xem danh sách các node mà Pod đang được đặt lên
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
  
- Chạy lệnh kubectl uncordon node01
  
  <details>
  ```
  $ kubectl uncordon node01
  ```
  </details>
  
- Chạy lệnh kubectl get pods -o wide
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
  
- Tại sao không có Pod nào trên node01?
  
  <details>
  ```
  Only when new pods are created they will be scheduled
  ```
  </details>

  > 💡 **Giải thích:** Uncordon chỉ cho phép schedule lại lên node01, chứ không di chuyển các Pod đang chạy về node01. Chỉ khi có Pod mới được tạo (scale up, rollout, Pod bị xóa và tạo lại...) thì scheduler mới có thể đặt Pod lên node01.
  
- Dùng lệnh kubectl describe node master và xem mục Taints để kiểm tra node này có taint nào không.
  
  <details>
  ```
  $ kubectl describe node master
  ```
  </details>

  > 💡 **Bổ sung:** Trên cluster kubeadm hiện tại, node control plane thường mang taint `node-role.kubernetes.io/control-plane:NoSchedule` (taint cũ `node-role.kubernetes.io/master` đã bị loại bỏ từ v1.25). Nếu không có taint này, Pod ứng dụng vẫn có thể được schedule lên node control plane. Lệnh nhanh: `kubectl describe node <tên-node> | grep -i taint`.
  
- Chạy lệnh kubectl drain node02 --ignore-daemonsets
  
  <details>
  ```
  $ kubectl drain node02 --ignore-daemonsets
  ```
  </details>
  
- Kiểm tra các ứng dụng được host trên node02.
  
  <details>
  ```
  node02 has a pod not part of a replicaset
  $ kubectl get pods -o wide
  ```
  </details>

  > 💡 **Giải thích:** Lệnh drain ở trên thất bại vì node02 có một Pod không thuộc ReplicaSet nào (Pod tạo trực tiếp). Kubernetes từ chối drain để tránh làm mất Pod đó mà không có gì tạo lại.
  
- Kiểm tra danh sách Pod
  
  <details>
  ```
  $ kubectl get pods -o wide
  ```
  </details>
    
- Điều gì sẽ xảy ra với hr-app nếu node02 bị drain cưỡng bức (forcefully)?
  
  <details>
  ```
  $ kubectl drain node02 --ignore-daemonsets --force
  hr-app will be lost forever
  ```
  </details>

  > 💡 **Giải thích:** Với `--force`, Pod `hr-app` (không có controller quản lý) sẽ bị xóa và không được tạo lại ở đâu cả, tức là mất vĩnh viễn.
    
- Chạy lệnh kubectl drain node02 --ignore-daemonsets --force

  <details>
  ```
  $ kubectl drain node02 --ignore-daemonsets --force
  ```
  </details>
  
- Chạy lệnh kubectl cordon node03
  
  <details>
  ```
  $ kubectl cordon node03
  ```
  </details>

  > 💡 **Giải thích:** Cordon chỉ ngăn Pod mới được schedule lên node03 (vd để bảo vệ ứng dụng quan trọng đang chạy trên đó khỏi bị ảnh hưởng bởi Pod mới), còn các Pod hiện có vẫn tiếp tục chạy bình thường.

