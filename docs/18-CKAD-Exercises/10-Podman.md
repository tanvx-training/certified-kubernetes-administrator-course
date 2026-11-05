# Định nghĩa, build và chỉnh sửa container image

- Lưu ý: chủ đề này nằm trong syllabus CKAD mới. Dưới đây là vài ví dụ dùng **podman** để quản lý vòng đời của container image. Nhiều năm qua **docker** là tiêu chuẩn của ngành, nhưng giờ các công ty lớn như [Red Hat](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo) đang chuyển sang bộ công cụ mã nguồn mở mới gồm podman, skopeo và buildah. Kubernetes cũng đã đi theo [hướng này](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). Cụ thể, `podman` được thiết kế để thay thế lệnh `docker`, nên làm quen với nó là hợp lý, dù hai công cụ gần như dùng thay thế được cho nhau vì có cùng cú pháp.

> 💡 **Bổ sung:**
> - Kubernetes gỡ dockershim từ **v1.24**, nên kubelet không gọi Docker Engine trực tiếp nữa mà dùng container runtime theo chuẩn CRI (containerd, CRI-O). Image do `docker` hay `podman` build đều theo chuẩn **OCI**, nên chạy được trên mọi runtime.
> - Khác biệt chính của podman: **không có daemon** (daemonless) và chạy được **rootless** (không cần quyền root). Hầu hết lệnh giống docker; có thể đặt `alias docker=podman`.
> - Podman build được cả `Dockerfile` lẫn `Containerfile` (tên file mặc định được tìm theo thứ tự `Containerfile`, rồi `Dockerfile`).
> - Với CKA: trên node bạn sẽ làm việc với `crictl` (ví dụ `crictl ps`, `crictl images`, `crictl logs`) chứ không phải docker/podman; image build bằng podman là phần của CKAD.

## Podman cơ bản

### Viết Dockerfile để triển khai Apache HTTP Server với trang chủ tùy chỉnh

<details><summary>Xem lời giải</summary>
<p>

```Dockerfile
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

> 💡 **Bổ sung:** Nên ghi tên image đầy đủ (`docker.io/httpd:2.4`) vì podman không mặc định dùng Docker Hub; nếu dùng tên ngắn, podman tra bảng alias hoặc hỏi bạn chọn registry. Các instruction hay dùng khác: `COPY`/`ADD`, `WORKDIR`, `ENV`, `EXPOSE`, `USER`, `ENTRYPOINT`/`CMD`. Trong Kubernetes, `command` ghi đè `ENTRYPOINT`, còn `args` ghi đè `CMD`.

</p>
</details>

### Build image và xem image gồm bao nhiêu layer

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman build -t simpleapp .
STEP 1/2: FROM httpd:2.4
STEP 2/2: RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
COMMIT simpleapp
--> ef4b14a72d0
Successfully tagged localhost/simpleapp:latest
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b

:~$ podman images
REPOSITORY               TAG         IMAGE ID      CREATED        SIZE
localhost/simpleapp      latest      ef4b14a72d02  8 seconds ago  148 MB
docker.io/library/httpd  2.4         98f93cd0ec3b  7 days ago     148 MB

:~$ podman image tree localhost/simpleapp:latest
Image ID: ef4b14a72d02
Tags:     [localhost/simpleapp:latest]
Size:     147.8MB
Image Layers
├── ID: ad6562704f37 Size:  83.9MB
├── ID: c234616e1912 Size: 3.072kB
├── ID: c23a797b2d04 Size: 2.721MB
├── ID: ede2e092faf0 Size: 61.11MB
├── ID: 971c2cdf3872 Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└── ID: 61644e82ef1f Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

> 💡 **Bổ sung:**
> - Image này có 6 layer: 5 layer từ base image `httpd:2.4` và 1 layer do lệnh `RUN` tạo ra. Mỗi instruction `RUN`/`COPY`/`ADD` sinh ra một layer mới, nên gộp nhiều lệnh `RUN` bằng `&&` sẽ giảm số layer.
> - Ở một số bản podman mới, lệnh `podman image tree` có thể đã bị đánh dấu deprecated (kiểm tra bằng `podman image tree --help`). Cách thay thế phổ biến là `podman history localhost/simpleapp` (dùng được cả với docker) hoặc `podman inspect --format '{{.RootFS.Layers}}' localhost/simpleapp`.
> - Image không có prefix registry được podman gắn tag `localhost/`. Build với Dockerfile ở đường dẫn khác: `podman build -t simpleapp -f path/to/Dockerfile .`

</p>
</details>

### Chạy image ở local, xem trạng thái và log, cuối cùng kiểm tra nó phản hồi đúng như mong đợi

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman run -d --name test -p 8080:80 localhost/simpleapp
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8

:~$ podman ps
CONTAINER ID  IMAGE                       COMMAND           CREATED        STATUS            PORTS                 NAMES
2f3d7d613ea6  localhost/simpleapp:latest  httpd-foreground  5 seconds ago  Up 6 seconds ago  0.0.0.0:8080->80/tcp  test

:~$ podman logs test
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
[Sat Jun 04 16:15:38.071377 2022] [mpm_event:notice] [pid 1:tid 139756978220352] AH00489: Apache/2.4.53 (Unix) configured -- resuming normal operations
[Sat Jun 04 16:15:38.073570 2022] [core:notice] [pid 1:tid 139756978220352] AH00094: Command line: 'httpd -D FOREGROUND'

:~$ curl 0.0.0.0:8080
Hello, World!
```

