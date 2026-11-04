# Tạo tài nguyên tính toán (Compute Resources)

Chúng ta sẽ tạo hạ tầng như dưới đây. Hạ tầng sẽ được tạo bằng Terraform để không tốn quá nhiều thời gian lab chỉ cho việc dựng hạ tầng, giúp bạn tập trung vào việc cài đặt cluster.

![Infra](../../../images/kubeadm-aws-architecture.png)


Như trong sơ đồ, chúng ta sẽ tạo ba EC2 instance để tạo thành cluster và thêm một instance `student-node` để thực hiện việc cấu hình. Chúng ta dựng hạ tầng bằng Terraform từ AWS CloudShell (nên bạn không phải cài Terraform trên máy của mình), sau đó đăng nhập vào `student-node`, máy có thể truy cập các node của cluster. Mối quan hệ giữa `student-node` và các node của cluster tương tự như CKA Ultimate Mocks và cách đề thi thật hoạt động — bạn bắt đầu trên một node riêng (ở đây là `student-node`), rồi dùng SSH để kết nối tới các node của cluster. Lưu ý rằng kết nối SSH chỉ thực hiện được theo chiều mũi tên. Không thể SSH từ, ví dụ, `controlplane` thẳng sang `node01`. Bạn phải `exit` về `student-node` trước. Trong đề thi cũng như vậy. `student-node` đóng vai trò một [bastion host](https://en.wikipedia.org/wiki/Bastion_host).

Chúng ta cũng sẽ thiết lập kết nối trực tiếp từ máy của bạn tới các node port của worker để bạn có thể mở bằng trình duyệt bất kỳ NodePort Service nào bạn tạo (xem phần bảo mật bên dưới).

Một số thiết lập bảo mật cơ bản sẽ được cấu hình:

* Chỉ `student-node` mới truy cập được API Server của cluster, và đây là nơi bạn sẽ chạy các lệnh `kubectl` khi cluster đã hoạt động.
* Chỉ `student-node` mới SSH được tới các node của cluster.
* Các port mà bản thân Kubernetes cần (bao gồm etcd) và Weave CNI sẽ được cấu hình trong security group trên các node của cluster.

> 💡 **Bổ sung:** Security group trong terraform của bài này mở các port của Weave (6783/tcp, 6783-6784/udp), nhưng các bước chung ở [generic](../../generic/05-controlplane.md) hiện cài **Calico**. Calico dùng các port khác (BGP 179/tcp, VXLAN 4789/udp, Typha 5473/tcp — bản [aws-ha](../../aws-ha/) đã mở sẵn các port này). Nếu Pod trên các node khác nhau không nói chuyện được với nhau (vd `curl` tới NodePort chỉ chạy được trên một node), hãy kiểm tra security group. Weave Net cũng đã ngừng được phát triển nên không còn được khuyến nghị.

Các vấn đề bảo mật khiến cách làm này không phù hợp cho một cluster production thật sự:

* Các kube node nên nằm trong private subnet (không truy cập trực tiếp được từ Internet) và đặt sau một NAT gateway để có thể tải package, hoặc với yêu cầu bảo mật cao hơn nữa thì hoàn toàn [airgapped](https://en.wikipedia.org/wiki/Air_gap_(networking)) (cách ly khỏi Internet).
* Truy cập tới API server và etcd cần được kiểm soát chặt chẽ hơn.
* Không khuyến nghị dùng default VPC.
* Các node port sẽ được mở ra toàn thế giới — tức là ai cũng có thể kết nối tới.
* Trong thực tế, ta sẽ tạo một cloud load balancer kết hợp với ingress controller để cung cấp ingress vào cluster. _Tuyệt đối_ không nên expose node port của worker node ra Internet như chúng ta đang làm ở đây!!!

Những thứ khác sẽ được cấu hình bởi code Terraform
* Hostname đặt trên các node: `controlplane`, `node01`, `node02`
* Nội dung `/etc/hosts` được thiết lập trên tất cả các node để dễ dùng lệnh `ssh` từ `student-node`.
* Tạo và phân phối một key pair để đăng nhập vào các instance qua SSH.

Nào, bắt tay vào dựng hạ tầng thôi!

[Bấm vào đây](https://kodekloud.com/topic/playground-aws/) để mở một playground, rồi bấm `START LAB` để yêu cầu một AWS Cloud Playground mới. Sau vài giây, bạn sẽ nhận được URL và thông tin đăng nhập để truy cập AWS Cloud console. Hãy đăng nhập vào console.

Lưu ý rằng bạn phải có gói KodeKloud Pro mới chạy được AWS playground. Nếu bạn có tài khoản AWS riêng, cách này vẫn hoạt động, tuy nhiên bạn sẽ phải trả chi phí cho mọi tài nguyên được tạo cho tới khi xóa chúng.

Chúng ta sẽ chạy toàn bộ bài lab này trong AWS CloudShell — một terminal Linux chạy ngay trong AWS console và đã được cấu hình sẵn phần lớn những gì ta cần, như git và AWS credentials mà Terraform cần. [Bấm vào đây](https://us-east-1.console.aws.amazon.com/cloudshell/home?region=us-east-1) để mở CloudShell — lưu ý link này sẽ không hoạt động cho tới khi bạn đã đăng nhập vào AWS console.


## Cài đặt Terraform

Từ dấu nhắc lệnh của CloudShell...

```bash
curl -O https://releases.hashicorp.com/terraform/1.6.2/terraform_1.6.2_linux_amd64.zip
unzip terraform_1.6.2_linux_amd64.zip
mkdir -p ~/bin
mv terraform ~/bin/
terraform version
```

## Clone repo này

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
```

Giờ chuyển vào thư mục `aws/terraform`

```bash
cd certified-kubernetes-administrator-course/kubeadm-clusters/aws/terraform
```

## Tạo hạ tầng

1. Chạy terraform

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

    Quá trình này mất khoảng nửa phút. Nếu mọi thứ chạy đúng, bạn sẽ thấy nội dung tương tự như sau ở cuối output. Địa chỉ IP của bạn _sẽ khác_

    ```
    Apply complete! Resources: 22 added, 0 changed, 0 destroyed.

    Outputs:

    address_node01 = "44.220.138.27"
    address_node02 = "54.167.161.210"
    address_student_node = "34.205.252.168"
    connect_student_node = <<EOT
    Use the following command to log into student-node

      ssh ubuntu@34.205.252.168

    You should wait till all instances are fully ready in the EC2 console.
    The Status Check colunm should contain "2/2 checks passed"


    EOT
    ```

    Copy toàn bộ output này vào notepad để dùng sau.

1. Đợi tất cả các instance sẵn sàng (Instance state - `running`, Status check - `2/2 checks passed`). Việc này mất 2-3 phút. Xem tại [EC2 console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:instanceState=running).

1. Đăng nhập vào `student-node`

    Copy lệnh `ssh` từ output `connect_student_node` của terraform, ví dụ

    ```
    ssh ubuntu@100.26.200.3
    ```

    Lưu ý địa chỉ IP của bạn _sẽ khác_.

    Bạn sẽ thấy dấu nhắc lệnh trông như thế này

    ```
    ubuntu in 🌐 student-node in ~
    ❯
    ```

## Chuẩn bị student node

Chúng ta sẽ cài kubectl ở đây để có thể chạy lệnh tới cluster khi nó được dựng xong

1. Cài phiên bản kubectl mới nhất và đặt vào thư mục chương trình của user
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin
    ```

1. Kiểm tra

    ```bash
    kubectl version
    ```

    Trong output, ngoài các thông tin khác, nó sẽ báo

    > The connection to the server localhost:8080 was refused - did you specify the right host or port?

    điều này là bình thường, vì chúng ta chưa cài kubernetes.

    > 💡 **Bổ sung:** Khi không tìm thấy file kubeconfig (`~/.kube/config` hoặc biến `KUBECONFIG`), `kubectl` mặc định thử kết nối tới `localhost:8080`, nên mới có lỗi này. Trên cluster đã chạy, nếu gặp lỗi tương tự thì thường là do thiếu hoặc sai kubeconfig.

## Xóa cluster

Nếu dùng KodeKloud playground, bước này không thực sự cần thiết vì tài nguyên sẽ bị xóa khi playground kết thúc.

Nếu bạn dùng tài khoản riêng, bước này là *cực kỳ quan trọng* vì bạn sẽ bị tính tiền cho các tài nguyên đã tạo cho tới khi xóa chúng — trừ khi bạn muốn giữ lại và chịu trả phí. Hãy nhớ rằng đây *không* phải là một bản cài đặt đã được gia cố cho production và có thể gây rủi ro bảo mật cho tài khoản của bạn nếu cứ để đó.

Để xóa

1. Quay lại terminal CloudShell
1. Trong chính thư mục bạn đã chạy `terraform apply`, chạy

    ```
    terraform destroy -auto-approve
    ```

## Ghi chú về code terraform

Những ai đang học các khóa Terraform của chúng tôi nên xem các file terraform và thử hiểu những gì đang diễn ra ở đây.

Một điểm cần chú ý là với các instance `node`, chúng ta tạo network interface (ENI) cho chúng như những resource riêng biệt, rồi gắn các ENI này vào instance khi instance được tạo. Lý do là để biết trước địa chỉ IP của các instance, nhờ đó trong quá trình tạo instance, script user_data có thể tạo sẵn `/etc/hosts`.


Tiếp theo: [Kết nối](./03-connectivity.md)<br/>
Trước đó: [Điều kiện tiên quyết](./01-prerequisites.md)
