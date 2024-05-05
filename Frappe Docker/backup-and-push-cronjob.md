### مهمة أوتوماتيكية لعمل نسخة احتياطية ودفع الملفات

Create backup service or stack.

إنشاء خدمة أو استك

```yaml
# backup-job.yml
version: "3.7"
services:
  backup:
    image: frappe/erpnext:${VERSION}
    entrypoint: ["bash", "-c"]
    command:
      - |
        bench --site all backup
        ## قم بإلغاء التعليق لالتقاط لقطات Restic.
        # restic snapshots || restic init
        # restic backup sites
        ## قم بإلغاء التعليق للحفاظ على n=30 لقطة فقط.
        # restic forget --group-by=paths --keep-last=30 --prune
    environment:
      # قم بتعيين المتغيرات البيئية الصحيحة لـ Restic
      - RESTIC_REPOSITORY=s3:https://s3.endpoint.com/restic
      - AWS_ACCESS_KEY_ID=access_key
      - AWS_SECRET_ACCESS_KEY=secret_access_key
      - RESTIC_PASSWORD=restic_password
    volumes:
      - "sites:/home/frappe/frappe-bench/sites"
    networks:
      - erpnext-network

networks:
  erpnext-network:
    external: true
    name: ${PROJECT_NAME:-erpnext}_default

volumes:
  sites:
    external: true
    name: ${PROJECT_NAME:-erpnext}_sites
```

في حالة إعداد مضيف Docker واحد، أضف إدخالًا في crontab للنسخ الاحتياطي كل 6 ساعات.

```
0 */6 * * * /usr/local/bin/docker-compose -f /path/to/backup-job.yml up -d > /dev/null
```

أو

```
0 */6 * * * docker compose -p erpnext exec backend bench --site all backup --with-files > /dev/null
```

ملاحظات:

- تأكد من توفر `docker-compose` أو `docker compose` في المسار أثناء التنفيذ.
- قم بتغيير سلسلة cron حسب الحاجة.
- قم بتعيين اسم المشروع الصحيح بدلاً من `erpnext`.
- لـ Docker Swarm، أضفه كـ [swarm-cronjob](https://github.com/crazy-max/swarm-cronjob)
- أضفه كـ `CronJob` في حالة عمل إتشاريكلستر.