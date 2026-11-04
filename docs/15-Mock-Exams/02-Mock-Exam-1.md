# Mock Exam 1

  Kiểm tra kiến thức của bạn, đi tới [Mock Exam 1](https://kodekloud.com/topic/mock-exam-1-3/)

  #### Lời giải cho Mock Exam 1

  1. Apply manifest dưới đây:

     <details>
     
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       labels:
         run: nginx-pod
       name: nginx-pod
     spec:
       containers:
       - image: nginx:alpine
         name: nginx-pod
         resources: {}
       dnsPolicy: ClusterFirst
       restartPolicy: Always
     status: {}
     ```

     > 💡 **Bổ sung:** Manifest trên chính là output của `kubectl run nginx-pod --image=nginx:alpine --dry-run=client -o yaml`. Trong lúc thi, chỉ cần chạy `kubectl run nginx-pod --image=nginx:alpine` là xong, không cần viết YAML.
     </details>

  2. Chạy lệnh dưới đây để tạo một Pod có gắn label:

     <details>
     
     ```
     kubectl run messaging --image=redis:alpine --labels=tier=msg
     ```

     > 💡 **Bổ sung:** Nhiều label thì phân tách bằng dấu phẩy, ví dụ `--labels=tier=msg,env=prod`. Kiểm tra lại bằng `kubectl get pod messaging --show-labels`.
     </details>

 
  3. Chạy lệnh dưới đây để tạo namespace:
     
     <details>

     ```
     kubectl create namespace apx-x9984574
     ```
     </details>

  4. Dùng lệnh dưới đây để chuyển hướng (redirect) output ra file:

     <details>

     ```
     kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
     ```

     > 💡 **Bổ sung:** Với các câu yêu cầu ghi output ra file, hãy kiểm tra đúng đường dẫn và định dạng (`-o json`, `-o yaml`, `-o wide`...) như đề yêu cầu, rồi `cat` file để chắc chắn nội dung không rỗng.
     </details>

  5. Thực thi lệnh dưới đây để expose Pod trên port 6379:

     <details>

     ```
     kubectl expose pod messaging --port=6379 --name messaging-service
     ```

     > 💡 **Bổ sung:** `kubectl expose` mặc định tạo Service kiểu `ClusterIP` và tự động lấy label của Pod (`tier=msg`) làm selector – đó là lý do nên dùng `expose` thay vì `kubectl create service` (lệnh này đặt selector là `app=<tên service>`, dễ bị sai). Xác nhận Service đã có endpoint bằng `kubectl get endpoints messaging-service`.
     </details>

  6. Apply manifest dưới đây:

     <details>

      ```
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        creationTimestamp: null
        labels:
          app: hr-web-app
        name: hr-web-app
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: hr-web-app
        strategy: {}
        template:
          metadata:
            creationTimestamp: null
            labels:
              app: hr-web-app
          spec:
            containers:
            - image: kodekloud/webapp-color
              name: webapp-color
              resources: {}
      status: {}
      ```
      
      Từ v1.19, chúng ta có thể thêm flag `--replicas` vào lệnh `kubectl create deployment`:
      ```
      kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2
      ```
     </details>

  7. Để tạo một static Pod, hãy copy manifest vào thư mục chứa static Pod. Trong trường hợp này là `/etc/kubernetes/manifests`. Apply manifest dưới đây:

     <details>

     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       labels:
         run: static-busybox
       name: static-busybox
     spec:
       containers:
       - command:
         - sleep
         - "1000"
         image: busybox
         name: static-busybox
         resources: {}
       dnsPolicy: ClusterFirst
       restartPolicy: Always
     status: {}
     ```

     > 💡 **Bổ sung:** Static Pod không được tạo bằng `kubectl apply` mà do kubelet tự đọc file trong thư mục `staticPodPath` (xem trong `/var/lib/kubelet/config.yaml`, mặc định với kubeadm là `/etc/kubernetes/manifests`). Cách nhanh nhất: `kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml`. Chú ý `--command --` phải đứng **cuối** lệnh. Sau đó Pod sẽ xuất hiện với tên có hậu tố tên node, ví dụ `static-busybox-controlplane`.
     </details>

  8. Chạy lệnh dưới đây để tạo Pod trong namespace `finance`:

     <details>

     ```
     kubectl run temp-bus --image=redis:alpine -n finance
     ```
     </details>

  9. Chạy lệnh dưới đây và troubleshoot từng bước:

     <details>

     ```
     kubectl describe pod orange
     ```

     Export Pod đang chạy bằng lệnh dưới đây và sửa lỗi chính tả trong command **`sleeeep`** thành **`sleep`**

     ```
     kubectl get pod orange -o yaml > orange.yaml
     ```
   
     Xóa Pod orange đang chạy và tạo lại Pod bằng lệnh sau.
     
     ```
     kubectl delete pod orange
     kubectl create -f orange.yaml
     ```

     > 💡 **Bổ sung:** Vì sao phải xóa rồi tạo lại? Trường `command` của container (kể cả init container) là bất biến (immutable) sau khi Pod được tạo, nên `kubectl edit` sẽ bị từ chối (kubectl lưu bản sửa vào một file tạm trong `/tmp`). Trong `kubectl describe`, chú ý trạng thái `Init:CrashLoopBackOff` và xem log của init container bằng `kubectl logs orange -c <tên-init-container>`. Có thể gộp hai bước xóa/tạo bằng `kubectl replace --force -f orange.yaml`.
     </details>

  10. Apply manifest dưới đây:

      <details>

      ```
      apiVersion: v1
      kind: Service
      metadata:
        creationTimestamp: null
        labels:
          app: hr-web-app
        name: hr-web-app-service
      spec:
        ports:
        - port: 8080
          protocol: TCP
          targetPort: 8080
          nodePort: 30082
        selector:
          app: hr-web-app
        type: NodePort
      status:
        loadBalancer: {}
      ```

      > 💡 **Bổ sung:** Có thể sinh khung manifest bằng `kubectl expose deployment hr-web-app --name=hr-web-app-service --type=NodePort --port=8080 --dry-run=client -o yaml > svc.yaml`, sau đó chỉ cần thêm `nodePort: 30082` (lệnh `expose` không có flag để chỉ định nodePort). Giá trị nodePort phải nằm trong dải mặc định 30000–32767.
      </details>

  11. Chạy lệnh dưới đây để chuyển hướng output ra file:

      <details>

      ``` 
      kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
      ```

      > 💡 **Bổ sung:** Để tìm đường dẫn JSONPath, chạy `kubectl get nodes -o json | less` (hoặc `kubectl explain node.status.nodeInfo`) rồi lần theo cấu trúc. `.items[*]` dùng vì `kubectl get nodes` trả về một danh sách.
      </details>

  12. Apply manifest dưới đây để tạo PV:

      <details>
     
       ```
       apiVersion: v1
       kind: PersistentVolume
       metadata:
         name: pv-analytics
       spec:
         capacity:
           storage: 100Mi
         volumeMode: Filesystem
         accessModes:
           - ReadWriteMany
         hostPath:
             path: /pv/data-analytics
       ```

       > 💡 **Bổ sung:** Không có lệnh imperative nào để tạo PV, nên hãy copy mẫu từ tài liệu (tìm "PersistentVolume" trên kubernetes.io/docs) hoặc dùng `kubectl explain pv.spec`. `hostPath` chỉ phù hợp cho lab/cluster một node.
       </details>
