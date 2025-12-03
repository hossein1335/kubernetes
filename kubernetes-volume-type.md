# Kubernetes emptyDir – همه‌چیز در ۲ دقیقه  

`emptyDir` ساده‌ترین و سریع‌ترین راه برای اشتراک داده بین کانتینرهای یک Pod است.

### ویژگی‌های کلیدی
- با ساخت Pod ایجاد می‌شه  
- با حذف Pod یا جابه‌جایی به Node دیگه کاملاً پاک می‌شه  
- بین تمام کانتینرهای یک Pod مشترک است  
- فقط برای داده‌های موقتی (cache, temp files, scratch, log sharing)

### فقط ۴ حالت معتبر وجود داره

| حالت                  | مانیفست                                      | محل ذخیره       | نکته مهم                              |
|-----------------------|----------------------------------------------|------------------|--------------------------------------|
| دیسک (پیش‌فرض)        | `emptyDir: {}`                               | دیسک Node        | رایج‌ترین حالت                       |
| دیسک + محدودیت         | `emptyDir: { sizeLimit: 1Gi }`               | دیسک Node        | از پر شدن نود جلوگیری می‌کنه        |
| RAM (tmpfs)           | `emptyDir: { medium: Memory }`               | RAM Node         | فوق‌العاده سریع                      |
| RAM + محدودیت (بهترین) | `emptyDir: { medium: Memory, sizeLimit: 500Mi }` | RAM Node     | سریع + امن (توصیه می‌شه)             |

### دموی زنده (emptydir-demo.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo 'سلام کوبرنتیز! $(date)' > /data/msg.txt; sleep 3600"]
    volumeMounts:
    - name: shared
      mountPath: /data

  - name: reader
    image: busybox
    command: ["sh", "-c", "sleep 7; echo 'خوندم:'; cat /shared/msg.txt; sleep 3600"]
    volumeMounts:
    - name: shared
      mountPath: /shared

  volumes:
  - name: shared
    emptyDir:
      medium: Memory      # روی RAM (سریع)
      sizeLimit: 200Mi    # حداکثر ۲۰۰ مگ
