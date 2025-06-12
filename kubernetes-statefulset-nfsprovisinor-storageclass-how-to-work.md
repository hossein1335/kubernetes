# راهنمای تنظیمات StatefulSet و PVC برای ذخیره‌سازی دائمی

این سند مراحل تنظیم یک **StatefulSet** با استفاده از **PVC (Persistent Volume Claim)** و اتصال آن به **StorageClass** و **Provisioner** را توضیح می‌دهد.

## مراحل

1. **تعریف PVC در StatefulSet**  
   در بخش `volumeClaimTemplates` از مانیفست **StatefulSet**، یک **PVC** تعریف می‌شود.  
   مثال:
   ```yaml
   volumeClaimTemplates:
     - metadata:
         name: postgres-storage
       spec:
         accessModes: ["ReadWriteOnce"]
         storageClassName: nfs-storage
         resources:
           requests:
             storage: 10Gi
   ```

2. **اتصال PVC به StorageClass**  
   در تنظیمات بالا، `storageClassName` مشخص می‌کند که این **PVC** به کدام **StorageClass** متصل است.  
   مثال:
   ```yaml
   storageClassName: nfs-storage
   ```

3. **تعریف StorageClass و Provisioner**  
   **StorageClass** مشخص می‌کند که **PVC** باید به کدام **Provisioner** ارسال شود.  
   مثال:
   ```yaml
   provisioner: nfs-subdir-external-provisioner
   ```

4. **ایجاد دایرکتوری توسط Provisioner**  
   **Provisioner** روی سرور **NFS** (که در تنظیمات **StorageClass** یا فایل `values.yaml` مشخص شده) عمل می‌کند و یک دایرکتوری جدید در مسیر تعیین‌شده (مثلاً `/srv/nfs/postgres-data`) برای **PVC** ایجاد می‌کند.  
   نام دایرکتوری معمولاً به‌صورت زیر است:
   ```
   /srv/nfs/postgres-data/pvc-<uid>
   ```
   مثال:
   ```
   /srv/nfs/postgres-data/pvc-23af8c9f-...-1a2b
   ```

5. **اتصال PV به PVC و استفاده در Pod**  
   **PV (Persistent Volume)** که به‌صورت خودکار ساخته شده، به دایرکتوری ایجادشده **bind** می‌شود و به **PVC** متصل می‌گردد.  
   در نهایت:
   - **Pod** داخل **StatefulSet** می‌تواند از این **Volume** استفاده کند.
   - داده‌ها به‌صورت **persistent** ذخیره می‌مانند.
