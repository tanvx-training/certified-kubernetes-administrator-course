# Thiết lập biến môi trường cho Terraform

Bây giờ chúng ta cần thiết lập một số biến môi trường mà terraform yêu cầu để xác thực với Azure. Đây là các giá trị đã thu thập ở phần trước của hướng dẫn này.

---

## Bước 1: Di chuyển tới thư mục project

Mở một terminal (bash với Linux hoặc macOS, hoặc PowerShell/GitBash với Windows) tại thư mục mà repository này đã được tải về hoặc clone về.

---

## Bước 2: Thiết lập biến môi trường

Các biến này là bắt buộc để xác thực với Azure.

### **Dùng Linux, MacOS hoặc Windows Git Bash**

Chạy các lệnh sau (thay các giá trị bằng thông tin xác thực thực tế của bạn):

```bash
export TF_VAR_ARM_CLIENT_ID=<Paste the value>
export TF_VAR_ARM_CLIENT_SECRET=<Paste the value>
export TF_VAR_ARM_TENANT_ID=<Paste the value>
export TF_VAR_ARM_SUBSCRIPTION_ID=<Paste the value>
```

###  Dùng **Powershell (Windows)**

Chạy các lệnh sau (bắt buộc có dấu nháy kép):
```pwsh
$env:TF_VAR_ARM_CLIENT_ID = "<Paste the value>"
$env:TF_VAR_ARM_CLIENT_SECRET = "<Paste the value>"
$env:TF_VAR_ARM_TENANT_ID = "<Paste the value>"
$env:TF_VAR_ARM_SUBSCRIPTION_ID = "<Paste the value>"
```

> 💡 **Bổ sung:** Terraform tự động đọc mọi biến môi trường có tiền tố `TF_VAR_` và gán giá trị cho input variable cùng tên (phần sau tiền tố). Ví dụ `TF_VAR_ARM_CLIENT_ID` sẽ trở thành giá trị của `variable "ARM_CLIENT_ID"` trong cấu hình. Thay `<Paste the value>` bằng giá trị thật (với bash thì không cần giữ dấu `<` `>`).

---
## Bước 3: Cấu hình <code>main.tf</code> cho shell của bạn

Terraform dùng external data source để lấy các biến môi trường thông qua một script. Hãy dùng khối tương ứng với shell của bạn.

* Nếu dùng PowerShell (Windows)

    Bỏ comment khối sau trong `main.tf` và comment khối Bash nếu có:
    ```
    data "external" "environment" {
      program = ["powershell", "-ExecutionPolicy", "Bypass", "-File", "${path.module}/environment.ps1"]
    }
    ```

* Với tất cả các môi trường khác

    Bỏ comment khối sau trong `main.tf` và comment khối PowerShell nếu có:
    ```
    data "external" "environment" {
      program = ["bash", "${path.module}/environment.sh"]
    }
    ```
* Chỉ được có một khối external (Git Bash hoặc PowerShell) được kích hoạt tại một thời điểm.
* Các biến môi trường này chỉ là tạm thời và chỉ tồn tại trong phiên terminal hiện tại.

> 💡 **Bổ sung:** Vì biến môi trường chỉ tồn tại trong phiên terminal hiện tại, nếu bạn mở terminal mới (hoặc chạy `terraform destroy` sau này ở cửa sổ khác) thì phải `export` lại các biến trên.

Tiếp theo: [Triển khai Cluster](./04-deploy-cluster.md)
