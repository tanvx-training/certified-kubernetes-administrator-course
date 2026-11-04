# Practice Test - RBAC (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-role-based-access-controls/)

Lời giải cho bài practice test - RBAC
- Chạy lệnh kubectl describe pod kube-apiserver-controlplane -n kube-system và tìm tham số --authorization-mode
  
  <details>
  
  ```
  $ kubectl describe pod kube-apiserver-controlplane -n kube-system
  ```
  
  </details>

  > 💡 Mẹo: có thể xem trực tiếp static Pod manifest bằng `grep authorization-mode /etc/kubernetes/manifests/kube-apiserver.yaml`. Kết quả thường là `Node,RBAC`.
  
- Chạy lệnh kubectl get roles

  <details>
  
  ```
  $ kubectl get roles
  ```
  
  </details>
  
- Chạy lệnh kubectl get roles --all-namespaces
  
  <details>
  
  ```
  $ kubectl get roles --all-namespaces
  ```
  
  </details>

  > 💡 Có thể viết gọn là `kubectl get roles -A` và thêm `--no-headers | wc -l` để đếm.
  
- Chạy lệnh kubectl describe role kube-proxy -n kube-system
  
  <details>
  
  ```
  $ kubectl describe role kube-proxy -n kube-system
  ```
  
  </details>
  
- Kiểm tra các verb gắn với Role kube-proxy
  
  <details>
  ```
  $ kubectl describe role kube-proxy -n kube-system
  ```
  </details>
  
- Phát biểu nào sau đây là đúng?
  
  <details>
  ```
  kube-proxy role can get details of configmap object by the name kube-proxy
  ```
  </details>

  > 💡 Giải thích: Role kube-proxy chỉ có verb `get` trên resource `configmaps` với `resourceNames` là `kube-proxy`, nên nó chỉ đọc được đúng ConfigMap tên `kube-proxy` — không thể xóa hay sửa, cũng không đọc được ConfigMap khác.
  
- Chạy lệnh kubectl describe rolebinding kube-proxy -n kube-system
  
  <details>
  ```
  $ kubectl describe rolebinding kube-proxy -n kube-system
  ```
  </details>
  
- Chạy lệnh kubectl get pods --as dev-user
  
  <details>
  ```
  $ kubectl get pods --as dev-user
  ```
  </details>

  > 💡 Lệnh sẽ báo lỗi `Forbidden` vì dev-user chưa được cấp quyền. Cũng có thể kiểm tra bằng `kubectl auth can-i list pods --as dev-user`.
  
- File đáp án nằm tại /var/answers
  
  <details>
  
  ```
  $ kubectl create -f /var/answers/developer-role.yaml
  ```
  
  </details>

  > 💡 Cách làm imperative tương đương (yêu cầu: Role `developer` và RoleBinding `dev-user-binding` trong namespace `default`, cho phép list/create/delete Pod):
  > - `kubectl create role developer --verb=list,create,delete --resource=pods`
  > - `kubectl create rolebinding dev-user-binding --role=developer --user=dev-user`
  
- Role và RoleBinding mới đã được tạo trong namespace blue. Hãy kiểm tra chúng. Xem resourceNames được cấu hình trên Role
  
  <details>
  
  ```
  $ kubectl get roles,rolebindings -n blue
  $ kubectl describe role developer -n blue
  $ kubectl edit role developer -n blue (update the resourceNames)
  ```
  
  </details>

  > 💡 Role đang cho phép truy cập một Pod có tên khác với Pod đang chạy trong namespace blue. Sửa `resourceNames` thành đúng tên Pod (xem bằng `kubectl get pods -n blue`).
  
- Xem file đáp án tại /var/answers/dev-user-deploy.yaml
  
  <details>
  
  ```
  $ kubectl create -f /var/answers/dev-user-deploy.yaml
  ```
  
  </details>

  > 💡 Điểm mấu chốt: Deployment thuộc API group `apps`, nên phải thêm một rule mới với `apiGroups: ["apps"]`, `resources: ["deployments"]` vào Role developer trong namespace blue — không thể dùng chung rule có `apiGroups: [""]`.
