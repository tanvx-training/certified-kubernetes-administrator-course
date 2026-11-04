# Kubernetes Services
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/services-3/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **`Service`** trong Kubernetes

## Services
- Kubernetes Service cho phép giao tiếp giữa các thành phần khác nhau bên trong và bên ngoài ứng dụng.

  ![srv1](../../images/srv1.PNG)

> 💡 **Bổ sung:** Pod có vòng đời ngắn: mỗi khi Pod bị tạo lại, nó nhận một IP mới. Vì vậy không nên kết nối trực tiếp tới IP của Pod. Service cung cấp một địa chỉ ổn định (IP ảo + tên DNS) đứng trước một nhóm Pod (được chọn bằng label selector) và tự cân bằng tải giữa các Pod đó. Danh sách IP của các Pod phía sau được lưu trong object EndpointSlice (và Endpoints - API cũ, đã bị đánh dấu deprecated từ v1.33).
  
#### Hãy cùng xem một số khía cạnh khác về networking

## Giao tiếp từ bên ngoài (External Communication)

- Làm sao để một **`người dùng bên ngoài`** (external user) truy cập được **`trang web`**?

  - Từ chính node (truy cập được ứng dụng như mong đợi)
  
    ![srv2](../../images/srv2.PNG)
    
  - Từ thế giới bên ngoài (đây là điều ta mong muốn, nhưng nếu không có thứ gì đứng ở giữa thì sẽ không truy cập được ứng dụng)
  
    ![srv3](../../images/srv3.PNG)
   
    
 ## Các loại Service
 
 #### Kubernetes có 3 loại Service
 
   ![srv-types](../../images/srv-types.PNG)
 
 1. NodePort
    - Service ánh xạ một port nội bộ ra một port trên NODE để có thể truy cập được.
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: myapp-service
      spec:
       type: NodePort
       ports:
       - targetPort: 80
         port: 80
         nodePort: 30008
      ```
     ![srvnp](../../images/srvnp.PNG)

      > 💡 **Bổ sung:** Ba loại port trong NodePort Service (nhìn từ góc độ của Service):
      > - `targetPort`: port trên Pod, nơi ứng dụng thực sự lắng nghe. Nếu bỏ trống, mặc định bằng `port`.
      > - `port`: port của chính Service (trên ClusterIP). Đây là trường **bắt buộc** duy nhất.
      > - `nodePort`: port mở trên mọi node, nằm trong dải mặc định **30000-32767**. Nếu bỏ trống, Kubernetes tự chọn một port trống trong dải này.
      >
      > `ports` là một mảng, nên một Service có thể khai báo nhiều port. (Bản gốc ghi nhầm `types: NodePort` trong YAML ở trên - đã sửa thành `type`.)
      
      #### Kết nối Service với Pod
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: myapp-service
      spec:
       type: NodePort
       ports:
       - targetPort: 80
         port: 80
         nodePort: 30008
       selector:
         app: myapp
         type: front-end
       ```

    ![srvnp1](../../images/srvnp1.PNG)
      
      #### Tạo Service
      ```
      $ kubectl create -f service-definition.yaml
      ```
      
      #### Liệt kê các Service
      ```
      $ kubectl get services
      ```
      
      #### Truy cập ứng dụng từ CLI thay vì trình duyệt web
      ```
      $ curl http://192.168.1.2:30008
      ```
      
      ![srvnp2](../../images/srvnp2.PNG)

      #### Một Service với nhiều Pod
      
      ![srvnp3](../../images/srvnp3.PNG)
      
      #### Khi các Pod được phân bố trên nhiều node
     
      ![srvnp4](../../images/srvnp4.PNG)

      > 💡 **Bổ sung:** Khi có nhiều Pod khớp selector, Service tự động phân phối request tới các Pod đó (mặc định kube-proxy chọn ngẫu nhiên). Khi các Pod nằm trên nhiều node, Kubernetes mở cùng một `nodePort` trên **tất cả** các node, nên bạn có thể truy cập ứng dụng qua IP của bất kỳ node nào, kể cả node không chạy Pod đó. Bạn không cần cấu hình gì thêm.
     
            
 1. ClusterIP
    - Trong trường hợp này, Service tạo một **`IP ảo`** (Virtual IP) bên trong cluster để các service khác nhau giao tiếp với nhau, ví dụ giữa một nhóm frontend server và một nhóm backend server.
    
 1. LoadBalancer
    - Service sẽ cấp phát (provision) một **`load balancer`** cho ứng dụng trên các cloud provider được hỗ trợ.

> 💡 **Bổ sung:** ClusterIP là loại mặc định nếu không khai báo `type`. Ngoài 3 loại trên còn có `ExternalName` (ánh xạ Service tới một tên DNS bên ngoài thông qua bản ghi CNAME) và *headless Service* (`clusterIP: None`, thường dùng với StatefulSet). Trong kỳ thi, có thể tạo Service nhanh bằng `kubectl expose`, ví dụ `kubectl expose deployment myapp --type=NodePort --port=80 --name=myapp-service` (lưu ý `kubectl expose` không cho chỉ định `nodePort` - cần sinh YAML bằng `--dry-run=client -o yaml` rồi thêm `nodePort` vào).
    
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/services-networking/service/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

