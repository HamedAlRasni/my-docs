# إعداد الإنتاج في حاوية

تأكد من أنك قمت بنسخ هذا المستودع وتبديل الدليل قبل تنفيذ الأوامر التالية.

الأوامر ستولد ملف YAML وفقًا للبيئة للإعداد.

## المتطلبات الأساسية

- [دوكر](https://docker.com/get-started)
- [دوكر كومبوز v2](https://docs.docker.com/compose/cli-command)

## إعداد متغيرات البيئة

انسخ ملف البيئة دوكر المثال إلى `.env`:

```sh
cp example.env .env
```

ملاحظة: لمعرفة المزيد حول متغيرات البيئة [اقرأ هنا](./environment-variables.md). ضع المتغيرات اللازمة في ملف `.env`.

## توليد docker-compose.yml لمجموعة متنوعة من الإعدادات

ملاحظات:

- تأكد من استبدال `<project-name>` بالاسم المطلوب الذي ترغب في تعيينه للمشروع.
- هذا الإعداد لا يجب استخدامه للتطوير. بيئة تطوير كاملة متاحة [هنا](../development)

### تخزين ملفات yaml

يمكن تخزين ملفات YAML التي تم إنشاؤها بواسطة أمر `docker compose config` في دليل. سنقوم بإنشاء دليل يسمى `gitops` في دليل المستخدم.

```shell
mkdir ~/gitops
```

يمكنك تحويل الدليل إلى مستودع git خاص يخزن yaml والأسرار. يمكن أن يساعد في تتبع التغييرات.

بدلاً من `docker compose config`، يمكنك استخدام `docker compose up` مباشرة لبدء الحاويات وتخطي تخزين ملفات yaml في الدليل `gitops`.

### إعداد Frappe بدون بروكسي و MariaDB و Redis الخارجية

في هذه الحالة، تأكد من تعيين المتغيرات البيئية `DB_HOST`، `DB_PORT`، `REDIS_CACHE`، و `REDIS_QUEUE` أو سيفشل `configurator`.

```sh
# توليد YAML
docker compose -f compose.yaml -f overrides/compose.noproxy.yaml config > ~/gitops/docker-compose.yml

# بدء الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml up -d
```

### إعداد ERPNext مع بروكسي و MariaDB و Redis الخارجية

في هذه الحالة، تأكد من تعيين المتغيرات البيئية `DB_HOST`، `DB_PORT`، `REDIS_CACHE`، و `REDIS_QUEUE` أو سيفشل `configurator`.

```sh
# توليد YAML
docker compose -f compose.yaml \
  -f overrides/compose.proxy.yaml \
  config > ~/gitops/docker-compose.yml

# بدء الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml up -d
```

### إعداد Frappe باستخدام MariaDB و Redis المحايدة في حاويات مع شهادات Letsencrypt.

في هذه الحالة، تأكد من تعيين المتغيرات البيئية `LETSENCRYPT_EMAIL` و `SITES` أو لن تعمل الشهادات.

```sh
# توليد YAML
docker compose -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  config > ~/gitops/docker-compose.yml

# بدء الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml up -d
```

### إعداد ERPNext باستخدام MariaDB و Redis المحايدة في حاو

يات مع شهادات Letsencrypt.

في هذه الحالة، تأكد من تعيين المتغيرات البيئية `LETSENCRYPT_EMAIL` و `SITES` أو لن تعمل الشهادات.

```sh
# توليد YAML
docker compose -f compose.yaml \
  -f overrides/compose.mariadb.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.https.yaml \
  config > ~/gitops/docker-compose.yml

# بدء الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml up -d
```

## إنشاء الموقع الأول

بعد بدء الحاويات، يجب إنشاء الموقع الأول. راجع [عمليات الموقع](./site-operations.md#setup-new-site).

## تحديث الصور

انتقل إلى الجذر من دليل `frappe_docker` قبل تشغيل الأوامر التالية:

```sh
# تحديث متغيرات البيئة ERPNEXT_VERSION و FRAPPE_VERSION
nano .env

# سحب صور جديدة
docker compose -f compose.yaml \
  # ... الاستبدالات الأخرى الخاصة بك
  config > ~/gitops/docker-compose.yml

# سحب الصور
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml pull

# إيقاف الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml down

# إعادة تشغيل الحاويات
docker compose --project-name <project-name> -f ~/gitops/docker-compose.yml up -d
```

ملاحظة:

- يمكن تخطي أوامر سحب وإيقاف الحاويات إذا تم استخدام علامات الصور الثابتة.
- سيقوم `docker compose up -d` بسحب علامات ثابتة جديدة إذا لم تتوفر.

للمزيد من المعلومات حول هجرة المواقع، راجع [عمليات الموقع](./site-operations.md#migrate-site)