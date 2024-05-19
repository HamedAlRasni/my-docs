# كيفية سحب نسخة من فرع develop وإجراء تعديلات ورفعها على فرع version-15 في Frappe أو ERPNext

# How to Pull from the develop Branch, Make Changes, and Push to the version-15 Branch in Frappe or ERPNext

### 1. **استنساخ المستودع**

أولاً، تأكد من أنك قد استنسخت المستودع إلى جهازك المحلي:

```bash
git clone https://github.com/frappe/frappe.git
cd frappe
```

أو إذا كنت تعمل على `ERPNext`:

```bash
git clone https://github.com/frappe/erpnext.git
cd erpnext
```

### 2. **التبديل إلى فرع `develop`**

تنزيل الفرع `develop` والتبديل إليه:

```bash
git fetch
git checkout -b develop origin/develop
```

### 3. **إجراء التعديلات**

قم بإجراء التعديلات التي تحتاجها على الشفرة. يمكنك استخدام محرر النصوص المفضل لديك لإجراء هذه التعديلات.

### 4. **إضافة التعديلات وتنفيذ الالتزام (commit)**

بعد الانتهاء من التعديلات، أضف الملفات المعدلة إلى منطقة التجهيز:

```bash
git add .
```

ثم قم بعمل التزام بالتغييرات مع رسالة توضيحية:

```bash
git commit -m "Completed updates on develop branch"
```

### 5. **إنشاء فرع `version-15`**

إذا لم يكن لديك فرع `version-15`، قم بإنشائه:

```bash
git checkout -b version-15
```

إذا كان الفرع `version-15` موجودًا بالفعل على المستودع البعيد، يمكنك التبديل إليه مباشرة:

```bash
git checkout version-15
```

### 6. **دمج التغييرات من `develop` إلى `version-15`**

ادمج التغييرات من فرع `develop` إلى فرع `version-15`:

```bash
git merge develop
```

### 7. **رفع التغييرات إلى المستودع البعيد**

أخيرًا، ارفع التغييرات إلى المستودع البعيد:

```bash
git push origin version-15
```

### مثال عملي كامل

- استنساخ المستودع
```bash
git clone https://github.com/frappe/frappe.git
cd frappe
```

- تنزيل الفروع البعيدة والتبديل إلى develop
```bash
git fetch
git checkout -b develop origin/develop
```

- إجراء التعديلات اللازمة
(افترض أنك أجريت التعديلات هنا)
إضافة التعديلات وتنفيذ الالتزام
```bash
git add .
git commit -m "Completed updates on develop branch"
```

- إنشاء أو التبديل إلى فرع version-15
```bash
git checkout -b version-15
```

- أو إذا كان الفرع موجودًا بالفعل:
```bash
git checkout version-15
```

- دمج التغييرات من develop إلى version-15
```bash
git merge develop
```

- رفع التغييرات إلى المستودع البعيد
```bash
git push origin version-15
```

بهذه الطريقة، تكون قد قمت بسحب نسخة من فرع `develop`، أجريت التعديلات اللازمة، ثم قمت بدمج هذه التعديلات ورفعها إلى فرع `version-15` بنجاح.