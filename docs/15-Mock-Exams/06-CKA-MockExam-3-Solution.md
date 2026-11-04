# Lời giải Mock Exam 3


1. Chạy lệnh dưới đây để giải: 

     <details>

     ```
     kubectl create serviceaccount pvviewer
     kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
     kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
     ```

     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       labels:
         run: pvviewer
       name: pvviewer
     spec:
       containers:
       - image: redis
         name: pvviewer
         resources: {}
       serviceAccountName: pvviewer
     ```

     > 💡 **Bổ sung:** PersistentVolume là resource cấp cluster (không thuộc namespace), nên phải dùng ClusterRole + ClusterRoleBinding chứ không phải Role/RoleBinding. Cú pháp `--serviceaccount=<namespace>:<tên>`. Kiểm tra: `kubectl auth can-i list persistentvolumes --as=system:serviceaccount:default:pvviewer`. Trường `serviceAccountName` của Pod không sửa được sau khi tạo, nên nếu quên thì phải tạo lại Pod.
     </details>

2. Chạy lệnh dưới đây để giải: 

     <details>
 
     ```
     kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/CKA/node_ips
     ```

     > 💡 **Bổ sung:** `[?(@.type=="InternalIP")]` là biểu thức lọc (filter) của JSONPath: chỉ lấy phần tử trong mảng `addresses` có `type` bằng `InternalIP`. Bạn có thể tìm ví dụ tương tự trong trang "kubectl Quick Reference" (cheat sheet) trên kubernetes.io/docs.
     </details>
 
3. Chạy lệnh dưới đây để giải:  
 
     <details>
 
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       name: multi-pod
     spec:
       containers:
       - image: nginx
         name: alpha
         env:
         - name: name
           value: alpha
       - image: busybox
         name: beta
         command: ["sleep", "4800"]
         env:
         - name: name
           value: beta
     status: {}
     ```

     > 💡 **Bổ sung:** Container `busybox` không có tiến trình chạy lâu dài nên cần `command: ["sleep", "4800"]`, nếu không nó sẽ kết thúc ngay và rơi vào `CrashLoopBackOff`. Có thể sinh khung bằng `kubectl run multi-pod --image=nginx --env=name=alpha --dry-run=client -o yaml` rồi thêm container thứ hai.
     </details>
 
4. Chạy lệnh dưới đây để giải:
 
     <details>
     
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       name: non-root-pod
     spec:
       securityContext:
         runAsUser: 1000
         fsGroup: 2000
       containers:
       - name: non-root-pod
         image: redis:alpine
     ```

     > 💡 **Bổ sung:** `securityContext` ở cấp Pod áp dụng cho mọi container trong Pod; nếu container cũng khai báo `securityContext` thì giá trị ở cấp container sẽ ghi đè. `fsGroup` chỉ có ở cấp Pod.
     </details>
 
5. Chạy lệnh dưới đây để giải:  
 
     <details>
 
     ```
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       name: ingress-to-nptest
       namespace: default
     spec:
       podSelector:
         matchLabels:
           run: np-test-1
       policyTypes:
       - Ingress
       ingress:
       - ports:
         - protocol: TCP
           port: 80
     ```

     > 💡 **Bổ sung:** Rule `ingress` chỉ có `ports` mà không có `from` nghĩa là cho phép traffic từ **mọi nguồn** vào port 80/TCP của các Pod có label `run: np-test-1`; các port khác bị chặn. Trước tiên hãy xác định label thật của Pod bằng `kubectl get pod --show-labels`. Kiểm thử: `kubectl run test-np --image=busybox:1.28 --rm -it --restart=Never -- wget -qO- -T 2 <service-hoặc-pod-ip>:80`. Lưu ý Network Policy chỉ có hiệu lực khi CNI hỗ trợ (Calico, Cilium, Weave...; Flannel thì không).
     </details>
   
6. Chạy lệnh dưới đây để giải: 
 
     <details>
 
     ```
     kubectl taint node node01 env_type=production:NoSchedule
     ```

     Deploy Pod `dev-redis` và đảm bảo rằng workload không được schedule lên worker node `node01` này.
     ```
     kubectl run dev-redis --image=redis:alpine

     kubectl get pods -owide
     ```

     Deploy Pod mới `prod-redis` có toleration để được schedule lên worker node `node01`.
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       name: prod-redis
     spec:
       containers:
       - name: prod-redis
         image: redis:alpine
       tolerations:
       - effect: NoSchedule
         key: env_type
         operator: Equal
         value: production     
     ```

     Xem các Pod kèm thông tin ngắn gọn:
     ```
     kubectl get pods -owide | grep prod-redis
     ```

     > 💡 **Bổ sung:** Toleration chỉ **cho phép** Pod được schedule lên node có taint tương ứng, chứ không **bắt buộc** Pod phải chạy trên node đó. Nếu cần ép Pod chạy trên `node01`, phải kết hợp thêm `nodeSelector`/node affinity. Trong lab này chỉ có `node01` là worker nên `prod-redis` sẽ chạy trên đó (control plane có taint riêng). Gỡ taint bằng cách thêm dấu `-` ở cuối: `kubectl taint node node01 env_type=production:NoSchedule-`.
     </details>
 
7. Chạy lệnh dưới đây để giải: 
 
     <details>
 
     ```
     kubectl create namespace hr
     kubectl run hr-pod --image=redis:alpine --namespace=hr --labels=environment=production,tier=frontend
     ```
     </details>

8. Chạy lệnh dưới đây để giải:

     <details>

     ```
     vi /root/CKA/super.kubeconfig

     Đổi port 2379 thành 6443 và chạy lệnh dưới đây để kiểm tra
     
     kubectl cluster-info --kubeconfig=/root/CKA/super.kubeconfig     
     ```

     > 💡 **Bổ sung:** Vì sao là 6443? Trường `server` trong kubeconfig phải trỏ tới kube-apiserver, mặc định lắng nghe ở port `6443`; còn `2379` là port client của etcd. Đối chiếu nhanh với kubeconfig chuẩn: `grep server ~/.kube/config`. Ngoài sai port, các lỗi kubeconfig hay gặp trong đề là sai đường dẫn/nội dung certificate hoặc sai tên user/cluster trong context.
     </details>

9. Chạy lệnh dưới đây để giải:
   
     <details>
     
     ```
     sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' kube-controller-manager.yaml
     ```

     > 💡 **Bổ sung:** Lệnh trên chạy trong thư mục `/etc/kubernetes/manifests` trên control plane (kube-controller-manager là static Pod). Lỗi nằm ở `contro1ler` – chữ `l` bị thay bằng số `1`. Dấu hiệu nhận biết: Deployment đã scale nhưng số replica không tăng, vì chính kube-controller-manager (cụ thể là ReplicaSet controller) chịu trách nhiệm tạo Pod mới. Kiểm tra `kubectl get pods -n kube-system` và xem lỗi bằng `kubectl logs -n kube-system <pod-controller-manager>` hoặc `kubectl describe`. Nếu API server cũng không phản hồi, xem log container bằng `crictl ps -a` và `crictl logs <container-id>`. Sau khi sửa, kubelet tự tạo lại static Pod; đợi một lúc rồi kiểm tra Deployment đã đủ replica.
     </details>



