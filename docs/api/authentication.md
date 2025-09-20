# API認証 - Test Editor

Test Editor APIを安全に利用するための認証システムについて説明します。JWT、OAuth2、APIキーなど複数の認証方式をサポートしています。

## 認証方式

### 1. JWT (JSON Web Token)

最も一般的な認証方式です。ユーザーログイン後にトークンを取得し、APIリクエストのヘッダーに含めて認証を行います。

#### トークン取得

```bash
# ログインしてJWTトークンを取得
curl -X POST https://api.test-editor.dev/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "your-password"
  }'
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "refresh_token_here",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  },
  "user": {
    "id": "uuid-here",
    "email": "user@example.com",
    "username": "username"
  }
}
```

#### APIリクエストでの使用

```bash
# 取得したトークンを使用してAPIを呼び出し
curl -X GET https://api.test-editor.dev/projects \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### トークンリフレッシュ

```bash
# リフレッシュトークンで新しいアクセストークンを取得
curl -X POST https://api.test-editor.dev/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "refresh_token_here"
  }'
```

### 2. OAuth2

GitHub、Google、Microsoftなどの外部プロバイダーを使用した認証です。

#### GitHub OAuth設定

```bash
# GitHubアプリケーション設定
Client ID: your-github-client-id
Client Secret: your-github-client-secret
Callback URL: https://your-app.com/auth/github/callback
```

#### OAuth フロー

```typescript
// OAuth認証開始
const authUrl = `https://api.test-editor.dev/auth/github?redirect_uri=${encodeURIComponent(callbackUrl)}`;
window.location.href = authUrl;

// コールバック処理
app.get('/auth/github/callback', async (req, res) => {
  const { code } = req.query;
  
  const response = await fetch('https://api.test-editor.dev/auth/github/callback', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ code })
  });
  
  const { accessToken } = await response.json();
  // トークンを保存して認証完了
});
```

### 3. APIキー認証

自動化やサーバー間通信に適した認証方式です。

#### APIキーの生成

```bash
# APIキーを生成
curl -X POST https://api.test-editor.dev/auth/api-keys \
  -H "Authorization: Bearer your-jwt-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My App Integration",
    "permissions": ["read:projects", "write:files"],
    "expiresAt": "2024-12-31T23:59:59Z"
  }'
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "id": "api-key-id",
    "key": "te_live_abc123def456...",
    "name": "My App Integration",
    "permissions": ["read:projects", "write:files"],
    "createdAt": "2024-03-20T10:00:00Z",
    "expiresAt": "2024-12-31T23:59:59Z"
  }
}
```

#### APIキーの使用

```bash
# APIキーを使用してリクエスト
curl -X GET https://api.test-editor.dev/projects \
  -H "X-API-Key: te_live_abc123def456..."
```

## 権限管理 (RBAC)

### 権限レベル

| レベル | 説明 | 操作例 |
|-------|------|-------|
| `read` | 読み取り専用 | プロジェクト一覧、ファイル内容表示 |
| `write` | 読み書き | ファイル編集、作成 |
| `admin` | 管理者権限 | プロジェクト削除、ユーザー管理 |
| `owner` | 所有者権限 | 全権限、他ユーザーへの権限付与 |

### 権限スコープ

```json
{
  "permissions": [
    "read:projects",
    "write:files",
    "admin:users",
    "read:analytics",
    "write:settings"
  ]
}
```

### 権限チェック

```bash
# 現在のユーザーの権限を確認
curl -X GET https://api.test-editor.dev/auth/permissions \
  -H "Authorization: Bearer your-token"
```

## セキュリティベストプラクティス

### 1. トークンの安全な保存

```typescript
// ブラウザ環境でのトークン保存
class TokenStorage {
  private readonly TOKEN_KEY = 'test-editor-token';
  
  saveToken(token: string): void {
    // HttpOnly Cookieまたはセキュアストレージを使用
    document.cookie = `${this.TOKEN_KEY}=${token}; Secure; HttpOnly; SameSite=Strict`;
  }
  
