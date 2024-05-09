# Automatic Execution of dhclient Command on Ubuntu Server Boot

# تنفيذ أمر dhclient تلقائيًا عند بدء تشغيل سيرفر ابنتو



1. **استخدام systemd service**:
يمكنك إنشاء وحدة systemd لتنفيذ الأمر عند بدء التشغيل. هذا الأسلوب يعتبر الأكثر شيوعًا ويوفر إمكانيات تكوين متقدمة.
- أنشئ ملفًا للوحدة في `/etc/systemd/system/` مثل `dhclient.service`:
```
sudo nano /etc/systemd/system/dhclient.service
```
- أضف محتوى مشابه للتالي:
```
[Unit]
Description=DHCP client for eth0
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/sbin/dhclient eth0

[Install]
WantedBy=multi-user.target
```
- قم بحفظ الملف وإعادة تحميل وحدات systemd:
```
sudo systemctl daemon-reload
```
- قم بتمكين الوحدة لتشغيلها عند بدء التشغيل:
```
sudo systemctl enable dhclient.service
```

2. **استخدام cron و @reboot**:
يمكنك إضافة الأمر مباشرة إلى cron لتنفيذه عند بدء التشغيل.
- فتح ملف crontab:
```
sudo crontab -e
```
- أضف الأمر التالي في نهاية الملف:
```
@reboot /sbin/dhclient eth0
```
- احفظ التغييرات وأغلق المحرر.

باستخدام أحد الطرق المذكورة أعلاه، يمكنك تنفيذ الأمر `dhclient` تلقائيًا عند بدء تشغيل النظام في Ubuntu Server.