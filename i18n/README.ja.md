[English](../README.md) · [العربية](README.ar.md) · [Español](README.es.md) · [Français](README.fr.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Tiếng Việt](README.vi.md) · [中文 (简体)](README.zh-Hans.md) · [中文（繁體）](README.zh-Hant.md) · [Deutsch](README.de.md) · [Русский](README.ru.md)


# NHI ハードウェア制御とイベントキャプチャ

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2F%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)
![Hardware](https://img.shields.io/badge/Hardware-EVK5%20%7C%20FMC4030%20%7C%20Arduino-success)
![UI](https://img.shields.io/badge/Web_UI-Tornado-0ea5e9)

次を組み合わせた、イベントカメラ実験向けのハードウェア統合プロジェクトです。
- EVK5 イベントカメラのキャプチャ（Prophesee Metavision スタック）
- FMC4030 ベースの CNC モーション制御
- Arduino シリアル LED 制御
- 最小構成の Web トリガー UI（Tornado）

> この README は、このリポジトリスナップショットに対する最初の完全ドラフトです。
> 前提: このチェックアウトには既存のルート `README.md` がなかったため、本ドキュメントはソースコードとパイプライン解析成果物から構築しています。

## 概要

主要なエンドツーエンドのワークフローは `app.py` に実装されています。

1. 必要に応じて既存の `data/` をタイムスタンプ付きフォルダ（`data_YYYYMMDD_HHMMSS`）へローテーション
2. Arduino LED（デフォルト `COM4`）へ接続
3. `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll` を介して CNC コントローラを初期化
4. LED と Y 軸移動シーケンスを実行
5. 必要に応じて EVK5 イベントを記録（現在の有効フローでは無効。下記注記を参照）
6. 軸位置ログを `data/axis_1_positions.csv` に保存

### ワークフロースナップショット

| ステージ | コンポーネント | 出力 |
|---|---|---|
| トリガー | Tornado `/start` | 非同期シーケンス実行 |
| モーション | FMC4030 コントローラ | 軸移動 + 位置ポーリング |
| ライティング | Arduino シリアル（`'1'` / `'0'`） | LED 状態制御 |
| センシング | EVK5 + Metavision | イベントストリーム / CSV エクスポート |
| 永続化 | ローカルファイルシステム | `data/*.csv`、ローテーション済みフォルダ |

このリポジトリには、代替/旧式のカメラスクリプト、フレーム後処理ユーティリティ、Metavision の Python サンプル群も含まれています。

## 機能

- モーション/キャプチャシーケンスを非同期起動する Tornado Web エンドポイント（`/start`）
- Metavision HAL を用いたトリガーチャネル（`MAIN`）有効化付き EVK5 イベント記録
- イベント時刻とシステム時刻の両方を含む CSV エクスポート
- `ctypes` とベンダー DLL を使用した FMC4030 モーター制御ラッパー
- 軸移動中のモーション位置 CSV ログ
- Arduino LED シリアル制御（`'1'`/`'0'` コマンド）
- フレームユーティリティスクリプト（`.npy` 形状確認、`.npy` から MP4 変換）
- 実験・参照用に同梱された Metavision サンプル `python_samples/`

## プロジェクト構成

```text
.
├── app.py                                   # メイン Web オーケストレータ
├── event_sensor_evk5.py                     # EVK5 レコーダー（Metavision）
├── event_sensor.py                          # 代替レコーダー（dv パッケージ）
├── evk5_test.py                             # 最小 EVK5 テストレコーダー
├── EventCamera_extTrig_Version 240506_ForEVK5.py
├── led.py                                   # Arduino LED シリアル制御
├── npy2video.py                             # NPY フレームスタック -> MP4 変換
├── npy_shape.py                             # フレーム配列形状を表示
├── motor_system.ini                         # ソフト原点設定
├── cnc/
│   ├── cnc.py                               # FMC4030 制御 + CLI
│   ├── motor_system.ini
│   ├── FMC4030Lib-x64-20220329/
│   │   ├── FMC4030-Dll.dll
│   │   ├── FMC4030-Dll.h
│   │   └── FMC4030-Dll.lib
│   └── archived/                            # 旧コントローラスクリプト
├── led/
│   └── led.ino                              # Arduino ファームウェアスケッチ
├── templates/
│   └── index.html                           # Start ボタン UI
├── python_samples/                          # Metavision サンプルプログラム
├── data-0503/                               # 過去実験データセット
├── i18n/                                    # 翻訳 README 用
└── .auto-readme-work/20260228_231403/      # README パイプライン成果物
```

## 前提条件

### ハードウェア

- EVK5 互換イベントカメラとドライバ/SDK
- 設定済み IP/ポートで到達可能な FMC4030 互換モーションコントローラ
- LED 制御用 Arduino ボード

### ソフトウェア

- Python 3.x
- ベンダー/ランタイム要件:
  - Prophesee Metavision Python モジュール（`metavision_core`、`metavision_hal`、関連 SDK モジュール）
  - Python `ctypes` による FMC4030 DLL ロード（`cnc/cnc.py` の `windll` 利用は、CNC 経路について Windows を示唆）
- 各スクリプトで使用される Python ライブラリ:
  - `tornado`, `numpy`, `opencv-python`, `pytz`, `pyserial`
  - オプション/代替スタック: `dv`

### 環境デフォルト

| 設定 | デフォルト | 場所 |
|---|---|---|
| Arduino シリアルポート | `COM4` | `app.py`, `led.py` |
| CNC IP | `192.168.0.30` | `cnc/cnc.py` |
| CNC ポート | `8088` | `cnc/cnc.py` |

注記:
- 現在のスナップショットには `requirements.txt` または `pyproject.toml` がありません。
- シリアルポートは `app.py` と `led.py` で `COM4` がデフォルトです。
- `cnc/cnc.py` のデフォルト CNC ネットワーク設定: IP `192.168.0.30`、ポート `8088`。

## インストール

1. リポジトリをクローン:

```bash
git clone <your-repo-url>
cd nhi_hardware__lachlanchen
```

2. Python 環境を作成して有効化:

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
source .venv/bin/activate
```

3. コアスクリプトで使用する基本 Python 依存関係をインストール:

```bash
pip install tornado numpy opencv-python pytz pyserial
```

4. 利用環境に必要なカメラ SDK 依存関係をインストール:

```bash
# 例: Prophesee Metavision Python パッケージをインストール
# OS とカメラバージョンに応じた SDK/ディストリビューション手順に従ってください。
```

## 使い方

### 1) Web オーケストレーションシーケンス（主要経路）

リポジトリルートから実行してください（`app.py` の相対パス解決に重要）。

```bash
python app.py
```

その後、次を開きます。

```text
http://localhost:8888
```

**Start Sequence** をクリックすると、モーション/LED ワークフローがトリガーされます。

現在 app が解釈する任意引数:

```bash
python app.py --record_events True
```

重要な挙動メモ: `start_sequence()` は実行途中で `record_events = False` を再設定しているため、コードを調整しない限り EVK5 記録は無効のままになる可能性があります。

### 2) EVK5 イベント記録（直接 CLI）

```bash
python event_sensor_evk5.py -i "" -d 10 -z Asia/Hong_Kong -o event_output
```

引数:
- `-i, --input`: EVK5 デバイスまたは記録の入力ソース/パス
- `-d, --duration`: 記録時間（秒）
- `-z, --timezone`: タイムスタンプ整形用タイムゾーンラベル
- `-o, --output`: 出力ベース名（`data/<name>.csv` に保存）

### 3) CNC 制御（直接 CLI）

`cnc.py` の相対 DLL パスを正しく解決するため、`cnc/` から実行します。

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 20
```

その他の利用可能オプション:

```bash
# 現在位置をソフト原点として設定
python cnc.py --set-origin

# 絶対座標 x,y,z へ移動
python cnc.py --move 0,30,0 --speed 20

# 指定座標へ移動後に原点設定
python cnc.py --set-origin 0,30,0 --speed 20
```

### 4) LED テストスクリプト

```bash
python led.py
```

`COM4` を使わない場合はコード内のポートを更新してください。

### 5) ユーティリティ

```bash
# 1つのフォルダからフレーム配列形状を表示
python npy_shape.py data_20250101_120000

# カレントディレクトリ内の全 data_* フォルダのフレーム配列形状を表示
python npy_shape.py -a
```

`npy2video.py` は `npy_to_video(npy_file_path, output_video_path, fps=5)` を提供しており、インポートして使うかローカルパス向けに編集できます。

## 設定

- `motor_system.ini` と `cnc/motor_system.ini`:
  - ソフトウェア原点座標を永続化（X/Y/Z の `ORIGIN` セクション）
- `app.py`:
  - Arduino シリアルポート: `ArduinoLED(port='COM4')`
  - CNC DLL パス: `cnc/FMC4030Lib-x64-20220329/FMC4030-Dll.dll`
- `event_sensor.py`（DV 経路）:
  - `DV_PORT` デフォルト `7777`
  - `DV_PORT_FRAME` デフォルト `7778`

## 例

### 例 A: Web からフルシーケンスを開始

```bash
python app.py
# http://localhost:8888 にアクセスして Start Sequence を押す
```

想定出力:
- `data/axis_1_positions.csv`（CNC 位置トレース）
- 有効経路でイベント記録を有効化している場合は任意で `data/<events>.csv`

### 例 B: EVK5 イベントを 60 秒記録

```bash
python event_sensor_evk5.py -d 60 -o run_001_events -z Asia/Hong_Kong
```

想定出力:
- `data/run_001_events.csv`

### 例 C: CLI から Y 軸を往復移動

```bash
cd cnc
python cnc.py --axis 1 --dir 1 --distance 30 --speed 100
python cnc.py --axis 1 --dir -1 --distance 30 --speed 100
```

## 開発メモ

- 現在のリポジトリは、アクティブ/アーカイブ済みスクリプトが混在する研究・試作ワークスペースに見えます。
- 大きな生成物（`event_output.csv`、`data-0503/`）がコミット済みです。リポジトリを配布する場合は、データ保持方針と `.gitignore` 更新を検討してください。
- `python_samples/` には有用なカメラ SDK 例が含まれますが、コアの統合処理には不要な依存関係を含む可能性があります。
- 想定されるコード品質改善:
  - `app.py` の `argparse` 真偽値処理は改善余地があります（CLI 解析で `type=bool` は誤解を招きがちです）。
  - `start_sequence()` のイベント記録フラグ処理は、現状 CLI 初期値を上書きしています。

## トラブルシューティング

- `ImportError: metavision_*` モジュールが見つからない:
  - Metavision SDK の Python 環境をインストール/設定してください。
- `ImportError: No module named dv`:
  - `event_sensor.py` 経路を使う場合は DV Python パッケージをインストールしてください。
- CNC DLL のロード失敗:
  - OS 互換性と、`FMC4030-Dll.dll` が想定相対パスに存在することを確認してください。
  - `cnc/` ディレクトリから `cnc.py` を実行するか、`dll_path` を調整してください。
- LED シリアル接続エラー:
  - Arduino ポート割り当て（`COM4` と実際のポート）を確認してください。
  - 別プロセスがシリアルデバイスを占有していないことを確認してください。
- Web 実行後に `data/` にファイルがない:
  - 書き込み権限と、イベント記録経路が有効かを確認してください。

## ロードマップ

- 依存関係マニフェスト（`requirements.txt` または `pyproject.toml`）とバージョン固定を追加
- 実行時設定（ポート、IP、DLL パス、速度プロファイル）を統一設定ファイルへ外出し
- カメラバックエンド（EVK5 と DV）を 1 つのインターフェースに統合し、明確なモード選択を提供
- モーション/センサー I/F のテスト・モックを追加し、ハードウェアなしでも CI を実行可能にする
- 実験ごとの構造化ログと実行メタデータを追加
- `i18n/` 配下の翻訳 README を生成し維持

## コントリビュート

次の領域でのコントリビュートを歓迎します。
- ハードウェア抽象化の改善
- 設定と再現性の向上
- ドキュメント拡充と翻訳
- モーション制御の安全チェックと運用ガードレール

推奨コントリビューションフロー:
1. Fork して機能ブランチを作成
2. 焦点が明確でレビューしやすい変更を作成
3. 手元のハードウェア構成で検証
4. 再現手順とログを添えて pull request を提出

## ライセンス

このリポジトリスナップショットにはライセンスファイルがありません。

前提: プロジェクトライセンスが明示的に追加されるまで、すべての権利は留保されます。再利用条件を定義するには `LICENSE` ファイルを追加してください。

## サポート

このスナップショットではスポンサー/寄付メタデータは見つかりませんでした。サポートリンクを掲載したい場合は、ここおよび翻訳 README に追加してください。
