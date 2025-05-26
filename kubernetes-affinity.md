# انواع Affinity در Kubernetes

در Kubernetes، **Affinity** برای کنترل محل قرارگیری پادها (Pods) روی نودها یا نسبت به پادهای دیگر استفاده می‌شود. سه نوع Affinity وجود دارد که هر کدام می‌توانند در دو حالت **required** یا **preferred** تنظیم شوند. در ادامه، جزئیات هر نوع Affinity به همراه مثال‌ها و جدول جمع‌بندی ارائه شده است.

## ۱. nodeAffinity
- **توضیح**: مشخص می‌کند که پاد روی کدام نودها (سرورها) قرار بگیرد یا نگیرد.
- **کاربرد**: برای اطمینان از اجرای پاد روی نودهای خاص با ویژگی‌های مشخص (مثل برچسب‌های خاص یا سخت‌افزار خاص).
- **حالت‌ها**:
  - `requiredDuringSchedulingIgnoredDuringExecution`: پاد حتماً باید روی نودهایی با شرایط مشخص اجرا شود.
  - `preferredDuringSchedulingIgnoredDuringExecution`: ترجیحاً روی نودهایی با شرایط مشخص اجرا شود، اما اگر ممکن نبود، جای دیگری هم قابل قبول است.

### مثال nodeAffinity
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node1
                - node2
```

## ۲. podAffinity
- **توضیح**: مشخص می‌کند که پاد باید کنار پادهای دیگر (روی همان نود یا zone) قرار بگیرد.
- **کاربرد**: برای قرار گرفتن پادها در نزدیکی پادهای دیگر با برچسب‌های خاص (مثلاً برای بهبود عملکرد شبکه).
- **حالت‌ها**:
  - `requiredDuringSchedulingIgnoredDuringExecution`: پاد حتماً باید روی نودهایی اجرا شود که پادهای مشخص دیگر حضور دارند.
  - `preferredDuringSchedulingIgnoredDuringExecution`: ترجیحاً روی نودهایی با پادهای مشخص اجرا شود.

### مثال podAffinity
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      labelSelector:
        matchLabels:
          app: frontend
      topologyKey: kubernetes.io/hostname
```
**توضیح**: این تنظیم باعث می‌شود پاد جدید فقط روی نودی اجرا شود که پادی با برچسب `app: frontend` وجود دارد.

## ۳. podAntiAffinity
- **توضیح**: برعکس podAffinity، مشخص می‌کند که پاد نباید کنار پادهای خاصی قرار بگیرد.
- **کاربرد**: برای توزیع پادها و جلوگیری از تمرکز آن‌ها روی یک نود (مثلاً برای افزایش پایداری).
- **حالت‌ها**:
  - `requiredDuringSchedulingIgnoredDuringExecution`: پاد حتماً نباید روی نودهایی اجرا شود که پادهای مشخص دیگر حضور دارند.
  - `preferredDuringSchedulingIgnoredDuringExecution`: ترجیحاً روی نودهایی بدون پادهای مشخص اجرا شود.

### مثال podAntiAffinity
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      labelSelector:
        matchLabels:
          app: frontend
      topologyKey: kubernetes.io/hostname
```
**توضیح**: این تنظیم باعث می‌شود پاد جدید روی نودی اجرا نشود که پادی با برچسب `app: frontend` وجود دارد.

## جدول جمع‌بندی
| نوع Affinity       | توضیح                              | حالت‌ها                              |
|--------------------|------------------------------------|--------------------------------------|
| **nodeAffinity**   | وابستگی به نود                   | required / preferred                |
| **podAffinity**    | کنار پادهای خاص بودن             | required / preferred                |
| **podAntiAffinity**| دور از پادهای خاص بودن           | required / preferred                |

## Operatorهای قابل استفاده در nodeSelectorTerms
| Operator       | معنی                              |
|----------------|-----------------------------------|
| **In**         | باید در لیست باشد                |
| **NotIn**      | نباید در لیست باشد               |
| **Exists**     | فقط وجود داشته باشد             |
| **DoesNotExist**| نباید وجود داشته باشد           |

## نتیجه
- **Affinity** شامل سه نوع است:
  - `nodeAffinity`
  - `podAffinity`
  - `podAntiAffinity`
- هر کدام دو حالت دارند:
  - `requiredDuringSchedulingIgnoredDuringExecution`
  - `preferredDuringSchedulingIgnoredDuringExecution`

این تنظیمات به شما کمک می‌کنند تا مدیریت دقیق‌تری روی محل اجرای پادها در Kubernetes داشته باشید.
