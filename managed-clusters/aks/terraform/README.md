# Tạo AKS bằng Terraform

Cập nhật lần cuối: tháng 7 năm 2025

Trong hướng dẫn này, chúng ta sẽ triển khai một AKS cluster tuân thủ các chính sách bảo mật và sử dụng của KodeKloud playground, bằng terraform chạy ngay trong Cloud Shell. Chạy Cloud Shell từ trong Azure console có ưu điểm là toàn bộ thông tin xác thực và chi tiết project đã được thiết lập sẵn trong môi trường.

> 💡 **Bổ sung:** Terraform là công cụ Infrastructure as Code (IaC - hạ tầng dưới dạng code) của HashiCorp. Thay vì bấm từng bước trên console, bạn mô tả hạ tầng mong muốn trong các file `.tf`, rồi Terraform sẽ tạo/cập nhật/xóa tài nguyên tương ứng. Ưu điểm lớn: có thể dựng lại và xóa sạch cluster chỉ với một lệnh.


[Bấm vào đây](./docs/01-sign-in.md) để bắt đầu!
