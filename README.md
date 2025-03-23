# Second-Me Windows Port

## 概要

このプロジェクトは、[Mindverse/Second-Me](https://github.com/Mindverse/Second-Me)をWindows環境に移植するものです。Second-Meは、ローカルで動作する個人用AIアシスタントを構築するためのオープンソースプロジェクトです。

## 主な特徴

- **Windows環境での全稼働**: Mac専用だったSecond-Meをフル機能でWindows上で動作させることが可能
- **PowerShellスクリプト**: zshスクリプトをPowerShellスクリプトに移植
- **Windows向けビルド**: llama.cppなどのコンポーネントをWindows向けにビルド
- **簡単なセットアップ**: Windows環境に合わせた簡単なセットアップ手順

## 進捗状況

現在このプロジェクトは開発中です。詳細な進捗状況は[PROGRESS_REPORT.md](PROGRESS_REPORT.md)を参照してください。

## 設計ドキュメント

Windows移植に関する詳細な設計ドキュメントは[DESIGN_DOCUMENT.md](DESIGN_DOCUMENT.md)を参照してください。

## セットアップと利用（開発中）

*注意: このプロジェクトはまだ開発中であり、完全に機能する状態ではありません。*

### 前提条件

- Windows 10/11
- PowerShell 5.1以上
- 管理者権限（一部のインストールで必要）

### セットアップ手順（予定）

1. このリポジトリをクローン
```
git clone https://github.com/ZundamonnoVRChatkaisetu/Second-Me-Windows-Port.git
cd Second-Me-Windows-Port
```

2. セットアップスクリプトを実行（管理者権限で）
```
./scripts/setup.ps1
```

3. サービスを起動
```
./scripts/start.ps1
```

## 貢献方法

貢献は大歓迎です！以下の方法で参加できます：

- バグ報告: GItHubのIssuesで報告
- 機能追加: Pull Requestを送信
- ドキュメント改善: ドキュメントの更新や追加

## ライセンス

このプロジェクトは元のSecond-Meと同様、Apache License 2.0の下で提供されています。

## 謝辞

- [Mindverse/Second-Me](https://github.com/Mindverse/Second-Me): オリジナルのSecond-Meプロジェクト