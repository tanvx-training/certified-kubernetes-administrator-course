# Practice Test - Storage Class (Bài thực hành)
  
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-storage-class-2/)

#### Lời giải

Lời giải cho bài practice test.

  1. Xem lời giải

     <details>

      ```
      0
      ```

     > 💡 Dùng `kubectl get sc` để đếm số StorageClass trong cluster.

    
     </details>

  2. Xem lời giải

     <details>

      ```
      2
      ```
     </details>
 
  3. Xem lời giải

     <details>

      ```
      local-storage
      ```

     > 💡 StorageClass dùng `kubernetes.io/no-provisioner` không hỗ trợ dynamic provisioning — PV phải được tạo thủ công.

     </details>

  4. Xem lời giải
    
     <details>

      ```
      WaitForFirstConsumer
      ```

     > 💡 Xem cột VOLUMEBINDINGMODE trong `kubectl get sc`.

      </details>

  5. Xem lời giải

     <details>

      ```
      portworx-volume
      ```

     > 💡 Xem cột PROVISIONER trong `kubectl get sc` (ở đây là `kubernetes.io/portworx-volume`).


     </details>

  6. Xem lời giải

     <details>

      ```
      NO
      ```

     > 💡 StorageClass `local-storage` không có provisioner thực sự (`kubernetes.io/no-provisioner`) nên không thể cấp phát động.

     </details>

  7. Xem lời giải

     <details>

      ```
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: local-pvc
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 500Mi
        storageClassName: local-storage
      ```

     > 💡 Trường `storageClassName` phải khớp với StorageClass của PV có sẵn thì PVC mới bind được.

     </details>

  8. Xem lời giải

     <details>

      ```
      Pending
      ```

     > 💡 Kiểm tra lý do bằng `kubectl describe pvc local-pvc` — phần Events sẽ ghi `waiting for first consumer to be created before binding`.

     </details>

  9. Xem lời giải

     <details>

      ```
      A Pod consuming the volume in not scheduled
      ```
     </details>

  10. Xem lời giải

      <details>
 
       ```
       The Storage Class called local-storage makes use of VolumeBindingMode set to WaitForFirstConsumer. This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.
       ```
      </details>

  11. Xem lời giải

      <details>
 
       ```
       apiVersion: v1
       kind: Pod
       metadata:
         name: nginx
         labels:
           name: nginx
       spec:
           containers:
           - name: nginx
             image: nginx:alpine
             volumeMounts:
             - name: local-persistent-storage
               mountPath: /var/www/html
           volumes:
           - name: local-persistent-storage
             persistentVolumeClaim:
               claimName: local-pvc
       ```

      > 💡 Sau khi Pod `nginx` được tạo và schedule, PVC `local-pvc` sẽ chuyển sang `Bound`.

      </details>

  12. Xem lời giải

      <details>
 
       ```
       apiVersion: storage.k8s.io/v1
       kind: StorageClass
       metadata:
         name: delayed-volume-sc
       provisioner: kubernetes.io/no-provisioner
       volumeBindingMode: WaitForFirstConsumer
       ```
      </details>


