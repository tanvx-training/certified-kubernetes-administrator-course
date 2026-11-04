# Practice Test - Managing Application Logs (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-managing-application-logs/)
  
Lời giải cho bài practice test - managing application logs
- Chúng tôi đã triển khai một Pod chạy một ứng dụng. Hãy kiểm tra nó và đợi Pod khởi động.

  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Kiểm tra log của Pod
  
  <details>
  
  ```
  $ kubectl logs webapp-1
  ```

  > 💡 **Mẹo:** Đề thường hỏi người dùng nào gặp lỗi (vd bị khóa tài khoản). Có thể lọc nhanh bằng `kubectl logs webapp-1 | grep -i warning` hoặc `grep -i error`.
  </details>
  
- Chúng tôi đã triển khai một Pod mới - 'webapp-2' - chạy một ứng dụng. Hãy kiểm tra nó và đợi Pod khởi động.

  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Kiểm tra log của webapp trong Pod
  
  <details>
  
  ```
  $ kubectl logs webapp-2
  ```

  > 💡 **Bổ sung:** Pod `webapp-2` có nhiều hơn một container, nên cần chỉ định container cần xem log, ví dụ `kubectl logs webapp-2 -c simple-webapp`. Xem danh sách container của Pod bằng `kubectl describe pod webapp-2` hoặc `kubectl get pod webapp-2 -o jsonpath='{.spec.containers[*].name}'`.
  </details>
