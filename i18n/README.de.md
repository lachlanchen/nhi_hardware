[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# NHI Hardware-Steuerung und Ereigniserfassung

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

Ein Hardware-Orchestrierungsprojekt für Event-Kamera-Experimente, das Folgendes kombiniert:
- EVK5-Event-Kamera-Erfassung (Prophesee Metavision-Stack)
- CNC-Bewegungssteuerung auf Basis von FMC4030
- Serielle Arduino-LED-Steuerung
- Minimale Web-Trigger-UI (Tornado)

> Diese README ist der erste vollständige Entwurf für diesen Repository-Snapshot.
> Annahme: In diesem Checkout gab es zuvor keine `README.md` im Root-Verzeichnis, daher wurde dieses Dokument aus Quellcode und Pipeline-Analyseartefakten erstellt.

## Überblick

Der primäre End-to-End-Workflow ist in `app.py` implementiert:

1. Optional vorhandenes `data/` in einen Zeitstempel-Ordner verschieben (`data_YYYYMMDD_HHMMSS`)
2. Verbindung zur Arduino-LED herstellen (Standard `COM4`)
3. CNC-Controller über `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` initialisieren
4. LED- und Y-Achsen-Bewegungssequenz ausführen
5. Optional EVK5-Ereignisse aufzeichnen (im aktiven Ablauf derzeit deaktiviert; siehe Hinweise unten)
6. Achspositionsprotokolle in `data/axis_1_positions.csv` speichern

### Workflow-Snapshot

| Phase | Komponente | Ausgabe |
|---|---|---|
| Trigger | Tornado `/start` | Asynchrone Sequenzausführung |
| Bewegung | FMC4030-Controller | Achsbewegung + Positionsabfrage |
| Beleuchtung | Arduino Serial (`'1'` / `'0'`) | LED-Zustandssteuerung |
| Sensorik | EVK5 + Metavision | Event-Stream / CSV-Export |
| Persistenz | Lokales Dateisystem | `data/*.csv`, rotierte Ordner |

Das Repository enthält außerdem alternative/ältere Kamera-Skripte, Utilities für Frame-Nachbearbeitung und gebündelte Metavision-Python-Beispiele.

## Funktionen

- Tornado-Web-Endpunkt (`/start`), um eine Bewegungs-/Erfassungssequenz asynchron zu starten
- EVK5-Event-Aufzeichnung mit Trigger-Kanal-Aktivierung (`MAIN`) über Metavision HAL
- CSV-Export von Events mit Event- und Systemzeitstempeln
- FMC4030-Motorsteuerungs-Wrapper mit `ctypes` und Hersteller-DLL
- Bewegungspositions-Logging in CSV während der Achsbewegung
- Serielle Arduino-LED-Steuerung (`'1'`/`'0'`-Befehle)
- Frame-Utility-Skripte (`.npy`-Shape-Inspektion und `.npy`-zu-MP4-Konvertierung)
- Gebündelte `python_samples/`-Metavision-Beispiele für Experimente und Referenz

## Projektstruktur

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

## Voraussetzungen

### Hardware

- EVK5-kompatible Event-Kamera und Treiber/SDK
- FMC4030-kompatibler Motion-Controller, erreichbar unter konfigurierte IP/Port
- Arduino-Board für LED-Steuerung

### Software

- Python 3.x
- Hersteller-/Runtime-Unterstützung für:
  - Prophesee Metavision Python-Module (`metavision_core`, `metavision_hal`, zugehörige SDK-Module)
  - Laden der FMC4030-DLL via Python `ctypes` (`windll`-Nutzung in `cnc/cnc.py` impliziert Windows für den CNC-Pfad)
- In den Skripten verwendete Python-Bibliotheken:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Optionaler/alternativer Stack: `dv`

### Umgebungs-Standards

| Einstellung | Standard | Ort |
|---|---|---|
| Arduino-Serial-Port | `COM4` | `app.py`, `led.py` |
| CNC-IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC-Port | `8088` | `cnc/cnc.py` |

Hinweise:
- Im aktuellen Snapshot gibt es keine `requirements.txt` oder `pyproject.toml`.
- Der serielle Port ist in `app.py` und `led.py` standardmäßig `COM4`.
- Standardmäßige CNC-Netzwerkeinstellungen in `cnc/cnc.py`: IP `192.168.0.30`, Port `8088`.

## Installation

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

3. Basis-Python-Abhängigkeiten installieren, die von den Kernskripten genutzt werden:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Kamera-SDK-Abhängigkeiten installieren, die in deiner Umgebung erforderlich sind:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## Verwendung

### 1) Web-orchestrierte Sequenz (Primär)

Vom Repository-Root ausführen (wichtig für relative Pfade in `app.py`):

```bash
python app.py
```

Dann öffnen:

```text
http://localhost:8888
```

Klicke auf **Start Sequence**, um den Bewegungs-/LED-Workflow auszulösen.

Optionales Argument, das derzeit von der App geparst wird:

```bash
python app.py --record_events True
```

Wichtiger Verhaltenhinweis: `start_sequence()` setzt `record_events = False` später in der Ausführung zurück, sodass EVK5-Aufzeichnung deaktiviert bleiben kann, sofern der Code nicht angepasst wird.

### 2) EVK5-Event-Aufzeichnung (Direkte CLI)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Argumente:
- `-i, --input`: Eingabequelle/-pfad für EVK5-Gerät oder Aufzeichnung
- `-d, --duration`: Aufzeichnungsdauer in Sekunden
- `-z, --timezone`: Zeitzonenbezeichnung für Zeitstempelformatierung
- `-o, --output`: Ausgabe-Basisname (gespeichert unter `data/<name>.csv`)

### 3) CNC-Steuerung (Direkte CLI)

Von `cnc/` ausführen, damit der relative DLL-Pfad in `cnc.py` korrekt aufgelöst wird:

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Weitere verfügbare Optionen:

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED-Testskript

```bash
python led.py
```

Aktualisiere den Port im Code, wenn du nicht `COM4` verwendest.

### 5) Utilities

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` stellt `npy_to_video(npy_file_path, output_video_path, fps=5)` bereit und kann importiert oder für deine lokalen Pfade angepasst werden.

## Konfiguration

- `motor_system.ini` und `cnc/motor_system.ini`:
  - Persistieren Software-Ursprungskoordinaten (`ORIGIN`-Abschnitt für X/Y/Z)
- `app.py`:
  - Arduino-Serial-Port: `ArduinoLED(port='COM4')`
  - CNC-DLL-Pfad: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (DV-Pfad):
  - `DV_PORT` Standard `7777`
  - `DV_PORT_FRAME` Standard `7778`

## Beispiele

### Beispiel A: Vollständige Sequenz über Web starten

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Erwartete Ausgaben umfassen:
- `data/axis_1_positions.csv` (CNC-Positionsverlauf)
- Optional `data/<events>.csv`, falls Event-Aufzeichnung im aktiven Pfad aktiviert ist

### Beispiel B: EVK5-Events für 60 Sekunden aufzeichnen

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Erwartete Ausgabe:
- `data/run_001_events.csv`

### Beispiel C: Y-Achse per CLI hin und zurück bewegen

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## Entwicklungshinweise

- Das aktuelle Repository scheint ein Forschungs-/Prototyping-Workspace mit gemischten aktiven und archivierten Skripten zu sein.
- Große erzeugte Artefakte (`event_output.csv`, `data-0503/`) sind eingecheckt; erwäge eine Data-Retention-Strategie und `.gitignore`-Updates, falls dieses Repository verteilt werden soll.
- `python_samples/` enthält nützliche Kamera-SDK-Beispiele, kann jedoch Abhängigkeiten enthalten, die für die Kern-Orchestrierung nicht erforderlich sind.
- Potenzielle Verbesserung der Codequalität:
  - `argparse`-Boolean-Handling in `app.py` kann verbessert werden (`type=bool` ist bei CLI-Parsing oft irreführend).
  - Das Event-Recording-Flag-Handling in `start_sequence()` überschreibt aktuell den anfänglichen CLI-Wert.

## Fehlerbehebung

- `ImportError: metavision_*`-Module fehlen:
  - Installiere/konfiguriere deine Metavision-SDK-Python-Umgebung.
- `ImportError: No module named dv`:
  - Installiere das DV-Python-Paket, wenn du den `event_sensor.py`-Pfad nutzt.
- Fehler beim Laden der CNC-DLL:
  - Prüfe OS-Kompatibilität und ob `FMC4030-Dll.dll` unter dem erwarteten relativen Pfad verfügbar ist.
  - Führe `cnc.py` aus dem Verzeichnis `cnc/` aus oder passe `dll_path` an.
- LED-Serienverbindungsfehler:
  - Prüfe die Arduino-Portzuweisung (`COM4` vs. tatsächlicher Port).
  - Stelle sicher, dass kein anderer Prozess das serielle Gerät belegt.
- Keine Dateien in `data/` nach Web-Lauf:
  - Überprüfe Schreibberechtigungen und ob der Event-Aufzeichnungspfad aktiviert ist.

## Roadmap

- Abhängigkeitsmanifest hinzufügen (`requirements.txt` oder `pyproject.toml`) und Versionen fixieren
- Laufzeitkonfiguration (Ports, IP, DLL-Pfad, Geschwindigkeitsprofile) in eine einheitliche Konfigurationsdatei auslagern
- Kamera-Backends (EVK5 und DV) hinter einer Schnittstelle normalisieren, mit klarer Modusauswahl
- Tests/Mocks für Motion- und Sensor-Schnittstellen hinzufügen, um CI ohne Hardware zu ermöglichen
- Strukturiertes Logging und Run-Metadaten pro Experiment ergänzen
- Übersetzte README-Dateien unter `i18n/` erzeugen und pflegen

## Mitwirken

Beiträge sind willkommen für:
- Verbesserungen der Hardware-Abstraktion
- Bessere Konfiguration und Reproduzierbarkeit
- Ausbau und Übersetzung der Dokumentation
- Sicherheitsprüfungen und operative Schutzmechanismen für Motion-Control

Empfohlener Beitragsablauf:
1. Fork erstellen und Feature-Branch anlegen
2. Fokussierte, gut reviewbare Änderungen umsetzen
3. Gegen die eigene Hardware-Konfiguration validieren
4. Pull Request mit reproduzierbaren Schritten und Logs einreichen

## Lizenz

In diesem Repository-Snapshot ist keine Lizenzdatei vorhanden.

Annahme: Alle Rechte vorbehalten, bis ausdrücklich eine Projektlizenz hinzugefügt wird. Füge eine `LICENSE`-Datei hinzu, um Wiederverwendungsbedingungen festzulegen.

## Support

In diesem Snapshot wurden keine Sponsor-/Spenden-Metadaten gefunden. Wenn du Support-Links einbinden möchtest, füge sie hier und in den übersetzten READMEs hinzu.
