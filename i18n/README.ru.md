[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Управление аппаратурой NHI и захватом событий

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Быстрая навигация

| Раздел | Назначение |
|---|---|
| [Установка](#installation) | Подготовка окружения и зависимостей |
| [Использование](#usage) | Запуск веб-оркестратора и CLI-процессов |
| [Конфигурация](#configuration) | Настройка последовательности: сериал, сеть и значения по умолчанию |
| [Примеры](#examples) | Запуск практических примеров команд |
| [Устранение неполадок](#troubleshooting) | Устранение типовых проблем |

## 🧭 Проект с позиции эксперимента

| Направление | Детали |
|---|---|
| Миссия | Координация событийного захвата, позиционирования движения и LED-индикации для воспроизводимых лабораторных сценариев |
| Основной вход | `app.py` (веб-триггер Tornado + асинхронная оркестрация последовательности) |
| Ключевые входные данные | Поток событий EVK5, управление осью FMC4030, команды LED через Arduino по serial |
| Основные выходные данные | `data/axis_1_positions.csv`, опциональные экспортируемые CSV с событиями |
| Платформы | Windows / Linux (зависит от доступности SDK и драйверов) |

Проект для оркестрации оборудования в экспериментах с event-камерами, объединяющий:
- захват с event-камеры EVK5 (стек Prophesee Metavision)
- управление движением на базе FMC4030
- управление LED через Arduino по serial
- минимальный веб-интерфейс запуска (Tornado)

> Предположение: параметры оборудования, DLL и окружения SDK отличаются по хостам и определяются из кода проекта; фактическое поведение команд может зависеть от ОС, версий драйверов и доступности компонентов во время выполнения.

## 🧠 Обзор

Основной сквозной workflow реализован в `app.py`:

1. По желанию переносит существующий каталог `data/` в временную папку с меткой времени (`data_YYYYMMDD_HHMMSS`)
2. Подключается к Arduino LED (по умолчанию `COM4`)
3. Инициализирует CNC-контроллер через `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Запускает последовательность LED и движения по оси Y
5. По желанию записывает события EVK5 (на данный момент в активном пайплайне отключено; см. замечание ниже)
6. Сохраняет логи позиции оси в `data/axis_1_positions.csv`

### Снимок рабочего процесса

| Этап | Компонент | Результат |
|---|---|---|
| Триггер | Tornado `/start` | Асинхронное выполнение последовательности |
| Движение | Контроллер FMC4030 | Перемещение оси + опрос позиции |
| Освещение | Arduino serial (`'1'` / `'0'`) | Управление состоянием LED |
| Съёмка | EVK5 + Metavision | Поток событий / экспорт CSV |
| Хранение | Локальная файловая система | `data/*.csv`, ротация папок |

В репозитории также есть альтернативные/устаревшие скрипты камеры, утилиты постобработки кадров и поставляемые примеры Metavision для Python.

## ✨ Особенности

- Веб-эндпоинт Tornado (`/start`) для асинхронного запуска последовательности движения и захвата
- Запись событий EVK5 с включением канала триггера (`MAIN`) через Metavision HAL
- Экспорт событий в CSV с временными метками события и системного времени
- Обёртка управления приводом FMC4030 с использованием `ctypes` и DLL производителя
- Логирование осевой позиции в CSV во время движения
- Управление LED по serial интерфейсу Arduino (команды `'1'`/`'0'`)
- Скрипты утилит для кадров (`.npy` инспекция формы и конвертация `.npy` в MP4)
- В комплекте `python_samples/` с примерами Metavision для экспериментов и сравнения

## 🗂️ Структура проекта

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

## 🧰 Пререквизиты

### Оборудование

- Event-камера EVK5 и её драйверы/SDK
- Контроллер движения FMC4030 с доступом по настроенному IP/порту
- Плата Arduino для управления LED

### Программное обеспечение

- Python 3.x
- Поддержка vendor/runtime для:
  - Python-модулей Prophesee Metavision (`metavision_core`, `metavision_hal`, связанные модули SDK)
  - Загрузки DLL FMC4030 через Python `ctypes` (использование `windll` в `cnc/cnc.py` подразумевает путь Windows для CNC)
- Python-библиотеки, используемые в скриптах:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Опциональный/альтернативный стек: `dv`

### Значения окружения по умолчанию

| Параметр | По умолчанию | Расположение |
|---|---|---|
| Сериал Arduino | `COM4` | `app.py`, `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| Порт CNC | `8088` | `cnc/cnc.py` |

Примечания:
- В текущем снимке нет `requirements.txt` или `pyproject.toml`.
- Значение serial-порта по умолчанию `COM4` указано в `app.py` и `led.py`.
- Настройки CNC по умолчанию в `cnc/cnc.py`: IP `192.168.0.30`, порт `8088`.

<a id="installation"></a>
## 🔧 Установка

1. Клонируйте репозиторий:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Создайте и активируйте окружение Python:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Установите базовые зависимости Python, используемые основными скриптами:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Установите зависимости SDK камеры, необходимые в вашей среде:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

<a id="usage"></a>
## 🧪 Использование

### 1) Веб-оркестрация последовательности (основной режим)

Запускайте из корня репозитория (это важно для корректной работы относительных путей в `app.py`):

```bash
python app.py
```

Затем откройте:

```text
http://localhost:8888
```

Нажмите **Start Sequence**, чтобы запустить сценарий движения и LED.

Дополнительно обрабатываемый флаг:

```bash
python app.py --record_events True
```

Важно: `start_sequence()` в ходе выполнения позже устанавливает `record_events = False`, поэтому запись EVK5 может оставаться отключённой, пока код не будет скорректирован.

### 2) Запись событий EVK5 (непосредственно через CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Аргументы:
- `-i, --input`: Исходный источник/путь для устройства EVK5 или записи
- `-d, --duration`: Время записи в секундах
- `-z, --timezone`: Метка часового пояса для форматирования временных меток
- `-o, --output`: Базовое имя результата (сохраняется в `data/<name>.csv`)

### 3) Управление CNC (непосредственно через CLI)

Запускайте из каталога `cnc/`, чтобы относительный путь к DLL в `cnc.py` корректно разрешался:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Другие доступные опции:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Тестовый скрипт LED

```bash
python led.py
```

Обновите порт в коде, если вы используете порт, отличный от `COM4`.

### 5) Утилиты

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` предоставляет функцию `npy_to_video(npy_file_path, output_video_path, fps=5)` и может быть импортирована или изменена под локальные пути.

<a id="configuration"></a>
## ⚙️ Конфигурация

- `motor_system.ini` и `cnc/motor_system.ini`:
  - Сохраняют программный ноль (`ORIGIN` section для X/Y/Z)
- `app.py`:
  - Serial-порт Arduino: `ArduinoLED(port='COM4')`
  - Путь к CNC DLL: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (маршрут DV):
  - Значение по умолчанию `DV_PORT`: `7777`
  - Значение по умолчанию `DV_PORT_FRAME`: `7778`

<a id="examples"></a>
## 📸 Примеры

### Пример A: запуск полной последовательности через веб

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Ожидаемые результаты:
- `data/axis_1_positions.csv` (трасса позиции CNC)
- Опционально `data/<events>.csv`, если запись событий включена в активном пути

### Пример B: запись событий EVK5 в течение 60 секунд

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Ожидаемый результат:
- `data/run_001_events.csv`

### Пример C: перемещение оси Y туда и обратно через CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 Заметки по разработке

- Текущий репозиторий выглядит как исследовательский/прототипный рабочий набор с активными и архивными скриптами.
- Большие сгенерированные артефакты (`event_output.csv`, `data-0503/`) добавлены в репозиторий; при публикации стоит продумать политику хранения данных и обновить `.gitignore`.
- В `python_samples/` есть полезные примеры SDK камеры, однако могут быть зависимости, не нужные для базовой оркестрации.
- Потенциальные улучшения качества кода:
  - Обработка булевых аргументов `argparse` в `app.py` может быть улучшена (`type=bool` для CLI часто вводит в заблуждение).
  - Флаг записи событий в `start_sequence()` сейчас перекрывает первоначальное значение из CLI.

<a id="troubleshooting"></a>
## 🛠️ Устранение неполадок

- `ImportError: metavision_*` отсутствуют модули:
  - Установите и настройте окружение Prophesee Metavision SDK для Python.
- `ImportError: No module named dv`:
  - Установите пакет DV Python, если используется путь `event_sensor.py`.
- Ошибки загрузки DLL CNC:
  - Проверьте совместимость ОС и наличие `FMC4030-Dll.dll` по ожидаемому относительному пути.
  - Запускайте `cnc.py` из каталога `cnc/` или скорректируйте `dll_path`.
- Ошибки подключения LED по serial:
  - Проверьте назначение порта Arduino (`COM4` против фактического порта).
  - Убедитесь, что устройство не занято другим процессом.
- Нет файлов в `data/` после веб-запуска:
  - Проверьте права записи и активирована ли опция пути записи событий.

## 🗺️ План развития

- Добавить файл зависимостей (`requirements.txt` или `pyproject.toml`) и зафиксированные версии
- Вынести параметры запуска (порты, IP, путь к DLL, профили скорости) в единый файл конфигурации
- Унифицировать камеры (EVK5 и DV) через единый интерфейс с ясным выбором режима
- Добавить тесты/моки для motion- и сенсорных интерфейсов, чтобы CI работала без реального оборудования
- Добавить структурированное логирование и метаданные прогонов для каждого эксперимента
- Поддерживать и обновлять переводы README в каталоге `i18n/`

## 🤝 Участие

Приветствуются вклад и доработки:
- Улучшения аппаратной абстракции
- Лучшая конфигурация и воспроизводимость
- Расширение документации и переводов
- Защиты безопасности и эксплуатационные ограждения для управления движением

Предлагаемый процесс внесения изменений:
1. Сделайте форк и создайте feature-ветку
2. Вносите точечные, удобные для ревью изменения
3. Проверяйте на своей конфигурации аппаратуры
4. Отправьте pull request с воспроизводимыми шагами и логами

## Лицензия

Файл лицензии в текущем снимке репозитория отсутствует.

Предположение: все права защищены, пока не добавлена явная лицензия проекта. Добавьте файл `LICENSE`, чтобы определить условия повторного использования.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
