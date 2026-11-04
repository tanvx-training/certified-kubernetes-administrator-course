# Client để kiểm thử các vấn đề mạng

Bạn sẽ thường gặp những câu hỏi yêu cầu kiểm thử network policy, hoặc tra cứu thông tin trong DNS của Kubernetes. Có một Pod "dùng được cho mọi việc" mà bạn có thể deploy, chứa mọi công cụ kiểm thử mạng bạn cần, bao gồm

* curl
* nslookup
* netstat
* dig
* telnet
* nc

và còn nhiều nữa.

Chạy nó như sau. Hãy ghi nhớ tên image này – nó là cứu tinh! Không có quy định nào cấm bạn dùng nó trong kỳ thi.

```
kubectl run tester --image wbitt/network-multitool
```

Khi Pod đã chạy, bạn có thể exec vào và chạy các lệnh

```
$ kubectl exec tester -it -- bash

/# curl something
/# nslookup something-else
/# exit
```

Hoặc chạy lệnh trực tiếp nếu cần ghi kết quả ra file

```
$ kubectl exec tester -it -- nslookup my-service.default.svc > /opt/some-file.txt
```

> 💡 **Bổ sung:**
> - Khi redirect output ra file, nên **bỏ** `-it` (`kubectl exec tester -- nslookup my-service.default.svc > /opt/some-file.txt`). Với `-t`, kubectl cấp một TTY nên output có thể chứa ký tự `\r` (carriage return) ở cuối mỗi dòng, làm file khác với kết quả mong đợi.
> - Để test Network Policy từ một namespace khác, nhớ tạo Pod tester trong đúng namespace và với đúng label mà policy cho phép/chặn, ví dụ `kubectl run tester -n other-ns --image wbitt/network-multitool --labels=app=frontend`.
> - Dùng timeout để không phải chờ lâu khi kết nối bị chặn: `curl --connect-timeout 3 <host>:<port>` hoặc `nc -zv -w 3 <host> <port>`.
> - Nếu không pull được image (môi trường không có Internet), có thể dùng `busybox:1.28` có sẵn `wget`, `nslookup`, `nc`.
> - Dọn dẹp sau khi test: `kubectl delete pod tester --force --grace-period=0` để xóa ngay lập tức.
