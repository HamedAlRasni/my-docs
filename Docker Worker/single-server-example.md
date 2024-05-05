### مثال على خادم واحد

في هذا السيناريو، لدينا خادم واحد يحتوي على عنوان IP ثابت مرتبط به. يمكن استخدامه في السيناريوهات حيث يحتوي جهاز الافتراضي الواحد القوي على عدة مقاعد (benches) وتطبيقات أو جهاز افتراضي بمستوى دخول مع موقع واحد. لإعداد مقعد وموقع واحد فقط، اتبع الخطوات حتى تضاف أول مقعد وأول موقع. إذا اخترت هذا الإعداد، يمكنك فقط التوسيع رأسياً. إذا كنت بحاجة إلى التوسيع أفقياً، ستحتاج إلى نسخ مواقع الويب واستعادتها في إعداد العنقود.

سنقوم بإعداد ما يلي:

- تثبيت دوكر ودوكر كومبوز v2 على خادم لينكس.
- تثبيت خدمة تريفيك (Traefik) لتوزيع الأحمال الداخلية والحصول على شهادات SSL/TLS تلقائيًا.
- تثبيت قاعدة بيانات ماريا دي بي باستخدام حاويات (containers).
- إعداد مشروع يسمى `erpnext-one` وإنشاء مواقع `one.example.com` و `two.example.com` في المشروع.
- إعداد مشروع يسمى `erpnext-two` وإنشاء مواقع `three.example.com` و `four.example.com` في المشروع.

توضيح:

سيتم تثبيت مثيل واحد من **Traefik** وسيعمل كتوزيع الأحمال الداخلية لعدة مقاعد (benches) ومواقع مستضافة على الخادم. يمكنه أيضًا توزيع الأحمال لتطبيقات أخرى بجانب مقاعد فرابيه، مثل ووردبريس وميتابيس، إلخ. نكشف فقط عن المنافذ `80` و `443` مرة واحدة مع هذا المثيل من Traefik. سيهتم Traefik أيضًا بأتمتة توزيع شهادات letsencrypt لجميع المواقع المثبتة على الخادم. _لماذا اختيار Traefik على Nginx Proxy Manager؟_ Traefik لا يحتاج إلى خدمة قاعدة بيانات إضافية ويمكنه تخزين الشهادات في ملف json في حجم التخزين.

سيتم تثبيت مثيل واحد من **MariaDB** وسيعمل كخدمة قاعدة بيانات لجميع المقاعد/المشاريع المثبتة على الخادم.

كل مثيل من مشروع ERPNext (bench) سيحتوي على خادم redis، socketio، gunicorn، nginx، workers، وجدول زمني. سيتصل بقاعدة بيانات MariaDB الداخلية عن طريق الاتصال بشبكة MariaDB. سيعرض المواقع للجمهور من خلال Traefik عن طريق الاتصال بشبكة Traefik.

### تثبيت دوكر

أسهل طريقة لتثبيت دوكر هي استخدام [النص المريح](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script).

```shell
curl -fsSL https://get.docker.com | bash
```

ملاحظة: يفترض الوثائق استخدام خادم Ubuntu LTS. استخدم أي توزيعة طالما يعمل النص المريح لدوكر. إذا لم يعمل النص المريح، ستحتاج إلى تثبيت دوكر يدوياً.

### تثبيت Compose V2

