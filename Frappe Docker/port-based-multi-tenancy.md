تحذير: لا تستخدم هذا في الإنتاج إذا كان الموقع سيتم خدمته عبر http عادي.

### الخطوة 1

إزالة خدمة traefik من ملف docker-compose.yml

### الخطوة 2

إضافة خدمة لكل منفذ يجب تعريضه.

على سبيل المثال، `port-site-1`، `port-site-2`، `port-site-3`.

```yaml
# ... تم إزالته للتبسيط
services:
	# ... تم إزالته للتبسيط
  port-site-1:
    image: frappe/erpnext:v14.11.1
    deploy:
      restart_policy:
        condition: on-failure
    command:
      - nginx-entrypoint.sh
    environment:
      BACKEND: backend:8000
      FRAPPE_SITE_NAME_HEADER: site1.local
      SOCKETIO: websocket:9000
    volumes:
      - sites:/home/frappe/frappe-bench/sites
    ports:
      - "8080:8080"
  port-site-2:
    image: frappe/erpnext:v14.11.1
    deploy:
      restart_policy:
        condition: on-failure
    command:
      - nginx-entrypoint.sh
    environment:
      BACKEND: backend:8000
      FRAPPE_SITE_NAME_HEADER: site2.local
      SOCKETIO: websocket:9000
    volumes:
      - sites:/home/frappe/frappe-bench/sites
    ports:
      - "8081:8080"
  port-site-3:
    image: frappe/erpnext:v14.11.1
    deploy:
      restart_policy:
        condition: on-failure
    command:
      - nginx-entrypoint.sh
    environment:
      BACKEND: backend:8000
      FRAPPE_SITE_NAME_HEADER: site3.local
      SOCKETIO: websocket:9000
    volumes:
      - sites:/home/frappe/frappe-bench/sites
    ports:
      - "8082:8080"
```

ملاحظات:

- التهيئة أعلاه ستعرض `site1.local`، `site2.local`، `site3.local` على المنافذ `8080`، `8081`، `8082` على التوالي.
- قم بتغيير `site1.local` إلى اسم الموقع لتقديمه من bench.
- قم بتغيير المتغيرات البيئية `BACKEND` و `SOCKETIO` وفقًا لأسماء الخدمات الخاصة بك.
- تأكد من توافر حجم `sites:` كجزء من yaml.