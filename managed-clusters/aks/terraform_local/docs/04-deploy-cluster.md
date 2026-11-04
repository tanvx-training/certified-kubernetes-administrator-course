# Triển khai cluster

1. Bây giờ chúng ta có thể triển khai cluster

    ```powershell or Git bash
    terraform init
    terraform plan
    terraform apply
    ```

    Quá trình này sẽ mất khoảng 5 phút. Trong console, nếu bạn vào `Kubernetes Service` bạn có thể thấy cluster đang được tạo.

    > 💡 **Bổ sung:** `terraform init` tải các provider và khởi tạo thư mục làm việc; `terraform plan` hiển thị những tài nguyên sẽ được tạo/thay đổi; `terraform apply` thực sự tạo tài nguyên - lệnh này sẽ hỏi xác nhận, bạn cần gõ `yes` rồi Enter.

1. Kết nối tới cluster.

    Khi terraform chạy xong, bạn sẽ thấy kết quả tương tự như dưới đây. Sao chép hai lệnh `az` được hiển thị và dán vào terminal. Lưu ý rằng subscription ID và tên resource group sẽ khác với những gì hiển thị trên trang này.

    ```
    Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

    Outputs:

    commands = <<EOT

    Now run the following commands to gain kubectl access to the cluster

    az account set --subscription a2b28c85-1948-4263-90ca-bade2bac4df4
    az aks get-credentials --resource-group kml_rg_main-4d5e3c08a48840da --name kodekloud-demo --overwrite-existing
    EOT
    ```

    > 💡 **Bổ sung:** Trên máy local, trước khi chạy các lệnh `az` này bạn có thể cần đăng nhập Azure CLI bằng `az login` (dùng tài khoản playground). `az aks get-credentials` sẽ ghép thông tin cluster vào file `~/.kube/config` trên máy bạn.

1. Kiểm tra các node

    ```bash
    kubectl get nodes
    ```

Khi tất cả các node đều hiển thị trạng thái ready, cluster đã sẵn sàng để sử dụng!

> ⚠️ **Lưu ý:** Khi thực hành xong, hãy chạy `terraform destroy` trong cùng thư mục (với các biến môi trường vẫn được thiết lập) để xóa toàn bộ tài nguyên đã tạo. Bạn cũng có thể xóa context của cluster khỏi kubeconfig local bằng `kubectl config delete-context kodekloud-demo`.

Trước: [Thiết lập biến môi trường](./03-setting-variable.md)
