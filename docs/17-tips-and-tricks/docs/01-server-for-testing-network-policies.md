# Server để kiểm thử Network Policy

Đôi khi đề thi yêu cầu bạn chặn ingress vào một Pod trên mọi port trừ một vài port cụ thể. Nếu trong namespace đó chưa có Pod nào lắng nghe trên port được yêu cầu, thì vấn đề đặt ra là "Làm sao tạo một Pod lắng nghe trên port đó để gắn netpol vào và kiểm thử policy?". Bạn không thể chỉ chạy một Pod nginx vì nó luôn lắng nghe ở port 80. Có thể cấu hình khác đi, nhưng như vậy phải mount một ConfigMap chứa file cấu hình nginx với port mới vào Pod – quá rắc rối trong điều kiện phòng thi!

## Server đơn giản

May mắn là bản phân phối Python mặc định có sẵn một server đơn giản, có thể cấu hình port ngay trên dòng lệnh, nghĩa là bạn có thể chạy nó theo kiểu imperative. Giả sử network policy yêu cầu chặn mọi port trừ port 9000. Ta có thể khởi chạy một Pod server thử nghiệm lắng nghe trên port 9000 như sau. Nếu là port khác, chỉ cần thay 9000 bằng port đó.

```
kubectl run server --image python --command -- python -m http.server 9000
```

Lấy địa chỉ IP của Pod. Dùng IP để test bằng curl sẽ nhanh hơn gõ tên DNS.

```
controlplane $ k get pod server -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
server   1/1     Running   0          16s   192.168.1.12   node01   <none>           <none>
```

Giờ chạy một Pod có `curl` và kiểm tra kết nối tới server

```
curl 192.168.1.12:9000
```

Bạn sẽ nhận được phản hồi.

Bây giờ apply network policy của bạn và kiểm tra lại.

> 💡 **Bổ sung:** Network Policy chọn Pod theo **label**, nên hãy gắn label phù hợp cho Pod server (`kubectl run` tự gắn `run=server`; có thể thêm `--labels=app=db` nếu policy chọn theo label khác). Image `python` khá nặng, lần đầu pull có thể mất một lúc – chờ Pod `Running` rồi mới test. Ngoài ra, có thể thêm `--port 9000` vào lệnh `kubectl run` để khai báo containerPort (không bắt buộc để server hoạt động).

## Server nâng cao hơn một chút

Có thể bạn muốn dựng nhiều Pod, mỗi Pod trả về một thông điệp riêng trên một port tùy chỉnh để phân biệt chúng qua phản hồi. Ta có thể làm vậy với một Pod và một ConfigMap cho mỗi server. Pod giống hệt nhau mỗi lần – chỉ khác ở tên riêng và ConfigMap được mount.

Ví dụ sau mô phỏng một Pod xuất hiện trong một câu hỏi network policy của Killer.sh.

1. Tạo một ConfigMap chứa shell script để chạy server trên một port với một thông điệp cho trước

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: db1-configmap
    data:
      entrypoint.sh: |
        #!/bin/sh
        echo "database one" > index.htm  #<- Thông điệp trả về
        port=1111                        #<- Port của server
        # Chạy server...
        while true
        do
            { echo -ne "HTTP/1.0 200 OK\r\nContent-Length: $(wc -c <index.htm)\r\n\r\n"; cat index.htm; } | nc -l -p $port
        done
    ```
1. Tạo một Pod mount ConfigMap và chạy script bên trong nó

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: db-1
    spec:
      containers:
      - name: server
        image: alpine:3.19
        command:
        - /opt/server/entrypoint.sh
        volumeMounts:
        - name: script
          mountPath: /opt/server
      volumes:
      - name: script
        configMap:
          name: db1-configmap
          defaultMode: 0755
    ```

    > 💡 **Ghi chú:** Thụt lề YAML của hai manifest trên trong bản gốc bị lệch (ví dụ `name` không nằm dưới `metadata`) nên không apply được; bản dịch đã sửa lại thụt lề, nội dung giữ nguyên. `defaultMode: 0755` cần thiết để file script có quyền thực thi. Image `alpine` có sẵn `nc` (của busybox) nên không phải cài thêm.
1. Lấy địa chỉ IP của Pod. Dùng IP để test bằng curl sẽ nhanh hơn gõ tên DNS.

    ```
    controlplane $ k get pod db-1 -o wide
    NAME     READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
    db-1     1/1     Running   0          16s   192.168.1.12   node01   <none>           <none>
    ```

1. Giờ chạy một Pod có `curl` và kiểm tra kết nối tới server

    ```
    curl 192.168.1.12:1111
    ```


## Xem thêm

Xem thêm [client để kiểm thử](./02-client--for-testing-network-things.md)

