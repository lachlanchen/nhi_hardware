[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# Contrôle matériel NHI et capture d'événements

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 Navigation rapide

| Cette section | Rôle |
|---|---|
| [Installation](#installation) | Préparer l'environnement et les dépendances |
| [Utilisation](#usage) | Exécuter l'orchestrateur web et les workflows CLI |
| [Configuration](#configuration) | Ajuster la liaison série, le réseau et les valeurs par défaut |
| [Exemples](#exemples) | Exécuter des exemples pratiques |
| [Dépannage](#depannage) | Résoudre les problèmes de configuration courants |

## 🧭 Vue d'ensemble du projet

| Axe | Détails |
|---|---|
| Mission | Coordonner la capture d'événements, le contrôle de mouvement et la signalisation LED pour des flux de travail de laboratoire reproductibles |
| Entrée principale | `app.py` (déclenchement web Tornado + orchestration de séquence asynchrone) |
| Entrées clés | Flux d'événements EVK5, contrôle d'axe FMC4030, commandes LED via Arduino |
| Sorties principales | `data/axis_1_positions.csv`, exports CSV d'événements (optionnels) |
| Plateformes | Windows / Linux (selon la disponibilité du SDK et du matériel) |

Projet d'orchestration matérielle pour les expériences caméra-événement combinant :
- Capture par caméra événementielle EVK5 (stack Prophesee Metavision)
- Contrôle de mouvement CNC basé sur FMC4030
- Commande LED série Arduino
- Interface de déclenchement web minimale (Tornado)

> Hypothèse : les environnements matériel, DLL et SDK varient selon l'hôte et sont déduits du code ; le comportement exact des commandes peut différer selon l'OS, les versions de pilotes et la disponibilité runtime.

## 🧠 Vue d'ensemble

Le flux de bout en bout principal est implémenté dans `app.py` :

1. Déplace éventuellement `data/` existant vers un dossier horodaté (`data_YYYYMMDD_HHMMSS`)
2. Connecte la LED Arduino (défaut `COM4`)
3. Initialise le contrôleur CNC via `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Lance la séquence d'éclairage et de déplacement de l'axe Y
5. Enregistre éventuellement les événements EVK5 (actuellement désactivé dans le flux actif ; voir note ci-dessous)
6. Enregistre les journaux de position d'axe dans `data/axis_1_positions.csv`

### Aperçu du flux de travail

| Étape | Composant | Sortie |
|---|---|---|
| Déclenchement | Tornado `/start` | Exécution asynchrone de la séquence |
| Mouvement | Contrôleur FMC4030 | Déplacement d'axe + interrogation de position |
| Éclairage | Série Arduino (`'1'` / `'0'`) | Contrôle d'état de la LED |
| Acquisition | EVK5 + Metavision | Flux d'événements / export CSV |
| Persistance | Système de fichiers local | `data/*.csv`, dossiers rotatifs |

Le dépôt contient également des scripts de caméra alternatifs/hérités, des utilitaires de post-traitement d'image et des exemples Metavision Python fournis.

## ✨ Fonctionnalités

- Point de terminaison web Tornado (`/start`) pour lancer une séquence mouvement/capture de façon asynchrone
- Enregistrement EVK5 avec activation de canal de déclenchement (`MAIN`) via Metavision HAL
- Export CSV des événements avec horodatage événement et système
- Contrôle moteur FMC4030 via `ctypes` et DLL du fournisseur
- Journalisation position/mouvement en CSV pendant le déplacement d'axe
- Contrôle LED série Arduino (`'1'`/`'0'`)
- Utilitaires de trames (`inspection de forme .npy` et conversion `.npy` vers MP4)
- Exemples `python_samples/` Metavision fournis pour l'expérimentation et la référence

## 🗂️ Structure du projet

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
├── motor_system.ini                          # Soft-origin config
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

<a id="prerequis"></a>
## 🧰 Prérequis

### Matériel

- Caméra à événements compatible EVK5 et ses pilotes/SDK
- Contrôleur de mouvement compatible FMC4030 accessible via IP/port configurés
- Carte Arduino pour le contrôle LED

### Logiciel

- Python 3.x
- Support éditeur/fournisseur pour :
  - Modules Python Prophesee Metavision (`metavision_core`, `metavision_hal`, modules SDK associés)
  - Chargement de la DLL FMC4030 via Python `ctypes` (`windll` dans `cnc/cnc.py` implique Windows pour le chemin CNC)
- Bibliothèques Python utilisées par les scripts :
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Stack alternative/optionnelle : `dv`

### Paramètres d'environnement

| Paramètre | Défaut | Emplacement |
|---|---|---|
| Port série Arduino | `COM4` | `app.py`, `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| Port CNC | `8088` | `cnc/cnc.py` |

Remarques :
- Aucun fichier `requirements.txt` ou `pyproject.toml` n'est présent dans l'instantané actuel.
- Le port série par défaut est `COM4` dans `app.py` et `led.py`.
- Réglages réseau CNC par défaut dans `cnc/cnc.py` : IP `192.168.0.30`, port `8088`.

<a id="installation"></a>
## 🔧 Installation

1. Cloner le dépôt :

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Créer et activer un environnement Python :

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. Installer les dépendances Python de base utilisées par les scripts principaux :

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. Installer les dépendances SDK caméra requises par votre environnement :

```bash
# Exemple de placeholder : installer les paquets Prophesee Metavision Python
# Suivez les instructions de votre distribution SDK/OS et de la version de votre caméra.
```

<a id="usage"></a>
## 🧪 Utilisation

### 1) Séquence orchestrée via web (principale)

Exécuter depuis la racine du dépôt (important pour les chemins relatifs dans `app.py`) :

```bash
python app.py
```

Puis ouvrir :

```text
http://localhost:8888
```

Cliquer sur **Start Sequence** pour déclencher le flux mouvement/LED.

Argument optionnel actuellement analysé par l'application :

```bash
python app.py --record_events True
```

Comportement important : `start_sequence()` réinitialise actuellement `record_events = False` plus loin dans l'exécution, donc l'enregistrement EVK5 peut rester désactivé sauf ajustement du code.

### 2) Enregistrement des événements EVK5 (CLI direct)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Arguments :
- `-i, --input` : Source d'entrée/chemin EVK5 du périphérique ou de l'enregistrement
- `-d, --duration` : Durée d'enregistrement en secondes
- `-z, --timezone` : Libellé du fuseau horaire pour le formatage des timestamps
- `-o, --output` : Nom de base de sortie (sauvegardé dans `data/<nom>.csv`)

### 3) Contrôle CNC (CLI direct)

Exécuter depuis `cnc/` pour que le chemin DLL relatif dans `cnc.py` soit résolu correctement :

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Autres options disponibles :

```bash
# Définir la position courante comme origine logicielle
python cnc.py --set-origin

# Déplacer aux coordonnées absolues x,y,z
python cnc.py --move 0,30,0 --speed 20

# Définir l'origine après déplacement vers des coordonnées fournies
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script de test LED

```bash
python led.py
```

Modifier le port dans le code si vous n'utilisez pas `COM4`.

### 5) Utilitaires

```bash
# Afficher la forme des tableaux de frames d'un dossier
python npy_shape.py data_20250101_120000

# Afficher la forme des tableaux de frames pour tous les dossiers data_* dans cwd
python npy_shape.py -a
```

`npy2video.py` fournit `npy_to_video(npy_file_path, output_video_path, fps=5)` et peut être importé ou édité pour vos chemins locaux.

<a id="configuration"></a>
## ⚙️ Configuration

- `motor_system.ini` et `cnc/motor_system.ini` :
  - Persistente des coordonnées de l'origine logicielle (`section ORIGIN` pour X/Y/Z)
- `app.py` :
  - Port série Arduino : `ArduinoLED(port='COM4')`
  - Chemin DLL CNC : `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (chemin DV) :
  - `DV_PORT` par défaut `7777`
  - `DV_PORT_FRAME` par défaut `7778`

<a id="exemples"></a>
## 📸 Exemples

### Exemple A : Démarrer une séquence complète via web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Résultats attendus :
- `data/axis_1_positions.csv` (trace de position CNC)
- `data/<events>.csv` optionnel si l'enregistrement d'événements est activé dans le flux actif

### Exemple B : Enregistrer les événements EVK5 pendant 60 secondes

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Sortie attendue :
- `data/run_001_events.csv`

### Exemple C : Déplacer l'axe Y va-et-vient depuis la CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

<a id="depannage"></a>
## 🛠️ Dépannage

- `ImportError: metavision_*` modules manquants :
  - Installez et configurez votre environnement SDK Metavision Python.
- `ImportError: No module named dv` :
  - Installez le paquet DV Python si vous utilisez le chemin `event_sensor.py`.
- Échecs de chargement de la DLL CNC :
  - Vérifiez la compatibilité de l'OS et la présence de `FMC4030-Dll.dll` au chemin relatif attendu.
  - Exécutez `cnc.py` depuis le dossier `cnc/` ou ajustez `dll_path`.
- Erreurs de connexion série Arduino :
  - Vérifiez l'affectation du port Arduino (`COM4` vs port réel).
  - Assurez-vous qu'aucun autre processus ne monopolise le périphérique série.
- Aucun fichier dans `data/` après l'exécution web :
  - Vérifiez les droits d'écriture et si le chemin d'enregistrement des événements est activé.

## 🗺️ Feuille de route

- Ajouter un manifeste de dépendances (`requirements.txt` ou `pyproject.toml`) et fixer les versions
- Externaliser la configuration runtime (ports, IP, chemin DLL, profils de vitesse) dans un fichier de config unifié
- Normaliser les backends caméra (EVK5 et DV) derrière une interface unique avec sélection de mode explicite
- Ajouter des tests/mocks pour les interfaces motion et capteurs afin de permettre de la CI sans matériel
- Ajouter une journalisation structurée et des métadonnées de run par expérience
- Générer et maintenir des README traduits dans `i18n/`

## 🤝 Contribuer

Les contributions sont bienvenues pour :
- Améliorer l'abstraction matérielle
- Mieux gérer la configuration et la reproductibilité
- Étendre la documentation et les traductions
- Ajouter des contrôles de sécurité et garde-fous opérationnels pour le pilotage de mouvement

Flux de contribution recommandé :
1. Forker et créer une branche de fonctionnalité
2. Réaliser des changements ciblés et relisibles
3. Valider avec votre configuration matérielle
4. Soumettre une pull request avec des étapes reproductibles et des journaux

## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |

## License

Aucun fichier de licence n'est présent dans l'instantané actuel du dépôt.

Hypothèse : tous les droits sont réservés tant qu'une licence de projet n'est pas explicitement ajoutée. Ajoutez un fichier `LICENSE` pour définir les conditions de réutilisation.
