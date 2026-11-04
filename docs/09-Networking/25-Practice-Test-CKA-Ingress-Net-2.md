# Practice Test CKA Ingress 2 (Bài thực hành Ingress 2)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-cka-ingress-networking-2/)

#### Lời giải

  1. Xem lời giải

     <details>

      ```
      OK
      ```
     </details>

  1. Xem lời giải

     <details>

      ```
      kubectl create namespace ingress-nginx
      ```

      > 💡 Thông thường Ingress Controller được triển khai trong namespace riêng (`ingress-nginx`) để tách biệt với ứng dụng.
     </details>

  1. Xem lời giải

     <details>

      ```
      kubectl create configmap ingress-nginx-controller --namespace ingress-nginx
      ```
     </details>

  1. Xem lời giải

     <details>

      ```
      kubectl create serviceaccount ingress-nginx --namespace ingress-nginx
      kubectl create serviceaccount ingress-nginx-admission --namespace ingress-nginx
      ```

      > 💡 ServiceAccount `ingress-nginx` dùng cho controller; `ingress-nginx-admission` dùng cho Job tạo certificate của admission webhook (webhook này kiểm tra tính hợp lệ của Ingress trước khi lưu).
     </details>

  1. Xem lời giải

     <details>

      ```
      Ok

      kubectl get roles,rolebindings --namespace ingress-nginx
      ```
     </details>

  1. Xem lời giải

     <details>

     Sửa các lỗi

     ```
     vi /root/ingress-controller.yaml
     ```

     File này chứa một `Deployment` và một `Service`, cả hai đều có lỗi.


     1. `namespace` của Deployment bị sai.
     1. Lỗi thụt lề ở dòng 74 (dùng `:set nu` trong vi để bật hiển thị số dòng).
     1. `name` của Service bị sai.
     1. `nodeport` trong Service viết sai chữ hoa/thường, đúng phải là `nodePort`.

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        labels:
          app.kubernetes.io/component: controller
          app.kubernetes.io/instance: ingress-nginx
          app.kubernetes.io/managed-by: Helm
          app.kubernetes.io/name: ingress-nginx
          app.kubernetes.io/part-of: ingress-nginx
          app.kubernetes.io/version: 1.1.2
          helm.sh/chart: ingress-nginx-4.0.18
        name: ingress-nginx-controller
        namespace: ingress-nginx
      spec:
        minReadySeconds: 0
        revisionHistoryLimit: 10
        selector:
          matchLabels:
            app.kubernetes.io/component: controller
            app.kubernetes.io/instance: ingress-nginx
            app.kubernetes.io/name: ingress-nginx
        template:
          metadata:
            labels:
              app.kubernetes.io/component: controller
              app.kubernetes.io/instance: ingress-nginx
              app.kubernetes.io/name: ingress-nginx
          spec:
            containers:
            - args:
              - /nginx-ingress-controller
              - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
              - --election-id=ingress-controller-leader
              - --watch-ingress-without-class=true
              - --default-backend-service=app-space/default-http-backend
              - --controller-class=k8s.io/ingress-nginx
              - --ingress-class=nginx
              - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
              - --validating-webhook=:8443
              - --validating-webhook-certificate=/usr/local/certificates/cert
              - --validating-webhook-key=/usr/local/certificates/key
              env:
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: POD_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
              - name: LD_PRELOAD
                value: /usr/local/lib/libmimalloc.so
              image: registry.k8s.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
              imagePullPolicy: IfNotPresent
              lifecycle:
                preStop:
                  exec:
                    command:
                    - /wait-shutdown
              livenessProbe:
                failureThreshold: 5
                httpGet:
                  path: /healthz
                  port: 10254
                  scheme: HTTP
                initialDelaySeconds: 10
                periodSeconds: 10
                successThreshold: 1
                timeoutSeconds: 1
              name: controller
              ports:
              - name: http
                containerPort: 80
                protocol: TCP
              - containerPort: 443
                name: https
                protocol: TCP
              - containerPort: 8443
                name: webhook
                protocol: TCP
              readinessProbe:
                failureThreshold: 3
                httpGet:
                  path: /healthz
                  port: 10254
                  scheme: HTTP
                initialDelaySeconds: 10
                periodSeconds: 10
                successThreshold: 1
                timeoutSeconds: 1
              resources:
                requests:
                  cpu: 100m
                  memory: 90Mi
              securityContext:
                allowPrivilegeEscalation: true
                capabilities:
                  add:
                  - NET_BIND_SERVICE
                  drop:
                  - ALL
                runAsUser: 101
              volumeMounts:
              - mountPath: /usr/local/certificates/
                name: webhook-cert
                readOnly: true
            dnsPolicy: ClusterFirst
            nodeSelector:
              kubernetes.io/os: linux
            serviceAccountName: ingress-nginx
            terminationGracePeriodSeconds: 300
            volumes:
            - name: webhook-cert
              secret:
                secretName: ingress-nginx-admission

      ---

      apiVersion: v1
      kind: Service
      metadata:
        creationTimestamp: null
        labels:
          app.kubernetes.io/component: controller
          app.kubernetes.io/instance: ingress-nginx
          app.kubernetes.io/managed-by: Helm
          app.kubernetes.io/name: ingress-nginx
          app.kubernetes.io/part-of: ingress-nginx
          app.kubernetes.io/version: 1.1.2
          helm.sh/chart: ingress-nginx-4.0.18
        name: ingress-nginx-controller
        namespace: ingress-nginx
      spec:
        ports:
        - port: 80
          protocol: TCP
          targetPort: 80
          nodePort: 30080
        selector:
          app.kubernetes.io/component: controller
          app.kubernetes.io/instance: ingress-nginx
          app.kubernetes.io/name: ingress-nginx
        type: NodePort      
        ```
     </details>
  
  1. Xem lời giải

     <details>

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: ingress-wear-watch
        namespace: app-space
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /
          nginx.ingress.kubernetes.io/ssl-redirect: "false"
      spec:
        rules:
        - http:
            paths:
            - path: /wear
              pathType: Prefix
              backend:
                service:
                  name: wear-service
                  port:
                    number: 8080
            - path: /watch
              pathType: Prefix
              backend:
                service:
                  name: video-service
                  port:
                    number: 8080
      ```

      > 💡 **Ghi chú:** Bản gốc thụt lề sai ở `name`/`port` (nằm ngang hàng với `service:`), YAML đó sẽ bị API server từ chối; đoạn trên đã được sửa để `name` và `port` nằm bên trong `service`. Ingress phải nằm trong namespace `app-space` (cùng namespace với các Service backend). Có thể tạo nhanh bằng:
      `kubectl create ingress ingress-wear-watch -n app-space --rule="/wear*=wear-service:8080" --rule="/watch*=video-service:8080" --annotation nginx.ingress.kubernetes.io/rewrite-target=/ --annotation nginx.ingress.kubernetes.io/ssl-redirect=false`
     </details>

  1. Xem lời giải

     <details>

      Bấm nút `Ingress` phía trên khung terminal.
      Trong tab trình duyệt vừa mở, thử thêm `/wear` hoặc `/watch` vào sau `labs.kodekloud.com` trên thanh địa chỉ.

     </details>
