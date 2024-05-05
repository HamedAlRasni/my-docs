الاتصال بخدمات localhost من حاويات Docker لتطوير التطبيقات المحلية

أضف ما يلي إلى حاوية Frappe من `.devcontainer/docker-compose.yml`:

```yaml
...
  frappe:
    ...
    extra_hosts:
      app1.localhost: 172.17.0.1
      app2.localhost: 172.17.0.1
...
```

هذا يجعل أسماء النطاق `app1.localhost` و `app2.localhost` تتصل بمضيف Docker وتتصل بالخدمات التي تعمل على `localhost`.