# Kubernetes Services - ClusterIP
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/services-cluster-ip-2/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **`Service - ClusterIP`** trong Kubernetes
         
## ClusterIP
- Trong trường hợp này, Service tạo một **`IP ảo`** (Virtual IP) bên trong cluster để các service khác nhau giao tiếp với nhau, ví dụ giữa một nhóm frontend server và một nhóm backend server.
    
    ![srvc1](../../images/srvc1.PNG)
    
#### Cách đúng để thiết lập kết nối giữa các service hay các tầng (tier) này là gì?
- Một Kubernetes Service giúp ta gom nhóm các Pod lại với nhau và cung cấp một giao diện duy nhất để truy cập các Pod trong nhóm.

  ![srvc2](../../images/srvc2.PNG)

> 💡 **Bổ sung:** Ví dụ ứng dụng có 3 tầng: front-end, back-end và redis. Các Pod front-end không cần biết IP của từng Pod back-end (vốn thay đổi liên tục); chúng chỉ cần gọi tới Service `back-end`. Service chuyển request tới một trong các Pod back-end. Mỗi tầng có thể scale hoặc di chuyển độc lập mà không ảnh hưởng tới tầng khác.
  
#### Tạo Service loại ClusterIP
```
apiVersion: v1
kind: Service
metadata:
 name: back-end
spec:
 type: ClusterIP
 ports:
 - targetPort: 80
   port: 80
 selector:
   app: myapp
   type: back-end
```
```
$ kubectl create -f service-definition.yaml
```

> 💡 **Bổ sung:** Bản gốc ghi nhầm `types: ClusterIP` - trường đúng là `type`. Vì `ClusterIP` là loại mặc định, bạn có thể bỏ hẳn dòng `type`. Các Pod khác trong cluster có thể truy cập Service này qua ClusterIP hoặc qua tên DNS `back-end` (cùng namespace) / `back-end.<namespace>.svc.cluster.local`. ClusterIP chỉ truy cập được từ **bên trong** cluster. Lệnh imperative tương đương: `kubectl create service clusterip back-end --tcp=80:80` (lệnh này đặt selector mặc định là `app=back-end`; muốn dùng selector của Pod sẵn có thì dùng `kubectl expose`).

#### Liệt kê các Service
```
$ kubectl get services
```
  ![srvc3](../../images/srvc3.PNG)
   
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/services-networking/service/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
