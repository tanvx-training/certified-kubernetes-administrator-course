# Practice Test - Image Security (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-image-security/)

Lời giải cho bài practice test - Image Security
- Chúng ta có một ứng dụng đang chạy trên cluster. Hãy tìm hiểu nó trước. Ứng dụng đang dùng image nào?

  <details>
  
  ```
  $ kubectl get deploy -o wide
  ```
  
  </details>
  
- Dùng lệnh kubectl edit deployment để sửa tên image thành myprivateregistry.com:5000/nginx:alpine

  <details>
  
  ```
  $ kubectl edit deployment web
  ```
  
  </details>

  > 💡 Cách nhanh hơn: `kubectl set image deployment/web nginx=myprivateregistry.com:5000/nginx:alpine` (thay `nginx` bằng đúng tên container trong Deployment, xem bằng `kubectl get deploy web -o wide`).
  
- Chạy lệnh kubectl get pods và kiểm tra trạng thái các Pod

  <details>
  
  ```
  $ kubectl get pods
  ```
  
  </details>

  > 💡 Pod mới sẽ ở trạng thái `ErrImagePull`/`ImagePullBackOff` vì chưa có thông tin đăng nhập vào private registry. Xem chi tiết lỗi bằng `kubectl describe pod <tên-pod>`.
  
- Chạy lệnh kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
  
  <details>
  
  ```
  $ kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
  ```
  
  </details>
  
- Sửa Deployment bằng lệnh kubectl edit deploy web và thêm mục imagePullSecrets. Dùng Secret private-reg-cred
  
  <details>
  
  ```
  $ kubectl edit deploy web
  ```
  
  </details>

  > 💡 Thêm vào `spec.template.spec` (ngang hàng với `containers`) khối `imagePullSecrets:` với một phần tử `- name: private-reg-cred`. Sau khi lưu, Deployment sẽ tự rollout tạo Pod mới.
  
- Kiểm tra trạng thái các Pod. Chờ đến khi chúng ở trạng thái Running. Bạn đã cấu hình thành công Deployment để pull image từ private registry
  
  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
