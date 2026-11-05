# Quản lý Kubernetes với Helm

- Lưu ý: Helm nằm trong syllabus CKAD mới. Dưới đây là vài ví dụ dùng Helm để quản lý Kubernetes.

> 💡 **Bổ sung:** Helm cũng có trong curriculum **CKA 2025** (phần "Use Helm and Kustomize to install cluster components"). Vài khái niệm cần nắm:
> - **Chart**: gói chứa các template manifest cùng file `values.yaml` (giá trị mặc định).
> - **Release**: một lần cài đặt chart vào cluster, có tên riêng và thuộc một namespace. Có thể cài cùng một chart nhiều lần với tên release khác nhau.
> - **Repository**: nơi lưu và phân phối chart (HTTP repo hoặc OCI registry).
> - **Revision**: mỗi lần `install`/`upgrade`/`rollback` tạo ra một revision mới, lưu dưới dạng Secret trong namespace của release.
>
> Helm 3 không còn Tiller; quyền thao tác chính là quyền trong kubeconfig hiện tại. Khi thi, chạy `helm <lệnh> --help` để xem cú pháp nhanh.

## Helm trong K8s

### Tạo một Helm chart cơ bản

<details><summary>Xem lời giải</summary>
<p>

```bash
helm create chart-test ## lệnh này tạo khung một Helm chart
```

> 💡 **Bổ sung:** `helm create` sinh ra thư mục `chart-test/` gồm `Chart.yaml` (metadata, version), `values.yaml` (giá trị mặc định), `templates/` (các template manifest: deployment, service, ingress...) và `charts/` (chart phụ thuộc). Xem trước manifest sẽ được render mà không cài: `helm template chart-test ./chart-test`. Kiểm tra lỗi cú pháp: `helm lint ./chart-test`.

</p>
</details>

### Chạy (cài đặt) một Helm chart

<details><summary>Xem lời giải</summary>
<p>

```bash
helm install -f myvalues.yaml myredis ./redis
```

> 💡 **Bổ sung:** Cú pháp là `helm install <tên-release> <chart> [flags]`. Các flag hay dùng:
> - `-n <namespace> --create-namespace`: cài vào namespace chỉ định, tự tạo namespace nếu chưa có.
> - `--set key=value`: ghi đè từng giá trị; độ ưu tiên cao hơn `-f`.
> - `--version <x.y.z>`: chọn version chart cụ thể (đề thi hay yêu cầu điểm này).
> - `--dry-run`: thử trước mà không cài thật.
>
> Sau khi cài, kiểm tra bằng `helm list -n <namespace>` và `helm status <release>`.

</p>
</details>

### Tìm các Helm deployment đang ở trạng thái pending trên tất cả namespace

<details><summary>Xem lời giải</summary>
<p>

```bash
helm list --pending -A
```

> 💡 **Bổ sung:** `-A` là viết tắt của `--all-namespaces`. Mặc định `helm list` chỉ hiển thị release đang `deployed` hoặc `failed` trong namespace hiện tại. Ngoài `--pending` còn có `--failed`, `--uninstalled`, `--superseded`, hoặc `-a/--all` để xem mọi trạng thái. Release kẹt ở `pending-install`/`pending-upgrade` sẽ chặn lần upgrade tiếp theo. Xử lý bằng `helm rollback` về revision trước hoặc `helm uninstall`.

</p>
</details>

### Gỡ cài đặt một Helm release

<details><summary>Xem lời giải</summary>
<p>

```bash
helm uninstall -n namespace release_name
```

</p>
</details>

### Upgrade một Helm chart

<details><summary>Xem lời giải</summary>
<p>

```bash
helm upgrade -f myvalues.yaml -f override.yaml redis ./redis
```

> 💡 **Bổ sung:**
> - Khi truyền nhiều `-f`, file đứng **sau** có độ ưu tiên cao hơn (ở đây `override.yaml` ghi đè `myvalues.yaml`).
> - `helm upgrade --install` cài mới nếu release chưa tồn tại, upgrade nếu đã có.
> - Mặc định `upgrade` **không giữ** các giá trị `--set` của lần trước; dùng `--reuse-values` nếu muốn giữ lại.
> - Xem lịch sử và quay lại phiên bản cũ: `helm history redis`, rồi `helm rollback redis <revision>`.
> - Xem các giá trị đang áp dụng cho release: `helm get values redis` (thêm `--all` để xem cả giá trị mặc định).

