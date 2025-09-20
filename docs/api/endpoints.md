# APIエンドポイント - Test Editor

Test Editor APIの全エンドポイントのリファレンスです。RESTful設計に従い、直感的で一貫性のあるAPIインターフェースを提供します。

## ベースURL

```
https://api.test-editor.dev/v1
```

## 認証

すべてのAPIエンドポイントは認証が必要です。詳細は[認証ガイド](authentication.md)を参照してください。

```bash
# JWT認証
Authorization: Bearer your-jwt-token

# API Key認証
X-API-Key: te_live_abc123def456...
```

## エラーハンドリング

### 標準エラーレスポンス

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {
      "field": "Additional error details"
    }
  },
  "requestId": "req_123456789"
}
```

### HTTPステータスコード

| コード | 意味 | 説明 |
|--------|------|------|
| 200 | OK | リクエスト成功 |
| 201 | Created | リソース作成成功 |
| 400 | Bad Request | リクエストパラメータエラー |
| 401 | Unauthorized | 認証エラー |
| 403 | Forbidden | 権限不足 |
| 404 | Not Found | リソースが見つからない |
| 429 | Too Many Requests | レート制限 |
| 500 | Internal Server Error | サーバーエラー |

## ユーザー管理

### ユーザー情報取得

```bash
GET /users/me
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "username": "john_doe",
    "email": "john@example.com",
    "profile": {
      "displayName": "John Doe",
      "avatar": "https://cdn.test-editor.dev/avatars/user_123.jpg",
      "timezone": "Asia/Tokyo"
    },
    "subscription": {
      "plan": "pro",
      "expiresAt": "2024-12-31T23:59:59Z"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "lastActiveAt": "2024-03-20T15:30:00Z"
  }
}
```

### ユーザー情報更新

```bash
PATCH /users/me
Content-Type: application/json

{
  "profile": {
    "displayName": "John Smith",
    "timezone": "US/Pacific"
  }
}
```

### ユーザー設定

```bash
# 設定取得
GET /users/me/settings

# 設定更新
PUT /users/me/settings
Content-Type: application/json

{
  "editor": {
    "theme": "dark",
    "fontSize": 16
  },
  "ai": {
    "provider": "openai",
    "autoComplete": true
  }
}
```

## プロジェクト管理

### プロジェクト一覧

```bash
GET /projects?page=1&limit=20&sort=updated_at&order=desc
```

**クエリパラメータ:**
- `page`: ページ番号（デフォルト: 1）
- `limit`: 1ページあたりの件数（デフォルト: 20、最大: 100）
- `sort`: ソート項目（name, created_at, updated_at）
- `order`: ソート順（asc, desc）
- `search`: プロジェクト名での検索

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "id": "proj_123",
        "name": "My Web App",
        "description": "React based web application",
        "owner": {
          "id": "user_123",
          "username": "john_doe"
        },
        "visibility": "private",
        "language": "javascript",
        "framework": "react",
        "filesCount": 45,
        "sizeBytes": 1048576,
        "lastActivity": "2024-03-20T15:30:00Z",
        "createdAt": "2024-01-01T00:00:00Z",
        "updatedAt": "2024-03-20T15:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

### プロジェクト作成

```bash
POST /projects
Content-Type: application/json

{
  "name": "New Project",
  "description": "Project description",
  "visibility": "private",
  "template": "react-typescript",
  "settings": {
    "aiEnabled": true,
    "autoSave": true
  }
}
```

### プロジェクト詳細取得

```bash
GET /projects/{project_id}
```

### プロジェクト更新

```bash
PATCH /projects/{project_id}
Content-Type: application/json

{
  "name": "Updated Project Name",
  "description": "Updated description",
  "settings": {
    "aiEnabled": false
  }
}
```

### プロジェクト削除

```bash
DELETE /projects/{project_id}
```

## ファイル管理

### ファイルツリー取得

```bash
GET /projects/{project_id}/files?path=/&recursive=true
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "tree": [
      {
        "type": "directory",
        "name": "src",
        "path": "/src",
        "children": [
          {
            "type": "file",
            "name": "index.js",
            "path": "/src/index.js",
            "size": 1024,
            "language": "javascript",
            "modifiedAt": "2024-03-20T15:30:00Z"
          }
        ]
      }
    ]
  }
}
```

### ファイル内容取得

```bash
GET /projects/{project_id}/files?path=/src/index.js
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "file": {
      "path": "/src/index.js",
      "content": "console.log('Hello, World!');",
      "language": "javascript",
      "encoding": "utf-8",
      "size": 1024,
      "hash": "sha256:abc123...",
      "modifiedAt": "2024-03-20T15:30:00Z"
    }
  }
}
```

### ファイル作成・更新

```bash
PUT /projects/{project_id}/files
Content-Type: application/json

