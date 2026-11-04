# Kubeadm HA trên AWS EC2

Cập nhật tháng 3/2024

Hướng dẫn này trình bày cách cài đặt một cluster kubeadm 5 node có tính sẵn sàng cao (highly-available) trên các EC2 instance của AWS. Nếu dùng môi trường KodeKloud AWS Playground, hãy đảm bảo bạn đã chọn region `us-east-1` (N. Virginia) ở mục chọn region góc trên bên phải AWS console. Để tương thích với quyền hạn của playground, chúng ta sẽ dùng cấu hình EC2 instance như sau.

* Instance type: `t3.medium`
* Hệ điều hành: Ubuntu 22.04 (tại thời điểm viết)
* Storage: `gp2`, 8GB

Lưu ý rằng đây chỉ là bài tập để dựng được một cluster chạy được, mang tính học tập! Cluster này không phù hợp để phục vụ workload ra Internet, cũng không được bảo mật đúng mức — nếu không thì hướng dẫn này sẽ dài gấp ba! Không nên dùng nó làm nền tảng để xây dựng cluster production.

> 💡 **Bổ sung:** "Highly available" ở đây nghĩa là control plane có 3 node (mỗi node chạy kube-apiserver, kube-controller-manager, kube-scheduler và một thành viên etcd — mô hình *stacked etcd*), đứng sau một load balancer. Với 3 thành viên, etcd vẫn giữ được quorum (đa số 2/3) khi mất 1 node, nên cluster vẫn hoạt động. Đây là lý do số node control plane luôn là số lẻ (3, 5...).

[Bắt đầu](./docs/01-prerequisites.md)
