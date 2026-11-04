# Worker Node Failure (Lỗi Worker Node)

  - Xem [Bài giảng](https://kodekloud.com/topic/worker-node-failure/)

  - Kiểm tra trạng thái các node trong cluster: chúng đang **`Ready`** hay **`NotReady`**

    ```
    kubectl get nodes
    ```

  - Nếu node ở trạng thái **`NotReady`** thì kiểm tra **`LastHeartbeatTime`** của node để biết thời điểm node có thể đã bị sập

    ```
    kubectl describe node worker-1
    ```

    ![wrk](../../images/wrk.PNG)

    > 💡 **Bổ sung:** Trong phần `Conditions` của `kubectl describe node`, các điều kiện `MemoryPressure`, `DiskPressure`, `PIDPressure` có giá trị `True` cho biết node đang thiếu tài nguyên. Nếu tất cả có trạng thái `Unknown` (kèm thông báo `Kubelet stopped posting node status`) thì control plane đã mất liên lạc với kubelet trên node đó — thường do kubelet bị dừng/crash hoặc node bị sập/mất mạng.


  - Kiểm tra tình trạng **`CPU`** và **`MEMORY`** bằng **`top`**, và dung lượng đĩa bằng **`df -h`** 

 
    ![mem](../../images/mem.PNG)


  - Kiểm tra trạng thái và log của **`kubelet`** để tìm nguyên nhân có thể có.

    ```
    serivce kubelet status
    ```

    ```
    sudo journalctl -u kubelet
    ```
    ![kublet](../../images/kublet.PNG)

    > 💡 **Bổ sung:** Lệnh đầu tiên trong bản gốc bị gõ sai, đúng phải là `service kubelet status`, hoặc dùng `systemctl status kubelet` (phổ biến hơn trên hệ thống systemd). Một số thao tác hữu ích với kubelet:
    > - `systemctl start kubelet` / `systemctl restart kubelet` — khởi động lại sau khi sửa cấu hình (nếu sửa file unit thì chạy `systemctl daemon-reload` trước).
    > - `systemctl cat kubelet` — xem file unit và các drop-in (vd `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf`) để biết kubelet dùng file config nào (`--config=/var/lib/kubelet/config.yaml`) và kubeconfig nào (`--kubeconfig=/etc/kubernetes/kubelet.conf`).
    > - `journalctl -u kubelet -f` để theo dõi log theo thời gian thực, `journalctl -u kubelet --since "10 min ago"` hoặc `journalctl -u kubelet | tail -50` để chỉ xem log gần nhất.
    > - Đừng quên kiểm tra cả container runtime: `systemctl status containerd` và `crictl ps -a`.
  
    
  - Kiểm tra các certificate của **`kubelet`**: đảm bảo chúng chưa hết hạn, thuộc đúng group và được cấp bởi đúng CA.

    ```
    openssl x509 -in /var/lib/kubelet/worker-1.crt -text
    ```

    ![cert](../../images/cert.PNG)

    > 💡 **Bổ sung:** Với cluster kubeadm, client certificate của kubelet thường nằm ở `/var/lib/kubelet/pki/kubelet-client-current.pem` (được tự động xoay vòng - rotate). Khi đọc output của `openssl x509 ... -text -noout`, hãy chú ý `Issuer` (phải là CA của cluster), `Validity` (`Not After`), và `Subject` (phải có dạng `O = system:nodes, CN = system:node:<tên-node>`).


