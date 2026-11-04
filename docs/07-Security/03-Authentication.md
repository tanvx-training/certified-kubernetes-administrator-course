# Authentication (Xác thực)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/authentication/)
  
Trong phần này, chúng ta sẽ tìm hiểu về authentication trong một Kubernetes cluster.

## Các loại tài khoản (Accounts)

  ![auth1](../../images/auth1.PNG)
  
#### Có nhiều đối tượng khác nhau truy cập vào cluster. Riêng việc bảo mật cho end user (người dùng cuối truy cập vào các ứng dụng được deploy trên cluster) là do chính các ứng dụng đó tự quản lý bên trong.

 ![acc1](../../images/acc1.PNG)
 
- Vì vậy, chúng ta chỉ còn lại 2 loại người dùng:
  - Con người, chẳng hạn như Administrator và Developer (gọi là **User**)
  - "Robot", chẳng hạn như các process/service hoặc ứng dụng khác cần truy cập vào cluster (gọi là **ServiceAccount**).
  

  ![acc2](../../images/acc2.PNG)

> 💡 **Bổ sung:** Kubernetes **không** quản lý user account một cách native — không có object kiểu `User` nào để bạn `kubectl create user`. User được nhận diện từ bên ngoài (client certificate, token, OIDC...). Ngược lại, ServiceAccount là một object thực sự trong Kubernetes, có thể tạo bằng `kubectl create serviceaccount <tên>`.
  
- Mọi truy cập của user đều được quản lý bởi kube-apiserver, và mọi request đều đi qua kube-apiserver.
 
  ![acc3](../../images/acc3.PNG)
  
## Các cơ chế Authentication
- Có nhiều cơ chế authentication khác nhau có thể cấu hình.

  ![auth2](../../images/auth2.PNG)
  
## Cơ chế Authentication - Basic
  
  ![auth3](../../images/auth3.PNG)
  
## Cấu hình kube-apiserver
- Nếu cluster được dựng bằng kubeadm thì cập nhật file manifest `kube-apiserver.yaml` với tùy chọn tương ứng.
  
  ![auth4](../../images/auth4.PNG)
  
## Xác thực User

- Để xác thực bằng thông tin basic khi truy cập API server, chỉ định username và password trong lệnh curl.
  ```
  $ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
  ```
  ![auth5](../../images/auth5.PNG)
  
- Ta có thể thêm một cột trong file `user-details.csv` để gán user vào các group cụ thể.

  ![auth6](../../images/auth6.PNG)
  
## Lưu ý
 
 ![note](../../images/note.PNG)

> 💡 **Bổ sung (quan trọng):** Cơ chế xác thực bằng file mật khẩu tĩnh (`--basic-auth-file`) đã bị **loại bỏ hoàn toàn từ Kubernetes v1.19**, nên chỉ mang tính minh họa. File token tĩnh (`--token-auth-file`) vẫn tồn tại nhưng không được khuyến nghị. Trong thực tế và trong kỳ thi CKA, các cơ chế thường dùng là:
> - **X.509 client certificate** (`--client-ca-file`): CN của certificate là username, O là group — đây là cách kubeadm cấp quyền cho admin.
> - **ServiceAccount token** (JWT) cho các ứng dụng chạy trong Pod.
> - **OIDC** / webhook token authentication để tích hợp với Identity Provider bên ngoài.
>
> Lưu ý thêm: API server lắng nghe HTTPS trên cổng 6443, nên trong thực tế URL của lệnh curl ở trên phải là `https://...`.
  
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/authentication/ 
  
  
