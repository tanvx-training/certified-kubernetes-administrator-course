# API Groups
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/api-groups/)
  
Trong phần này, chúng ta sẽ tìm hiểu về API Group trong Kubernetes.

## Lấy thông tin version và liệt kê Pod qua API

 ![api3](../../images/api3.PNG)
 
- Kubernetes API được chia thành nhiều nhóm theo mục đích sử dụng, chẳng hạn một nhóm cho **`APIs`**, các nhóm khác cho **`healthz`**, **`metrics`**, **`logs`**, v.v.

  ![api4](../../images/api4.PNG)
 
## API và APIs
- Các API này được chia thành hai loại:
  - Core group (`/api`, ví dụ `/api/v1`) — nơi chứa các chức năng cốt lõi như Pod, Namespace, Service, ConfigMap, Secret, Node, PV, PVC...
    
    ![api5](../../images/api5.PNG)
 
  - Named group (`/apis`) — được tổ chức rõ ràng hơn, và về sau mọi tính năng mới đều được đưa vào các named group này (ví dụ `apps`, `networking.k8s.io`, `rbac.authorization.k8s.io`, `certificates.k8s.io`).
  
    ![api6](../../images/api6.PNG)
    
- Liệt kê tất cả API group

  ![api7](../../images/api7.PNG)

> 💡 **Bổ sung:** Mỗi API group chứa các **resource** (vd `deployments`, `replicasets`), và mỗi resource hỗ trợ một tập **verb** (`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`). Đây chính là các khái niệm được dùng khi viết Role trong RBAC (`apiGroups`, `resources`, `verbs`). Dùng `kubectl api-resources` để xem mọi resource cùng API group, tên viết tắt và việc resource có thuộc namespace hay không; `kubectl api-versions` để liệt kê các group/version hiện có. Lưu ý: trong Role, core group được ghi là chuỗi rỗng `""`.
  
## Lưu ý khi truy cập kube-apiserver
- Bạn phải xác thực bằng cách truyền vào các file certificate.

  ![api8](../../images/api8.PNG)
  
- Một cách khác là khởi chạy client **`kubectl proxy`** (bản gốc ghi `kubeproxy`): nó dùng thông tin xác thực trong kubeconfig của bạn và mở một proxy trên localhost cổng 8001, nhờ đó bạn có thể gọi API bằng curl mà không cần truyền certificate.
  
  ![api9](../../images/api9.PNG)
  
## kube-proxy và kubectl proxy
- Hai thứ này hoàn toàn khác nhau: **kube-proxy** chạy trên mỗi node để thiết lập kết nối mạng cho Service (giữa các Pod trên các node khác nhau), còn **kubectl proxy** là một HTTP proxy do kubectl tạo ra để truy cập kube-apiserver.
 
  ![kp](../../images/kp.PNG)
  
## Những điểm chính cần nhớ

  ![api10](../../images/api10.PNG)

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/overview/kubernetes-api/
- https://kubernetes.io/docs/reference/using-api/api-concepts/
- https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/
