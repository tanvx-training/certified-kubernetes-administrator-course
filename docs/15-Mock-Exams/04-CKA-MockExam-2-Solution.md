# Lời giải Mock Exam 2
  
  
  1. Chạy lệnh dưới đây để giải:

     <details>

     ```
     ETCDCTL_API=3 etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
     ```

     > 💡 **Bổ sung:** Đừng học thuộc đường dẫn certificate – hãy lấy từ manifest của static Pod etcd: `grep -E 'cert|key|listen-client' /etc/kubernetes/manifests/etcd.yaml` (các flag `--trusted-ca-file`, `--cert-file`, `--key-file`, `--listen-client-urls`). Từ etcd v3.4, `ETCDCTL_API=3` là mặc định nên có thể bỏ qua, nhưng ghi rõ cũng không sao. Sau khi backup, kiểm tra file bằng `etcdutl snapshot status /opt/etcd-backup.db` (hoặc `etcdctl snapshot status`, đã deprecated từ etcd v3.5).
     </details>

  2. Chạy lệnh dưới đây để giải:

     <details>
 
     ```
     apiVersion: v1
     kind: Pod
     metadata:
        creationTimestamp: null
        labels:
          run: redis-storage
        name: redis-storage
     spec:
      volumes:
      - name: redis-storage
        emptyDir: {}
      
      containers:
      - image: redis:alpine
        name: redis-storage
        resources: {}
        volumeMounts:
        - name: redis-storage
          mountPath: /data/redis
      dnsPolicy: ClusterFirst
      restartPolicy: Always
     status: {}
     ```

     > 💡 **Bổ sung:** Sinh khung bằng `kubectl run redis-storage --image=redis:alpine --dry-run=client -o yaml > redis.yaml`, rồi thêm `volumes` (ở cấp `spec`) và `volumeMounts` (ở cấp container). Volume `emptyDir` tồn tại theo vòng đời của Pod: Pod bị xóa thì dữ liệu cũng mất.
     </details>
 
  3. Chạy lệnh dưới đây để giải:

     <details>

     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       name: super-user-pod
     spec:
       containers:
       - image: busybox:1.28
         name: super-user-pod
         command: ["sleep", "4800"]
         securityContext:
           capabilities:
             add: ["SYS_TIME"]
     ```

     > 💡 **Bổ sung:** `capabilities` chỉ khai báo được trong `securityContext` ở cấp **container**, không có ở cấp Pod. Tra nhanh bằng `kubectl explain pod.spec.containers.securityContext.capabilities`.
     </details>

  4. Chạy lệnh dưới đây để giải:

     <details>
     
     ```
     apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: my-pvc
     spec:
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 10Mi      
     ```
    
     ```
     apiVersion: v1
     kind: Pod
     metadata:
       creationTimestamp: null
       labels:
         run: use-pv
       name: use-pv
     spec:
       containers:
       - image: nginx
         name: use-pv
         volumeMounts:
         - mountPath: "/data"
           name: mypod
       volumes:
       - name: mypod
         persistentVolumeClaim:
           claimName: my-pvc
     ```

     > 💡 **Bổ sung:** Trước khi tạo PVC, chạy `kubectl get pv` để xem PV có sẵn: PVC chỉ bind được khi `accessModes`, `storageClassName` khớp và dung lượng PV ≥ dung lượng yêu cầu. Sau khi tạo, kiểm tra `kubectl get pvc my-pvc` phải ở trạng thái `Bound`; nếu `Pending` thì `kubectl describe pvc my-pvc` để xem lý do.
     </details>

  5. Chạy lệnh dưới đây để giải:

     <details>
 
     Với Kubernetes phiên bản <=1.17
 
     ```
     kubectl run nginx-deploy --image=nginx:1.16 --replicas=1 --record
     kubectl rollout history deployment nginx-deploy
     kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
     kubectl rollout history deployment nginx-deploy
     ```
 
     Với Kubernetes phiên bản >1.17
 
     ```
     kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml
   
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deploy
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: nginx-deploy
       strategy: {}
       template:
         metadata:
           creationTimestamp: null
           labels:
             app: nginx-deploy
         spec:
           containers:
           - image: nginx:1.16
             name: nginx
     ```
     
     ```
     kubectl create -f deploy.yaml --record
     kubectl rollout history deployment nginx-deploy
     kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
     kubectl rollout history deployment nginx-deploy
     ```

     > 💡 **Bổ sung (cập nhật cho phiên bản mới):**
     > - Từ v1.18, `kubectl run` chỉ tạo Pod; các flag như `--replicas` không còn tác dụng với `kubectl run`, vì vậy cách thứ nhất chỉ dùng được trên cluster rất cũ.
     > - Flag `--record` đã bị deprecated. Cách được khuyến nghị để ghi lại lý do thay đổi (cột `CHANGE-CAUSE` trong `rollout history`) là gán annotation: `kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="update image to nginx:1.17"`.
     > - Tên container trong `set image` (`nginx=...`) phải khớp tên container trong Deployment. `kubectl create deployment` tự đặt tên container theo tên image (ở đây là `nginx`); kiểm tra bằng `kubectl get deploy nginx-deploy -o jsonpath='{.spec.template.spec.containers[*].name}'`.
     > - Theo dõi tiến trình bằng `kubectl rollout status deployment nginx-deploy`.
     </details>
  
  6. Chạy lệnh dưới đây để giải:

     <details>
 
     ```
      apiVersion: certificates.k8s.io/v1
      kind: CertificateSigningRequest
      metadata:
        name: john-developer
      spec:
        signerName: kubernetes.io/kube-apiserver-client
        request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
        usages:
        - digital signature
        - key encipherment
        - client auth
        groups:
        - system:authenticated
       ```
 
      ```
      kubectl certificate approve john-developer
      kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development
      kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
      kubectl auth can-i update pods --as=john --namespace=development
      ```

      > 💡 **Bổ sung:** Giá trị `request` là nội dung file CSR (ví dụ `/root/CKA/john.csr`) được mã hóa base64 trên **một dòng**: `cat /root/CKA/john.csr | base64 | tr -d "\n"`. Với `signerName: kubernetes.io/kube-apiserver-client`, `usages` phải có `client auth`. Sau khi approve, kiểm tra `kubectl get csr john-developer` có trạng thái `Approved,Issued`. Lệnh `kubectl auth can-i ... --as=john` trả về `yes` nghĩa là RBAC đã cấu hình đúng.
     </details>
 
  7. Chạy lệnh dưới đây để giải:

     <details>
 
     ```
     kubectl run nginx-resolver --image=nginx
     kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP
     kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
     kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
 
     Lấy IP của Pod nginx-resolver và thay dấu chấm (.) bằng dấu gạch ngang (-), giá trị này sẽ được dùng ở lệnh bên dưới.
 
     kubectl get pod nginx-resolver -o wide
     kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod
 
     ```

     > 💡 **Bổ sung:** Bản ghi DNS của Pod có dạng `<pod-ip-dùng-dấu-gạch>.<namespace>.pod.cluster.local`, ví dụ Pod IP `10.244.1.5` trong namespace `default` → `10-244-1-5.default.pod.cluster.local`. Còn Service có dạng `<service>.<namespace>.svc.cluster.local`. Dùng image `busybox:1.28` vì `nslookup` trong các bản busybox mới hơn hay cho kết quả lỗi/khó đọc.
 
     </details>

  8. Chạy lệnh dưới đây để giải:

     <details>
 
     ```
     kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml
     
     cat static.yaml - Copy nội dung của file này.
 
     kubectl get nodes -o wide
     ssh node01 
     OR
     ssh <IP of node01>
 
     Kiểm tra thư mục static Pod /etc/kubernetes/manifests đã tồn tại chưa, nếu chưa thì tạo mới.
     mkdir -p /etc/kubernetes/manifests
 
     Dán nội dung file (static.yaml) đã copy ở bước đầu tiên vào file nginx-critical.yaml.
 
     Di chuyển/copy file nginx-critical.yaml vào đường dẫn /etc/kubernetes/manifests/
 
     cp nginx-critical.yaml /etc/kubernetes/manifests
 
     Quay lại master node
 
     kubectl get pods 
     ```

     > 💡 **Bổ sung:** Thư mục static Pod trên node01 không nhất thiết là `/etc/kubernetes/manifests` – hãy kiểm tra `staticPodPath` trong file cấu hình kubelet (`grep staticPodPath /var/lib/kubelet/config.yaml`, hoặc xem flag `--config` qua `ps aux | grep kubelet`). Nếu phải tạo hoặc đổi thư mục này, cần khởi động lại kubelet (`systemctl restart kubelet`). Pod sẽ hiện ra với tên `nginx-critical-node01`. Nhớ `exit` khỏi node01 trước khi làm câu tiếp theo.
 
     </details>

  
