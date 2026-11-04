# Practice Test - Explore Env (Bài thực hành khám phá môi trường)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-explore-environment/)

#### Lời giải

1. <details>
   <summary>Cluster này có bao nhiêu node?</summary>

      ```
      kubectl get nodes
      ```

      Đếm số dòng kết quả

   </details>

1. <details>
   <summary>Địa chỉ Internal IP của node controlplane trong cluster này là gì?</summary>

      ```
      kubectl get nodes -o wide
      ```

      Xem giá trị ở cột `INTERNAL-IP` của `controlplane`

   </details>

1. <details>
   <summary>Network interface nào được cấu hình cho kết nối cluster trên node controlplane?</summary>

   Đó là network interface có cùng địa chỉ IP mà bạn tìm được ở câu trước.

   ```
   ip a
   ```

   Lệnh trên in ra khá nhiều. Ta có thể lọc cho gọn hơn:

   ```
   ip a | grep -B2 X.X.X.X
   ```

   trong đó `X.X.X.X` là địa chỉ IP bạn tìm được ở câu trước. `grep -B2` sẽ tìm dòng chứa giá trị cần tìm và in ra dòng đó cùng 2 dòng phía trước. Kết quả sẽ trông như dưới đây, dù các giá trị sẽ khác nhau mỗi lần bạn chạy lab.

   ```
   3058: eth0@if3059: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
      link/ether 02:42:c0:08:ea:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
     inet 192.8.234.3/24 brd 192.8.234.255 scope global eth0
   ```

   Từ đây, ta xác định được đáp án là

   > `eth0`

   > 💡 **Bổ sung:** Cách nhanh hơn: `ip -br a` (brief) in mỗi interface trên một dòng kèm trạng thái và IP; hoặc `ip route get <IP-của-node-khác>` sẽ cho biết interface được dùng để đi tới IP đó.

   </details>

1. <details>
   <summary>MAC address của interface đó trên node controlplane là gì?</summary>

   Giá trị này cũng có trong output của lệnh bạn chạy ở câu trước. MAC address là giá trị ở trường `link/ether`, gồm 6 số hex phân cách bởi `:`. Lưu ý giá trị có thể khác nhau mỗi lần chạy lab.

   Nếu output của `eth0` là

   ```
   3058: eth0@if3059: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
      link/ether 02:42:c0:08:ea:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
     inet 192.8.234.3/24 brd 192.8.234.255 scope global eth0
   ```

   thì MAC address là

   > `02:42:c0:08:ea:03`

   </details>

1. <details>
   <summary>Địa chỉ IP gán cho node01 là gì?</summary>

   ```
   kubectl get nodes -o wide
   ```

   Xem giá trị ở cột `INTERNAL-IP` của `node01`

   </details>

1. <details>
   <summary>MAC address gán cho node01 là gì?</summary>

   Để làm câu này, ta cần SSH vào `node01` để xem các interface của nó. Ta đã biết cần tìm IP nào nhờ câu trước

   ```
   ssh node01
   ip a | grep -B2 X.X.X.X
   ```

   trong đó `X.X.X.X` là địa chỉ IP bạn tìm được ở câu trước. Một lần nữa, hãy xem trường `link/ether`.

   Ta có thể đoán interface đúng trên `node01` cũng là `eth0` và chỉ cần chạy

   ```
   ip link show eth0
   ```

   nhưng tốt nhất là nên kiểm tra chắc chắn.

   Giờ quay về `controlplane`

   ```
   exit
   ```

   </details>

