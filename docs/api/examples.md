# API使用例 - Test Editor

Test Editor APIの実用的な使用例を紹介します。一般的なユースケースから高度な統合まで、様々なシナリオでの実装方法を学べます。

## 基本的な使用例

### プロジェクト作成からファイル編集まで

```typescript
import { TestEditorClient } from '@test-editor/sdk';

const client = new TestEditorClient({
  token: process.env.TEST_EDITOR_TOKEN
});

async function createAndSetupProject() {
  // 1. プロジェクト作成
  const project = await client.projects.create({
    name: "My React App",
    description: "React TypeScript application",
    template: "react-typescript"
  });

  console.log(`プロジェクト作成完了: ${project.id}`);

  // 2. ファイル構造の確認
  const fileTree = await client.files.getTree(project.id);
  console.log("ファイル構造:", fileTree);

  // 3. 新しいコンポーネント作成
  await client.files.create(project.id, {
    path: "/src/components/UserProfile.tsx",
    content: `import React from 'react';

interface UserProfileProps {
  name: string;
  email: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ name, email }) => {
  return (
    <div className="user-profile">
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
};

export default UserProfile;`
  });

  // 4. ファイル更新
  const existingFile = await client.files.get(project.id, "/src/App.tsx");
  await client.files.update(project.id, {
    path: "/src/App.tsx",
    content: existingFile.content + `\nimport UserProfile from './components/UserProfile';`
  });

  return project;
}
```

### AI支援コード生成

```typescript
async function generateCodeWithAI(projectId: string) {
  // 1. コンテキストを含むコード生成
  const response = await client.ai.generate({
    prompt: "Create a custom hook for managing localStorage state",
    context: {
      projectId,
      language: "typescript",
      framework: "react"
    },
    options: {
      maxTokens: 1000,
      temperature: 0.7
    }
  });

  // 2. 生成されたコードをファイルに保存
  await client.files.create(projectId, {
    path: "/src/hooks/useLocalStorage.ts",
    content: response.generated.code
  });

  // 3. コード補完の例
  const completion = await client.ai.complete({
    context: {
      projectId,
      filePath: "/src/utils/helpers.ts",
      content: "export const formatDate = (date: Date) => {\n  return",
      cursorPosition: 54
    }
  });

  console.log("補完候補:", completion.suggestions);

  return response;
}
```

## 実用的なワークフロー

### 自動コードレビューシステム

```typescript
class AutoCodeReviewer {
  private client: TestEditorClient;

  constructor(token: string) {
    this.client = new TestEditorClient({ token });
  }

  async reviewPullRequest(projectId: string, changedFiles: string[]) {
    const reviews = await Promise.all(
      changedFiles.map(filePath => this.reviewFile(projectId, filePath))
    );

    return this.generateReviewSummary(reviews);
  }

  private async reviewFile(projectId: string, filePath: string) {
    // ファイル内容取得
    const file = await this.client.files.get(projectId, filePath);

    // AI分析実行
    const analysis = await this.client.ai.analyze({
      projectId,
      filePath,
      content: file.content,
      analysisType: "code_review"
    });

    return {
      file: filePath,
      issues: analysis.issues,
      suggestions: analysis.suggestions,
      score: analysis.qualityScore
    };
  }

  private generateReviewSummary(reviews: ReviewResult[]) {
    const totalIssues = reviews.reduce((sum, r) => sum + r.issues.length, 0);
    const averageScore = reviews.reduce((sum, r) => sum + r.score, 0) / reviews.length;

    return {
      summary: {
        filesReviewed: reviews.length,
        totalIssues,
        averageQualityScore: averageScore
      },
      details: reviews,
      recommendations: this.generateRecommendations(reviews)
    };
  }
}

// 使用例
const reviewer = new AutoCodeReviewer(process.env.TEST_EDITOR_TOKEN);
const review = await reviewer.reviewPullRequest("proj_123", [
  "/src/components/NewFeature.tsx",
  "/src/utils/validation.ts"
]);
```

### バッチファイル処理

```typescript
async function batchProcessFiles(projectId: string, pattern: string) {
  // 1. パターンに一致するファイルを検索
  const searchResults = await client.search.files(projectId, {
    pattern,
    limit: 100
  });

  console.log(`${searchResults.files.length}個のファイルが見つかりました`);

  // 2. バッチでファイル内容を取得
  const files = await client.files.getBatch(
    projectId,
    searchResults.files.map(f => f.path)
  );

  // 3. 各ファイルにAI分析を適用
  const results = await Promise.all(
    files.map(async (file) => {
      const analysis = await client.ai.analyze({
        projectId,
        filePath: file.path,
        content: file.content,
        analysisType: "optimization"
      });

      return {
        file: file.path,
        originalSize: file.content.length,
        suggestions: analysis.suggestions,
        potentialSavings: analysis.metrics?.potentialSavings || 0
      };
    })
  );

  // 4. レポート生成
  const report = {
    totalFiles: results.length,
    totalSavings: results.reduce((sum, r) => sum + r.potentialSavings, 0),
    topOptimizations: results
      .sort((a, b) => b.potentialSavings - a.potentialSavings)
      .slice(0, 10)
  };

  return report;
}
```

## Git統合例

### 自動コミットワークフロー

```typescript
class AutoCommitWorkflow {
  private client: TestEditorClient;

  constructor(token: string) {
    this.client = new TestEditorClient({ token });
  }

  async autoCommitChanges(projectId: string) {
    // 1. Git状態確認
    const status = await this.client.git.getStatus(projectId);

    if (status.clean) {
      console.log("変更がありません");
      return;
    }

    // 2. 変更されたファイルの分析
    const changedFiles = [...status.modified, ...status.staged];
    const analysis = await this.analyzeChanges(projectId, changedFiles);

    // 3. 自動コミットメッセージ生成
    const commitMessage = await this.generateCommitMessage(analysis);

    // 4. ファイルをステージング
    await this.client.git.add(projectId, changedFiles);

    // 5. コミット実行
    const commit = await this.client.git.commit(projectId, {
      message: commitMessage,
      author: {
        name: "Test Editor Bot",
        email: "bot@test-editor.dev"
      }
    });

    console.log(`コミット作成完了: ${commit.hash}`);
    return commit;
  }

  private async analyzeChanges(projectId: string, files: string[]) {
    const analyses = await Promise.all(
      files.map(async (filePath) => {
        const diff = await this.client.git.getDiff(projectId, filePath);

        return {
          file: filePath,
          additions: diff.additions,
          deletions: diff.deletions,
          type: this.categorizeChange(diff)
        };
      })
    );

    return analyses;
  }

  private categorizeChange(diff: GitDiff): string {
    if (diff.additions > diff.deletions * 2) return "feature";
    if (diff.deletions > diff.additions * 2) return "cleanup";
    if (diff.additions === 0 && diff.deletions > 0) return "removal";
    return "update";
  }

  private async generateCommitMessage(analysis: ChangeAnalysis[]): Promise<string> {
    const types = analysis.map(a => a.type);
    const mostCommonType = this.getMostCommon(types);

    const fileCount = analysis.length;
    const totalAdditions = analysis.reduce((sum, a) => sum + a.additions, 0);

    return `${mostCommonType}: Update ${fileCount} files (+${totalAdditions} lines)`;
  }
}
```

### ブランチ管理とマージ

```typescript
async function featureBranchWorkflow(projectId: string, featureName: string) {
  // 1. feature ブランチ作成
  const branch = await client.git.createBranch(projectId, {
    name: `feature/${featureName}`,
    from: "main"
  });

  // 2. ブランチ切り替え
  await client.git.checkout(projectId, branch.name);

  // 3. AI でコード生成
  const feature = await client.ai.generate({
    prompt: `Implement ${featureName} feature`,
    context: {
      projectId,
      branch: branch.name
    }
  });

  // 4. 生成されたコードを保存
  await client.files.create(projectId, {
    path: `/src/features/${featureName}.ts`,
    content: feature.generated.code
  });

  // 5. 変更をコミット
  await client.git.add(projectId, [`/src/features/${featureName}.ts`]);
  await client.git.commit(projectId, {
    message: `Add ${featureName} feature\n\n${feature.generated.explanation}`
  });

  // 6. main ブランチに戻ってマージ
  await client.git.checkout(projectId, "main");

  const mergeResult = await client.git.merge(projectId, {
    from: branch.name,
    strategy: "squash"
  });

  return mergeResult;
}
```

## 高度な統合例

### CI/CD統合

```typescript
// GitHub Actions ワークフロー例
class CICDIntegration {
  private client: TestEditorClient;

  constructor() {
    this.client = new TestEditorClient({
      token: process.env.TEST_EDITOR_TOKEN
    });
  }

  async runQualityChecks(projectId: string) {
    // 1. プロジェクト全体の分析
    const projectAnalysis = await this.client.analytics.getProjectStats(projectId);

    // 2. 品質ゲートのチェック
    const qualityGates = {
      testCoverage: 80,
      codeComplexity: "medium",
      maxIssues: 10
    };

    const results = {
      testCoverage: projectAnalysis.quality.testCoverage,
      codeComplexity: projectAnalysis.quality.codeComplexity,
      issuesFound: projectAnalysis.quality.issuesFound,
      passed: true,
      details: []
    };

    // 3. 各ゲートのチェック
    if (results.testCoverage < qualityGates.testCoverage) {
      results.passed = false;
      results.details.push(`テストカバレッジが不足: ${results.testCoverage}% < ${qualityGates.testCoverage}%`);
    }

    if (results.issuesFound > qualityGates.maxIssues) {
      results.passed = false;
      results.details.push(`問題が多すぎます: ${results.issuesFound} > ${qualityGates.maxIssues}`);
    }

    return results;
  }

  async deploymentChecks(projectId: string) {
    // デプロイ前のセキュリティチェック
    const securityScan = await this.client.ai.analyze({
      projectId,
      analysisType: "security_scan"
    });

    // パフォーマンステスト
    const performanceCheck = await this.client.ai.analyze({
      projectId,
      analysisType: "performance_analysis"
    });

    return {
      security: securityScan,
      performance: performanceCheck,
      ready: securityScan.riskLevel === "low" && performanceCheck.score > 80
    };
  }
}
```

### Slack Bot統合

```typescript
import { WebClient } from '@slack/web-api';

class TestEditorSlackBot {
  private testEditor: TestEditorClient;
  private slack: WebClient;

  constructor(testEditorToken: string, slackToken: string) {
    this.testEditor = new TestEditorClient({ token: testEditorToken });
    this.slack = new WebClient(slackToken);
  }

  async handleSlashCommand(command: string, text: string, userId: string) {
    switch (command) {
      case '/code-review':
        return await this.handleCodeReview(text, userId);
      case '/generate-code':
        return await this.handleCodeGeneration(text, userId);
      case '/project-status':
        return await this.handleProjectStatus(text, userId);
      default:
        return { text: "不明なコマンドです" };
    }
  }

  private async handleCodeReview(projectId: string, userId: string) {
    try {
      // プロジェクトの変更状況を確認
      const status = await this.testEditor.git.getStatus(projectId);

      if (status.clean) {
        return { text: "レビューする変更がありません" };
      }

      // 変更ファイルのレビュー実行
      const review = await this.performCodeReview(projectId, status.modified);

      // Slack に結果を投稿
      return {
        text: `コードレビュー完了 📝`,
        attachments: [{
          color: review.score > 80 ? "good" : review.score > 60 ? "warning" : "danger",
          fields: [
            {
              title: "レビューしたファイル数",
              value: review.filesCount.toString(),
              short: true
            },
            {
              title: "品質スコア",
              value: `${review.score}/100`,
              short: true
            },
            {
              title: "見つかった問題",
              value: review.issues.join("\\n"),
              short: false
            }
          ]
        }]
      };
    } catch (error) {
      return { text: `エラーが発生しました: ${error.message}` };
    }
  }

  private async handleCodeGeneration(prompt: string, userId: string) {
    try {
      const result = await this.testEditor.ai.generate({
        prompt,
        context: { language: "typescript" }
      });

      return {
        text: `コード生成完了 🎉`,
        attachments: [{
          title: "生成されたコード",
          text: "```typescript\\n" + result.generated.code + "\\n```",
          color: "good"
        }]
      };
    } catch (error) {
      return { text: `コード生成に失敗しました: ${error.message}` };
    }
  }
}

// Express.js でのSlack統合例
app.post('/slack/commands', async (req, res) => {
  const { command, text, user_id } = req.body;

  const bot = new TestEditorSlackBot(
    process.env.TEST_EDITOR_TOKEN,
    process.env.SLACK_TOKEN
  );

  const response = await bot.handleSlashCommand(command, text, user_id);
  res.json(response);
});
```

### Webhook活用例

```typescript
// Webhook受信サーバー
class WebhookHandler {
  private client: TestEditorClient;

  constructor(token: string) {
    this.client = new TestEditorClient({ token });
  }

  async handleWebhook(event: WebhookEvent) {
    switch (event.type) {
      case 'file.updated':
        await this.handleFileUpdate(event);
        break;
      case 'git.pushed':
        await this.handleGitPush(event);
        break;
      case 'ai.generation_completed':
        await this.handleAIGeneration(event);
        break;
    }
  }

  private async handleFileUpdate(event: FileUpdateEvent) {
    // ファイル更新時に自動でコード品質チェック
    const analysis = await this.client.ai.analyze({
      projectId: event.projectId,
      filePath: event.filePath,
      content: event.newContent,
      analysisType: "quality_check"
    });

    // 問題があれば通知
    if (analysis.issues.length > 0) {
      await this.sendNotification({
        type: "quality_alert",
        project: event.projectId,
        file: event.filePath,
        issues: analysis.issues
      });
    }
  }

  private async handleGitPush(event: GitPushEvent) {
    // プッシュ時に自動デプロイメントチェック
    const deployCheck = await this.client.analytics.getDeploymentReadiness(
      event.projectId
    );

    if (deployCheck.ready) {
      // 自動デプロイ実行
      await this.triggerDeployment(event.projectId, event.branch);
    }
  }
}

// Express.js での Webhook 受信
app.post('/webhooks/test-editor', async (req, res) => {
  const signature = req.headers['x-test-editor-signature'];
  const payload = req.body;

  // 署名検証
  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }

  const handler = new WebhookHandler(process.env.TEST_EDITOR_TOKEN);
  await handler.handleWebhook(payload);

  res.status(200).send('OK');
});
```

## エラーハンドリングとリトライ

```typescript
class RobustTestEditorClient {
  private client: TestEditorClient;
  private maxRetries = 3;

  constructor(token: string) {
    this.client = new TestEditorClient({
      token,
      timeout: 30000
    });
  }

  async safeRequest<T>(operation: () => Promise<T>): Promise<T> {
    let lastError: Error;

    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;

        // リトライすべきエラーかチェック
        if (!this.shouldRetry(error) || attempt === this.maxRetries) {
          throw error;
        }

        // 指数バックオフで待機
        const delay = Math.pow(2, attempt) * 1000;
        await this.sleep(delay);

        console.log(`リトライ ${attempt}/${this.maxRetries} (${delay}ms後)`);
      }
    }

    throw lastError;
  }

  private shouldRetry(error: any): boolean {
    // 一時的なエラーの場合のみリトライ
    const retryableErrors = [429, 500, 502, 503, 504];
    return retryableErrors.includes(error.status);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // ラッパーメソッド例
  async getProject(projectId: string) {
    return this.safeRequest(() => this.client.projects.get(projectId));
  }

  async generateCode(params: GenerateParams) {
    return this.safeRequest(() => this.client.ai.generate(params));
  }
}
```

## 関連ドキュメント

- [認証ガイド](authentication.md) - API認証の詳細
- [APIエンドポイント](endpoints.md) - 全エンドポイント一覧
- [レート制限](rate-limiting.md) - API使用制限について
- [SDK リファレンス](../../reference/sdk.md) - 各言語のSDK詳細