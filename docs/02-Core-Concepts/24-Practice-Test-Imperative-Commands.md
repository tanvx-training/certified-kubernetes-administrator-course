# Practice Test - Imperative Commands (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-imperative-commands-2/)

Lời giải cho bài Practice Test - Imperative Commands

1. Thông tin (không cần trả lời)

1.  <details>
    <summary>Deploy một Pod tên nginx-pod sử dụng image nginx:alpine.</summary>

    ```
    kubectl run nginx-pod --image=nginx:alpine
    ```
    </details>

1.  <details>
    <summary>Deploy một Pod redis sử dụng image redis:alpine với label tier=db.</summary>

    ```
    kubectl run redis --image=redis:alpine -l tier=db
    ```

    > 💡 **Bổ sung:** `-l` là viết tắt của `--labels`. Nhiều label thì phân tách bằng dấu phẩy, ví dụ `--labels=tier=db,app=redis`.
    </details>

1.  <details>
    <summary>Tạo Service redis-service để expose ứng dụng redis bên trong cluster trên port 6379.</summary>

    ```
    kubectl expose pod redis --port=6379 --name redis-service
    ```

    > 💡 **Bổ sung:** Dùng `kubectl expose` ở đây tốt hơn `kubectl create service clusterip`, vì `expose` tự lấy label của Pod (`tier=db`) làm selector, còn `create service` sẽ đặt selector là `app=redis-service` và không khớp Pod nào. Loại Service mặc định là ClusterIP.
    </details>

1.  <details>
    <summary>Tạo một Deployment tên webapp sử dụng image kodekloud/webapp-color với 3 replica.</summary>

    ```
    kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
    ```
    </details>

1.  <details>
    <summary>Tạo một Pod mới tên custom-nginx sử dụng image nginx và expose nó trên container port 8080.</summary>

    ```
    kubectl run custom-nginx --image=nginx --port=8080
    ```

    > 💡 **Bổ sung:** `--port` chỉ khai báo `containerPort` trong spec của Pod (mang tính thông tin), không tạo Service.
    </details>

1.  <details>
    <summary>Tạo một namespace mới tên dev-ns.</summary>

    ```
    kubectl create ns dev-ns
    ```
    </details>

1.  <details>
    <summary>Tạo một Deployment mới tên redis-deploy trong namespace dev-ns với image redis. Deployment phải có 2 replica.</summary>

    ```
    kubectl create deployment redis-deploy -n dev-ns --image redis --replicas 2
    ```
    </details>

1.  <details>
    <summary>Tạo một Pod tên httpd sử dụng image httpd:alpine trong namespace default.</br>Tiếp theo, tạo một Service loại ClusterIP cùng tên (httpd).</br>Target port của Service phải là 80.</summary>

    ```
    kubectl run httpd --image httpd:alpine --expose --port 80
    ```

    > 💡 **Bổ sung:** Flag `--expose` khiến `kubectl run` tạo thêm một Service ClusterIP cùng tên với Pod, dùng label `run=httpd` của Pod làm selector, với `port` (và `targetPort`) bằng giá trị `--port`. Một lệnh làm được cả hai việc.
    </details>


