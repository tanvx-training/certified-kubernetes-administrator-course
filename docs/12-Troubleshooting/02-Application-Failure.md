# Application Failure (Lỗi ứng dụng)
  
  - Xem [Bài giảng](https://kodekloud.com/topic/application-failure/)

  - Trong bài giảng này, chúng ta sẽ đi từng bước để troubleshoot lỗi ứng dụng. Cách tiếp cận chung: bắt đầu từ phía người dùng (front-end) rồi lần ngược dần về phía sau (Service → Pod → database Service → database Pod).

  - Kiểm tra trạng thái của ứng dụng/Service web server (truy cập thử qua NodePort)

    ```
    curl http://web-service-ip:node-port
    ```

    ![app](../../images/app.PNG)

  - Kiểm tra endpoint của Service và so sánh với selector (selector của Service phải khớp với label của Pod)

    ```
    kubectl describe service web-service
    ```   

    ![svc](../../images/svc.PNG)

    > 💡 **Bổ sung:** Nếu trường `Endpoints` trống (`<none>`), gần như chắc chắn selector của Service không khớp với label của Pod nào, hoặc Pod chưa ở trạng thái Ready. Có thể kiểm tra nhanh bằng `kubectl get endpoints web-service` và `kubectl get pods --show-labels`. Ngoài ra hãy so sánh `targetPort` của Service với `containerPort` mà ứng dụng thực sự lắng nghe.


  - Kiểm tra trạng thái và log của Pod

    ```
    kubectl get pod
    ```

    ```
    kubectl describe pod web
    ```

    ```
    kubectl logs web
    ```

  - Kiểm tra log của Pod ở lần chạy trước (hữu ích khi container đã bị restart, vd trạng thái `CrashLoopBackOff`)

    ```
    kubectl logs web -f --previous
    ```
    
    ![db](../../images/db.PNG)

  > 💡 **Bổ sung — các lệnh hay dùng khi troubleshoot ứng dụng:**
  > - `kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp`: xem các event gần nhất (lỗi pull image, lỗi schedule, lỗi mount volume...).
  > - Phần `Events` ở cuối output của `kubectl describe pod` thường chỉ ra nguyên nhân trực tiếp: `ImagePullBackOff`/`ErrImagePull` (sai tên image/tag hoặc thiếu quyền registry), `Pending` (không đủ tài nguyên, taint, node affinity, PVC chưa bound), `CrashLoopBackOff` (ứng dụng chết ngay khi khởi động — xem `kubectl logs --previous`), `OOMKilled` (vượt memory limit).
  > - Pod có nhiều container thì cần chỉ định container: `kubectl logs web -c <container-name>`.
  > - Kiểm tra kết nối/DNS từ bên trong cluster: `kubectl exec -it web -- sh` hoặc tạo Pod tạm `kubectl run tmp --rm -it --image=busybox -- sh`, rồi thử `nslookup mysql-service` hay `wget -qO- http://web-service:8080`.
  > - Với Pod do Deployment quản lý, hãy sửa Deployment (`kubectl edit deployment ...`) thay vì sửa Pod; với Pod độc lập, nhiều trường không thể sửa trực tiếp — dùng `kubectl replace --force -f pod.yaml`.


  #### Lab thực hành

  - Cùng troubleshoot [Ứng dụng](https://kodekloud.com/topic/practice-test-application-failure/)