  getToken(): string | null {
    // セキュアな方法でトークンを取得
    return this.getSecureStorageItem(this.TOKEN_KEY);
  }
}
```

### 2. リクエストレート制限

```bash
# レート制限情報はレスポンスヘッダーで確認
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### 3. HTTPS強制

```nginx
# nginx設定例
server {
    listen 443 ssl http2;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # HSTS有効化
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

## トラブルシューティング

### 1. 認証エラーの診断

```bash
# 認証ステータスの確認
curl -X GET https://api.test-editor.dev/auth/status \
  -H "Authorization: Bearer your-token" \
  -v
```

**一般的なエラーと解決方法:**

#### 401 Unauthorized
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Token is invalid or expired",
    "details": {
      "expiredAt": "2024-03-20T10:00:00Z"
    }
  }
}
```

**解決方法:**
- トークンの有効期限を確認
- リフレッシュトークンで新しいトークンを取得
- 必要に応じて再ログイン

#### 403 Forbidden
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "User does not have required permissions",
    "required": ["write:files"],
    "current": ["read:projects"]
  }
}
```

**解決方法:**
- 管理者に権限昇格を依頼
- 適切な権限を持つAPIキーを使用

### 2. APIキー管理

```bash
# APIキー一覧表示
curl -X GET https://api.test-editor.dev/auth/api-keys \
  -H "Authorization: Bearer your-jwt-token"

# APIキーの無効化
curl -X DELETE https://api.test-editor.dev/auth/api-keys/key-id \
  -H "Authorization: Bearer your-jwt-token"
```

### 3. セッション管理

```bash
# アクティブセッション確認
curl -X GET https://api.test-editor.dev/auth/sessions \
  -H "Authorization: Bearer your-token"

# 他のセッションを無効化
curl -X POST https://api.test-editor.dev/auth/revoke-all-sessions \
  -H "Authorization: Bearer your-token"
```

## SDK使用例

### JavaScript/TypeScript

```typescript
import { TestEditorClient } from '@test-editor/sdk';

// JWT認証
const client = new TestEditorClient({
  baseURL: 'https://api.test-editor.dev',
  auth: {
    type: 'jwt',
    token: 'your-jwt-token'
  }
});

// APIキー認証
const apiClient = new TestEditorClient({
  baseURL: 'https://api.test-editor.dev',
  auth: {
    type: 'apiKey',
    key: 'te_live_abc123def456...'
  }
});

// プロジェクト一覧を取得
const projects = await client.projects.list();
```

### Python

```python
from test_editor import TestEditorClient

# JWT認証
client = TestEditorClient(
    base_url="https://api.test-editor.dev",
    auth_token="your-jwt-token"
)

# APIキー認証
api_client = TestEditorClient(
    base_url="https://api.test-editor.dev",
    api_key="te_live_abc123def456..."
)

# プロジェクト一覧を取得
projects = client.projects.list()
```

## セキュリティ監査

### ログ監視

```bash
# 認証ログの確認
curl -X GET https://api.test-editor.dev/auth/audit-logs \
  -H "Authorization: Bearer admin-token" \
  -G \
  -d "from=2024-03-01" \
  -d "to=2024-03-31" \
  -d "event_type=authentication"
```

### 異常検知

```json
{
  "alerts": [
    {
      "type": "SUSPICIOUS_LOGIN",
      "user_id": "user-123",
      "ip_address": "192.168.1.100",
      "location": "Tokyo, Japan",
      "timestamp": "2024-03-20T15:30:00Z",
      "risk_score": 0.8
    }
  ]
}
```

## 関連ドキュメント

- [APIエンドポイント](endpoints.md) - 全APIエンドポイントの詳細
- [レート制限](rate-limiting.md) - API使用制限について
- [セキュリティガイド](../advanced/security.md) - セキュリティ実装の詳細
- [SDK リファレンス](../reference/sdk.md) - 各言語のSDK使用方法