# Networking

1. Chọn VPC - chỉ nên có một lựa chọn duy nhất, là Default VPC.

1. Chọn giá trị cho các trường Subnets. Cần chọn ít nhất hai subnet, và không cần nhiều hơn 3. Đảm bảo `us-east-1e` *không* nằm trong số các subnet được chọn. Nếu có ít hơn 2 subnet, bạn sẽ phải vào VPC console để thêm subnet (nằm ngoài phạm vi của hướng dẫn này), hoặc khởi động lại playground cho đến khi có từ 2 subnet trở lên.

    ![](../images/04-subnets.png)

    > 💡 **Bổ sung:** EKS yêu cầu các subnet thuộc ít nhất hai Availability Zone (AZ) khác nhau để đảm bảo tính sẵn sàng cao cho control plane. AZ `us-east-1e` cần tránh vì đây là AZ cũ có nhiều hạn chế: EKS không hỗ trợ đặt control plane ở một số AZ nhất định của `us-east-1`, và nhiều loại instance mới cũng không có ở AZ này, nên việc tạo cluster/node có thể bị lỗi.

1. Cuộn xuống cuối và bấm `Next`.
1. Ở trang tiếp theo (Observability), bấm `Next` lần nữa vì không có gì cần làm ở đây.
1. Ở trang tiếp theo (Select add-ons), bấm `Next` lần nữa vì ở đây cũng không có gì cần làm.
1. Ở trang tiếp theo (Configure selected add-ons settings), bấm `Next` lần nữa vì ở đây cũng không có gì cần làm.

Trước: [Cấu hình Cluster](./03-configure-cluster.md)<br/>
Tiếp theo: [Tạo Cluster](./05-create-cluster.md)





