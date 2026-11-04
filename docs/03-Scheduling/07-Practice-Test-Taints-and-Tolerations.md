# Practice Test - Taints và Tolerations (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-taints-and-tolerations/)

Lời giải cho bài Practice Test - Taints và Tolerations

1.  <details>
    <summary>Hệ thống có bao nhiêu node?</summary>

    ```
    $ kubectl get nodes
    ```

    Đếm số node

    </details>

1.  <details>
    <summary>Node node01 có taint nào không?</summary>

    ```
    $ kubectl describe node node01
    ```

    Tìm thuộc tính `Taints` trong output.
    </details>

1.  <details>
    <summary>Tạo taint trên node01 với key là spray, value là mortein và effect là NoSchedule</summary>

    ```
    kubectl taint nodes node01 spray=mortein:NoSchedule
    ```
    </details>

1.  <details>
    <summary>Tạo một Pod mới dùng image nginx, tên Pod là mosquito.</summary>

    ```
    kubectl run mosquito --image nginx
    ```
    </details>

1.  <details>
    <summary>Trạng thái của Pod là gì?</summary>

    ```
    kubectl get pods
    ```

    Xem cột `STATUS`
    </details>

1.  <details>
    <summary>Theo bạn, tại sao Pod ở trạng thái Pending?</summary>

    Muỗi (mosquito) thì sợ thuốc xịt muỗi Mortein!

    Vậy câu trả lời là: Pod không tolerate được taint trên node. (Node controlplane cũng có taint `NoSchedule`, nên không còn node nào nhận Pod. Xem phần `Events` của `kubectl describe pod mosquito` sẽ thấy thông báo dạng `0/2 nodes are available: ... untolerated taint ...`.)

    </details>

1.  <details>
    <summary>Tạo một Pod khác tên bee với image nginx, có toleration cho taint mortein.</summary>

    Nghe nói loài ong (bee) miễn nhiễm với mortein!

    1.  Tạo khung YAML cho Pod bằng cách imperative

        ```
        kubectl run bee --image nginx --dry-run=client -o yaml > bee.yaml
        ```
    1.  Sửa file để thêm toleration
        ```
        vi bee.yaml
        ```
    1. Thêm toleration. Phần này đặt cùng mức thụt lề với `containers` vì đây là thiết lập của Pod (thuộc `spec`).
        ```yaml
          tolerations:
          - key: spray
            value: mortein
            effect: NoSchedule
            operator: Equal
        ```
    1. Lưu và thoát, sau đó tạo Pod
        ```
        kubectl create -f bee.yaml
        ```
    </details>

1. Câu hỏi chỉ mang tính thông tin.

1.  <details>
    <summary>Node controlplane có taint nào không?</summary>

    ```
    kubectl describe node controlplane
    ```

    Xem thuộc tính `Taints`.
    </details>

1.  <details>
    <summary>Xóa taint trên controlplane (taint đang có effect NoSchedule).</summary>

    Dấu `-` ở cuối nghĩa là xóa taint.

    ```
    kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
    ```
    </details>

1.  <details>
    <summary>Bây giờ trạng thái của Pod mosquito là gì?</summary>

    ```
    $ kubectl get pods
    ```
    </details>

1.  <details>
    <summary>Pod mosquito hiện đang chạy trên node nào?</summary>

    ```
    $ kubectl get pods -o wide
    ```

    Điều này cũng giải thích vì sao trước đó Pod `mosquito` không thể được lập lịch ở bất kỳ đâu: nó cũng không tolerate được taint của `controlplane`, taint mà giờ ta đã xóa. Vì vậy Pod được đặt lên `controlplane`.
    </details>
