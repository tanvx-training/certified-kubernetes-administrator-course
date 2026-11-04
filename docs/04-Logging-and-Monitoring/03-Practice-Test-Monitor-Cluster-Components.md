# Practice Test - Monitor Cluster Components (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-monitor-cluster-components/)
  
Lời giải cho bài practice test - monitor cluster components
1.  <details>
    <summary>Chúng tôi đã triển khai một vài Pod đang chạy workload. Hãy kiểm tra chúng.</summary>

    ```
    kubectl get pods
    ```
    </details>
  
1.  <details>
    <summary>Hãy triển khai metrics-server để giám sát các Pod và Node. Pull git repository chứa các file triển khai.</summary>

    ```
    git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
    ```
    </details>
  
1.  <details>
    <summary>Triển khai metrics-server bằng cách tạo tất cả các thành phần vừa tải về.</summary>
    
    Chạy lệnh 'kubectl create -f .' từ bên trong thư mục repository vừa tải về.
  
    ```
    cd kubernetes-metrics-server
    kubectl create -f .
    ```

    > 💡 **Bổ sung:** `kubectl create -f .` sẽ tạo mọi manifest trong thư mục hiện tại (ServiceAccount, RBAC, Deployment, Service, APIService...). Sau đó có thể kiểm tra bằng `kubectl get pods -n kube-system | grep metrics-server`.
    </details>
    
1.  <details>
    <summary>Metrics server cần vài phút để bắt đầu thu thập dữ liệu.</summary>

    Chạy lệnh `kubectl top node` và đợi cho đến khi có output hợp lệ.
    
    ```
    kubectl top node
    ```

    > 💡 **Bổ sung:** Nếu nhận được lỗi kiểu `error: metrics not available yet` thì chỉ cần đợi thêm khoảng 1 phút rồi chạy lại.
    </details>
  
1.  <details>
    <summary>Xác định node tiêu thụ nhiều CPU(cores) nhất.</summary>

     Chạy lệnh `kubectl top node`

      ```
      kubectl top node
      ```

      Xem cột `CPU(cores)` trong output để tìm đáp án.

      </details>
  
1.  <details>
    <summary>Xác định node tiêu thụ nhiều Memory(bytes) nhất.</summary>
    Chạy lệnh `kubectl top node`
  
    ```
    $ kubectl top node
    ```

    Xem cột `MEMORY(bytes)` trong output để tìm đáp án.

    </details>
  
1.  <details>
    <summary>Xác định Pod tiêu thụ nhiều Memory(bytes) nhất.</summary>

    Chạy lệnh `kubectl top pod`
  
    ```
    kubectl top pod
    ```

    Xem cột `MEMORY(bytes)` trong output để tìm đáp án.

    > 💡 **Mẹo thi:** Dùng `kubectl top pod --sort-by=memory` để Pod tiêu thụ nhiều memory nhất hiện ngay ở dòng đầu.

    </details>
  
1.  <details>

    <summary>Xác định Pod tiêu thụ ít CPU(cores) nhất.</summary>

    Chạy lệnh `kubectl top pod`
  
    ```
    kubectl top pod
    ```

    Xem cột `CPU(cores)` trong output để tìm đáp án.

    > 💡 **Mẹo thi:** `kubectl top pod --sort-by=cpu` sắp xếp giảm dần theo CPU, nên Pod tiêu thụ ít CPU nhất nằm ở dòng cuối cùng.

  </details>
  


