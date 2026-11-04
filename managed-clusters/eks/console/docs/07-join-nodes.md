# Join các Worker Node

Bây giờ CloudFormation đã tạo xong các EC2 instance cho worker node, chúng ta cần join chúng vào cluster. Một lần nữa, nếu đây là bài lab hoặc task KKE, các bước sau phải được thực hiện ở terminal của *lab*.

1. Quay lại terminal CloudShell. Nếu bạn đã đóng nó, hãy bấm nút ở phía trên để mở lại. Nếu nó báo phiên làm việc đã bị đóng do không hoạt động, hãy bấm Enter để khởi động lại.
1. Tải file configmap cho node về

    ```
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml
    ```
1. Bây giờ chỉnh sửa nó bằng `vi`

    ```
    vi aws-auth-cm.yaml
    ```

    Ở chỗ ghi `rolearn: <ARN of instance role (not instance profile)>`, bạn cần xóa `<ARN of instance role (not instance profile)>` và dán vào giá trị `NodeInstanceRole` đã lấy ở phần trước. Khi xong, file sẽ trông tương tự như sau (tuy nhiên giá trị thực tế của bạn sẽ khác).

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: aws-auth
      namespace: kube-system
    data:
      mapRoles: |
        - rolearn: arn:aws:iam::851725221429:role/eks-cluster-stack-NodeInstanceRole-8OYkncRSa4gA
          username: system:node:{{EC2PrivateDNSName}}
          groups:
            - system:bootstrappers
            - system:nodes
    ```

    > 💡 **Bổ sung:** ConfigMap `aws-auth` ánh xạ IAM role của các EC2 instance sang người dùng Kubernetes `system:node:<tên-DNS-private>` thuộc các group `system:bootstrappers` và `system:nodes`. Đây chính là các group mà Node authorizer và các ClusterRoleBinding mặc định dùng để cấp quyền cho kubelet đăng ký node và cập nhật trạng thái - tương tự cơ chế TLS bootstrapping mà bạn học trong phần bảo mật của CKA. Nếu thiếu bước này, kubelet trên các instance sẽ bị từ chối khi gọi tới API server và node sẽ không bao giờ xuất hiện trong `kubectl get nodes`.

1. Lưu và thoát khỏi vi (`:wq`), sau đó apply configmap

    ```
    kubectl apply -f aws-auth-cm.yaml
    ```

    Sẽ mất khoảng một phút để tất cả các node join vào cluster

    ```
    kubectl get nodes
    ```

    > 💡 **Mẹo:** Dùng `kubectl get nodes --watch` (hoặc `-w`) để theo dõi các node chuyển sang trạng thái `Ready` mà không cần chạy lại lệnh nhiều lần. Bấm `Ctrl+C` để thoát.

Chúc mừng! Cluster của bạn đã hoạt động và bạn có thể tạo tài nguyên.

Trước: [Thêm các node vào Cluster](./06-nodes.md)<br/>
Tiếp theo: [Truy cập các NodePort service](./08-node-port.md)

