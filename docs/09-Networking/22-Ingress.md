# Ingress

  - Xem [Bài giảng](https://kodekloud.com/topic/ingress/)

Trong phần này, chúng ta sẽ tìm hiểu về **Ingress**, gồm hai thành phần:

- Ingress Controller
- Ingress Resources

> 💡 **Bổ sung:** Ingress cho phép đưa nhiều Service ra bên ngoài qua **một điểm truy cập duy nhất** (một IP/LoadBalancer), định tuyến HTTP/HTTPS theo host (tên miền) và path (đường dẫn), đồng thời có thể xử lý TLS termination (gắn certificate qua `spec.tls` và một Secret kiểu `kubernetes.io/tls`).
> - **Ingress Controller**: phần mềm thực sự xử lý traffic (ví dụ ingress-nginx, Traefik, HAProxy, Contour...). Kubernetes **không** cài sẵn Ingress Controller; nếu không có controller, tạo Ingress resource sẽ không có tác dụng gì.
> - **Ingress Resource**: object Kubernetes (`kind: Ingress`) chứa các rule định tuyến, được controller đọc và chuyển thành cấu hình (ví dụ cấu hình nginx).

## Ingress Controller

- Triển khai **Ingress Controller** (trong ví dụ này là NGINX Ingress Controller). Một bộ triển khai tối thiểu gồm: ConfigMap (lưu cấu hình nginx), Deployment (chạy controller), Service (đưa controller ra ngoài) và ServiceAccount (kèm quyền RBAC).

## ConfigMap

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```

## Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

> 💡 **Bổ sung:** Image `quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0` trong bài giảng đã rất cũ. Hiện nay controller ingress-nginx dùng image `registry.k8s.io/ingress-nginx/controller:<version>` và thường được cài bằng manifest chính thức hoặc Helm chart thay vì tự viết từng object. Lưu ý thêm: dự án ingress-nginx (do cộng đồng Kubernetes duy trì) đã được thông báo ngừng phát triển (retirement) vào tháng 11/2025, chỉ bảo trì ở mức tối thiểu đến tháng 3/2026; cộng đồng khuyến nghị chuyển dần sang **Gateway API** (xem phần bổ sung cuối trang). Tuy vậy, kiến thức Ingress vẫn nằm trong curriculum CKA.

## ServiceAccount

- ServiceAccount là cần thiết cho việc xác thực, đi kèm với các Role, ClusterRole và RoleBinding phù hợp (controller cần quyền đọc Ingress, Service, EndpointSlice, Secret... qua API).

- Tạo ServiceAccount cho ingress
```
$ kubectl create -f ingress-sa.yaml
serviceaccount/ingress-serviceaccount created
```

## Service loại NodePort

```
# service-Nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

> 💡 **Bổ sung:** Trên cloud, Service này thường có `type: LoadBalancer` để nhận IP public; trong lab/bare-metal thì dùng `NodePort` như trên, khi đó truy cập qua `http://<node-ip>:<nodePort>`.

- Tạo Service
```
$ kubectl create -f service-Nodeport.yaml
```
- Xem Service

```
$ kubectl get service
```

## Ingress Resources

```
Ingress-wear.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
     backend:
        serviceName: wear-service
        servicePort: 80
```

- Tạo Ingress resource
```
$ kubectl create -f Ingress-wear.yaml
ingress.extensions/ingress-wear created
```

- Xem Ingress
```
$ kubectl get ingress
NAME           CLASS    HOSTS   ADDRESS   PORTS   AGE
ingress-wear   <none>   *                 80      18s
```

> ⚠️ **Lưu ý (API đã lỗi thời):** Các ví dụ Ingress trong bài giảng dùng `apiVersion: extensions/v1beta1` với `serviceName`/`servicePort`. API này (và `networking.k8s.io/v1beta1`) đã **bị gỡ bỏ từ Kubernetes v1.22**. Ở các phiên bản hiện tại (và trong đề thi CKA), bắt buộc dùng `networking.k8s.io/v1`. Ví dụ `ingress-wear` ở trên viết lại theo API mới:
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: ingress-wear
> spec:
>   defaultBackend:
>     service:
>       name: wear-service
>       port:
>         number: 80
> ```
>
> Những thay đổi chính: `spec.backend` → `spec.defaultBackend`; `serviceName`/`servicePort` → `service.name` và `service.port.number` (hoặc `service.port.name`); mỗi path **bắt buộc** có trường `pathType`.

## Ingress Resource - Rules (quy tắc định tuyến)

- 1 rule với 2 path.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```

> 💡 **Bổ sung:** Ví dụ `ingress-wear-watch` (1 rule, 2 path) theo `networking.k8s.io/v1`, kèm `ingressClassName` và `pathType`:
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: ingress-wear-watch
> spec:
>   ingressClassName: nginx
>   rules:
>   - http:
>       paths:
>       - path: /wear
>         pathType: Prefix
>         backend:
>           service:
>             name: wear-service
>             port:
>               number: 80
>       - path: /watch
>         pathType: Prefix
>         backend:
>           service:
>             name: watch-service
>             port:
>               number: 80
> ```
>
> **`pathType`** có 3 giá trị:
> - `Exact`: khớp chính xác đường dẫn (phân biệt hoa thường), ví dụ `/wear` không khớp `/wear/`.
> - `Prefix`: khớp theo tiền tố, tách theo từng phần tử phân cách bởi `/`. Ví dụ `/wear` khớp `/wear`, `/wear/`, `/wear/shirts` nhưng **không** khớp `/wearable`.
> - `ImplementationSpecific`: cách khớp do Ingress Controller (IngressClass) tự quyết định.
>
> Khi nhiều path cùng khớp, path dài nhất được ưu tiên; nếu dài bằng nhau thì `Exact` được ưu tiên hơn `Prefix`.
- Describe Ingress resource vừa tạo

```
$ kubectl describe ingress ingress-wear-watch
Name:             ingress-wear-watch
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /wear    wear-service:80 (<none>)
              /watch   watch-service:80 (<none>)
Annotations:  <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  23s   nginx-ingress-controller  Ingress default/ingress-wear-watch

```

> 💡 **Bổ sung:** Dòng `Default backend: default-http-backend:80` cho biết request không khớp rule nào sẽ được chuyển tới default backend (thường trả về lỗi `404`). Bạn có thể tự cấu hình default backend bằng `spec.defaultBackend` trong Ingress, hoặc qua flag `--default-backend-service` của ingress-nginx controller.

- 2 rule, mỗi rule 1 path (định tuyến theo host).
```
# Ingress-wear-watch.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```

> 💡 **Bổ sung:** Ví dụ định tuyến theo host với `networking.k8s.io/v1`:
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: ingress-wear-watch
> spec:
>   ingressClassName: nginx
>   rules:
>   - host: wear.my-online-store.com
>     http:
>       paths:
>       - path: /
>         pathType: Prefix
>         backend:
>           service:
>             name: wear-service
>             port:
>               number: 80
>   - host: watch.my-online-store.com
>     http:
>       paths:
>       - path: /
>         pathType: Prefix
>         backend:
>           service:
>             name: watch-service
>             port:
>               number: 80
> ```
>
> Nếu rule không có `host`, rule áp dụng cho mọi host (`*`). Có thể dùng wildcard dạng `*.my-online-store.com` (chỉ khớp đúng một cấp subdomain).

## Bổ sung: IngressClass

> 💡 **Bổ sung:** Một cluster có thể chạy nhiều Ingress Controller. Object **IngressClass** (`networking.k8s.io/v1`) xác định controller nào xử lý Ingress nào. Ingress chọn class qua trường `spec.ingressClassName` (annotation cũ `kubernetes.io/ingress.class` đã deprecated).
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: IngressClass
> metadata:
>   name: nginx
>   annotations:
>     ingressclass.kubernetes.io/is-default-class: "true"
> spec:
>   controller: k8s.io/ingress-nginx
> ```
>
> - IngressClass có annotation `ingressclass.kubernetes.io/is-default-class: "true"` sẽ được gán tự động cho các Ingress mới không khai báo `ingressClassName`.
> - Nếu không có default class và Ingress không chỉ định class, có thể không controller nào nhận Ingress đó (cột `CLASS` hiện `<none>`, `ADDRESS` trống), trừ khi controller được cấu hình để nhận Ingress không có class (ví dụ flag `--watch-ingress-without-class=true` của ingress-nginx).
> - Xem các class hiện có: `kubectl get ingressclass`.

## Bổ sung: Tạo Ingress bằng lệnh imperative (rất hữu ích khi thi)

> 💡 **Bổ sung:** Thay vì viết YAML từ đầu, dùng `kubectl create ingress` để tạo nhanh, hoặc thêm `--dry-run=client -o yaml` để sinh file rồi chỉnh sửa:
>
> ```bash
> # 1 rule, 2 path (không có host); dấu * ở cuối path => pathType: Prefix, không có * => Exact
> kubectl create ingress ingress-wear-watch --class=nginx \
>   --rule="/wear*=wear-service:80" \
>   --rule="/watch*=watch-service:80"
>
> # Định tuyến theo host
> kubectl create ingress ingress-hosts --class=nginx \
>   --rule="wear.my-online-store.com/*=wear-service:80" \
>   --rule="watch.my-online-store.com/*=watch-service:80"
>
> # Kèm annotation và TLS (Secret my-tls-secret phải tồn tại)
> kubectl create ingress secure-ingress --class=nginx \
>   --rule="secure.my-online-store.com/*=wear-service:80,tls=my-tls-secret" \
>   --annotation nginx.ingress.kubernetes.io/rewrite-target=/
> ```
>
> Xem thêm ví dụ: `kubectl create ingress --help`.

## Bổ sung: Gateway API (có trong curriculum CKA mới)

> 💡 **Bổ sung:** Từ curriculum CKA 2025, **Gateway API** được đưa vào phần Services & Networking ("Use the Gateway API to manage Ingress traffic"). Gateway API là thế hệ kế nhiệm của Ingress (GA từ v1.0, tháng 10/2023), được cài dưới dạng CRD (không có sẵn trong Kubernetes core), dùng nhóm API `gateway.networking.k8s.io/v1`. Nó tách vai trò rõ ràng:
> - **GatewayClass** (do nhà cung cấp hạ tầng định nghĩa): loại controller sẽ hiện thực Gateway, tương tự IngressClass.
> - **Gateway** (do người vận hành cluster quản lý): một điểm vào traffic cụ thể, khai báo các `listeners` (port, protocol, hostname, TLS).
> - **HTTPRoute** (do đội phát triển ứng dụng quản lý): quy tắc định tuyến HTTP (theo host, path, header, query param...) tới các Service; còn có `GRPCRoute`, `TLSRoute`, `TCPRoute`...
>
> Ví dụ tương đương với `ingress-wear-watch`:
>
> ```yaml
> apiVersion: gateway.networking.k8s.io/v1
> kind: Gateway
> metadata:
>   name: my-gateway
> spec:
>   gatewayClassName: nginx
>   listeners:
>   - name: http
>     protocol: HTTP
>     port: 80
> ---
> apiVersion: gateway.networking.k8s.io/v1
> kind: HTTPRoute
> metadata:
>   name: wear-watch-route
> spec:
>   parentRefs:
>   - name: my-gateway
>   rules:
>   - matches:
>     - path:
>         type: PathPrefix
>         value: /wear
>     backendRefs:
>     - name: wear-service
>       port: 80
>   - matches:
>     - path:
>         type: PathPrefix
>         value: /watch
>     backendRefs:
>     - name: watch-service
>       port: 80
> ```
>
> Ưu điểm so với Ingress: các tính năng như rewrite URL, redirect, chia traffic theo trọng số (`weight` trong `backendRefs`, phục vụ canary), thêm/sửa header... được chuẩn hóa ngay trong spec qua `filters` thay vì phụ thuộc vào annotation riêng của từng controller. Kiểm tra nhanh: `kubectl get gatewayclass,gateway,httproute -A`.






#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- https://thenewstack.io/kubernetes-ingress-for-beginners/