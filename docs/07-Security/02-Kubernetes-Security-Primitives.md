# Các cơ chế bảo mật cơ bản của Kubernetes (Security Primitives)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kubernetes-security-primitives/)
  
Trong phần này, chúng ta sẽ tìm hiểu các cơ chế bảo mật cơ bản (security primitives) của Kubernetes.

## Bảo mật các host

 ![sech](../../images/sech.PNG)

- Trước hết, các host tạo nên cluster phải được bảo mật: chỉ cho phép truy cập root qua SSH key thay vì mật khẩu, tắt các truy cập không cần thiết, v.v. Nếu host bị chiếm quyền thì mọi thứ chạy trên đó đều bị ảnh hưởng.
  
## Bảo mật Kubernetes
- kube-apiserver là trung tâm của mọi thao tác trong Kubernetes, vì vậy việc kiểm soát truy cập vào nó là tuyến phòng thủ đầu tiên. Chúng ta cần đưa ra hai loại quyết định:
  - Ai được phép truy cập?
  - Họ được phép làm gì?
 
  ![seck](../../images/seck.PNG)
  
## Authentication (Xác thực)
- Ai được phép truy cập API Server được quyết định bởi các cơ chế authentication.
  
## Authorization (Phân quyền)
- Sau khi đã truy cập được vào cluster, những gì họ được phép làm được quyết định bởi các cơ chế authorization.

> 💡 **Bổ sung:** Một request gửi tới kube-apiserver lần lượt đi qua 3 bước: **Authentication** (bạn là ai?) → **Authorization** (bạn có quyền làm việc này không? — ví dụ RBAC, Node authorizer, Webhook) → **Admission Control** (request có hợp lệ/cần chỉnh sửa không? — ví dụ `NodeRestriction`, Pod Security Admission). Chỉ khi qua cả 3 bước, đối tượng mới được lưu vào etcd.

## TLS Certificates
- Mọi giao tiếp với cluster, giữa các thành phần như etcd cluster, kube-controller-manager, kube-scheduler, kube-apiserver, cũng như các thành phần chạy trên worker node như kubelet và kube-proxy, đều được bảo mật bằng mã hóa TLS.

 ![tls](../../images/tls.PNG)
 
## Network Policy
Còn giao tiếp giữa các ứng dụng bên trong cluster thì sao? Mặc định, mọi Pod đều có thể kết nối với mọi Pod khác trong cluster. Ta có thể giới hạn điều này bằng Network Policy.

  ![np](../../images/np.PNG)
  