</p>
</details>

### Làm việc với Helm repo

<details><summary>Xem lời giải</summary>
<p>

Thêm, liệt kê, xóa, cập nhật và tạo index cho chart repo

```bash
helm repo add [NAME] [URL]  [flags]

helm repo list / helm repo ls

helm repo remove [REPO1] [flags]

helm repo update / helm repo up

helm repo update [REPO1] [flags]

helm repo index [DIR] [flags]
```

> 💡 **Bổ sung:** Tìm chart trong các repo đã thêm: `helm search repo <từ-khóa>`. Thêm `--versions` để liệt kê mọi version, rất tiện khi đề yêu cầu cài một version cụ thể. Tìm trên Artifact Hub: `helm search hub <từ-khóa>`. Nhớ chạy `helm repo update` sau khi thêm repo để có index mới nhất.

</p>
</details>

### Tải một Helm chart từ repository về

<details><summary>Xem lời giải</summary>
<p>

```bash
helm pull [chart URL | repo/chartname] [...] [flags] ## lệnh này tải chart về, không cài đặt
helm pull --untar [rep/chartname] # giải nén chart sau khi tải về
```

> 💡 **Bổ sung:** Ngoài repo HTTP, Helm 3.8+ hỗ trợ OCI registry chính thức, ví dụ `helm pull oci://registry-1.docker.io/bitnamicharts/nginx --version <x.y.z>`. Với OCI không cần `helm repo add`; chart được tham chiếu thẳng bằng URL `oci://`.

</p>
</details>

### Thêm Bitnami repo tại https://charts.bitnami.com/bitnami vào Helm
<details><summary>Xem lời giải</summary>
<p>
    
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

> 💡 **Bổ sung:** Từ năm 2025, Bitnami đổi chính sách phân phối. Chart chủ yếu được phát hành dạng OCI tại `oci://registry-1.docker.io/bitnamicharts`, và phần lớn image miễn phí bị chuyển sang `bitnamilegacy`, không còn được cập nhật. Vì vậy các lệnh trong file này có thể lỗi hoặc cài ra Pod không pull được image. Khi tự luyện tập, bạn có thể thay bằng một repo khác; cú pháp lệnh Helm vẫn giữ nguyên.
  
</p>
</details>

### In nội dung file values.yaml của chart `bitnami/node` ra standard output
<details><summary>Xem lời giải</summary>
<p>
    
```bash
helm show values bitnami/node
```

> 💡 **Bổ sung:** Các lệnh `helm show` khác: `helm show chart` (Chart.yaml), `helm show readme`, `helm show all`. Mẹo: `helm show values bitnami/node > values.yaml`, sửa file đó rồi cài bằng `-f values.yaml`.
  
</p>
</details>

### Cài chart `bitnami/node` với số replica là 5
<details><summary>Xem lời giải</summary>
<p>

Để làm được, ta cần hai thông tin chính:
- Tên thuộc tính trong values.yaml điều khiển số replica
- Một cách đơn giản để đặt giá trị cho thuộc tính đó khi cài đặt

Để tìm tên thuộc tính trong file values.yaml, ta lấy toàn bộ values như ở bài trước, rồi dùng grep để lọc các thuộc tính khớp với mẫu `replica`
```bash
helm show values bitnami/node | grep -i replica
```
kết quả trả về
```bash
## @param replicaCount Specify the number of replicas for the application
replicaCount: 1
```
 
Ta có thể dùng tham số `--set` khi cài đặt để ghi đè giá trị thuộc tính. Vì vậy, để đặt số replica là 5, ta chạy
```bash
helm install mynode bitnami/node --set replicaCount=5
```

> 💡 **Bổ sung:** Với giá trị lồng nhau thì dùng dấu chấm, ví dụ `--set image.tag=1.2.3` hoặc `--set service.type=NodePort`. Có thể truyền nhiều giá trị cách nhau bằng dấu phẩy: `--set a=1,b=2`. Kiểm tra lại kết quả: `kubectl get deploy` (số `READY` phải là `5/5`) hoặc `helm get values mynode`.

</p>
</details>

