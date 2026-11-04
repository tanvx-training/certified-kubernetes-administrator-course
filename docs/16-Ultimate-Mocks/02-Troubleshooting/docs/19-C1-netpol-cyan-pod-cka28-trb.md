# Cluster 1, NetPol, cyan-pod-cka28-trb

Với câu hỏi này, hãy chuyển context sang cluster1 bằng lệnh:

```
kubectl config use-context cluster1
```

Một Pod chạy nginx tên là `cyan-pod-cka28-trb` đang chạy trong namespace `cyan-ns-cka28-trb` và được expose trong nội bộ cluster qua Service `cyan-svc-cka28-trb`.

Đây là một Pod bị giới hạn truy cập, vì vậy một network policy tên là `cyan-np-cka28-trb` đã được tạo trong cùng namespace để áp đặt một số hạn chế lên Pod này.

Hai Pod khác tên là `cyan-white-cka28-trb` và `cyan-black-cka28-trb` cũng đang chạy trong namespace default.

Ứng dụng nginx chạy trên Pod `cyan-pod-cka28-trb` được expose nội bộ trên port mặc định của nginx (80).

**Yêu cầu**: Ứng dụng này chỉ được phép truy cập từ Pod `cyan-white-cka28-trb`.

**Vấn đề**: Hiện tại không thể truy cập ứng dụng này từ bất cứ đâu.

Hãy troubleshoot và sửa kết nối theo đúng yêu cầu nêu trên.

Lưu ý: Bạn có thể exec vào các Pod `cyan-white-cka28-trb` và `cyan-black-cka28-trb` để kiểm tra kết nối bằng công cụ curl.

Bạn được phép sửa network policy, nhưng không được xóa nó khỏi namespace `cyan-ns-cka28-trb`.

---

### Cập nhật - Lỗi lab xảy ra không thường xuyên!

