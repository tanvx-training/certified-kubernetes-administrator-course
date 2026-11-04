# Practice Test - Resource Limits (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-resource-limits/)
  
Lời giải cho bài practice test - Resource Limits
- Chạy lệnh 'kubectl describe pod rabbit' và xem phần requests.
  
  <details>

  ```
  $ kubectl describe pod rabbit
  ```
  </details>

- Chạy lệnh 'kubectl delete pod rabbit'.

  <details>

  ```
  $ kubectl delete pod rabbit
  ```
  </details>

- Chạy lệnh 'kubectl get pods' và kiểm tra trạng thái của Pod elephant

  <details>

  ```
  $ kubectl get pods
  ```
  </details>

- Trạng thái 'OOMKilled' cho biết Pod đã dùng hết memory cho phép (vượt memory limit). Xác định memory limit đang đặt trên Pod (xem phần `Limits` và `Last State: Terminated, Reason: OOMKilled` trong `kubectl describe pod elephant`).

- Tạo file template từ Pod đang tồn tại.

  <details>

  ```
  $ kubectl get pods elephant -o yaml > elephant.yaml
  ```
  </details>

  Sửa file định nghĩa Pod elephant.yaml, đặt memory limit thành 20Mi
  
  <details>

  ```
  resources:
      limits:
        memory: 20Mi
  ---
  ```
  </details>

  Xóa Pod và tạo lại.
  
  <details>

  ```
  $ kubectl delete pod elephant
  $ kubectl create -f elephant.yaml
  ```
  </details>

  > 💡 **Mẹo:** Hầu hết các trường trong spec của Pod đang chạy không sửa được (kể cả `resources`, trừ khi dùng in-place resize từ v1.33). Khi `kubectl edit pod` báo lỗi, kubectl sẽ lưu bản sửa vào một file tạm trong `/tmp/`; có thể dùng `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml` để xóa và tạo lại Pod chỉ bằng một lệnh.

- Kiểm tra trạng thái của Pod. Đảm bảo Pod đang ở trạng thái Running
  
  <details>

  ```
  $ kubectl get pods
  ```
  </details>

- Chạy lệnh 'kubectl delete pod elephant'.

  <details>

  ```
  $ kubectl delete pod elephant
  ```
  </details>



