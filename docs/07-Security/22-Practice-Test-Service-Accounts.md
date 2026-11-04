# Practice Test - Service Accounts (Bài thực hành)
  - Đi tới [Practice Test](https://uklabs.kodekloud.com/topic/practice-test-service-accounts-2/)

Lời giải cho bài practice test - Service Accounts
1.  <details>
    <summary>Có bao nhiêu ServiceAccount trong namespace default?</summary>

    Chạy lệnh `kubectl get serviceaccounts` (viết tắt `kubectl get sa`) và đếm số lượng.

    </details>

1.  <details>
    <summary>Secret token mà ServiceAccount default đang dùng là gì?</summary>

    Chạy lệnh `kubectl describe serviceaccount default` và xem trường `Tokens`.

    > `none`

    💡 Vì sao: từ Kubernetes v1.24, ServiceAccount không còn được tự động tạo Secret chứa token. Token cho Pod được cấp qua TokenRequest API và mount bằng projected volume, nên trường `Tokens` hiển thị `<none>`.

    </details>

1.  <details>
    <summary>Chúng ta vừa triển khai ứng dụng Dashboard.<br/>Kiểm tra Deployment. Deployment này dùng image nào?</summary>


    Chạy lệnh `kubectl describe deployment` và xem trường `Image`

    > `gcr.io/kodekloud/customimage/my-kubernetes-dashboard`

    </details>

1. Chỉ là thông tin (không cần trả lời).

1.  <details>
    <summary>Trạng thái của dashboard là gì? Thông tin Pod đã được tải thành công chưa?</summary>

    Mở link `web-dashboard` ở phía trên terminal và kiểm tra trạng thái. Ta thấy có thông báo lỗi, do đó trạng thái là...

    > `Failed`

    </details>

1.  <details>
    <summary>Ứng dụng Dashboard dùng loại account nào để truy vấn Kubernetes API?</summary>

    Như thấy rõ từ thông báo lỗi trên giao diện web-dashboard, Pod dùng một ServiceAccount để truy vấn Kubernetes API.

    > Service Account

    </details>

1.  <details>
    <summary>Ứng dụng Dashboard dùng account nào để truy vấn Kubernetes API?</summary>

    Để tìm ra, ta cần kiểm tra YAML của Pod đang chạy. Trường dùng để chỉ định ServiceAccount của Pod là `serviceAccountName`. Để khỏi phải đọc _toàn bộ_ YAML, ta có thể dùng lệnh `grep` để lọc riêng trường đó:

    ```
    kubectl get po -o yaml | grep 'serviceAccountName:'
    ```

    Bạn cũng có thể dùng JSONPath. Trước tiên lấy tên Pod bằng `kubectl get pods` (tên này sẽ khác nhau mỗi lần chạy lab). Sau đó chạy lệnh, ví dụ:

    ```
    kubectl get po web-dashboard-65b9cf6cbb-79vbs -o jsonpath='{.spec.serviceAccountName}'
    ```

    > `default`

    </details>

1.  <details>
    <summary>Kiểm tra Pod của ứng dụng Dashboard và xác định ServiceAccount được mount vào nó.</summary>

    Câu này giống câu trước.

    > `default`

    </details>

1.  <details>
    <summary>Thông tin xác thực của ServiceAccount nằm ở vị trí nào bên trong Pod?</summary>

    Token của ServiceAccount được mount vào Pod dưới dạng volume mount, vì vậy ta cần xem phần `volumeMounts`.

    ```
    kubectl describe pod
    ```

    Tìm phần `Mounts` (danh sách các volume được mount), bạn sẽ thấy đường dẫn tới ServiceAccount được mount (đầy đủ là `/var/run/secrets/kubernetes.io/serviceaccount`, chứa các file `token`, `ca.crt`, `namespace`). Trong các đáp án, chọn đáp án có tiền tố đường dẫn đúng

    > `/var/run/secrets`

    </details>

1.  <details>
    <summary> Tạo một ServiceAccount mới tên là <b>dashboard-sa</b>.</summary>

    Chạy lệnh `kubectl create serviceaccount dashboard-sa`

    </details>

1. Chỉ là thông tin (không cần trả lời). Lab đã cấp thêm quyền cho `dashboard-sa` bằng RBAC (các file cấu hình nằm tại `/var/rbac`) — có thể kiểm tra bằng `kubectl auth can-i list pods --as=system:serviceaccount:default:dashboard-sa`.

1.  Bây giờ chúng ta sẽ kiểm tra quyền truy cập của ServiceAccount thông qua dashboard.

    1. Tạo token

        ```
        kubectl create token dashboard-sa
        ```

        Lệnh này sẽ sinh ra một chuỗi ký tự dài (một JWT, mặc định hết hạn sau 1 giờ; lệnh không tạo Secret nào).

    1. Dùng chuột bôi đen toàn bộ output và copy.
    1. Quay lại giao diện dashboard và dán vào ô `Token`
    1. Nhấn Load Dashboard. Lúc này dashboard sẽ hiển thị Pod

1.  <details>
    <summary>Sửa Deployment để đổi ServiceAccount từ <b>default</b> sang <b>dashboard-sa</b>.</summary>

    1. Dùng lệnh `kubectl edit deployment web-dashboard`, lệnh này mở Deployment đang chạy trong `vi`
    1. Di chuyển xuống phần spec của Pod template và chèn ServiceAccount như dưới đây:

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        annotations:
          deployment.kubernetes.io/revision: "2"
        creationTimestamp: "2023-02-21T19:29:21Z"
        generation: 2
        name: web-dashboard
        namespace: default
        resourceVersion: "1499"
        uid: ac5a26bf-7a88-41cc-8db3-d5a4bd2ad31c
      spec:
        progressDeadlineSeconds: 600
        replicas: 1
        revisionHistoryLimit: 10
        selector:
          matchLabels:
            name: web-dashboard
        strategy:
          rollingUpdate:
            maxSurge: 25%
            maxUnavailable: 25%
          type: RollingUpdate
        template:
          metadata:
            creationTimestamp: null
            labels:
              name: web-dashboard
          spec:
            serviceAccountName: dashboard-sa    # <- Chèn dòng này
            containers:
            - env:
              - name: PYTHONUNBUFFERED
                value: "1"
              image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
              imagePullPolicy: Always
              name: web-dashboard
              ports:
              - containerPort: 8080
                protocol: TCP
      ```

    1. Lưu và thoát `vi`. Deployment sẽ được cập nhật và tạo Pod mới với ServiceAccount mới

    💡 Cách nhanh hơn: `kubectl set serviceaccount deployment web-dashboard dashboard-sa`

    </details>

1. Tải lại dashboard và xác nhận nó hoạt động mà không cần dán token (vì token của `dashboard-sa` giờ đã được tự động mount vào Pod).