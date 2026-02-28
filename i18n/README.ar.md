[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# التحكم في عتاد NHI والتقاط الأحداث

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

مشروع لتنسيق العتاد لتجارب الكاميرات الحدثية، ويجمع بين:
- التقاط أحداث كاميرا EVK5 (حزمة Prophesee Metavision)
- التحكم الحركي CNC المعتمد على FMC4030
- التحكم التسلسلي بإضاءة LED عبر Arduino
- واجهة ويب بسيطة للتحفيز (Tornado)

> هذا README هو أول مسودة مكتملة لهذه اللقطة من المستودع.
> افتراض: لم يكن هناك ملف `README.md` أساسي موجود مسبقًا في هذا الـ checkout، لذلك تم بناء هذا المستند من الشيفرة المصدرية ونتائج تحليل خط الأنابيب.

## نظرة عامة

سير العمل الشامل الأساسي مطبّق في `app.py`:

1. تدوير مجلد `data/` الحالي اختياريًا إلى مجلد مختوم بالوقت (`data_YYYYMMDD_HHMMSS`)
2. الاتصال بإضاءة Arduino LED (الافتراضي `COM4`)
3. تهيئة متحكم CNC عبر `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. تنفيذ تسلسل LED وحركة المحور Y
5. تسجيل أحداث EVK5 اختياريًا (معطّل حاليًا في المسار النشط؛ راجع الملاحظات أدناه)
6. حفظ سجلات مواضع المحور في `data/axis_1_positions.csv`

### لقطة سير العمل

| المرحلة | المكوّن | المخرجات |
|---|---|---|
| التحفيز | Tornado `/start` | تنفيذ تسلسلي لا تزامني |
| الحركة | متحكم FMC4030 | حركة محور + استطلاع الموضع |
| الإضاءة | Arduino serial (`'1'` / `'0'`) | التحكم بحالة LED |
| الاستشعار | EVK5 + Metavision | تدفق أحداث / تصدير CSV |
| التخزين | نظام الملفات المحلي | `data/*.csv`، مجلدات مدوّرة |

يتضمن المستودع أيضًا سكربتات كاميرا بديلة/قديمة، وأدوات للمعالجة اللاحقة للإطارات، وعينات Python مرفقة من Metavision.

## الميزات

- نقطة نهاية ويب Tornado (`/start`) لبدء تسلسل الحركة/الالتقاط بشكل لا تزامني
- تسجيل أحداث EVK5 مع تفعيل قناة التحفيز (`MAIN`) عبر Metavision HAL
- تصدير الأحداث إلى CSV مع الطوابع الزمنية للأحداث والنظام
- غلاف تحكم بمحرك FMC4030 باستخدام `ctypes` وDLL المورّد
- تسجيل مواضع الحركة إلى CSV أثناء تحرك المحور
- تحكم LED التسلسلي عبر Arduino (أوامر `'1'`/`'0'`)
- سكربتات أدوات الإطارات (فحص شكل `.npy` وتحويل `.npy` إلى MP4)
- أمثلة Metavision مرفقة داخل `python_samples/` للتجربة والمرجعية

## هيكل المشروع

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

## المتطلبات المسبقة

### العتاد

- كاميرا أحداث متوافقة مع EVK5 مع التعريفات/حزمة SDK
- متحكم حركة متوافق مع FMC4030 يمكن الوصول إليه عبر IP/port المضبوطين
- لوحة Arduino للتحكم في LED

### البرمجيات

- Python 3.x
- دعم المورّد/بيئة التشغيل لـ:
  - وحدات Python الخاصة بـ Prophesee Metavision (`metavision_core`, `metavision_hal`, ووحدات SDK ذات الصلة)
  - تحميل DLL الخاص بـ FMC4030 عبر Python `ctypes` (استخدام `windll` في `cnc/cnc.py` يشير إلى Windows لمسار CNC)
- مكتبات Python المستخدمة عبر السكربتات:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - حزمة اختيارية/بديلة: `dv`

### الإعدادات الافتراضية للبيئة

| الإعداد | القيمة الافتراضية | الموقع |
|---|---|---|
| منفذ Arduino التسلسلي | `COM4` | `app.py`, `led.py` |
| عنوان CNC | `192.168.0.30` | `cnc/cnc.py` |
| منفذ CNC | `8088` | `cnc/cnc.py` |

ملاحظات:
- لا يوجد ملف `requirements.txt` أو `pyproject.toml` في اللقطة الحالية.
- المنفذ التسلسلي مضبوط افتراضيًا على `COM4` في `app.py` و`led.py`.
- إعدادات شبكة CNC الافتراضية في `cnc/cnc.py`: العنوان `192.168.0.30`، المنفذ `8088`.

## التثبيت

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

3. تثبيت تبعيات Python الأساسية المستخدمة من السكربتات الجوهرية:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. تثبيت تبعيات SDK الخاصة بالكاميرا المطلوبة في بيئتك:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## الاستخدام

### 1) تسلسل مُدار عبر الويب (الأساسي)

شغّل من جذر المستودع (مهم للمسارات النسبية في `app.py`):

```bash
python app.py
```

ثم افتح:

```text
http://localhost:8888
```

انقر **Start Sequence** لتحفيز سير عمل الحركة/LED.

وسيطة اختيارية يجري تحليلها حاليًا في التطبيق:

```bash
python app.py --record_events True
```

ملاحظة سلوكية مهمة: الدالة `start_sequence()` تعيد ضبط `record_events = False` لاحقًا أثناء التنفيذ، لذلك قد يبقى تسجيل EVK5 معطّلًا ما لم يتم تعديل الشيفرة.

### 2) تسجيل أحداث EVK5 (CLI مباشر)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

المعاملات:
- `-i, --input`: مصدر/مسار الإدخال لجهاز EVK5 أو التسجيل
- `-d, --duration`: مدة التسجيل بالثواني
- `-z, --timezone`: تسمية المنطقة الزمنية لتنسيق الطابع الزمني
- `-o, --output`: اسم أساس المخرجات (يُحفَظ تحت `data/<name>.csv`)

### 3) تحكم CNC (CLI مباشر)

شغّل من `cnc/` لكي يُحل المسار النسبي لـ DLL في `cnc.py` بشكل صحيح:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

خيارات أخرى متاحة:

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

حدّث المنفذ في الشيفرة إذا لم تكن تستخدم `COM4`.

### 5) الأدوات

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

يوفر `npy2video.py` الدالة `npy_to_video(npy_file_path, output_video_path, fps=5)` ويمكن استيراده أو تعديله لمساراتك المحلية.

## الإعداد

- `motor_system.ini` و`cnc/motor_system.ini`:
  - حفظ إحداثيات الأصل البرمجي (`ORIGIN` للمحاور X/Y/Z)
- `app.py`:
  - منفذ Arduino التسلسلي: `ArduinoLED(port='COM4')`
  - مسار DLL الخاص بـ CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (مسار DV):
  - القيمة الافتراضية لـ `DV_PORT` هي `7777`
  - القيمة الافتراضية لـ `DV_PORT_FRAME` هي `7778`

## أمثلة

### المثال A: بدء التسلسل الكامل عبر الويب

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

المخرجات المتوقعة تتضمن:
- `data/axis_1_positions.csv` (أثر موضع CNC)
- `data/<events>.csv` اختياريًا إذا كان تسجيل الأحداث مفعّلًا في المسار النشط

### المثال B: تسجيل أحداث EVK5 لمدة 60 ثانية

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

المخرجات المتوقعة:
- `data/run_001_events.csv`

### المثال C: تحريك المحور Y ذهابًا وإيابًا من CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## ملاحظات التطوير

- يبدو المستودع الحالي مساحة عمل للبحث/النمذجة الأولية، مع خليط من سكربتات نشطة ومؤرشفة.
- تم عمل commit لملفات ناتجة كبيرة (`event_output.csv`, `data-0503/`)؛ يُنصح بوضع استراتيجية للاحتفاظ بالبيانات وتحديث `.gitignore` إذا كان سيتم توزيع هذا المستودع.
- يحتوي `python_samples/` على أمثلة مفيدة من SDK الكاميرا، لكنه قد يتضمن تبعيات غير مطلوبة للتنسيق الأساسي.
- تحسينات محتملة على جودة الشيفرة:
  - يمكن تحسين التعامل مع القيم المنطقية في `argparse` داخل `app.py` (`type=bool` غالبًا ما يكون مُضللًا في تحليل CLI).
  - التعامل مع علم تسجيل الأحداث في `start_sequence()` يتجاوز حاليًا قيمة CLI الأولية.

## استكشاف الأخطاء وإصلاحها

- `ImportError: metavision_*` الوحدات مفقودة:
  - ثبّت/اضبط بيئة Metavision SDK الخاصة بـ Python.
- `ImportError: No module named dv`:
  - ثبّت حزمة DV لـ Python إذا كنت تستخدم مسار `event_sensor.py`.
- فشل تحميل DLL الخاص بـ CNC:
  - تأكد من توافق نظام التشغيل وأن `FMC4030-Dll.dll` متاح في المسار النسبي المتوقع.
  - شغّل `cnc.py` من مجلد `cnc/` أو عدّل `dll_path`.
- أخطاء الاتصال التسلسلي لـ LED:
  - تحقق من تعيين منفذ Arduino (`COM4` مقابل المنفذ الفعلي).
  - تأكد من عدم وجود عملية أخرى تستخدم الجهاز التسلسلي.
- لا توجد ملفات في `data/` بعد التشغيل عبر الويب:
  - تحقّق من أذونات الكتابة وما إذا كان مسار تسجيل الأحداث مفعّلًا.

## خارطة الطريق

- إضافة ملف تبعيات (`requirements.txt` أو `pyproject.toml`) مع تثبيت الإصدارات
- فصل إعدادات التشغيل (المنافذ، IP، مسار DLL، ملفات تعريف السرعة) في ملف إعداد موحّد
- توحيد واجهات الكاميرا الخلفية (EVK5 وDV) خلف واجهة واحدة مع اختيار وضع واضح
- إضافة اختبارات/محاكاة لواجهات الحركة والاستشعار لتمكين CI بدون عتاد
- إضافة تسجيلات منظّمة وبيانات وصفية لكل تجربة
- توليد وصيانة ملفات README المترجمة تحت `i18n/`

## المساهمة

المساهمات مرحّب بها في:
- تحسينات تجريد العتاد
- تحسين الإعداد وإمكانية إعادة الإنتاج
- توسيع التوثيق والترجمة
- فحوصات السلامة وحواجز التشغيل للتحكم الحركي

تدفق مساهمة مقترح:
1. اعمل Fork وأنشئ فرع ميزة
2. نفّذ تغييرات مركّزة وقابلة للمراجعة
3. تحقّق من العمل على إعداد العتاد لديك
4. قدّم Pull Request مع خطوات قابلة لإعادة الإنتاج وسجلات التنفيذ

## الترخيص

لا يوجد ملف ترخيص في لقطة هذا المستودع.

افتراض: جميع الحقوق محفوظة حتى تتم إضافة ترخيص للمشروع بشكل صريح. أضف ملف `LICENSE` لتحديد شروط إعادة الاستخدام.

## الدعم

لم يتم العثور على بيانات sponsor/donation في هذه اللقطة. إذا أردت تضمين روابط دعم، أضفها هنا وفي ملفات README المترجمة.
