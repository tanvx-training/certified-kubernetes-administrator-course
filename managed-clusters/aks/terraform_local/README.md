# Tạo AKS bằng Terraform từ máy local

Cập nhật lần cuối: tháng 7 năm 2025

Trong hướng dẫn này, chúng ta sẽ triển khai một AKS cluster tuân thủ các chính sách bảo mật và sử dụng của KodeKloud playground, bằng terraform chạy từ máy của người dùng. Nghĩa là chúng ta sẽ truy cập KodeKloud Playground bằng Terraform từ máy local và triển khai một AKS cluster. Điều duy nhất bạn cần lưu ý là phải tham chiếu location và resource group từ khối `locals`.

> 💡 **Bổ sung:** Tài khoản playground chỉ cho phép dùng resource group và region có sẵn, nên cấu hình Terraform lấy hai giá trị này qua `locals` thay vì tự tạo resource group mới. Nếu tự tạo resource group, bạn sẽ gặp lỗi không có quyền.

[Bấm vào đây](./docs/01-sign-in.md) để bắt đầu!
