# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

このリポジトリはPavel Yosifovich著「Windows Kernel Programming Second Edition」のサンプルコードです。章ごと（02-15）に整理され、Windowsカーネルドライバー開発の様々な側面を実演しています。

## ビルドコマンド

全プロジェクトはWindows Driver Kit (WDK)を使用したVisual Studioで構築します：

```bash
# 特定のドライバーをビルド（章ディレクトリから）
msbuild Chapter##.sln /p:Configuration=Debug /p:Platform=x64

# 個別プロジェクトをビルド
msbuild ProjectName/ProjectName.vcxproj /p:Configuration=Debug /p:Platform=x64

# クリーンビルド
msbuild Chapter##.sln /t:Clean /p:Configuration=Debug /p:Platform=x64
```

## アーキテクチャと主要コンポーネント

### KTL (Kernel Template Library)
`/ktl/`に配置 - カーネル開発用のカスタムSTL風ライブラリ：
- `BasicString.h` - カーネルセーフな文字列実装
- `Vector.h` - カーネルモード用動的配列
- `FastMutex.cpp/h`, `ExecutiveResource.cpp/h`, `SpinLock.cpp/h` - 同期プリミティブ
- `Memory.cpp/h` - メモリ管理ユーティリティ
- `Locker.h` - RAIIロックラッパー

### ドライバーカテゴリ

1. **基本デバイスドライバー** (第2-5章): IOCTL通信を使用したシンプルなドライバー
   - パターン: ドライバー + テストクライアントのペア
   - 例: `Booster` - スレッド優先度ブーストドライバー

2. **高度なドライバー** (第7-11章): 複雑な状態管理とコールバック
   - `SysMon` - プロセス/スレッドアクティビティ監視
   - `KMelody` - オーディオ再生ドライバー
   - `Tables` - プロセス/スレッド情報プロバイダー

3. **ファイルシステムミニフィルター** (第12章): ファイルシステムフィルタリング
   - `KBackup/KBackup2` - ファイルバックアップフィルター
   - `KDelProtect` - 削除保護
   - `KHide` - ファイル隠蔽

4. **ネットワークフィルター** (第13章): WFP (Windows Filtering Platform)
   - `ProcessNetFilter` - プロセスベースのネットワークブロッキング

5. **PnPドライバー** (第14-15章): INFファイル付きプラグアンドプレイドライバー
   - `KDevMon` - USB/PnPデバイスモニター

### 共通パターン

- **ドライバー構造**: DriverEntry → デバイス作成 → IRP処理 → DriverUnload
- **通信**: デバイスシンボリックリンク + `*Common.h`ファイルで定義されたIOCTL
- **同期**: KMUTEX、KEVENT、KTLからのカスタム同期の多用
- **クライアントパターン**: CreateFile → DeviceIoControl/ReadFile/WriteFile → CloseHandle

### テスト

各ドライバーには通常、対応するテストクライアントがあります（例：`Booster.sys`に対する`Boost.cpp`）。テスト方法：

1. ドライバーとクライアントプロジェクトをビルド
2. ドライバーをインストール（PnPドライバーはINF使用、その他は`sc create`使用）
3. 管理者権限のコマンドプロンプトからテストクライアントを実行

### 開発メモ

- ドライバーは`x64/Debug/`または`x64/Release/`に出力
- 多くのドライバーはKTLにリンクせず同期コードをコピー
- IOCTLと共有構造体は`*Common.h`ファイルで定義
- ほとんどのプロジェクトでプリコンパイルヘッダー（`pch.h/cpp`）を使用
- ミニフィルターの場合、INFファイルで高度要件を確認