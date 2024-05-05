# التطوير

## البدء

## المتطلبات الأولية

للبدء في التطوير، تحتاج إلى توفير المتطلبات الأولية التالية:

- Docker
- docker-compose
- المستخدم مضاف إلى مجموعة docker

يُوصى بتخصيص ما لا يقل عن 4 غيغابايت من ذاكرة الوصول العشوائي (RAM) لـ Docker:

- [إرشادات لنظام Windows](https://docs.docker.com/docker-for-windows/#resources)
- [إرشادات لنظام macOS](https://docs.docker.com/desktop/settings/mac/#advanced)

هنا صورة شاشة تُظهر الإعدادات ذات الصلة في دليل المساعدة
![صورة](images/Docker%20Manual%20Screenshot%20-%20Resources%20section.png)

هنا صورة شاشة تُظهر الإعدادات في Docker Desktop على نظام macOS
![صور](images/Docker%20Desktop%20Screenshot%20-%20Resources%20section.png)

## تهيئة حاويات التطوير

استنسخ وغير الدليل إلى دليل frappe_docker

```shell
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

انسخ تكوين devcontainer المثالي من `devcontainer-example` إلى `.devcontainer`

```shell
cp -R devcontainer-example .devcontainer
```

انسخ تكوين vscode المثالي لـ devcontainer من `development/vscode-example` إلى `development/.vscode`. سيتم إعداد التكوين الأساسي للتصحيح.

```shell
cp -R development/vscode-example development/.vscode
```

## استخدام ملحقة VSCode Remote Containers

بالنسبة لمعظم الأشخاص الذين يبدؤون في تطوير Frappe، أفضل حل هو استخدام [ملحقة VSCode Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

قبل فتح المجلد في الحاوية، قم بتحديد قاعدة البيانات التي تريد استخدامها. الافتراضي هو MariaDB. إذا كنت ترغب في استخدام PostgreSQL بدلاً من ذلك، قم بتحرير `.devcontainer/docker-compose.yml` وألغِ تعليق القسم الخاص بالخدمة `postgresql`، وقد ترغب أيضًا في تعليق `mariadb` أيضًا.

يجب أن تقوم ملحقة VSCode تلقائيًا بطلب تثبيت الامتدادات المطلوبة، يمكن أيضًا تثبيتها يدويًا كما يلي:

- تثبيت Remote - Containers for VSCode
  - من خلال سطر الأوامر `code --install-extension ms-vscode-remote.remote-containers`
  - بالنقر فوق زر التثبيت في سوق فيسوال ستوديو: [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  - View: Extensions command في VSCode (Windows: Ctrl+Shift+X؛ macOS: Cmd+Shift+X) ثم البحث عن امتداد `ms-vscode-remote.remote-containers`

بعد تثبيت الامتدادات، يمكنك:

- فتح مجلد frappe_docker في VS Code.
  - `code .`
- تشغيل الأمر، من Command Palette (Ctrl + Shift + P) `Remote-Containers: Reopen in Container`. يمكنك أيضًا النقر في الزاوية السفلية اليسرى للوصول إلى قائمة الحاوية البعيدة.

ملاحظات:

- تتجاهل المجلد `development` من git. يتم تثبيته وتوفره داخل الحاوية. قم بإنشاء جميع مقاعد البنش (تثبيتات البنش، الأداة التي تدير Frappe) داخل هذا الدليل.
- يتم تثبيت Node v14 و v10. تحقق باستخدام `nvm ls`. يتم استخدام Node v14 افتراضيًا.

### إعداد مقعد البنش الأول

> انتقل إلى [السكربتات](#setup-bench--new-site-using-script) لإعداد مقعد بشكل تلقائي. بديلًا عن ذلك، يمكنك إعداد مقعد بشكل يدوي باستخدام الدليل أدناه.

قم بتشغيل الأوامر التالية في الطرفية داخل الحاوية. قد تحتاج إلى إنشاء طرفية جديدة في VSCode.

ملاحظة: قبل القيام بما يلي، تأكد من أن المستخدم هو **frappe**.

```shell
bench init --skip-redis-config-generation frappe-bench
cd frappe-bench
```

لإعداد مقعد بنش لإصدار 14 من إطار عمل frappe، ضع المتغير البيئي `PYENV_VERSION` إلى `3.10.5` (افتراضي) واستخدم إصدار NodeJS 16 (افتراضي)،

```shell
# استخدام بيئات افتراضية
bench init --skip-redis-config-generation --frappe-branch version-14 frappe-bench
# أو ضبط إصدارات البيئة بشكل صريح
nvm use v16
PYENV_VERSION=3.10.13 bench init --skip-redis-config-generation --frappe-branch version-14 frappe-bench
# التبديل إلى الدليل
cd frappe-bench
```

لإعداد مقعد بنش لإصدار 13 من إطار عمل frappe، ضع المتغير البيئي `PYENV_VERSION` إلى `3.9.17` واستخدم إصدار NodeJS 14،

```shell
nvm use v14
PYENV_VERSION=3.9.17 bench init --skip-redis-config-generation --frappe-branch version-13 frappe-bench
cd frappe-bench
```

### إعداد المضيفين

نحتاج إلى إخبار بينش باستخدام الحاويات الصحيحة بدلاً من localhost. قم بتشغيل الأوامر التالية داخل الحاوية:

```shell
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

في حالة فشل الأوامر أعلاه لأي سبب من الأسباب، قم بتعيين القيم في `common_site_config.json` يدويًا.

```json
{
  "db_host": "mariadb",
  "redis_cache": "redis://redis-cache:6379",
  "redis_queue": "redis://redis-queue:6379",
  "redis_socketio": "redis://redis-queue:6379"
}
```

### تحرير ملف Procfile لـ Honcho

ملاحظة: مع الخيار '--skip-redis-config-generation' أثناء bench init، لم يعد هذه الإجراءات مطلوبة. ولكن على الأقل، قم بمراجعة ملف ProcFile لرؤية ما يحدث عندما يقوم bench بتشغيل honcho عند بدء الأمر.

Honcho هو الأداة المستخدمة بواسطة Bench لإدارة جميع العمليات التي يتطلبها Frappe. عادةً ما يتم تشغيل كل هذه العمليات في localhost، ولكن في هذه الحالة، لدينا حاويات خارجية لـ Redis. لهذا السبب، علينا أن نمنع Honcho من محاولة بدء عمليات Redis.

يتم تثبيت Honcho في بيئة Python العالمية إلى جانب bench. لجعله متاحًا محليًا، يجب عليك تثبيته في كل مجلد `frappe-bench/env` تقوم بإنشائه. قم بتثبيته باستخدام الأمر `./env/bin/pip install honcho`. إنه مطلوب محليًا إذا كنت ترغب في استخدامه كجزء من تكوين الإطلاق في VSCode.

افتح ملف Procfile وقم بإزالة الأسطر الثلاث التي تحتوي على التكوين من Redis، سواءً عن طريق تحرير الملف يدويًا:

```shell
code Procfile
```

أو تشغيل الأمر التالي:

```shell
sed -i '/redis/d' ./Procfile
```

### إنشاء موقع جديد باستخدام bench

يمكنك إنشاء موقع جديد باستخدام الأمر التالي:

```shell
bench new-site --no-mariadb-socket sitename
```

يجب أن ينتهي اسم الموقع بـ .localhost لتجربة النشر محليًا.

على سبيل المثال:

```shell
bench new-site --no-mariadb-socket development.localhost
```

يمكن تشغيل نفس الأمر بشكل غير تفاعلي أيضًا:

```shell
bench new-site --mariadb-root-password 123 --admin-password admin --no-mariadb-socket development.localhost
```

سيطلب الأمر كلمة مرور root لـ MariaDB. كلمة المرور الافتراضية للمستخدم root هي `123`.
سينشأ هذا الموقع الجديد وسيتم إنشاء مجلد `development.localhost` تحت `frappe-bench/sites`.
الخيار `--no-mariadb-socket` سيقوم بتكوين بيانات اعتماد قاعدة البيانات للموقع للعمل مع Docker.
قد تحتاج إلى تكوين ملف /etc/hosts على نظامك إذا كنت تستخدم Linux أو Mac، أو نظام Windows المعادل له.

لإعداد الموقع باستخدام PostgreSQL كقاعدة بيانات، استخدم الخيار `--db-type postgres` و `--db-host postgresql`. (متوفر فقط في الإصدار 12 فما فوق، حاليًا غير متوفر لـ ERPNext).

مثال:

```shell
bench new-site --db-type postgres --db-host postgresql mypgsql.localhost
```

لتجنب إدخال اسم مستخدم وكلمة مرور root لـ postgresql، قم بتعيينه في `common_site_config.json`,

```shell
bench config set-common-config -c root_login postgres
bench config set-common-config -c root_password '"123"'
```

ملاحظة: إذا لم يكن PostgreSQL مطلوبًا، يمكن إيقاف خدمة / حاوية postgresql.

### تعيين وضع تطوير bench على الموقع الجديد

لتطوير تطبيق جديد، سيكون الخطوة الأخيرة هي تعيين الموقع في وضع المطور. الوثائق متوفرة في [هذا الرابط](https://frappe.io/docs/user/en/guides/app-development/how-enable-developer-mode-in-frappe).

```shell
bench --site development.localhost set-config developer_mode 1
bench --site development.localhost clear-cache
```

### تثبيت تطبيق

لتثبيت تطبيق، نحتاج إلى جلبه من مستودع git المناسب، ثم تثبيته على الموقع المناسب:

يمكنك التحقق من [وثائق ملحقة VSCode للحاوية البعيدة](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container) بخصوص مشاركة بيانات اعتماد git.

لتثبيت تطبيق مخصص

```shell
# --branch اختياري، استخدمه للإشارة إلى الفرع على مستودع التطبيق المخصص
bench get-app --branch version-12 https://github.com/myusername/myapp
bench --site development.localhost install-app myapp
```

في وقت كتابة هذا النص، تم استخراج تطبيق المدفوعات من تطبيق ERPNext الإصدار 14 وأصبح الآن تطبيقًا منفصلاً. لن يقوم ERPNext بتثبيته.

```shell
bench get-app --branch version-14 --resolve-deps erpnext
bench --site development.localhost install-app erpnext
```

لتثبيت ERPNext (من الفرع version-13):

```shell
bench get-app --branch version-13 erpnext
bench --site development.localhost install-app erpnext
```

ملاحظة: يجب أن تكون كل من frappe و erpnext على نفس الفرع. مثل version-14

### بدء Frappe بدون تصحيح

قم بتنفيذ الأمر التالي من دليل `frappe-bench`.

```shell
bench start
```

يمكنك الآن تسجيل الدخول باستخدام المستخدم `Administrator` وكلمة المرور التي اخترتها عند إنشاء الموقع.
سيكون موقع الويب الخاص بك الآن متاحًا على [development.localhost:8000](http://development.localhost:8000)
ملاحظة: لبدء bench مع مصحح الأخطاء، راجع القسم المخصص للتصحيح.

### إعداد Bench / موقع جديد باستخدام السيناريو

يعمل معظم المطورين مع العديد من العملاء والإصدارات. علاوة على ذلك، قد يتطلب تثبيت التطبيقات من جميع أفراد الفريق العامل للعميل.

يتم تبسيط هذا باستخدام سيناريو لتوتير العملية الآلية لإنشاء Bench / موقع جديد وتثبيت التطبيقات المطلوبة. كلمة مرور `المسؤول` للمواقع المنشأة هي `admin`.

يتم استخدام ملف `apps-example.json` المثال كقيمة افتراضية، حيث يقوم بتثبيت erpnext على أحدث إصدار مستقر. لتثبيت التطبيقات المخصصة، قم بنسخ ملف `apps-example.json` إلى ملف JSON مخصص وقم بإجراء التغييرات على قائمة التطبيقات. قم بتمرير هذا الملف إلى سيناريو `installer.py`.

> قد يكون لديك تطبيقات في مستودعات خاصة قد تتطلب الوصول عبر SSH. يمكنك استخدام SSH من داخل دليل المستخدم الخاص بك في نظام Linux (قابل للتكوين في docker-compose.yml).

```shell
python installer.py  # قم بتمرير --db-type postgres لقاعدة بيانات postgresdb
```

للحصول على مساعدة بشأن الأوامر

```shell
python installer.py --help
usage: installer.py [-h] [-j APPS_JSON] [-b BENCH_NAME] [-s SITE_NAME] [-r FRAPPE_REPO] [-t FRAPPE_BRANCH] [-p PY_VERSION] [-n NODE_VERSION] [-v] [-a ADMIN_PASSWORD] [-d DB_TYPE]

options:
  -h, --help            عرض رسالة المساعدة هذه والخروج
  -j APPS_JSON, --apps-json APPS_JSON
                        مسار إلى apps.json، الافتراضي: apps-example.json
  -b BENCH_NAME, --bench-name BENCH_NAME
                        اسم دليل Bench، الافتراضي: frappe-bench
  -s SITE_NAME, --site-name SITE_NAME
                        اسم الموقع، يجب أن ينتهي بـ .localhost، الافتراضي: development.localhost
  -r FRAPPE_REPO, --frappe-repo FRAPPE_REPO
                        مستودع frappe المستخدم، الافتراضي: https://github.com/frappe/frappe
  -t FRAPPE_BRANCH, --frappe-branch FRAPPE_BRANCH
                        فرع frappe المستخدم، الافتراضي: version-15
  -p PY_VERSION, --py-version PY_VERSION
                        إصدار Python، الافتراضي: Not Set
  -n NODE_VERSION, --node-version NODE_VERSION
                        إصدار Node، الافتراضي: Not Set
  -v, --verbose         إخراج مفصل
  -a ADMIN_PASSWORD, --admin-password ADMIN_PASSWORD
                        كلمة مرور المسؤول للموقع، الافتراضي: admin
  -d DB_TYPE, --db-type DB_TYPE
                        نوع قاعدة البيانات المستخدمة (مثل mariadb أو postgres)
```

يتم إنشاء Bench و / أو موقع جديد للعميل بالقيم الافترجمة المتبقي من النص:

التالية.

- كلمة مرور root لـ MariaDB: `123`
- كلمة مرور المسؤول: `admin`

> لاستخدام قاعدة بيانات Postegres، قم بتعليق خدمة mariabdb وإلغاء تعليق خدمة postegres.

### تشغيل Frappe مع تصحيح الأخطاء في Python باستخدام Visual Studio Code

لتمكين تصحيح الأخطاء في Python داخل برنامج Visual Studio Code، يجب عليك أولاً تثبيت امتداد `ms-python.python` داخل الحاوية. يجب أن يكون قد تم ذلك تلقائيًا بالفعل، ولكن اعتمادًا على تكوين VSCode الخاص بك، يمكنك إجباره عن طريق:

- انقر على رمز الامتداد داخل VSCode
- ابحث عن `ms-python.python`
- انقر على "تثبيت في Dev Container: Frappe Bench"
- انقر على "إعادة التحميل"

نحن بحاجة إلى بدء Bench بشكل منفصل من خلال مصحح VSCode. لهذا السبب، **بدلاً من** تشغيل `bench start` يجب عليك تشغيل الأمر التالي داخل دليل frappe-bench:

```shell
honcho start \
    socketio \
    watch \
    schedule \
    worker_short \
    worker_long
```

بديلًا عن ذلك، يمكنك استخدام تكوين الإطلاق في VSCode "Honcho SocketIO Watch Schedule Worker" الذي يقوم بتشغيل نفس الأمر السابق.

يقوم هذا الأمر بتشغيل جميع العمليات باستثناء Redis (الذي يعمل بالفعل في حاوية منفصلة) وعملية `web`. يمكن بدء هذه العملية في النهاية من علامة المصحح في VSCode عن طريق النقر فوق زر "تشغيل".

يمكنك الآن تسجيل الدخول باستخدام المستخدم "Administrator" وكلمة المرور التي اخترتها عند إنشاء الموقع، إذا قمت بمتابعة تثبيت هذا الدليل بدون تدخل، فإن كلمة المرور ستكون "admin".

لتصحيح أخطاء العمال، قم بتخطي بدء العمل بواسطة honcho وقم بتشغيله باستخدام مصحح VSCode.

للتكوين المتقدم في VSCode في الـ devcontainer، قم بتغيير ملفات التكوين في `development/.vscode`.

## التطوير باستخدام واجهة الأوامر التفاعلية

يمكنك تشغيل واجهة أوامر تفاعلية بسيطة في وحدة التحكم بواسطة الأمر التالي:

```shell
bench --site development.localhost console
```

على الأغلب، قد ترغب في تشغيل واجهة الأوامر التفاعلية لـ VSCode باستخدام نواة Jupyter.

قم بتشغيل لوحة الأوامر في VSCode (cmd+shift+p أو ctrl+shift+p)، ثم قم بتنفيذ الأمر `Python: Select interpreter to start Jupyter server` وحدد `/workspace/development/frappe-bench/env/bin/python`.

الخطوة الأولى هي تثبيت وتحديث البرامج المطلوبة. عادةً ما يحتاج إطار عمل frappe إلى إصدار قديم من Jupyter، بينما يفضل VSCode التقدم بسرعة، وهذا يمكن أن يتسبب في [مشاكل](https://github.com/jupyter/jupyter_console/issues/158). لهذا السبب نحتاج إلى تنفيذ الأمر التالي.

```shell
/workspace/development/frappe-bench/env/bin/python -m pip install --upgrade jupyter ipykernel ipython
```

ثم، قم بتنفيذ الأمر `Python: Show Python interactive window` من لوحة الأوامر في VSCode.

استبدل `development.localhost` بموقعك الخاص وقم بتنفيذ الشفرة التالية في خلية Jupyter:

```python
import frappe

frappe.init(site='development.localhost', sites_path='/workspace/development/frappe-bench/sites')
frappe.connect()
frappe.local.lang = frappe.db.get_default('lang')
frappe.db.connect()
```

قد يستغرق تنفيذ الأمر الأول بضع ثوانٍ، وهذا أمر متوقع.

## بدء الحاويات يدويًا

في حالة عدم استخدامك لـ VSCode، يمكنك بدء الحاويات يدويًا باستخدام الأمر التالي:

### تشغيل الحاويات

```shell
docker-compose -f .devcontainer/docker-compose.yml up -d
```

وأدخل واجهة الأوامر التفاعلية لحاوية التطوير باستخدام الأمر التالي:

```shell
docker exec -e "TERM=xterm-256color" -w /workspace/development -it devcontainer-frappe-1 bash
```

## استخدام خدمات إضافية أثناء التطوير

أضف أي خدمة تحتاجها للتطوير في ملف `.devcontainer/docker-compose.yml` ثم قم بإعادة البناء وإعادة فتحها في devcontainer.

على سبيل المثال:

```yaml
...
services:
 ...
  postgresql:
    image: postgres:11.8
    environment:
      POSTGRES_PASSWORD: 123
    volumes:
      - postgresql-data:/var/lib/postgresql/data
    ports:
      - 5432:5432

volumes:
  ...
  postgresql-data:
```

يمكن الوصول إلى الخدمة بواسطة اسم الخدمة من حاوية التطوير `frappe`. ستكون الخدمة المذكورة أعلاه متاحة عبر اسم المضيف `postgresql`. إذا تم نشر المنافذ على المضيف، يمكن الوصول إليها عبر `localhost:5432`.

## استخدام اختبارات واجهة المستخدم باستخدام Cypress

لتشغيل اختبارات واجهة المستخدم القائمة على Cypress في بيئة Docker، قم باتباع الخطوات التالية:

1. قم بتثبيت وإعداد أدوات X11 على الجهاز الظاهري باستخدام السيناريو "install_x11_deps.sh".

```shell
  sudo bash ./install_x11_deps.sh
```

سيقوم هذا السيناريو بتثبيت التبعيات المطلوبة وتمكين X11Forwarding وإعادة تشغيل خدمة SSH وتصدير متغير "DISPLAY".

2. قم بتشغيل خدمة X11 باستخدام "startx" أو "xquartz".
3. قم بتشغيل خدمات Docker Compose.
4. قم بالاتصال بخدمة ui-tester باستخدام أمر "docker exec..".
5. قم بتصدير CYPRESS_baseUrl وغيرها من المتغيرات البيئية المطلوبة.
6. قم ببدء واجهة مستخدم Cypress عن طريق إصدار أمر "cypress run".

> لمزيد من المراجع: [توثيق Cypress الرسمي](https://www.cypress.io/blog/2019/05/02/run-cypress-with-a-single-docker-command)

> تأكد من تصدير متغير البيئة DISPLAY دائمًا.

## استخدام Mailpit لاختبار خدمات البريد

لاستخدام Mailpit، قم بإلغاء تعليق الخدمة في ملف docker-compose.yml.
سيكون الواجهة متاحة عبر المنفذ 8025 ويمكن استخدام خدمة SMTP كـ mailpit:1025.
