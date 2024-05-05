### تصحيح أخطاء لوحة تحكم Bench باستخدام VSCode
### bench-console-and-vscode-debugger

أضف الضبط التالي إلى مصفوفة `configurations` في ملف `launch.json` لتشغيل وحدة التحكم Bench واستخدام مصحح الأخطاء. قم بتبديل `development.localhost` بالموقع المناسب. كما قم بتبديل `frappe-bench` باسم دليل Bench.

```json
{
  "name": "Bench Console",
  "type": "python",
  "request": "launch",
  "program": "${workspaceFolder}/frappe-bench/apps/frappe/frappe/utils/bench_helper.py",
  "args": ["frappe", "--site", "development.localhost", "console"],
  "pythonPath": "${workspaceFolder}/frappe-bench/env/bin/python",
  "cwd": "${workspaceFolder}/frappe-bench/sites",
  "env": {
    "DEV_SERVER": "1"
  }
}
```