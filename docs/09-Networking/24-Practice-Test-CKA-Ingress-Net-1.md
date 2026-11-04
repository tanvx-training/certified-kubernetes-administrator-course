# Practice Test CKA Ingress 1 (Bài thực hành Ingress 1)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-cka-ingress-networking-1/)

#### Lời giải

  1. Xem lời giải

     <details>

      ```
      Ok
      ```
     </details>
  
  2. Xem lời giải

     <details>

      ```
      INGRESS-SPACE
      ```
     </details>

  3. Xem lời giải

     <details>

      ```
      NGINX-INGRESS-CONTROLLER
      ```
     </details>

  4. Xem lời giải

     <details>

      ```
      APP-SPACE
      ```
     </details>

  5. Xem lời giải

     <details>

      ```
      3
      ```
     </details>

  6. Xem lời giải

     <details>

      ```
      APP-SPACE
      ```
     </details>

  7. Xem lời giải

     <details>

      ```
      INGRESS-WEAR-WATCH
      ```
     </details>

  8. Xem lời giải

     <details>

      ```
      ALL-HOSTS(*)
      ```
     </details>

  9. Xem lời giải

     <details>

      ```
      WEAR-SERVICE
      ```
     </details>

  10. Xem lời giải

      <details>

       ```
        /WATCH
       ```
      </details>

  11. Xem lời giải

      <details>

       ```
        DEFAULT-HTTP-BACKEND
       ```
      </details>

  12. Xem lời giải

      <details>

       ```
        404-ERROR-PAGE
       ```
      </details>

  13. Xem lời giải

      <details>

       ```
        OK
       ```
      </details>

  14. Xem lời giải

      <details>
 
        ```
        kubectl edit ingress --namespace app-space
        ```

        > 💡 **Lưu ý:** Đoạn YAML dưới đây là output cũ (`extensions/v1beta1`, `serviceName`/`servicePort`) từ phiên bản Kubernetes trước v1.22. Trên cluster hiện tại, `kubectl edit ingress ingress-wear-watch -n app-space` sẽ hiển thị dạng `networking.k8s.io/v1` với `backend.service.name` và `backend.service.port.number`; bạn chỉ cần đổi giá trị `path: /watch` thành `path: /stream`.

        Đổi path từ /watch thành /stream
    
        HOẶC
 
        ```yaml
        apiVersion: v1
        items:
        - apiVersion: extensions/v1beta1
          kind: Ingress
          metadata:
            annotations:
              nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: "false"
            name: ingress-wear-watch
            namespace: app-space
          spec:
            rules:
            - http:
                paths:
                - backend:
                    serviceName: wear-service
                    servicePort: 8080
                  path: /wear
                  pathType: ImplementationSpecific
                - backend:
                    serviceName: video-service
                    servicePort: 8080
                  path: /stream
                  pathType: ImplementationSpecific
          status:
            loadBalancer:
              ingress:
              - {}
        kind: List
        metadata:
          resourceVersion: ""
          selfLink: ""
       ```
      </details>

  15. Xem lời giải

      <details>

       ```
        OK
       ```
      </details>

  16. Xem lời giải

      <details>

       ```
        404 ERROR PAGE
       ```
      </details>

  17. Xem lời giải

      <details>

       ```
        OK
       ```
      </details>

  18. Xem lời giải

      <details>

        Chạy lệnh `kubectl edit ingress --namespace app-space` và thêm một mục path mới cho Service mới.

        HOẶC

       ```yaml
       apiVersion: v1
       items:
       - apiVersion: extensions/v1beta1
         kind: Ingress
         metadata:
           annotations:
             nginx.ingress.kubernetes.io/rewrite-target: /
             nginx.ingress.kubernetes.io/ssl-redirect: "false"
           name: ingress-wear-watch
           namespace: app-space
         spec:
           rules:
           - http:
               paths:
               - backend:
                   serviceName: wear-service
                   servicePort: 8080
                 path: /wear
                 pathType: ImplementationSpecific
               - backend:
                   serviceName: video-service
                   servicePort: 8080
                 path: /stream
                 pathType: ImplementationSpecific
               - backend:
                   serviceName: food-service
                   servicePort: 8080
                 path: /eat
                 pathType: ImplementationSpecific
         status:
           loadBalancer:
             ingress:
             - {}
       kind: List
       metadata:
         resourceVersion: ""
         selfLink: ""
       ```
      </details>

  19. Xem lời giải

      <details>

       ```
        OK
       ```
      </details>

  20. Xem lời giải

      <details>

       ```
        CRITICAL-SPACE
       ```

       > 💡 Tìm namespace chứa ứng dụng mới bằng `kubectl get deploy -A` (hoặc `kubectl get pods -A`).
      </details>

  21. Xem lời giải

      <details>

       ```
        WEBAPP-PAY
       ```
      </details>

  22. Xem lời giải

      <details>

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: test-ingress
        namespace: critical-space
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /
          nginx.ingress.kubernetes.io/ssl-redirect: "false"
      spec:
        rules:
        - http:
            paths:
            - path: /pay
              pathType: Prefix
              backend:
                service:
                  name: pay-service
                  port:
                    number: 8282 
       ```

        > 💡 **Giải thích:** Ingress phải được tạo trong **cùng namespace** với Service backend (`critical-space`), vì Ingress chỉ tham chiếu được Service trong namespace của chính nó. Annotation `rewrite-target: /` cần thiết vì ứng dụng pay chỉ phục vụ tại `/`. Tạo nhanh bằng lệnh imperative:
        `kubectl create ingress test-ingress -n critical-space --rule="/pay*=pay-service:8282" --annotation nginx.ingress.kubernetes.io/rewrite-target=/`
        (thêm `--class=nginx` nếu cluster yêu cầu IngressClass). Kiểm tra port của Service bằng `kubectl get svc -n critical-space`.
        </details>

  23. Xem lời giải

      <details>

       ```
        OK
       ```
      </details>
