# インストールガイド - Test Editor

Test Editorの詳細なインストール手順です。様々な環境とプラットフォームでの導入方法を段階的に説明します。

## システム要件の確認

### ハードウェア要件

| 項目 | 最小要件 | 推奨要件 | 説明 |
|------|----------|----------|------|
| CPU | Intel i3 / AMD Ryzen 3 | Intel i7 / AMD Ryzen 7 | AI機能使用時はマルチコア推奨 |
| RAM | 4GB | 16GB | 大規模プロジェクト対応 |
| ストレージ | 2GB空き容量 | 10GB空き容量 | プラグインとキャッシュ用 |
| GPU | 不要 | CUDA対応GPU | AI機能の高速化 |

### ソフトウェア要件

#### 必須ソフトウェア
- **Node.js**: 18.0.0 以降（LTS版推奨）
- **Git**: 2.20.0 以降
- **OpenSSL**: 1.1.1 以降

#### オプションソフトウェア
- **Docker**: コンテナ環境使用時
- **Python**: AI機能拡張時（3.8以降）
- **Visual Studio Code**: 既存ワークフロー統合時

## プラットフォーム別インストール

### Windows 10/11

#### 方法1: インストーラー使用（推奨）

```powershell
# PowerShellを管理者権限で実行
# 1. Chocolateyのインストール（未インストールの場合）
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# 2. 依存関係のインストール
choco install nodejs git -y

# 3. Test Editorのインストール
npm install -g test-editor@latest

# 4. インストール確認
test-editor --version
```

#### 方法2: MSIパッケージ