Lời giải bên dưới là đúng, tuy nhiên đôi khi nó không hoạt động do một lỗi thỉnh thoảng xảy ra khi cài Weave vào môi trường lab. Lỗi này được một thành viên rất tinh ý của cộng đồng phát hiện trong [chủ đề này](https://kodekloud.com/community/t/network-policy-blocking-all-the-ingress-traffic/300501/15?u=alistair_kodekloud) trên diễn đàn cộng đồng.

Tóm tắt - Để phát hiện lỗi này, hãy chạy hai lệnh sau. Thử thách thêm - hãy xem bạn có hiểu cách chúng hoạt động không! Lưu ý rằng lệnh đầu tiên được ngắt thành nhiều dòng bằng `\` cho dễ đọc. Đây là cú pháp hợp lệ trong shell script.

```
kubectl exec -n kube-system \
   $(kubectl get po -n kube-system --selector name=weave-net -o jsonpath='{.items[0].metadata.name}') \
   -c weave -- printenv | grep IPALLOC

kubectl get configmap -n kube-system kube-proxy -o jsonpath={'.data.config\.conf}' | yq e .clusterCIDR -
```

Cả hai phải cho ra cùng một dải CIDR, ví dụ `10.244.0.0/16`. Nếu chúng không giống nhau (giá trị cụ thể là gì không quan trọng, nhưng phải giống nhau) thì lab đang bị lỗi. Nếu gặp tình huống này (netpol không hoạt động dù đã làm đúng lời giải bên dưới), hãy vận dụng kỹ năng [schedule Pod thủ công](../../../03-Scheduling/02-Manual-Scheduling.md) để cho cả ba Pod liên quan khởi động lại trên cùng một worker node (chọn node nào cũng được). Khi đó netpol sẽ có hiệu lực.




### Lời giải

Trước tiên, hãy xem xét policy hiện có

```
k get netpol -n cyan-ns-cka28-trb cyan-np-cka28-trb -o yaml
```

> Output. (Phần metadata bổ sung đã được lược bỏ vì mỗi lần mỗi khác)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cyan-np-cka28-trb
  namespace: cyan-ns-cka28-trb
spec:
  egress:
  - ports:
    - port: 8080
      protocol: TCP
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      app: cyan-app-cka28-trb
  policyTypes:
  - Ingress
  - Egress
status: {}
```

Phần egress policy ở đây là một [chi tiết đánh lạc hướng (red herring)](https://dictionary.cambridge.org/dictionary/english/red-herring). Vì ta không quan tâm đến traffic egress đi ra từ Pod nginx, mà chỉ quan tâm ingress đi vào nó từ các Pod khác, nên phần này không liên quan đến lời giải và có thể bỏ qua.

Có hai vấn đề cần sửa ở đây. Bạn có thể sửa policy và thực hiện cả hai thay đổi chỉ với một lần chạy lệnh:

```
kubectl edit netpol -n cyan-ns-cka28-trb cyan-np-cka28-trb
```

1. Lý do ban đầu không thứ gì kết nối được là port ingress 8080 trong netpol bị sai. Nó phải là 80. Vì sao? Đề bài cho biết ứng dụng nginx trong Pod mà policy áp dụng đang lắng nghe trên port mặc định `80`. Do đó port *ingress* phải là `80` chứ không phải `8080`. Hãy sửa lại.
1. Sau khi sửa xong, mọi Pod trong namespace default đều truy cập được Pod này qua port 80, và curl sẽ trả về trang mặc định của nginx. Vì vậy ta cần thêm một podSelector vào rule để đảm bảo traffic đi vào chỉ đến từ đúng Pod được chỉ định trong namespace default – tức là một rule dạng AND.

Kết quả cuối cùng như sau. Một lần nữa phần metadata bổ sung đã được lược bỏ, nhưng bạn cứ để nguyên cũng được. Lưu và thoát `vi` để thay đổi được áp dụng.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cyan-np-cka28-trb
  namespace: cyan-ns-cka28-trb
spec:
  egress:
  - ports:
    - port: 8080
      protocol: TCP
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
      podSelector:      # <- Thêm dòng này. KHÔNG có dấu gạch trước podSelector!
        matchLabels:
          app: cyan-white-cka28-trb
    ports:
    - port: 80          # <- Sửa dòng này
      protocol: TCP
  podSelector:
    matchLabels:
      app: cyan-app-cka28-trb
  policyTypes:
  - Ingress
  - Egress
```

Chú ý rằng **không được** có dấu `-` trước podSelector vừa thêm. Nếu đặt dấu `-`, rule sẽ hoạt động như sau

> **CHO PHÉP** mọi Pod trong namespace `default` **HOẶC** mọi Pod ở bất kỳ namespace nào có label `app=cyan-white-cka28-trb`

Như vậy `cyan-black-cka28-trb` cũng truy cập được – sai yêu cầu! Không có dấu `-`, rule hoạt động đúng như sau

> **CHO PHÉP** mọi Pod trong namespace `default` **MÀ CÓ** label `app=cyan-white-cka28-trb`

Về cơ bản nghĩa là các Pod nằm trong namespace `default` **VÀ** có đúng label.

Hãy kiểm tra. Ta dùng tham số `--connect-timeout` của `curl` để không phải chờ quá lâu cho kết nối (dự kiến sẽ thất bại) từ Pod black.

```
k exec -n default cyan-white-cka28-trb -it -- curl --connect-timeout 10 cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc
```

> Output

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```
k exec -n default cyan-black-cka28-trb -it -- curl --connect-timeout 10 cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc
```

> Output

```
curl: (28) Connection timeout after 10000 ms
command terminated with exit code 28
```

Pod white kết nối được, Pod black thì không - THÀNH CÔNG!

> 💡 **Bổ sung:**
> - Trước khi sửa, hãy kiểm tra label thật của các Pod bằng `kubectl get pod -n default --show-labels` và `kubectl get pod -n cyan-ns-cka28-trb --show-labels`: `podSelector` trong rule phải khớp label của Pod nguồn (ở đây `app=cyan-white-cka28-trb`), còn `spec.podSelector` phải khớp label của Pod đích.
> - Port trong Network Policy là port của **Pod** (containerPort/targetPort), không phải port của Service. Nếu Service map port 8080 → targetPort 80 thì policy vẫn phải ghi 80.
> - Label `kubernetes.io/metadata.name` được Kubernetes tự gắn cho mọi namespace (từ v1.21), nên rất tiện để chọn namespace theo tên.
> - Dùng `kubectl describe netpol -n cyan-ns-cka28-trb cyan-np-cka28-trb` để xem policy được diễn giải ra lời – rất hữu ích để kiểm tra logic AND/OR.
