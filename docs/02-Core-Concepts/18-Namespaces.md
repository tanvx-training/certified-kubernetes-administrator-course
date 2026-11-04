# Namespaces
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/namespaces/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **`Namespaces`**

Cho đến giờ trong khóa học, chúng ta đã tạo các **`Object`** như **`Pod`**, **`Deployment`** và **`Service`** trong cluster. Tất cả những gì ta làm đều diễn ra bên trong một **`NAMESPACE`**.
- Namespace này là namespace **`default`** của Kubernetes. Nó được tạo tự động khi Kubernetes được cài đặt lần đầu.

  ![ns](../../images/ns.PNG)

> 💡 **Bổ sung:** Namespace là cơ chế phân vùng logic (cách ly theo tên) các resource bên trong một cluster. Ngoài `default`, Kubernetes còn tự tạo:
> - `kube-system`: chứa các thành phần hệ thống (CoreDNS, kube-proxy, các static Pod của control plane...).
> - `kube-public`: resource có thể đọc công khai, kể cả với người dùng chưa xác thực.
> - `kube-node-lease`: chứa các object Lease dùng làm heartbeat của node, giúp control plane phát hiện node bị lỗi.
>
> Lưu ý không phải resource nào cũng thuộc namespace: Node, PersistentVolume, StorageClass, ClusterRole, Namespace... là resource cấp cluster. Kiểm tra bằng `kubectl api-resources --namespaced=false`.
 
- Bạn cũng có thể tạo namespace của riêng mình.

  ![ns3](../../images/ns3.PNG)
  
- Liệt kê các Pod trong namespace default
  ```
  $ kubectl get pods
  ```
- Liệt kê các Pod trong một namespace khác: dùng lệnh **`kubectl get pods`** kèm flag **`--namespace`**.
  ```
  $ kubectl get pods --namespace=kube-system
  ```
  ![ns8](../../images/ns8.PNG)
  
- Ở đây ta có một file định nghĩa Pod. Khi tạo Pod bằng file này, Pod sẽ được tạo trong namespace default.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
     app: myapp
     type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
 ```
  ```
  $ kubectl create -f pod-definition.yaml
  ```
- Để tạo Pod từ file định nghĩa trong một namespace khác, dùng tùy chọn **`--namespace`**.
  ```
  $ kubectl create -f pod-definition.yaml --namespace=dev
  ```
  ![ns9](../../images/ns9.PNG)

- Nếu muốn đảm bảo Pod này luôn được tạo trong môi trường **`dev`**, kể cả khi không chỉ định trên dòng lệnh, bạn có thể đưa phần khai báo namespace vào trong file định nghĩa Pod.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
  labels:
     app: myapp
     type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
 ```
  
  ![ns10](../../images/ns10.PNG)
  
- Để tạo namespace mới, viết file định nghĩa namespace như dưới đây rồi chạy **`kubectl create`**
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

  ```
  $ kubectl create -f namespace-dev.yaml
  ```
  Một cách khác để tạo namespace
  ```
  $ kubectl create namespace dev
  ```
  ![ns11](../../images/ns11.PNG)
  
- Mặc định, chúng ta ở trong namespace **`default`**. Để chuyển hẳn sang một namespace cụ thể, chạy lệnh dưới đây.
  ```
  $ kubectl config set-context $(kubectl config current-context) --namespace=dev
  ```

> 💡 **Bổ sung:** Cách viết ngắn tương đương: `kubectl config set-context --current --namespace=dev`. Kiểm tra namespace đang dùng: `kubectl config view --minify | grep namespace`. Trong kỳ thi CKA, mỗi câu hỏi thường chỉ định namespace cụ thể - an toàn nhất là luôn thêm `-n <namespace>` vào lệnh thay vì đổi context mặc định.

- Xem Pod trong tất cả các namespace
  ```
  $ kubectl get pods --all-namespaces
  ```
  ![ns12](../../images/ns12.PNG)

> 💡 **Bổ sung:** `--all-namespaces` có thể viết tắt là `-A`, và `--namespace` viết tắt là `-n`, ví dụ `kubectl get pods -A`, `kubectl get pods -n kube-system`.
  
- Để giới hạn tài nguyên trong một namespace, hãy tạo resource quota. Bắt đầu bằng một file định nghĩa **`ResourceQuota`**.
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
  ```
  $ kubectl create -f compute-quota.yaml
  ```
  ![ns13](../../images/ns13.PNG)

> 💡 **Bổ sung:** Khi namespace có ResourceQuota giới hạn `requests.cpu`/`requests.memory` (hoặc `limits.*`), mọi Pod mới tạo trong namespace đó **bắt buộc** phải khai báo requests/limits tương ứng, nếu không sẽ bị từ chối. Có thể dùng LimitRange để đặt giá trị mặc định. Xem mức sử dụng quota bằng `kubectl describe resourcequota -n dev`.
  
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/
- https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/
  

