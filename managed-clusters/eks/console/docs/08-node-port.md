# Truy cập các NodePort service

Để có thể truy cập bất kỳ NodePort service nào bạn tạo, chúng ta phải chỉnh sửa security group được gắn với các node để cho phép laptop của bạn truy cập chúng. Trên AWS, security group hoạt động giống như một firewall. Cấu hình mặc định của security group được tạo ở bước trước chỉ cho phép truy cập giữa control plane và các node. Bây giờ chúng ta sẽ chỉnh sửa nó để cho phép IP public của đường mạng internet nhà bạn cũng truy cập được các node trên dải port dành cho NodePort service.

## Chỉnh sửa security group của node

1. Vào EC2 console bằng ô tìm kiếm.
1. Trong menu bên trái, cuộn xuống cho tới khi thấy `Security groups` và bấm vào đó.
1. Trong danh sách security group, bấm vào `eks-cluster-stack-NodeSecurityGroup`.

    ![](../images/08-sg-list.png)

1. Ở khung bên dưới vừa xuất hiện, bấm vào `Edit inbound rules`.

    ![](../images/08-edit-rule.png)

1. Ở trang chỉnh sửa rule hiện ra, bấm nút `Add Rule` nằm ở cuối danh sách `Inbound Rules`.
1. Điền thông tin cho security group rule mới
    * `Type`: `Custom TCP`
    * `Port range`: `30000 - 32768` là dải port mặc định cho node port như bạn đã biết từ khóa học.
    * `Source`: `My IP` Nó sẽ tự động xác định IP public của đường mạng nhà bạn. Bạn có thể tự kiểm tra IP này bằng cách truy cập http://checkip.amazonaws.com

    ![](../images/08-rules.png)

    > 💡 **Bổ sung:** Chính xác thì dải port NodePort mặc định của Kubernetes là `30000-32767` (cấu hình bằng flag `--service-node-port-range` của kube-apiserver). Mở tới `32768` như trên vẫn bao trùm toàn bộ dải này nên không ảnh hưởng gì. Chỉ nên chọn `My IP` thay vì `Anywhere` (`0.0.0.0/0`) để không công khai các service của bạn ra toàn bộ internet.

1. Bấm nút màu cam `Save rules`.

## Tạo một service để thử nghiệm

1. Trong khi vẫn đang ở EC2 console, tìm mục `Instances` trong menu bên trái và bấm vào đó
1. Ghi lại địa chỉ `Public IP` của các node. Bạn sẽ cần dùng một trong số này để kết nối tới nodeport service.

    ![](../images/08-instances.png)

1. Quay lại cloudshell và tạo một service

    1. Tạo pod và service

        ```
        kubectl run nginx --image nginx --expose --port=80
        ```

        > 💡 **Bổ sung:** Tùy chọn `--expose` cùng với `--port` khiến `kubectl run` tạo đồng thời một Pod `nginx` và một Service kiểu ClusterIP cùng tên, với selector `run: nginx`.

    1. Chỉnh sửa service và đổi nó sang nodeport.

        ```
        kubectl edit service nginx
        ```

        Sửa manifest để trông như sau (ở đây chỉ hiển thị các phần liên quan của service manifest)

        ```yaml
        ports:
        - port: 80
          protocol: TCP
          targetPort: 80
          nodePort: 30080       #<- Thêm dòng này
        selector:
          run: nginx
        sessionAffinity: None
        type: NodePort          #<- Sửa từ ClusterIP thành NodePort
        ```

        > 💡 **Mẹo:** Cách nhanh hơn (và hay dùng khi thi CKA) là tạo trực tiếp NodePort service bằng lệnh imperative, ví dụ `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-np`. Tuy nhiên `kubectl expose` không cho phép chỉ định giá trị `nodePort` cụ thể - nếu đề bài yêu cầu một port cụ thể, bạn vẫn phải sửa bằng `kubectl edit` hoặc dùng file YAML.

1. Bây giờ bạn có thể xem service trên trình duyệt bằng cách tạo URL từ địa chỉ IP public lấy được ở EC2 console và node port `30080`. Trong ví dụ này URL là `http://44.198.158.250:30080`, nhưng với bạn địa chỉ IP sẽ khác. Dán URL bạn vừa tạo vào trình duyệt. Bạn sẽ thấy trang chào mừng của nginx.

> ⚠️ **Lưu ý:** Nếu bạn làm theo hướng dẫn này trên tài khoản AWS thật, hãy xóa tài nguyên khi xong theo thứ tự: xóa CloudFormation stack `eks-cluster-stack` (worker node), sau đó xóa cluster `demo-eks` trong EKS console, rồi xóa key pair và IAM role `eksClusterRole` nếu không dùng nữa. EKS tính phí control plane theo giờ ngay cả khi không có node nào.

Trước: [Join các Node](./07-join-nodes.md)
