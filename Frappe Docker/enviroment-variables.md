# المتغيرات البيئية

يتم تمرير جميع الأوامر مباشرة إلى الحاوية حسب نوع الخدمة. البيئة المستخدمة في الصورة فقط تُستخدم لأمر `nginx-entrypoint.sh`. وهي كالتالي:

- `BACKEND`: يتم تعيينه إلى `{host}:{port}`، القيمة الافتراضية `0.0.0.0:8000`
- `SOCKETIO`: يتم تعيينه إلى `{host}:{port}`، القيمة الافتراضية `0.0.0.0:9000`
- `UPSTREAM_REAL_IP_ADDRESS`: يتم تعيين إعدادات Nginx لـ [ngx_http_realip_module#set_real_ip_from](http://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from)، القيمة الافتراضية `127.0.0.1`
- `UPSTREAM_REAL_IP_HEADER`: يتم تعيين إعدادات Nginx لـ [ngx_http_realip_module#real_ip_header](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header)، القيمة الافتراضية `X-Forwarded-For`
- `UPSTREAM_REAL_IP_RECURSIVE`: يتم تعيين إعدادات Nginx لـ [ngx_http_realip_module#real_ip_recursive](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_recursive)، القيمة الافتراضية `off`
- `FRAPPE_SITE_NAME_HEADER`: يضبط رأس الوكيل `X-Frappe-Site-Name` ويخدم اسم الموقع المُحدد في الرأس، القيمة الافتراضية `$host`، أي العثور على اسم الموقع من رأس المضيف. تفاصيل إضافية [أدناه](#frappe_site_name_header)
- `PROXY_READ_TIMEOUT`: المهلة لخدمة gunicorn الخادمة، القيمة الافتراضية `120`
- `CLIENT_MAX_BODY_SIZE`: الحد الأقصى لحجم الجسم للتحميلات، القيمة الافتراضية `50m`

لتجاوز `nginx-entrypoint.sh`، قم بتعيين `/etc/nginx/conf.d/default.conf` المطلوب وتشغيل `nginx -g 'daemon off;'` كأمر للحاوية.

## التكوين

نستخدم المتغيرات البيئية لتكوين إعداداتنا. يستخدم docker-compose المتغيرات من ملف `.env`. للبدء، قم بنسخ `example.env` إلى `.env`.

### `FRAPPE_VERSION`

إصدار إطار Frappe. يمكنك العثور على جميع الإصدارات [هنا](https://github.com/frappe/frappe/releases).

### `DB_PASSWORD`

كلمة مرور قاعدة بيانات MariaDB (أو Postgres).

### `DB_HOST`

اسم المضيف لقاعدة بيانات MariaDB (أو Postgres). ضع القيمة إذا تم استخدام خدمة خارجية لقاعدة البيانات فقط.

### `DB_PORT`

منفذ قاعدة بيانات MariaDB (3306) أو Postgres (5432). ضع القيمة إذا تم استخدام خدمة خارجية لقاعدة البيانات فقط.

### `REDIS_CACHE`

اسم المضيف لخادم redis لتخزين الذاكرة المؤقتة. ضع القيمة إذا تم استخدام خدمة خارجية لخدمة redis فقط.

### `REDIS_QUEUE`

اسم المضيف لخادم redis لتخزين بيانات الطابور والsocketio. ضع القيمة إذا تم استخدام خدمة خارجية لخدمة redis فقط.

### `ERPNEXT_VERSION`

[الإصدار](https://github.com/frappe/frappe/releases) ERPNext. هذا المتغير مطلوب إذا كنت تستخدم ERPNext override.

### `LETSENCRYPT_EMAIL`

البريد الإلكتروني المستخدم لتسجيل شهادة HTTPS. هذا مطلوب فقط إذا كنت تستخدم HTTPS override.

### `FRAPPE_SITE_NAME_HEADER`

هذا المتغير البيئي غير مطلوب. القيمة الافتراضية هي `$$host` التي تُحل الموقع بواسطة المضيف. على سبيل المثال، إذا كان المضيف الخاص بك هو `example.com`، يجب أن يكون اسم الموقع `example.com`، أو إذا كان المضيف هو `127.0.0.1` (تصحيح محلي)، يجب أن يكون `127.0.0.1`. يسمح هذا المتغير بتجاوز السلوك الموصوف. على سبيل المثال، إذا قمت بإنشاء موقع بالاسم `mysite` وتريد الوصول إليه بواسطة المضيف `127.0.0.1`، فسيكون عليك تعيين هذا المتغير إلى `mysite

`.

هناك متغيرات أخرى غير مذكورة هنا. إنها تعتبر داخلية بعض الشيء ولا داعي للقلق بشأنها ما لم ترغب في تغيير الملف الرئيسي للتكوين.