راجع [الوثائق الأصلية](https://docs.docker.com/compose/cli-command/#install-on-linux) للإصدار المحدث.

```shell
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

### التحضير

انسخ مستودع `frappe_docker` للحصول على ملفات YAML المطلوبة وقم بتغيير دليل العمل الحالي للشل إلى المستودع المُنسخ.

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

أنشئ مجلدًا لتخزين ملفات التكوين والموارد

```shell
mkdir ~/gitops
```

سيقوم المجلد `~/gitops` بتخزين جميع الموارد التي نستخدمها للإعداد. سنحتفظ أيضًا بملفات البيئة في هذا المجلد حيث ستكون هناك مشاريع متعددة تحتوي على متغيرات بيئية مختلفة. يمكنك إنشاء مستودع خاص لهذا المجلد وتتبع التغييرات هناك.

### تثبيت Traefik

إعداد أساسي لـ Traefik باستخدام دوكر كومبوز.

أنشئ ملفًا يُسمى `traefik.env` في `~/gitops`

```shell
echo 'TRAEFIK_DOMAIN=traefik.example.com' > ~/gitops/traefik.env
echo 'EMAIL=admin@example.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 changeit | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
```

ملاحظة:

- قم بتغيير النطاق من `traefik.example.com` إلى النطاق المستخدم في الإنتاج. يجب أن يشير إدخال DNS إلى عنوان IP للخادم.
- قم بتغيير عنوان البريد الإلكتروني للإخطارات من `admin@example.com` إلى البريد الإلكتروني الصحيح.
- قم بتغيير كلمة المرور من `changeit` إلى كلمة مرور أكثر أمانًا.

سيبدو ملف env الذي تم إنشاؤه في المسار `~/gitops/traefik.env` كالتالي:

```env
TRAEFIK_DOMAIN=traefik.example.com
EMAIL=admin@example.com
HASHED_PASSWORD=$apr1$K.4gp7RT$tj9R2jHh0D4Gb5o5fIAzm/
```

إذا لم يتم نشر الحاوية، ضع HASHED_PASSWORD بين ''.

نشر حاوية traefik مع SSL من letsencrypt

```shell
docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml up -d
```

سيجعل ذلك لوحة تحكم traefik متاحة على `traefik.example.com` وسيتم تخزين جميع الشهادات في حجم دوكر `cert-data`.

لإعداد LAN، نشر حاوية traefik بدون تجاوز `overrides/compose.traefik-ssl.yaml`.

### تثبيت MariaDB

إعداد MariaDB الأساسي باستخدام دوكر كومبوز.

أنشئ ملفًا يُسمى `mariadb.env` في `~/gitops`

```shell
echo "DB_PASSWORD=changeit" > ~/gitops/mariadb.env
```

ملاحظة:

- قم بتغيير كلمة المرور من `changeit` إلى كلمة مرور أكثر أمانًا.

سيبدو ملف env الذي تم إنشاؤه في المسار `~/gitops/mariadb.env` كالتالي:

```env
DB_PASSWORD=changeit
```

ملاحظة: قم بتغيير كلمة المرور من `changeit` إلى كلمة مرور أكثر أمانًا.

نشر حاوية mariadb

```shell
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
```

سيجعل هذا الخدمة `mariadb-database` متاحة تحت `mariadb-network`. ستتم تخزين البيانات في `/data/mariadb`.

### تثبيت ERPNext

#### إنشاء المقاعد الأولى

إنشاء المقعد الأول المسمى `erpnext-one` مع `one.example.com` و `two.example.com`

أنشئ ملفًا يُسمى `erpnext-one.env` في `~/gitops`

```shell
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
sed -i 's/SITES=`erp.example.com`/SITES=\`one.example.com\`,\`two.example.com\`/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```

ملاحظة:

- قم بتغيير كلمة المرور من `changeit` إلى تلك التي تم تعيينها لماريا دي بي كومبوز في الخطوة السابقة.

يتم إنشاء ملف env في المسار `~/gitops/erpnext-one.env`.

أنشئ ملف yaml يُسمى `erpnext-one.yaml` في دليل `~/gitops`:

```shell
docker compose --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-one.yaml
```

لإعداد LAN، لا تعدل على `compose.multi-bench-ssl.yaml`.

استخدم الأمر أعلاه بعد أي تغييرات تتم على ملف `erpnext-one.env` لإعادة إنشاء `~/gitops/erpnext-one.yaml`. مثال: بعد تغيير الإصدار لتهجير المقعد.

نشر حاويات `erpnext-one`:

```shell
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```

إنشاء المواقع `one.example.com` و `two.example.com`:

```shell
# one.example.com
docker compose --project-name erpnext-one exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit one.example.com
```

يمكنك التوقف هنا واكتمال إعداد مقعد واحد وموقع واحد. تابع لإضافة موقع إضافي إلى المقعد الحالي.

```shell
# two.example.com
docker compose --project-name erpnext-one exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit two.example.com
```

#### إنشاء مقعد ثاني

إعداد مقعد إضافي اختياري. استمر فقط إذا كنت بحاجة إلى إعداد مقاعد متعددة.

إنشاء المقعد الثاني المسمى `erpnext-two` مع `three.example.com` و `four.example.com`

أنشئ ملفًا يُسمى `erpnext-two.env` في `~/gitops`

```shell
curl -sL https://raw.githubusercontent.com/frappe/frappe_docker/main/example.env -o ~/gitops/erpnext-two.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-two.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-two.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-two.env
echo "ROUTER=erpnext-two" >> ~/gitops/erpnext-two.env
echo "SITES=\`three.example.com\`,\`four.example.com\`" >> ~/gitops/erpnext-two.env
echo "BENCH_NETWORK=erpnext-two" >> ~/gitops/erpnext-two.env
```

ملحوظة:

- قم بتغيير كلمة المرور من `changeit` إلى تلك التي تم تعيينها لماريا دي بي في الخطوة السابقة.

تم إنشاء ملف البيئة في المسار `~/gitops/erpnext-two.env`.

أنشئ ملفًا yaml يُسمى `erpnext-two.yaml` في دليل `~/gitops`:

```shell
docker compose --project-name erpnext-two \
  --env-file ~/gitops/erpnext-two.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-two.yaml
```

استخدم الأمر أعلاه بعد أي تغييرات تُجرى على ملف `erpnext-two.env` لإعادة إنشاء `~/gitops/erpnext-two.yaml`. على سبيل المثال، بعد تغيير الإصدار لترقية المقاعد.

نشر حاويات `erpnext-two`:

```shell
docker compose --project-name erpnext-two -f ~/gitops/erpnext-two.yaml up -d
```

إنشاء المواقع `three.example.com` و `four.example.com`:

```shell
# three.example.com
docker compose --project-name erpnext-two exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit three.example.com
# four.example.com
docker compose --project-name erpnext-two exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit four.example.com
```

#### إنشاء نطاق مخصص للموقع الحالي

في حال الحاجة إلى توجيه نطاق مخصص إلى الموقع الحالي، اتبع هذه الخطوات.
كما أنه مفيد أيضًا في حالة الحاجة إلى نطاق مخصص للوصول عبر الشبكة المحلية.

أنشئ ملف بيئة:

```shell
echo "ROUTER=custom-one-example" > ~/gitops/custom-one-example.env
echo "SITES=\`custom-one.example.com\`" >> ~/gitops/custom-one-example.env
echo "BASE_SITE=one.example.com" >> ~/gitops/custom-one-example.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/custom-one-example.env
```

ملحوظة:

- قم بتغيير اسم الملف من `custom-one-example.env` إلى واحد منطقي.
- غير قيمة المتغير `ROUTER` من `custom-one.example.com` إلى الموجود.
- غير قيمة المتغير `SITES` من `custom-one.example.com` إلى الموجود. يمكنك إضافة مواقع متعددة مقترنة بعلامات backtick (`) ومفصولة بفواصل.
- غير قيمة المتغير `BASE_SITE` من `one.example.com` إلى الموجود الذي يُشير إليه.
- غير قيمة المتغير `BENCH_NETWORK` من `erpnext-one` إلى الموجود الذي تم إنشاؤه مع المقعد.

تم إنشاء ملف البيئة في المسار المذكور في الأمر.

إنشاء yaml للوكيل العكسي:

```shell
docker compose --project-name custom-one-example \
  --env-file ~/gitops/custom-one-example.env \
  -f overrides/compose.custom-domain.yaml \
  -f overrides/compose.custom-domain-ssl.yaml config > ~/gitops/custom-one-example.yaml
```

للإعداد المحلي، لا تستبدل `compose.custom-domain-ssl.yaml`.

نشر حاويات `erpnext-two`:

```shell
docker compose --project-name custom-one-example -f ~/gitops/custom-one-example.yaml up -d
```

### عمليات الموقع

انظر: [عمليات الموقع](./site-operations.md)