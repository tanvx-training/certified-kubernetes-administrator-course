# Cấu hình Cluster

Bây giờ chúng ta sẽ cấu hình các tùy chọn của cluster

1. Vào EKS console
    1. Bấm vào ô tìm kiếm và gõ `eks`
    1. Bấm vào `Elastic Kubernetes Service` trong danh sách kết quả

    ![](../images/03-eks.png)

1. Bấm nút màu cam `Create cluster`. Một danh sách sẽ mở ra, và từ đó chọn `Create`

    ![](../images/03-add-cluster.png)

1. Tại trang Configure Cluster:
    1. Chọn `Custom Configuration`
    1. Đảm bảo `EKS Auto Mode` đang được TẮT (OFF)
      ![](../images/03-configure.png)
    1. Nhập tên là `demo-eks` và chọn cluster service role đã tạo ở trên.
      ![](../images/03-configure-2.png)
    1. Bật ConfigMap cho authentication (cần thiết để join các node ở bước sau)
      ![](../images/03-configure-3.png)
    1. Sau đó cuộn xuống cuối và bấm nút `Next`.

> 💡 **Bổ sung:** EKS có hai cơ chế ánh xạ danh tính IAM sang người dùng/nhóm Kubernetes: **ConfigMap `aws-auth`** (cách cũ) và **EKS access entries** (cách mới, quản lý qua API của EKS). Chế độ authentication `EKS API and ConfigMap` cho phép dùng cả hai. Ở hướng dẫn này chúng ta cần ConfigMap vì sẽ dùng `aws-auth` để cho phép các worker node tự quản (unmanaged) join vào cluster.

Trước: [Tạo Service Role](./02-create-service-role.md)<br/>
Tiếp theo: [Networking](./04-networking.md)

