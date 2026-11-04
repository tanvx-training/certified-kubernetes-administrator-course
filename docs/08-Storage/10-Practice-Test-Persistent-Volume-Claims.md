# Practice Test - Persistent Volume Claims (Bài thực hành)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-persistent-volume-claims/)

#### Lời giải

Lời giải cho bài practice test.

  1. Xem lời giải

     <details>

      ```
      OK
      ```
    
     </details>

  2. Xem lời giải

     <details>

      ```
      OK
      ```
     </details>
 
  3. Xem lời giải

     <details>

      ```
      No
      ```
     </details>

  4. Xem lời giải
    
     <details>

      ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: webapp
      spec:
        containers:
        - name: event-simulator
          image: kodekloud/event-simulator
          env:
          - name: LOG_HANDLERS
            value: file
          volumeMounts:
          - mountPath: /log
            name: log-volume
      
        volumes:
        - name: log-volume
          hostPath:
            # directory location on host
            path: /var/log/webapp
            # this field is optional
            type: Directory
      ```

     > 💡 Pod `webapp` ghi log vào `/log` trong container; mount thư mục `/var/log/webapp` của node vào đó bằng `hostPath` để log được giữ lại trên node. Pod không sửa được phần `volumes` khi đang chạy, nên hãy `kubectl get pod webapp -o yaml > webapp.yaml`, sửa file, rồi `kubectl replace --force -f webapp.yaml`.

      </details>

  5. Xem lời giải

     <details>

      ```
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: pv-log
      spec:
        accessModes:
          - ReadWriteMany
        capacity:
          storage: 100Mi
        hostPath:
          path: /pv/log
      ```

     </details>

  6. Xem lời giải

     <details>

      ```
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: claim-log-1
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 50Mi
      ```
     </details>

  7. Xem lời giải

     <details>

      ```
      PENDING
      ```

     > 💡 PVC ở trạng thái `Pending` vì không tìm được PV phù hợp để bind.

     </details>

  8. Xem lời giải

     <details>

      ```
      AVAILABLE
      ```

     > 💡 PV `pv-log` vẫn `Available` vì chưa bind với PVC nào.

     </details>

  9. Xem lời giải

     <details>

      ```
      Access Modes Mismatch
      ```

     > 💡 PV hỗ trợ `ReadWriteMany` còn PVC lại yêu cầu `ReadWriteOnce` nên không khớp. Dùng `kubectl describe pvc claim-log-1` và `kubectl get pv` để so sánh cột ACCESS MODES.

     </details>

  10. Xem lời giải

      <details>
 
       ```
       kind: PersistentVolumeClaim
       apiVersion: v1
       metadata:
         name: claim-log-1
       spec:
         accessModes:
           - ReadWriteMany
         resources:
           requests:
             storage: 50Mi
       ```
      </details>

  11. Xem lời giải

      <details>
 
       ```
       100Mi
       ```

      > 💡 PVC xin 50Mi nhưng dung lượng hiển thị là 100Mi, vì PVC bind với cả PV `pv-log` (100Mi) — quan hệ PV/PVC là 1-1, PVC nhận toàn bộ dung lượng của PV.

      </details>

  12. Xem lời giải

      <details>
 
       ```
       apiVersion: v1
       kind: Pod
       metadata:
         name: webapp
       spec:
         containers:
         - name: event-simulator
           image: kodekloud/event-simulator
           env:
           - name: LOG_HANDLERS
             value: file
           volumeMounts:
           - mountPath: /log
             name: log-volume
       
         volumes:
         - name: log-volume
           persistentVolumeClaim:
             claimName: claim-log-1
       ```
      </details>

  13. Xem lời giải

      <details>
 
       ```
       Retain
       ```

      > 💡 PV tạo thủ công mặc định có `persistentVolumeReclaimPolicy: Retain`. Xem cột RECLAIM POLICY trong `kubectl get pv`.

      </details>

  14. Xem lời giải

      <details>
 
       ```
       The PV is not delete but not available
       ```

      > 💡 Với `Retain`, khi xóa PVC thì PV không bị xóa nhưng cũng không sẵn sàng để PVC khác sử dụng (chuyển sang trạng thái `Released`).

      </details>

  15. Xem lời giải

      <details>
 
       ```
       The PVC is stuck in `terminating` state
       ```

      > 💡 PVC kẹt ở `Terminating` nhờ cơ chế bảo vệ (finalizer `kubernetes.io/pvc-protection`): Kubernetes không xóa PVC khi nó còn đang được Pod sử dụng.

      </details>

  16. Xem lời giải

      <details>
 
       ```
       The PVC is being used by a POD
       ```
      </details>

  17. Xem lời giải

      <details>
 
       ```
       kubectl delete pod webapp
       ```
      </details>

  18. Xem lời giải

      <details>
 
       ```
       Deleted
       ```
      </details>

  19. Xem lời giải

      <details>
 
       ```
       Released
       ```

      > 💡 Sau khi xóa Pod, PVC được xóa hẳn; PV chuyển sang `Released` (dữ liệu vẫn còn). Muốn dùng lại PV này, quản trị viên phải xóa trường `spec.claimRef` của PV (hoặc xóa và tạo lại PV).

      </details>