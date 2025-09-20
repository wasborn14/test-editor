# トラブルシューティング - Test Editor

Test Editorの一般的な問題と解決方法を網羅的に説明します。エラーメッセージ、パフォーマンス問題、設定トラブルなど、様々な状況での対処法を学べます。

## 一般的な問題と解決方法

### 起動・インストール問題

#### Test Editorが起動しない

**症状**: アプリケーションが起動しない、またはクラッシュする

**原因と解決方法**:

1. **Node.jsバージョンの確認**
```bash
# Node.jsバージョン確認
node --version

# バージョンが18未満の場合は更新
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS (Homebrew)
brew install node

# Windows (Chocolatey)
choco install nodejs
```

2. **権限問題の解決**
```bash
# Linux/macOS: 実行権限の付与
chmod +x /usr/local/bin/test-editor

# Windows: 管理者権限で実行
# PowerShellを管理者として実行してインストール
```

3. **依存関係の再インストール**
```bash
# グローバルキャッシュのクリア
npm cache clean --force

# Test Editorの再インストール
npm uninstall -g test-editor
npm install -g test-editor@latest
```

#### インストールエラー

**エラー例**: `EACCES: permission denied`

```bash
# npmの権限設定
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# .bashrc または .zshrc に追加
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 再インストール実行
npm install -g test-editor
```

**エラー例**: `gyp ERR! stack Error: ENOENT: no such file or directory`

```bash
# build-essential のインストール (Ubuntu/Debian)
sudo apt-get install build-essential

# Xcode Command Line Tools のインストール (macOS)
xcode-select --install

# Visual Studio Build Tools のインストール (Windows)
npm install -g windows-build-tools
```

### AI機能のトラブル

#### AI機能が応答しない

**診断手順**:

```bash
# 1. AI設定の確認
test-editor config get ai

# 2. API接続テスト
test-editor ai test-connection

# 3. APIキーの有効性確認
test-editor ai validate-key
```

**よくある原因と解決**:

1. **APIキーが無効**
```bash
# 新しいAPIキーを設定
test-editor config set ai.apiKey "your-new-api-key"

# プロバイダーの確認
test-editor config set ai.provider "openai"
```

2. **ネットワーク接続問題**
```bash
# プロキシ設定
test-editor config set network.proxy.host "proxy.company.com"
test-editor config set network.proxy.port 8080
test-editor config set network.proxy.auth.username "your-username"

# SSL証明書問題の回避（開発環境のみ）
test-editor config set network.ssl.rejectUnauthorized false
```

3. **レート制限に達している**
```bash
# レート制限状況の確認
test-editor ai quota

# プランのアップグレードまたは時間をおいて再試行
```

#### AI生成結果が期待と異なる

**改善方法**:

1. **プロンプトの最適化**
```typescript
// 悪い例
const result = await ai.generate("make a component");

// 良い例
const result = await ai.generate({
  prompt: "Create a React TypeScript component for user profile display",
  context: {
    framework: "react",
    language: "typescript",
    style: "functional-component"
  }
});
```

2. **設定パラメータの調整**
```bash
# 創造性を下げる（より確実な結果）
test-editor config set ai.temperature 0.3

# 出力トークン数を増やす
test-editor config set ai.maxTokens 2048
```

### ファイル・プロジェクト問題

#### ファイルが保存されない

**確認事項**:

1. **ディスク容量の確認**
```bash
# ディスク使用量確認
df -h

# Test Editor キャッシュサイズ確認
test-editor cache size
```

2. **ファイル権限の確認**
```bash
# プロジェクトディレクトリの権限確認
ls -la /path/to/project

# 権限修正
chmod -R 755 /path/to/project
chown -R $USER:$USER /path/to/project
```

3. **自動保存設定の確認**
```bash
# 自動保存が有効か確認
test-editor config get editor.autoSave

# 自動保存を有効化
test-editor config set editor.autoSave "afterDelay"
test-editor config set editor.autoSaveDelay 1000
```

#### プロジェクトが開けない

**診断手順**:

```bash
# 1. プロジェクト設定ファイルの確認
cat .test-editor/project.json

# 2. ファイルシステムの整合性チェック
test-editor project validate

# 3. インデックスの再構築
test-editor project reindex
```

**復旧方法**:

