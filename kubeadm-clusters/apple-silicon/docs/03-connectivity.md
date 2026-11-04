# Kết nối (Connectivity)

Chúng ta sẽ thực hiện hầu hết các tác vụ cấu hình từ node `controlplane`. Sau các bước dưới đây, bạn có thể dùng ssh để đăng nhập vào các worker node ngay từ dấu nhắc lệnh của `controlplane`.

Nếu muốn, bạn cũng có thể đăng nhập vào các worker node từ các phiên terminal riêng, ví dụ

```
multipass shell node01
```

## Truy cập tất cả các VM

Ở đây chúng ta tạo một cặp SSH key cho user `ubuntu` mà ta đang đăng nhập. Chúng ta sẽ sao chép public key của cặp key này lên cả hai worker để có thể SSH (và SCP) không cần mật khẩu từ `controlplane` sang các node khác dưới danh nghĩa user `ubuntu`, user này có trên tất cả các node.

Tạo cặp key trên node `controlplane`

[//]: # (host:controlplane)

```bash
ssh-keygen
```

Để mọi thiết lập ở giá trị mặc định (chỉ cần nhấn ENTER cho mọi câu hỏi).

Sao chép key sang các host khác. Ở bước này, hãy nhập `ubuntu` khi được hỏi mật khẩu.

Tùy chọn `-o StrictHostKeyChecking=no` bảo lệnh không hỏi xác nhận khi kết nối tới một host chưa từng biết. Đây không phải best practice ngoài thực tế, nhưng giúp làm nhanh hơn ở đây.

```bash
ssh-copy-id -o StrictHostKeyChecking=no ubuntu@node01
```
```bash
ssh-copy-id -o StrictHostKeyChecking=no ubuntu@node02
```

Với mỗi host, output sẽ tương tự như sau. Nếu không, có thể bạn đã nhập sai mật khẩu. Hãy thử lại bước này.

```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ubuntu@node01'"
and check to make sure that only the key(s) you wanted were added.
```

> 💡 **Bổ sung:** Kiểm tra nhanh bằng `ssh node01 hostname` — nếu in ra `node01` mà không hỏi mật khẩu là thành công. `ssh-copy-id` thực chất nối public key của bạn vào file `~/.ssh/authorized_keys` trên máy đích. Nếu Mac chỉ có 8GB RAM thì không có `node02`, hãy bỏ qua lệnh thứ hai.


Tiếp theo: [Node Setup](../../generic/04-node-setup.md)<br>
Trước: [Compute Resources](02-compute-resources.md)