> 💡 **Bổ sung:** `-p 8080:80` có dạng `<port-host>:<port-container>`. Ở chế độ rootless, podman không bind được port nhỏ hơn 1024 trên host (trừ khi chỉnh `net.ipv4.ip_unprivileged_port_start`), nên dùng port như 8080.

</p>
</details>

### Chạy một lệnh bên trong container để in file index.html

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman exec -it test cat /usr/local/apache2/htdocs/index.html
Hello, World!
```

> 💡 **Bổ sung:** Bản gốc ghi "inside the pod", nhưng `test` thực ra là một container. Podman cũng có khái niệm pod riêng (`podman pod create`), một nhóm container dùng chung network namespace, giống Pod trong Kubernetes. Podman còn sinh được manifest Kubernetes từ container/pod đang chạy bằng `podman kube generate test` (bản cũ: `podman generate kube`).

</p>
</details>

### Gắn tag cho image theo IP và port của một private registry ở local, rồi push image lên registry đó

<details><summary>Xem lời giải</summary>
<p>

> Lưu ý: một số bản phân phối Kubernetes nhỏ (như [microk8s](https://microk8s.io/docs/registry-built-in)) có sẵn registry tích hợp để bạn dùng cho bài này. Nếu không, bạn sẽ phải tự dựng một registry.

```bash
:~$ podman tag localhost/simpleapp $registry_ip:5000/simpleapp

:~$ podman push $registry_ip:5000/simpleapp
```

> 💡 **Bổ sung:**
> - Dựng nhanh một registry local: `podman run -d -p 5000:5000 --name registry docker.io/library/registry:2`.
> - Registry chạy HTTP (không có TLS) sẽ bị từ chối với lỗi `http: server gave HTTP response to HTTPS client`. Cách xử lý: thêm `--tls-verify=false` vào `podman push`/`podman pull`, hoặc khai báo registry là `insecure = true` trong `/etc/containers/registries.conf`.
> - Để Pod trong cluster pull được image này, container runtime trên **node** (containerd/CRI-O) cũng phải được cấu hình tin cậy registry đó.

</p>
</details>

### Kiểm tra registry đã chứa image vừa push và bạn có thể pull image đó về

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ curl http://$registry_ip:5000/v2/_catalog
{"repositories":["simpleapp"]}

# xóa image đang có ở local
:~$ podman rmi $registry_ip:5000/simpleapp

:~$ podman pull $registry_ip:5000/simpleapp
Trying to pull 10.152.183.13:5000/simpleapp:latest...
Getting image source signatures
Copying blob 643ea8c2c185 skipped: already exists
Copying blob 972107ece720 skipped: already exists
Copying blob 9857eeea6120 skipped: already exists
Copying blob 93859aa62dbd skipped: already exists
Copying blob 8e47efbf2b7e skipped: already exists
Copying blob 42e0f5a91e40 skipped: already exists
Copying config ef4b14a72d done
Writing manifest to image destination
Storing signatures
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
```

> 💡 **Bổ sung:** Các blob báo `skipped: already exists` vì `podman rmi` chỉ gỡ tag `$registry_ip:5000/simpleapp`; các layer vẫn còn trong storage local vì tag `localhost/simpleapp` đang dùng chúng. Xem danh sách tag của một image trên registry: `curl http://$registry_ip:5000/v2/simpleapp/tags/list`.

</p>
</details>

### Tạo container mà không chạy/khởi động nó

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman create busybox # tạo
Resolved "busybox" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob sha256:213a27df5921cd9ae24732504c590bb6408911c20fb50a597f2a40896d554a8f
Copying config sha256:3fba0c87fcc8ba126bf99e4ee205b43c91ffc6b15bb052315312e71bc6296551
Writing manifest to image destination
51b613406e8889213c176523e1c430e4bd00047965b0c22cff5b1c9badfbc452

:~$ podman container ls -a
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS      PORTS       NAMES
51b613406e88  docker.io/library/busybox:latest  sh          2 minutes ago  Created                 adoring_almeida
```

> 💡 **Bổ sung:** Container ở trạng thái `Created` cho tới khi bạn chạy `podman start <tên/id>`. `podman run` thực chất bằng `create` + `start`. Cần `-a` để `ls`/`ps` hiển thị cả container chưa chạy hoặc đã dừng.

</p>
</details>

### Export một container ra file output.tar

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman container ls -a # chọn container id
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS      PORTS       NAMES
51b613406e88  docker.io/library/busybox:latest  sh          2 minutes ago  Created                 adoring_almeida

:~$ podman export <container id> --output=output.tar

:~$ ls -al output.tar
-rw-r--r--@ 1 limistah  wheel  4272640 28 Aug 13:48 output.tar
```

