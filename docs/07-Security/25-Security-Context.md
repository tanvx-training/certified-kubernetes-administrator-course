# Security Context
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/security-contexts-2/)
  
Trong phần này, chúng ta sẽ tìm hiểu về security context.

## Bảo mật container (Container Security)
- Khi chạy container bằng Docker, ta có thể chỉ định user ID dùng để chạy tiến trình trong container, cũng như thêm/bớt các Linux capability:
 ```
 $ docker run --user=1001 ubuntu sleep 3600
 $ docker run -cap-add MAC_ADMIN ubuntu
 ```

 > Lưu ý: flag đúng của Docker là `--cap-add` (hai dấu gạch), ngoài ra còn có `--cap-drop` và `--privileged`.
 
 ![csec](../../images/csec.PNG)
 
## Bảo mật trong Kubernetes
- Các thiết lập này cũng có thể cấu hình trong Kubernetes. Bạn có thể chọn cấu hình security ở cấp container hoặc cấp Pod. Cấu hình ở cấp Pod sẽ áp dụng cho tất cả container trong Pod. Nếu cấu hình ở cả hai cấp, thiết lập ở cấp container sẽ ghi đè thiết lập ở cấp Pod.

 ![ksec](../../images/ksec.PNG)

## Security Context
- Để cấu hình security context cho Pod, thêm trường **`securityContext`** dưới phần `spec`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    securityContext:
      runAsUser: 1000
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
  ```
  ![sxc1](../../images/sxc1.PNG)
  
- Để đặt cùng thiết lập đó ở cấp container, chuyển toàn bộ phần này vào bên trong phần định nghĩa container:
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
  ```
  ![sxc2](../../images/sxc2.PNG)
  
- Để thêm capability, dùng tùy chọn **`capabilities`**:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities: 
          add: ["MAC_ADMIN"]
  ```
  ![cap](../../images/cap.PNG)

> 💡 **Bổ sung:**
> - **`capabilities` chỉ được hỗ trợ ở cấp container**, không đặt được ở cấp Pod (`spec.securityContext`).
> - Một số trường hay gặp khác: `runAsGroup`, `runAsNonRoot: true` (kubelet từ chối chạy container nếu image chạy bằng root), `fsGroup` (chỉ ở cấp Pod — GID sở hữu volume), `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, `privileged: true`, `capabilities.drop: ["ALL"]`, `seccompProfile`.
> - Kiểm tra nhanh: `kubectl exec web-pod -- id` hoặc `kubectl exec web-pod -- whoami`.
> - Phần lớn các trường trong securityContext của Pod đang chạy không sửa trực tiếp được. Trong lab/kỳ thi, hãy `kubectl get pod <tên> -o yaml > pod.yaml`, sửa file rồi `kubectl replace --force -f pod.yaml` (xóa và tạo lại Pod).
  
  
### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
