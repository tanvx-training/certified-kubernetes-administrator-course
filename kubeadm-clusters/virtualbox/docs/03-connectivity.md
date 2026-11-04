# Kết nối (Connectivity)

Trước tiên, hãy chọn một hệ thống mà từ đó bạn sẽ thực hiện các tác vụ quản trị, như tạo certificate, file kubeconfig và phân phối chúng tới các VM khác nhau.

Nếu bạn dùng laptop Linux, laptop của bạn có thể là hệ thống này. Trong trường hợp của tôi, tôi chọn node `controlplane` để thực hiện các tác vụ quản trị. Dù chọn hệ thống nào, hãy đảm bảo hệ thống đó có thể truy cập tất cả các VM đã tạo qua SSH để sao chép file.

## Truy cập tất cả các VM

Ở đây chúng ta tạo một cặp SSH key cho user `vagrant` mà ta đang đăng nhập. Chúng ta sẽ sao chép public key của cặp key này lên cả hai worker để có thể SSH (và SCP) không cần mật khẩu từ `controlplane` sang các node khác dưới danh nghĩa user `vagrant`, user này có trên tất cả các node.

Tạo cặp key trên node `controlplane`

[//]: # (host:controlplane)

```bash
ssh-keygen
```

Để mọi thiết lập ở giá trị mặc định (chỉ cần nhấn ENTER cho mọi câu hỏi).

Sao chép key sang các host khác. Ở bước này, hãy nhập `vagrant` khi được hỏi mật khẩu.

Tùy chọn `-o StrictHostKeyChecking=no` bảo lệnh không hỏi xác nhận khi kết nối tới một host chưa từng biết. Đây không phải best practice ngoài thực tế, nhưng giúp làm nhanh hơn ở đây.

```bash
ssh-copy-id -o StrictHostKeyChecking=no vagrant@node01
```

```bash
ssh-copy-id -o StrictHostKeyChecking=no vagrant@node02
```

Với mỗi host, output sẽ tương tự như sau. Nếu không, có thể bạn đã nhập sai mật khẩu. Hãy thử lại bước này.

```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@node01'"
and check to make sure that only the key(s) you wanted were added.
```

> 💡 **Bổ sung:** Kiểm tra nhanh bằng `ssh node01 hostname` — nếu in ra `node01` mà không hỏi mật khẩu là thành công. `ssh-copy-id` thực chất nối public key (`~/.ssh/id_rsa.pub` hoặc `~/.ssh/id_ed25519.pub`, tùy loại key mà `ssh-keygen` tạo) vào file `~/.ssh/authorized_keys` trên máy đích.


Tiếp theo: [Node Setup](../../generic/04-node-setup.md)<br>
Trước: [Compute Resources](02-compute-resources.md)
