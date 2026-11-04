# Ingress Annotations và rewrite-target

  - Xem [Bài giảng](https://kodekloud.com/topic/ingress-annotations-and-rewrite-target/)

Trong phần này, chúng ta sẽ tìm hiểu về **Ingress annotations và rewrite-target**.

- Mỗi Ingress Controller có những tùy chọn riêng để tùy biến cách hoạt động (thường khai báo qua annotation). NGINX Ingress Controller có rất nhiều tùy chọn, nhưng ở đây chúng ta sẽ tìm hiểu một trong số đó: tùy chọn "Rewrite Target".

- Vấn đề: ứng dụng `pay-service` chỉ phục vụ ở đường dẫn gốc `/`. Nếu không rewrite, request `http://<ingress>/pay` được chuyển nguyên đường dẫn `/pay` tới backend, và ứng dụng sẽ trả về `404`. Annotation `nginx.ingress.kubernetes.io/rewrite-target: /` viết lại phần path khớp (`/pay`) thành `/` trước khi chuyển request tới backend.

- Kubernetes phiên bản 1.18 (API cũ `extensions/v1beta1`)

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282

```

> 💡 **Bổ sung:** Từ Kubernetes v1.22, `extensions/v1beta1` đã bị gỡ bỏ. Ví dụ trên viết lại theo `networking.k8s.io/v1`:
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: test-ingress
>   namespace: critical-space
>   annotations:
>     nginx.ingress.kubernetes.io/rewrite-target: /
> spec:
>   ingressClassName: nginx
>   rules:
>   - http:
>       paths:
>       - path: /pay
>         pathType: Prefix
>         backend:
>           service:
>             name: pay-service
>             port:
>               number: 8282
> ```
>
> **Lưu ý về rewrite-target:**
> - Với `rewrite-target: /`, **toàn bộ** path khớp đều bị viết lại thành `/`, nghĩa là `/pay/abc` cũng sẽ thành `/`. Nếu muốn giữ phần đuôi (ví dụ `/pay/abc` → `/abc`), dùng capture group của regex:
>
>   ```yaml
>   metadata:
>     annotations:
>       nginx.ingress.kubernetes.io/use-regex: "true"
>       nginx.ingress.kubernetes.io/rewrite-target: /$2
>   spec:
>     rules:
>     - http:
>         paths:
>         - path: /pay(/|$)(.*)
>           pathType: ImplementationSpecific
>           backend:
>             service:
>               name: pay-service
>               port:
>                 number: 8282
>   ```
>
> - Annotation là đặc thù của từng controller: `nginx.ingress.kubernetes.io/*` chỉ có tác dụng với ingress-nginx; controller khác (Traefik, HAProxy...) dùng annotation/cơ chế khác. Một annotation hay gặp khác trong lab là `nginx.ingress.kubernetes.io/ssl-redirect: "false"` (tắt chuyển hướng HTTP → HTTPS).
> - Trong Gateway API, rewrite được chuẩn hóa qua filter `URLRewrite` của HTTPRoute (ví dụ `type: ReplacePrefixMatch`), không cần annotation riêng của controller.



#### Tài liệu tham khảo

- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
- https://kubernetes.github.io/ingress-nginx/examples/
- https://kubernetes.github.io/ingress-nginx/examples/rewrite/
- https://github.com/kubernetes/ingress-nginx/blob/master/docs/troubleshooting.md