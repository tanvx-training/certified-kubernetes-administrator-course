# Network Policies
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/network-policies-3/)
  
#### Luồng traffic qua một web server phục vụ frontend cho người dùng, một app server cung cấp backend API và một database server

  ![traffic](../../images/traffic.PNG)
  
- Có hai loại traffic:
  - Ingress: traffic đi **vào** một đối tượng (ví dụ request từ người dùng tới web server)
  - Egress: traffic đi **ra** từ một đối tượng (ví dụ request từ web server tới API server)
- Ingress/egress được xét theo hướng khởi tạo kết nối; traffic phản hồi (response) không cần rule riêng.
  
   ![ing1](../../images/ing1.PNG)
  
   ![ing2](../../images/ing2.PNG)
  
## Network Security
- Mặc định, Kubernetes áp dụng rule **"All Allow"**: mọi Pod có thể giao tiếp với mọi Pod hoặc Service khác trong cluster, bất kể namespace.

  ![nsec](../../images/nsec.PNG)
  
## Network Policy
- Nếu muốn, ví dụ, chỉ cho phép API Pod truy cập DB Pod (web Pod không được truy cập trực tiếp DB), ta tạo một Network Policy và gắn nó vào DB Pod.

  ![npol](../../images/npol.PNG)
  
  ![npol1](../../images/npol1.PNG)
  
## Network Policy Selectors
- Network Policy được gắn với Pod thông qua label và selector, tương tự như ReplicaSet hay Service.
  
  ![npolsec](../../images/npolsec.PNG)
  
## Network Policy Rules

  ![npol2](../../images/npol2.PNG)
  
## Tạo Network Policy
 
- Để tạo một Network Policy:
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
   name: db-policy
  spec:
    podSelector:
      matchLabels:
        role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
      - podSelector:
          matchLabels:
            role: api-pod
      ports:
      - protocol: TCP
        port: 3306
  ```
  
  ```
  $ kubectl create -f policy-definition.yaml
  ```
  
 ![npol3](../../images/npol3.PNG)
 
 ![npol4](../../images/npol4.PNG)

> 💡 **Bổ sung — các điểm hay bị nhầm khi thi:**
> - Chỉ khi một Pod được chọn bởi ít nhất một Network Policy với `policyTypes` tương ứng thì traffic loại đó mới bị cô lập (deny mặc định, trừ những gì rule cho phép). Ví dụ trên chỉ có `Ingress` nên egress của DB Pod vẫn không bị hạn chế.
> - Trong `from`/`to`, các selector nằm **cùng một phần tử** (cùng một dấu `-`) được kết hợp bằng **AND**, ví dụ `podSelector` + `namespaceSelector` = Pod có label đó **trong** namespace có label đó. Nếu tách thành **các phần tử riêng** (mỗi cái một dấu `-`) thì là **OR**.
> - Ngoài `podSelector` và `namespaceSelector` còn có `ipBlock` (dải CIDR, dùng cho đích bên ngoài cluster).
> - `podSelector: {}` chọn tất cả Pod trong namespace — thường dùng để tạo policy "default deny".
> - Khi viết policy egress, nhớ cho phép DNS (UDP/TCP port 53) nếu Pod cần phân giải tên Service.
> - Namespace có sẵn label `kubernetes.io/metadata.name: <tên-namespace>` (từ v1.22), tiện dùng cho `namespaceSelector`.
  
## Lưu ý
- Network Policy được thực thi bởi plugin mạng (CNI) của cluster. Không phải giải pháp mạng nào cũng hỗ trợ Network Policy: ví dụ Calico, Cilium, Kube-router, Weave Net có hỗ trợ, còn Flannel thì không. Nếu CNI không hỗ trợ, bạn vẫn tạo được Network Policy nhưng chúng sẽ không có tác dụng và cũng không có thông báo lỗi.
 
 ![note1](../../images/note1.PNG)
 
#### Bài giảng bổ sung về [Developing Networking Policies](https://kodekloud.com/topic/developing-network-policies/)

#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
 
  
  
  
  