{
  "path": "/src/new-file.js",
  "content": "// New file content",
  "encoding": "utf-8"
}
```

### ファイル削除

```bash
DELETE /projects/{project_id}/files?path=/src/old-file.js
```

### ファイル移動・リネーム

```bash
POST /projects/{project_id}/files/move
Content-Type: application/json

{
  "from": "/src/old-name.js",
  "to": "/src/new-name.js"
}
```

## AI機能

### コード生成

```bash
POST /ai/generate
Content-Type: application/json

{
  "prompt": "Create a React component for user profile",
  "context": {
    "projectId": "proj_123",
    "filePath": "/src/components/",
    "language": "javascript",
    "framework": "react"
  },
  "options": {
    "maxTokens": 1000,
    "temperature": 0.7
  }
}
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "generated": {
      "code": "import React from 'react';\n\nconst UserProfile = ({ user }) => {\n  return (\n    <div className=\"user-profile\">\n      <h2>{user.name}</h2>\n      <p>{user.email}</p>\n    </div>\n  );\n};\n\nexport default UserProfile;",
      "explanation": "Created a React functional component that displays user information",
      "suggestions": [
        "Add PropTypes for type checking",
        "Consider adding loading states"
      ]
    },
    "usage": {
      "tokensUsed": 150,
      "remainingTokens": 9850
    }
  }
}
```

### コード補完

```bash
POST /ai/complete
Content-Type: application/json

{
  "context": {
    "projectId": "proj_123",
    "filePath": "/src/utils.js",
    "content": "function calculateSum(a, b) {\n  return",
    "cursorPosition": 35
  },
  "options": {
    "maxSuggestions": 5
  }
}
```

### セマンティック検索

```bash
POST /ai/search
Content-Type: application/json

{
  "query": "user authentication functions",
  "projectId": "proj_123",
  "filters": {
    "fileTypes": ["js", "ts"],
    "excludePaths": ["/node_modules", "/dist"]
  },
  "limit": 10
}
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "file": "/src/auth/login.js",
        "line": 15,
        "content": "async function authenticateUser(email, password) {",
        "score": 0.95,
        "context": "User authentication logic with email and password validation"
      }
    ],
    "totalResults": 25,
    "searchTime": 120
  }
}
```

### バグ検出

```bash
POST /ai/analyze
Content-Type: application/json

{
  "projectId": "proj_123",
  "filePath": "/src/buggy-code.js",
  "content": "function divide(a, b) { return a / b; }",
  "analysisType": "bug_detection"
}
```

## Git統合

### リポジトリ状態

```bash
GET /projects/{project_id}/git/status
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "branch": "main",
    "staged": ["/src/new-file.js"],
    "modified": ["/src/existing-file.js"],
    "untracked": ["/temp/cache.json"],
    "ahead": 2,
    "behind": 0,
    "clean": false
  }
}
```

### コミット履歴

```bash
GET /projects/{project_id}/git/commits?limit=20&since=2024-01-01
```

### ファイルのコミット

```bash
POST /projects/{project_id}/git/commit
Content-Type: application/json

