# Tile Crawl 日本語 / 簡体字中国語 ローカライズパッチ（TileCrawlPatch）

[中文版 README](README.md)

Steam 版ゲーム **Tile Crawl Demo** に **日本語** と **簡体字中国語** の UI を追加する非公式パッチです。

- ゲーム内 `Settings → Language` に `日本語` / `中文` が追加され、いつでも切り替えられます
- **9 つの String Table / 589 エントリ（各言語）** を翻訳済み。空欄やフォールバック（原文まま）はありません
- **Noto Sans JP / Noto Sans SC**（SIL OFL 1.1）を同梱。日本語と中国語でそれぞれ専用フォントを使うため、字形が混ざりません
- 単一ファイルのインストーラー（約 22 MB）。**ゲームフォルダを自動検出**、言語ごとのインストール、**ワンクリック uninstall**、インストール前の自動バックアップに対応

![日本語表示のメインメニュー](assets/screenshot_menu_ja.png)

---

## 目次

- [クイックスタート](#クイックスタート)
- [主な機能](#主な機能)
- [インストーラーの使い方](#インストーラーの使い方)
- [コマンドライン](#コマンドライン)
- [アンインストールと復元](#アンインストールと復元)
- [バックアップ](#バックアップ)
- [変更されるファイル](#変更されるファイル)
- [ソースからビルド](#ソースからビルド)
- [ディレクトリ構成](#ディレクトリ構成)
- [既知の問題](#既知の問題)
- [ライセンスとサードパーティ](#ライセンスとサードパーティ)

---

## クイックスタート

1. Releases から `TileCrawlPatchInstaller.exe` をダウンロード
2. ダブルクリックで実行（SmartScreen にブロックされた場合は「詳細情報 → 実行」）
3. ゲームフォルダが自動検出されるので、`日本語` / `簡体字中国語` を選んで「インストール」
4. ゲームを起動 → `Settings → Language` → `日本語` または `中文` を選択

> インストール時は**ゲームを終了**してください。インストーラーはゲームのファイルを削除せず、追記と追加のみを行い、変更前に必ずバックアップを作成します。

## 主な機能

| 項目 | 内容 |
| --- | --- |
| 追加言語 | 日本語（ja）、簡体字中国語（zh-Hans） |
| 翻訳範囲 | 9 つの String Table、589 エントリ / 言語（メニュー・設定・各種テキスト） |
| フォント | Noto Sans JP（日本語）/ Noto Sans SC（中国語）。実行時に文字セットから TMP フォントアトラスを生成します |
| 字形の分離 | 日中は同じ漢字コードポイントを共有しますが字形設計が異なります。本パッチは日 / 中環境では**その言語のフォントのみ**を適用するため、「中国語なのに日本語の字形になる」ことを防ぎます。その他の言語では両方を適用し、言語一覧の「日本語 / 中文」が豆腐になるのを防ぎます |
| 切り詰め対策 | ゲーム側の `Ellipsis` 切り詰め + 固定フォントサイズにより、幅の広い日中テキストが 0 文字に切り詰められて**ラベルが空白になる**不具合があります。本パッチはフォントサイズの縮小を許可し、切り詰めを解除します |
| 言語リセット防止 | 起動後に `LocalizationSettings` の初期化コルーチンが再実行され、言語がシステム既定の `en` に戻される仕様があります（「設定画面を開くと言語が勝手に変わる」ように見える原因）。本パッチはプレイヤー操作ではない変更を検出し、翌フレームで選択言語を復元します |
| インストーラー言語 | 日本語 / 簡体字中国語 / English。UI とログはすべてローカライズ済みで、テキスト長に合わせてレイアウトも自動調整されます |
| 復元可能 | ワンクリック uninstall、インストールごとの自動バックアップ、Steam の「ゲームファイルの整合性を確認」でも復元できます |

## インストーラーの使い方

画面の上から順に：

- **UI 言語**：インストーラー自体の表示言語を切り替えます
- **ゲームフォルダ**：自動検出（Steam の全ライブラリをスキャン）または「参照」で手動指定
- **インストール言語**：`日本語` / `簡体字中国語` を選択（どちらか一方だけでも可）
- **状態**：インストール済みか、バージョン、インストール済み言語、書き込み権限を表示
- **ログ**：インストール / アンインストールの全行程
- ボタン：**インストール / 更新** · **アンインストール** · **バックアップを開く** · **再検出**

すでにインストール済みの場合は、自動的に旧版をアンインストールしてから新版を入れ、新しいバックアップを作成します。

## コマンドライン

GUI なしのバッチモードも用意しています（自動化やトラブルシュート向け）：

```bat
TileCrawlPatchInstaller.exe --detect
TileCrawlPatchInstaller.exe --install  "<ゲームフォルダ>" ja,zh-Hans
TileCrawlPatchInstaller.exe --install  "<ゲームフォルダ>" ja
TileCrawlPatchInstaller.exe --uninstall "<ゲームフォルダ>"
```

出力はコンソールに表示されると同時に、同じフォルダの `installer_cli.log` に追記されます。

## アンインストールと復元

次のいずれかで元に戻せます：

1. インストーラーの「アンインストール」
2. コマンドラインの `--uninstall "<ゲームフォルダ>"`
3. Steam でゲームを右クリック → プロパティ → インストールファイル → **ゲームファイルの整合性を確認**（変更された json が復元されます）

アンインストールでは、パッチ DLL の削除、`ScriptingAssemblies.json` / `RuntimeInitializeOnLoads.json` からのパッチ項目の削除、`LocalizationPatch` フォルダの削除を行います。バックアップは既定で保持されます。

## バックアップ

インストールのたびに（更新前の自動アンインストール含む）次の場所へ作成されます。

```
<ゲームフォルダ>\tile-crawl_Data\_tilecrawl_patch_backup\backup_YYYYMMDD-HHMMSS\
```

変更対象の `ScriptingAssemblies.json`、`RuntimeInitializeOnLoads.json` と、`record.json`（日時、ゲームフォルダ、パッチ DLL / データフォルダのインストール前の状態）が含まれます。

インストーラーの「バックアップを開く」で直接開けます。強制的に戻したい場合は、バックアップ内の json を `tile-crawl_Data\` にコピーしてください。

## 変更されるファイル

| ファイル | 内容 |
| --- | --- |
| `tile-crawl_Data\Managed\TileCrawlLocPatch.dll` | 追加（パッチ本体、約 26 KB） |
| `tile-crawl_Data\ScriptingAssemblies.json` | アセンブリ名を 1 件追記（Mono が読み込むため） |
| `tile-crawl_Data\RuntimeInitializeOnLoads.json` | `RuntimeInitializeOnLoad` のフックを 1 件追記（起動時にパッチが走ります） |
| `tile-crawl_Data\LocalizationPatch\` | データフォルダを追加（翻訳 json、文字セット txt、フォント otf、manifest.json、patch.log） |

パッチはゲーム本来のアセットやアセンブリを**書き換えず**、アセンブリを 1 つ「追加」するだけです。ゲーム更新でこれらの json が上書きされた場合は、インストーラーをもう一度実行してください。

## ソースからビルド

必要なもの：

- .NET SDK（netstandard2.1）
- Python 3.8 以降（インストーラー UI とデータ生成）
- PyInstaller（`pip install pyinstaller`）

```bash
# 1) パッチ DLL をビルド（既定は Steam の既定パス。-p:GameManagedDir で上書き可）
cd plugin
dotnet build -c Release -o out --no-restore -p:GameManagedDir="D:\...\tile-crawl_Data\Managed"

# 2) 翻訳と文字セットを再生成（任意。data/ に生成済みのものがあります）
cd .. && python src/gen_data.py

# 3) payload/ を構成（フォントが無い場合は Google Fonts リポジトリから自動ダウンロード）
python src/build.py

# 4) 単一 exe にパッケージング
cd installer
pyinstaller TileCrawlPatchInstaller.spec --noconfirm --distpath ../dist --workpath ../build/pyinstaller
```

成果物：`dist\TileCrawlPatchInstaller.exe`

## ディレクトリ構成

```
TileCrawlPatch/
├── VERSION                 パッチバージョン
├── assets/                 README 用スクリーンショット
├── data/                   生成物：翻訳 json、文字セット txt
│   ├── shared.json             テーブル構成のメタデータ
│   ├── strings_ja.json         日本語訳（589 エントリ）
│   ├── strings_zh-Hans.json    簡体字中国語訳（589 エントリ）
│   └── chars_*.txt             事前生成する文字セット
├── installer/
│   ├── app.py              インストーラー UI（tkinter）
│   ├── i18n.json           インストーラーの文言（日 / 中 / 英）
│   └── TileCrawlPatchInstaller.spec
├── plugin/
│   ├── PatchEntry.cs       パッチ本体（Unity 実行時に注入）
│   └── TileCrawlLocPatch.csproj
├── src/
│   ├── patchlib.py         インストール / アンインストール / バックアップ / 検出のコアロジック
│   ├── build.py            payload/ の構成
│   ├── gen_data.py         翻訳と文字セットの生成
│   └── gen_shared.py / catbin.py   ゲームアセットから原文を抽出する補助スクリプト
└── payload/                ビルド中間生成物（インストーラーがコピーするファイル一式）
```

## 既知の問題

- **漢字の統一（Han Unification）**：日中は同じコードポイントを共有するため、「画」「面」などは Noto Sans JP と SC で字形が完全に同一です。これはフォントの使い分けミスではなく漢字自体の共通字形です。実際に差が出る字（「语 / 频 / 门 / 设」など）は、日 / 中それぞれの環境で専用フォントが使われます。
- ウクライナ語など一部の言語名に含まれるキリル文字 `ї`（U+0457）は Noto Sans JP / SC に収録されておらず、事前生成ログに `missing=1` が出ます。日中表示には影響ありません。

## ライセンスとサードパーティ

- パッチの**コードとスクリプト**：MIT License（[LICENSE](LICENSE)）
- **Noto Sans JP / Noto Sans SC**：SIL Open Font License 1.1、© Google LLC（[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)）
- 本パッチは**非公式**のファンメイド作品であり、ゲーム開発者とは関係がありません。ゲームのオリジナルデータは一切含みません。ゲーム本体はご自身で入手してください。
