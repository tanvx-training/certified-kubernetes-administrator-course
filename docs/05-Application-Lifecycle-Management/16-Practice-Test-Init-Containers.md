# Practice Test - Init-Containers (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-init-containers/)
  
Lời giải cho bài practice test - init-containers
- Xác định Pod có cấu hình initContainer.

  <details>
  ```
  $ kubectl get pods
  $ kubectl describe pods
  ```

  Tìm Pod có mục `Init Containers:` trong output của `kubectl describe`.
  </details>
  
- initContainer trên Pod blue dùng image gì?
  
  <details>
  ```
  $ kubectl describe pods blue
  ```
  </details>
    
- Chạy lệnh kubectl describe pod blue và kiểm tra trường State của initContainer.

  <details>
  ```
  $ kubectl describe pod blue
  ```
  </details>
  
- Kiểm tra trường Reason của initContainer
  
  <details>
  ```
  $ kubectl describe pod blue
  ```

  Init container đã chạy xong nên State là `Terminated` với Reason là `Completed`.
  </details>
  
- Chạy lệnh kubectl describe pod purple
  
  <details>
  ```
  $ kubectl describe pod purple
  ```
  </details>
  
- Chạy lệnh kubectl describe pod purple và xem trạng thái (State) của container
  
  <details>
  ```
  $ kubectl describe pod purple
  ```

  Container chính đang ở trạng thái `Waiting` (Reason `PodInitializing`) vì các init container chưa chạy xong.
  </details>
  
- Kiểm tra các command được dùng trong initContainers. Cái đầu tiên sleep 600 giây (10 phút) và cái thứ hai sleep 1200 giây (20 phút)
  
  <details>
  ```
  $ kubectl describe pod purple
  ```

  > 💡 **Giải thích:** Init container chạy **tuần tự**, nên thời gian chờ là tổng của hai cái: 10 + 20 = 30 phút trước khi ứng dụng chính khởi động.
  </details>
  
- Cập nhật Pod red để dùng một initContainer dùng image busybox và sleep 20 giây
  
  <details>
  ```
  $ kubectl get pod red -o yaml > red.yaml
  $ kubectl delete pod red
  ```
  </details>
  
  Cập nhật red.yaml, thêm initContainer với lệnh sleep 20 giây
  
  <details>
  ```
  $ kubectl create -f red.yaml
  ```
  </details>

  > 💡 **Gợi ý:** Phần cần thêm vào `spec` của red.yaml: `initContainers:` với một phần tử có `name: red-initcontainer`, `image: busybox`, `command: ["sleep", "20"]`. Nhớ xóa phần `status` trong file xuất ra trước khi tạo lại (hoặc dùng `kubectl replace --force -f red.yaml`).
  
- Kiểm tra command được dùng bởi initContainer. Có vẻ có lỗi gõ (typo) trong lệnh sleep. Hãy sửa lại: phải là **`sleep 2`** chứ không phải **`sleeeep 2`**
  
  <details>
  ```
  $ kubectl describe pod orange
  $ kubectl get pod orange -o yaml > orange.yaml
  $ kubectl delete pod orange
  
  Cập nhật orange.yaml với lệnh sleep đúng rồi tạo lại Pod
  $ kubectl create -f orange.yaml
  ```

  > 💡 **Mẹo:** Dấu hiệu nhận biết: Pod ở trạng thái `Init:Error` hoặc `Init:CrashLoopBackOff`. Xem nguyên nhân bằng `kubectl logs orange -c <tên-init-container>` (sẽ thấy lỗi kiểu `sleeeep: not found`).
 </details>