```bash
# プロジェクト設定の初期化
test-editor project init --force

# バックアップからの復元
test-editor project restore --backup latest
```

### Git統合問題

#### Git操作が失敗する

**一般的な解決方法**:

1. **Git設定の確認**
```bash
# Git設定確認
git config --global user.name
git config --global user.email

# Test Editor内のGit設定確認
test-editor config get git
```

2. **SSH鍵の設定**
```bash
# SSH鍵の生成
ssh-keygen -t ed25519 -C "your-email@example.com"

# SSH Agentに鍵を追加
ssh-add ~/.ssh/id_ed25519

# GitHub接続テスト
ssh -T git@github.com
```

3. **認証トークンの設定**
```bash
# GitHub Personal Access Token の設定
git config --global credential.helper store
echo "https://username:token@github.com" > ~/.git-credentials
```

#### マージコンフリクトの解決

```bash
# コンフリクトファイルの確認
test-editor git conflicts

# マージツールの起動
test-editor git mergetool

# または手動解決後にマーク
git add resolved-file.js
git commit -m "Resolve merge conflicts"
```

### パフォーマンス問題

#### 動作が重い・遅い

**診断手順**:

```bash
# 1. システムリソース使用量確認
test-editor monitor system

# 2. メモリ使用量分析
test-editor monitor memory --detail

# 3. プロファイリング実行
test-editor profile start
# 操作実行
test-editor profile stop
```

**最適化方法**:

1. **インデックス設定の調整**
```bash
# インデックス対象ファイルの制限
test-editor config set search.exclude '["node_modules/**", "*.min.js", "dist/**"]'

# インデックスサイズの制限
test-editor config set search.maxFileSize "1MB"
```

2. **キャッシュの最適化**
```bash
# キャッシュサイズの調整
test-editor config set performance.cache.maxSize "500MB"

# 古いキャッシュの削除
test-editor cache clean --older-than 7d
```

3. **プラグインの無効化**
```bash
# 重いプラグインの特定
test-editor plugins performance

# 不要なプラグインの無効化
test-editor plugins disable heavy-plugin-name
```

#### メモリリークの対処

```bash
# メモリ使用量の監視
test-editor monitor memory --watch

# ガベージコレクションの強制実行
test-editor gc force

# プロセスの再起動
test-editor restart
```

### ネットワーク・接続問題

#### APIエンドポイントに接続できない

**診断手順**:

```bash
# 1. 基本的な接続テスト
curl -I https://api.test-editor.dev/health

# 2. DNS解決の確認
nslookup api.test-editor.dev

# 3. Test Editor内での接続テスト
test-editor network test
```

**解決方法**:

1. **プロキシ設定**
```bash
# 企業環境でのプロキシ設定
test-editor config set network.proxy.host "proxy.company.com"
test-editor config set network.proxy.port 8080
test-editor config set network.proxy.auth.username "username"
test-editor config set network.proxy.auth.password "password"
```

2. **ファイアウォール設定**
```bash
# 必要なポートの開放確認
sudo ufw status
sudo ufw allow 443/tcp
sudo ufw allow 80/tcp
```

3. **SSL証明書問題**
```bash
# 証明書の確認
openssl s_client -connect api.test-editor.dev:443

# カスタム証明書の設定
test-editor config set network.ssl.caFile "/path/to/ca-cert.pem"
```

### 設定・環境問題

#### 設定が反映されない

**確認手順**:

```bash
# 1. 設定ファイルの場所確認
test-editor config path

# 2. 設定の有効性検証
test-editor config validate

# 3. 設定の優先順位確認
test-editor config debug
```

**解決方法**:

```bash
# 設定キャッシュのクリア
test-editor config cache clear

# 設定の再読み込み
test-editor config reload

# 設定のリセット
test-editor config reset --section editor
```

#### プラグインが動作しない

**トラブルシューティング**:

```bash
# 1. プラグインステータス確認
test-editor plugins status plugin-name

# 2. プラグインログ確認
test-editor plugins logs plugin-name

# 3. 依存関係確認
test-editor plugins deps plugin-name
```

**修復方法**:

```bash
# プラグインの再インストール
test-editor plugins uninstall plugin-name
test-editor plugins install plugin-name

# プラグイン設定のリセット
test-editor plugins config reset plugin-name
```

## デバッグとログ

### ログレベルの設定

