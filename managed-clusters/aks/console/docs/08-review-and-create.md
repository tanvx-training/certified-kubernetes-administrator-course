# Review and Create (Xem lại và tạo)

## Checklist cuối cùng

Dưới đây là tóm tắt các thiết lập mà bạn cần cấu hình ở từng phần của giao diện. Trường nào trong form không được nhắc đến thì bạn không nên thay đổi, hãy để giá trị mặc định.

1. **Basics**
    * **Resource Group**: Chọn lựa chọn duy nhất có trong danh sách
    * **Cluster Preset Configuration**: `Dev/Test`
    * **Kubernetes Cluster Name**: `kodekloud-demo`
    * **Region**: `(US) East US`
1. **Node Pools**

    Ở đây chỉ nên có `agentpool`. Nếu còn có cả `userpool`, hãy chọn và xóa nó.

    * **Node size**: [Đổi](./05-node-pools.md) thành `Standard D2s_v3`
    * **Scale method**: `manual`
    * **Node count**: `2`
1. **Networking**
    * **DNS name prefix**: Phải là duy nhất trên toàn cầu. Đặt thành `kodekloud-demo-` theo sau là vài chữ số ngẫu nhiên tùy bạn chọn, ví dụ `kodekloud-demo-321435423`
1. **Integrations**
    * Tất cả các tùy chọn ở đây phải được bỏ chọn.
1. **Monitoring**
    * Tất cả các tùy chọn ở đây phải được bỏ chọn.
1. **Security**
    * Tất cả các tùy chọn ở đây phải được bỏ chọn.
1. **Advanced**
    * Không thay đổi gì ở đây.
1. **Tags**
    * Bạn có thể thêm tag nếu muốn, nhưng không bắt buộc.

## Triển khai

Bấm nút `Review and Create` ở cuối cửa sổ trình duyệt. Hệ thống sẽ kiểm tra (validate) trong vài giây rồi hiển thị màn hình tóm tắt với nút `Create` ở cuối.

Bấm nút `Create`. Quá trình khởi tạo sẽ mất vài phút, vậy hãy đi pha một tách trà/cà phê :smile:!

Khi hoàn tất, bạn sẽ thấy màn hình này

![image](../images/08-create-deployment-complete.png)

Bấm vào `Go to resource`

Bạn có thể thấy một lỗi tương tự như dưới đây, nhưng đây là vấn đề đã biết trước trên Azure Playground và không ảnh hưởng tới bất kỳ tính năng nào của AKS, nên cứ bỏ qua nó.

> The client 'odl_user_1278229@cloudlabs4kodeKloud.onmicrosoft.com' with object id 'da188846-792f-40be-8c62-ed65018e3a6e' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourceGroups/read' over scope '/subscriptions/1ca63aff-186a-4e2b-b3bc-f7dddf1d8969/resourceGroups/MC_ODL-azure-1278229_kodekloud-demo_eastus' or the scope is invalid. If access was recently granted, please refresh your credentials.

> 💡 **Bổ sung:** Resource group có tiền tố `MC_` trong thông báo lỗi là "node resource group" do AKS tự động tạo để chứa các tài nguyên hạ tầng của cluster (VMSS, ổ đĩa, load balancer, virtual network...). Tài khoản playground không có quyền đọc resource group này nên mới xuất hiện lỗi trên.

Tiếp theo: [Kết nối tới Cluster](./09-connect.md)

