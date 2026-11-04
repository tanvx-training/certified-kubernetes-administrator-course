# Thêm các node vào Cluster

Bây giờ chúng ta sẽ thêm một số node *tự quản (unmanaged)* vào cluster. AWS Playground không hỗ trợ managed node group.

> 💡 **Bổ sung:** Với **managed node group**, EKS tự tạo và quản lý vòng đời của các EC2 instance (tạo, cập nhật AMI, drain khi upgrade...). Với **self-managed (unmanaged) node**, bạn tự tạo các EC2 instance (ở đây thông qua một CloudFormation template) và tự chịu trách nhiệm cho việc join, cập nhật và bảo trì chúng.

1. Tạo một SSH keypair để các node sử dụng
    1. Vào EC2 console bằng cách gõ `ec2` vào ô tìm kiếm ở phía trên, rồi chọn nó trong danh sách.
    1. Cuộn menu bên trái xuống cho tới khi thấy `Key Pairs`, rồi bấm vào đó để vào console quản lý key pair.
    1. Bấm nút màu cam `Create key pair` ở phía trên bên phải
    1. Điền form với tên là `node-key-pair`, key pair type là `RSA` và private key format là `.pem`

        ![](../images/06-key-pair.png)

    1. Bấm nút `Create key pair`

    > 💡 **Bổ sung:** Khi tạo key pair, trình duyệt sẽ tự động tải file private key `node-key-pair.pem` về máy. AWS không lưu lại file này, nên nếu muốn SSH vào node sau này (`ssh -i node-key-pair.pem ec2-user@<public-ip>`), hãy giữ file đó cẩn thận.

1. Vào CloudFormation console, cũng bằng ô tìm kiếm. CloudFormation là công cụ Infrastructure as Code gốc của AWS, và chúng ta sẽ dùng một template được chuẩn bị sẵn để định nghĩa các worker node.
    1. Bấm nút màu cam `Create stack`
    1. Chọn Choose an existing template, chọn Amazon S3 URL và dùng URL sau: `https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2022-12-23/amazon-eks-nodegroup.yaml`

        ![](../images/06-create-stack.png)

    1. Cuộn xuống cuối trang và bấm `Next`

1. Bây giờ chúng ta điền các tham số cần thiết để triển khai các node
    1. `Stack Name`: `eks-cluster-stack`
    1. `ClusterName`: `demo-eks`
    1. `ClusterControlPlaneSecurityGroup`: Bấm vào ô và chọn security group có tên chứa `eks-cluster-sg`
    1. `NodeGroupName`: `eks-demo-node`
    1. `KeyName`: (có thể bạn sẽ cần cuộn xuống để tìm trường này) - `node-key-pair` như đã tạo ở trên.
    1. `VpcId`: Bấm vào ô và chọn mục duy nhất có ở đó
    1. `Subnets`. Tiếc là danh sách thả xuống không hiển thị AZ của các subnet, nên hơi lắt nhắt khi xác định cần chọn subnet nào
        1. Mở [subnets console](https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#subnets:) trong một tab hoặc cửa sổ trình duyệt khác.
        1. Bấm vào từng subnet để xem thông tin của nó ở khung bên dưới.
        1. Ghi lại IPv4 CIDR của những subnet mà bạn đã chọn khi cấu hình networking cho cluster.

            ![](../images/06-subnets.png)

        1. Chọn các subnet này khi quay lại CloudFormation console

            ![](../images/06-subnets-2.png)

    1. Giữ nguyên mặc định cho các thiết lập khác

    Màn hình lúc này sẽ trông tương tự như sau

    ![](../images/06-params-1.png)

    ![](../images/06-params-2.png)

1. Bấm `Next`
1. Không có gì cần làm ở màn hình tiếp theo. Cuộn xuống cuối và bấm `Next`
1. Bây giờ bạn đang ở màn hình cuối cùng. Cuộn xuống cuối và tích vào ô xác nhận (acknowledge), sau đó bấm `Submit`

    ![](../images/06-iam-ack.png)

    > 💡 **Bổ sung:** Ô xác nhận này là bắt buộc vì template sẽ tạo IAM resource (cụ thể là IAM role cho các node - `NodeInstanceRole`).

1. Chờ quá trình tạo stack hoàn tất, sẽ mất vài phút. Bạn có thể cần bấm nút Refresh vài lần cho tới khi trạng thái chuyển thành `CREATE COMPLETE`

    ![](../images/06-stack-complete.png)

1. Bây giờ chọn tab `Outputs` và ghi lại giá trị của `NodeInstanceRole`. Bạn sẽ cần giá trị này khi join các Amazon EKS node.

    ![](../images/06-node-role.png)

Trước: [Tạo Cluster](./05-create-cluster.md)<br/>
Tiếp theo: [Join các Worker Node](./07-join-nodes.md)

