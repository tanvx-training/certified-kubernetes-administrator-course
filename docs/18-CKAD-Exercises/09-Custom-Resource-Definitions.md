# Mở rộng Kubernetes API bằng CRD (CustomResourceDefinition)

- Lưu ý: CRD nằm trong syllabus CKAD mới. Dưới đây là vài ví dụ về cách thêm custom resource vào Kubernetes API bằng cách tạo một CRD.

> 💡 **Bổ sung:** CRD cũng có trong curriculum **CKA 2025** ("Understand extension interfaces (CNI, CSI, CRI, etc.)" và "Understand CRDs, install and configure operators"). Ý tưởng chính:
> - **CRD** định nghĩa một *loại* resource mới (giống như `kind: Deployment` có sẵn). Sau khi tạo CRD, API server phục vụ thêm endpoint `/apis/<group>/<version>/<plural>`.
> - **Custom Resource (CR)** là một *object* thuộc loại đó.
> - Bản thân CRD chỉ lưu dữ liệu vào etcd, không "làm" gì cả. **Operator** là controller theo dõi các CR rồi thực hiện hành động (tạo Pod, backup DB...). Ví dụ thường gặp: cert-manager, Prometheus Operator, các CRD của Gateway API.

## CRD trong K8s

### Tạo file manifest CustomResourceDefinition cho một Operator với các thông số sau:
* *Name* : `operators.stable.example.com`
* *Group* : `stable.example.com`
* *Schema*: `<email: string><name: string><age: integer>`
* *Scope*: `Namespaced`
* *Names*: `<plural: operators><singular: operator><shortNames: op>`
* *Kind*: `Operator`

<details><summary>Xem lời giải</summary>
<p>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name phải khớp với các trường trong spec bên dưới, theo dạng: <plural>.<group>
  name: operators.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      # Phải có một và chỉ một version được đánh dấu là storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                email:
                  type: string
                name:
                  type: string
                age:
                  type: integer
  scope: Namespaced
  names:
    plural: operators
    singular: operator
    # kind thường là dạng số ít viết CamelCase. Các manifest resource của bạn sẽ dùng giá trị này.
    kind: Operator
    shortNames:
    - op
```

> 💡 **Bổ sung:**
> - Không có lệnh imperative để tạo CRD; copy mẫu từ kubernetes.io > Documentation > Tasks > Extend Kubernetes > Custom Resources > [Extend the Kubernetes API with CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).
> - Từ `apiextensions.k8s.io/v1` (bản `v1beta1` đã bị gỡ ở v1.22), trường `schema.openAPIV3Schema` là **bắt buộc** với mỗi version.
> - `scope` có hai giá trị: `Namespaced` hoặc `Cluster`.
> - Có thể thêm ràng buộc validation như `minimum`, `maximum`, `enum`, `pattern`, `required: [...]` vào schema. Ngoài ra còn có validation rule bằng CEL qua `x-kubernetes-validations` (GA từ v1.29).

</p>
</details>

### Tạo CRD resource trong K8s API

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl apply -f operator-crd.yml
```

> 💡 **Bổ sung:** Kiểm tra CRD đã được đăng ký:
> - `kubectl get crd operators.stable.example.com`
> - `kubectl api-resources --api-group=stable.example.com`: xem tên, shortname, có thuộc namespace hay không, và kind.
> - `kubectl explain operator.spec`: xem schema các trường (cũng rất hữu ích với CRD của operator lạ trong đề thi).

</p>
</details>

### Tạo custom object từ CRD

* *Name* : `operator-sample`
* *Kind*: `Operator`
* Spec:
  * email: `operator-sample@stable.example.com`
  * name: `operator sample`
  * age: `30`

<details><summary>Xem lời giải</summary>
<p>

```yaml
apiVersion: stable.example.com/v1
kind: Operator
metadata:
  name: operator-sample
spec:
  email: operator-sample@stable.example.com
  name: "operator sample"
  age: 30
```

```bash
kubectl apply -f operator.yml
```

> 💡 **Bổ sung:** `apiVersion` của custom object có dạng `<group>/<version>` lấy từ CRD. Nếu ghi sai kiểu dữ liệu (ví dụ `age: "30"` dạng chuỗi), API server sẽ từ chối object nhờ schema đã khai báo. Với trường không có trong schema, `kubectl` hiện tại (mặc định `--validate=strict`) sẽ báo lỗi `unknown field`; nếu bỏ qua validation thì API server âm thầm loại bỏ trường đó (pruning) chứ không lưu vào etcd.

</p>
</details>

### Liệt kê các operator

<details><summary>Xem lời giải</summary>
<p>

Dùng dạng số ít, số nhiều và dạng viết tắt

```bash
kubectl get operators
or
kubectl get operator
or
kubectl get op
```

> 💡 **Bổ sung:** Có thể dùng tên đầy đủ kèm group để tránh trùng tên với resource khác: `kubectl get operators.stable.example.com`. Khi xóa CRD (`kubectl delete crd operators.stable.example.com`), **mọi custom object thuộc loại đó cũng bị xóa theo**.

</p>
</details>
