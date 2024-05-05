### التطبيقات المخصصة

### استنسخ مستودع frappe_docker وانتقل إلى الدليل

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

### تحميل التطبيقات المخصصة من خلال json

يجب تمرير `apps.json` كمتغير بيئي للوسيطة بناءً.

```shell
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/payments",
    "branch": "version-15"
  },
  {
    "url": "https://{{ PAT }}@git.example.com/project/repository.git",
    "branch": "main"
  }
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)
```
يمكنك أيضًا إنشاء سلسلة base64 من ملف json:

```shell
export APPS_JSON_BASE64=$(base64 -w 0 /path/to/apps.json)
```

ملاحظة:

- `url` يجب أن يكون رابط git http(s) مع رموز الوصول الشخصي بدون اسم المستخدم، على سبيل المثال:- http://{{PAT}}@github.com/project/repository.git في حالة المستودع الخاص.
- أضف التبعيات يدويًا في `apps.json` مثل إضافة `payments` إذا كنت تقوم بتثبيت `erpnext`.
- استخدم المستودع المنشعب أو الفرع لـ ERPNext في حالة الحاجة إلى استخدام فرعك الخاص أو اختبار طلب سحب.

### بناء الصورة

```shell
docker build \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/custom/Containerfile .
```

ملاحظة:

- استخدم `buildah` بدلاً من `docker` وفقًا لإعدادك.
- تأكد من أن متغير `APPS_JSON_BASE64` يحتوي على سلسلة JSON المشفرة بنظام base64 بشكل صحيح. يُستهلك كوسيط بناء، وتشفير base64 يضمن أنه يمكن استخدامه بسهولة مع المتغيرات البيئية. استخدم `jq empty apps.json` للتحقق من صحة ملف `apps.json`.
- تأكد من أن `--tag` هو اسم صورة صالح سيتم دفعه إلى السجل. انظر القسم [أدناه](#use-images) لملاحظات حول استخدامه.
- يتم إزالة مجلدات `.git` لجميع التطبيقات من الصورة.

قم بتخصيص هذه `--build-arg` الاختيارية لاستخدام مستودع وفرع Frappe Framework مختلف، أو نسخة مختلفة من Python و NodeJS:

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=18.20.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=ghcr.io/user/repo/custom:1.0.0 \
  --file=images/custom/Containerfile .
```

### دفع الصورة لاستخدامها في ملفات yaml

تسجيل الدخول إلى `docker` أو `buildah`

```shell
buildah login
```

دفع الصورة

```shell
buildah push ghcr.io/user/repo/custom:1.0.0
```

### استخدام Kaniko

تُطلب متغيرات تنفيذ التنفيذ التالية. يتم تشغيل المثال محليًا في حاوية Docker. يمكنك تشغيله كجزء من CI/CD أو كجزء من عنقودك.

```shell
podman run --rm -it \
  -v "$HOME"/.docker/config.json:/kaniko/.docker/config.json \
  gcr.io/kaniko-project/executor:latest \
  --dockerfile=images/custom/Containerfile \
  --context=git://github.com/frappe/frappe_docker \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --cache=true \
  --destination=ghcr.io/user/repo/custom:1.0.0 \
  --destination=ghcr.io/user/repo/custom:latest
```

المزيد عن [Kaniko](https://github.com/GoogleContainerTools/kaniko)

### استخدام الصور

في [compose.yaml](../compose.yaml) قم بتغيير مرجع الصورة إلى `tag` الذي استخدمته عند بنائها. ثم، إذا كنت تستخدم علامة مثل `custom_erpnext:staging`، ستبدو جزء `x-customizable-image` كالتالي:

```
x-customizable-image: &customizable_image
  image: custom_erpnext:staging
  pull_policy: never
```

`pull_policy` أعلاه اختياري ويمنع `docker` من محاولة تنزيل الصورة عندما يتم بناؤها محليًا.

تأكد من أن اسم الصورة صحيح ليتم دفعه إلى السجل. بعد دفع الصور، يمكنك سحبها إلى الخوادم ليتم نشرها. إذا كان السجل خاصًا، فإنه يتطلب مصادقة إضافية.