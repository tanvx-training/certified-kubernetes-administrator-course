# Service Account
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/service-account/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **ServiceAccount** trong Kubernetes, vai trò và cách sử dụng chúng.

### Vì sao cần ServiceAccount trong cluster?

Kubernetes có hai loại "người dùng" chính:

1. **Người dùng (Human Users)**: những người tương tác với cluster thông qua các công cụ như `kubectl` (admin, developer...).
2. **Bot/Ứng dụng**: các dịch vụ như Prometheus, Jenkins và các ứng dụng hệ thống khác.


ServiceAccount chủ yếu được dùng để xác thực (authenticate) các bot hoặc ứng dụng này với kube-apiserver.

> 💡 **Bổ sung:** Kubernetes không có object "User" — user là danh tính được quản lý bên ngoài (certificate, OIDC...). Ngược lại, ServiceAccount là một object thực sự trong Kubernetes, thuộc về một namespace, được tạo bằng `kubectl create serviceaccount <tên>` (viết tắt `sa`).

### Tạo ServiceAccount

![alt text](../../images/sa.png)

> 💡 **Bổ sung:** Các lệnh cơ bản: `kubectl create serviceaccount dashboard-sa`, `kubectl get serviceaccount`, `kubectl describe serviceaccount dashboard-sa`. Sau khi tạo, hãy dùng RBAC (Role/ClusterRole + RoleBinding) để cấp quyền cho ServiceAccount, ví dụ `kubectl create rolebinding dashboard-sa-binding --role=pod-reader --serviceaccount=default:dashboard-sa`. Trong RBAC, danh tính của ServiceAccount có dạng `system:serviceaccount:<namespace>:<tên>`.




### ServiceAccount và Authentication



Khi muốn truy cập Kubernetes API từ bên trong một Pod, bạn cần xác thực Pod đó với kube-apiserver. **ServiceAccount** được dùng cho quá trình xác thực này.

Trước Kubernetes v1.24, khi tạo một ServiceAccount, một Secret chứa token cũng được tự động tạo ra. Token này được dùng như một bearer token để xác thực với kube-apiserver. Mặc định, token được mount vào Pod dưới dạng volume và có thể truy cập bên trong Pod tại đường dẫn: `/var/run/secrets/kubernetes.io/serviceaccount/token`.
 

![alt text](../../images/sa2.png)


Mặc định, mỗi namespace trong Kubernetes đều có một ServiceAccount tên là `default`. Mọi Pod trong namespace đó sẽ tự động được gán ServiceAccount `default` nếu không chỉ định khác. Nếu muốn dùng một ServiceAccount khác cho một Pod, bạn khai báo nó trong spec của Pod.

Dưới đây là ví dụ cách chỉ định ServiceAccount cho Pod trong spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  serviceAccountName: myapp-service-account
  
```

> 💡 **Bổ sung:** Bạn không thể thay đổi ServiceAccount của một Pod đang chạy — phải xóa và tạo lại Pod. Với Deployment thì có thể sửa `serviceAccountName` trong Pod template; Deployment sẽ tự rollout tạo Pod mới.

Để tắt việc tự động mount token của ServiceAccount:

```yaml
spec:
  automountServiceAccountToken: false
```

> 💡 **Bổ sung:** `automountServiceAccountToken: false` có thể đặt ở Pod spec hoặc ngay trên object ServiceAccount; nếu cả hai cùng được đặt, giá trị trong Pod spec được ưu tiên. Nên tắt cho các ứng dụng không cần gọi Kubernetes API để giảm rủi ro bảo mật.
 


### Cải tiến bảo mật (Kubernetes v1.22+)

Các token ServiceAccount kiểu cũ (lưu trong Secret) mặc định không có thời hạn và không giới hạn audience. Từ Kubernetes v1.22 trở đi đã có các cải tiến để tăng cường bảo mật cho token:

- Audience Bound: token chỉ dùng được cho một audience cụ thể (ví dụ kube-apiserver).
- Time Bound: token có thời gian hết hạn (mặc định khi dùng `kubectl create token` là 1 giờ).
- Object Bound: token gắn với các object cụ thể, như Pod hoặc ServiceAccount — khi Pod bị xóa, token cũng mất hiệu lực.

Nhờ đó, bảo mật được nâng cao bằng cách giới hạn phạm vi hiệu lực và vòng đời của token.

Như vậy, TokenRequest API (đã có từ v1.12 và trở thành cơ chế mặc định cấp token cho Pod từ v1.22) được dùng để yêu cầu token với audience và thời hạn cụ thể. API này được dùng để lấy token thay cho việc dùng các Secret chứa token ServiceAccount. Token lấy từ TokenRequest API an toàn hơn token lưu trong Secret vì có thời hạn giới hạn và không thể bị các API client khác đọc.

Cũng từ phiên bản này, token của ServiceAccount được đưa vào Pod dưới dạng projected volume như hình dưới. Kubelet sẽ tự động làm mới (rotate) token trước khi hết hạn.

 

![alt text](../../images/sa3.png)

 

Trong Kubernetes, Secret có type `kubernetes.io/service-account-token` được dùng để lưu thông tin token (token credential) định danh một ServiceAccount.
Từ v1.22, loại Secret này không còn được dùng để mount thông tin xác thực vào Pod nữa, và Kubernetes khuyến nghị lấy token qua TokenRequest API thay vì dùng Secret chứa token ServiceAccount. Token lấy từ TokenRequest API an toàn hơn token lưu trong Secret vì có thời hạn giới hạn và không thể bị các API client khác đọc. Bạn có thể dùng lệnh `kubectl create token` để lấy token từ TokenRequest API.

### Tạo token cho ServiceAccount
Từ Kubernetes v1.24, KEP-2799 (LegacyServiceAccountTokenNoAutoGeneration) đã loại bỏ việc tự động tạo Secret chứa token cho ServiceAccount. Giờ đây, khi cần token, bạn phải tạo thủ công bằng lệnh `kubectl create token`:

```bash
 kubectl create token <serviceaccount>
```

![alt text](../../images/sa4.png)

> 💡 **Bổ sung:**
> - `kubectl create token` **không tạo Secret** nào cả; nó gọi TokenRequest API và in token ra màn hình. Mặc định token có hiệu lực 1 giờ; có thể đổi bằng `--duration`, ví dụ `kubectl create token dashboard-sa --duration=24h` (server có thể giới hạn thời hạn tối đa). Chỉ định namespace bằng `-n`.
> - Vì thế, từ v1.24 lệnh `kubectl describe serviceaccount <tên>` sẽ hiển thị `Tokens: <none>`.
> - Token là JWT, có thể giải mã phần payload (ví dụ trên jwt.io) để xem `aud`, `exp`, `sub` (`system:serviceaccount:<ns>:<tên>`).
> - Từ v1.29, các token kiểu Secret cũ (legacy) không được sử dụng sẽ bị đánh dấu và tự động dọn dẹp (KEP-2799, LegacyServiceAccountTokenCleanUp).


Bạn chỉ nên tạo Secret chứa token ServiceAccount khi không thể dùng TokenRequest API để lấy token, và bạn chấp nhận được rủi ro bảo mật khi lưu một token không hết hạn trong một API object có thể đọc được. Khi đó:

- Tạo một ServiceAccount.
- Tạo một Secret cho ServiceAccount đó (type `kubernetes.io/service-account-token`).
- Trong Secret, thêm annotation `kubernetes.io/service-account.name: <serviceaccountname>`. Token controller sẽ tự động điền token vào Secret này.

 

![alt text](../../images/sa5.png)

 
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
 
  
  
