# Practice Test - Application Failure (Bài thực hành)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-application-failure/) về Application Failure

### Lời giải

> 💡 **Bổ sung — quy trình chung cho cả bài:** Với ứng dụng 2 tầng (web → `mysql-service` → mysql), hãy so sánh từng chi tiết với sơ đồ kiến trúc: tên Service, `port`/`targetPort`/`nodePort`, selector ↔ label của Pod, biến môi trường (`DB_Host`, `DB_User`, `DB_Password`) và mật khẩu của mysql. Thông báo lỗi hiển thị trên trang web là manh mối quan trọng nhất cho biết nên tìm ở tầng nào.

1.  <details>
    <summary>Một ứng dụng 2 tầng đơn giản được triển khai trong namespace <code>alpha</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi. Hãy troubleshoot và sửa lỗi.</summary>


    Tuân thủ đúng kiến trúc đã cho. Dùng đúng tên và số port như trong sơ đồ kiến trúc bên dưới. Có thể tùy ý chỉnh sửa, xóa hoặc tạo lại các object khi cần.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

      Nó cho biết

      ```
      Name does not resolve
      ```

      Đây là lỗi tra cứu DNS. CoreDNS không biết Service nào tên là `mysql-service`, trong khi sơ đồ kiến trúc nói rằng phải có Service này.

    1.  Kiểm tra các Service trong namespace `alpha`

        ```bash
        kubectl get service -n alpha
        ```

        Ta thấy có Service `mysql`, vậy khả năng cao là Service đã được tạo với tên sai.

    1.  Sửa lỗi

        Lưu ý rằng không thể dùng `kubectl edit` để đổi tên một resource.

        ```
        kubectl get service -n alpha mysql -o yaml > service.yaml
        vi service.yaml
        ```

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          creationTimestamp: "2023-10-14T11:40:30Z"
          name: mysql           # <- Sửa thành mysql-service
          namespace: alpha
          resourceVersion: "824"
          uid: d9a85021-547a-4a39-b254-0480830eab6a
        spec:
        ```

    1. Xóa và tạo lại Service

        ```bash
        kubectl delete svc mysql -n alpha
        kubectl create -f service.yaml
        ```

        > 💡 **Bổ sung:** Tên DNS của Service có dạng `<service>.<namespace>.svc.cluster.local`; Pod trong cùng namespace chỉ cần gọi `<service>`. Vì vậy chỉ cần sai một ký tự trong tên Service là ứng dụng báo `Name does not resolve`.

    </details>

2.  <details>
    <summary>Cùng ứng dụng 2 tầng đó được triển khai trong namespace <code>beta</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi. Hãy troubleshoot và sửa lỗi.</summary>

    Tuân thủ đúng kiến trúc đã cho. Dùng đúng tên và số port như trong sơ đồ kiến trúc bên dưới. Có thể tùy ý chỉnh sửa, xóa hoặc tạo lại các object khi cần.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

        Nó cho biết

        ```
        Can't connect to MySQL server on 'mysql-service:3306'
        ```

        Điều này gợi ý rằng Service đã tồn tại (DNS phân giải được) nhưng cấu hình port có thể sai.

    1. Sửa Service

        ```
        kubectl edit service mysql-service -n beta
        ```

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          creationTimestamp: "2023-10-14T11:54:04Z"
          name: mysql-service
          namespace: beta
          resourceVersion: "1166"
          uid: d6b07c71-5c49-4118-849a-0b12dd382597
        spec:
          clusterIP: 10.43.42.85
          clusterIPs:
          - 10.43.42.85
          internalTrafficPolicy: Cluster
          ipFamilies:
          - IPv4
          ipFamilyPolicy: SingleStack
          ports:
          - port: 3306        # <- Đúng
            protocol: TCP
            targetPort: 8080  # <- Sai, cũng phải là 3306
          selector:
            name: mysql
          sessionAffinity: None
          type: ClusterIP
        ```

        Database server mysql lắng nghe trên port `3306`, nghĩa là Pod của nó sẽ expose port này, và `targetPort` phải trỏ tới đó. Có thể kiểm tra như sau

        ```
        kubectl get po -n beta mysql -o jsonpath='{.spec.containers[*].ports[*].containerPort}'
        ```

        hoặc đơn giản là xem YAML của Pod.

        > 💡 **Bổ sung:** `port` là port mà Service lắng nghe (client gọi tới), còn `targetPort` là port của container mà traffic được chuyển tới. Hai giá trị này có thể khác nhau, nhưng `targetPort` bắt buộc phải khớp với port mà ứng dụng trong container thực sự lắng nghe.

    </details>

