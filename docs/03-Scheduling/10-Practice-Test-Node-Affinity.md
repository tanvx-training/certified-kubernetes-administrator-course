# Practice Test - Node Affinity (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-node-affinity-2/)

Lời giải cho bài practice test - Node Affinity

1.  <details>
    <summary>Node node01 có bao nhiêu label?</summary>

    ```
    kubectl describe node node01
    ```

    Xem phần `Labels`

    --- HOẶC ---

    ```
    kubectl get node node01 --show-labels
    ```

    </details>

1.  <details>
    <summary>Giá trị của label có key beta.kubernetes.io/arch trên node01 là gì?</summary>

    Tìm câu trả lời trong output của câu 1. (Label `beta.kubernetes.io/arch` đã deprecated, label hiện hành là `kubernetes.io/arch`, nhưng cả hai vẫn được đặt trên node với cùng giá trị, ví dụ `amd64`.)
    </details>

1.  <details>
    <summary>Gắn label color=blue cho node node01</summary>

    ```
    kubectl label node node01 color=blue
    ```
    </details>

1.  <details>
    <summary>Tạo Deployment mới tên blue với image nginx và 3 replica.</summary>

    ```
    kubectl create deployment blue --image=nginx --replicas=3
    ```
    </details>

1.  <details>
    <summary>Các Pod của Deployment blue có thể được đặt lên những node nào?</summary>


    Kiểm tra xem node controlplane và node01 có taint nào ngăn Pod được lập lịch lên không. Nếu không có taint, Pod có thể được đặt lên bất kỳ node nào.

    ```
    kubectl describe nodes controlplane | grep -i taints
    kubectl describe nodes node01 | grep -i taints
    ```
    </details>

1.  <details>
    <summary>Đặt Node Affinity cho Deployment để Pod chỉ được đặt lên node01.</summary>
    Giờ ta sửa trực tiếp Deployment đã tạo ở trên. Nhớ là ta đã gắn label `color=blue` cho `node01` chứ? Bây giờ ta sẽ tạo affinity tới label đó, nó sẽ "hút" các Pod của Deployment về node này.

    1.
        ```
        $ kubectl edit deployment blue
        ```
    1. Thêm đoạn YAML dưới đây vào phần `template.spec`, tức là cùng mức với `containers` vì đây là thiết lập của Pod. Affinity chỉ được xét ở giai đoạn lập lịch, tuy nhiên thay đổi này làm `template` của Deployment thay đổi nên Deployment sẽ rollout lại (tạo Pod mới), và các Pod mới sẽ được lập lịch theo affinity.

      ```yaml
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                  - blue
      ```
    </details>

1. <details>
    <summary>Bây giờ các Pod được đặt trên những node nào?</summary>

    ```
    $ kubectl get pods -o wide
    ```
    </details>

1.  <details>
    <summary>Tạo Deployment mới tên red với image nginx và 2 replica, đảm bảo Pod chỉ được đặt lên node controlplane.</summary>

    1. Tạo YAML mẫu cho Deployment

        ```
        kubectl create deployment red --image nginx --replicas 2 --dry-run=client -o yaml > red.yaml
        ```
    1. Sửa file
        ```
        vi red.yaml
        ```
    1.  Thêm node affinity (bản gốc ghi nhầm là toleration) sử dụng label mà đề bài nêu, đặt ở cùng vị trí như với Deployment `blue`. Label `node-role.kubernetes.io/control-plane` không có giá trị, nên dùng toán tử `Exists`.
      ```yaml
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: node-role.kubernetes.io/control-plane
                  operator: Exists
      ```
    1. Lưu, thoát và tạo Deployment
      ```
      kubectl create -f red.yaml
      ```
    1. Kiểm tra kết quả
      ```
      $ kubectl get pods -o wide
      ```

    > 💡 **Lưu ý:** Nếu node controlplane có taint `node-role.kubernetes.io/control-plane:NoSchedule`, Pod sẽ `Pending` dù affinity đúng — khi đó cần thêm cả toleration tương ứng. Trong lab này controlplane không có taint (đã kiểm tra ở câu 5), nên chỉ cần affinity.
    </details>



