# Authorization (Phân quyền)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/authorization/)
  
Trong phần này, chúng ta sẽ tìm hiểu về authorization (phân quyền) trong Kubernetes.

> 💡 **Bổ sung:** Authentication (xác thực) trả lời câu hỏi "bạn là ai?", còn Authorization trả lời câu hỏi "bạn được phép làm gì?". Mỗi request tới kube-apiserver lần lượt đi qua: Authentication → Authorization → Admission Control → ghi vào etcd.

## Vì sao cần Authorization trong cluster?
- Với tư cách admin, bạn có thể thực hiện mọi thao tác:
  ```
  $ kubectl get nodes
  $ kubectl get pods
  $ kubectl delete node worker-2
  ```
  
  ![at1](../../images/at1.PNG)

- Nhưng khi có thêm người dùng khác (developer, tester...), ứng dụng (thông qua ServiceAccount) hay các thành phần khác truy cập cluster, ta cần giới hạn họ chỉ được làm những gì cần thiết. Ví dụ: developer được xem/tạo Pod nhưng không được xóa node hay sửa cấu hình cluster.
  
## Các cơ chế Authorization
- Kubernetes hỗ trợ nhiều cơ chế authorization khác nhau:
  - Node Authorization
  - Attribute-based Authorization (ABAC)
  - Role-Based Authorization (RBAC)
  - Webhook
  
## Node Authorization
- Dùng để phân quyền cho các request từ kubelet tới kube-apiserver (đọc thông tin Service, Endpoint, Node, Pod; ghi trạng thái node, trạng thái Pod, event...). Kubelet phải thuộc group `system:nodes` và có tên dạng `system:node:<node-name>` (thông tin này nằm trong certificate của kubelet). Các request như vậy được xử lý bởi **Node Authorizer**.

  ![node-auth](../../images/node-auth.png)
  
## ABAC
- Gán quyền trực tiếp cho từng user hoặc group thông qua một file policy (định dạng JSON, mỗi dòng một policy) được truyền vào kube-apiserver. Mỗi lần thay đổi quyền, bạn phải sửa file policy và **khởi động lại kube-apiserver** → khó quản lý khi cluster lớn.

  ![abac](../../images/abac.PNG)
  
## RBAC
- Thay vì gán quyền trực tiếp cho từng user, ta định nghĩa một Role với tập quyền cụ thể (ví dụ "developer"), sau đó gắn (bind) user/group vào Role đó. Khi cần thay đổi quyền, chỉ cần sửa Role và thay đổi sẽ áp dụng ngay cho mọi user liên quan. Đây là cách quản lý phân quyền chuẩn trong Kubernetes.

  ![rbac](../../images/rbac.PNG)

## Webhook
- Khi muốn giao việc phân quyền cho một công cụ bên ngoài (ví dụ Open Policy Agent), kube-apiserver sẽ gửi thông tin request tới dịch vụ đó và dịch vụ trả lời request có được phép hay không.
  
  ![webhook](../../images/webhook.PNG)
  
## Authorization Modes
- Các mode được cấu hình bằng tham số `--authorization-mode` trên kube-apiserver. Ngoài 4 cơ chế trên còn có 2 mode đặc biệt: `AlwaysAllow` (cho phép mọi request — mặc định nếu không cấu hình) và `AlwaysDeny` (từ chối mọi request).

  ![mode](../../images/mode.PNG)
  
- Khi chỉ định nhiều mode (phân cách bằng dấu phẩy), request sẽ được kiểm tra lần lượt theo đúng thứ tự khai báo. Nếu một module từ chối (deny/no opinion), request được chuyển tới module tiếp theo; ngay khi một module chấp thuận (allow), request được cho phép và không kiểm tra tiếp.

  ![mode1](../../images/mode1.PNG)

> 💡 **Bổ sung:** Cluster dựng bằng kubeadm mặc định dùng `--authorization-mode=Node,RBAC`. Để kiểm tra nhanh trong lab/kỳ thi: `grep authorization-mode /etc/kubernetes/manifests/kube-apiserver.yaml` hoặc `kubectl describe pod kube-apiserver-controlplane -n kube-system | grep authorization-mode`. Từ v1.30, kube-apiserver còn hỗ trợ cấu hình authorization qua file bằng `--authorization-config` (Structured Authorization Configuration), cho phép khai báo nhiều webhook — nhưng không dùng chung với `--authorization-mode`.
  
  
  #### Tài liệu tham khảo K8s
  - https://kubernetes.io/docs/reference/access-authn-authz/authorization/
  
