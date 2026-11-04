# Cấp phát tài nguyên tính toán (Compute Resources)

Lưu ý: Đến bước này bạn phải cài đặt và cấu hình xong VirtualBox và Vagrant.

Mở một ứng dụng terminal (trên Windows dùng PowerShell). Mọi lệnh trong hướng dẫn này đều được chạy từ terminal.

Tải github repository này về và cd vào thư mục `virtualbox`

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
```

CD vào thư mục virtualbox

```bash
cd kubeadm-clusters/virtualbox
```

> 💡 **Bổ sung:** Lệnh `git clone` tạo thư mục `certified-kubernetes-administrator-course`, vì vậy nếu lệnh `cd` trên báo lỗi không tìm thấy thư mục, hãy chạy `cd certified-kubernetes-administrator-course` trước.

Chạy Vagrant up để tạo các máy ảo.

```bash
vagrant up
```


## Chọn bridge interface

Bridged networking khiến các VM xuất hiện như các host trực tiếp trên mạng LAN của bạn. Nghĩa là bạn có thể dùng chính trình duyệt của mình để kết nối tới bất kỳ Service NodePort nào bạn tạo trong cluster.

Nếu máy của bạn có nhiều hơn một network interface có thể tạo bridge, Vagrant có thể dừng lại và hỏi bạn dùng interface nào nếu nó không tự xác định được interface tốt nhất. Hiện tại nó *lẽ ra* sẽ tự xác định được mà không cần hỏi, nhưng nếu không, thì interface "tốt nhất" là interface dùng để kết nối tới router mạng nhà bạn. Trên laptop, đó thường là card Wi-Fi, khá dễ nhận ra trong danh sách. Ví dụ dưới đây lấy từ một máy để bàn Windows có card mạng có dây.

Theo bạn, lựa chọn nào trong hai lựa chọn dưới đây là đúng?

<details>
<summary>Xem đáp án</summary>

> `Intel(R) Ethernet Connection (2) I219-V`

Vì sao? Bởi vì
1. Ethernet là thuật ngữ thường dùng cho kết nối mạng có dây.
2. Lựa chọn còn lại là Hyper-V, một adapter nội bộ dùng để chạy VM gốc của Windows (thực ra có thể dùng thay VirtualBox, nhưng đó là chuyện khác).

</details>

```text
==> controlplane: Available bridged network interfaces:
1) Intel(R) Ethernet Connection (2) I219-V
2) Hyper-V Virtual Ethernet Adapter
==> controlplane: When choosing an interface, it is usually the one that is
==> controlplane: being used to connect to the internet.
==> controlplane:
    controlplane: Which interface should the network bridge to?
```

Khi triển khai xong, nó sẽ cho bạn biết cách truy cập các Service NodePort từ trình duyệt sau khi đã cấu hình Kubernetes. Hãy ghi lại thông tin này.

Nếu gặp sự cố khi khởi động các VM, bạn có thể thử chế độ NAT. Lưu ý ở chế độ NAT, bạn sẽ không thể kết nối tới Service NodePort bằng trình duyệt nếu không thiết lập quy tắc port forwarding trong giao diện VirtualBox.

1. Chạy
    ```
    vagrant destroy -f
    ```
1. Sửa file `vagrantfile`, đổi `BUILD_MODE = "BRIDGE"` thành `BUILD_MODE = "NAT"` ở dòng 10.

## SSH vào các node

Có hai cách để SSH vào các node:

### 1. SSH bằng Vagrant

  Từ thư mục bạn đã chạy lệnh `vagrant up`, chạy `vagrant ssh <vm>`, ví dụ `vagrant ssh controlplane`.

  Đây là cách dễ nhất vì không cần cấu hình gì.

### 2. SSH bằng công cụ SSH client

Dùng công cụ SSH Terminal yêu thích của bạn (PuTTY/MobaXTerm v.v.).

Dùng các địa chỉ IP ở trên (địa chỉ được in ra khi `vagrant up` kết thúc). Đăng nhập SSH bằng username và password bị tắt theo mặc định.
Vagrant tạo một private key cho mỗi VM. Key này nằm trong thư mục .vagrant (trong thư mục bạn đã chạy lệnh `vagrant up`) theo đường dẫn dưới đây cho từng VM:

**Đường dẫn private key:** `.vagrant/machines/<machine name>/virtualbox/private_key`

**Username/Password:** `vagrant/vagrant`


## Kiểm tra môi trường

- Đảm bảo tất cả các VM đã chạy
- Đảm bảo các VM được gán đúng các địa chỉ IP ở trên
- Đảm bảo bạn có thể SSH vào các VM bằng IP và private key, hoặc bằng `vagrant ssh`
- Đảm bảo các VM có thể ping lẫn nhau

> 💡 **Bổ sung:** Có thể dùng `vagrant status` để xem trạng thái các VM (`running`, `poweroff`...). Trong VM, `echo $PRIMARY_IP` cho biết IP mà Kubernetes sẽ dùng, và `ping node01` / `ping node02` từ `controlplane` để kiểm tra kết nối (tên host đã được ghi sẵn vào `/etc/hosts`).

## Mẹo troubleshooting

### Provisioning thất bại

Nếu có VM nào provisioning thất bại hoặc cấu hình không đúng, hãy xóa VM đó bằng lệnh:

```bash
vagrant destroy <vm>
```

Sau đó provisioning lại. Chỉ những VM bị thiếu mới được tạo lại

```bash
vagrant up
```


Đôi khi lệnh xóa không xóa được thư mục tạo cho VM và báo lỗi tương tự như sau:

Lỗi VirtualBox:

    VBoxManage.exe: error: Could not rename the directory 'D:\VirtualBox VMs\ubuntu-bionic-18.04-cloudimg-20190122_1552891552601_76806' to 'D:\VirtualBox VMs\kubernetes-ha-worker-2' to save the settings file (VERR_ALREADY_EXISTS)
    VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component SessionMachine, interface IMachine, callee IUnknown
    VBoxManage.exe: error: Context: "SaveSettings()" at line 3105 of file VBoxManageModifyVM.cpp

Khi đó hãy xóa VM, rồi xóa thư mục của VM, sau đó provisioning lại, ví dụ

```bash
vagrant destroy node02
rmdir "<path-to-vm-folder>\node02
vagrant up
```

### Provisioner bị treo

Hiện tượng này thường xảy ra nhất ở bước "Waiting for machine to reboot"

1. Nhấn `CTRL+C`
1. Kill mọi tiến trình `ruby` đang chạy, nếu không Vagrant sẽ báo lỗi.
1. Destroy VM bị treo: `vagrant destroy <vm>`
1. Provisioning lại. Vagrant sẽ tiếp tục từ chỗ đã dừng: `vagrant up`

# Tạm dừng môi trường

Bạn không cần hoàn thành toàn bộ lab trong một lần. Nếu cần tắt máy tính, bạn có thể tắt và khởi động lại môi trường như sau.

Để tắt. Lệnh này sẽ tắt nhẹ nhàng (graceful shutdown) tất cả các VM theo thứ tự ngược với thứ tự khởi động:

```
vagrant halt
```

Để bật lại:

```
vagrant up
```

# Xóa các máy ảo

Khi đã dùng xong cluster và muốn thu hồi tài nguyên, thực hiện các bước sau

1. Thoát khỏi tất cả các phiên làm việc trên VM
1. Chạy lệnh sau

    ```
    vagrant destroy -f
    ```

Tiếp theo: [Connectivity](./03-connectivity.md)<br/>
Trước: [Prerequisites](./01-prerequisites.md)
