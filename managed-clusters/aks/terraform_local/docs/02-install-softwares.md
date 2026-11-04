# Cài đặt phần mềm

Để có kết quả tốt nhất, hãy cài các phần mềm cần thiết bằng trình quản lý gói (package manager).

* Windows - Dùng [trình quản lý gói chocolatey](https://github.com/kodekloudhub/community-faq/blob/main/docs/how-tos/howto-package-management-on-windows.md)
* macOS - Dùng [HomeBrew](https://brew.sh/)
* Linux - Dùng trình quản lý gói có sẵn, ví dụ `apt`, `yum`, v.v.

Cài đặt những phần mềm sau nếu bạn chưa có.

* [terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#install-terraform)
* `kubectl`
   - [macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macOS/)
   - [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
   - [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
* `azure-cli`
    * [macOS](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-macos?view=azure-cli-latest)
    * [Linux](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?view=azure-cli-latest)
    * Windows
        ```
        choco install -y azure-cli
        ```

> 💡 **Bổ sung:** Ngoài ra bạn cũng cần `git` để clone repository ở bước dưới. Trên macOS có thể cài nhanh mọi thứ bằng `brew install terraform kubectl azure-cli git` (với terraform, HashiCorp khuyến nghị dùng tap `hashicorp/tap/terraform`).


# Kiểm tra cài đặt

Mở một shell mới và kiểm tra xem từng phần mềm ở trên đã được cài đặt đúng chưa:

1. Để kiểm tra Terraform, mở powershell (hoặc terminal) và gõ
    ```
    terraform version
    ```
1. Để kiểm tra phiên bản kubectl, mở powershell (hoặc terminal) và gõ
    ```
    kubectl version --client
    ```
1. Để kiểm tra phiên bản Azure CLI, mở powershell (hoặc terminal) và gõ
    ```
    az version
    ```

    Lưu ý rằng bạn có thể gặp lỗi ở bước này trên Windows. Nếu gặp, hãy sửa bằng cách chạy lệnh sau trong một command prompt được mở bằng quyền `as Administrator`

    ```
    "C:\Program Files\Microsoft SDKs\Azure\CLI2\python.exe" -m pip install pywin32 --force-reinstall
    ```


# Clone repository

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/managed-clusters/aks/terraform_local
```

Khi bạn đã kiểm tra các gói phần mềm và clone repository xong, hãy chuyển sang bước tiếp theo.

Tiếp theo: [Thiết lập biến môi trường](./03-setting-variable.md)
