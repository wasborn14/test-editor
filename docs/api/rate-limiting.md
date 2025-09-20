# レート制限 - Test Editor API

Test Editor APIは、サービスの安定性とパフォーマンスを維持するため、レート制限を実装しています。このドキュメントでは、制限の詳細と適切な対処方法について説明します。

## 制限概要

### プラン別制限

| プラン | リクエスト/分 | リクエスト/時間 | リクエスト/日 | AI機能制限 |
|-------|---------------|----------------|---------------|------------|
| Free | 60 | 1,000 | 10,000 | 100回/日 |
| Pro | 300 | 5,000 | 50,000 | 1,000回/日 |
| Team | 600 | 15,000 | 200,000 | 5,000回/日 |
| Enterprise | カスタム | カスタム | カスタム | カスタム |

### エンドポイント別制限

```json
{
  "limits": {
    "authentication": {
      "/auth/login": "5/minute",
      "/auth/register": "3/minute",
      "/auth/reset-password": "3/minute"
    },
    "ai_functions": {
      "/ai/generate": "10/minute",
      "/ai/complete": "30/minute",
      "/ai/search": "20/minute"
    },
    "file_operations": {
      "/projects/*/files": "100/minute",
      "/projects/*/files/upload": "20/minute"
    },
    "general": {
      "default": "1000/hour"
    }
  }
}
```

## レート制限の確認

### レスポンスヘッダー

すべてのAPIレスポンスには、現在の制限状況を示すヘッダーが含まれます：

```bash
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
X-RateLimit-Window: 3600
X-RateLimit-Type: sliding_window
```

#### ヘッダー詳細

| ヘッダー | 説明 |
|----------|------|
| `X-RateLimit-Limit` | 制限時間内の最大リクエスト数 |
| `X-RateLimit-Remaining` | 残りリクエスト数 |
| `X-RateLimit-Reset` | 制限リセット時刻（UNIX timestamp） |
| `X-RateLimit-Window` | 制限ウィンドウ（秒） |
| `X-RateLimit-Type` | 制限タイプ（fixed_window, sliding_window） |

### 制限状況API

```bash
GET /rate-limits/status
Authorization: Bearer your-token
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "limits": [
      {
        "endpoint": "/projects",
        "limit": 1000,
        "remaining": 856,
        "resetAt": "2024-03-20T16:00:00Z",
        "windowSize": 3600
      },
      {
        "endpoint": "/ai/generate",
        "limit": 10,
        "remaining": 7,
        "resetAt": "2024-03-20T15:35:00Z",
        "windowSize": 60
      }
    ],
    "subscription": {
      "plan": "pro",
      "upgradeAvailable": true
    }
  }
}
```

## 制限超過時の対応

### 429エラーレスポンス

制限を超過した場合、HTTP 429ステータスが返されます：

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "details": {
      "limit": 1000,
      "window": 3600,
      "resetAt": "2024-03-20T16:00:00Z",
      "retryAfter": 180
    }
  }
}
```

### 自動リトライ実装

```typescript
class TestEditorClient {
  private async makeRequest(
    endpoint: string,
    options: RequestOptions,
    retryCount = 0
  ): Promise<Response> {
    try {
      const response = await fetch(endpoint, options);

      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        const delay = retryAfter ? parseInt(retryAfter) * 1000 : this.calculateBackoff(retryCount);

        if (retryCount < this.maxRetries) {
          await this.sleep(delay);
          return this.makeRequest(endpoint, options, retryCount + 1);
        }
      }

      return response;
    } catch (error) {
      throw error;
    }
  }

  private calculateBackoff(attempt: number): number {
    // 指数バックオフ: 2^attempt * 1000ms
    return Math.min(Math.pow(2, attempt) * 1000, 30000);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### バッチ処理とキューイング

```typescript
class RequestQueue {
  private queue: Array<QueuedRequest> = [];
  private processing = false;

  async addRequest(request: QueuedRequest): Promise<any> {
    return new Promise((resolve, reject) => {
      this.queue.push({
        ...request,
        resolve,
        reject
      });

      if (!this.processing) {
        this.processQueue();
      }
    });
  }

  private async processQueue(): Promise<void> {
    this.processing = true;

    while (this.queue.length > 0) {
      const request = this.queue.shift()!;

      try {
        const result = await this.executeRequest(request);
        request.resolve(result);
      } catch (error) {
        request.reject(error);
      }

      // レート制限を考慮した遅延
      await this.sleep(this.calculateDelay());
    }

    this.processing = false;
  }

  private calculateDelay(): number {
    // プランに応じた適切な遅延を計算
    const requestsPerSecond = this.getRateLimit() / 60;
    return 1000 / requestsPerSecond;
  }
}
```

## 制限回避のベストプラクティス

### 1. リクエスト最適化

```typescript
// 悪い例: 個別にファイルを取得
for (const file of files) {
  const content = await client.files.get(projectId, file.path);
  processFile(content);
}

// 良い例: バッチでファイルを取得
const contents = await client.files.getBatch(projectId, filePaths);
contents.forEach(processFile);
```

### 2. キャッシング活用

```typescript
class CachedClient {
  private cache = new Map<string, CachedResponse>();

  async get(endpoint: string): Promise<any> {
    const cached = this.cache.get(endpoint);

    if (cached && !this.isExpired(cached)) {
      return cached.data;
    }

    const response = await this.makeRequest(endpoint);
    this.cache.set(endpoint, {
      data: response,
      timestamp: Date.now(),
      ttl: this.getTTL(endpoint)
    });

    return response;
  }

  private isExpired(cached: CachedResponse): boolean {
    return Date.now() - cached.timestamp > cached.ttl;
  }

  private getTTL(endpoint: string): number {
    // エンドポイントに応じたTTLを設定
    if (endpoint.includes('/files/')) return 30000;  // 30秒
    if (endpoint.includes('/projects')) return 300000; // 5分
    return 60000; // デフォルト1分
  }
}
```

### 3. WebSocket利用

```typescript
// リアルタイム更新が必要な場合はWebSocketを使用
class RealtimeClient {
  private ws: WebSocket;

  connect(projectId: string): void {
    this.ws = new WebSocket(`wss://api.test-editor.dev/projects/${projectId}/ws`);

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleRealtimeUpdate(data);
    };
  }

  // ポーリングの代わりにWebSocketで更新を受信
  private handleRealtimeUpdate(data: any): void {
    switch (data.type) {
      case 'file_updated':
        this.updateFileCache(data.file);
        break;
      case 'project_settings_changed':
        this.invalidateProjectCache(data.projectId);
        break;
    }
  }
}
```

## 特別な制限

### AI機能の制限

AI機能には追加の制限があります：

```json
{
  "ai_limits": {
    "generation": {
      "requests_per_day": 100,
      "tokens_per_request": 4096,
      "max_concurrent": 3
    },
    "completion": {
      "requests_per_minute": 30,
      "max_concurrent": 5
    },
    "analysis": {
      "files_per_request": 10,
      "max_file_size": "1MB"
    }
  }
}
```

### ファイルアップロード制限

```json
{
  "upload_limits": {
    "max_file_size": "10MB",
    "max_files_per_request": 20,
    "allowed_types": [
      "text/*",
      "application/json",
      "application/javascript",
      "image/*"
    ],
    "requests_per_minute": 20
  }
}
```

## 企業向けカスタム制限

### 専用制限の設定

```bash
POST /admin/rate-limits
Authorization: Bearer admin-token
Content-Type: application/json

