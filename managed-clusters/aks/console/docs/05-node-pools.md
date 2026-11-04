# Node Pools

1. Chọn **Node Pools** từ thanh tab ở phía trên.

    ![image](../images/05-node-pools.png)

    Lưu ý rằng menu này cũng có thể xuất hiện ở bên trái:

    ![image](../images/05-node-pools-left.png)

1. Để tuân thủ giới hạn của playground, chúng ta cần thay đổi một số cấu hình. Bấm vào giá trị trong cột **Node size**...

    ![image](../images/05a-node-pools.png)


1.  Bây giờ bạn đang ở màn hình **Update Node Pool**

    1. Bấm vào **Choose a size**

        ![image](../images/05b-node-pools.png)

        Và từ màn hình lựa chọn, chọn `D2s_v3`

        ![image](../images/05c-node-pools.png)

        Bấm nút `Select` xuất hiện ở cuối màn hình để quay lại **Update Node Pool**.

    1. Tại **Scale Method**, bấm chọn radio button `Manual`, sau đó đổi **Node count** thành `2`


    1. Bấm nút `Update` ở cuối màn hình để hoàn tất cấu hình pool.

> 💡 **Bổ sung:** Node pool là một nhóm các worker node có cùng cấu hình (kích thước VM, OS...). Trong AKS, mỗi node pool được triển khai dưới dạng một Virtual Machine Scale Set (VMSS) - đó là lý do tên node có dạng `aks-agentpool-xxxx-vmss000000`. `D2s_v3` là loại VM có 2 vCPU và 8 GiB RAM. Chọn `Manual` scale nghĩa là số node cố định, không bật cluster autoscaler.

Tiếp theo: [Networking](./06-networking.md)


