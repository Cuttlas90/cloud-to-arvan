# Remote Downloader to S3 / Arvan Object Storage

> [English](README.md) | 📖 فارسی

دانلود فایل‌های حجیم روی GitHub Actions و انتقال خودکار به Object Storage سازگار با S3 (مانند ابرآروان)

---

# چرا این پروژه ساخته شد؟

در بعضی کشورها یا شرایط شبکه، دانلود مستقیم فایل‌های حجیم از اینترنت بین‌المللی:

- بسیار کند است
- ناپایدار است
- نیاز به VPN/VPS دارد
- یا هزینه بالایی دارد

این پروژه تلاش می‌کند با استفاده از GitHub Actions، دانلود فایل را روی زیرساخت GitHub انجام دهد و فایل را به Object Storage شخصی کاربر منتقل کند.

---

# معماری پروژه

```text
Internet
   ↓
GitHub Actions
   ↓
Chunked Upload
   ↓
S3-Compatible Object Storage
(Arvan / MinIO / ...)
```

---

# ویژگی‌ها

- دانلود فایل روی GitHub Actions
- پشتیبانی از لینک مستقیم
- آپلود به Object Storage سازگار با S3
- تقسیم فایل به پارت‌های کوچک
- قابلیت Resume
- جلوگیری از upload مجدد پارت‌های آپلودشده
- مقاوم در برابر timeout شش‌ساعته GitHub Actions
- مناسب فایل‌های بسیار حجیم
- بدون نیاز به VPS

---

# چرا فایل Chunk می‌شود؟

GitHub Actions دارای محدودیت زمانی است و معمولا بعد از حدود ۶ ساعت job را متوقف می‌کند.

اگر فایل به‌صورت یکجا upload شود:

- با قطع شدن workflow
- کل upload از بین می‌رود

اما در این پروژه:

```text
8GB file
↓
16 × 512MB chunks
```

هر پارت جداگانه upload می‌شود.

در اجرای بعدی:

- پارت‌های قبلی شناسایی می‌شوند
- فقط پارت‌های باقی‌مانده upload می‌شوند

بنابراین workflow کاملا Resumeable است.

---

# ساختار فایل‌ها داخل Bucket

مثال:

```text
needed_file.iso/
  needed_file.iso.part-00000
  needed_file.iso.part-00001
  needed_file.iso.part-00002
  needed_file.iso.sha256
```

---

# نحوه Assemble فایل

بعد از دانلود تمام پارت‌ها:

```bash
cat needed_file.iso.part-* > needed_file.iso
```

سپس برای بررسی صحت فایل:

```bash
sha256sum -c needed_file.iso.sha256
```

اگر hash صحیح باشد:

- فایل کاملا سالم reconstruct شده است.

---

# راه‌اندازی

## 1. Fork Repository

ابتدا repository را fork کنید.

---

## 2. ساخت Object Storage

این پروژه با هر storage سازگار با S3 کار می‌کند:

- Arvan Object Storage
- MinIO
- AWS S3
- Backblaze B2
- Cloudflare R2

---

## 3. ساخت Secrets در GitHub

مسیر:

```text
Repository → Settings → Secrets and variables → Actions
```

Secrets مورد نیاز:

```text
ARVAN_ACCESS_KEY
ARVAN_SECRET_KEY
ARVAN_BUCKET
ARVAN_ENDPOINT
```

---

# اجرای دانلود

مسیر:

```text
Actions → Download and Upload to Arvan → Run workflow
```

پارامترها:

| پارامتر | توضیح |
|---|---|
| url | لینک فایل |
| filename | نام فایل خروجی |
| chunk_size | اندازه هر پارت (مثلا 512M) |

---

# Resume Upload

اگر workflow:

- متوقف شود
- timeout شود
- fail شود

کافی است دوباره همان workflow را Re-Run کنید.

پارت‌های قبلی:

- دوباره upload نمی‌شوند
- شناسایی و skip می‌شوند

---

# محدودیت‌ها

- GitHub Actions برای استفاده شخصی و معقول طراحی شده است.
- این پروژه نباید برای abuse، scraping سنگین، یا mirror عمومی عظیم استفاده شود.
- upload به زیرساخت‌های داخل ایران ممکن است کند باشد.
- Cloudflare و بعضی providerها ممکن است در بعضی ISPها در دسترس نباشند.

## استفاده مسئولانه

این پروژه برای:

- دانلود قانونی فایل‌هایی که اجازه دسترسی به آن‌ها را دارید
- انتقال فایل به storage شخصی کاربر
- استفاده شخصی، تحقیقاتی، آموزشی و آرشیوی

طراحی شده است.

---

## نکات مهم

- این پروژه یک public relay service نیست.
- هر کاربر باید از storage و GitHub account خود استفاده کند.
- استفاده سنگین، abusive، یا automated mass scraping ممکن است باعث محدودیت GitHub Actions شود.
- مسئولیت استفاده از این پروژه بر عهده کاربر است.
- نویسنده پروژه مسئول استفاده غیرقانونی یا ناقض قوانین providerها نیست.

---

## استفاده از GitHub Actions

GitHub Actions برای CI/CD طراحی شده است.

این پروژه تلاش می‌کند:

- استفاده محافظه‌کارانه
- resumable uploads
- chunked transfers
- و جلوگیری از مصرف غیرمنطقی منابع

را رعایت کند.

در صورت استفاده سنگین یا غیرمعمول:

- ممکن است GitHub workflowها را محدود کند
- یا account را rate-limit کند

# GitHub Timeout

GitHub-hosted runners معمولا دارای timeout حدود ۶ ساعت هستند.

این پروژه با استفاده از:

- chunked uploads
- resumable uploads
- part detection

این محدودیت را دور می‌زند.

---

# امنیت

هرگز:

- Access Key اصلی خود را public نکنید.
- secretها را داخل repository قرار ندهید.

از GitHub Secrets استفاده کنید.

---

# ایده اصلی پروژه

این پروژه تلاش می‌کند:

- بدون VPS
- بدون سرور دائمی
- بدون relay عمومی
- بدون نقض قوانین providerها

امکان انتقال فایل‌های حجیم را برای کاربران فراهم کند.

---

# License

MIT
