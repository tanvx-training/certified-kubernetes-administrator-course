# Lightning Lab 1

  - Tôi đã sẵn sàng! [Đi tới Lightning Lab 1](https://kodekloud.com/topic/lightning-lab-1-2/)

## Lời giải LL-1

1.  <details>
    <summary>Upgrade phiên bản Kubernetes hiện tại từ 1.28.0 lên đúng 1.29.0 bằng công cụ kubeadm.</summary>

    Đảm bảo việc upgrade được thực hiện lần lượt từng node, bắt đầu từ node controlplane. Để giảm thiểu thời gian gián đoạn (downtime), Deployment `gold-nginx` phải được chuyển sang chạy trên một node khác trước khi upgrade mỗi node.


    Upgrade node `controlplane` trước và drain node `node01` trước khi upgrade nó. Sau đó, các Pod của `gold-nginx` phải chạy trên node controlplane.

    **Upgrade `controlplane`**

    1.  Cập nhật package repo

        ```bash
        apt update
        ```

    1.  Dùng madison để xem các package Kubernetes nào đang có sẵn

        ```bash
        apt-cache madison kubeadm
        ```

        Để ý rằng chỉ có các phiên bản 1.28, nghĩa là bạn phải thêm repo của 1.29

    1.  Lấy repo Kubernetes 1.29

        Để làm việc này, cần sửa file nguồn apt, file này nằm ở `/etc/apt/sources.list.d/kubernetes.list`

        ```bash
        vi /etc/apt/sources.list.d/kubernetes.list
        ```

        Tìm mọi chỗ có `1.28` trong file này và đổi thành `1.29`, sau đó lưu và thoát vi.

        > 💡 **Bổ sung:** Kể từ khi chuyển sang repo cộng đồng `pkgs.k8s.io` (repo cũ `apt.kubernetes.io` đã ngừng hoạt động từ năm 2024), mỗi minor version có một repo riêng (`.../core:/stable:/v1.29/deb/`). Vì vậy khi upgrade lên minor version mới, luôn phải đổi repo trước rồi `apt update` lại. Có thể sửa nhanh bằng `sed -i 's/v1.28/v1.29/g' /etc/apt/sources.list.d/kubernetes.list`. Tài liệu "Upgrading kubeadm clusters" trên kubernetes.io mô tả đầy đủ các bước và được phép tra cứu khi thi.

    1.  Chạy lại madison để tìm phiên bản package của 1.29

        ```bash
        apt-cache madison kubeadm
        ```

        Bạn sẽ thấy dòng sau trong danh sách

        > `kubeadm | 1.29.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.29/deb  Packages`

        Giờ ta đã biết phiên bản package là `1.29.0-1.1`, có thể tiến hành upgrade

    1. Drain node

        ```
        kubectl drain controlplane --ignore-daemonsets
        ```

    1. Upgrade kubeadm

        ```
        apt-mark unhold kubeadm
        apt install -y kubeadm=1.29.0-1.1
        ```

    1. Lập kế hoạch (plan) và áp dụng (apply) upgrade

        ```
        kubeadm upgrade plan
        kubeadm upgrade apply v1.29.0
        ```

    1. Upgrade kubelet

        ```
        apt-mark unhold kubelet
        apt install -y kubelet=1.29.0-1.1
        systemctl daemon-reload
        systemctl restart kubelet
        ```

    1. Đưa node controlplane trở lại hoạt động

        ```
        kubectl uncordon controlplane
        ```

    1. Upgrade kubectl

        ```
        apt-mark unhold kubectl
        apt install -y kubectl=1.29.0-1.1
        ```

    1. Hold lại các package

        ```
        apt-mark hold kubeadm kubelet kubectl
        ```

        > 💡 **Bổ sung:** `apt-mark hold` ngăn package bị nâng cấp ngoài ý muốn khi chạy `apt upgrade`. Trình tự chuẩn trên controlplane: upgrade kubeadm → `kubeadm upgrade apply` → drain → upgrade kubelet/kubectl → restart kubelet → uncordon. Kiểm tra kết quả bằng `kubectl get nodes` (cột `VERSION` hiển thị phiên bản kubelet của từng node) và `kubeadm version`.

    **Upgrade `node01`**

    1. Drain worker node

        ```
        kubectl drain node01 --ignore-daemonsets
        ```

        > 💡 **Bổ sung:** Lúc này controlplane đã được uncordon, nên khi drain `node01` các Pod của `gold-nginx` sẽ được tạo lại trên controlplane — đúng như yêu cầu đề bài. Nếu lệnh drain báo lỗi do có Pod không được quản lý bởi controller hoặc dùng `emptyDir`, có thể cần thêm `--force` / `--delete-emptydir-data` (hãy đọc kỹ thông báo lỗi trước khi dùng).

    1. Đi tới worker node

        ```
        ssh node01
        ```

    1. Như trước, bạn cần cập nhật package cache cho v1.29

        Làm theo các bước giống như đã làm trên `controlplane`

    1. Upgrade kubeadm

        ```
        apt-mark unhold kubeadm
        apt install -y kubeadm=1.29.0-1.1
        ```

    1. Upgrade node

        ```
        kubeadm upgrade node
        ```

    1. Upgrade kubelet

        ```
        apt-mark unhold kubelet
        apt install kubelet=1.29.0-1.1
        systemctl daemon-reload
        systemctl restart kubelet
        ```

    1. Hold lại các package

        ```
        apt-mark hold kubeadm kubelet
        ```

    1. Quay về controlplane

        ```
        exit
        ```

    1. Đưa worker node trở lại hoạt động

        ```
        kubectl uncordon node01
        ```

    1. Xác nhận `gold-nginx` đã được schedule lên node controlplane

        ```
        kubectl get pods -o wide | grep gold-nginx
        ```

    **MẸO**

    Để upgrade cluster nhanh hơn và tiết kiệm ít nhất 3 phút, bạn có thể làm việc trên cả hai node cùng lúc.

    Trong khi `kubeadm upgrade apply` đang chạy trên `controlplane` (mất vài phút), hãy mở terminal thứ hai và thực hiện các bước `ii`, `iii` và `iv` của phần "Upgrade `node01`", để node sẵn sàng chạy `kubeadm upgrade node` ngay sau khi bạn drain nó.


    </details>

2.  <details>
    <summary>In ra tên của tất cả Deployment trong namespace admin2406 theo định dạng sau...</summary>

    Đây là việc dành cho output `custom-columns` của kubectl

    ```
    kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
    ```

    > 💡 **Giải thích:** Mỗi cặp `TÊN-CỘT:đường-dẫn` tạo ra một cột; đường dẫn tính từ từng Deployment. Image của container nằm trong pod template nên đường dẫn là `.spec.template.spec.containers[].image` (cũng có thể viết `[0]` hoặc `[*]`). Kiểm tra lại kết quả bằng `cat /opt/admin2406_data` để chắc chắn tên cột và thứ tự sắp xếp đúng như đề bài.
    </details>

3.  <details>
    <summary>Một file kubeconfig tên là admin.kubeconfig đã được tạo trong /root/CKA. Cấu hình này có chỗ bị sai. Hãy troubleshoot và sửa lỗi.</summary>

    Trước tiên, hãy thử kubeconfig này

    ```
    kubectl get pods --kubeconfig /root/CKA/admin.kubeconfig
    ```

    Để ý thông báo lỗi.

    Giờ xem kubeconfig mặc định để biết cấu hình đúng.

    ```
    cat ~/.kube/config
    ```

    Sửa lại

    ```
    vi /root/CKA/admin.kubeconfig
    ```

    Kiểm tra

    ```
    kubectl get pods --kubeconfig /root/CKA/admin.kubeconfig
    ```

    > 💡 **Giải thích:** Lỗi thường gặp trong file này là sai port của API server ở trường `server:` (port đúng là `6443`), dẫn tới thông báo `connection refused`. Hãy so sánh trường `server`, các certificate và tên cluster/user/context giữa hai file.
  </details>

4.  <details>
    <summary>Tạo một Deployment mới tên là nginx-deploy, với image nginx:1.16 và 1 replica. Sau đó upgrade Deployment lên phiên bản 1.17 bằng rolling update.</summary>

    ```
    kubectl create deployment nginx-deploy --image=nginx:1.16
    kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
    ```

    Bạn có thể bỏ qua cảnh báo deprecation (tính năng sắp bị loại bỏ).

    > 💡 **Bổ sung:** Cờ `--record` đã bị deprecated; bạn có thể bỏ nó đi. Trong `nginx=nginx:1.17`, phần bên trái dấu `=` là **tên container** — `kubectl create deployment` tự đặt tên container theo tên image (ở đây là `nginx`). Nếu không chắc, kiểm tra bằng `kubectl get deploy nginx-deploy -o jsonpath='{.spec.template.spec.containers[*].name}'`. Xác nhận rolling update bằng `kubectl rollout status deployment/nginx-deploy` và `kubectl rollout history deployment/nginx-deploy`. `kubectl create deployment` mặc định tạo 1 replica, nên không cần `--replicas=1`.

    </details>

5.  <details>
    <summary>Một Deployment mới tên là alpha-mysql đã được triển khai trong namespace alpha. Tuy nhiên, các Pod của nó không chạy. Hãy troubleshoot và sửa lỗi.</summary>

    Deployment này phải sử dụng persistent volume alpha-pv được mount tại /var/lib/mysql và phải dùng biến môi trường MYSQL_ALLOW_EMPTY_PASSWORD=1 để dùng mật khẩu root rỗng.

    Quan trọng: Không được thay đổi persistent volume.

    Kiểm tra Deployment để chắc chắn biến môi trường đã được đặt. Ở đây tôi dùng `yq` — giống `jq` nhưng dành cho YAML — để không phải xem _toàn bộ_ YAML của Deployment, chỉ xem phần dưới `containers` trong spec của Deployment.

    ```
    kubectl get deployment -n alpha alpha-mysql  -o yaml | yq e .spec.template.spec.containers -
    ```

    Tìm hiểu vì sao Deployment không đạt minimum availability. Trước tiên ta cần biết tên Pod của Deployment, sau đó describe Pod để xem lỗi.

    ```
    kubectl get pods -n alpha
    kubectl describe pod -n alpha alpha-mysql-xxxxxxxx-xxxxx
    ```

    Ta thấy PVC được yêu cầu không tồn tại, vậy hãy tạo nó. Trước tiên, kiểm tra Persistent Volume để lấy giá trị của access modes, capacity (storage) và storage class name

    ```
    kubectl get pv alpha-pv
    ```

    Giờ dùng `vi` để tạo manifest cho PVC

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-alpha-pvc
      namespace: alpha
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: slow
    ```

    > 💡 **Bổ sung:** Tên PVC (`mysql-alpha-pvc`) phải đúng với tên `claimName` mà Deployment tham chiếu (xem phần `volumes` trong output của `kubectl get deploy alpha-mysql -n alpha -o yaml`). Sau khi lưu manifest (vd `pvc.yaml`), đừng quên tạo nó bằng `kubectl apply -f pvc.yaml`, rồi kiểm tra `kubectl get pvc -n alpha` — PVC phải ở trạng thái `Bound` với `alpha-pv`. Để PVC bind được vào PV, `accessModes` và `storageClassName` phải khớp, và dung lượng yêu cầu không được lớn hơn capacity của PV. Khi PVC đã Bound, Pod sẽ tự chuyển từ `Pending` sang `Running`.
  </details>

6.  <details>
    <summary>Backup ETCD vào vị trí /opt/etcd-backup.db trên node controlplane.</summary>

    Câu hỏi này diễn đạt hơi khó hiểu. Nó yêu cầu ta tạo bản backup của etcd và lưu bản backup tại vị trí đã cho.

    Cần biết rằng các certificate dùng để xác thực `etcdctl` nằm trong `/etc/kubernetes/pki/etcd`

    ```
    ETCDCTL_API='3' etcdctl snapshot save \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key \
      /opt/etcd-backup.db
    ```

    Dù _có thể_ thêm tham số `--endpoints=127.0.0.1:2379`, nhưng ở đây không cần thiết vì ta đang ở trên node controlplane, cùng chỗ với `etcd`. Endpoint mặc định là localhost.

    > 💡 **Bổ sung:** Nếu không nhớ đường dẫn certificate, hãy xem trong manifest của etcd: `grep -E "cert|key|listen-client" /etc/kubernetes/manifests/etcd.yaml` (các tham số `--trusted-ca-file`, `--cert-file`, `--key-file`). Từ etcdctl v3.4, API v3 là mặc định nên `ETCDCTL_API=3` không còn bắt buộc, nhưng thêm vào cũng không sao. Kiểm tra file backup bằng `etcdutl snapshot status /opt/etcd-backup.db -w table` (hoặc lệnh cũ `etcdctl snapshot status`, đã deprecated ở các bản mới).
    </details>

7.  <details>
    <summary>Tạo một Pod tên là secret-1401 trong namespace admin1401 dùng image busybox....</summary>

    Container trong Pod phải có tên `secret-admin` và sleep trong 4800 giây.

    Container phải mount một secret volume chỉ đọc (read-only) tên là secret-volume tại đường dẫn `/etc/secret-volume`. Secret được mount đã được tạo sẵn và có tên là `dotfile-secret`.

    1. Dùng lệnh imperative để tạo manifest khởi đầu

        ```
        kubectl run secret-1401 -n admin1401 --image busybox --dry-run=client -o yaml --command -- sleep 4800 > admin.yaml
        ```

    1. Sửa manifest này để thêm thông tin mount secret

        ```
        vi admin.yaml
        ```

        Thêm phần volume và volume mount như bên dưới (đồng thời đổi tên container thành `secret-admin`, vì `kubectl run` mặc định đặt tên container trùng tên Pod)

        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: null
          labels:
            run: secret-1401
          name: secret-1401
          namespace: admin1401
        spec:
          volumes:
          - name: secret-volume
            secret:
              secretName: dotfile-secret
          containers:
          - command:
            - sleep
            - "4800"
            image: busybox
            name: secret-admin
            volumeMounts:
            - name: secret-volume
              readOnly: true
              mountPath: /etc/secret-volume
        ```

    1. Và tạo Pod

        ```
        kubectl create -f admin.yaml
        ```

        > 💡 **Bổ sung:** Kiểm tra lại bằng `kubectl -n admin1401 describe pod secret-1401` (phần `Mounts` và `Volumes`) hoặc `kubectl -n admin1401 exec secret-1401 -- ls -la /etc/secret-volume`. Mỗi key trong Secret sẽ trở thành một file trong thư mục mount (ở đây là một "dotfile", tức file có tên bắt đầu bằng dấu chấm, nên cần `ls -a` mới thấy).

  </details>
