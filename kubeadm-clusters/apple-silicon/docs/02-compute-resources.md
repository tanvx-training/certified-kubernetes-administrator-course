# Tài nguyên tính toán (Compute Resources)

Vì không dùng được VirtualBox mà thay bằng Multipass, chúng tôi cung cấp sẵn [một script](../deploy-virtual-machines.sh) để tạo ba VM.

1. Chạy script tạo VM từ ứng dụng terminal trên Mac

    ```bash
    ./deploy-virtual-machines.sh
    ```

2. Kiểm tra bạn có thể kết nối tới cả ba VM (hai VM nếu Mac chỉ có 8GB RAM):

    ```bash
    multipass shell controlplane
    ```

    Bạn sẽ thấy dấu nhắc lệnh dạng `ubuntu@controlplane:~$`

    Gõ lệnh sau để quay về terminal của Mac

    ```bash
    exit
    ```

    Làm tương tự với `node01` và `node02`

> 💡 **Bổ sung:** `multipass list` hiển thị danh sách VM cùng trạng thái và địa chỉ IP (VM có thể có hai IP — tương ứng hai network adapter đã nói ở phần trước).

Nếu gặp sự cố khi khởi động các VM, bạn có thể thử chế độ NAT. Lưu ý ở chế độ NAT, bạn sẽ không thể kết nối tới Service NodePort bằng trình duyệt.

1. Chạy
    ```
    ./delete-virtual-machines.sh
    ```
1. Sửa `deploy-virtual-machines.sh`, đổi `BUILD_MODE="BRIDGE"` thành `BUILD_MODE="NAT"` ở dòng 15.


# Xóa các máy ảo

Khi đã dùng xong cluster và muốn thu hồi tài nguyên, thực hiện các bước sau

1. Thoát khỏi tất cả các phiên làm việc trên VM
1. Chạy [script xóa](../delete-virtual-machines.sh) từ ứng dụng terminal trên Mac

    ```bash
    ./delete-virtual-machines.sh
    ````

1. Dọn các DHCP lease cũ. Multipass không tự làm việc này, và nếu bạn không tự dọn thì sớm muộn sẽ hết địa chỉ IP trên mạng VM của multipass.

    1. Sửa file sau

        ```bash
        sudo vi /var/db/dhcpd_leases
        ```

    1. Xóa tất cả các khối có dạng như dưới đây, cụ thể là những khối có `name` là `controlplane`, `node01` hoặc `node02`
        ```text
        {
            name=controlplane
            ip_address=192.168.64.4
            hw_address=1,52:54:0:78:4d:ff
            identifier=1,52:54:0:78:4d:ff
            lease=0x65dc3134
        }
        ```

    1. Lưu file và thoát

Tiếp theo: [Connectivity](./03-connectivity.md)<br>
Trước: [Prerequisites](./01-prerequisites.md)
