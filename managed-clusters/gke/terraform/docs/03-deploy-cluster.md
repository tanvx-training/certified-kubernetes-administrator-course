# Triển khai cluster

1. Export project ID cho terraform. Project ID được Cloudshell cung cấp dưới dạng một biến môi trường. Chúng ta cần thiết lập biến môi trường tương ứng cho terraform từ giá trị đó:

    ```bash
     export TF_VAR_project_id=$GOOGLE_CLOUD_PROJECT
     ```

    > 💡 **Bổ sung:** Terraform tự động đọc các biến môi trường có tiền tố `TF_VAR_`, nên `TF_VAR_project_id` sẽ trở thành giá trị của input variable `project_id` trong cấu hình. Bạn có thể kiểm tra giá trị bằng `echo $GOOGLE_CLOUD_PROJECT` - nếu trống, hãy chọn project bằng `gcloud config set project <project-id>`.

1. Bây giờ chúng ta có thể triển khai cluster

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

    Quá trình này sẽ mất ít nhất 5 phút. Trong console, nếu bạn vào `Kubernetes Engine` bạn có thể thấy cluster đang được tạo.

    > 💡 **Bổ sung:** `terraform apply` sẽ hiển thị kế hoạch và hỏi xác nhận - gõ `yes` rồi Enter để tiếp tục.

1. Kết nối tới cluster.

    Chạy lệnh sau trong Cloudshell để xác thực cho kubectl sử dụng cluster

    ```bash
    gcloud container clusters get-credentials kodekloud-demo-cluster --region us-west1 --project $GOOGLE_CLOUD_PROJECT
    ```

1. Kiểm tra các node

    ```bash
    kubectl get nodes
    ```

Khi tất cả các node đều hiển thị trạng thái ready, cluster đã sẵn sàng để sử dụng!

> ⚠️ **Lưu ý:** Khi thực hành xong (đặc biệt trên tài khoản GCP thật), hãy chạy `terraform destroy` trong cùng thư mục (với biến `TF_VAR_project_id` vẫn được thiết lập) để xóa toàn bộ tài nguyên đã tạo và tránh phát sinh chi phí.

Trước: [Cài đặt terraform](./02-install-terraform.md)
