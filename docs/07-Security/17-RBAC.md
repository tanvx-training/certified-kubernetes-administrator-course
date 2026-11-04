# RBAC
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/role-based-access-controls/)

Trong phần này, chúng ta sẽ tìm hiểu về RBAC (Role-Based Access Control — phân quyền dựa trên vai trò).

## Làm thế nào để tạo một Role?
- Mỗi rule trong Role gồm 3 phần:
  - apiGroups
  - resources
  - verbs
- Tạo Role bằng lệnh kubectl:
  ```
  $ kubectl create -f developer-role.yaml
  ```

## Bước tiếp theo là liên kết user với Role đó.
- Để làm việc này, ta tạo một object khác gọi là **`RoleBinding`**. Object RoleBinding liên kết một user với một Role.
- Tạo RoleBinding bằng lệnh kubectl:
  ```
  $ kubectl create -f devuser-developer-binding.yaml
  ```
- Lưu ý rằng Role và RoleBinding có phạm vi trong namespace (namespaced). Nếu không chỉ định `namespace` trong `metadata`, chúng sẽ được tạo trong namespace `default`.
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "list", "update", "delete", "create"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["create"]
  ```
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: devuser-developer-binding
  subjects:
  - kind: User
    name: dev-user # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
  ```
  ![rbac1](../../images/rbac1.PNG)

> ⚠️ **Lưu ý (sửa lỗi bản gốc):** Trong trường `resources`, tên resource phải viết thường, số nhiều — đúng như trong `kubectl api-resources`. Ví dụ trên ghi `"ConfigMap"` sẽ **không có tác dụng**, phải là `"configmaps"`. `apiGroups: [""]` là core API group (pods, configmaps, secrets, services...); với Deployment phải dùng `apiGroups: ["apps"]`.

> 💡 **Bổ sung — tạo nhanh bằng lệnh imperative (rất nên dùng khi thi):**
> - Tạo Role: `kubectl create role developer --verb=get,list,create,update,delete --resource=pods -n default`
> - Thêm giới hạn theo tên: `kubectl create role developer --verb=get --resource=pods --resource-name=blue --resource-name=orange`
> - Tạo RoleBinding: `kubectl create rolebinding devuser-developer-binding --role=developer --user=dev-user -n default`
> - Gắn cho ServiceAccount: `--serviceaccount=<namespace>:<sa-name>`; cho group: `--group=<group>`.
> - Có thể thêm `--dry-run=client -o yaml > file.yaml` để sinh YAML rồi chỉnh sửa.
  

## Xem thông tin RBAC
  
- Liệt kê các Role:
  ```
  $ kubectl get roles
  ```
- Liệt kê các RoleBinding:
  ```
  $ kubectl get rolebindings
  ```
- Xem chi tiết Role:
  ```
  $ kubectl describe role developer
  ```
  
  ![rbac2](../../images/rbac2.PNG)
    
- Xem chi tiết RoleBinding:
  ```
  $ kubectl describe rolebinding devuser-developer-binding
  ```
  
  ![rbac3](../../images/rbac3.PNG)
  
#### Nếu bạn là một user và muốn kiểm tra xem mình có quyền truy cập một resource cụ thể trong cluster hay không?
## Kiểm tra quyền truy cập

- Bạn có thể dùng lệnh `kubectl auth can-i`:
  ```
  $ kubectl auth can-i create deployments
  $ kubectl auth can-i delete nodes
  ```
- Là admin, bạn có thể giả lập (impersonate) một user khác để kiểm tra quyền của họ bằng `--as`:
  ```
  $ kubectl auth can-i create deployments --as dev-user
  $ kubectl auth can-i create pods --as dev-user
  ```
- Kiểm tra trong một namespace cụ thể:
  ```
  $ kubectl auth can-i create pods --as dev-user --namespace test
  ```
  
  ![rbac5](../../images/rbac5.PNG)

> 💡 **Bổ sung:**
> - Lệnh trả về `yes` hoặc `no`.
> - Kiểm tra quyền của ServiceAccount: `kubectl auth can-i list pods --as=system:serviceaccount:<namespace>:<sa-name>`.
> - Liệt kê toàn bộ quyền của một user trong namespace: `kubectl auth can-i --list --as dev-user -n default`.
> - Lệnh `--as` hoạt động được vì admin có quyền `impersonate`. Bạn cũng có thể chạy thử lệnh thật dưới danh nghĩa user đó, ví dụ `kubectl get pods --as dev-user`.
  
## Resource Names
- Ở trên ta đã thấy cách cấp quyền cho user trên các resource như Pod trong namespace. Bạn còn có thể giới hạn chặt hơn nữa: chỉ cho phép truy cập một số Pod cụ thể theo tên bằng trường `resourceNames`.
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "update", "create"]
    resourceNames: ["blue", "orange"]
  ```  
  ![rbac4](../../images/rbac4.PNG)

> 💡 **Bổ sung:** `resourceNames` không áp dụng được cho verb `create` (lúc tạo object, tên chưa được biết khi phân quyền), và với `list`/`watch` chỉ có tác dụng khi request có chọn đúng một object theo tên (field selector `metadata.name`). Vì vậy trong ví dụ trên, request `create` Pod sẽ **không khớp** rule này (request create không mang tên object nên không khớp `resourceNames`) — muốn cho phép tạo Pod, hãy tách `create` sang một rule riêng không có `resourceNames`.
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities
