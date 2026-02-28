[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Control de Hardware NHI y Captura de Eventos

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

Un proyecto de orquestación de hardware para experimentos con cámaras de eventos que combina:
- Captura con cámara de eventos EVK5 (stack Prophesee Metavision)
- Control de movimiento CNC basado en FMC4030
- Control de LED por serie con Arduino
- Interfaz web mínima de disparo (Tornado)

> Este README es el primer borrador completo para esta instantánea del repositorio.
> Suposición: no había un `README.md` preexistente en la raíz en este checkout, por lo que este documento se construye a partir del código fuente y de artefactos de análisis del pipeline.

## Visión general

El flujo principal de extremo a extremo está implementado en `app.py`:

1. Opcionalmente rota `data/` existente a una carpeta con marca temporal (`data_YYYYMMDD_HHMMSS`)
2. Conecta el LED de Arduino (por defecto `COM4`)
3. Inicializa el controlador CNC mediante `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Ejecuta la secuencia de LED y movimiento del eje Y
5. Opcionalmente registra eventos EVK5 (actualmente deshabilitado en el flujo activo; ver notas más abajo)
6. Guarda logs de posición del eje en `data/axis_1_positions.csv`

### Resumen del flujo

| Etapa | Componente | Salida |
|---|---|---|
| Disparo | Tornado `/start` | Ejecución asíncrona de secuencia |
| Movimiento | Controlador FMC4030 | Movimiento de eje + sondeo de posición |
| Iluminación | Serie Arduino (`'1'` / `'0'`) | Control de estado del LED |
| Sensado | EVK5 + Metavision | Flujo de eventos / exportación CSV |
| Persistencia | Sistema de archivos local | `data/*.csv`, carpetas rotadas |

El repositorio también contiene scripts alternativos/heredados de cámara, utilidades para posprocesamiento de frames y ejemplos Python de Metavision incluidos.

## Características

- Endpoint web de Tornado (`/start`) para lanzar una secuencia de movimiento/captura de forma asíncrona
- Registro de eventos EVK5 con habilitación de canal de disparo (`MAIN`) mediante Metavision HAL
- Exportación de eventos a CSV con marcas de tiempo tanto de evento como de sistema
- Wrapper de control de motor FMC4030 usando `ctypes` y DLL del proveedor
- Registro de posiciones de movimiento a CSV durante el desplazamiento del eje
- Control de LED por serie con Arduino (comandos `'1'`/`'0'`)
- Scripts utilitarios para frames (inspección de forma de `.npy` y conversión de `.npy` a MP4)
- Ejemplos de Metavision incluidos en `python_samples/` para experimentación y referencia

## Estructura del proyecto

```text
.
├── app.py                                   # Orquestador web principal
├── event_sensor_evk5.py                     # Grabador EVK5 (Metavision)
├── event_sensor.py                          # Grabador alternativo (paquete dv)
├── evk5_test.py                             # Grabador de prueba mínimo para EVK5
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Control serie del LED con Arduino
├── npy2video.py                             # Convierte stack de frames NPY -> MP4
├── npy_shape.py                             # Imprime formas de arreglos de frames
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
│   └── led.ino                              # Sketch de firmware para Arduino
├── templates/
│   └── index.html                           # UI con botón de inicio
├── python_samples/                          # Programas de ejemplo de Metavision
├── data-0503/                               # Datasets históricos de experimentos
├── i18n/                                    # Reservado para READMEs traducidos
└── .auto-readme-work/20260228_231403/      # Artefactos del pipeline del README
```

## Requisitos previos

### Hardware

- Cámara de eventos compatible con EVK5 y drivers/SDK
- Controlador de movimiento compatible con FMC4030 accesible en IP/puerto configurados
- Placa Arduino para control del LED

### Software

- Python 3.x
- Soporte de proveedor/runtime para:
  - Módulos Python de Prophesee Metavision (`metavision_core`, `metavision_hal`, módulos SDK relacionados)
  - Carga de DLL FMC4030 vía Python `ctypes` (el uso de `windll` en `cnc/cnc.py` implica Windows para la ruta CNC)
- Bibliotecas de Python usadas en los scripts:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Stack opcional/alternativo: `dv`

### Valores predeterminados del entorno

| Configuración | Predeterminado | Ubicación |
|---|---|---|
| Puerto serie de Arduino | `COM4` | `app.py`, `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| Puerto CNC | `8088` | `cnc/cnc.py` |

Notas:
- No hay `requirements.txt` ni `pyproject.toml` en la instantánea actual.
- El puerto serie por defecto es `COM4` en `app.py` y `led.py`.
- Ajustes de red CNC por defecto en `cnc/cnc.py`: IP `192.168.0.30`, puerto `8088`.

## Instalación

1. Clonar el repositorio:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Crear y activar un entorno de Python:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Instalar dependencias base de Python usadas por los scripts principales:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Instalar las dependencias del SDK de cámara requeridas en tu entorno:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## Uso

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

Argumento opcional que actualmente procesa la app:

```bash
python app.py --record_events True
```

Nota de comportamiento importante: `start_sequence()` actualmente vuelve a establecer `record_events = False` más adelante en la ejecución, por lo que el registro EVK5 puede permanecer deshabilitado a menos que se ajuste el código.

### 2) Registro de eventos EVK5 (CLI directo)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Argumentos:
- `-i, --input`: Fuente/ruta de entrada para dispositivo EVK5 o grabación
- `-d, --duration`: Duración de grabación en segundos
- `-z, --timezone`: Etiqueta de zona horaria para formateo de marcas de tiempo
- `-o, --output`: Nombre base de salida (se guarda en `data/<name>.csv`)

### 3) Control CNC (CLI directo)

Ejecuta desde `cnc/` para que la ruta relativa de DLL en `cnc.py` se resuelva correctamente:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Otras opciones disponibles:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script de prueba de LED

```bash
python led.py
```

Actualiza el puerto en el código si no usas `COM4`.

### 5) Utilidades

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` proporciona `npy_to_video(npy_file_path, output_video_path, fps=5)` y puede importarse o editarse para tus rutas locales.

## Configuración

- `motor_system.ini` y `cnc/motor_system.ini`:
  - Persisten coordenadas de origen de software (sección `ORIGIN` para X/Y/Z)
- `app.py`:
  - Puerto serie de Arduino: `ArduinoLED(port='COM4')`
  - Ruta DLL CNC: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (ruta DV):
  - `DV_PORT` predeterminado `7777`
  - `DV_PORT_FRAME` predeterminado `7778`

## Ejemplos

### Ejemplo A: Iniciar secuencia completa vía web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Las salidas esperadas incluyen:
- `data/axis_1_positions.csv` (traza de posición CNC)
- `data/<events>.csv` opcional si el registro de eventos está habilitado en la ruta activa

### Ejemplo B: Registrar eventos EVK5 durante 60 segundos

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Salida esperada:
- `data/run_001_events.csv`

### Ejemplo C: Mover el eje Y de ida y vuelta desde CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## Notas de desarrollo

- El repositorio actual parece ser un espacio de trabajo de investigación/prototipado con scripts activos y archivados mezclados.
- Artefactos generados grandes (`event_output.csv`, `data-0503/`) están versionados; considera una estrategia de retención de datos y actualizaciones de `.gitignore` si este repositorio se va a distribuir.
- `python_samples/` contiene ejemplos útiles del SDK de cámara, pero puede incluir dependencias no requeridas para la orquestación principal.
- Mejora potencial de calidad de código:
  - El manejo booleano con `argparse` en `app.py` puede mejorarse (`type=bool` suele ser engañoso en el parseo CLI).
  - El manejo de la bandera de registro de eventos en `start_sequence()` actualmente sobrescribe el valor inicial de CLI.

## Solución de problemas

- `ImportError: metavision_*` módulos faltantes:
  - Instala/configura tu entorno Python del SDK Metavision.
- `ImportError: No module named dv`:
  - Instala el paquete Python DV si usas la ruta de `event_sensor.py`.
- Fallos al cargar DLL de CNC:
  - Confirma compatibilidad de SO y que `FMC4030-Dll.dll` esté disponible en la ruta relativa esperada.
  - Ejecuta `cnc.py` desde el directorio `cnc/` o ajusta `dll_path`.
- Errores de conexión serie del LED:
  - Verifica la asignación del puerto de Arduino (`COM4` vs puerto real).
  - Asegura que ningún otro proceso esté usando el dispositivo serie.
- No hay archivos en `data/` después de la ejecución web:
  - Verifica permisos de escritura y si la ruta de registro de eventos está habilitada.

## Hoja de ruta

- Añadir manifiesto de dependencias (`requirements.txt` o `pyproject.toml`) y versiones fijadas
- Externalizar configuración de runtime (puertos, IP, ruta DLL, perfiles de velocidad) a un archivo de configuración unificado
- Normalizar backends de cámara (EVK5 y DV) detrás de una interfaz con selección de modo clara
- Añadir tests/mocks para interfaces de movimiento y sensores para habilitar CI sin hardware
- Añadir logging estructurado y metadatos de ejecución por experimento
- Generar y mantener READMEs traducidos bajo `i18n/`

## Contribuciones

Se aceptan contribuciones para:
- Mejoras en abstracción de hardware
- Mejor configuración y reproducibilidad
- Ampliación y traducción de documentación
- Comprobaciones de seguridad y guardrails operativos para control de movimiento

Flujo sugerido de contribución:
1. Haz un fork y crea una rama de funcionalidad
2. Realiza cambios enfocados y revisables
3. Valida con tu configuración de hardware
4. Envía un pull request con pasos reproducibles y logs

## Licencia

No hay archivo de licencia en esta instantánea del repositorio.

Suposición: todos los derechos están reservados hasta que se añada explícitamente una licencia del proyecto. Añade un archivo `LICENSE` para definir términos de reutilización.

## Soporte

No se encontró metadata de sponsor/donaciones en esta instantánea. Si quieres incluir enlaces de soporte, añádelos aquí y en los READMEs traducidos.
