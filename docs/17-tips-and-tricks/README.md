# Mẹo và thủ thuật (Tips and Tricks)

Trong phần bonus này, chúng ta sẽ bàn về một số mẹo hữu ích có thể dùng khi ôn luyện cho kỳ thi.

- [01-Server for testing network policies](docs/01-server-for-testing-network-policies.md) – dựng một server để kiểm thử Network Policy
- [02-Client-for-testing-network-things](docs/02-client--for-testing-network-things.md) – dùng một Pod client để kiểm thử các vấn đề về mạng

> 💡 **Bổ sung – Một số mẹo chung khi thi CKA:**
>
> - Dùng alias `k=kubectl` và bash completion (`source <(kubectl completion bash)`, `complete -o default -F __start_kubectl k`) để gõ lệnh nhanh hơn.
> - Sinh manifest bằng `--dry-run=client -o yaml` thay vì viết tay, ví dụ `k create deploy web --image=nginx --replicas=3 --dry-run=client -o yaml > web.yaml`.
> - Quên tên trường? Dùng `kubectl explain <resource>.<field>` (thêm `--recursive` để xem toàn bộ cây trường).
> - Kiểm thử kết nối mạng nhanh bằng một Pod tạm: `k run tmp --image=busybox:1.28 --rm -it --restart=Never -- wget -qO- -T 2 <service>:<port>` (hoặc `nslookup <service>` để kiểm tra DNS). Pod tự xóa sau khi lệnh kết thúc nhờ `--rm`.
> - Kiểm tra quyền RBAC bằng `kubectl auth can-i <verb> <resource> --as=<user> -n <namespace>`.
> - Quản lý thời gian: bỏ qua câu đang bí, đánh dấu để quay lại sau; ưu tiên gom điểm từ các câu dễ trước.
