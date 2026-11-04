# Practice Test - Rolling Updates and Rollback (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-rolling-updates-and-rollbacks/)
  
Lời giải cho bài practice test - rolling updates and rollback
- Chúng tôi đã triển khai một ứng dụng web đơn giản. Hãy kiểm tra các Pod và Service.

  <details>
  
  ```
  $ kubectl get pods
  $ kubectl get services
  ```
  </details>
  
- Màu hiện tại của ứng dụng web là gì?
  
  <details>
  
  ```
  Access the web portal
  ```

  Truy cập giao diện web (web portal) của ứng dụng để xem màu.
  </details>
    
- Chạy script tại /root/curl-test.sh.

- Chạy lệnh 'kubectl describe deployment' và xem trường 'Desired Replicas'.

  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Chạy lệnh 'kubectl describe deployment' và tìm trường 'Images'.
  
  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Chạy lệnh 'kubectl describe deployment' và xem trường 'StrategyType'.
  
  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Nếu nâng cấp ứng dụng ngay lúc này thì điều gì sẽ xảy ra?
  
  <details>
  
  ```
  PODs are upgraded few at a time
  ```

  Các Pod sẽ được nâng cấp lần lượt từng ít một (vì StrategyType là RollingUpdate).
  </details>
  
- Chạy lệnh 'kubectl edit deployment frontend' và sửa trường cần thiết.
  
  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```

  > 💡 **Mẹo:** Để đổi image nhanh hơn, có thể dùng `kubectl set image deployment/frontend <tên-container>=<image-mới>` (xem tên container bằng `kubectl describe deployment frontend`), sau đó theo dõi bằng `kubectl rollout status deployment/frontend`.
  </details>
    
- Chạy script tại /root/curl-test.sh.

- Xem giá trị Max Unavailable trong phần RollingUpdateStrategy ở thông tin chi tiết của Deployment.

  <details>
  ```
  $ kubectl describe deployment
  ```

  > 💡 **Giải thích:** `maxUnavailable` là số Pod tối đa có thể không sẵn sàng cùng lúc trong quá trình update. Nếu giá trị là phần trăm (vd 25% với 4 replicas) thì kết quả được làm tròn **xuống** (4 × 25% = 1 Pod). Ngược lại, `maxSurge` được làm tròn **lên**.
  </details>
  
- Chạy lệnh 'kubectl edit deployment frontend' và sửa trường cần thiết. Nhớ xóa luôn các thuộc tính của rollingUpdate tại 'strategy.rollingUpdate'.
  
  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```

  > 💡 **Giải thích:** Đổi `spec.strategy.type` thành `Recreate` và xóa toàn bộ khối `rollingUpdate` (chứa `maxSurge`/`maxUnavailable`). Nếu giữ lại khối này, API server sẽ từ chối với lỗi kiểu `spec.strategy.rollingUpdate: Forbidden: may not be specified when strategy type is 'Recreate'`.
  
  </details>
  
- Chạy lệnh 'kubectl edit deployment frontend' và sửa trường cần thiết.

  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```
  </details>
  
- Chạy script tại /root/curl-test.sh.

  > 💡 **Giải thích:** Vì chiến lược bây giờ là Recreate, tất cả Pod cũ bị xóa trước khi Pod mới được tạo, nên trong một khoảng thời gian script sẽ báo lỗi do ứng dụng không truy cập được (downtime).




