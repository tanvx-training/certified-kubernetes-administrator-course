# Services và Networking (13%)

### Tạo một Pod tên nginx dùng image nginx và expose port 80 của nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# để ý rằng cả Pod lẫn Service đều được tạo ra
```

> 💡 **Bổ sung:** Flag `--expose` (đi kèm `--port`) khiến `kubectl run` tạo thêm một Service kiểu ClusterIP cùng tên với Pod, có selector `run=nginx`. Đây là cách nhanh nhất để có cặp Pod + Service khi thi. Nếu Pod đã tồn tại, dùng `kubectl expose pod nginx --port=80` cũng cho kết quả tương tự.

</p>
</details>


### Xác nhận Service ClusterIP đã được tạo. Kiểm tra thêm các endpoint

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get svc nginx # services
kubectl get ep # endpoints
```

> 💡 **Bổ sung:** Từ Kubernetes v1.33, API `Endpoints` (v1) đã bị đánh dấu deprecated và được thay bằng `EndpointSlice` (`discovery.k8s.io/v1`). `kubectl get ep` vẫn chạy (kèm cảnh báo), nhưng nên làm quen với `kubectl get endpointslices -l kubernetes.io/service-name=nginx`. Nếu Service không có endpoint nào, gần như chắc chắn selector của Service không khớp label của Pod. Đây là lỗi kinh điển trong các bài troubleshooting của cả CKA lẫn CKAD.

</p>
</details>