{
  "message": "Add new feature",
  "files": ["/src/feature.js", "/src/feature.test.js"],
  "author": {
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### ブランチ管理

```bash
# ブランチ一覧
GET /projects/{project_id}/git/branches

# ブランチ作成
POST /projects/{project_id}/git/branches
{
  "name": "feature/new-functionality",
  "from": "main"
}

# ブランチ切り替え
POST /projects/{project_id}/git/checkout
{
  "branch": "feature/new-functionality"
}
```

## 検索

### 全文検索

```bash
GET /projects/{project_id}/search?q=function&type=text&limit=50
```

**クエリパラメータ:**
- `q`: 検索クエリ
- `type`: 検索タイプ（text, semantic, regex）
- `files`: ファイルパターン（例: "*.js,*.ts"）
- `exclude`: 除外パターン
- `case_sensitive`: 大文字小文字区別
- `limit`: 結果数制限

### ファイル検索

```bash
GET /projects/{project_id}/search/files?name=component&extension=js
```

### 正規表現検索

```bash
GET /projects/{project_id}/search?q=function\s+\w+&type=regex
```

## 共同編集

### アクティブセッション

```bash
GET /projects/{project_id}/collaboration/sessions
```

### ファイルロック

```bash
POST /projects/{project_id}/collaboration/lock
Content-Type: application/json

{
  "filePath": "/src/shared-file.js",
  "lockType": "exclusive"
}
```

### リアルタイム編集

```bash
# WebSocket接続
wss://api.test-editor.dev/projects/{project_id}/collaboration/ws

# メッセージ例
{
  "type": "edit",
  "filePath": "/src/file.js",
  "operation": {
    "type": "insert",
    "position": 100,
    "text": "new code"
  },
  "userId": "user_123"
}
```

## 分析・レポート

### プロジェクト統計

```bash
GET /projects/{project_id}/analytics/stats
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalFiles": 156,
      "totalLines": 15420,
      "totalCharacters": 456789,
      "languages": {
        "javascript": 70,
        "css": 20,
        "html": 10
      }
    },
    "activity": {
      "commitsToday": 3,
      "commitsThisWeek": 15,
      "commitsThisMonth": 67
    },
    "quality": {
      "codeComplexity": "medium",
      "testCoverage": 85.5,
      "issuesFound": 12
    }
  }
}
```

### AI使用状況

```bash
GET /ai/usage?period=monthly&start=2024-01-01&end=2024-03-31
```

## ウェブフック

### ウェブフック一覧

```bash
GET /projects/{project_id}/webhooks
```

### ウェブフック作成

```bash
POST /projects/{project_id}/webhooks
Content-Type: application/json

{
  "url": "https://your-app.com/webhook",
  "events": ["file.created", "file.updated", "git.pushed"],
  "secret": "webhook-secret-key"
}
```

### ウェブフックイベント

利用可能なイベント:
- `file.created`
- `file.updated`
- `file.deleted`
- `git.pushed`
- `git.branch_created`
- `ai.generation_completed`
- `collaboration.user_joined`

## プラグイン

### インストール済みプラグイン

```bash
GET /projects/{project_id}/plugins
```

### プラグイン有効化

```bash
POST /projects/{project_id}/plugins/{plugin_id}/enable
```

### プラグイン設定

```bash
PUT /projects/{project_id}/plugins/{plugin_id}/config
Content-Type: application/json

{
  "settings": {
    "autoFormat": true,
    "lintOnSave": true
  }
}
```

## API制限とページネーション

### レート制限

```bash
# レスポンスヘッダーで制限情報を確認
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### ページネーション

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  },
  "links": {
    "first": "/api/v1/endpoint?page=1",
    "next": "/api/v1/endpoint?page=2",
    "last": "/api/v1/endpoint?page=8"
  }
}
```

## SDKとライブラリ

### 公式SDK

- **JavaScript/TypeScript**: `@test-editor/sdk`
- **Python**: `test-editor-python`
- **Go**: `github.com/test-editor/go-sdk`
- **Java**: `com.testeditor:java-sdk`

### 使用例

```typescript
import { TestEditorClient } from '@test-editor/sdk';

const client = new TestEditorClient({
  token: 'your-jwt-token'
});

// プロジェクト一覧
const projects = await client.projects.list();

// ファイル作成
await client.files.create('proj_123', {
  path: '/src/new-file.js',
  content: 'console.log("Hello");'
});

// AI コード生成
const result = await client.ai.generate({
  prompt: 'Create a login form',
  context: { framework: 'react' }
});
```

## 関連ドキュメント

- [認証ガイド](authentication.md) - 認証方法の詳細
- [レート制限](rate-limiting.md) - API使用制限について
- [API例](examples.md) - 実用的な使用例
- [SDK リファレンス](../../reference/sdk.md) - 各言語のSDK詳細