3.  <details>
    <summary>Cùng ứng dụng 2 tầng đó được triển khai trong namespace <code>gamma</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi hoặc không phản hồi. Hãy troubleshoot và sửa lỗi.</summary>

    Tuân thủ đúng kiến trúc đã cho. Dùng đúng tên và số port như trong sơ đồ kiến trúc bên dưới. Có thể tùy ý chỉnh sửa, xóa hoặc tạo lại các object khi cần.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

        Nó cho biết

        ```
         Can't connect to MySQL server on 'mysql-service:3306' (111 Connection refused)
        ```

        Điều này gợi ý rằng Service đã tồn tại nhưng có gì đó không ổn với nó.

    1. Sửa Service

        ```
        kubectl edit service mysql-service -n gamma
        ```

        Lần này tên đã đúng, cả hai port cũng đúng. Vậy có lẽ vấn đề nằm ở selector chọn Pod.

        Mở thêm một terminal (nút `+` phía trên terminal) để không phải thoát khỏi vi lúc này, rồi chạy

        ```
        kubectl get pods -n gamma --show-labels
        ```

        Để ý các label. Luôn bỏ qua label `pod-template-hash`, nó được Kubernetes dùng nội bộ.

        ```
        name=webapp-mysql,pod-template-hash=5456999f7b
        ```

        Label đó không khớp với `sql00001` trong selector của Service. Quay lại phiên vi ở Terminal 1 và sửa selector để dùng đúng giá trị của label `name`.

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          creationTimestamp: "2023-10-14T12:04:31Z"
          name: mysql-service
          namespace: gamma
          resourceVersion: "1441"
          uid: 872dfbcd-0b1d-4292-85f6-803d62d05b0a
        spec:
          clusterIP: 10.43.21.189
          clusterIPs:
          - 10.43.21.189
          internalTrafficPolicy: Cluster
          ipFamilies:
          - IPv4
          ipFamilyPolicy: SingleStack
          ports:
          - port: 3306
            protocol: TCP
            targetPort: 3306
          selector:
            name: sql00001      # <- Sửa chỗ này
          sessionAffinity: None
          type: ClusterIP
        ```

        > 💡 **Bổ sung:** Ở đây cần đối chiếu với label của Pod **mysql** (label `name=mysql`), không phải Pod webapp. Cách kiểm tra nhanh nhất là `kubectl get endpoints mysql-service -n gamma`: nếu cột `ENDPOINTS` trống (`<none>`) thì selector không khớp Pod nào. Sau khi sửa đúng, endpoint sẽ xuất hiện IP:3306 của Pod mysql.

    </details>

4.  <details>
    <summary>Cùng ứng dụng 2 tầng đó được triển khai trong namespace <code>delta</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi. Hãy troubleshoot và sửa lỗi.</summary>

    Tuân thủ đúng kiến trúc đã cho. Dùng đúng tên và số port như trong sơ đồ kiến trúc bên dưới. Có thể tùy ý chỉnh sửa, xóa hoặc tạo lại các object khi cần.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

        Nó cho biết

        ```
        Access denied for user 'sql-user'@'10.42.0.16'
        ```

        Nghĩa là ứng dụng đang dùng sai tài khoản người dùng mysql. Sơ đồ kiến trúc trong đề bài cho biết thông tin đăng nhập (credentials) đúng là gì.

        Vậy lần này cần sửa ở Deployment của ứng dụng chứ không phải ở Service.

    1.  Sửa Deployment

        ```
        kubectl edit deployment webapp-mysql -n delta
        ```

        Cuộn xuống phần biến môi trường (env) của container và sửa lại tên người dùng

        ```yaml
          template:
            metadata:
              creationTimestamp: null
              labels:
                name: webapp-mysql
              name: webapp-mysql
            spec:
              containers:
              - env:
                - name: DB_Host
                  value: mysql-service
                - name: DB_User
                  value: sql-user   # <- Sửa chỗ này
                - name: DB_Password
                  value: paswrd
                image: mmumshad/simple-webapp-mysql
                imagePullPolicy: Always
                name: webapp-mysql

        ```

        > 💡 **Bổ sung:** Sửa template của Deployment sẽ kích hoạt một rolling update — Pod cũ bị thay bằng Pod mới mang biến môi trường mới. Có thể theo dõi bằng `kubectl rollout status deployment webapp-mysql -n delta`. Cách khác nhanh hơn là `kubectl set env deployment/webapp-mysql -n delta DB_User=<giá-trị-đúng>`.

    </details>

5.  <details>
    <summary>Cùng ứng dụng 2 tầng đó được triển khai trong namespace <code>epsilon</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi. Hãy troubleshoot và sửa lỗi.</summary>

    Đề bài cho biết có *hai* lỗi cần sửa.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

        Nó cho biết

        ```
        Access denied for user 'sql-user'@'10.42.0.16'
        ```

        Vậy lại là một lỗi xác thực.

    1. Kiểm tra Deployment

        Đối chiếu các biến môi trường với giá trị trong sơ đồ kiến trúc. Có vẻ giống lỗi ở câu trước, nên hãy sửa theo cách tương tự.

    1.  Khoan đã! Đã sửa rồi mà *vẫn* chưa chạy!

        Kiểm tra Pod mysql, vì phía mysql cũng cần một số thông tin xác thực.

        Không thể dùng `kubectl edit` để thay đổi giá trị env của một Pod độc lập (standalone Pod).

        ```
        kubectl get pod -n epsilon mysql -o yaml > mysql.yaml
        vi mysql.yaml
        ```

        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: "2023-10-14T12:21:51Z"
          labels:
            name: mysql
          name: mysql
          namespace: epsilon
          resourceVersion: "2002"
          uid: f369770b-be24-43c1-b754-d7e1396d6952
        spec:
          containers:
          - env:
            - name: MYSQL_ROOT_PASSWORD
              value: passwooooorrddd    # <- Sửa chỗ này
            image: mysql:5.6
            imagePullPolicy: IfNotPresent
            name: mysql
            ports:
        ```

    1. Tạo lại Pod

        ```
        kubectl replace --force -f mysql.yaml
        ```

        > 💡 **Bổ sung:** Với Pod đang chạy, chỉ một số ít trường được phép sửa trực tiếp (vd `image`, `activeDeadlineSeconds`, `tolerations` bổ sung). Nếu bạn vẫn dùng `kubectl edit` và lưu, kubectl sẽ báo lỗi và lưu bản sửa vào một file tạm trong `/tmp/` — có thể dùng ngay file đó với `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml`. `replace --force` thực chất là xóa rồi tạo lại Pod.

    </details>

