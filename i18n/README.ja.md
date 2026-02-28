[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


[![LazyingArt banner](https://github.com/lachlanchen/lachlanchen/raw/main/figs/banner.png)](https://github.com/lachlanchen/lachlanchen/blob/main/figs/banner.png)

# NHI ハードウェア制御とイベントキャプチャ

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)
![Docs](https://img.shields.io/badge/Docs-English%20%2B%20i18n-0f766e)

## 📖 クイックナビゲーション

| 利用セクション | 目的 |
|---|---|
| インストール | 環境と依存関係を準備 |
| 使用方法 | WebオーケストレータとCLIの実行 |
| 設定 | シリアル、ネットワーク、デフォルト値を調整 |
| 例 | 実用的なコマンド例を実行 |
| トラブルシューティング | よくあるセットアップ問題を解決 |

## 🧭 プロジェクト概要

| フォーカス | 詳細 |
|---|---|
| ミッション | 再現性のある研究向けワークフローのために、イベントキャプチャ・モーション制御・LEDシグナルを一体化する |
| メインエントリ | `app.py`（TornadoのWebトリガー + 非同期シーケンス制御） |
| 主な入力 | EVK5 イベントストリーム、FMC4030 軸制御、Arduino シリアルコマンド |
| 主な出力 | `data/axis_1_positions.csv`、任意でイベントCSVエクスポート |
| 対応プラットフォーム | Windows / Linux（SDK とハードウェアの入手可否に依存） |

EVK5イベントカメラ実験向けに、次を統合するハードウェア制御プロジェクトです。
- EVK5 イベントカメラ取り込み（Prophesee Metavision スタック）
- FMC4030 ベースの CNC モーション制御
- Arduino シリアル LED 制御
- 最小限の Web トリガーUI（Tornado）

> 前提: ハードウェア、DLL、SDK 環境はホストごとに異なり、リポジトリのコードから推定しています。コマンドの実際の挙動は OS、ドライバ版、実行時の利用可否によって変わる場合があります。

## 🧠 概要

主なエンドツーエンドのワークフローは `app.py` で実装されています。

1. 任意で既存 `data/` をタイムスタンプ付きフォルダ（`data_YYYYMMDD_HHMMSS`）へローテーション
2. Arduino LED（既定 `COM4`）に接続
3. `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` で CNC コントローラを初期化
4. LED とY軸移動シーケンスを実行
5. 任意で EVK5 イベントを記録（現在の実行フローでは無効、下記注記を参照）
6. 軸位置ログを `data/axis_1_positions.csv` に保存

### ワークフロースナップショット

| フェーズ | コンポーネント | 出力 |
|---|---|---|
| トリガー | Tornado `/start` | 非同期シーケンス実行 |
| モーション | FMC4030 コントローラ | 軸移動 + 位置ポーリング |
| ライティング | Arduino シリアル（`'1'` / `'0'`） | LED 状態制御 |
| センサー | EVK5 + Metavision | イベントストリーム / CSV 出力 |
| 保存 | ローカルファイルシステム | `data/*.csv`、ローテーション済みフォルダ |

このリポジトリには、代替/レガシーのカメラスクリプト、フレーム後処理ユーティリティ、Metavision Python のサンプル群も含まれています。

## ✨ 機能

- モーション/キャプチャシーケンスを非同期で開始する Tornado Web エンドポイント（`/start`）
- Metavision HAL を用いたトリガーチャネル（`MAIN`）有効化付き EVK5 イベント録画
- イベント時刻とシステム時刻を含むイベント CSV のエクスポート
- `ctypes` とベンダー DLL を利用した FMC4030 モーター制御ラッパー
- 軸移動中のモーション位置ログを CSV に記録
- Arduino LED シリアル制御（`'1'` / `'0'` コマンド）
- フレームユーティリティスクリプト（`.npy` 形状の確認、`.npy` から MP4 変換）
- 実験とリファレンスのために同梱された `python_samples/` Metavision サンプル

## 🗂️ プロジェクト構成

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

## 🧰 前提条件

### ハードウェア

- EVK5 互換イベントカメラとドライバ/SDK
- 設定済み IP/ポートで到達可能な FMC4030 互換モーションコントローラ
- LED制御用 Arduino ボード

### ソフトウェア

- Python 3.x
- ベンダー/実行時サポート:
  - Prophesee Metavision Python モジュール（`metavision_core`、`metavision_hal`、関連 SDK モジュール）
  - `ctypes` 経由での FMC4030 DLL 読み込み（`cnc/cnc.py` の `windll` 使用から、CNC 経路は Windows 前提）
- スクリプト全体で使われる Python ライブラリ:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - 任意/代替スタック: `dv`

### 環境設定の既定値

| 設定 | 既定値 | 場所 |
|---|---|---|
| Arduino シリアルポート | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC ポート | `8088` | `cnc/cnc.py` |

補足:
- 現在のスナップショットには `requirements.txt` または `pyproject.toml` がありません。
- `app.py` と `led.py` ではシリアルポートが既定 `COM4` です。
- `cnc/cnc.py` の既定 CNC ネットワーク設定: IP `192.168.0.30`、ポート `8088`。

## 🔧 インストール

1. リポジトリをクローン:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Python 環境を作成し有効化:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. コアスクリプトで使う基本 Python 依存をインストール:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 環境に必要なカメラ SDK 依存をインストール:

```bash
# Example placeholder: install Prophesee Metavision Python packages
# OS とカメラバージョン向けの SDK / 配布手順に従ってください。
```

## 🧪 使い方

### 1) Web オーケストレーションシーケンス（主要）

リポジトリのルートから実行してください（`app.py` の相対パス解決に重要です）:

```bash
python app.py
```

次に以下を開きます。

```text
http://localhost:8888
```

**Start Sequence** をクリックすると、モーション/LED ワークフローをトリガーできます。

`app.py` で現在解釈される任意引数:

```bash
python app.py --record_events True
```

重要な挙動: `start_sequence()` は実行途中で `record_events = False` を再設定するため、コードを変更しない限り EVK5 録画は無効のまま残ることがあります。

### 2) EVK5 イベント録画（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

引数:
- `-i, --input`: EVK5 デバイスまたは録画の入力元/パス
- `-d, --duration`: 録画時間（秒）
- `-z, --timezone`: タイムスタンプ整形用タイムゾーンラベル
- `-o, --output`: 出力ベース名（`data/<name>.csv` に保存）

### 3) CNC 制御（直接 CLI）

`cnc.py` の相対 DLL パスを正しく解決するため、`cnc/` から実行してください。

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

他の利用可能オプション:

```bash
# 現在位置をソフト原点として設定
python cnc.py --set-origin

# 絶対座標 x,y,z へ移動
python cnc.py --move 0,30,0 --speed 20

# 指定座標へ移動後に原点を設定
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED テストスクリプト

```bash
python led.py
```

`COM4` を使っていない場合は、コード内のポートを更新してください。

### 5) ユーティリティ

```bash
# 1 フォルダ分のフレーム配列形状を表示
python npy_shape.py data_20250101_120000

# カレントディレクトリ配下の全 `data_*` フォルダのフレーム配列形状を表示
python npy_shape.py -a
```

`npy2video.py` は `npy_to_video(npy_file_path, output_video_path, fps=5)` を提供しており、インポートして使用するか、ローカルパス向けに編集できます。

## ⚙️ 設定

- `motor_system.ini` と `cnc/motor_system.ini`:
  - ソフト原点座標を保持（X/Y/Z の `ORIGIN` セクション）
- `app.py`:
  - Arduino シリアルポート: `ArduinoLED(port='COM4')`
  - CNC DLL パス: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 経路）:
  - `DV_PORT` の既定値 `7777`
  - `DV_PORT_FRAME` の既定値 `7778`

## 📸 例

### 例 A: Web でフルシーケンスを開始

```bash
python app.py
# http://localhost:8888 を開き、Start Sequence を押す
```

期待される出力:
- `data/axis_1_positions.csv`（CNC 位置トレース）
- EVK5 録画をアクティブ経路で有効化した場合、任意で `data/<events>.csv`

### 例 B: 60秒間 EVK5 イベントを記録

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

期待される出力:
- `data/run_001_events.csv`

### 例 C: CLI で Y 軸を往復移動

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 🧭 開発ノート

- 現在このリポジトリは、実行系とアーカイブ済みスクリプトが混在した研究/試作ワークスペースに見えます。
- 大きな生成成果物（`event_output.csv`、`data-0503/`）がコミット済みのため、配布用途ではデータ保持方針と `.gitignore` の見直しを検討してください。
- `python_samples/` には有用なカメラ SDK サンプルが含まれますが、コア制御のために必須ではない依存を含む可能性があります。
- コード品質改善候補:
  - `app.py` の `argparse` の bool 取り扱いは改善の余地があります（CLI で `type=bool` は誤解を招きやすい）。
  - `start_sequence()` のイベント録画フラグ制御は現状、初期 CLI 値を上書きします。

## 🛠️ トラブルシューティング

- `ImportError: metavision_*` モジュールがない:
  - Metavision SDK Python 環境のインストール/設定を行ってください。
- `ImportError: No module named dv`:
  - `event_sensor.py` を使う場合、DV Python パッケージをインストールしてください。
- CNC DLL 読み込みエラー:
  - OS の互換性、および `FMC4030-Dll.dll` が想定の相対パスにあることを確認してください。
  - `cnc.py` は `cnc/` ディレクトリから実行するか、`dll_path` を調整してください。
- LED シリアル接続エラー:
  - Arduino ポート割当（`COM4` と実際のポート）を確認してください。
  - 他プロセスがシリアルデバイスを占有していないか確認してください。
- Web 実行後に `data/` が空:
  - 書き込み権限とイベント録画パスの有効化状態を確認してください。

## 🗺️ ロードマップ

- 依存関係マニフェスト（`requirements.txt` または `pyproject.toml`）の追加とバージョン固定
- 実行時設定（ポート、IP、DLL パス、速度プロファイル）を統一設定ファイルへ外部化
- カメラバックエンド（EVK5 と DV）を1つのインターフェースに正規化し、明確なモード選択を実装
- モーションおよびセンサー I/F のテスト/モックを追加し、ハードウェアなしで CI 可能に
- 実験ごとの構造化ログと実行メタデータの追加
- `i18n/` 配下で README の翻訳を生成・更新

## 🤝 コントリビュート

コントリビュート歓迎領域:
- ハードウェア抽象化の改善
- 設定・再現性の改善
- ドキュメントの拡充と翻訳
- モーション制御向けの安全チェックと運用ガードレール

推奨コントリビューション手順:
1. Fork して feature ブランチを作成
2. 焦点を絞ったレビューしやすい変更を行う
3. 自身のハードウェア環境で検証
4. 再現手順とログ付きで pull request を提出

## ライセンス

このリポジトリスナップショットにはライセンスファイルがありません。

前提: プロジェクトに明示的なライセンスが追加されるまでは、すべての権利は保有されます。利用条件を定義するため `LICENSE` を追加してください。


## ❤️ Support

| Donate | PayPal | Stripe |
| --- | --- | --- |
| [![Donate](https://camo.githubusercontent.com/24a4914f0b42c6f435f9e101621f1e52535b02c225764b2f6cc99416926004b7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446f6e6174652d4c617a79696e674172742d3045413545393f7374796c653d666f722d7468652d6261646765266c6f676f3d6b6f2d6669266c6f676f436f6c6f723d7768697465)](https://chat.lazying.art/donate) | [![PayPal](https://camo.githubusercontent.com/d0f57e8b016517a4b06961b24d0ca87d62fdba16e18bbdb6aba28e978dc0ea21/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f50617950616c2d526f6e677a686f754368656e2d3030343537433f7374796c653d666f722d7468652d6261646765266c6f676f3d70617970616c266c6f676f436f6c6f723d7768697465)](https://paypal.me/RongzhouChen) | [![Stripe](https://camo.githubusercontent.com/1152dfe04b6943afe3a8d2953676749603fb9f95e24088c92c97a01a897b4942/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5374726970652d446f6e6174652d3633354246463f7374796c653d666f722d7468652d6261646765266c6f676f3d737472697065266c6f676f436f6c6f723d7768697465)](https://buy.stripe.com/aFadR8gIaflgfQV6T4fw400) |
