# الصور

هناك 3 صور يمكنك العثور عليها في الدليل `/images`:

- `bench`. تُستخدم للتطوير. [تعرّف على المزيد حول كيفية بدء التطوير](../development/README.md).
- `production`.
  - خادم Python متعدد الأغراض. يعمل باستخدام [خادم Werkzeug](https://werkzeug.palletsprojects.com/en/2.0.x/) مع [gunicorn](https://gunicorn.org)، وطوابير (عبر `bench worker`)، أو الجدول الزمني (عبر `bench schedule`).
  - يحتوي على ملفات JS و CSS ويوجه الطلبات الواردة باستخدام [nginx](https://www.nginx.com).
  - يعالج طلبات الويب الحية في الوقت الفعلي باستخدام [Socket.IO](https://socket.io).
- `custom`. تُستخدم لبناء bench باستخدام ملف `apps.json` المعين بـ `--apps_path` أثناء بدء تهيئة bench. `apps.json` هو مصفوفة json. مثال: `[{"url":"{{repo_url}}","branch":"{{repo_branch}}"}]`

الصورة تحتوي على كل ما نحتاجه لتشغيل جميع العمليات التي يتطلبها إطار Frappe (انظر [مرجع Bench Procfile](https://frappeframework.com/docs/v14/user/en/bench/resources/bench-procfile)). نتبع [أفضل الممارسات في Docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#decouple-applications) ونقسم هذه العمليات إلى حاويات مختلفة.

> نحن نستخدم [بناء متعدد المراحل](https://docs.docker.com/develop/develop-images/multistage-build/) و [Docker Buildx](https://docs.docker.com/engine/reference/commandline/buildx/) لإعادة استخدام أكبر قدر ممكن من الأشياء وجعل عملياتنا أكثر كفاءة.

# ملفات الإيقاف

بعد بناء الصور يتعين علينا تشغيل الحاويات. أفضل وأبسط طريقة للقيام بذلك هي استخدام [ملفات الإيقاف](https://docs.docker.com/compose/compose-file/).

لدينا ملف إيقاف رئيسي واحد، `compose.yaml`. يتم التعامل أيضًا فيه مع الخدمات الموصوفة والشبكات والمجلدات.

## الخدمات

تُوصف جميع الخدمات في `compose.yaml`

- `configurator`. يحدث `common_site_config.json` بحيث يعرف Frappe كيفية الوصول إلى قاعدة البيانات و redis. يتم تنفيذه في كل `docker-compose up` (ويُغادر على الفور). تبدأ الخدمات الأخرى بعد أن يغادر هذا الحاوي بنجاح.
- `backend`. [خادم Werkzeug](https://werkzeug.palletsprojects.com/en/2.0.x/).
- `db`. خدمة اختيارية تشغل [MariaDB](https://mariadb.com) إذا كنت أيضًا تستخدم `overrides/compose.mariadb.yaml` أو [Postgres](https://www.postgresql.org) إذا كنت أيضًا تستخدم `overrides/compose.postgres.yaml`.
- `redis`. خدمة اختيارية تشغل خادم [Redis](https://redis.io) مع ذاكرة مؤقتة، [Socket.IO](https://socket.io) وبيانات الطوابير.
- `frontend`. خادم [nginx](https://www.nginx.com) الذي يخدم ملفات JS/CSS ويوجه الطلبات الواردة.
- `proxy`. [Traefik](https://traefik.io/traefik/) بروكسي. هنا لإعدادات معقدة أو تجاوز HTTPS (باستخدام `overrides/compose.https.yaml`).
- `websocket`. خادم Node يشغل [Socket.IO](https://socket.io).
- `queue-short`, `queue-long`. خوادم Python تشغل طوابير المهام باستخدام [rq](https://python-rq.org).
- `scheduler`. خادم Python يشغل المهام على الجدول الزمني باستخدام [schedule](https://schedule.readthedocs.io/en/stable/).

## التجاويف

لدينا العديد من [التجاويف](https://docs.docker.com/compose/extends/):

- `overrides/compose.proxy.yaml`. يضيف Traefik proxy إلى الإعداد.
- `overrides/compose.noproxy.yaml`. ينشر منافذ `frontend` مباشرة دون أي بروكسي.
- `overrides/compose.https.yaml`. يضبط تلقائيًا شهادة Let's Encrypt ويوجه جميع الطلبات إلى تحويلها من http إلى https.
- `overrides/compose.mariadb.yaml`. يضيف خدمة `db` ويضبط صورتها على MariaDB.
- `overrides/compose.postgres.yaml`. يضيف خدمة `db`

 ويضبط صورتها على Postgres. لاحظ أن ERPNext حاليًا لا يدعم Postgres.
- `overrides/compose.redis.yaml`. يضيف خدمة `redis` ويضبط صورتها على `redis`.

من السهل تشغيل التجاويف. كل ما علينا فعله هو تحديد ملفات الإيقاف التي يجب استخدامها بواسطة docker-compose. على سبيل المثال، نريد ERPNext:

```bash
# استشهد إلى ملف الإيقاف الرئيسي (compose.yaml) وأضف واحد إضافي.
docker-compose -f compose.yaml -f overrides/compose.redis.yaml config
```

⚠ تأكد من استخدام docker-compose v2 (قم بتشغيل `docker-compose -v` للتحقق). إذا كنت تريد استخدام v1، تأكد من وجود علامات `$` الصحيحة حيث يتم تكرارها من قبل الأمر `config`!

هذا كل شيء! بالطبع، علينا أيضًا إعداد `.env` قبل كل ذلك، لكن هذا ليس النقطة.

تحقق من [المتغيرات البيئية](environment-variables.md) للمزيد.