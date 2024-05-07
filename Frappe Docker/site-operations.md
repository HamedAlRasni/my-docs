# عمليات الموقع

> 💡 يجب عليك إعداد خيار `--project-name` في أوامر `docker-compose` إذا كان لديك اسم مشروع غير قياسي.

## إعداد موقع جديد

ملاحظة:

- انتظر حتى تبدأ خدمة `db` وتنتهي `configurator` قبل محاولة إنشاء موقع جديد. عادةً ما يستغرق هذا ما يصل إلى 10 ثوانٍ.

```sh
docker-compose exec backend bench new-site --no-mariadb-socket --mariadb-root-password <db-password> --admin-password <admin-password> <site-name>
```

إذا كنت بحاجة إلى تثبيت تطبيق معين، فحدد `--install-app`. لعرض جميع الخيارات، قم بتشغيل `bench new-site --help`.

لإنشاء موقع Postgres (من المفترض أن تكون قد استخدمت بالفعل [Postgres compose override](images-and-compose-files.md#overrides))، يجب عليك تعيين `root_login` و `root_password` في التكوين المشترك قبل ذلك:

```sh
docker-compose exec backend bench set-config -g root_login <root-login>
docker-compose exec backend bench set-config -g root_password <root-password>
```

أيضًا يتم التحكم بالأمر بشكل بسيط:

```sh
docker-compose exec backend bench new-site --no-mariadb-socket --db-type postgres --admin-password <admin-password> <site-name>
```

## رفع النسخ الاحتياطي إلى تخزين S3

لدينا سكريبت يساعد في رفع النسخة الاحتياطية الأحدث إلى S3.

```sh
docker-compose exec backend push_backup.py --site-name <site-name> --bucket <bucket> --region-name <region> --endpoint-url <endpoint-url> --aws-access-key-id <access-key> --aws-secret-access-key <secret-key>
```

لاحظ أنه يمكنك استعادة النسخ الاحتياطية يدويًا فقط.

## تحرير التكوينات

قد يكون من الضروري تحرير التكوين يدويًا في بعض الحالات،
أحد هذه الحالات هو استخدام Amazon RDS (أو أي قاعدة بيانات كخدمة). للحصول على تعليمات كاملة، راجع [الويكي](<https://github.com/frappe/frappe/wiki/Using-Frappe-with-Amazon-RDS-(or-any-other-DBaaS)>). يمكن العثور على الأسئلة الشائعة في القضايا وعلى المنتدى.

يجب تحرير `common_site_config.json` أو `site_config.json` من حجم `sites` باستخدام الأمر التالي:

```sh
docker run --rm -it \
    -v <project-name>_sites:/sites \
    alpine vi /sites/common_site_config.json
```

استبدل `alpine` بأي صورة تختارها بدلاً من ذلك.

## فحص الصحة

بالنسبة لخدمة socketio و gunicorn، قم بعمل ping على الاسم المضيف:المنفذ وسيكون هذا كافيًا. بالنسبة للعمال والمجدول، هناك أمر يجب تنفيذه.

```shell
docker-compose exec backend healthcheck.sh --ping-service mongodb:27017
```

يمكن أيضًا عمل ping للخدمات الإضافية كجزء من فحص الصحة باستخدام الخيار `-p` أو `--ping-service`.

يتأكد هذا الفحص من أن الخدمة المعطاة يجب أن تكون متصلة جنبًا إلى جنب مع الخدمات في common_site_config.json.
إذا فشلت الاتصال بالخدمة (أو الخدمات)، فإن الأمر يفشل برمز الخروج 1.

---

للإشارة إلى الأوامر مثل `backup`، `drop-site` أو `

migrate`، راجع [الدليل الرسمي](https://frappeframework.com/docs/v13/user/en/bench/frappe-commands) أو قم بتشغيل:

```sh
docker-compose exec backend bench --help
```

## ترحيل الموقع

ملاحظة:

- انتظر حتى تبدأ خدمة `db` وتنتهي `configurator` قبل محاولة ترحيل الموقع. عادةً ما يستغرق هذا ما يصل إلى 10 ثوانٍ.

```sh
docker-compose exec backend bench --site <site-name> migrate
```