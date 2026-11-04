# Certificate API
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/certificates-api/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách quản lý certificate và Certificate API trong Kubernetes.

## CA (Certificate Authority)
- CA thực chất chỉ là cặp file key và certificate mà chúng ta đã tạo. Bất kỳ ai có được cặp file này đều có thể ký bất kỳ certificate nào cho môi trường Kubernetes, vì vậy chúng phải được lưu trữ an toàn (với kubeadm, CA nằm trên master node tại `/etc/kubernetes/pki/ca.crt` và `ca.key`).

#### Kubernetes có sẵn một Certificate API có thể làm việc này cho bạn.
- Với Certificate API, ta gửi một certificate signing request (CSR) trực tiếp tới Kubernetes thông qua một lời gọi API, thay vì phải đăng nhập vào master node và tự ký bằng openssl.
   
  ![csr](../../images/csr.PNG)
   
#### Certificate sau khi được ký có thể được lấy ra và chia sẻ cho user.
- Đầu tiên user tạo một key
  ```
  $ openssl genrsa -out jane.key 2048
  ```
- Tạo một CSR
  ```
  $ openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
  ```
- Gửi CSR cho administrator; administrator dùng nó để tạo một object CSR có kind là "CertificateSigningRequest", trong đó trường `request` chứa nội dung file "jane.csr" đã được mã hóa base64
  ```
  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: jane
  spec:
    groups:
    - system:authenticated
    usages:
    - digital signature
    - key encipherment
    - server auth
    request:
      <certificate-goes-here>
  ```
  ```
  $ cat jane.csr |base64 
  $ kubectl create -f jane.yaml
  ```
 ![csr1](../../images/csr1.PNG)

> 💡 **Bổ sung (quan trọng):** API `certificates.k8s.io/v1beta1` đã bị **gỡ bỏ từ v1.22**. Hiện nay phải dùng `certificates.k8s.io/v1`, trong đó trường `signerName` là **bắt buộc**. Để cấp client certificate cho user, dùng signer `kubernetes.io/kube-apiserver-client` và usage `client auth` (không phải `server auth` như ví dụ cũ ở trên). Ví dụ:
>
> ```yaml
> apiVersion: certificates.k8s.io/v1
> kind: CertificateSigningRequest
> metadata:
>   name: jane
> spec:
>   request: <nội dung của: cat jane.csr | base64 -w 0>
>   signerName: kubernetes.io/kube-apiserver-client
>   expirationSeconds: 86400   # tùy chọn, 1 ngày
>   usages:
>   - client auth
> ```
>
> Mẹo thi: dùng `base64 -w 0` để mã hóa trên **một dòng** (không xuống dòng), tránh lỗi khi dán vào YAML. Trường `groups` trong spec được API server tự điền theo người tạo request, không cần khai báo.
  
- Liệt kê các CSR
  ```
  $ kubectl get csr
  ```
- Phê duyệt (approve) request
  ```
  $ kubectl certificate approve jane
  ```
- Xem certificate đã được cấp
  ```
  $ kubectl get csr jane -o yaml
  ```
- Giải mã certificate
  ```
  $ echo "<certificate>" |base64 --decode
  ```
  
  ![csr2](../../images/csr2.PNG)

> 💡 **Bổ sung:** Certificate đã ký nằm trong trường `status.certificate` của object CSR. Có thể lấy ra và lưu thành file trong một lệnh: `kubectl get csr jane -o jsonpath='{.status.certificate}' | base64 -d > jane.crt`. Để từ chối một request dùng `kubectl certificate deny <tên>`. Sau khi có `jane.crt`, user vẫn chưa có quyền gì — cần tạo Role/RoleBinding (RBAC) cho user `jane`, rồi có thể thêm vào kubeconfig bằng `kubectl config set-credentials jane --client-certificate=jane.crt --client-key=jane.key --embed-certs=true`.
  
#### Mọi thao tác liên quan đến certificate đều do kube-controller-manager thực hiện.
- Bất kỳ ai muốn ký certificate đều cần root certificate và private key của CA server. Cấu hình của kube-controller-manager có hai tùy chọn để chỉ định chúng: `--cluster-signing-cert-file` và `--cluster-signing-key-file`.

  ![csr3](../../images/csr3.PNG)
  
  ![csr4](../../images/csr4.PNG)
  
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
