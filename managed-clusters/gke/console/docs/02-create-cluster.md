# Khởi tạo Cluster

Bây giờ không còn cần chọn project, hay thiết lập route hoặc firewall rule như được hướng dẫn trong [bản hướng dẫn gốc đăng trên Community Forum](https://kodekloud.com/community/t/playground-series-how-to-create-a-managed-kubernetes-cluster-with-google-kubernetes-engine/230314) nữa. Tất cả những thứ này đã được thiết lập sẵn khi bạn bắt đầu, vì vậy bạn có thể bắt tay ngay vào việc cấu hình cluster.

Lưu ý rằng khi bạn di chuyển qua lại trên trang GCP, nhiều popup thông tin màu xanh có thể xuất hiện. Hãy tắt chúng đi nếu có.

Nếu dùng màn hình nhỏ, một số mục trên menu phía trên có thể bị ẩn sau dấu ba chấm dọc - `⋮`. Hãy kiểm tra chỗ này nếu không tìm thấy một số tùy chọn được hiển thị trong ảnh chụp màn hình.<br/><br/>![vert-ellipsis](../images/02-vert-ellipsis.png)

1. Trong menu bên trái, chọn `Kubernetes Engine`. Bạn không cần chọn mục nào trong submenu bật ra.<br/><br/>![engine](../images/02-engine.png)
1. Ở màn hình tiếp theo, bấm nút `CREATE CLUSTER`. Bạn sẽ tới màn hình Cluster Basics.<br/><br/>![create](../images/02-create.png)
1. Gần góc trên bên phải của màn hình này, bấm vào `SWITCH TO STANDARD CLUSTER`, và xác nhận đây là điều bạn muốn làm trong hộp thoại bật ra<br/><br/>![standard](../images/02-standard.png)
1. Sửa tên cluster như hình dưới. Các tên khác *hẳn là* vẫn được, nhưng nếu gặp vấn đề, hãy dùng tên này. Đảm bảo bạn chọn `Regional`, và đặt region là `us-west1`. Giữ nguyên mặc định cho tất cả các trường khác<br/><br/> ![basics](../images/02-cluster-basics.png)

> 💡 **Bổ sung:** GKE có hai chế độ: **Autopilot** (Google quản lý cả node, bạn chỉ trả tiền theo tài nguyên Pod yêu cầu) và **Standard** (bạn tự cấu hình node pool). Playground chỉ cho phép chế độ Standard với cấu hình giới hạn. Cluster **Regional** có control plane và node được nhân bản trên nhiều zone trong region, giúp tăng tính sẵn sàng so với cluster **Zonal** (chỉ một zone).

Chúng ta đã bắt đầu cấu hình cluster, nhưng vẫn còn vài thứ cần thiết lập, vì vậy đừng bấm nút CREATE ở cuối màn hình này vội. Trước đó, chúng ta sẽ cấu hình node pool ở trang tiếp theo...

Trước: [Đăng nhập](./01-sign-in.md)<br/>
Tiếp theo: [Thiết lập Node](./03-node-setup.md)
