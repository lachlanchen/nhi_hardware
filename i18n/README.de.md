[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# NHI Hardwaresteuerung und Ereignisaufnahme

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Schnelle Navigation

| Abschnitt | Zweck |
|---|---|
| [Installation](#installation) | Umgebung und Abhängigkeiten vorbereiten |
| [Verwendung](#verwendung) | Web-Orchestrator und CLI-Workflows ausführen |
| [Konfiguration](#konfiguration) | Serielle, Netzwerk- und Standardwerte anpassen |
| [Beispiele](#beispiele) | Praktische Befehlsbeispiele ausführen |
| [Fehlersuche](#fehlersuche) | Häufige Setup-Probleme beheben |

## 🧭 Projekt im Überblick

| Fokus | Details |
|---|---|
| Ziel | Koordiniert Ereigniserfassung, Bewegungssteuerung und LED-Signalgebung für wiederholbare Laborabläufe |
| Haupteinstieg | `app.py` (Tornado-Webauslöser + asynchrone Sequenz-Orchestrierung) |
| Haupt-Eingaben | EVK5-Ereignis-Stream, FMC4030-Achsensteuerung, Arduino-Seriellbefehle |
| Primäre Ausgaben | `data/axis_1_positions.csv`, optionale Ereignis-CSV-Exporte |
| Plattformen | Windows / Linux (abhängig von SDK- und Hardware-Verfügbarkeit) |

Ein Projekt zur Hardware-Orchestrierung für Event-Kamera-Experimente mit:
- EVK5 Event-Kameracapture (Prophesee Metavision Stack)
- FMC4030-basierter CNC-Bewegungssteuerung
- Arduino-basierter LED-Steuerung per Serial
- Minimaler Web-Trigger-Oberfläche (Tornado)

> Annahme: Hardware-, DLL- und SDK-Umgebungen unterscheiden sich je nach Host und werden aus dem Projektcode abgeleitet; das konkrete Kommandoverhalten kann je nach OS, Treiber-Versionen und Laufzeitverfügbarkeit variieren.

## 🧠 Überblick

Der primäre End-to-End-Workflow ist in `app.py` implementiert:

1. Optionales Verschieben vorhandener `data/`-Ordner in einen Zeitstempel-Ordner (`data_YYYYMMDD_HHMMSS`)
2. Verbinden mit Arduino-LED (standardmäßig `COM4`)
3. Initialisieren des CNC-Controllers über `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. LED- und Y-Achsen-Bewegungssequenz starten
5. Optional EVK5-Ereignisse aufzeichnen (im aktiven Ablauf derzeit deaktiviert, siehe Hinweis unten)
6. Achspositionsprotokolle in `data/axis_1_positions.csv` speichern

### Ablauf-Schnappschuss

| Stufe | Komponente | Ausgabe |
|---|---|---|
| Auslöser | Tornado `/start` | Asynchrone Sequenzausführung |
| Bewegung | FMC4030-Controller | Achsenbewegung + Positionsabfrage |
| Beleuchtung | Arduino Serial (`'1'` / `'0'`) | LED-Zustandssteuerung |
| Erfassung | EVK5 + Metavision | Ereignis-Stream / CSV-Export |
| Persistenz | Lokales Dateisystem | `data/*.csv`, rotierte Ordner |

Das Repository enthält außerdem alternative/Legacy-Kamera-Skripte, Werkzeuge für die Nachbearbeitung von Frames sowie gebündelte Metavision-Python-Beispiele.

## ✨ Funktionen

- Tornado-Web-Endpunkt (`/start`) zum asynchronen Starten einer Bewegungs-/Erfassungssequenz
- EVK5-Aufzeichnung mit aktivierbaren Triggerkanälen (`MAIN`) über Metavision HAL
- CSV-Export von Ereignissen mit Event- und Systemzeitstempeln
- FMC4030-Motorsteuerung über `ctypes` und herstellerspezifische DLL
- Bewegungsposten-Logging als CSV während der Achsenbewegung
- Arduino-LED-Steuerung per Serial (`'1'` / `'0'`)
- Frame-Hilfsskripte (`.npy`-Formenanzeige und `.npy` zu MP4-Konvertierung)
- Gebündelte `python_samples/`-Metavision-Beispiele zum Experimentieren und als Referenz

## 🗂️ Projektstruktur

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

## 🧰 Voraussetzungen

### Hardware

- EVK5-kompatible Event-Kamera sowie Treiber/SDK
- FMC4030-kompatibler Bewegungscontroller erreichbar unter konfigurierter IP/Port-Kombination
- Arduino-Board für LED-Steuerung

### Software

- Python 3.x
- Vendor-/Laufzeitunterstützung für:
  - Prophesee Metavision Python-Module (`metavision_core`, `metavision_hal`, zugehörige SDK-Module)
  - Laden der FMC4030-DLL über Python `ctypes` (`windll`-Verwendung in `cnc/cnc.py` impliziert Windows-Pfad für CNC)
- Python-Bibliotheken, die in Skripten verwendet werden:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Optional/alternativer Stack: `dv`

### Umgebungsvorgaben

| Einstellung | Standard | Ort |
|---|---|---|
| Arduino-Seriell-Port | `COM4` | `app.py`, `led.py` |
| CNC-IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC-Port | `8088` | `cnc/cnc.py` |

Hinweise:
- In der aktuellen Momentaufnahme gibt es keine `requirements.txt` oder `pyproject.toml`.
- Der serielle Standard-Port ist `COM4` in `app.py` und `led.py`.
- Standard-CNC-Netzwerkeinstellungen in `cnc/cnc.py`: IP `192.168.0.30`, Port `8088`.

## 🔧 Installation

1. Repository klonen:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Python-Umgebung erstellen und aktivieren:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Basis-Python-Abhängigkeiten der Kernskripte installieren:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Kamera-SDK-Abhängigkeiten installieren, die in Ihrer Umgebung erforderlich sind:

```bash
# Beispiel-Platzhalter: Prophesee Metavision Python-Pakete installieren
# Folgen Sie den Installationsanweisungen Ihres SDKs/der Distribution für Ihr OS und Kameraversion.
```

## 🧪 Verwendung

### 1) Web-orchestrierte Sequenz (Hauptablauf)

Ausführen im Repository-Root (wichtig für relative Pfade in `app.py`):

```bash
python app.py
```

Dann öffnen:

```text
http://localhost:8888
```

Klicken Sie auf **Start Sequence**, um die Bewegungs-/LED-Workflow auszulösen.

Optionales Argument, das aktuell von app geparst wird:

```bash
python app.py --record_events True
```

Wichtiger Hinweis zum Verhalten: `start_sequence()` setzt `record_events = False` später während der Ausführung wieder zurück, daher kann die EVK5-Aufzeichnung deaktiviert bleiben, sofern der Code angepasst wird.

### 2) EVK5-Ereignisaufnahme (direkte CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Argumente:
- `-i, --input`: Eingabequelle/Pfad für EVK5-Gerät oder Aufzeichnung
- `-d, --duration`: Aufzeichnungsdauer in Sekunden
- `-z, --timezone`: Zeitzonenbezeichnung für Zeitstempel-Formatierung
- `-o, --output`: Basisname der Ausgabe (gespeichert unter `data/<name>.csv`)

### 3) CNC-Steuerung (direkte CLI)

Führen Sie dies innerhalb von `cnc/` aus, damit der relative DLL-Pfad in `cnc.py` korrekt aufgelöst wird:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Weitere verfügbare Optionen:

```bash
# Aktuelle Position als Soft-Origin setzen
python cnc.py --set-origin

# Zu absoluten Koordinaten x,y,z bewegen
python cnc.py --move 0,30,0 --speed 20

# Ursprung nach Bewegung zu den angegebenen Koordinaten setzen
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED-Testskript

```bash
python led.py
```

Port im Code aktualisieren, falls nicht `COM4` verwendet wird.

### 5) Hilfsprogramme

```bash
# Frame-Array-Form aus einem Ordner ausgeben
python npy_shape.py data_20250101_120000

# Frame-Array-Form für alle data_*-Ordner im aktuellen Verzeichnis ausgeben
python npy_shape.py -a
```

`npy2video.py` stellt `npy_to_video(npy_file_path, output_video_path, fps=5)` bereit und kann importiert oder für lokale Pfade angepasst werden.

## ⚙️ Konfiguration

- `motor_system.ini` und `cnc/motor_system.ini`:
  - Persistieren Software-Origin-Koordinaten (`ORIGIN`-Sektion für X/Y/Z)
- `app.py`:
  - Arduino-Seriell-Port: `ArduinoLED(port='COM4')`
  - CNC-DLL-Pfad: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (DV-Pfad):
  - Standard `DV_PORT`: `7777`
  - Standard `DV_PORT_FRAME`: `7778`

## 📸 Beispiele

### Beispiel A: Vollständige Sequenz über Web starten

```bash
python app.py
# öffnen Sie http://localhost:8888 und drücken Sie Start Sequence
```

Erwartete Ausgaben umfassen:
- `data/axis_1_positions.csv` (CNC-Positionsverlauf)
- Optional `data/<events>.csv`, falls die Ereignisaufnahme im aktiven Pfad aktiv ist

### Beispiel B: EVK5-Ereignisse 60 Sekunden aufzeichnen

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Erwartete Ausgabe:
- `data/run_001_events.csv`

### Beispiel C: Y-Achse per CLI vor- und zurückfahren

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 Entwicklungsnotizen

- Das aktuelle Repository ist erkennbar ein Forschungs-/Prototyping-Arbeitsbereich mit gemischtem aktivem und archiviertem Skriptbestand.
- Große generierte Artefakte (`event_output.csv`, `data-0503/`) sind versioniert; erwägen Sie eine Datenaufbewahrungsstrategie und `.gitignore`-Anpassungen, falls dieses Repository verteilt werden soll.
- `python_samples/` enthält nützliche Kamera-SDK-Beispiele, kann aber Abhängigkeiten enthalten, die für die Kernorchestrierung nicht erforderlich sind.
- Potenzieller Verbesserungsbedarf in der Codequalität:
  - `argparse`-Boolean-Verarbeitung in `app.py` kann verbessert werden (`type=bool` ist in CLI-Parsing oft irreführend).
  - Die Behandlung des Event-Recordings-Flags in `start_sequence()` überschreibt aktuell den anfänglichen CLI-Wert.

## 🛠️ Fehlersuche

- `ImportError: metavision_*`-Module fehlen:
  - Installieren/Sie konfigurieren Ihre Metavision-SDK-Python-Umgebung.
- `ImportError: No module named dv`:
  - Installieren Sie das DV-Python-Paket, falls Sie den Pfad `event_sensor.py` verwenden.
- CNC-DLL kann nicht geladen werden:
  - Prüfen Sie die OS-Kompatibilität und ob `FMC4030-Dll.dll` am erwarteten relativen Pfad vorhanden ist.
  - Führen Sie `cnc.py` aus dem Verzeichnis `cnc/` aus oder passen Sie `dll_path` an.
- LED-Serienverbindungsfehler:
  - Prüfen Sie die Arduino-Port-Zuordnung (`COM4` vs. tatsächlicher Port).
  - Stellen Sie sicher, dass kein anderer Prozess die serielle Schnittstelle blockiert.
- Keine Dateien in `data/` nach Web-Start:
  - Prüfen Sie Schreibberechtigungen und ob der Ereignis-Aufzeichnungspfad aktiviert ist.

## 🗺️ Roadmap

- Hinzufügen eines Dependency-Manifests (`requirements.txt` oder `pyproject.toml`) samt Versions-Pinning
- Auslagern der Laufzeitkonfiguration (Ports, IP, DLL-Pfad, Geschwindigkeitsprofile) in eine einheitliche Konfigurationsdatei
- Vereinheitlichung von Kamera-Backends (EVK5 und DV) hinter einer Schnittstelle mit klarer Moduswahl
- Hinzufügen von Tests/Mocks für Bewegungs- und Sensorschnittstellen, um CI ohne Hardware zu ermöglichen
- Strukturierte Logs und Laufzeit-Metadaten pro Experiment bereitstellen
- Generierte und gepflegte Übersetzungen der READMEs unter `i18n/` beibehalten

## 🤝 Mitwirken

Beiträge sind willkommen für:
- Verbesserungen bei der Hardware-Abstraktion
- Bessere Konfiguration und Reproduzierbarkeit
- Dokumentationsausbau und Übersetzungen
- Sicherheitsprüfungen und Betriebsgrenzen für Bewegungssteuerung

Empfohlener Beitragspfad:
1. Fork und Feature-Branch erstellen
2. Fokussierte, reviewbare Änderungen vornehmen
3. Gegen Ihre Hardware-Umgebung validieren
4. Einen Pull Request mit reproduzierbaren Schritten und Logs einreichen

## Lizenz

Keine Lizenzdatei ist in dieser Repository-Momentaufnahme vorhanden.

Annahme: Alle Rechte bleiben vorbehalten, bis eine Projektlizenz explizit hinzugefügt wird. Fügen Sie eine `LICENSE`-Datei hinzu, um die Nutzungsbedingungen festzulegen.


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
