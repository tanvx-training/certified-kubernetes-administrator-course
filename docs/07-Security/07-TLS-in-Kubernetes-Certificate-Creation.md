# TLS trong Kubernetes - Tạo Certificate
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/tls-in-kubernetes-certificate-creation/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách tạo TLS certificate trong Kubernetes.

## Tạo Certificate
- Có nhiều công cụ để tạo certificate như easyrsa, openssl, cfssl, v.v. Trong khóa học này chúng ta dùng openssl.

## Certificate Authority (CA)

- Tạo key
  ```
  $ openssl genrsa -out ca.key 2048
  ```
- Tạo CSR
  ```
  $ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
  ```
- Ký certificate (CA tự ký bằng chính private key của nó — self-signed)
  ```
  $ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  ```
 
 ![ca1](../../images/ca1.PNG)
 
## Tạo Client Certificate

#### Certificate cho Admin User

- Tạo key
  ```
  $ openssl genrsa -out admin.key 2048
  ```
- Tạo CSR
  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
  ```
- Ký certificate (lần này ký bằng cặp key/certificate của CA)
  ```
  $ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  ```
  
  ![ca2](../../images/ca2.PNG)
  
- Certificate có quyền admin: thêm group `system:masters` vào trường O (Organization)
  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
  ```

> 💡 **Bổ sung:** kube-apiserver lấy **CN** làm username và **O** làm group. Group `system:masters` được gắn với ClusterRole `cluster-admin` và bỏ qua mọi kiểm tra RBAC. Từ v1.29, kubeadm tạo `admin.conf` với group `kubeadm:cluster-admins` (được bind tới `cluster-admin` qua RBAC) và tách riêng `super-admin.conf` dùng `system:masters`, để có thể thu hồi quyền admin thông thường khi cần.
>
> Lệnh `openssl x509 -req` mặc định chỉ cấp certificate có hạn 30 ngày; thêm `-days <số ngày>` để đặt thời hạn. Lần đầu ký bằng CA có thể thêm `-CAcreateserial` để tạo file serial `ca.srl`.
  
#### Làm tương tự để tạo client certificate cho tất cả các thành phần khác truy cập kube-apiserver.

  ![crt1](../../images/crt1.PNG)
  
  ![crt2](../../images/crt2.PNG)
  
  ![crt3](../../images/crt3.PNG)
   
  ![crt4](../../images/crt4.PNG)

- Lưu ý: các thành phần hệ thống như kube-scheduler, kube-controller-manager, kube-proxy cần có tiền tố `system:` trong CN (vd `system:kube-scheduler`).
  
## Tạo Server Certificate

## Certificate cho etcd server

  ![etc1](../../images/etc1.PNG)
  
  ![etc2](../../images/etc2.PNG)
  
## Certificate cho kube-apiserver

  ![api1](../../images/api1.PNG)
  
  ![api2](../../images/api2.PNG)

- kube-apiserver được truy cập qua nhiều tên khác nhau (`kubernetes`, `kubernetes.default`, `kubernetes.default.svc`, `kubernetes.default.svc.cluster.local`, IP của node, IP ClusterIP của Service `kubernetes`...), nên tất cả các tên/IP này phải được khai báo trong phần **Subject Alternative Name (SAN)** của certificate thông qua file cấu hình openssl.
  
## Kubelet trên các node (Server Cert)

   ![kctl1](../../images/kctl1.PNG)
   
## Kubelet trên các node (Client Cert)

   ![kctl2](../../images/kctl2.PNG)

> 💡 **Bổ sung:** Tiêu đề gốc ghi "Kubectl Nodes" nhưng nội dung thực ra nói về **kubelet**. Client certificate của kubelet có CN dạng `system:node:<tên-node>` và thuộc group `system:nodes` để Node authorizer nhận diện. Trong cluster dựng bằng kubeadm, bạn không phải tự làm thủ công: kubelet được cấp certificate qua cơ chế TLS bootstrapping và tự động xoay vòng (rotate) client certificate.
