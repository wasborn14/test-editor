# はじめに - Test Editor

Test Editorは、AI技術を活用した次世代のテキストエディタです。このガイドでは、初回セットアップから基本的な使用方法まで、段階的に説明します。

## システム要件

### 必須要件
- **Node.js**: 18.0.0 以降
- **RAM**: 最低 4GB（推奨 8GB）
- **ストレージ**: 最低 2GB の空き容量
- **OS**: Windows 10/11、macOS 10.15+、Ubuntu 18.04+

### 推奨環境
- **CPU**: Intel i5 / AMD Ryzen 5 以上
- **GPU**: CUDA対応（AI機能の高速化用）
- **ネットワーク**: 安定したインターネット接続（クラウド機能用）

## インストール手順

### 方法1: npm経由でのインストール

```bash
# グローバルインストール
npm install -g test-editor

# インストール確認
test-editor --version
```

### 方法2: ソースからのビルド

```bash
# リポジトリのクローン
git clone https://github.com/wasborn14/test-editor.git
cd test-editor

# 依存関係のインストール
npm install

# ビルド実行
npm run build

# アプリケーション起動
npm start
```

### 方法3: Docker利用

```bash
# Docker イメージの取得
docker pull wasborn14/test-editor:latest

# コンテナ起動
docker run -p 8080:8080 wasborn14/test-editor:latest
```

## 初期設定

### 1. ユーザーアカウント作成

アプリケーション初回起動時に、アカウント作成画面が表示されます：

```bash
test-editor --setup
```

必要な情報を入力：
- ユーザー名
- メールアドレス
- パスワード（8文字以上、英数字記号を含む）

### 2. AI機能の設定

AI機能を利用するには、APIキーの設定が必要です：

```bash
# 設定ファイルの編集
test-editor config set ai.apiKey "your-api-key-here"

# プロバイダーの選択（OpenAI, Anthropic, Cohere等）
test-editor config set ai.provider "openai"
```

### 3. ワークスペースの設定

```bash
# デフォルトワークスペースの設定
test-editor config set workspace.default "/path/to/your/projects"

# Git設定の連携
test-editor config set git.username "your-git-username"
test-editor config set git.email "your-email@example.com"
```

## 基本的な使用方法

### プロジェクトの作成

```bash
# 新規プロジェクト作成
test-editor create my-project

# 既存ディレクトリを開く
test-editor open ./existing-project

# リモートリポジトリからクローン
test-editor clone https://github.com/user/repo.git
```

### エディタの基本操作

#### ファイル操作
- `Ctrl+N`: 新規ファイル作成
- `Ctrl+O`: ファイルを開く
- `Ctrl+S`: ファイル保存
- `Ctrl+Shift+S`: 名前を付けて保存

#### AI支援機能
- `Ctrl+Space`: AI補完の呼び出し
- `Ctrl+Shift+A`: AIアシスタントの起動
- `Ctrl+Alt+R`: コードリファクタリング提案
- `Ctrl+Alt+G`: コード生成

#### 検索・置換
- `Ctrl+F`: ファイル内検索
- `Ctrl+Shift+F`: プロジェクト全体検索
- `Ctrl+H`: 置換
- `Alt+F`: セマンティック検索

### プラグインの管理

```bash
# 利用可能なプラグイン一覧
test-editor plugins list

# プラグインのインストール
test-editor plugins install eslint-integration

# プラグインの有効化
test-editor plugins enable eslint-integration

# プラグインの設定
test-editor plugins config eslint-integration
```

## トラブルシューティング

### よくある問題

#### 1. AI機能が動作しない
- APIキーが正しく設定されているか確認
- インターネット接続を確認
- プロバイダーのサービス状況を確認

```bash
# AI接続テスト
test-editor ai test-connection
```

#### 2. パフォーマンスが遅い
- 大きなファイルのインデックス作成を無効化
- 不要なプラグインを無効化
- メモリ使用量を確認

```bash
# パフォーマンス診断
test-editor diagnose performance
```

#### 3. 設定ファイルの破損
```bash
# 設定リセット
test-editor config reset

# 設定バックアップから復元
test-editor config restore backup-file.json
```

### ログファイルの確認

問題が発生した場合は、ログファイルを確認してください：

- **Windows**: `%APPDATA%\test-editor\logs\`
- **macOS**: `~/Library/Logs/test-editor/`
- **Linux**: `~/.local/share/test-editor/logs/`

## 次のステップ

基本的なセットアップが完了したら、以下のドキュメントを参照してください：

- [設定ガイド](configuration.md) - 詳細なカスタマイズ方法
- [基本的な使い方チュートリアル](../tutorials/basic-usage.md) - 実践的な使用例
- [API認証](api/authentication.md) - 外部連携の設定
- [トラブルシューティング](../guides/troubleshooting.md) - より詳細な問題解決

## サポート

さらなるサポートが必要な場合：
- [FAQ](../reference/faq.md)
- [Discord コミュニティ](https://discord.gg/test-editor)
- [GitHub Issues](https://github.com/wasborn14/test-editor/issues)