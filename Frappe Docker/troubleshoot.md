1. [إصلاح مشكلات MariaDB بعد إعادة بناء الحاوية](#إصلاح-مشكلات-mariadb-بعد-إعادة-بناء-الحاوية)
1. [docker-compose لا يتعرف على المتغيرات من ملف `.env`](#docker-compose-لا-يتعرف-على-المتغيرات-من-ملف-env)
1. [تثبيت مبني على نظام ويندوز](#تثبيت-مبني-على-نظام-ويندوز)

### إصلاح مشكلات MariaDB بعد إعادة بناء الحاوية

في أي حالة بعد إعادة بناء الحاوية إذا لم تتمكن من الوصول إلى MariaDB بشكل صحيح باستخدام التكوين السابق. اتبع هذه التعليمات.

يجب تعيين المعلمة `'db_name'@'%'` في MariaDB وتخصيص الإذن لقاعدة البيانات للمستخدم بشكل مناسب.

يجب تكرار هذه الخطوة لجميع المواقع المتاحة تحت Bench الحالي.
يظهر المثال الاستعلامات التي يجب تنفيذها للموقع `localhost`

افتح ملف `sites/localhost/site_config.json`:

```shell
code sites/localhost/site_config.json
```

وقم بتدوين المعلمات `db_name` و `db_password`.

ادخل إلى محطة MariaDB التفاعلية:

```shell
mysql -uroot -p123 -hmariadb
```

قم بتنفيذ الاستعلامات التالية مع استبدال `db_name` و `db_password` بالقيم الموجودة في site_config.json.

```sql
UPDATE mysql.user SET Host = '%' where User = 'db_name'; FLUSH PRIVILEGES;
SET PASSWORD FOR 'db_name'@'%' = PASSWORD('db_password'); FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON `db_name`.* TO 'db_name'@'%'; FLUSH PRIVILEGES;
EXIT;
```

ملاحظة: للإصدارات 10.4 وما فوق لـ MariaDB استخدم `mysql.global_priv` بدلاً من `mysql.user`.

### docker-compose لا يتعرف على المتغيرات من ملف .env

إذا كنت تستخدم إصدارًا قديمًا من `docker-compose` فإن ملف .env يجب أن يكون موجودًا في الدليل الذي تتم منه تنفيذ أمر docker-compose. قد يكون هناك أيضًا فرق في `docker-compose` الرسمي والمعبأ من قبل التوزيع. استخدم `--env-file=.env` إذا كان متاحًا لتحديد المسار بوضوح إلى الملف.

### تثبيت مبني على نظام ويندوز

- قم بتعيين المتغير `COMPOSE_CONVERT_WINDOWS_PATHS` في البيئة، على سبيل المثال: `set COMPOSE_CONVERT_WINDOWS_PATHS=1`
- أثناء استخدام الدوكر ماشين، قم بتوجيه المنافذ من الآلة الظاهرية إلى منافذ الجهاز المضيف. (المنافذ 8080/8000/9000)
- قم بتسمية جميع المواقع التي تنتهي بـ `.localhost`. والوصول إليها عبر المتصفح محليًا. على سبيل المثال: `http://site1.localhost`

### إعادة التثبيت

- إذا قمت بإجراء تغييرات وترغب فقط في البدء من جديد (بالتخلي عن جميع التغييرات)، قم بإزالة جميع
  - containers
  - images
  - volumes
- Install a fresh
