# Tạo Service Role

Lưu ý rằng nhiều popup thông tin có thể xuất hiện ở phía trên màn hình console trong lúc bạn thao tác và có thể che khuất các mục khác. Hãy tắt chúng khi chúng xuất hiện.

Trước khi tạo EKS cluster, bạn cần tạo một IAM (Identity and Access Management) role dành cho cluster service. Role này cấp quyền cho dịch vụ EKS truy cập các API của AWS thay mặt bạn.

1. Vào IAM console
    1. Bấm vào ô tìm kiếm và gõ `iam`
    1. Bấm vào `IAM` trong danh sách kết quả

    ![](../images/02-iam.png)

1. Bấm vào `Roles` ở menu bên trái
1. Bấm nút màu cam `Create Role` ở phía trên bên phải
1. Ở màn hình tiếp theo, `AWS Service` phải đang được chọn. Nếu chưa, hãy bấm vào để chọn.
1. Trong ô `Use case`, bấm vào `Choose a service or use case` và gõ `eks`. EKS sẽ xuất hiện trong danh sách. Bấm vào nó.
1. Trong danh sách radio button vừa xuất hiện, đảm bảo `EKS - Cluster` đang được chọn. Màn hình lúc này sẽ trông như sau:

    ![](../images/02-trusted-entity.png)

1. Bấm nút màu cam `Next` ở phía dưới bên phải. Bạn có thể cần cuộn xuống để thấy nó.
1. Bấm `Next` ở màn hình Add Permission. Không cần thêm quyền nào khác.
1. Cuối cùng đặt tên role là `eksClusterRole`, cuộn xuống cuối và bấm nút màu cam `Create role`

    ![](../images/02-name-role.png)

> 💡 **Bổ sung:** Khi chọn use case `EKS - Cluster`, AWS tự động gắn managed policy `AmazonEKSClusterPolicy` và trust policy cho phép dịch vụ `eks.amazonaws.com` "assume" (đảm nhận) role này. Nhờ đó control plane của EKS có thể tạo các tài nguyên như network interface, load balancer... trong tài khoản của bạn.

Trước: [Đăng nhập](./01-sign-in.md)<br/>
Tiếp theo: [Cấu hình Cluster](./03-configure-cluster.md)
