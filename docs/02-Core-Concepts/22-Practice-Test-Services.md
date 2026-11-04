# Practice Test - Services (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-services/)

#### Lời giải cho bài Practice Test - Services

1. <details>
    <summary>Có bao nhiêu Service trên hệ thống?</summary>

    ```
    kubectl get services
    ```

    Đếm số Service hiển thị (nếu có).
    </details>


1. Chỉ là thông tin (không cần trả lời)

1.  <details>
    <summary>Service mặc định kubernetes thuộc loại (type) nào?</summary>

    Từ output của câu 1, xem cột `TYPE`.

    > 💡 **Bổ sung:** Service `kubernetes` trong namespace `default` được tạo tự động, có loại `ClusterIP`. Nó trỏ tới kube-apiserver, giúp các Pod trong cluster gọi được API server qua địa chỉ `kubernetes.default.svc`.
    </details>

1.  <details>
    <summary>targetPort được cấu hình trên Service kubernetes là bao nhiêu?</summary>

    ```
    $ kubectl describe service | grep TargetPort
    ```

    > 💡 **Bổ sung:** Ở đây chỉ có một Service nên lệnh trên đủ dùng; khi có nhiều Service, hãy chỉ rõ tên: `kubectl describe service kubernetes`. Giá trị thường là `6443/TCP` - port mà kube-apiserver lắng nghe.
    </details>

1.  <details>
    <summary>Có bao nhiêu label được cấu hình trên Service kubernetes?</summary>

    ```
    kubectl describe service
    ```

    ...rồi tìm phần labels.

    --- HOẶC ---

    ```
    kubectl get service --show-labels
    ```

    </details>

1.  <details>
    <summary>Có bao nhiêu Endpoint được gắn vào Service kubernetes?</summary>

    ```
    kubectl describe service
    ```

    ...rồi tìm phần endpoints.

    > 💡 **Bổ sung:** Endpoint là địa chỉ `IP:port` của các đích mà Service chuyển traffic tới (với Service thông thường là các Pod khớp selector). Nếu một Service có `Endpoints: <none>`, gần như chắc chắn selector không khớp label của Pod nào - đây là lỗi rất hay gặp khi troubleshooting. Có thể xem nhanh bằng `kubectl get endpoints` hoặc `kubectl get endpointslices`.

    </details>

1.  <details>
    <summary>Bây giờ có bao nhiêu Deployment trên hệ thống?</summary>

    ```
    kubectl get deployment
    ```

    Đếm số Deployment hiển thị (nếu có).
    </details>

1.  <details>
    <summary>Image nào được dùng để tạo các Pod trong Deployment?</summary>

    ```
    kubectl describe deployment
    ```

    Xem trong mục containers.

    --- HOẶC ---

    ```
    kubectl get deployment -o wide
    ```

    Xem cột `IMAGES`.

    </details>

1.  <details>
    <summary>Bạn có truy cập được giao diện Web App không?</summary>

    Thử truy cập giao diện Web Application bằng tab simple-webapp-ui phía trên terminal.

    </details>

1.  <details>
    <summary>Tạo một Service mới để truy cập ứng dụng web, sử dụng file service-definition-1.yaml.</summary>

    ```
    vi service-definition-1.yaml
    ```

    Điền các giá trị theo yêu cầu, lưu và thoát.

    ```
    kubectl create -f service-definition-1.yaml
    ```

    > 💡 **Bổ sung:** Các giá trị cần điền thường gồm `type: NodePort`, `port`, `targetPort`, `nodePort` và `selector` khớp với label của Pod trong Deployment (xem bằng `kubectl get pods --show-labels` hoặc `kubectl describe deployment`). Sau khi tạo, kiểm tra bằng `kubectl describe service <tên-service>` để chắc chắn mục `Endpoints` không rỗng.
    </details>

1. Kiểm tra Service vừa được deploy.