6.  <details>
    <summary>Cùng ứng dụng 2 tầng đó được triển khai trong namespace <code>zeta</code>. Khi chạy thành công, nó phải hiển thị một trang web màu xanh lá. Click vào tab <code>App</code> ở phía trên terminal để xem ứng dụng. Hiện tại nó đang bị lỗi. Hãy troubleshoot và sửa lỗi.</summary>

    Đề bài cho biết có *ba* lỗi cần sửa.

    1. Click vào nút `App` ở phía trên terminal. Quan sát thông báo lỗi.

        Ta nhận được lỗi `502 Bad Gateway`.

        Lỗi này cho thấy hạ tầng hiển thị của lab không kết nối được tới Service mà nó cần kết nối. Hãy xem URL trên thanh địa chỉ của trình duyệt

        ```
        30081-port-1795f98fde814933.labs.kodekloud.com/
        ```

        Trên lab của KodeKloud, phần `30081-port` cho biết NodePort mà nó đang cố kết nối tới. Để ý thêm rằng sơ đồ hạ tầng ghi `30081` phải là NodePort của `web-service`.

    1.  Kiểm tra `web-service` vì đó là cách chúng ta truy cập ứng dụng.

        ```
        kubectl edit service -n zeta web-service
        ```

        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          creationTimestamp: "2023-10-14T13:27:26Z"
          name: web-service
          namespace: zeta
          resourceVersion: "1530"
          uid: 9b655dda-675a-43ae-80e4-deadb3a38179
        spec:
          clusterIP: 10.43.48.98
          clusterIPs:
          - 10.43.48.98
          externalTrafficPolicy: Cluster
          internalTrafficPolicy: Cluster
          ipFamilies:
          - IPv4
          ipFamilyPolicy: SingleStack
          ports:
          - nodePort: 30088  # <- Sửa chỗ này
            port: 8080
            protocol: TCP
            targetPort: 8080
          selector:
            name: webapp-mysql
          sessionAffinity: None
          type: NodePort
        ```

    1.  Thử lại ứng dụng trên trình duyệt

        Thông báo này ta đã gặp trước đây! Sửa theo cách như ở trên (lỗi `DB_User` của Deployment), dùng namespace `zeta`.

    1.  Thử lại ứng dụng trên trình duyệt

        Thông báo này ta cũng đã gặp! Sửa theo cách như ở trên (lỗi `MYSQL_ROOT_PASSWORD` của Pod mysql), dùng namespace `zeta`.

    </details>
