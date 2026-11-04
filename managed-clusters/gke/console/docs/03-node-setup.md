# Thiết lập Node

1. Bấm vào `NODE POOLS/Default Pool` ở bên trái. Đặt `Number of nodes (per zone)` thành `1`. Giữ nguyên mặc định cho tất cả các trường khác.<br/><br/> ![default-pool](../images/03-default-pool.png)
1. Bấm vào Nodes. Đảm bảo `Machine Configuration` được đặt là `E2`. Đặt `Boot disk type` thành `Standard persistent disk` và `Boot disk size` thành `20`<br/><br/>![node](../images/03-node.png)

> 💡 **Bổ sung:** Vì đây là cluster Regional, số node được tính *trên mỗi zone*. Region `us-west1` mặc định có 3 zone, nên với `1` node mỗi zone bạn sẽ có tổng cộng 3 worker node. Dòng máy `E2` là loại VM đa dụng giá rẻ; `Standard persistent disk` (ổ HDD) và dung lượng 20 GB giúp giữ trong giới hạn quota của playground.

Các tính năng nâng cao hơn của GKE không được bật trong KodeKloud playground, nên không cần cấu hình gì thêm. Như vậy là bạn đã sẵn sàng! Bấm nút `CREATE` ở cuối màn hình này. Sẽ mất ít nhất 5 phút để triển khai cluster mới. Bạn sẽ được đưa tới màn hình này. Có thể bạn phải cuộn xuống để thấy cluster ở phía dưới. Bạn có thể bấm vào tên cluster để xem màn hình tiến trình.

![creating](../images/03-creating.png)

Trước: [Khởi tạo Cluster](./02-create-cluster.md)<br/>
Tiếp theo: [Kết nối tới Cluster](./04-connect.md)

