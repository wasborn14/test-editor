# 設定ガイド - Test Editor

Test Editorの動作をカスタマイズするための包括的な設定ガイドです。設定ファイル、環境変数、コマンドラインオプションについて詳しく説明します。

## 設定ファイルの場所

Test Editorは以下の場所で設定ファイルを検索します：

### プラットフォーム別設定ディレクトリ

- **Windows**: `%APPDATA%\test-editor\config\`
- **macOS**: `~/Library/Preferences/test-editor/`
- **Linux**: `~/.config/test-editor/`

### 設定ファイルの優先順位

1. `./test-editor.config.json` (プロジェクトルート)
2. `~/.test-editor/config.json` (ユーザーホーム)
3. `{CONFIG_DIR}/test-editor.json` (システム設定)

## 主要設定項目

### エディタ設定

```json
{
  "editor": {
    "theme": "dark",
    "fontSize": 14,
    "fontFamily": "Fira Code, Consolas, monospace",
    "tabSize": 2,
    "insertSpaces": true,
    "wordWrap": "bounded",
    "lineNumbers": "on",
    "minimap": {
      "enabled": true,
      "maxColumn": 120
    },
    "autoSave": "afterDelay",
    "autoSaveDelay": 1000
  }
}
```

#### エディタオプション詳細

| 設定項目 | 型 | デフォルト | 説明 |
|---------|----|-----------|----- |
| `theme` | string | "dark" | テーマ（dark, light, auto） |
| `fontSize` | number | 14 | フォントサイズ（8-72） |
| `fontFamily` | string | "Fira Code" | フォントファミリー |
| `tabSize` | number | 2 | タブサイズ（1-8） |
| `insertSpaces` | boolean | true | スペース挿入（タブの代わり） |
| `wordWrap` | string | "bounded" | 折り返し（off, on, bounded） |
| `lineNumbers` | string | "on" | 行番号表示（on, off, relative） |

### AI機能設定

```json
{
  "ai": {
    "enabled": true,
    "provider": "openai",
    "model": "gpt-4-turbo",
    "apiKey": "${AI_API_KEY}",
    "timeout": 30000,
    "maxTokens": 4096,
    "temperature": 0.7,
    "features": {
      "codeCompletion": true,
      "codeGeneration": true,
      "refactoring": true,
      "bugDetection": true,
      "documentation": true
    },
    "customPrompts": {
      "codeReview": "以下のコードをレビューして、改善点を提案してください：",
      "optimization": "このコードのパフォーマンスを最適化してください："
    }
  }
}
```

#### 対応AIプロバイダー

| プロバイダー | モデル例 | 機能 |
|-------------|---------|------|
| `openai` | gpt-4-turbo, gpt-3.5-turbo | コード生成、補完、分析 |
| `anthropic` | claude-3-opus, claude-3-sonnet | 長文解析、推論 |
| `cohere` | command-r-plus | 多言語対応 |
| `local` | llama-3, code-llama | ローカル実行 |

### Git統合設定

```json
{
  "git": {
    "enabled": true,
    "autoCommit": false,
    "autoCommitMessage": "Auto-commit by Test Editor",
    "signing": {
      "enabled": false,
      "key": "your-gpg-key-id"
    },
    "hooks": {
      "preCommit": ["npm run lint", "npm run test"],
      "postCommit": ["npm run build"]
    },
    "ignorePatterns": [
      "node_modules/",
      "*.log",
      ".env*"
    ]
  }
}
```

### 検索・インデックス設定

```json
{
  "search": {
    "semanticSearch": {
      "enabled": true,
      "model": "sentence-transformers/all-MiniLM-L6-v2",
      "indexPath": ".test-editor/index",
      "chunkSize": 512,
      "overlap": 50
    },
    "textSearch": {
      "caseSensitive": false,
      "regex": true,
      "maxResults": 100
    },
    "exclude": [
      "node_modules/**",
      "*.min.js",
      "dist/**",
      ".git/**"
    ]
  }
}
```

### ネットワーク・セキュリティ設定

```json
{
  "network": {
    "proxy": {
      "enabled": false,
      "host": "proxy.company.com",
      "port": 8080,
      "auth": {
        "username": "${PROXY_USER}",
        "password": "${PROXY_PASS}"
      }
    },
    "ssl": {
      "rejectUnauthorized": true,
      "caFile": "./certs/ca.pem"
    }
  },
  "security": {
    "allowUnsafeEval": false,
    "csp": {
      "enabled": true,
      "directives": {
        "default-src": "'self'",
        "script-src": "'self' 'unsafe-inline'",
        "connect-src": "'self' https:"
      }
    }
  }
}
```

## 環境変数

### 必須環境変数

```bash
# AI API キー
export AI_API_KEY="your-api-key-here"

