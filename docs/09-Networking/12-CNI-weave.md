# CNI Weave
  
  - Đi tới [Lecture](https://kodekloud.com/topic/cni-weave/)

Trong phần này, chúng ta sẽ tìm hiểu "CNI Weave trong Kubernetes Cluster"

> ⚠️ **Lưu ý (cập nhật 2025+):** **Weave Net đã ngừng phát triển.** Công ty Weaveworks đóng cửa vào đầu năm 2024, dự án Weave Net không còn được bảo trì (phiên bản cuối là 2.8.x), và trang `weave.works` cùng endpoint cài đặt `cloud.weave.works` không còn hoạt động. Bài học vẫn hữu ích để hiểu cách một CNI overlay làm việc, nhưng với cluster mới và khi ôn thi, bạn nên làm quen với các CNI đang được duy trì như **Calico**, **Cilium** hoặc **Flannel**.

> 💡 **Bổ sung — Weave hoạt động như thế nào:** Weave triển khai một **agent (peer)** trên mỗi node (dưới dạng DaemonSet). Các agent kết nối với nhau thành một mạng mesh và trao đổi thông tin về node, mạng và Pod, nên mỗi agent biết Pod nào nằm ở node nào. Weave tạo một bridge tên `weave` trên mỗi node và cấp IP cho Pod. Khi một Pod gửi gói tin tới Pod ở node khác, agent Weave chặn gói tin, đóng gói (encapsulate) nó vào một gói tin mới với địa chỉ nguồn/đích là IP của các node, gửi qua mạng vật lý; agent ở node đích mở gói (decapsulate) và chuyển tới đúng Pod. Đây là mô hình **overlay network**. Mặc định Weave dùng dải IP `10.32.0.0/12`.

## Triển khai Weave

- Cài [weave net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) vào Kubernetes cluster chỉ với một lệnh.

```
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

> 💡 **Bổ sung:** URL `cloud.weave.works` ở trên không còn hoạt động. Nếu vẫn cần cài Weave (ví dụ trong lab), dùng manifest từ GitHub release: `kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml`. Nếu cluster dùng Pod CIDR khác dải mặc định của Weave, cần đặt biến môi trường `IPALLOC_RANGE` cho container `weave` trong DaemonSet sao cho khớp với `--cluster-cidr`/`--pod-network-cidr`.
>
> Cách cài các CNI phổ biến hiện nay (luôn kiểm tra phiên bản mới nhất trong tài liệu chính thức):
> - **Flannel:** `kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml` (mặc định dùng Pod CIDR `10.244.0.0/16`, nên khởi tạo cluster bằng `kubeadm init --pod-network-cidr=10.244.0.0/16`).
> - **Calico:** cài Tigera operator rồi tạo custom resource `Installation` theo hướng dẫn tại `docs.tigera.io` (mặc định Pod CIDR `192.168.0.0/16`, có thể sửa trong `custom-resources.yaml`).
> - **Cilium:** thường cài bằng `cilium` CLI (`cilium install`) hoặc Helm chart.

## Các Weave Peer

```
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS             RESTARTS   AGE
coredns-66bff467f8-894jf                  1/1     Running            0          52m
coredns-66bff467f8-nck5f                  1/1     Running            0          52m
etcd-controlplane                         1/1     Running            0          52m
kube-apiserver-controlplane               1/1     Running            0          52m
kube-controller-manager-controlplane      1/1     Running            0          52m
kube-keepalived-vip-mbr7d                 1/1     Running            0          52m
kube-proxy-p2mld                          1/1     Running            0          52m
kube-proxy-vjcwp                          1/1     Running            0          52m
kube-scheduler-controlplane               1/1     Running            0          52m
weave-net-jgr8x                           2/2     Running            0          45m
weave-net-tb9tz                           2/2     Running            0          45m
```

> 💡 **Bổ sung:** Có một Pod `weave-net` trên mỗi node (do là DaemonSet), mỗi Pod có 2 container: `weave` và `weave-npc` (Network Policy Controller — Weave có hỗ trợ Network Policy). Dùng `kubectl get pods -n kube-system -o wide` để thấy Pod nào chạy trên node nào.

## Xem log của các Pod Weave

```
$ kubectl logs weave-net-tb9tz weave -n kube-system 
```

## Xem default route trong Pod

```
$ kubectl run test --image=busybox --command -- sleep 4500
pod/test created

$ kubectl exec test -- ip route
default via 10.244.1.1 dev eth0
```


#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/cluster-administration/addons/
- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
