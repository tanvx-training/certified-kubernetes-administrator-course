# Cluster Roles
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/cluster-roles/)
  
Trong phần này, chúng ta sẽ tìm hiểu về ClusterRole.

## Roles
- Role và RoleBinding là namespaced, nghĩa là chúng được tạo bên trong một namespace (nếu không chỉ định, mặc định là namespace `default`) và chỉ cấp quyền trong namespace đó.
  
  ![roles](../../images/roles.PNG)
  
## Namespaces
- Có thể nhóm hoặc cô lập các node trong một namespace không?
  - Không. Node là resource có phạm vi toàn cluster (cluster-wide hay cluster-scoped). Chúng không thể gắn với bất kỳ namespace cụ thể nào.
  
  ![namespace](../../images/namespace.PNG)
  
- Vì vậy, resource được chia thành hai loại: namespaced (Pod, Deployment, Service, ConfigMap, Secret, Role, RoleBinding, PVC...) và cluster-scoped (Node, PV, Namespace, ClusterRole, ClusterRoleBinding, CertificateSigningRequest, StorageClass...).
  
- Xem danh sách resource namespaced:
  ```
  $ kubectl api-resources --namespaced=true
  ```
- Xem danh sách resource không thuộc namespace (cluster-scoped):
  ```
  $ $ kubectl api-resources --namespaced=false
  ```
  
  ![namespace1](../../images/namespace1.PNG)
  
## ClusterRole và ClusterRoleBinding
- ClusterRole cũng giống Role, nhưng dùng cho resource cluster-scoped. Kind là **`ClusterRole`** (bản gốc gõ nhầm thành `CLusterRole`).
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: cluster-administrator
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["nodes"]
    verbs: ["get", "list", "delete", "create"]
  ```
- Sau đó, dùng ClusterRoleBinding để liên kết user với ClusterRole:
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluster-admin-role-binding
  subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-administrator
    apiGroup: rbac.authorization.k8s.io
  ```
  ```
  $ kubectl create -f cluster-admin-role.yaml
  $ kubectl create -f cluster-admin-role-binding.yaml
  ```
  
 ![cr1](../../images/cr1.PNG)
  
- Bạn cũng có thể tạo ClusterRole cho resource namespaced. Khi đó (gắn bằng ClusterRoleBinding), user sẽ có quyền truy cập các resource này trên tất cả namespace.

> 💡 **Bổ sung:**
> - Lệnh imperative: `kubectl create clusterrole node-admin --verb=get,list,watch,create,delete --resource=nodes` và `kubectl create clusterrolebinding michelle-binding --clusterrole=node-admin --user=michelle`.
> - Các tổ hợp hợp lệ: Role + RoleBinding (quyền trong một namespace); ClusterRole + ClusterRoleBinding (quyền toàn cluster); **ClusterRole + RoleBinding** (tái sử dụng một ClusterRole nhưng chỉ cấp quyền trong namespace của RoleBinding). Không thể dùng ClusterRoleBinding để tham chiếu tới một Role.
> - `roleRef` của binding là bất biến (immutable): muốn đổi Role được tham chiếu, bạn phải xóa và tạo lại binding.
> - Kubernetes có sẵn các ClusterRole mặc định như `cluster-admin`, `admin`, `edit`, `view`. Ví dụ: `kubectl create rolebinding dev-view --clusterrole=view --user=dev-user -n dev`.

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities
  
  
