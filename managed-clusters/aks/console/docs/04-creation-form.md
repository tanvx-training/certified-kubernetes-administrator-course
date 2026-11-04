# Cấu hình Cluster

Bây giờ hãy điền vào form cấu hình

### Project Details

| Trường | Giá trị |
|-------|-------|
| Subscription | *Giữ nguyên* |
| Resource Group | Từ danh sách thả xuống, chọn lựa chọn duy nhất có sẵn |

### Cluster Details

Đặt các trường sau. Giữ nguyên tất cả các trường khác

| Trường | Giá trị |
|-------|-------|
| Cluster preset configuration | `Dev/Test` |
| Kubernetes Cluster Name | `kodekloud-demo` |
| Region | `(US) East US` |

![image](../images/04-creation-form-basics.png)

> 💡 **Bổ sung:** Preset `Dev/Test` chọn sẵn cấu hình tối thiểu, chi phí thấp, phù hợp cho mục đích học tập/thử nghiệm. Với môi trường production, bạn nên chọn preset có nhiều node hơn, bật availability zone và monitoring.

Tiếp theo: [Node Pools](./05-node-pools.md)