1. <details>
   <summary>Chúng ta dùng Containerd làm container runtime. Interface/bridge nào được Containerd tạo trên host này?</summary>

   Câu này không hiển nhiên ngay lập tức.

   ```
   ip link show
   ```

   Cần biết rằng

   * Mọi interface có tên bắt đầu bằng `eth` là interface "vật lý", đại diện cho một card mạng gắn vào host.
   * Interface `lo` là loopback, bao gồm mọi địa chỉ IP bắt đầu bằng `127`. Máy tính nào cũng có interface này.
   * Mọi interface có tên bắt đầu bằng `veth` là network interface ảo, dùng làm "đường ống" nối giữa host và mạng của Pod. Chúng được nối vào các bridge, và tên interface của bridge được liệt kê trong thông tin của chúng (`master ...`).

   Ta thấy hai thiết bị `veth` đều gắn với một thiết bị khác trong danh sách là `cni0`, do đó đáp án là `cni0`.

   > 💡 **Bổ sung:** Thực ra bridge `cni0` được tạo bởi CNI plugin (`bridge`, ví dụ khi dùng Flannel) mà containerd gọi tới, không phải bởi bản thân containerd. Có thể liệt kê nhanh các bridge bằng `ip link show type bridge`. Tên bridge phụ thuộc CNI: Weave tạo `weave`, Flannel tạo `cni0` (và `flannel.1` cho VXLAN), Calico không dùng bridge mà định tuyến trực tiếp qua các interface `cali*`.

   </details>

1. <details>
   <summary>Trạng thái của interface cni0 là gì?</summary>

   Trong output của lệnh ở câu trước, bạn sẽ thấy trường state của `cni0` là

   > `UP`

   </details>

1. <details>
   <summary>Nếu ping google từ node controlplane, gói tin sẽ đi theo route nào?</summary>

   Địa chỉ IP của Default Gateway là gì?

   Chạy

   ```
   ip route show default
   ```

   và ghi lại output

   </details>

1. <details>
   <summary>kube-scheduler đang lắng nghe trên port nào ở node controlplane?</summary>

   Dùng lệnh [netstat](https://linux.die.net/man/8/netstat) để xem các network socket được các chương trình chạy trên host sử dụng. Output khá dài, nên ta lọc theo tên process, tức `kube-scheduler`

   ```
   netstat -nplt | grep kube-scheduler
   ```

   <details>
   <summary>Ý nghĩa các tùy chọn netstat đã dùng</summary>

   * `-n` - Hiển thị địa chỉ IP (không cố phân giải thành hostname)
   * `-p` - Hiển thị tên process (ví dụ `kube-scheduler`)
   * `-l` - Chỉ lấy các socket đang _lắng nghe_ (listening)
   * `-t` - Chỉ lấy các socket TCP

   </details>

   Output:

   ```
   tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3291/kube-scheduler
   ```

   Ta thấy nó đang lắng nghe trên localhost, port `10259`

   > 💡 **Bổ sung:** Nếu không có `netstat`, dùng `ss -nplt | grep kube-scheduler` cho kết quả tương tự.
   </details>

1. <details>
   <summary>Để ý rằng ETCD đang lắng nghe trên hai port. Port nào có nhiều kết nối client được thiết lập hơn?</summary>

   Ta dùng `netstat` với các tùy chọn hơi khác và lọc theo `etcd`

   ```
   netstat -anp | grep etcd
   ```

   <details>
   <summary>Ý nghĩa các tùy chọn netstat đã dùng</summary>

   * `-a` - Lấy socket ở mọi trạng thái
   * `-n` - Hiển thị địa chỉ IP (không cố phân giải thành hostname)
   * `-p` - Hiển thị tên process (ví dụ `etcd`)

   </details>

   Bạn sẽ thấy port được dùng nhiều nhất, vượt trội hẳn, là `2379`.

   > 💡 **Bổ sung:** Để đếm nhanh, có thể dùng `netstat -anp | grep etcd | grep 2379 | wc -l` và so sánh với `2380`.

   </details>

1. Thông tin thêm

   Đó là vì `2379` là port của ETCD mà API server kết nối tới. Có nhiều kết nối đồng thời để API server có thể xử lý nhiều thao tác etcd cùng lúc.

    `2380` chỉ dùng cho kết nối ngang hàng (peer-to-peer) giữa các etcd khi bạn có nhiều node controlplane. Trong trường hợp này thì không có.
