[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# Contrôle matériel NHI et capture d'événements

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

Un projet d'orchestration matérielle pour des expériences avec caméra événementielle, combinant :
- Capture EVK5 (stack Prophesee Metavision)
- Contrôle de mouvement CNC basé sur FMC4030
- Contrôle de LED Arduino via série
- Interface web minimale de déclenchement (Tornado)

> Ce README est la première ébauche complète pour cet instantané du dépôt.
> Hypothèse : aucun `README.md` racine préexistant n'était présent dans ce checkout, donc ce document est construit à partir du code source et des artefacts d'analyse du pipeline.

## Vue d'ensemble

Le flux de travail principal de bout en bout est implémenté dans `app.py` :

1. Fait éventuellement pivoter le dossier `data/` existant vers un dossier horodaté (`data_YYYYMMDD_HHMMSS`)
2. Se connecte à la LED Arduino (par défaut `COM4`)
3. Initialise le contrôleur CNC via `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
4. Exécute la séquence LED et mouvement sur l'axe Y
5. Enregistre éventuellement les événements EVK5 (actuellement désactivé dans le flux actif ; voir les notes ci-dessous)
6. Enregistre les journaux de position d'axe dans `data/axis_1_positions.csv`

### Aperçu du flux

| Étape | Composant | Sortie |
|---|---|---|
| Déclenchement | Tornado `/start` | Exécution asynchrone de la séquence |
| Mouvement | Contrôleur FMC4030 | Mouvement d'axe + interrogation de position |
| Éclairage | Série Arduino (`'1'` / `'0'`) | Contrôle d'état de la LED |
| Mesure | EVK5 + Metavision | Flux d'événements / export CSV |
| Persistance | Système de fichiers local | `data/*.csv`, dossiers pivotés |

Le dépôt contient aussi des scripts caméra alternatifs/hérités, des utilitaires de post-traitement de frames, et des exemples Python Metavision fournis.

## Fonctionnalités

- Point d'entrée web Tornado (`/start`) pour lancer une séquence mouvement/capture de manière asynchrone
- Enregistrement d'événements EVK5 avec activation du canal de déclenchement (`MAIN`) via Metavision HAL
- Export CSV des événements avec horodatages événement et système
- Wrapper de contrôle moteur FMC4030 utilisant `ctypes` et la DLL du fournisseur
- Journalisation des positions de mouvement en CSV pendant le déplacement de l'axe
- Contrôle série de la LED Arduino (commandes `'1'`/`'0'`)
- Scripts utilitaires de frames (inspection de forme `.npy` et conversion `.npy` vers MP4)
- Exemples Metavision fournis dans `python_samples/` pour expérimentation et référence

## Structure du projet

```text
.
├── app.py                                   # Orchestrateur web principal
├── event_sensor_evk5.py                     # Enregistreur EVK5 (Metavision)
├── event_sensor.py                          # Enregistreur alternatif (package dv)
├── evk5_test.py                             # Enregistreur de test EVK5 minimal
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Contrôle série LED Arduino
├── npy2video.py                             # Conversion pile de frames NPY -> MP4
├── npy_shape.py                             # Affiche les dimensions des tableaux de frames
├── motor_system.ini                         # Configuration de l'origine logicielle
├── cnc/
│   ├── cnc.py                               # Contrôle FMC4030 + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # Anciens scripts de contrôleur
├── led/
│   └── led.ino                              # Sketch firmware Arduino
├── templates/
│   └── index.html                           # UI avec bouton de démarrage
├── python_samples/                          # Programmes d'exemple Metavision
├── data-0503/                               # Jeux de données d'expériences historiques
├── i18n/                                    # Réservé aux README traduits
└── .auto-readme-work/20260228_231403/      # Artefacts du pipeline README
```

## Prérequis

### Matériel

- Caméra événementielle compatible EVK5 et drivers/SDK
- Contrôleur de mouvement compatible FMC4030 accessible à l'IP/port configurés
- Carte Arduino pour le contrôle LED

### Logiciel

- Python 3.x
- Support fournisseur/runtime pour :
  - Modules Python Prophesee Metavision (`metavision_core`, `metavision_hal`, modules SDK associés)
  - Chargement de DLL FMC4030 via `ctypes` Python (l'utilisation de `windll` dans `cnc/cnc.py` implique Windows pour le chemin CNC)
- Bibliothèques Python utilisées dans les scripts :
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - Stack optionnelle/alternative : `dv`

### Valeurs d'environnement par défaut

| Paramètre | Valeur par défaut | Emplacement |
|---|---|---|
| Port série Arduino | `COM4` | `app.py`, `led.py` |
| IP CNC | `192.168.0.30` | `cnc/cnc.py` |
| Port CNC | `8088` | `cnc/cnc.py` |

Notes :
- Il n'y a pas de `requirements.txt` ni de `pyproject.toml` dans l'instantané actuel.
- Le port série est défini par défaut à `COM4` dans `app.py` et `led.py`.
- Paramètres réseau CNC par défaut dans `cnc/cnc.py` : IP `192.168.0.30`, port `8088`.

## Installation

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

4. Installer les dépendances SDK caméra requises dans votre environnement :

```bash
# Example placeholder: install Prophesee Metavision Python packages
# Follow your SDK/distribution instructions for your OS and camera version.
```

## Utilisation

### 1) Séquence orchestrée par le web (principale)

Lancer depuis la racine du dépôt (important pour les chemins relatifs dans `app.py`) :

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

Note de comportement importante : `start_sequence()` réinitialise actuellement `record_events = False` plus tard dans l'exécution, donc l'enregistrement EVK5 peut rester désactivé tant que le code n'est pas ajusté.

### 2) Enregistrement d'événements EVK5 (CLI direct)

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

Arguments :
- `-i, --input` : source/chemin d'entrée pour l'appareil EVK5 ou un enregistrement
- `-d, --duration` : durée d'enregistrement en secondes
- `-z, --timezone` : libellé de fuseau horaire pour le formatage des horodatages
- `-o, --output` : nom de base de sortie (enregistré sous `data/<name>.csv`)

### 3) Contrôle CNC (CLI direct)

Exécuter depuis `cnc/` pour que le chemin DLL relatif dans `cnc.py` soit résolu correctement :

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

Autres options disponibles :

```bash
# Set current position as soft origin
python cnc.py --set-origin

# Move to absolute coordinates x,y,z
python cnc.py --move 0,30,0 --speed 20

# Set origin after moving to provided coordinates
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) Script de test LED

```bash
python led.py
```

Mettez à jour le port dans le code si vous n'utilisez pas `COM4`.

### 5) Utilitaires

```bash
# Print frame-array shape from one folder
python npy_shape.py data_20250101_120000

# Print frame-array shape for all data_* folders in cwd
python npy_shape.py -a
```

`npy2video.py` fournit `npy_to_video(npy_file_path, output_video_path, fps=5)` et peut être importé ou modifié pour vos chemins locaux.

## Configuration

- `motor_system.ini` et `cnc/motor_system.ini` :
  - Conservent les coordonnées d'origine logicielle (section `ORIGIN` pour X/Y/Z)
- `app.py` :
  - Port série Arduino : `ArduinoLED(port='COM4')`
  - Chemin DLL CNC : `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py` (chemin DV) :
  - `DV_PORT` par défaut `7777`
  - `DV_PORT_FRAME` par défaut `7778`

## Exemples

### Exemple A : démarrer la séquence complète via le web

```bash
python app.py
# visit http://localhost:8888 and press Start Sequence
```

Les sorties attendues incluent :
- `data/axis_1_positions.csv` (trace de position CNC)
- `data/<events>.csv` optionnel si l'enregistrement d'événements est activé dans le chemin actif

### Exemple B : enregistrer des événements EVK5 pendant 60 secondes

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

Sortie attendue :
- `data/run_001_events.csv`

### Exemple C : déplacer l'axe Y en aller-retour depuis la CLI

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## Notes de développement

- Le dépôt actuel semble être un espace de recherche/prototypage avec des scripts actifs et archivés mélangés.
- De gros artefacts générés (`event_output.csv`, `data-0503/`) sont versionnés ; envisagez une stratégie de rétention des données et des mises à jour de `.gitignore` si ce dépôt doit être distribué.
- `python_samples/` contient des exemples utiles du SDK caméra, mais peut inclure des dépendances non requises pour l'orchestration principale.
- Amélioration potentielle de la qualité du code :
  - La gestion booléenne `argparse` dans `app.py` peut être améliorée (`type=bool` est souvent trompeur en parsing CLI).
  - La gestion du drapeau d'enregistrement d'événements dans `start_sequence()` remplace actuellement la valeur CLI initiale.

## Dépannage

- `ImportError: metavision_*` modules missing:
  - Installez/configurez votre environnement Python Metavision SDK.
- `ImportError: No module named dv`:
  - Installez le package Python DV si vous utilisez le chemin `event_sensor.py`.
- Échecs de chargement de DLL CNC :
  - Confirmez la compatibilité OS et que `FMC4030-Dll.dll` est disponible au chemin relatif attendu.
  - Exécutez `cnc.py` depuis le répertoire `cnc/` ou ajustez `dll_path`.
- Erreurs de connexion série LED :
  - Vérifiez l'affectation du port Arduino (`COM4` vs port réel).
  - Assurez-vous qu'aucun autre processus ne monopolise le périphérique série.
- Aucun fichier dans `data/` après exécution web :
  - Vérifiez les permissions d'écriture et si le chemin d'enregistrement d'événements est activé.

## Feuille de route

- Ajouter un manifeste de dépendances (`requirements.txt` ou `pyproject.toml`) et des versions figées
- Externaliser la configuration d'exécution (ports, IP, chemin DLL, profils de vitesse) vers un fichier de configuration unifié
- Normaliser les backends caméra (EVK5 et DV) derrière une interface unique avec sélection de mode claire
- Ajouter des tests/mocks pour les interfaces de mouvement et de capteurs afin d'activer la CI sans matériel
- Ajouter des logs structurés et des métadonnées d'exécution par expérience
- Générer et maintenir les README traduits sous `i18n/`

## Contribution

Les contributions sont les bienvenues pour :
- Améliorations de l'abstraction matérielle
- Meilleure configuration et reproductibilité
- Extension et traduction de la documentation
- Vérifications de sécurité et garde-fous opérationnels pour le contrôle de mouvement

Flux de contribution suggéré :
1. Forker et créer une branche de fonctionnalité
2. Apporter des modifications ciblées et faciles à relire
3. Valider avec votre configuration matérielle
4. Soumettre une pull request avec des étapes reproductibles et des logs

## Licence

Aucun fichier de licence n'est présent dans cet instantané du dépôt.

Hypothèse : tous droits réservés tant qu'une licence projet n'est pas explicitement ajoutée. Ajoutez un fichier `LICENSE` pour définir les conditions de réutilisation.

## Support

Aucune métadonnée sponsor/don n'a été trouvée dans cet instantané. Si vous souhaitez inclure des liens de support, ajoutez-les ici et dans les README traduits.
