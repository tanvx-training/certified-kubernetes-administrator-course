# Khởi tạo lab

Cập nhật lần cuối: tháng 3 năm 2024

### Khởi tạo Azure playground lab

Điều kiện tiên quyết: có gói đăng ký KodeKloud Pro (hoặc cao hơn)

Truy cập link sau: https://kodekloud.com/playgrounds/playground-azure và bấm `START LAB` để yêu cầu một Azure Playground instance mới. Sau vài giây, bạn sẽ nhận được một tài khoản Azure portal tạm thời như sau:

![image](../../images/01-sign-in.png)

Sao chép **Console Link** bằng cách bấm nút copy ở bên phải của nó, dán vào trình duyệt và đăng nhập bằng thông tin đăng nhập được cung cấp. Nếu bạn được yêu cầu chọn tài khoản như trong hộp thoại dưới đây, hãy chọn **Use another account**

1. Tại ô đăng nhập, sao chép `Username` và dán vào đó, sau đó bấm Next.
1. Bây giờ sao chép `Password` và dán vào ô Temporary Access Pass.
1. Sao chép `Application Client ID`, giá trị này cần được gán cho biến `TF_VAR_ARM_CLIENT_ID`
1. Sao chép `Client Secret`, giá trị này cần được gán cho biến `TF_VAR_ARM_CLIENT_SECRET`

Bỏ qua các phần giới thiệu cách sử dụng và Start a Tour, trừ khi bạn quan tâm tới chúng.

1. Sao chép `Subscription ID` → Tìm trong Azure Portal → Subscriptions → Subscription ID, giá trị này cần được gán cho biến `TF_VAR_ARM_SUBSCRIPTION_ID`
2. Sao chép `Tenant ID` → Tìm trong Azure Portal → Microsoft Entra ID → Overview → Tenant ID, giá trị này cần được gán cho biến `TF_VAR_ARM_TENANT_ID`

> 💡 **Bổ sung:** Bốn giá trị trên là thông tin xác thực của một **service principal** (danh tính dành cho ứng dụng trong Microsoft Entra ID, trước đây gọi là Azure AD). Terraform dùng chúng để đăng nhập Azure thay cho bạn. Hãy giữ bí mật `Client Secret` - không commit nó vào git.

Tiếp theo: [Cài đặt phần mềm](./02-install-softwares.md)
