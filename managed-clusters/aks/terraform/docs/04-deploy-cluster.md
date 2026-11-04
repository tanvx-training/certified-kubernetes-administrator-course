# Triển khai cluster

1. Bây giờ chúng ta có thể triển khai cluster

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

    Quá trình này sẽ mất khoảng 5 phút. Trong console, nếu bạn vào `Kubernetes Service` bạn có thể thấy cluster đang được tạo.

    > 💡 **Bổ sung:** `terraform init` tải các provider (ở đây là `azurerm`) và khởi tạo thư mục làm việc; `terraform plan` hiển thị những tài nguyên sẽ được tạo/thay đổi; `terraform apply` thực sự tạo tài nguyên - lệnh này sẽ hỏi xác nhận, bạn cần gõ `yes` rồi Enter.

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

    > 💡 **Bổ sung:** `az aks get-credentials` tải thông tin kết nối của cluster và ghép vào file kubeconfig (`~/.kube/config`); tùy chọn `--overwrite-existing` ghi đè entry cũ cùng tên nếu có. Sau đó `kubectl` sẽ dùng context này để làm việc với cluster.

1. Kiểm tra các node

    ```bash
    kubectl get nodes
    ```

Khi tất cả các node đều hiển thị trạng thái ready, cluster đã sẵn sàng để sử dụng!

> ⚠️ **Lưu ý:** Khi thực hành xong (đặc biệt trên tài khoản Azure thật), hãy chạy `terraform destroy` trong cùng thư mục để xóa toàn bộ tài nguyên đã tạo và tránh phát sinh chi phí.

Trước: [Cài đặt terraform](./03-install-terraform.md)