### Lấy ClusterIP của Service, tạo một Pod busybox tạm thời rồi dùng wget gọi tới IP đó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get svc nginx # lấy IP (dạng như 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never --
wget -O- [PUT THE POD'S IP ADDRESS HERE]:80
exit
```

> Lưu ý: ở đây cần điền **ClusterIP của Service** vừa lấy ở lệnh đầu (bản gốc ghi nhầm là "POD'S IP"; gọi thẳng IP của Pod cũng chạy nhưng không phải là đang kiểm tra Service).

</p>
hoặc
<p>

```bash
IP=$(kubectl get svc nginx --template={{.spec.clusterIP}}) # lấy IP (dạng như 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80 --timeout 2
# Mẹo: --timeout là tùy chọn, nhưng giúp nhận kết quả nhanh hơn khi kết nối thất bại (vài giây thay vì vài phút)
```

> 💡 **Bổ sung:** `$IP` trong lệnh thứ hai được shell trên máy bạn thay giá trị trước khi gửi lên cluster, nên thực ra không cần `--env`. Cách tương đương với jsonpath: `kubectl get svc nginx -o jsonpath='{.spec.clusterIP}'`.

</p>
</details>

### Chuyển Service đó từ ClusterIP sang NodePort và tìm port NodePort. Gọi Service qua IP của node. Cuối cùng xóa Service và Pod.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # đổi ClusterIP thành NodePort
status:
  loadBalancer: {}
```

hoặc

```bash
kubectl patch svc nginx -p '{"spec":{"type":"NodePort"}}' 
```

```bash
kubectl get svc
```

```
# result:
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx        NodePort    10.107.253.138   <none>        80:31931/TCP   3m
```

```bash
wget -O- NODE_IP:31931 # nếu bạn dùng Kubernetes của Docker Desktop cho Windows/Mac, thử 127.0.0.1
# nếu dùng minikube, chạy minikube ip để lấy IP của node, ví dụ 192.168.99.117
```

```bash
kubectl delete svc nginx # Xóa Service
kubectl delete pod nginx # Xóa Pod
```

> 💡 **Bổ sung:**
> - YAML ở trên là output cũ: trường `selfLink` đã bị gỡ khỏi API từ v1.20 nên bạn sẽ không thấy nó trên cluster hiện tại.
> - Port NodePort mặc định được cấp trong dải `30000-32767`. Muốn chọn port cố định thì thêm `nodePort: 30080` vào phần `ports`. Lấy nhanh port được cấp: `kubectl get svc nginx -o jsonpath='{.spec.ports[0].nodePort}'`.
> - Để lấy IP của node: `kubectl get nodes -o wide` (cột `INTERNAL-IP`).

</p>
</details>

### Tạo Deployment tên foo dùng image 'dgkanatsios/simpleapp' (một server đơn giản trả về hostname) với 3 replica. Gắn label 'app=foo'. Khai báo rằng container trong Pod này nhận traffic ở port 8080 (CHƯA tạo Service)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
kubectl label deployment foo --overwrite app=foo # Bước này không bắt buộc vì kubectl create deploy foo đã mặc định gắn label app=foo
```
</p>
</details>

### Lấy IP của các Pod. Tạo một Pod busybox tạm thời và thử gọi tới các IP đó ở port 8080

<details><summary>Xem lời giải</summary>
<p>


```bash
kubectl get pods -l app=foo -o wide # 'wide' sẽ hiển thị IP của Pod
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh
wget -O- <POD_IP>:8080 # đừng dùng tên Pod, sẽ không chạy được
# thử gọi tất cả các IP từ lệnh đầu tiên để thấy hostname trả về khác nhau
exit
# hoặc
kubectl get po -o wide -l app=foo | awk '{print $6}' | grep -v IP | xargs -L1 -I '{}' kubectl run --rm -ti tmp --restart=Never --image=busybox -- wget -O- http://\{\}:8080
# hoặc
kubectl get po -l app=foo -o jsonpath='{range .items[*]}{.status.podIP}{"\n"}{end}' | xargs -L1 -I '{}' kubectl run --rm -ti tmp --restart=Never --image=busybox -- wget -O- http://\{\}:8080
```

> 💡 **Bổ sung:** Tên Pod không phân giải được qua DNS vì CoreDNS chỉ tạo bản ghi cho Service (và cho Pod theo dạng `<ip-gạch-ngang>.<namespace>.pod.cluster.local`, ví dụ `10-244-1-5.default.pod.cluster.local`). Pod chỉ có bản ghi DNS theo tên khi nằm sau một headless Service và đặt `hostname`/`subdomain`, như trường hợp StatefulSet.

</p>
</details>

### Tạo Service expose Deployment ở port 6262. Kiểm tra Service đã tồn tại và xem các endpoint

<details><summary>Xem lời giải</summary>
<p>


```bash
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo # sẽ thấy ClusterIP cùng port 6262
kubectl get endpoints foo # sẽ thấy IP của 3 Pod replica, lắng nghe ở port 8080
```

> 💡 **Bổ sung:** `--port` là port của Service (client gọi vào), còn `--target-port` là port của container mà traffic được chuyển tới. Nếu bỏ `--target-port`, giá trị mặc định bằng `--port`, và trong bài này Service sẽ không chạy vì container lắng nghe ở 8080.

</p>
</details>

### Tạo một Pod busybox tạm thời và dùng wget kết nối tới Service foo. Kiểm tra rằng mỗi lần gọi trả về một hostname khác nhau. Xóa Deployment và Service để dọn dẹp cluster

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get svc # lấy ClusterIP của Service foo
kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
wget -O- foo:6262 # DNS hoạt động! chạy nhiều lần, bạn sẽ thấy các Pod khác nhau trả lời
wget -O- <SERVICE_CLUSTER_IP>:6262 # gọi bằng ClusterIP cũng được
# bạn cũng có thể chạy kubectl logs trên các Pod của Deployment để xem log của container
kubectl delete svc foo
kubectl delete deploy foo
```

> 💡 **Bổ sung:** Tên ngắn `foo` chỉ phân giải được khi Pod gọi nằm cùng namespace với Service. Từ namespace khác thì dùng `foo.<namespace>` hoặc FQDN `foo.<namespace>.svc.cluster.local`. Đề thi rất hay hỏi kiểu này.

</p>
</details>

### Tạo Deployment nginx với 2 replica, expose qua Service ClusterIP ở port 80. Tạo và apply một NetworkPolicy để chỉ những Pod có label 'access: granted' mới truy cập được các Pod của Deployment này

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

> Lưu ý: tùy cách triển khai Kubernetes, network policy có thể không được thực thi mặc định. Ví dụ Azure AKS mặc định không bật tính năng này; cluster phải được tạo với hỗ trợ `netpol` một cách tường minh: https://docs.microsoft.com/en-us/azure/aks/use-network-policies#overview-of-network-policy  
  
<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80

kubectl describe svc nginx # xem selector 'app=nginx' dùng để chọn Pod
# hoặc
kubectl get svc nginx -o yaml

vi policy.yaml
```

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # đặt tên tùy ý
spec:
  podSelector:
    matchLabels:
      app: nginx # selector chọn các Pod được bảo vệ
  ingress: # cho phép traffic ingress
  - from:
    - podSelector: # từ các Pod
        matchLabels: # có label này
          access: granted
```

```bash
# Tạo NetworkPolicy
kubectl create -f policy.yaml

# Kiểm tra NetworkPolicy đã được tạo đúng chưa
# đảm bảo network provider (CNI) của cluster hỗ trợ Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # Lệnh này sẽ thất bại. --timeout là tùy chọn, nhưng giúp nhận kết quả nhanh hơn (vài giây thay vì vài phút)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # Lệnh này sẽ thành công
```

> 💡 **Bổ sung:**
> - NetworkPolicy có trong cả curriculum CKA 2025 lẫn CKAD. Policy chỉ có hiệu lực khi CNI hỗ trợ (Calico, Cilium, Weave...). Flannel thuần không thực thi NetworkPolicy.
> - Khi một Pod đã bị một policy có `policyTypes: [Ingress]` chọn, mọi traffic ingress không khớp rule nào đều bị chặn. Nếu bỏ trống `policyTypes`, Kubernetes tự suy ra `Ingress` (và thêm `Egress` nếu có mục `egress`).
> - Lỗi hay gặp khi thi là thụt lề sai trong `from`. Một phần tử list chứa cả `namespaceSelector` và `podSelector` nghĩa là **AND**; hai phần tử list riêng biệt (mỗi phần tử bắt đầu bằng `-`) nghĩa là **OR**.
> - Policy không chặn traffic **ra** từ Pod busybox, nên lệnh thứ hai vẫn phân giải DNS `nginx` bình thường. Nếu bài yêu cầu thêm policy egress, nhớ mở port 53 UDP/TCP cho DNS.
> - Xem policy đang chọn Pod nào: `kubectl describe networkpolicy access-nginx`.

</p>
</details>

### Tạo Ingress resource để expose một Service có sẵn bằng định tuyến theo path HTTP
Bạn đã có sẵn:
- một Deployment nginx
- một Service nginx expose port 80

Bạn muốn expose nó ra bên ngoài bằng một Ingress rule.
> Lưu ý: trong kỳ thi CKAD, bạn không phải cài Ingress Controller, nhưng phải biết cách viết và troubleshooting một Ingress resource, cũng như hiểu cách nó kết nối với Service.

<details><summary>Xem lời giải</summary> <p>
Kiểm tra Service đã tồn tại:
  
```bash
kubectl get svc nginx
```
Kết quả mong đợi:
- Type: ClusterIP
- Port: 80

Nếu môi trường luyện tập của bạn chưa có Deployment và Service, tạo chúng bằng các lệnh sau:
```bash
kubectl create deployment nginx --image=nginx --port=80
kubectl expose deployment nginx --port=80 --name=nginx
```

Tạo Ingress resource:
```bash
vi ingress.yaml
```
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
Apply:
```bash
kubectl apply -f ingress.yaml
```

Kiểm tra Ingress:
```bash
kubectl get ingress
kubectl describe ingress nginx-ingress
```

> 💡 **Bổ sung:**
> - Tạo nhanh bằng lệnh imperative (tiết kiệm thời gian khi thi): `kubectl create ingress nginx-ingress --rule="nginx.example.com/*=nginx:80"`. Dấu `*` ở cuối path tương ứng với `pathType: Prefix`; không có `*` thì là `Exact`.
> - Nếu cluster có nhiều Ingress Controller, cần chỉ định `spec.ingressClassName` (ví dụ `nginx`), hoặc thêm `--class=nginx` với lệnh imperative. Xem danh sách bằng `kubectl get ingressclass`.
> - Kiểm tra từ bên ngoài khi chưa có DNS: `curl -H "Host: nginx.example.com" http://<IP-của-controller>/`.
> - Troubleshooting: trong `kubectl describe ingress`, dòng `Backends` phải hiện IP của Pod; nếu báo `<error: endpoints "nginx" not found>` thì tên Service hoặc port bị sai.
> - Curriculum CKA 2025 có thêm **Gateway API** (`Gateway`, `HTTPRoute`), là thế hệ kế tiếp của Ingress, nên ôn cả hai. Lưu ý thêm: dự án ingress-nginx (Ingress Controller phổ biến nhất) đã thông báo ngừng bảo trì từ tháng 3/2026; Ingress API vẫn là một phần ổn định của Kubernetes.

</p>
</details>
