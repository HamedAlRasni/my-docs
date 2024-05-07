## الانتقال من إعداد الصور المتعددة

الآن تستخدم جميع الحاويات نفس الصورة. استخدم `frappe/erpnext` بدلاً من `frappe/frappe-worker`، `frappe/frappe-nginx`، `frappe/frappe-socketio`، `frappe/erpnext-worker` و `frappe/erpnext-nginx`.

الآن تحتاج إلى تحديد الأمر والمتغيرات البيئية للحاويات التالية:

### الواجهة الأمامية

لتعمل خدمة `frontend` كواجهة أمامية للملفات الثابتة والتوجيه العكسي، تحتاج إلى تمرير `nginx-entrypoint.sh` كأمر `command` للحاوية والمتغيرات البيئية `BACKEND` و `SOCKETIO` التي تشير إلى `{host}:{port}` لخدمات gunicorn و websocket. تحقق من [متغيرات البيئة](environment-variables.md).

الآن تحتاج فقط إلى توصيل محرك الأقراص `sites` في الموقع `/home/frappe/frappe-bench/sites`. لا حاجة لحجم `assets` ونص أو خطوات تعبئة الأصول.

تغيير مثال:

```yaml
# ... تم إزالته للتبسيط
frontend:
  image: frappe/erpnext:${ERPNEXT_VERSION:?إصدار ERPNext غير محدد}
  command:
    - nginx-entrypoint.sh
  environment:
    BACKEND: backend:8000
    SOCKETIO: websocket:9000
  volumes:
    - sites:/home/frappe/frappe-bench/sites
# ... تم إزالته للتبسيط
```

### ويب سوكت

لجعل خدمة `websocket` تعمل كخادم لـ socketio backend، تحتاج إلى تمرير `["node", "/home/frappe/frappe-bench/apps/frappe/socketio.js"]` كـ `command` للحاوية.

تغيير المثال:

```yaml
# ... تم إزالته للتبسيط
websocket:
  image: frappe/erpnext:${ERPNEXT_VERSION:?إصدار ERPNext غير محدد}
  command:
    - node
    - /home/frappe/frappe-bench/apps/frappe/socketio.js
# ... تم إزالته للتبسيط
```

### Configurator

لجعل خدمة `configurator` تعمل كمهمة تكوين تشغيل مرة واحدة، تحتاج إلى تمرير `["bash", "-c"]` كـ `entrypoint` للحاوية ونص السكربت الخاص بـ bash مباشرةً إلى yaml. لا يوجد `configure.py` في الحاوية الآن.

مثال التغيير:

```yaml
# ... تم إزالته للتبسيط
configurator:
  image: frappe/erpnext:${ERPNEXT_VERSION:?إصدار ERPNext غير محدد}
  restart: "no"
  entrypoint:
    - bash
    - -c
  command:
    - >
      bench set-config -g db_host $$DB_HOST;
      bench set-config -gp db_port $$DB_PORT;
      bench set-config -g redis_cache "redis://$$REDIS_CACHE";
      bench set-config -g redis_queue "redis://$$REDIS_QUEUE";
      bench set-config -gp socketio_port $$SOCKETIO_PORT;
  environment:
    DB_HOST: db
    DB_PORT: "3306"
    REDIS_CACHE: redis-cache:6379
    REDIS_QUEUE: redis-queue:6379
    SOCKETIO_PORT: "9000"
# ... تم إزالته للتبسيط
```

### إنشاء الموقع

لجعل خدمة `create-site` تعمل كمهمة إنشاء موقع تشغيل مرة واحدة، تحتاج إلى تمرير `["bash", "-c"]` كـ `entrypoint` للحاوية ونص السكربت الخاص بـ bash مباشرةً إلى yaml. تأكد من استخدام `--no-mariadb-socket` لأن bench المصدر يتم تثبيته في الحاوية.

تم تغيير `WORKDIR` إلى `/home/frappe/frappe-bench` مثل إعداد bench الذي نحن معتادون عليه. لذا فإن المسار للعثور على `common_site_config.json` قد تغير إلى `sites/common_site_config.json`.

مثال التغيير:

```yaml
# ... تم إزالته للتبسيط
create-site:
  image: frappe/erpnext:${ERPNEXT_VERSION:?إصدار ERPNext غير محدد}
  restart: "no"
  entrypoint:
    - bash
    - -c
  command:
    - >
      wait-for-it -t 120 db:3306;
      wait-for-it -t 120 redis-cache:6379;
      wait-for-it -t 120 redis-queue:6379;
      export start=`date +%s`;
      until [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".db_host // empty"` ]] && \
        [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".redis_cache // empty"` ]] && \
        [[ -n `grep -hs ^ sites/common_site_config.json | jq -r ".redis_queue // empty"` ]];
      do
        echo "Waiting for sites/common_site_config.json to be created";
        sleep 5;
        if (( `date +%s`-start > 120 )); then
          echo "could not find sites/common_site_config.json with required keys";
          exit 1
        fi
      done;
      echo "sites/common_site_config.json found";
      bench new-site --no-mariadb-socket --admin-password=admin --db-root-password=admin --install-app erpnext --set-default frontend;

# ... تم إزالته للتبسيط
```