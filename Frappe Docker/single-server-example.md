### مثال على خادم واحد

في هذا السيناريو، لدينا خادم واحد يحتوي على عنوان IP ثابت مرتبط به. يمكن استخدامه في السيناريوهات حيث يحتوي جهاز الافتراضي الواحد القوي على عدة مقاعد (benches) وتطبيقات أو جهاز افتراضي بمستوى دخول مع موقع واحد. لإعداد مقعد وموقع واحد فقط، اتبع الخطوات حتى تضاف أول مقعد وأول موقع. إذا اخترت هذا الإعداد، يمكنك فقط التوسيع رأسياً. إذا كنت بحاجة إلى التوسيع أفقياً، ستحتاج إلى نسخ مواقع الويب واستعادتها في إعداد العنقود.

سنقوم بإعداد ما يلي:

- تثبيت دوكر ودوكر كومبوز v2 على خادم لينكس.
- تثبيت خدمة تريفيك (Traefik) لتوزيع الأحمال الداخلية والحصول على شهادات SSL/TLS تلقائيًا.
- تثبيت قاعدة بيانات ماريا دي بي باستخدام حاويات (containers).
- إعداد مشروع يسمى `bench-one` وإنشاء مواقع `demo.hamedalrasni.com` و `two.hamedalrasni.com` في المشروع.
- إعداد مشروع يسمى `bench-two` وإنشاء مواقع `three.hamedalrasni.com` و `four.hamedalrasni.com` في المشروع.

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
echo 'TRAEFIK_DOMAIN=demo.hamedalrasni.com' > ~/gitops/demo.env
echo 'EMAIL=demo@hamedalrasni.com' >> ~/gitops/demo.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 changeit | sed 's/\$/\\\$/g') >> ~/gitops/demo.env
```

ملاحظة:

- قم بتغيير النطاق من `traefik.hamedalrasni.com` إلى النطاق المستخدم في الإنتاج. يجب أن يشير إدخال DNS إلى عنوان IP للخادم.
- قم بتغيير عنوان البريد الإلكتروني للإخطارات من `admin@hamedalrasni.com` إلى البريد الإلكتروني الصحيح.
- قم بتغيير كلمة المرور من `changeit` إلى كلمة مرور أكثر أمانًا.

سيبدو ملف env الذي تم إنشاؤه في المسار `~/gitops/traefik.env` كالتالي:

```env
TRAEFIK_DOMAIN=traefik.hamedalrasni.com
EMAIL=admin@hamedalrasni.com
HASHED_PASSWORD=$apr1$K.4gp7RT$tj9R2jHh0D4Gb5o5fIAzm/
```

إذا لم يتم نشر الحاوية، ضع HASHED_PASSWORD بين ''.

نشر حاوية traefik مع SSL من letsencrypt

```shell
docker compose --project-name demo \
  --env-file ~/gitops/demo.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml up -d
```

سيجعل ذلك لوحة تحكم traefik متاحة على `demo.hamedalrasni.com` وسيتم تخزين جميع الشهادات في حجم دوكر `cert-data`.

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

إنشاء المقعد الأول المسمى `bench-one` مع `demo.hamedalrasni.com` و `dev.hamedalrasni.com`

أنشئ ملفًا يُسمى `bench-one.env` في `~/gitops`

```shell
cp example.env ~/gitops/bench-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=qeMuFqdPhmfpfNYm/g' ~/gitops/bench-one.env
sed -i 's/LETSENCRYPT_EMAIL=mail@example.com/LETSENCRYPT_EMAIL=demo@hamedalrasni.com/g' ~/gitops/bench-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/bench-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/bench-one.env
sed -i 's/SITES=`erp.example.com`/SITES=\`demo.hamedalrasni.com\`,\`dev.hamedalrasni.com\`/g' ~/gitops/bench-one.env
echo 'ROUTER=bench-one' >> ~/gitops/bench-one.env
echo "BENCH_NETWORK=bench-one" >> ~/gitops/bench-one.env
```

ملاحظة:

- قم بتغيير كلمة المرور من `changeit` إلى تلك التي تم تعيينها لماريا دي بي كومبوز في الخطوة السابقة.

يتم إنشاء ملف env في المسار `~/gitops/bench-one.env`.

أنشئ ملف yaml يُسمى `bench-one.yaml` في دليل `~/gitops`:

```shell
docker compose --project-name bench-one \
  --env-file ~/gitops/bench-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/bench-one.yaml
