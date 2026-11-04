# Điều kiện tiên quyết

* Máy dùng chip Apple Silicon (M1/M2/M3 v.v.)
* 8GB RAM (khuyến nghị 16GB).
    * Mọi cấu hình - Sẽ tạo một node control plane - `controlplane`
    * Nếu máy có ít hơn 16GB, chỉ tạo một worker node - `node01`
    * Nếu máy có từ 16GB trở lên, sẽ tạo hai worker - `node01` và `node02`

Bạn cần cài các công cụ sau trước.

* Multipass - https://multipass.run/install. Làm theo hướng dẫn để cài đặt và kiểm tra nó hoạt động bình thường. Bạn phải tạo được một VM Ubuntu thử nghiệm theo hướng dẫn của họ. Xóa VM thử nghiệm khi xong.
* JQ - https://github.com/stedolan/jq/wiki/Installation#macos

> 💡 **Bổ sung:** `jq` là công cụ xử lý JSON trên dòng lệnh; script tạo VM dùng nó để đọc output JSON của `multipass` (vd lấy IP của từng VM). Nếu đã có Homebrew, có thể cài bằng `brew install jq`.

Ngoài ra

* Tài khoản của bạn trên Mac phải có quyền admin và dùng được `sudo`
* Clone repo này về Mac. Mở ứng dụng terminal của Mac. Mọi lệnh trong hướng dẫn này đều được chạy từ terminal.

    ```bash
    mkdir ~/kodekloud
    cd ~/kodekloud
    git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
    cd certified-kubernetes-administrator-course/kubeadm-clusters/apple-silicon
    ```

## Mạng của máy ảo

Do cách ảo hóa hoạt động, mỗi VM cần hai network adapter: một adapter do Multipass sử dụng và một adapter cho mọi thứ khác. Mặc định các thành phần Kubernetes có thể bind vào adapter của Multipass, và đó *không* phải điều chúng ta muốn. Vì vậy, chúng tôi đã đặt sẵn biến môi trường `PRIMARY_IP` trên tất cả các VM, chứa địa chỉ IP mà các thành phần Kubernetes nên sử dụng. Trong các lab tiếp theo, bạn sẽ thấy biến này được dùng để đảm bảo các thành phần Kubernetes bind vào đúng network interface.

`PRIMARY_IP` được định nghĩa là địa chỉ IP của network interface trên node nối với mạng có default gateway, và đó là interface mà node dùng để nói chuyện với các node khác cũng như ra Internet. Ở chế độ bridge, đây cũng là một địa chỉ trong mạng nội bộ nhà bạn, do router cấp phát.

### Bridge Networking

Cấu hình mặc định của lab là khởi động VM trên các bridged interface. Nghĩa là các node Kubernetes sẽ xuất hiện như những máy bổ sung trong mạng nội bộ (LAN) của bạn, với địa chỉ IP do router mạng nhà bạn cấp phát động (DHCP). Điều này giúp bạn dùng trình duyệt để truy cập bất kỳ Service NodePort nào bạn triển khai.

Nếu gặp vấn đề khi triển khai bridge networking, hãy tạo [bug report](https://github.com/kodekloudhub/certified-kubernetes-administrator-course/issues) kèm đầy đủ thông tin, bao gồm output của `deploy-virtual-machines`.

Sau đó thử lại lab ở chế độ NAT. Cách làm được trình bày ở [phần tiếp theo](./02-compute-resources.md).

### NAT Networking

Trong cấu hình NAT, mạng mà các VM chạy trên đó bị tách biệt khỏi mạng của router nhà bạn bởi một NAT gateway do hypervisor quản lý. Nghĩa là các VM có thể nhìn ra ngoài (và kết nối Internet), nhưng bạn không thể nhìn vào trong (tức là dùng trình duyệt để kết nối tới NodePort). Hiện tại Multipass chưa hỗ trợ thiết lập quy tắc port forwarding để làm việc này.

Mạng mà các VM sử dụng do Multipass lựa chọn.

*Khuyến nghị* giữ nguyên mạng Pod và mạng Service ở giá trị mặc định. Nếu thay đổi, bạn sẽ phải sửa cả manifest của CNI cho phù hợp.

> 💡 **Bổ sung:** Bản gốc nhắc tới manifest mạng Weave, nhưng hướng dẫn hiện tại (xem [05-controlplane](../../generic/05-controlplane.md)) dùng **Calico**. Weave Net đã ngừng được phát triển sau khi Weaveworks đóng cửa (2024). Dù đổi hay không, hãy nhớ nguyên tắc: mạng node, mạng Pod (`POD_CIDR`) và mạng Service (`SERVICE_CIDR`) không được chồng lấn nhau.

Nếu bạn quyết định thay đổi bất kỳ giá trị nào, hãy coi đó là tùy chọn cá nhân và đừng tạo pull request.


## Chạy lệnh song song với iterm2

[iterm2](https://iterm2.com/) là ứng dụng thay thế phổ biến cho terminal mặc định của Mac, có thể dùng để chạy cùng một lệnh trên nhiều compute instance cùng lúc. Một số lab trong tutorial này yêu cầu chạy cùng một lệnh trên nhiều compute instance, ví dụ khi cài phần mềm Kubernetes. Khi đó bạn có thể cân nhắc dùng iterm2, chia cửa sổ thành nhiều pane và bật *Broadcast input to all panes* để tăng tốc quá trình provisioning.

*Việc dùng iterm2 là tùy chọn, không bắt buộc để hoàn thành tutorial này*.

![titerm2 screenshot](../../../images/iterm2-broadcast.png)

Để thiết lập như hình trên, làm như sau trong iterm2
1. Chuột phải và chọn split pane horizontally (chia pane theo chiều ngang)
1. Làm lại lần nữa để tạo pane thứ ba (nếu dựng 2 worker)
1. Trong mỗi pane, kết nối tới một node khác nhau bằng `Multipass shell`
1. Từ menu `Session` ở trên cùng, chọn `Broadcast` -> `Broadcast imput to all panes`. Biểu tượng nhỏ ở góc trên bên phải mỗi pane cho biết chế độ broadcast đã bật.

Nội dung gõ hoặc dán vào một dấu nhắc lệnh sẽ được lặp lại ở các dấu nhắc khác. Nhớ tắt broadcast khi đã xong một phần áp dụng cho tất cả các node.

Tiếp theo: [Compute Resources](02-compute-resources.md)

[](#running-commands-in-parallel-with-iterm2)