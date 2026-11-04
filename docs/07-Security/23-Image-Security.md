# Image Security
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/image-security/)

Trong phần này, chúng ta sẽ tìm hiểu về bảo mật image (image security).

# Image
   
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: nginx
  ```

- Tên image `nginx` thực chất là cách viết tắt của `docker.io/library/nginx`: `docker.io` là registry mặc định (Docker Hub), `library` là account/repository mặc định chứa các image chính thức, còn `nginx` là tên image/repository. Nếu không ghi tag, mặc định là `latest`.
  
  ![img1](../../images/img1.PNG)

- Ngoài Docker Hub còn nhiều registry public khác, ví dụ `gcr.io`, `registry.k8s.io` (registry chứa image của Kubernetes)...
  
  ![img2](../../images/img2.PNG)
  
# Private Registry
- Với các ứng dụng nội bộ không muốn công khai, ta dùng private registry. Để đăng nhập vào registry:
  ```
  $ docker login private-registry.io
  ```
- Chạy ứng dụng bằng image có trong private registry:
  ```
  $ docker run private-registry.io/apps/internal-app
  ```
  
  ![prvr](../../images/prvr.PNG)
  
- Trong Kubernetes, image được kéo về bởi container runtime trên worker node. Để truyền thông tin đăng nhập cho container runtime trên worker node, trước tiên ta tạo một Secret có type `docker-registry` chứa thông tin đăng nhập:
  ```
  $ kubectl create secret docker-registry regcred \
    --docker-server=private-registry.io \ 
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry-user@org.com
  ```
- Sau đó khai báo Secret này trong file định nghĩa Pod, dưới mục `imagePullSecrets`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
    imagePullSecrets:
    - name: regcred
  ```
  ![prvr1](../../images/prvr1.PNG)

> 💡 **Bổ sung:**
> - Secret tạo ra có type `kubernetes.io/dockerconfigjson`. Secret phải nằm **cùng namespace** với Pod.
> - `imagePullSecrets` nằm ở cấp `spec` của Pod (ngang hàng với `containers`), không phải bên trong từng container. Với Deployment thì nằm ở `spec.template.spec.imagePullSecrets`.
> - Thay vì khai báo cho từng Pod, có thể gắn `imagePullSecrets` vào ServiceAccount; mọi Pod dùng ServiceAccount đó sẽ tự động dùng Secret này, ví dụ: `kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "regcred"}]}'`.
> - Từ v1.24 Kubernetes đã gỡ dockershim, nên các node thường dùng containerd/CRI-O thay vì Docker — nhưng cơ chế `imagePullSecrets` vẫn hoạt động như cũ vì kubelet truyền thông tin đăng nhập cho container runtime qua CRI.
  
  #### Tài liệu tham khảo K8s
  - https://kubernetes.io/docs/concepts/containers/images/