```

لإعداد LAN، لا تعدل على `compose.multi-bench-ssl.yaml`.

استخدم الأمر أعلاه بعد أي تغييرات تتم على ملف `bench-one.env` لإعادة إنشاء `~/gitops/bench-one.yaml`. مثال: بعد تغيير الإصدار لتهجير المقعد.

نشر حاويات `bench-one`:

```shell
docker compose --project-name bench-one -f ~/gitops/bench-one.yaml up -d
```

إنشاء المواقع `demo.hamedalrasni.com` و `dev.hamedalrasni.com`:

```shell
# demo.hamedalrasni.com
docker compose --project-name bench-one exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit demo.hamedalrasni.com
```

يمكنك التوقف هنا واكتمال إعداد مقعد واحد وموقع واحد. تابع لإضافة موقع إضافي إلى المقعد الحالي.

```shell
# dev.hamedalrasni.com
docker compose --project-name bench-one exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit dev.hamedalrasni.com
```

#### إنشاء مقعد ثاني

إعداد مقعد إضافي اختياري. استمر فقط إذا كنت بحاجة إلى إعداد مقاعد متعددة.

إنشاء المقعد الثاني المسمى `bench-two` مع `three.hamedalrasni.com` و `ciscoo.hamedalrasni.com`

أنشئ ملفًا يُسمى `bench-two.env` في `~/gitops`

```shell
curl -sL https://raw.githubusercontent.com/frappe/frappe_docker/main/example.env -o ~/gitops/bench-two.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/bench-two.env
sed -i 's/LETSENCRYPT_EMAIL=mail@example.com/LETSENCRYPT_EMAIL=demo@hamedalrasni.com/g' ~/gitops/bench-two.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/bench-two.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/bench-two.env
echo "ROUTER=bench-two" >> ~/gitops/bench-two.env
echo "SITES=\`erp.example.com\`,\`ciscoo.hamedalrasni.com\`" >> ~/gitops/bench-two.env
echo "BENCH_NETWORK=bench-two" >> ~/gitops/bench-two.env
```

ملحوظة:

- قم بتغيير كلمة المرور من `changeit` إلى تلك التي تم تعيينها لماريا دي بي في الخطوة السابقة.

تم إنشاء ملف البيئة في المسار `~/gitops/bench-two.env`.

أنشئ ملفًا yaml يُسمى `bench-two.yaml` في دليل `~/gitops`:

```shell
docker compose --project-name bench-two \
  --env-file ~/gitops/bench-two.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/bench-two.yaml
```

استخدم الأمر أعلاه بعد أي تغييرات تُجرى على ملف `bench-two.env` لإعادة إنشاء `~/gitops/bench-two.yaml`. على سبيل المثال، بعد تغيير الإصدار لترقية المقاعد.

نشر حاويات `bench-two`:

```shell
docker compose --project-name bench-two -f ~/gitops/bench-two.yaml up -d
```

إنشاء المواقع `three.hamedalrasni.com` و `four.hamedalrasni.com`:

```shell
# three.hamedalrasni.com
docker compose --project-name bench-two exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit three.hamedalrasni.com
# four.hamedalrasni.com
docker compose --project-name bench-two exec backend \
  bench new-site --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit four.hamedalrasni.com
```

#### إنشاء نطاق مخصص للموقع الحالي

في حال الحاجة إلى توجيه نطاق مخصص إلى الموقع الحالي، اتبع هذه الخطوات.
كما أنه مفيد أيضًا في حالة الحاجة إلى نطاق مخصص للوصول عبر الشبكة المحلية.

أنشئ ملف بيئة:

```shell
echo "ROUTER=custom-demo-hamedalrasni" > ~/gitops/custom-demo-hamedalrasni.env
echo "SITES=\`custom-demo.hamedalrasni.com\`" >> ~/gitops/custom-demo-hamedalrasni.env
echo "BASE_SITE=demo.hamedalrasni.com" >> ~/gitops/custom-demo-hamedalrasni.env
echo "BENCH_NETWORK=bench-one" >> ~/gitops/custom-demo-hamedalrasni.env
```

ملحوظة:

- قم بتغيير اسم الملف من `custom-demo-hamedalrasni.env` إلى واحد منطقي.
- غير قيمة المتغير `ROUTER` من `custom-demo.hamedalrasni.com` إلى الموجود.
- غير قيمة المتغير `SITES` من `custom-demo.hamedalrasni.com` إلى الموجود. يمكنك إضافة مواقع متعددة مقترنة بعلامات backtick (`) ومفصولة بفواصل.
- غير قيمة المتغير `BASE_SITE` من `demo.hamedalrasni.com` إلى الموجود الذي يُشير إليه.
- غير قيمة المتغير `BENCH_NETWORK` من `bench-one` إلى الموجود الذي تم إنشاؤه مع المقعد.

تم إنشاء ملف البيئة في المسار المذكور في الأمر.

إنشاء yaml للوكيل العكسي:

```shell
docker compose --project-name custom-demo-hamedalrasni \
  --env-file ~/gitops/custom-demo-hamedalrasni.env \
  -f overrides/compose.custom-domain.yaml \
  -f overrides/compose.custom-domain-ssl.yaml config > ~/gitops/custom-demo-hamedalrasni.yaml
```

للإعداد المحلي، لا تستبدل `compose.custom-domain-ssl.yaml`.

نشر حاويات `bench-two`:

```shell
docker compose --project-name custom-demo-hamedalrasni -f ~/gitops/custom-demo-hamedalrasni.yaml up -d
```

### عمليات الموقع

انظر: [عمليات الموقع](./site-operations.md)
