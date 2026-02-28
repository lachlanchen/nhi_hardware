[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Control de Hardware NHI y Captura de Eventos

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Navegación rápida

| Sección | Propósito |
|---|---|
| [Instalación](#instalación) | Preparar entorno y dependencias |
| [Uso](#uso) | Ejecutar el orquestador web y flujos CLI |
| [Configuración](#configuración) | Ajustar serial, red y valores por defecto |
| [Ejemplos](#ejemplos) | Ejecutar ejemplos prácticos de comandos |
| [Solución de problemas](#solución-de-problemas) | Resolver incidencias comunes de configuración |

## 🧭 Resumen del proyecto

| Enfoque | Detalles |
|---|---|
| Misión | Coordinar captura de eventos, control de movimiento y señalización LED para flujos de laboratorio repetibles |
| Entrada principal | `app.py` (trigger web de Tornado + orquestación asíncrona de secuencias) |
| Entradas clave | Flujo de eventos EVK5, control de eje FMC4030, comandos LED por serial de Arduino |
| Resultados principales | `data/axis_1_positions.csv`, exportaciones CSV de eventos opcionales |
| Plataformas | Windows / Linux (según disponibilidad de SDK y hardware) |

Proyecto de orquestación de hardware para experimentos con cámara de eventos que combina:
- Captura con cámara EVK5 (stack Prophesee Metavision)
- Control de movimiento CNC basado en FMC4030
- Control serial de LEDs con Arduino
- UI web mínima de disparo (Tornado)

> Supuesto: el hardware, DLL y entornos de SDK varían según el host y se infieren del código del proyecto; el comportamiento exacto de los comandos puede cambiar según el sistema operativo, la versión del controlador y la disponibilidad en tiempo de ejecución.

## 🧠 Visión general

El flujo end-to-end principal está implementado en `app.py`:

1. Opcionalmente mueve el contenido existente de `data/` a una carpeta con marca temporal (`data_YYYYMMDD_HHMMSS`)
2. Conecta el LED de Arduino (por defecto `COM4`)
3. Inicializa el controlador CNC mediante `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Ejecuta la secuencia de LED y movimiento del eje Y
5. Opcionalmente registra eventos EVK5 (actualmente deshabilitado en el flujo activo; ver notas abajo)
6. Guarda los registros de posición del eje en `data/axis_1_positions.csv`

### Resumen del flujo

| Etapa | Componente | Resultado |
|---|---|---|
| Trigger | Tornado `/start` | Ejecución asíncrona de secuencia |
| Movimiento | Controlador FMC4030 | Movimiento de eje + lectura de posición |
| Iluminación | Serial de Arduino (`'1'` / `'0'`) | Control de estado del LED |
| Sensado | EVK5 + Metavision | Flujo de eventos / exportación CSV |
| Persistencia | Sistema de archivos local | `data/*.csv`, carpetas rotadas |

El repositorio también incluye scripts alternativos/legados de cámara, utilidades para post-procesamiento de fotogramas y ejemplos de `Metavision` de Python integrados.

## ✨ Características

- Endpoint web de Tornado (`/start`) para lanzar secuencias de movimiento/captura de forma asíncrona
- Registro de eventos EVK5 con activación del canal de disparo (`MAIN`) vía Metavision HAL
- Exportación CSV de eventos con marcas de tiempo de evento y de sistema
- Wrapper de control de motor FMC4030 usando `ctypes` y DLL del fabricante
- Registro de posición durante el movimiento del eje a CSV
- Control LED de Arduino por serial (`'1'`/`'0'`)
- Scripts de utilidad para fotogramas (`inspección de forma de `.npy` y conversión de `.npy` a MP4)
- Ejemplos `python_samples/` de Metavision incluidos para experimentar y consultar

## 🗂️ Estructura del proyecto

```text
.
├── app.py                                   # Orquestador web principal
├── event_sensor_evk5.py                     # Grabador EVK5 (Metavision)
├── event_sensor.py                          # Grabador alternativo (ruta dv)
├── evk5_test.py                             # Prueba mínima de grabación EVK5
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Control serial de LED Arduino
├── npy2video.py                             # Convertir pila NPY de fotogramas -> MP4
├── npy_shape.py                             # Mostrar formas de arrays de fotogramas
├── motor_system.ini                         # Configuración de origen suave
├── cnc/
│   ├── cnc.py                               # Control FMC4030 + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # Scripts de controlador antiguos
├── led/
│   └── led.ino                              # Firmware de Arduino
├── templates/
│   └── index.html                           # UI del botón de inicio
├── python_samples/                          # Programas de ejemplo de Metavision
├── data-0503/                               # Conjuntos de datos experimentales históricos
├── i18n/                                    # Reservado para READMEs traducidos
└── .auto-readme-work/20260228_231403/      # Artefactos de pipeline del README
```

## 🧰 Requisitos previos

### Hardware

- Cámara de eventos compatible con EVK5 y drivers/SDK
- Controlador de movimiento compatible con FMC4030 accesible en la IP/puerto configurados
- Placa Arduino para control de LED

### Software

- Python 3.x
- Soporte de proveedor/ejecución para:
  - Módulos Python de Prophesee Metavision (`metavision_core`, `metavision_hal`, módulos SDK relacionados)
  - Carga de DLL de FMC4030 via `ctypes` de Python (`windll` en `cnc/cnc.py` implica ruta Windows para el camino de CNC)
- Librerías Python usadas en los scripts:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Stack opcional/alternativo: `dv`

### Valores por defecto de entorno

| Parámetro | Valor por defecto | Ubicación |
|---|---|---|
| Puerto serial de Arduino | `COM4` | `app.py`, `led.py` |
| IP del CNC | `192.168.0.30` | `cnc/cnc.py` |
| Puerto del CNC | `8088` | `cnc/cnc.py` |

Notas:
- No hay `requirements.txt` ni `pyproject.toml` en la instantánea actual.
- El puerto serial por defecto es `COM4` en `app.py` y `led.py`.
- Configuración de red CNC predeterminada en `cnc/cnc.py`: IP `192.168.0.30`, puerto `8088`.

## 🔧 Instalación

1. Clona el repositorio:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Crea y activa un entorno de Python:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Instala las dependencias base de Python usadas por los scripts principales:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Instala las dependencias del SDK de cámara requeridas en tu entorno:

```bash
# Placeholder de ejemplo: instala los paquetes de Python de Prophesee Metavision
# Sigue las instrucciones de tu distribución/SDK para tu SO y versión de cámara.
```

## 🧪 Uso

### 1) Secuencia orquestada por web (principal)

Ejecuta desde la raíz del repositorio (importante para rutas relativas en `app.py`):

```bash
python app.py
```

Luego abre:

```text
http://localhost:8888
```

Haz clic en **Start Sequence** para disparar el flujo de movimiento/LED.

Argumento opcional actualmente analizado por la app:

```bash
python app.py --record_events True
```

Nota de comportamiento importante: `start_sequence()` vuelve a establecer `record_events = False` más adelante en la ejecución, por lo que la grabación EVK5 puede quedar desactivada si no se ajusta el código.

### 2) Grabación de eventos EVK5 (CLI directa)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Argumentos:
- `-i, --input`: Origen de entrada/ruta del dispositivo EVK5 o grabación
- `-d, --duration`: Duración de la grabación en segundos
- `-z, --timezone`: Etiqueta de huso horario para el formato de marcas de tiempo
- `-o, --output`: Nombre base de salida (guardado en `data/<name>.csv`)

### 3) Control CNC (CLI directa)

Ejecuta desde `cnc/` para que la ruta relativa de la DLL en `cnc.py` se resuelva correctamente:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Otras opciones disponibles:

```bash
# Establecer la posición actual como origen suave
python cnc.py --set-origin

# Mover a coordenadas absolutas x,y,z
python cnc.py --move 0,30,0 --speed 20

# Establecer origen después de mover a coordenadas proporcionadas
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script de prueba de LED

```bash
python led.py
```

Actualiza el puerto en el código si no usas `COM4`.

### 5) Utilidades

```bash
# Mostrar la forma del array de fotogramas desde una carpeta
python npy_shape.py data_20250101_120000

# Mostrar la forma del array de fotogramas para todas las carpetas data_* en cwd
python npy_shape.py -a
```

`npy2video.py` expone `npy_to_video(npy_file_path, output_video_path, fps=5)` y puede importarse o editarse para tus rutas locales.

## ⚙️ Configuración

- `motor_system.ini` y `cnc/motor_system.ini`:
  - Persisten coordenadas de origen de software (`sección ORIGIN` para X/Y/Z)
- `app.py`:
  - Puerto serial de Arduino: `ArduinoLED(port='COM4')`
  - Ruta de DLL del CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (ruta DV):
  - `DV_PORT` por defecto `7777`
  - `DV_PORT_FRAME` por defecto `7778`

## 📸 Ejemplos

### Ejemplo A: Iniciar secuencia completa desde web

```bash
python app.py
# visita http://localhost:8888 y pulsa Start Sequence
```

Resultados esperados:
- `data/axis_1_positions.csv` (traza de posición CNC)
- `data/<events>.csv` opcional si el registro de eventos está habilitado en la ruta activa

### Ejemplo B: Grabar eventos EVK5 durante 60 segundos

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Resultado esperado:
- `data/run_001_events.csv`

### Ejemplo C: Mover eje Y de ida y vuelta desde CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 Notas de desarrollo

- El repositorio actual parece ser un espacio de trabajo de investigación/prototipado con scripts activos y archivados mezclados.
- Existen artefactos grandes generados (`event_output.csv`, `data-0503/`) versionados; considera una estrategia de retención de datos y actualizaciones de `.gitignore` si este repositorio se distribuirá.
- `python_samples/` contiene ejemplos útiles del SDK de cámara, pero puede incluir dependencias que no son necesarias para la orquestación central.
- Posibles mejoras de calidad:
  - El manejo de booleanos con `argparse` en `app.py` puede mejorarse (`type=bool` suele ser confuso en CLI).
  - El control del flag de grabación en `start_sequence()` actualmente sobrescribe el valor inicial de CLI.

## 🛠️ Solución de problemas

- `ImportError: metavision_*` módulos inexistentes:
  - Instala/configura tu entorno de Python del SDK Metavision.
- `ImportError: No module named dv`:
  - Instala el paquete Python DV si utilizas la ruta `event_sensor.py`.
- Fallos al cargar la DLL CNC:
  - Confirma compatibilidad del SO y que `FMC4030-Dll.dll` esté disponible en la ruta relativa esperada.
  - Ejecuta `cnc.py` desde el directorio `cnc/` o ajusta `dll_path`.
- Errores de conexión serial de LED:
  - Comprueba el puerto asignado a Arduino (`COM4` frente al puerto real).
  - Asegúrate de que ningún otro proceso esté bloqueando el dispositivo serial.
- No se crean archivos en `data/` tras ejecutar por web:
  - Verifica permisos de escritura y si la ruta de grabación de eventos está habilitada.

## 🗺️ Hoja de ruta

- Añadir manifiesto de dependencias (`requirements.txt` o `pyproject.toml`) y versiones ancladas
- Externalizar la configuración de ejecución (puertos, IP, ruta de DLL, perfiles de velocidad) en un único archivo de configuración
- Unificar backends de cámara (EVK5 y DV) bajo una interfaz común con selección clara de modo
- Añadir pruebas/mocks para interfaces de movimiento y sensores y permitir CI sin hardware
- Generar y mantener registros de logs estructurados y metadatos por experimento
- Generar y mantener README traducidos en `i18n/`

## 🤝 Contribuciones

Las contribuciones son bienvenidas para:
- Mejoras de abstracción de hardware
- Mejor configuración y reproducibilidad
- Expansión y traducción de documentación
- Verificaciones de seguridad y límites operativos para control de movimiento

Flujo sugerido de contribución:
1. Haz un fork y crea una rama de características
2. Realiza cambios acotados y revisables
3. Valida con tu configuración de hardware
4. Envía un pull request con pasos reproducibles y registros

## Licencia

No existe un archivo de licencia en esta instantánea del repositorio.

Supuesto: todos los derechos están reservados hasta que se añada explícitamente una licencia del proyecto. Añade un archivo `LICENSE` para definir las condiciones de reutilización.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
