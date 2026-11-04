# Tạo Cluster

Bây giờ bạn sẽ ở trang Review and Create. Tại đây chúng ta sẽ khởi tạo control plane. Lúc này chưa có node nào sẵn sàng - việc đó sẽ làm ở bước sau.

1. Xem lại thông tin bạn đã nhập hoặc chọn ở các trang trước. Nếu cần thay đổi, hãy bấm nút `Edit` bên cạnh phần muốn sửa. Sau đó cuộn xuống cuối và bấm `Create`. Trường Status sẽ hiển thị "Creating" trong khi cluster đang được khởi tạo. Quá trình này sẽ mất *ít nhất* 10 phút, vậy hãy đi pha một tách trà! Bạn có thể cần bấm nút refresh (được chỉ bằng mũi tên) định kỳ để đảm bảo thông tin được cập nhật.

1. Chờ control plane được triển khai. Khi control plane triển khai xong, trạng thái sẽ đổi từ "Creating" sang "Active"

    ![](../images/05-creating.png)

1. **QUAN TRỌNG**: nếu bạn dựng cluster như một phần của bài lab hoặc task KKE (KodeKloud Engineer), bạn phải nhập tất cả các lệnh dưới đây ở terminal của *lab*, không phải CloudShell, nếu không hệ thống chấm điểm sẽ không thể kết nối tới cluster của bạn.<br/><br/>Khi cluster đã được tạo, hãy cho phép kubectl giao tiếp với cluster bằng cách thêm một context mới vào file cấu hình kubectl, thông qua việc chạy lệnh sau trong CloudShell. Bạn có thể mở CloudShell bằng cách bấm nút cloudshell được đánh số `3` trong hình ở trên. Thao tác này sẽ mở một dấu nhắc lệnh cloudshell bên dưới EKS console, nơi bạn có thể dán và chạy lệnh sau:

    ```
    aws eks update-kubeconfig --region us-east-1 --name demo-eks
    ```

    Kiểm tra xem đã kết nối được chưa. Bạn sẽ thấy một số tài nguyên. Tất cả các Pod đều ở trạng thái Pending vì chúng ta chưa tạo node nào để chạy chúng.

    ```
    kubectl get all -A
    ```

    > 💡 **Bổ sung:** `aws eks update-kubeconfig` ghi thông tin cluster vào `~/.kube/config` và đặt context mới làm context hiện tại. Trong kubeconfig này, kubectl dùng lệnh `aws eks get-token` để lấy token xác thực bằng chính danh tính IAM của bạn. Các Pod đang Pending (ví dụ `coredns`) là do scheduler không tìm được node nào để gán chúng - đúng như dự kiến ở giai đoạn này.

Trước: [Networking](./04-networking.md)<br/>
Tiếp theo: [Thêm node](./06-nodes.md)
