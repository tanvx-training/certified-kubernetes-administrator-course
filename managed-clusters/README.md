# Managed Clusters

Kiến thức về managed cluster và cách xây dựng chúng *không* nằm trong chương trình (curriculum) của khóa CKA, vì vậy bạn sẽ không bị kiểm tra phần này trong đề thi. Các bài lab này chỉ mang tính tham khảo cho vui, và vì nhiều người hỏi cách tạo managed cluster trong KodeKloud playground.

Managed cluster là loại cluster mà một phần của nó được quản lý bởi một nhà cung cấp cloud lớn. Với loại cluster này, control plane *luôn luôn* được quản lý thay bạn, nghĩa là bạn không phải cấu hình các control plane node, thiết lập high availability (tính sẵn sàng cao) hay quản lý etcd - tất cả đều được "quản lý" hộ bạn với một mức phí cố định tính theo mỗi giờ cluster tồn tại.

Tùy theo các tùy chọn khi tạo cluster, worker node có thể là không được quản lý (unmanaged), được quản lý một phần hoặc được quản lý hoàn toàn. Tuy nhiên, trong KodeKloud playground chỉ có unmanaged node, nên đó là loại mà chúng ta sẽ tạo.

Bên dưới là hướng dẫn xây dựng managed cluster trong KodeKloud playground. Các cách dựng này đều nằm trong giới hạn IAM (quyền truy cập) của playground.

> 💡 **Bổ sung:** Ba dịch vụ managed Kubernetes phổ biến là **EKS** (Amazon Elastic Kubernetes Service - AWS), **AKS** (Azure Kubernetes Service - Microsoft Azure) và **GKE** (Google Kubernetes Engine - Google Cloud). Điểm khác biệt chính so với cluster tự dựng bằng `kubeadm` (nội dung được thi trong CKA): bạn **không** truy cập được vào control plane node, nên không thể xem các file static Pod manifest trong `/etc/kubernetes/manifests`, không tự backup/restore etcd hay tự `kubeadm upgrade` được - những việc này do nhà cung cấp cloud đảm nhiệm. Tuy vậy, mọi thao tác với `kubectl` (Pod, Deployment, Service, RBAC...) vẫn hoạt động y hệt.

> ⚠️ **Lưu ý chi phí:** Trên tài khoản cloud thật (không phải playground), managed cluster tính phí theo giờ cho control plane (với EKS, AKS gói Standard, GKE...) cộng thêm chi phí VM của worker node, load balancer, ổ đĩa... Hãy luôn **xóa cluster và các tài nguyên liên quan** sau khi thực hành xong để tránh phát sinh hóa đơn ngoài ý muốn. Với KodeKloud playground, các tài nguyên sẽ bị xóa khi phiên lab hết hạn.

**LƯU Ý:** Nếu bạn triển khai cluster như một phần của bài lab trong một khóa học khác, thì ở bất cứ chỗ nào hướng dẫn này yêu cầu chạy lệnh trong terminal CloudShell, bạn phải chạy các lệnh đó trong terminal của bài lab thay thế.

* AWS EKS

    Các cách dựng này tránh dùng `eksctl`, managed node group và EKS Auto Mode, vì playground không hỗ trợ những thứ này.
    * [Dựng thủ công bằng console](./eks/console/README.md)
    * [Dựng tự động bằng Terraform](https://github.com/kodekloudhub/amazon-elastic-kubernetes-service-course/blob/main/docs/playground.md)
* Azure AKS
    * [Dựng thủ công bằng console](./aks/console/README.md)
    * [Dựng tự động bằng Terraform](./aks/terraform/README.md) - Chạy từ Azure CloudShell. Không cần cài gì trên laptop của bạn
    * [Dựng tự động bằng Terraform](./aks/terraform_local/README.md) - Chạy từ laptop của bạn. Yêu cầu cài đặt terraform, git, kubectl, azure client (Azure CLI).
* Google GKE
    * [Dựng thủ công bằng console](./gke/console/README.md)
    * [Dựng tự động bằng Terraform](./gke/terraform/README.md)

