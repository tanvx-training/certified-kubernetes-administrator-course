# Bind PVC orange-pvc-cka13-trb

Với câu hỏi này, hãy chuyển context sang cluster1 bằng lệnh:

```
kubectl config use-context cluster1
```

Đã có sẵn một persistent volume tên là orange-pv-cka13-trb. Một persistent volume claim tên là orange-pvc-cka13-trb đã được tạo để claim storage từ orange-pv-cka13-trb.

Tuy nhiên, PVC này đang bị kẹt ở trạng thái Pending. Hiện tại volume chưa có dữ liệu nào.

Hãy troubleshoot và sửa lỗi này, đảm bảo PVC orange-pvc-cka13-trb ở trạng thái Bound.

---

### Lời giải

1. Describe PVC để xác định vấn đề

    ```
    kubectl describe pvc orange-pvc-cka13-trb
    ```

    Chú ý thông báo "requested PV is too small" (PV được yêu cầu quá nhỏ). Ta phải điều chỉnh PVC cho vừa.

2.  Describe PV để xem các thuộc tính của nó. Lưu ý rằng các thuộc tính của PVC phải được điều chỉnh cho khớp.

    ```
    kubectl describe pv orange-pv-cka13-trb
    ```

3.  Điều chỉnh PVC. Lưu ý rằng không thể sửa trực tiếp để giảm dung lượng của PVC, vì vậy ta phải thay thế (replace) nó.

    ```
    kubectl get pvc orange-pvc-cka13-trb -o yaml  > pvc.yaml
    vi pvc.yaml
    ```

    Sửa dung lượng yêu cầu (`spec.resources.requests.storage`) cho bằng dung lượng của PV. Lưu và thoát vi, rồi replace PVC bằng manifest đã sửa:

    ```
    kubectl replace --force -f pvc.yaml
    ```

> 💡 **Bổ sung:**
> - Vì sao không `kubectl edit` được? Kubernetes chỉ cho phép **tăng** dung lượng PVC (và chỉ khi StorageClass có `allowVolumeExpansion: true`); giảm dung lượng sẽ bị API server từ chối. `replace --force` xóa PVC cũ rồi tạo lại – an toàn ở đây vì đề nói volume chưa có dữ liệu.
> - Một PVC bind được với PV khi: dung lượng PV ≥ dung lượng yêu cầu, `accessModes` của PVC nằm trong `accessModes` của PV, `storageClassName` khớp (kể cả khi cả hai đều rỗng), và `volumeName`/selector (nếu có) trỏ đúng PV. Khi PVC Pending, hãy so sánh lần lượt các trường này giữa `kubectl get pv -o yaml` và `kubectl get pvc -o yaml`.
> - Nếu PV đang ở trạng thái `Released` (đã từng bind với PVC cũ), nó sẽ không tự bind lại; cần xóa `spec.claimRef` của PV (`kubectl edit pv ...`) để PV trở về `Available`.
