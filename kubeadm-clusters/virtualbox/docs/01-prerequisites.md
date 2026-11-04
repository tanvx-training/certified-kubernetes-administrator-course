# Điều kiện tiên quyết

Nếu bạn dùng Mac chip M-series (Apple Silicon) thì không thể chạy VirtualBox. Hãy xem hướng dẫn [Apple Silicon](../../apple-silicon/) thay thế.

## Yêu cầu phần cứng cho VM

* 8 GB RAM (khuyến nghị 16 GB)
* CPU 8 nhân, hoặc 4 nhân có hyperthreading trở lên, ví dụ Core-i7/Core-i9 (nếu yếu hơn sẽ chạy chậm)
* 50 GB dung lượng ổ đĩa

## git

Cần git để tải repo về. Trên Mac git thường được cài sẵn, còn Windows thì không. Nếu cần cài, xem [tại đây](https://git-scm.com/download).

## VirtualBox

Tải và cài [VirtualBox](https://www.virtualbox.org/wiki/Downloads) trên một trong các nền tảng được hỗ trợ:

 - Máy host Windows
 - Máy host MacOS, chỉ x86. Với Apple Silicon (bộ xử lý M-series), xem [tại đây](../../apple-silicon/).
 - Các bản phân phối Linux

Lab này được kiểm thử lần cuối với VirtualBox 7.0.12, các phiên bản mới hơn cũng sẽ chạy được.

## Vagrant

Sau khi cài VirtualBox, bạn có thể tự tạo máy ảo thủ công trên đó.
Tuy nhiên, Vagrant giúp triển khai nhiều máy ảo trên VirtualBox dễ dàng và nhất quán hơn.

Tải và cài [Vagrant](https://www.vagrantup.com/) cho nền tảng của bạn.

Lab này được kiểm thử lần cuối với Vagrant 2.3.7, các phiên bản mới hơn cũng sẽ chạy được.

> 💡 **Bổ sung:** Vagrant đọc file [Vagrantfile](../Vagrantfile) trong thư mục `virtualbox` để biết cần tạo bao nhiêu VM, cấu hình CPU/RAM, mạng và các script provisioning chạy khi VM khởi động lần đầu. Nhờ vậy cả lớp đều có một môi trường giống hệt nhau chỉ với `vagrant up`.

## Giá trị mặc định của lab

Các lab đã được cấu hình với các giá trị mạng mặc định dưới đây. Nếu bạn thay đổi bất kỳ giá trị nào sau khi đã triển khai một phần lab, bạn sẽ phải reset hoàn toàn và làm lại từ đầu:

```bash
vagrant destroy -f
vagrant up
```

Nếu bạn có thay đổi các giá trị này, **hãy coi đó là tùy chọn cá nhân và đừng gửi PR cho thay đổi đó**.

### Mạng của máy ảo

Do cách VirtualBox/Vagrant hoạt động, mỗi VM cần hai network adapter: một adapter NAT (`enp0s3`) để giao tiếp với bên ngoài, và một adapter nội bộ (`enp0s8`) gắn vào mạng VirtualBox nói trên. Mặc định, các thành phần Kubernetes sẽ kết nối qua network adapter mặc định — tức adapter NAT — và đó *không* phải điều chúng ta muốn. Vì vậy, chúng tôi đã đặt sẵn biến môi trường `PRIMARY_IP` trên tất cả các VM, chứa địa chỉ IP mà các thành phần Kubernetes nên sử dụng. Trong các lab tiếp theo, bạn sẽ thấy biến này được dùng để đảm bảo các thành phần Kubernetes bind vào đúng network interface.

`PRIMARY_IP` được định nghĩa là địa chỉ IP của network interface trên node nối với mạng có default gateway, và đó là interface mà node dùng để nói chuyện với các node khác. Nếu bạn tò mò, biến này được gán bằng kết quả của lệnh sau

```bash
ip route | grep default | awk '{ print $9 }'
```

> 💡 **Bổ sung:** Vì sao điều này quan trọng? Với adapter NAT của VirtualBox, *mọi* VM đều nhận cùng một IP (thường là `10.0.2.15`), nên nếu kube-apiserver quảng bá (advertise) IP này hoặc kubelet đăng ký node với IP này thì các node không thể liên lạc với nhau. Đó là lý do khi chạy `kubeadm init` ta sẽ truyền `--apiserver-advertise-address` và cấu hình `--node-ip` cho kubelet bằng `PRIMARY_IP`. Bạn có thể kiểm tra IP mà node đang dùng bằng `kubectl get nodes -o wide` (cột `INTERNAL-IP`).

#### Bridge Networking

Cấu hình mặc định của lab là khởi động VM trên các bridged interface. Nghĩa là các node Kubernetes sẽ xuất hiện như những máy bổ sung trong mạng nội bộ (LAN) của bạn, với địa chỉ IP do router mạng nhà bạn cấp phát động (DHCP). Điều này giúp bạn dùng trình duyệt để truy cập bất kỳ Service NodePort nào bạn triển khai.

Nếu gặp vấn đề khi triển khai bridge networking, hãy tạo [bug report](https://github.com/kodekloudhub/certified-kubernetes-administrator-course/issues) kèm đầy đủ thông tin, bao gồm output của `vagrant up`.

Sau đó thử lại lab ở chế độ NAT. Cách làm được trình bày ở [phần tiếp theo](./02-compute-resources.md).

#### NAT Networking

Trong cấu hình NAT, mạng mà các VM chạy trên đó bị tách biệt khỏi mạng của router nhà bạn bởi một NAT gateway do hypervisor quản lý. Nghĩa là các VM có thể nhìn ra ngoài (và kết nối Internet), nhưng bạn không thể nhìn vào trong (tức là dùng trình duyệt để kết nối tới NodePort) nếu không tự thiết lập quy tắc port forwarding cho từng NodePort trong giao diện VirtualBox.

Mạng mà các máy ảo VirtualBox sử dụng là `192.168.56.0/24`.

Để thay đổi, hãy sửa [Vagrantfile](../Vagrantfile) trong bản clone của bạn (không sửa trực tiếp trên github) và đặt giá trị mới cho network prefix ở dòng 9. Mạng này không được chồng lấn với bất kỳ thiết lập mạng nào khác.

Lưu ý bạn không cần sửa các script khác để thực hiện thay đổi trên. Mọi thứ đều được tính toán bằng biến shell dựa trên địa chỉ IP gán cho VM và các giá trị trong file hosts (cũng được tính toán tự động).

*Khuyến nghị* giữ nguyên mạng Pod và mạng Service ở giá trị mặc định. Nếu thay đổi, bạn sẽ phải sửa cả manifest của CNI cho phù hợp.

> 💡 **Bổ sung:** Bản gốc nhắc tới manifest mạng Weave, nhưng hướng dẫn hiện tại (xem [05-controlplane](../../generic/05-controlplane.md)) dùng **Calico**. Weave Net đã ngừng được phát triển sau khi Weaveworks đóng cửa (2024), nên không còn được khuyến nghị.

Nếu bạn quyết định thay đổi bất kỳ giá trị nào, hãy coi đó là tùy chọn cá nhân và đừng tạo pull request.


### Mạng Pod

Mạng dùng để cấp phát địa chỉ IP cho các Pod là `10.244.0.0/16`.

Để thay đổi, mở tất cả các file `.md` trong thư mục [docs](../docs/) bằng IDE yêu thích và thay thế toàn cục (global replace)<br>
`POD_CIDR=10.244.0.0/16`<br>
bằng dải CIDR mới. Dải này không được chồng lấn với bất kỳ thiết lập mạng nào khác.

### Mạng Service

Mạng dùng để cấp phát địa chỉ IP cho các Service Cluster IP là `10.96.0.0/16`.

Để thay đổi, mở tất cả các file `.md` trong thư mục [docs](../docs/) bằng IDE yêu thích và thay thế toàn cục<br>
`SERVICE_CIDR=10.96.0.0/16`<br>
bằng dải CIDR mới. Dải này không được chồng lấn với bất kỳ thiết lập mạng nào khác.

> 💡 **Bổ sung:** Có ba dải mạng cần phân biệt và **không được chồng lấn**: mạng của node (LAN/VirtualBox, vd `192.168.56.0/24` hoặc mạng nhà bạn), mạng Pod (`--pod-network-cidr`, do CNI cấp IP cho Pod) và mạng Service (`--service-cidr`, IP ảo cho ClusterIP do kube-proxy xử lý). Nếu chồng lấn, traffic sẽ bị định tuyến sai và cluster hoạt động thất thường. Lưu ý các lệnh thực tế nằm trong các file ở thư mục [generic](../../generic/), nên khi đổi CIDR hãy tìm-thay thế cả ở đó.

## Chạy lệnh song song với tmux

Nếu bạn làm tutorial này trên Mac x86, có thể dùng iterm2 thay thế. Xem phần thiết lập iterm2 trong [hướng dẫn Apple Silicon](../../apple-silicon/docs/01-prerequisites.md#running-commands-in-parallel-with-iterm2).

[tmux](https://github.com/tmux/tmux/wiki) có thể dùng để chạy cùng một lệnh trên nhiều compute instance cùng lúc. Các lab trong tutorial này có thể yêu cầu chạy cùng một lệnh trên nhiều compute instance. Khi đó bạn có thể cân nhắc dùng tmux, chia cửa sổ thành nhiều pane và bật synchronize-panes để tăng tốc quá trình provisioning.

Để dùng tmux, trước tiên bạn phải kết nối vào `controlplane` và chạy tmux tại đó. Bên trong phiên tmux, bạn có thể mở nhiều pane và ssh tới các worker node từ các pane này.

*Việc dùng tmux là tùy chọn, không bắt buộc để hoàn thành tutorial này*.

![tmux screenshot](../../../images/tmux-screenshot.png)

> Để bật synchronize-panes, nhấn `CTRL+B` rồi `"` để chia cửa sổ thành hai pane. Trong mỗi pane (chọn bằng chuột), ssh tới host bạn sẽ làm việc.</br>Tiếp theo gõ `CTRL+X` tại dấu nhắc để bắt đầu đồng bộ. Ở chế độ đồng bộ, đường phân cách giữa các pane sẽ có màu đỏ. Mọi thứ bạn gõ hoặc dán vào một pane sẽ được lặp lại ở pane kia.<br>Để tắt đồng bộ, gõ lại `CTRL+X`.</br></br>Lưu ý phím tắt `CTRL-X` được cung cấp bởi file `.tmux.conf` mà vagrant provisioner nạp sẵn vào VM.<br/>Để dán lệnh vào một pane tmux, dùng `SHIFT-RightMouseButton` (Shift + chuột phải).

Tiếp theo: [Compute Resources](02-compute-resources.md)