# データベース接続
export DATABASE_URL="postgresql://user:pass@localhost:5432/testdb"

# ログレベル
export LOG_LEVEL="info"  # debug, info, warn, error
```

### オプション環境変数

```bash
# プロキシ設定
export HTTP_PROXY="http://proxy.company.com:8080"
export HTTPS_PROXY="http://proxy.company.com:8080"

# カスタム設定ディレクトリ
export TEST_EDITOR_CONFIG_DIR="/path/to/custom/config"

# 一時ディレクトリ
export TEST_EDITOR_TEMP_DIR="/tmp/test-editor"

# プラグインディレクトリ
export TEST_EDITOR_PLUGINS_DIR="./plugins"
```

## コマンドライン設定

### 設定の表示・変更

```bash
# 全設定の表示
test-editor config list

# 特定の設定を表示
test-editor config get editor.theme

# 設定の変更
test-editor config set editor.theme "light"

# 設定の削除
test-editor config unset editor.customSetting

# 設定ファイルの場所を表示
test-editor config path
```

### 設定のインポート・エクスポート

```bash
# 設定のエクスポート
test-editor config export > my-config.json

# 設定のインポート
test-editor config import my-config.json

# 設定のバックアップ
test-editor config backup

# バックアップからの復元
test-editor config restore backup-20240320.json
```

## テーマとカスタマイズ

### カスタムテーマの作成

```json
{
  "themes": {
    "myCustomTheme": {
      "base": "dark",
      "colors": {
        "background": "#1e1e1e",
        "foreground": "#d4d4d4",
        "accent": "#007acc",
        "error": "#f44747",
        "warning": "#ffcc02",
        "success": "#89d185"
      },
      "syntax": {
        "keyword": "#569cd6",
        "string": "#ce9178",
        "comment": "#6a9955",
        "function": "#dcdcaa",
        "variable": "#9cdcfe"
      }
    }
  }
}
```

### カスタムフォントの設定

```bash
# システムフォントの確認
test-editor fonts list

# カスタムフォントの追加
test-editor fonts install ./fonts/MyFont.ttf

# フォントの適用
test-editor config set editor.fontFamily "MyFont, Fira Code, monospace"
```

## プラグイン設定

### プラグイン固有設定

```json
{
  "plugins": {
    "eslint-integration": {
      "enabled": true,
      "configFile": ".eslintrc.json",
      "autoFix": true,
      "rulesDir": "./custom-rules"
    },
    "prettier-formatter": {
      "enabled": true,
      "formatOnSave": true,
      "configFile": ".prettierrc"
    },
    "git-lens": {
      "enabled": true,
      "showBlame": true,
      "showChanges": true
    }
  }
}
```

## 設定の検証

### 設定ファイルの妥当性チェック

```bash
# 設定ファイルの検証
test-editor config validate

# 設定の修復
test-editor config repair

# デフォルト設定へのリセット
test-editor config reset --confirm
```

### 設定テンプレート

```bash
# 利用可能なテンプレート一覧
test-editor config templates

# テンプレートの適用
test-editor config apply-template web-development

# カスタムテンプレートの作成
test-editor config create-template my-template
```

## パフォーマンス最適化

### メモリ使用量の最適化

```json
{
  "performance": {
    "maxMemoryUsage": "2GB",
    "garbageCollection": {
      "enabled": true,
      "interval": 300000
    },
    "indexing": {
      "batchSize": 1000,
      "throttle": 100
    },
    "cache": {
      "maxSize": "500MB",
      "ttl": 3600000
    }
  }
}
```

## トラブルシューティング

### 設定関連の問題

1. **設定が適用されない**
   ```bash
   # 設定ファイルの優先順位を確認
   test-editor config debug
   ```

2. **設定ファイルが見つからない**
   ```bash
   # 設定ディレクトリの作成
   test-editor config init
   ```

3. **環境変数が認識されない**
   ```bash
   # 環境変数の確認
   test-editor env check
   ```

## 関連ドキュメント

- [アーキテクチャ](architecture.md) - システム構成の詳細
- [API認証](api/authentication.md) - API設定の詳細
- [パフォーマンス最適化](../guides/advanced/performance.md) - 詳細な最適化手法
- [セキュリティ](../guides/advanced/security.md) - セキュリティ設定の詳細