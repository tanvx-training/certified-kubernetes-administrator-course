# Mở Cloudshell

1. Bấm nút CloudShell ở góc trên bên phải

    ![open](../../images/02-open-cloudshell.png)
1. Tại popup mở ra, chọn `Bash`
1. Tại popup tiếp theo, chọn như sau
    * Mount storage account
    * Subscription - chọn từ danh sách thả xuống. Chỉ nên có một lựa chọn, đại loại như `azurekmlprod`
    * Bấm `Apply`
1. Tại popup `Mount Storage Account`, chọn `I want to create a storage account` và bấm `Next`
1. Bây giờ chọn như sau trong popup `Create storage account`
    * `Subscription` - Không thay đổi trường này.
    * `Resource gorup` - chọn từ danh sách thả xuống. Chỉ nên có một lựa chọn.
    * `Storage account name` - Tên bất kỳ bạn thích hoặc các ký tự ngẫu nhiên gõ từ bàn phím. Ít nhất 16 ký tự.
    * `File share` - Tên bất kỳ bạn thích hoặc các ký tự ngẫu nhiên gõ từ bàn phím. Ít nhất 16 ký tự.
    * `Region` - Chọn `(US) East US` từ danh sách thả xuống
    * Bấm `Create`, chờ triển khai xong, rồi chờ dấu nhắc lệnh bash xuất hiện.

> 💡 **Bổ sung:** Storage account và file share được dùng để lưu thư mục home (`$HOME`) của Cloud Shell, nhờ vậy các file bạn tạo (ví dụ repo đã clone, file terraform state) vẫn còn khi mở lại Cloud Shell trong cùng phiên lab.

Tiếp theo: [Cài đặt Terraform](./03-install-terraform.md)
