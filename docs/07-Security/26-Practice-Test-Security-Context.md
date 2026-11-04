# Practice Test - Security Context (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-security-contexts/)
  
Lời giải cho bài practice test - Security Context
- Chạy lệnh 'kubectl exec ubuntu-sleeper -- whoami' để xem user đang chạy tiến trình trong Pod (bản gốc ghi nhầm là "đếm số Pod").

  <details>
  
  ```
  $ kubectl exec ubuntu-sleeper whoami
  ```
  
  </details>

  > 💡 Cú pháp `kubectl exec <pod> <lệnh>` (không có `--`) đã bị loại bỏ trong các bản kubectl mới; hãy dùng `kubectl exec ubuntu-sleeper -- whoami`. Kết quả là `root`.
  
- Đặt security context để chạy với user 1010.

  <details>
  
  ```
  $ kubectl get pods ubuntu-sleeper -o yaml > ubuntu.yaml
  $ kubectl delete pod ubuntu-sleeper
  $ vi ubuntu.yaml ( add securityContext Section)
    securityContext:
      runAsUser: 1010
  $ kubectl create -f ubuntu.yaml
  ```
  
  </details>

  > 💡 Có thể gộp bước xóa và tạo lại: sau khi sửa file, chạy `kubectl replace --force -f ubuntu.yaml`.
  
- User ID được khai báo trong securityContext của container sẽ ghi đè User ID khai báo ở cấp Pod.
 
- User ID được khai báo trong securityContext của Pod sẽ được áp dụng cho tất cả container trong Pod (bản gốc viết nhầm "all the PODs in the container").

- Chạy lệnh kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
  
  <details>
  
  ```
  $ kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
  ```
  
  </details>

  > 💡 Lệnh sẽ thất bại (`Operation not permitted`) vì container mặc định không có capability `SYS_TIME`, kể cả khi chạy bằng root.
  
- Thêm capability SYS_TIME vào securityContext của container
  
  <details>
  
  ```
  $ kubectl get pods ubuntu-sleeper -o yaml > ubuntu.yaml
  $ kubectl delete pod ubuntu-sleeper
  $ vi ubuntu.yaml
  
  Under container section add the below
  
  securityContext:
      capabilities:
        add: ["SYS_TIME"]
        
  $ kubectl create -f ubuntu.yaml
  ```
  
  </details>

  > 💡 Lưu ý: `capabilities` phải đặt trong securityContext **của container** (`spec.containers[].securityContext`), không phải ở cấp Pod.
  
 - Bây giờ hãy chạy lệnh dưới đây trong Pod để đặt ngày giờ. Nếu capability đã được thêm đúng, lệnh sẽ chạy thành công. Nếu không, hãy chắc chắn rằng bạn đã đổi user về lại root.
  
   <details>
  
   ```
   $ kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
   ```
  
   </details>
   