1. [リリースページ](https://github.com/wasborn14/test-editor/releases)からMSIファイルをダウンロード
2. `TestEditor-{version}-win64.msi`を実行
3. インストールウィザードに従って設定

**カスタムインストールオプション:**
- インストール先ディレクトリ
- プラグインディレクトリ
- スタートメニュー統合
- ファイル関連付け

#### トラブルシューティング（Windows）

**実行ポリシーエラー**
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**パスが通らない場合**
```powershell
$env:PATH += ";$env:APPDATA\npm"
[Environment]::SetEnvironmentVariable("PATH", $env:PATH, [EnvironmentVariableTarget]::User)
```

### macOS

#### 方法1: Homebrew使用（推奨）

```bash
# 1. Homebrewのインストール（未インストールの場合）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. 依存関係のインストール
brew install node git

# 3. Test Editorのインストール
npm install -g test-editor@latest

# 4. インストール確認
test-editor --version
```

#### 方法2: DMGパッケージ

1. [リリースページ](https://github.com/wasborn14/test-editor/releases)からDMGファイルをダウンロード
2. `TestEditor-{version}-macos.dmg`をマウント
3. Test Editor.appをApplicationsフォルダにドラッグ

#### 方法3: ソースからビルド

```bash
# 1. リポジトリのクローン
git clone https://github.com/wasborn14/test-editor.git
cd test-editor

# 2. 依存関係のインストール
npm install

# 3. ビルド実行
npm run build:macos

# 4. インストール
npm run install:global
```

#### macOS固有の設定

**Gatekeeper設定**
```bash
# 初回実行時の警告を回避
sudo spctl --master-disable
```

**Rosetta 2対応（Apple Silicon）**
```bash
# Intel版Node.jsを使用する場合
arch -x86_64 npm install -g test-editor
```

### Linux (Ubuntu/Debian)

#### APTパッケージ使用

```bash
# 1. リポジトリキーの追加
curl -fsSL https://repo.test-editor.dev/gpg | sudo gpg --dearmor -o /usr/share/keyrings/test-editor.gpg

# 2. リポジトリの追加
echo "deb [signed-by=/usr/share/keyrings/test-editor.gpg] https://repo.test-editor.dev/apt stable main" | sudo tee /etc/apt/sources.list.d/test-editor.list

# 3. パッケージリストの更新
sudo apt update

# 4. Test Editorのインストール
sudo apt install test-editor

# 5. インストール確認
test-editor --version
```

#### Snapパッケージ使用

```bash
# 1. Snapdのインストール（未インストールの場合）
sudo apt install snapd

# 2. Test Editorのインストール
sudo snap install test-editor --classic

# 3. インストール確認
test-editor --version
```

#### 手動インストール

```bash
# 1. 依存関係のインストール
sudo apt update
sudo apt install nodejs npm git curl

# 2. npmからインストール
npm install -g test-editor@latest

# 3. パス設定
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Linux (CentOS/RHEL/Fedora)

#### DNF/YUMパッケージ使用

```bash
# 1. EPELリポジトリの有効化（CentOS/RHEL）
sudo dnf install epel-release

# 2. Node.jsリポジトリの追加
curl -fsSL https://rpm.nodesource.com/setup_lts.x | sudo bash -

# 3. 依存関係のインストール
sudo dnf install nodejs npm git

# 4. Test Editorのインストール
npm install -g test-editor@latest
```

### Arch Linux

```bash
# 1. 依存関係のインストール
sudo pacman -S nodejs npm git

# 2. AURからインストール（yay使用）
yay -S test-editor

# または npm から
npm install -g test-editor@latest
```

## Docker環境でのインストール

### 公式Dockerイメージ使用

```bash
# 1. イメージの取得
docker pull wasborn14/test-editor:latest

# 2. コンテナ起動
docker run -d \
  --name test-editor \
  -p 8080:8080 \
  -v $(pwd)/workspace:/workspace \
  -v test-editor-data:/data \
  wasborn14/test-editor:latest

# 3. ブラウザでアクセス
open http://localhost:8080
```

### Docker Compose使用

```yaml
# docker-compose.yml
version: '3.8'

services:
  test-editor:
    image: wasborn14/test-editor:latest
    ports:
      - "8080:8080"
    volumes:
      - ./workspace:/workspace
      - test-editor-data:/data
      - test-editor-config:/config
    environment:
      - NODE_ENV=production
      - AI_PROVIDER=openai
      - AI_API_KEY=${AI_API_KEY}
    restart: unless-stopped

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=testeditor
      - POSTGRES_USER=testeditor
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  test-editor-data:
  test-editor-config:
  postgres-data:
  redis-data:
```

起動コマンド:
```bash
# 環境変数設定
export AI_API_KEY="your-openai-api-key"
export DB_PASSWORD="secure-password"

# サービス起動
docker-compose up -d
```

## 開発環境セットアップ

### フロントエンド開発

```bash
# 1. リポジトリクローン
git clone https://github.com/wasborn14/test-editor.git
cd test-editor

# 2. フロントエンド依存関係インストール
cd client
npm install

# 3. 開発サーバー起動
npm run dev
```

### バックエンド開発

```bash
# 1. バックエンドディレクトリに移動
cd server

# 2. 依存関係インストール
npm install

# 3. 環境変数設定
cp .env.example .env
# .envファイルを編集

# 4. データベースセットアップ
npm run db:migrate
npm run db:seed

# 5. 開発サーバー起動
npm run dev
```

### 完全な開発環境

```bash
# ルートディレクトリで全体をビルド
npm run setup:dev

# 全サービスを起動
npm run dev:all
```

## 設定とカスタマイズ

### 初期設定ウィザード

```bash
# 初期設定を開始
test-editor init

# 対話形式で以下を設定:
# - ユーザー情報
# - AIプロバイダー
# - デフォルトワークスペース
# - Git設定
# - テーマ選択
```

### 設定ファイルのカスタマイズ

```bash
# 設定ファイルの場所を確認
test-editor config path

# エディタで設定を開く
test-editor config edit

# または手動で編集
vim ~/.config/test-editor/config.json
```

### プラグインのインストール

```bash
# 人気プラグインのインストール
test-editor plugins install eslint-integration
test-editor plugins install prettier-formatter
test-editor plugins install git-lens
test-editor plugins install ai-assistant-plus

# プラグイン一覧確認
test-editor plugins list

# プラグインの有効化
test-editor plugins enable eslint-integration
```

## インストール後の確認

### 動作テスト

```bash
# 1. バージョン確認
test-editor --version

# 2. ヘルプ表示
test-editor --help

# 3. 設定確認
test-editor config check

# 4. AI機能テスト
test-editor ai test

# 5. Git統合テスト
test-editor git status
```

### パフォーマンステスト

```bash
# システム情報確認
test-editor system info

# パフォーマンス診断
test-editor diagnose performance

# メモリ使用量確認
test-editor monitor memory
```

## アップデート手順

### 自動アップデート

```bash
# アップデート確認
test-editor update check

# アップデート実行
test-editor update install

# 特定バージョンへの更新
test-editor update install --version 2.1.0
```

### 手動アップデート

```bash
# npm経由でのアップデート
npm update -g test-editor

# 設定の移行確認
test-editor config migrate
```

## アンインストール

### 完全なアンインストール

```bash
# 1. アプリケーションのアンインストール
npm uninstall -g test-editor

# 2. 設定ファイルの削除
test-editor config clean --all

# 3. キャッシュクリア
rm -rf ~/.cache/test-editor

# 4. プラグインディレクトリの削除
rm -rf ~/.test-editor/plugins
```

### データ保持アンインストール

```bash
# アプリケーションのみ削除（設定とプロジェクトは保持）
npm uninstall -g test-editor
```

## トラブルシューティング

### よくある問題

#### 1. 権限エラー

```bash
# npm権限問題の解決
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

#### 2. ネットワークエラー

```bash
# プロキシ設定
npm config set proxy http://proxy.company.com:8080
npm config set https-proxy http://proxy.company.com:8080

# SSL証明書問題
npm config set strict-ssl false
```

#### 3. AI機能が動作しない

```bash
# APIキー確認
test-editor config get ai.apiKey

# 接続テスト
test-editor ai test-connection

# ログ確認
test-editor logs --level debug
```

#### 4. Git統合問題

```bash
# Git設定確認
git config --global user.name
git config --global user.email

# SSH設定確認
ssh -T git@github.com
```

### ログとデバッグ

```bash
# 詳細ログ出力
test-editor --verbose

# デバッグモード
test-editor --debug

# ログファイル確認
test-editor logs show

# 問題レポート生成
test-editor diagnose report
```

## 関連ドキュメント

- [はじめに](../docs/getting-started.md) - 基本的な使用方法
- [設定ガイド](../docs/configuration.md) - 詳細な設定方法
- [トラブルシューティング](troubleshooting.md) - 問題解決方法
- [アーキテクチャ](../docs/architecture.md) - システム構成の理解