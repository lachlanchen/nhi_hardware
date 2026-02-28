[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# التحكم في عتاد NHI والتقاط الأحداث

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 دليل التنقل السريع

| Use this section | Purpose |
|---|---|
| [Installation](#installation) | إعداد البيئة والتبعيات |
| [Usage](#usage) | تشغيل المنسق الويب وسير العمل عبر CLI |
| [Configuration](#configuration) | ضبط المنفذ التسلسلي والشبكة والإعدادات الافتراضية |
| [Examples](#examples) | تشغيل أمثلة الأوامر العملية |
| [Troubleshooting](#troubleshooting) | إصلاح مشاكل الإعداد الشائعة |

## 🧭 نظرة عامة على المشروع

| التركيز | التفاصيل |
|---|---|
| المهمة | تنسيق التقاط الأحداث، والتحكم في الحركة، وإشارات LED لسير عمل مختبري قابل لإعادة الإنتاج |
| نقطة الدخول الأساسية | `app.py` (واجهة ويب Tornado + تنسيق تسلسل غير متزامن) |
| المدخلات الأساسية | تيار أحداث EVK5، التحكم بمحور FMC4030، أوامر LED التسلسلية عبر Arduino |
| المخرجات الأساسية | `data/axis_1_positions.csv`، تصدير CSV للأحداث اختياري |
| المنصات | Windows / Linux (حسب توفر SDK والعتاد) |

مشروع تنسيق للعتاد في تجارب كاميرات الأحداث يجمع بين:
- التقاط كاميرا EVK5 (حزمة Prophesee Metavision)
- التحكم الحركي للمحور CNC المبني على FMC4030
- التحكم التسلسلي لليمود عبر Arduino
- واجهة إطلاق ويب بسيطة (Tornado)

> افتراض: بيئات العتاد وملفات DLL وSDK تختلف باختلاف المضيف وتُستنتج من شفرة المشروع؛ قد يختلف سلوك الأوامر الفعلي حسب نظام التشغيل وإصدارات التعريفات وتوافر وقت التشغيل.

## 🧠 نظرة عامة

تنفذ سير العمل الأساسي من الطرف إلى الطرف في `app.py`:

1. تدوير مجلد `data/` الموجود فعليًا إلى مجلد موسوم بالطابع الزمني (`data_YYYYMMDD_HHMMSS`) اختياريًا
2. الاتصال بـ LED عبر Arduino (الافتراضي `COM4`)
3. تهيئة متحكم CNC عبر `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. تشغيل تسلسل إضاءة LED وحركة محور Y
5. تسجيل أحداث EVK5 اختيارياً (معطل حاليًا في المسار النشط؛ راجع الملاحظات أدناه)
6. حفظ سجلات مواقع المحاور في `data/axis_1_positions.csv`

### لقطة سير العمل

| المرحلة | المكوّن | المخرج |
|---|---|---|
| Trigger | Tornado `/start` | تنفيذ التسلسل بشكل غير متزامن |
| Motion | متحكم FMC4030 | حركة المحور + استعلام المواقع |
| Lighting | Arduino تسلسلي (`'1'` / `'0'`) | التحكم في حالة LED |
| Sensing | EVK5 + Metavision | تيار الأحداث / تصدير CSV |
| Persistence | نظام الملفات المحلي | `data/*.csv`، مجلدات الدوران |

يتضمن المستودع أيضًا سكربتات كاميرا بديلة/قديمة، أدوات لمعالجة الإطارات بعد الالتقاط، ونماذج Metavision Python مدمجة.

## ✨ الميزات

- نقطة نهاية ويب في Tornado (`/start`) لإطلاق تسلسل الحركة/الالتقاط بشكل غير متزامن
- تسجيل أحداث EVK5 مع تفعيل قناة `MAIN` للـ trigger عبر Metavision HAL
- تصدير الأحداث إلى CSV مع طابع زمن الحدث وطابع زمن النظام
- غلاف تحكم محرك FMC4030 باستخدام `ctypes` وملف DLL للمورد
- تسجيل مواقع الحركة إلى CSV أثناء تحريك المحور
- التحكم بالـ LED عبر Arduino تسلسليًا (أوامر `'1'` / `'0'`)
- سكربتات مساعدة للإطارات (فحص شكل `.npy` وتحويل `.npy` إلى MP4)
- نماذج `python_samples/` الخاصة بـ Metavision مرفقة للتجارب والمرجعية

## 🗂️ هيكل المشروع

```text
.
├── app.py                                   # Main web orchestrator
├── event_sensor_evk5.py                     # EVK5 recorder (Metavision)
├── event_sensor.py                          # Alternative recorder (dv package)
├── evk5_test.py                             # Minimal EVK5 test recorder
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Arduino LED serial control
├── npy2video.py                             # Convert NPY frame stack -> MP4
├── npy_shape.py                             # Print frame-array shapes
├── motor_system.ini                         # Soft-origin config
├── cnc/
│   ├── cnc.py                               # FMC4030 control + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # Older controller scripts
├── led/
│   └── led.ino                              # Arduino firmware sketch
├── templates/
│   └── index.html                           # Start button UI
├── python_samples/                          # Metavision sample programs
├── data-0503/                               # Historical experiment datasets
├── i18n/                                    # Reserved for translated READMEs
└── .auto-readme-work/20260228_231403/      # README pipeline artifacts
```

## 🧰 المتطلبات المسبقة

### العتاد

- كاميرا أحداث متوافقة مع EVK5 والتعريفات/SDK
- متحكم حركة متوافق مع FMC4030 ويمكن الوصول إليه عبر IP/المنفذ المكوّن
- لوحة Arduino للتحكم في LED

### البرمجيات

- Python 3.x
- دعم المورّد/وقت التشغيل لـ:
  - وحدات Python الخاصة بـ Prophesee Metavision (`metavision_core`، `metavision_hal`، وملفات SDK المرتبطة)
  - تحميل DLL الخاص بـ FMC4030 عبر Python `ctypes` (`windll` داخل `cnc/cnc.py` يشير إلى Windows لمسار CNC)
- مكتبات Python المستخدمة عبر السكربتات:
  - `tornado`، `numpy`، `opencv-python`، `pytz`، `pyserial`
  - نظام بديل/اختياري: `dv`

### الإعدادات الافتراضية للبيئة

| الإعداد | الافتراضي | الموقع |
|---|---|---|
| منفذ Arduino التسلسلي | `COM4` | `app.py`، `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| منفذ CNC | `8088` | `cnc/cnc.py` |

ملاحظات:
- لا يوجد ملف `requirements.txt` أو `pyproject.toml` في لقطة المشروع الحالية.
- منفذ السيريال الافتراضي هو `COM4` في `app.py` و `led.py`.
- إعدادات الشبكة الافتراضية لـ CNC في `cnc/cnc.py`: IP `192.168.0.30`، المنفذ `8088`.

## 🔧 التثبيت

1. استنساخ المستودع:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. إنشاء وتفعيل بيئة Python:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. تثبيت تبعيات Python الأساسية المستخدمة من قِبل السكربتات الأساسية:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. تثبيت تبعيات SDK الخاصة بالكاميرا المطلوبة في بيئتك:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## 🧪 الاستخدام

### 1) تسلسل مُدار عبر الويب (الرئيسي)

نفّذ من جذر المستودع (مهم لمسارات النسبية داخل `app.py`):

```bash
python app.py
```

ثم افتح:

```text
http://localhost:8888
```

اضغط **Start Sequence** لبدء سير عمل الحركة/LED.

وسيطة اختيارية حالية تُحلّلها التطبيق:

```bash
python app.py --record_events True
```

ملاحظة سلوكية مهمة: الدالة `start_sequence()` تعيد تعيين `record_events = False` لاحقًا أثناء التنفيذ، لذلك قد يظل تسجيل EVK5 معطّلًا ما لم يتم تعديل الشفرة.

### 2) تسجيل أحداث EVK5 (CLI مباشر)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

المعاملات:
- `-i, --input`: مصدر/مسار الإدخال لجهاز EVK5 أو التسجيل
- `-d, --duration`: مدة التسجيل بالثواني
- `-z, --timezone`: تسمية المنطقة الزمنية لتنسيق الطابع الزمني
- `-o, --output`: اسم الإخراج الأساسي (يُحفظ تحت `data/<name>.csv`)

### 3) التحكم بـ CNC (CLI مباشر)

نفّذ من داخل `cnc/` حتى تُحل المسارات النسبية للـ DLL في `cnc.py` بشكل صحيح:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

الخيارات المتاحة الأخرى:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) سكربت اختبار LED

```bash
python led.py
```

حدّث المنفذ في الشفرة إذا لم تستخدم `COM4`.

### 5) الأدوات المساعدة

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

يوفّر `npy2video.py` الدالة `npy_to_video(npy_file_path, output_video_path, fps=5)` ويمكن استيرادها أو تعديلها لمساراتك المحلية.

## ⚙️ الإعداد

- `motor_system.ini` و`cnc/motor_system.ini`:
  - تخزين إحداثيات الأصل البرمجي (`ORIGIN` في قسم X/Y/Z)
- `app.py`:
  - منفذ Arduino التسلسلي: `ArduinoLED(port='COM4')`
  - مسار DLL الخاص بـ CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (مسار DV):
  - `DV_PORT` الافتراضي `7777`
  - `DV_PORT_FRAME` الافتراضي `7778`

## 📸 الأمثلة

### المثال A: بدء التسلسل الكامل عبر الويب

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

تشمل المخرجات المتوقعة:
- `data/axis_1_positions.csv` (مسار مواضع CNC)
- `data/<events>.csv` اختياريًا إذا كان تسجيل الأحداث مفعّلًا في المسار النشط

### المثال B: تسجيل أحداث EVK5 لمدة 60 ثانية

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

المخرجات المتوقعة:
- `data/run_001_events.csv`

### المثال C: تحريك محور Y ذهابًا وإيابًا من CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 ملاحظات التطوير

- يبدو أن المستودع الحالي هو مساحة بحث/نموذج أولي مع خليط من السكربتات النشطة والمؤرشفة.
- توجد ملفات مولدة كبيرة (`event_output.csv`، `data-0503/`) تم حفظها في الـ repository؛ فكّر في استراتيجية احتفاظ بالبيانات وتحديث `.gitignore` إذا كان الهدف توزيعه.
- يحتوي `python_samples/` على أمثلة SDK للكاميرا مفيدة، لكن قد تشتمل على تبعيات غير لازمة للتنسيق الأساسي.
- تحسين محتمل لجودة الكود:
  - يمكن تحسين معالجة boolean في `argparse` داخل `app.py` (`type=bool` غالبًا مضلِّل في تحليل CLI).
  - التعامل مع علامة تسجيل الأحداث في `start_sequence()` يتجاوز القيمة الابتدائية القادمة من CLI حاليًا.

## 🛠️ استكشاف الأخطاء وإصلاحها

- `ImportError: metavision_*` modules missing:
  - ثبّت/اضبط بيئة Metavision SDK الخاصة بـ Python.
- `ImportError: No module named dv`:
  - ثبّت حزمة DV لـ Python إذا كنت تستخدم مسار `event_sensor.py`.
- أخطاء تحميل DLL الخاصة بـ CNC:
  - تأكد من توافق نظام التشغيل وتوافر `FMC4030-Dll.dll` في المسار النسبي المتوقع.
  - شغّل `cnc.py` من دليل `cnc/` أو عدّل `dll_path`.
- أخطاء اتصال منفذ LED التسلسلي:
  - افحص تعيين منفذ Arduino (`COM4` مقابل المنفذ الفعلي).
  - تأكد من عدم احتجاز أي عملية أخرى للجهاز التسلسلي.
- لا توجد ملفات في `data/` بعد تشغيل الويب:
  - تحقق من صلاحيات الكتابة وهل مسار تسجيل الأحداث مفعل أم لا.

## 🗺️ خارطة الطريق

- إضافة ملف تبعيات (`requirements.txt` أو `pyproject.toml`) وإصدارات مقفلة
- توحيد الإعدادات الزمنية للتنفيذ (المنفذ، IP، مسار DLL، ملفات تعريف السرعة) في ملف إعدادات موحد
- توحيد واجهات الكاميرا الخلفية (EVK5 وDV) خلف واجهة واحدة مع تحديد واضح للوضع
- إضافة اختبارات/محاكيات لواجهات الحركة والمستشعر لتمكين CI دون عتاد
- إضافة سجلات تشغيل منظمة وميتاداتا لكل تجربة
- توليد وصيانة ملفات README مترجمة ضمن `i18n/`

## 🤝 المساهمة

المساهمات مرحّبة في:
- تحسينات تجريد العتاد
- إعداد أفضل وقابلية أعلى لإعادة الإنتاج
- توسيع التوثيق والترجمة
- فحوص السلامة وقيود تشغيلية إضافية للتحكم الحركي

سير المساهمة المقترح:
1. Fork وإنشاء فرع ميزة
2. إجراء تغييرات مركزة وقابلة للمراجعة
3. التحقق عبر إعداد العتاد لديك
4. إرسال طلب سحب (Pull Request) بخطوات قابلة للتكرار وسجلات

## الترخيص

لا يوجد ملف ترخيص في لقطة هذا المستودع.

الافتراض: جميع الحقوق محفوظة حتى يتم إضافة ترخيص صريح للمشروع. أضف ملف `LICENSE` لتحديد شروط إعادة الاستخدام.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