{
  "userId": "user_123",
  "customLimits": {
    "/ai/generate": "50/minute",
    "/projects/*/files": "500/minute"
  },
  "validUntil": "2024-12-31T23:59:59Z"
}
```

### IP許可リスト

```bash
POST /admin/ip-whitelist
Authorization: Bearer admin-token
Content-Type: application/json

{
  "ipAddresses": [
    "192.168.1.0/24",
    "10.0.0.0/8"
  ],
  "exemptFromRateLimit": true,
  "description": "Company office network"
}
```

## 監視とアラート

### 制限監視

```typescript
class RateLimitMonitor {
  private checkLimits(): void {
    setInterval(async () => {
      const status = await this.client.rateLimits.getStatus();

      status.limits.forEach(limit => {
        const usagePercent = (limit.limit - limit.remaining) / limit.limit * 100;

        if (usagePercent > 80) {
          this.sendAlert(`Rate limit warning: ${limit.endpoint} at ${usagePercent}% usage`);
        }
      });
    }, 60000); // 1分ごとにチェック
  }

  private sendAlert(message: string): void {
    // Slack、メール、またはその他の通知システムに送信
    console.warn(message);
  }
}
```

### ダッシュボード

```bash
# 使用量ダッシュボードAPI
GET /analytics/rate-limits?period=last_24h
Authorization: Bearer your-token
```

**レスポンス例:**
```json
{
  "success": true,
  "data": {
    "period": "last_24h",
    "totalRequests": 15420,
    "limitExceeded": 23,
    "averageResponseTime": 145,
    "topEndpoints": [
      {
        "endpoint": "/projects/*/files",
        "requests": 8234,
        "percentage": 53.4
      }
    ],
    "hourlyBreakdown": [
      {
        "hour": "2024-03-20T14:00:00Z",
        "requests": 642,
        "throttled": 3
      }
    ]
  }
}
```

## SDK での制限対応

### JavaScript SDK

```typescript
import { TestEditorClient, RateLimitHandler } from '@test-editor/sdk';

const client = new TestEditorClient({
  token: 'your-token',
  rateLimitHandler: new RateLimitHandler({
    maxRetries: 3,
    backoffStrategy: 'exponential',
    enableQueue: true,
    queueMaxSize: 100
  })
});

// 自動的にレート制限を処理
const projects = await client.projects.list();
```

### Python SDK

```python
from test_editor import TestEditorClient
from test_editor.rate_limiting import ExponentialBackoff

client = TestEditorClient(
    token="your-token",
    rate_limit_strategy=ExponentialBackoff(
        max_retries=3,
        base_delay=1.0
    )
)

# レート制限を自動処理
projects = client.projects.list()
```

## トラブルシューティング

### よくある問題

#### 1. 突然の制限超過
```bash
# 現在の使用量を確認
curl -X GET https://api.test-editor.dev/rate-limits/status \
  -H "Authorization: Bearer your-token"

# 最近のリクエスト履歴を確認
curl -X GET https://api.test-editor.dev/analytics/requests?period=last_hour \
  -H "Authorization: Bearer your-token"
```

#### 2. 制限リセット時刻の確認
```javascript
const resetTime = new Date(parseInt(response.headers['X-RateLimit-Reset']) * 1000);
console.log(`制限は ${resetTime.toLocaleString()} にリセットされます`);
```

#### 3. プラン制限の確認
```bash
# サブスクリプション情報を確認
curl -X GET https://api.test-editor.dev/users/me/subscription \
  -H "Authorization: Bearer your-token"
```

## 関連ドキュメント

- [認証ガイド](authentication.md) - 認証方法とセキュリティ
- [APIエンドポイント](endpoints.md) - 全エンドポイント一覧
- [パフォーマンス最適化](../../guides/advanced/performance.md) - アプリケーション最適化
- [SDK リファレンス](../../reference/sdk.md) - 各言語のSDK詳細