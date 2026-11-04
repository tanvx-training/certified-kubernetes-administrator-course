# KubeConfig 
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kubeconfig/)

Trong phần này, chúng ta sẽ tìm hiểu về kubeconfig trong Kubernetes.


#### Client dùng file certificate và key để gọi Kubernetes REST API lấy danh sách Pod bằng curl.
- Bạn cũng có thể chỉ định các thông tin tương tự với kubectl

  ![kc1](../../images/kc1.PNG)
  
- Gõ các tùy chọn này mỗi lần chạy lệnh rất bất tiện, nên ta chuyển chúng vào một file cấu hình gọi là kubeconfig, rồi chỉ định file này qua tùy chọn `--kubeconfig` của lệnh.
  ```
  $ kubectl get pods --kubeconfig config
  ```

- Mặc định kubectl tìm file `$HOME/.kube/config`, nên nếu đặt cấu hình ở đó thì không cần chỉ định `--kubeconfig`. Cũng có thể chỉ định qua biến môi trường `KUBECONFIG`.
  
## File kubeconfig
- File kubeconfig gồm 3 phần:
  - Clusters (các cluster cần truy cập: địa chỉ server và CA)
  - Contexts (liên kết giữa một user và một cluster, vd `admin@production`)
  - Users (các tài khoản user cùng thông tin xác thực: certificate/key, token...)
  
  ![kc4](../../images/kc4.PNG)
  
  ![kc5](../../images/kc5.PNG)
  
- Xem file cấu hình đang được sử dụng
  ```
  $ kubectl config view
  ```
- Có thể chỉ định file kubeconfig cho lệnh kubectl config view bằng flag "--kubeconfig"
  ```
  $ kubectl config veiw --kubeconfig=my-custom-config
  ```
  (Lưu ý: bản gốc gõ nhầm `veiw`, lệnh đúng là `kubectl config view`.)
  
  ![kc6](../../images/kc6.PNG)
  
- Làm sao để thay đổi current context (context hiện tại)?
  ```
  $ kubectl config use-context <context-name>
  ex: 
  $ kubectl config use-context prod-user@production
  ```
  
  ![kc7](../../images/kc7.PNG)
  
- Xem trợ giúp của kubectl config
  ```
  $ kubectl config -h
  ```
  
  ![kc8](../../images/kc8.PNG)

> 💡 **Bổ sung — các lệnh `kubectl config` hay dùng khi thi:**
> - `kubectl config get-contexts` — liệt kê các context, dấu `*` đánh dấu context hiện tại.
> - `kubectl config current-context` — xem context hiện tại.
> - `kubectl config set-context --current --namespace=<ns>` — đổi namespace mặc định của context hiện tại.
> - `kubectl config set-credentials`, `kubectl config set-cluster`, `kubectl config set-context` — thêm user/cluster/context mà không cần sửa file bằng tay.
>
> Trong kỳ thi CKA, mỗi câu hỏi thường yêu cầu chạy một lệnh `kubectl config use-context ...` (hoặc SSH sang một host khác) trước khi làm. Luôn chạy lệnh đó để tránh làm đúng nhưng trên sai cluster.
  
## Còn namespace thì sao?

  ![kc9](../../images/kc9.PNG)

- Mỗi context có thể có thêm trường `namespace`; khi chuyển sang context đó, kubectl sẽ tự động làm việc trong namespace này.
 
## Certificate trong kubeconfig

  ![kc10](../../images/kc10.PNG)
 
  ![kc12](../../images/kc12.PNG)
  
  ![kc11](../../images/kc11.PNG)

- Nên dùng đường dẫn đầy đủ (absolute path) tới file certificate. Ngoài ra, có thể nhúng trực tiếp nội dung certificate (đã mã hóa base64) bằng các trường `certificate-authority-data`, `client-certificate-data`, `client-key-data` thay cho `certificate-authority`, `client-certificate`, `client-key`.
 
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config
