# Kết nối tới Cluster

Chúng ta sẽ kết nối tới cluster bằng Azure CloudShell.

![image](../images/09-connect.png)

Khi bạn bấm vào **Connect**, một cửa sổ sẽ bật ra ở bên phải. Bấm vào link **Open Cloud Shell**

1. Tại popup mở ra, chọn `Bash`
1. Tại popup tiếp theo, chọn như sau
    * Mount storage account
    * Subscription - chọn từ danh sách thả xuống. Chỉ nên có một lựa chọn, đại loại như `azurekmlprod`
    * Bấm `Apply`
1. Tại popup `Mount Storage Account`, chọn `I want to create a storage account` và bấm `Next`
1. Bây giờ chọn như sau trong popup `Create storage account`
    * `Subscription` - Không thay đổi trường này.
    * `Resource gorup` - chọn từ danh sách thả xuống. Chỉ nên có một lựa chọn.
    * `Storage account name` - Tên bất kỳ bạn thích hoặc các ký tự ngẫu nhiên gõ từ bàn phím. Ít nhất 16 ký tự.
    * `File share` - Tên bất kỳ bạn thích hoặc các ký tự ngẫu nhiên gõ từ bàn phím. Ít nhất 16 ký tự.
    * `Region` - Chọn `(US) East US` từ danh sách thả xuống
    * Bấm `Create`, chờ triển khai xong, rồi chờ dấu nhắc lệnh bash xuất hiện.

Cửa sổ cloud shell sẽ mở ở phía dưới cửa sổ trình duyệt và sẽ tự động chạy hai lệnh cần thiết để thiết lập kết nối tới cluster.

![image](../images/09c-cloudshell.png)

> 💡 **Bổ sung:** Hai lệnh đó thường là `az account set --subscription <subscription-id>` (chọn subscription) và `az aks get-credentials --resource-group <resource-group> --name kodekloud-demo` (tải thông tin xác thực của cluster và ghép vào file `~/.kube/config`). Nếu chúng không tự chạy, bạn có thể tự chạy lại - lấy lệnh chính xác từ cửa sổ **Connect**.

Bây giờ bạn có thể chạy lệnh với cluster, và thế là xong!

```text
odl_user [ ~ ]$ kubectl get nodes
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-33115789-vmss000000   Ready    agent   27m   v1.28.5
aks-agentpool-33115789-vmss000001   Ready    agent   26m   v1.28.5
odl_user [ ~ ]$ kubectl run nginx --image=nginx
pod/nginx created
odl_user [ ~ ]$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s
odl_user [ ~ ]$ 
```

> 💡 **Bổ sung:** Lưu ý `kubectl get nodes` chỉ hiển thị các worker node - control plane do Azure quản lý nên bạn sẽ không thấy master node nào. Khi thực hành xong trên tài khoản Azure thật, hãy xóa cluster (ví dụ `az aks delete --resource-group <resource-group> --name kodekloud-demo`) để tránh phát sinh chi phí.