> 💡 **Bổ sung:** Phân biệt hai cặp lệnh hay bị nhầm (đề CKAD hay hỏi):
> - `podman export` / `podman import`: xuất **filesystem của container** thành một tar phẳng, mất toàn bộ lịch sử layer và metadata (CMD, ENV...).
> - `podman save` / `podman load`: lưu **image** kèm đủ layer và metadata, ví dụ `podman save -o simpleapp.tar localhost/simpleapp`, rồi `podman load -i simpleapp.tar` ở máy khác. Thêm `--format oci-archive` nếu muốn định dạng OCI.

</p>
</details>

### Chạy Pod với image đã push lên registry

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ kubectl run simpleapp --image=$registry_ip:5000/simpleapp --port=80

:~$ curl $(kubectl get pods simpleapp -o jsonpath='{.status.podIP}')
Hello, World!
```

> Lưu ý: bản gốc viết `${kubectl ...}`, cú pháp này sai (bash báo `bad substitution`). Lệnh trên đã được sửa thành command substitution `$(...)`. `curl` tới IP của Pod chỉ chạy được khi bạn đứng trên một node của cluster (hoặc máy có route tới Pod network).

</p>
</details>

### Đăng nhập vào một remote registry server rồi đọc thông tin xác thực từ file mặc định

<details><summary>Xem lời giải</summary>
<p>

> Lưu ý: hai container registry phổ biến nhất có gói miễn phí là [DockerHub](https://hub.docker.com/) và [Quay.io](https://quay.io/).

```bash
:~$ podman login --username $YOUR_USER --password $YOUR_PWD docker.io
:~$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
{
        "auths": {
                "docker.io": {
                        "auth": "Z2l1bGl0JLSGtvbkxCcX1xb617251xh0x3zaUd4QW45Q3JuV3RDOTc="
                }
        }
}
```

> 💡 **Bổ sung:** Trường `auth` chỉ là `username:password` mã hóa base64, **không phải mã hóa bảo mật**. Truyền mật khẩu bằng `--password` sẽ để lại trong shell history; an toàn hơn là `echo "$YOUR_PWD" | podman login --username $YOUR_USER --password-stdin docker.io`. Trên Linux, file auth mặc định nằm ở `${XDG_RUNTIME_DIR}/containers/auth.json` (thường là `/run/user/<uid>/`, mất khi reboot). Trên macOS/Windows (podman machine) thì nằm ở `~/.config/containers/auth.json`. Có thể chỉ định file khác bằng `--authfile` (hoặc biến môi trường `REGISTRY_AUTH_FILE`).

</p>
</details>

### Tạo Secret từ thông tin đăng nhập có sẵn và từ CLI

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ kubectl create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockerconfigjson
secret/mysecret created
:~$ kubectl create secret docker-registry mysecret2 --docker-server=https://index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
secret/mysecret2 created
```

> Lưu ý: bản gốc ghi sai `--type=kubernetes.io/dockeconfigjson` (thiếu chữ `r`). Với Secret `generic`, kubectl vẫn chấp nhận một type tùy ý nên lệnh không báo lỗi, nhưng kubelet sẽ không dùng Secret đó để pull image. Lệnh trên đã được sửa thành `kubernetes.io/dockerconfigjson`.

> 💡 **Bổ sung:** Cách thứ hai (`kubectl create secret docker-registry`) dễ nhớ hơn và hay dùng khi thi. Kiểm tra nội dung: `kubectl get secret mysecret2 -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d`.

</p>
</details>

### Viết manifest cho một Pod dùng một trong hai Secret vừa tạo để pull image lưu trên private remote registry tương ứng

<details><summary>Xem lời giải</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: $YOUR_PRIVATE_IMAGE
  imagePullSecrets:
  - name: mysecret
```

> 💡 **Bổ sung:** Secret trong `imagePullSecrets` phải nằm **cùng namespace** với Pod. Để mọi Pod trong namespace tự động dùng Secret này mà không phải khai báo từng Pod, gắn nó vào ServiceAccount: `kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "mysecret"}]}'`. Nếu pull lỗi, Pod sẽ ở trạng thái `ErrImagePull`/`ImagePullBackOff`; xem nguyên nhân trong `kubectl describe pod private-reg` (phần Events).

</p>
</details>

### Dọn dẹp toàn bộ image và container

<details><summary>Xem lời giải</summary>
<p>

```bash
:~$ podman rm --all --force
:~$ podman rmi --all
:~$ kubectl delete pod simpleapp
```

> 💡 **Bổ sung:** Có thể dùng `podman system prune -a` để xóa trong một lệnh mọi container đã dừng, pod, network không dùng và image không được container nào dùng (thêm `--volumes` để xóa cả volume).

</p>
</details>