```bash
# デバッグレベルでの実行
test-editor --log-level debug

# 特定コンポーネントのログ
test-editor --log-component ai,git

# ログファイルの出力
test-editor --log-file /tmp/test-editor.log
```

### 詳細なデバッグ情報

```bash
# システム情報の収集
test-editor diagnose system > system-info.txt

# エラーレポートの生成
test-editor diagnose report --include-logs

# パフォーマンストレースの取得
test-editor trace start
# 問題の再現
test-editor trace stop --output trace.json
```

### ログファイルの解析

```bash
# エラーログの検索
grep -n "ERROR" ~/.local/share/test-editor/logs/app.log

# 特定時間範囲のログ
test-editor logs --since "2024-03-20 10:00" --until "2024-03-20 11:00"

# ログの統計情報
test-editor logs stats
```

## エラーメッセージ別対処法

### 認証エラー

#### "Authentication failed"
```bash
# トークンの再取得
test-editor auth login

# APIキーの更新
test-editor config set ai.apiKey "new-api-key"
```

#### "Token expired"
```bash
# トークンのリフレッシュ
test-editor auth refresh

# 自動リフレッシュの有効化
test-editor config set auth.autoRefresh true
```

### ファイルシステムエラー

#### "ENOENT: no such file or directory"
```bash
# ファイルパスの確認
ls -la "$(dirname /path/to/file)"

# プロジェクトの再初期化
test-editor project init
```

#### "EACCES: permission denied"
```bash
# ディレクトリ権限の修正
sudo chown -R $USER:$USER /path/to/project
chmod -R 755 /path/to/project
```

### ネットワークエラー

#### "ECONNREFUSED"
```bash
# サービス状態の確認
test-editor service status

# ローカルサービスの再起動
test-editor service restart
```

#### "ETIMEDOUT"
```bash
# タイムアウト設定の調整
test-editor config set network.timeout 60000

# リトライ設定の調整
test-editor config set network.retries 3
```

## 緊急時の対処法

### データ復旧

#### プロジェクトファイルの復旧

```bash
# 自動バックアップからの復旧
test-editor backup list
test-editor backup restore --id backup-20240320-10-30

# Git履歴からの復旧
git log --oneline
git checkout HEAD~1 -- lost-file.js
```

#### 設定の復旧

```bash
# 設定のバックアップ
test-editor config export > config-backup.json

# バックアップからの復元
test-editor config import config-backup.json
```

### 完全なリセット

```bash
# WARNING: 全データが削除されます
test-editor reset --all --confirm

# 段階的なリセット
test-editor reset --config    # 設定のみ
test-editor reset --cache     # キャッシュのみ
test-editor reset --plugins   # プラグインのみ
```

## サポートとコミュニティ

### 公式サポート

1. **GitHub Issues**: [test-editor/issues](https://github.com/wasborn14/test-editor/issues)
2. **Discord**: [コミュニティチャンネル](https://discord.gg/test-editor)
3. **メール**: support@test-editor.dev

### 問題報告時の情報

```bash
# サポート用の情報収集
test-editor support-info > support-data.txt
```

**含めるべき情報**:
- OS・バージョン
- Test Editorバージョン
- エラーメッセージ
- 再現手順
- 関連ログ

### コミュニティリソース

- **フォーラム**: [community.test-editor.dev](https://community.test-editor.dev)
- **Stack Overflow**: タグ `test-editor`
- **Reddit**: r/TestEditor

## 予防的メンテナンス

### 定期的なチェック

```bash
# 週次メンテナンススクリプト
#!/bin/bash
test-editor cache clean --older-than 7d
test-editor backup create --auto
test-editor update check
test-editor diagnose health
```

### 設定の最適化

```bash
# パフォーマンス設定の見直し
test-editor config optimize

# セキュリティ設定の確認
test-editor config security-check
```

### 定期バックアップ

```bash
# 自動バックアップの設定
test-editor config set backup.enabled true
test-editor config set backup.interval "daily"
test-editor config set backup.retention 30
```

## 関連ドキュメント

- [インストールガイド](installation.md) - 詳細なインストール手順
- [設定ガイド](../docs/configuration.md) - 設定の詳細
- [パフォーマンス最適化](advanced/performance.md) - パフォーマンス改善
- [セキュリティガイド](advanced/security.md) - セキュリティ設定