# Second-Me Windows移植プロジェクト進捗管理

## プロジェクト概要

このプロジェクトは、[Mindverse/Second-Me](https://github.com/Mindverse/Second-Me)リポジトリで提供されているMac用のAIセルフプロトタイプをWindows環境で動作するように移植するものです。

## 環境情報

### 現状の環境（オリジナル）
- **OS**: macOS
- **シェル**: zsh
- **パッケージマネージャ**: Homebrew
- **Python環境**: Conda
- **外部ライブラリ**: llama.cpp (ビルドが必要)
- **フロントエンド**: Node.js/npm
- **バックエンド**: Python (Flask)

### 目標環境
- **OS**: Windows
- **シェル**: PowerShell/Command Prompt
- **パッケージマネージャ**: Chocolatey (Homebrewの代替)
- **Python環境**: Conda (Windows版)
- **外部ライブラリ**: llama.cpp (Windows向けにビルド)
- **フロントエンド**: Node.js/npm (変更なし)
- **バックエンド**: Python (Flask) (変更なし)

## 主要な変更点

### 1. スクリプト対応
- zshスクリプト → PowerShellスクリプト (.ps1) への置き換え
- パス区切り文字の変更 (`/` → `\`)
- バックグラウンドプロセス管理の変更

### 2. ビルド環境対応
- llama.cpp のWindows向けビルド方法の修正
- CMakeおよびVisual Studioの利用
- Windowsでのパス長制限への対応

### 3. 環境変数管理
- `.env`ファイルの変更（Windows環境向け）
- Windowsでの環境変数設定方法の適用

### 4. ファイルシステム対応
- 相対パスの使用方法の修正
- ファイルアクセス権限の管理方法の変更

## 進捗状況トラッカー

### Phase 1: 分析・準備
- [x] プロジェクト全体の把握
- [x] 主要な変更点の洗い出し
- [x] 進捗管理体制の確立
- [ ] 開発環境のセットアップ
- [ ] 必要なツールのインストール

### Phase 2: 基本機能の移植
- [ ] 環境構築スクリプトの移植 (setup.sh → setup.ps1)
- [ ] 起動スクリプトの移植 (start.sh → start.ps1)
- [ ] 停止スクリプトの移植 (stop.sh → stop.ps1)
- [ ] 再起動スクリプトの移植 (restart.sh → restart.ps1)
- [ ] 状態確認スクリプトの移植 (status.sh → status.ps1)

### Phase 3: llama.cpp対応
- [ ] Windowsでのllama.cppビルド方法の確立
- [ ] ビルドスクリプトの作成
- [ ] テスト実行の確認

### Phase 4: 環境設定・設定ファイル対応
- [ ] .envファイルのWindows対応
- [ ] confa_utils.shの移植
- [ ] 環境変数設定の修正

### Phase 5: 統合・テスト
- [ ] 全体フローのテスト
- [ ] エラーケースの対応
- [ ] パフォーマンス確認

### Phase 6: ドキュメント整備
- [ ] READMEの更新
- [ ] インストール手順書の作成
- [ ] トラブルシューティングガイドの作成

## 詳細タスクリスト

### setup.ps1 (setup.sh代替)
- [ ] Chocolateyのインストールチェック・セットアップ
- [ ] Condaのインストールチェック・セットアップ
- [ ] Python環境のセットアップ
- [ ] Node.js/npmのセットアップ
- [ ] CMakeの確認・インストール
- [ ] llama.cppのビルド

### start.ps1 (start.sh代替)
- [ ] 環境変数の読み込み
- [ ] ポートの使用状況確認
- [ ] バックエンドサービスの起動
- [ ] フロントエンドサービスの起動
- [ ] サービス起動確認

### conda_utils.ps1 (conda_utils.sh代替)
- [ ] Conda環境のアクティベーション関数
- [ ] Conda設定の検証関数

## 既知の課題・リスク

1. **llama.cppのビルド**: Windows環境でのビルドが複雑になる可能性。Visual Studioの依存関係、CMakeの設定などが必要。
2. **プロセス管理**: Windowsでのバックグラウンドプロセス管理はLinux/macOSと大きく異なる。
3. **パス長の制限**: Windowsのパス長制限（260文字）に注意が必要。
4. **権限の問題**: UAC（ユーザーアカウント制御）による権限の問題が発生する可能性。
5. **改行コードの違い**: LF (Unix) vs CRLF (Windows) の改行コードの違いによる問題。

## 次のステップ

1. ローカル開発環境でのテスト体制の確立
2. スクリプト変換の基本方針の決定と最初のスクリプト移植
3. llama.cppのWindows向けビルド方法の検証

## 参考リソース

- [オリジナルリポジトリ](https://github.com/Mindverse/Second-Me)
- [PowerShellスクリプティングガイド](https://learn.microsoft.com/en-us/powershell/scripting/overview)
- [llama.cpp Windows Build Instructions](https://github.com/ggerganov/llama.cpp/blob/master/README.md)
- [Chocolatey パッケージマネージャ](https://chocolatey.org/)
- [Miniconda for Windows](https://docs.conda.io/en/latest/miniconda.html)

## プロジェクト管理者

- 担当者: [担当者名]
- 最終更新日: 2025-03-23