# Khởi tạo lab

Cập nhật lần cuối: tháng 12 năm 2024

Lưu ý rằng trong ba loại managed cluster, EKS là loại phức tạp nhất để dựng xét về số bước cần thực hiện. Hãy đặc biệt chú ý tới từng bước, vì nếu bạn làm sai sẽ gặp lỗi. *Hướng dẫn này hoạt động được*, và khi nó không hoạt động thì đó là vì bạn đã bỏ sót điều gì đó - khi ấy tốt nhất là kết thúc lab và làm lại từ đầu! Dù vậy, AWS thỉnh thoảng có sắp xếp lại các màn hình và luồng thao tác trên console. Nếu bạn thấy những gì hiển thị trên console khác với hướng dẫn này, vui lòng báo lại trên Discord hoặc [Community Forum](https://kodekloud.com/community/c/kubernetes/6).

### Khởi tạo AWS playground lab

Điều kiện tiên quyết: có gói đăng ký KodeKloud Premium

Truy cập link sau: https://kodekloud.com/playgrounds/playground-aws và bấm `LAUNCH NOW` để yêu cầu một AWS Playground instance mới. Sau vài giây, bạn sẽ nhận được một tài khoản AWS portal tạm thời như sau:

![image](../images/01-sign-in.png)

Sao chép **Console Link** bằng cách bấm nút copy ở bên phải của nó, dán vào trình duyệt và đăng nhập bằng thông tin đăng nhập được cung cấp. Tại màn hình đăng nhập:
* Dán username được cấp vào trường `IAM user name`.
* Dán password vào trường `Password`.
* Bỏ chọn checkbox `Remember this account` nếu nó đang được chọn.
* Bấm nút `Sign in`.

Sau khi đăng nhập, hãy đảm bảo region được đặt là `us-east-1` (N. Virginia). Nếu không hiển thị như vậy, hãy bấm vào danh sách thả xuống của region (được chỉ bằng mũi tên) và chọn `US East (N. Virginia) us-east-1` từ menu.

![](../images/01-region.png)

> 💡 **Bổ sung:** Region rất quan trọng trên AWS: hầu hết tài nguyên (EKS cluster, EC2 instance, key pair, CloudFormation stack...) chỉ tồn tại trong region nơi chúng được tạo. Nếu lỡ chuyển sang region khác, bạn sẽ không thấy tài nguyên đã tạo, và các lệnh ở các bước sau (dùng `--region us-east-1`) sẽ không tìm thấy cluster.

Tiếp theo: [Tạo Service Role](./02-create-service-role.md)



