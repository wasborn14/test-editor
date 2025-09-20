# カスタムフック開発 - Test Editor

Test Editorの機能を拡張するカスタムフックの開発方法を説明します。イベントハンドリング、AI機能拡張、自動化ワークフローなど、様々なカスタマイズが可能です。

## フック システム概要

### フックの種類

Test Editorは以下のフックポイントを提供します：

```typescript
interface HookPoints {
  // ファイル操作
  'file.before-save': (file: FileInfo) => Promise<FileInfo | void>;
  'file.after-save': (file: FileInfo) => Promise<void>;
  'file.before-open': (path: string) => Promise<string | void>;
  'file.after-open': (file: FileInfo) => Promise<void>;

  // AI機能
  'ai.before-generate': (request: AIRequest) => Promise<AIRequest | void>;
  'ai.after-generate': (response: AIResponse) => Promise<AIResponse | void>;
  'ai.code-completion': (context: CompletionContext) => Promise<Completion[]>;

  // Git操作
  'git.before-commit': (files: string[]) => Promise<string[] | void>;
  'git.after-commit': (commit: CommitInfo) => Promise<void>;
  'git.pre-push': (branch: string) => Promise<boolean>;

  // プロジェクト
  'project.before-open': (path: string) => Promise<void>;
  'project.after-open': (project: ProjectInfo) => Promise<void>;
  'project.before-close': (project: ProjectInfo) => Promise<void>;

  // エディタ
  'editor.text-change': (change: TextChange) => Promise<void>;
  'editor.selection-change': (selection: Selection) => Promise<void>;
  'editor.cursor-move': (position: Position) => Promise<void>;
}
```

### フック登録の基本

```typescript
// フックプラグインの基本構造
import { TestEditorPlugin, HookContext } from '@test-editor/sdk';

export class MyCustomHook implements TestEditorPlugin {
  name = 'my-custom-hook';
  version = '1.0.0';

  async activate(context: HookContext): Promise<void> {
    // ファイル保存前の処理
    context.hooks.register('file.before-save', this.onBeforeSave.bind(this));

    // AI生成後の処理
    context.hooks.register('ai.after-generate', this.onAIGenerated.bind(this));
  }

  private async onBeforeSave(file: FileInfo): Promise<FileInfo> {
    // カスタム処理
    console.log(`Saving file: ${file.path}`);
    return file;
  }

  private async onAIGenerated(response: AIResponse): Promise<AIResponse> {
    // AI生成後の後処理
    response.metadata.customData = { processedAt: new Date() };
    return response;
  }
}
```

## 実用的なカスタムフック例

### 1. 自動フォーマッタフック

```typescript
class AutoFormatterHook implements TestEditorPlugin {
  name = 'auto-formatter';

  async activate(context: HookContext): Promise<void> {
    context.hooks.register('file.before-save', this.formatFile.bind(this));
  }

  private async formatFile(file: FileInfo): Promise<FileInfo> {
    const formatters = {
      '.ts': this.formatTypeScript,
      '.tsx': this.formatTypeScript,
      '.js': this.formatJavaScript,
      '.jsx': this.formatJavaScript,
      '.json': this.formatJSON,
      '.md': this.formatMarkdown
    };

    const extension = path.extname(file.path);
    const formatter = formatters[extension];

    if (formatter) {
      file.content = await formatter(file.content);
    }

    return file;
  }

  private async formatTypeScript(content: string): Promise<string> {
    const prettier = await import('prettier');
    return prettier.format(content, {
      parser: 'typescript',
      singleQuote: true,
      trailingComma: 'es5',
      printWidth: 100
    });
  }

  private async formatJSON(content: string): Promise<string> {
    try {
      const parsed = JSON.parse(content);
      return JSON.stringify(parsed, null, 2);
    } catch {
      return content; // 無効なJSONの場合はそのまま
    }
  }

  private async formatMarkdown(content: string): Promise<string> {
    // markdownの整形ロジック
    return content
      .replace(/\n{3,}/g, '\n\n') // 連続する空行を2行まで
      .replace(/\s+$/gm, '') // 行末の空白を削除
      .trim();
  }
}
```

### 2. AIコード品質チェッカー

```typescript
class AICodeQualityChecker implements TestEditorPlugin {
  name = 'ai-quality-checker';

  async activate(context: HookContext): Promise<void> {
    context.hooks.register('ai.after-generate', this.checkQuality.bind(this));
    context.hooks.register('file.after-save', this.analyzeCode.bind(this));
  }

  private async checkQuality(response: AIResponse): Promise<AIResponse> {
    if (response.type === 'code-generation') {
      const quality = await this.assessCodeQuality(response.content);

      if (quality.score < 0.7) {
        // 品質が低い場合は改善提案を追加
        const improvement = await this.generateImprovement(response.content);
        response.suggestions = [...(response.suggestions || []), improvement];
      }

      response.metadata.qualityScore = quality.score;
      response.metadata.qualityIssues = quality.issues;
    }

    return response;
  }

  private async assessCodeQuality(code: string): Promise<QualityAssessment> {
    // ESLintベースの品質チェック
    const linter = new ESLint({
      useEslintrc: false,
      baseConfig: {
        extends: ['eslint:recommended', '@typescript-eslint/recommended'],
        parser: '@typescript-eslint/parser',
        parserOptions: {
          ecmaVersion: 2022,
          sourceType: 'module'
        }
      }
    });

    const results = await linter.lintText(code);
    const issues = results[0]?.messages || [];

    // 複雑度計算
    const complexity = this.calculateComplexity(code);

    return {
      score: this.calculateOverallScore(issues, complexity),
      issues: issues.map(issue => ({
        line: issue.line,
        message: issue.message,
        severity: issue.severity
      })),
      complexity
    };
  }

  private calculateComplexity(code: string): number {
    // 簡単な複雑度計算（実際にはASTを使用）
    const complexityIndicators = [
      /if\s*\(/g,
      /else\s*{/g,
      /for\s*\(/g,
      /while\s*\(/g,
      /switch\s*\(/g,
      /catch\s*\(/g
    ];

    return complexityIndicators.reduce((total, pattern) => {
      const matches = code.match(pattern);
      return total + (matches ? matches.length : 0);
    }, 1);
  }

  private async generateImprovement(code: string): Promise<string> {
    // AIを使用して改善提案を生成
    const ai = TestEditorAPI.getAI();
    const response = await ai.generate({
      prompt: `
        Analyze this code and suggest specific improvements for:
        - Readability
        - Performance
        - Maintainability
        - Best practices

        Code:
        \`\`\`
        ${code}
        \`\`\`
      `,
      options: { temperature: 0.3 }
    });

    return response.content;
  }
}
```

### 3. スマートGitコミットフック

```typescript
class SmartGitCommitHook implements TestEditorPlugin {
  name = 'smart-git-commit';

  async activate(context: HookContext): Promise<void> {
    context.hooks.register('git.before-commit', this.prepareCommit.bind(this));
    context.hooks.register('git.after-commit', this.postCommitAnalysis.bind(this));
  }

  private async prepareCommit(files: string[]): Promise<string[]> {
    // 1. 自動フォーマット
    await this.formatFiles(files);

    // 2. リンターチェック
    const lintResults = await this.runLinter(files);
    if (lintResults.hasErrors) {
      throw new Error(`Lint errors found: ${lintResults.errors.join(', ')}`);
    }

    // 3. テスト実行
    const testResults = await this.runTests(files);
    if (!testResults.passed) {
      throw new Error(`Tests failed: ${testResults.failures.join(', ')}`);
    }

    // 4. AIによるコード分析
    const analysis = await this.analyzeChanges(files);
    if (analysis.hasSecurityIssues) {
      throw new Error(`Security issues found: ${analysis.securityIssues.join(', ')}`);
    }

    return files;
  }

  private async postCommitAnalysis(commit: CommitInfo): Promise<void> {
    // コミット後の分析とメトリクス収集
    const metrics = await this.collectMetrics(commit);

    // 技術債務の分析
    const debtAnalysis = await this.analyzeTechnicalDebt(commit.files);

    // レポート生成
    await this.generateCommitReport({
      commit,
      metrics,
      debtAnalysis
    });
  }

  private async analyzeChanges(files: string[]): Promise<ChangeAnalysis> {
    const ai = TestEditorAPI.getAI();
    const diffs = await Promise.all(
      files.map(file => this.getFileDiff(file))
    );

    const analysis = await ai.analyze({
      type: 'security-review',
      context: {
        files: diffs,
        patterns: ['sql-injection', 'xss', 'authentication', 'authorization']
      }
    });

    return {
      hasSecurityIssues: analysis.issues.length > 0,
      securityIssues: analysis.issues.map(issue => issue.description),
      suggestions: analysis.suggestions
    };
  }

  private async runTests(files: string[]): Promise<TestResults> {
    // 関連するテストファイルの特定
    const testFiles = files
      .map(file => this.findTestFile(file))
      .filter(Boolean);

    if (testFiles.length === 0) return { passed: true, failures: [] };

    // テスト実行
    const testRunner = new TestRunner();
    return await testRunner.run(testFiles);
  }
}
```

### 4. プロジェクト統計フック

```typescript
class ProjectStatisticsHook implements TestEditorPlugin {
  name = 'project-statistics';
  private stats: ProjectStats;

  async activate(context: HookContext): Promise<void> {
    this.stats = new ProjectStats();

    context.hooks.register('file.after-save', this.updateFileStats.bind(this));
    context.hooks.register('ai.after-generate', this.trackAIUsage.bind(this));
    context.hooks.register('project.after-open', this.initializeStats.bind(this));

    // 定期的な統計レポート
    setInterval(() => this.generateReport(), 24 * 60 * 60 * 1000); // 1日ごと
  }

  private async updateFileStats(file: FileInfo): Promise<void> {
    const language = this.detectLanguage(file.path);
    const lineCount = file.content.split('\n').length;
    const charCount = file.content.length;

    this.stats.updateFile({
      path: file.path,
      language,
      lineCount,
      charCount,
      lastModified: new Date()
    });

    // リアルタイム統計の更新
    await this.broadcastStats();
  }

  private async trackAIUsage(response: AIResponse): Promise<void> {
    this.stats.recordAIUsage({
      type: response.type,
      tokensUsed: response.metadata.tokensUsed,
      duration: response.metadata.duration,
      timestamp: new Date()
    });
  }

  private async generateReport(): Promise<void> {
    const report = {
      period: '24h',
      summary: {
        filesModified: this.stats.getFilesModified(),
        linesAdded: this.stats.getLinesAdded(),
        linesRemoved: this.stats.getLinesRemoved(),
        aiGenerations: this.stats.getAIGenerations(),
        commitCount: this.stats.getCommitCount()
      },
      productivity: this.calculateProductivityMetrics(),
      codeQuality: await this.analyzeCodeQuality(),
      recommendations: await this.generateRecommendations()
    };

    // レポートの保存と通知
    await this.saveReport(report);
    await this.notifyReport(report);
  }

  private calculateProductivityMetrics(): ProductivityMetrics {
    const stats = this.stats.getLast24Hours();

    return {
      filesPerHour: stats.filesModified / 24,
      linesPerHour: (stats.linesAdded + stats.linesRemoved) / 24,
      aiAssistanceRate: stats.aiGenerations / stats.filesModified,
      averageFileSize: stats.totalCharacters / stats.filesModified,
      activeLanguages: Object.keys(stats.languageBreakdown)
    };
  }
}
```

## 高度なフック技術

### 非同期フック処理

```typescript
class AsyncHookManager {
  private hooks: Map<string, Hook[]> = new Map();

  register(event: string, hook: Hook): void {
    if (!this.hooks.has(event)) {
      this.hooks.set(event, []);
    }
    this.hooks.get(event)!.push(hook);
  }

  async trigger(event: string, data: any): Promise<any> {
    const hooks = this.hooks.get(event) || [];

    // 並列実行（順序が重要でない場合）
    if (this.isParallelSafe(event)) {
      const results = await Promise.all(
        hooks.map(hook => this.executeHook(hook, data))
      );
      return this.mergeResults(results);
    }

    // 順次実行（順序が重要な場合）
    let result = data;
    for (const hook of hooks) {
      result = await this.executeHook(hook, result) || result;
    }
    return result;
  }

  private async executeHook(hook: Hook, data: any): Promise<any> {
    try {
      return await hook(data);
    } catch (error) {
      console.error(`Hook execution failed:`, error);
      // エラーハンドリング戦略
      if (this.isCriticalHook(hook)) {
        throw error;
      }
      return data; // 非クリティカルなフックのエラーは無視
    }
  }
}
```

### 条件付きフック実行

```typescript
class ConditionalHook implements TestEditorPlugin {
  name = 'conditional-hook';

  async activate(context: HookContext): Promise<void> {
    context.hooks.register('file.before-save', this.conditionalSave.bind(this));
  }

  private async conditionalSave(file: FileInfo): Promise<FileInfo> {
    // ファイルタイプに基づく条件分岐
    if (this.isProductionFile(file.path)) {
      await this.validateForProduction(file);
    }

    // プロジェクト設定に基づく条件分岐
    const project = await TestEditorAPI.getCurrentProject();
    if (project.settings.strictMode) {
      await this.strictValidation(file);
    }

    // ユーザー設定に基づく条件分岐
    const user = await TestEditorAPI.getCurrentUser();
    if (user.preferences.autoOptimize) {
      file.content = await this.optimizeCode(file.content);
    }

    return file;
  }

  private isProductionFile(path: string): boolean {
    const productionPaths = ['/src/', '/lib/', '/dist/'];
    return productionPaths.some(prodPath => path.includes(prodPath));
  }
}
```

### フック間のデータ共有

```typescript
class SharedDataHook implements TestEditorPlugin {
  name = 'shared-data-hook';
  private sharedContext: Map<string, any> = new Map();

  async activate(context: HookContext): Promise<void> {
    context.hooks.register('file.before-save', this.collectData.bind(this));
    context.hooks.register('file.after-save', this.useSharedData.bind(this));
    context.hooks.register('project.before-close', this.cleanup.bind(this));
  }

  private async collectData(file: FileInfo): Promise<FileInfo> {
    // データ収集
    const analysis = await this.analyzeFile(file);
    this.sharedContext.set(`analysis:${file.path}`, analysis);

    // グローバル統計の更新
    const globalStats = this.sharedContext.get('globalStats') || {};
    globalStats.totalFiles = (globalStats.totalFiles || 0) + 1;
    this.sharedContext.set('globalStats', globalStats);

    return file;
  }

  private async useSharedData(file: FileInfo): Promise<void> {
    // 共有データの利用
    const analysis = this.sharedContext.get(`analysis:${file.path}`);
    const globalStats = this.sharedContext.get('globalStats');

    if (analysis && globalStats) {
      await this.generateInsights(file, analysis, globalStats);
    }
  }

  private async cleanup(): Promise<void> {
    // プロジェクト終了時のクリーンアップ
    this.sharedContext.clear();
  }
}
```

## フックのテストとデバッグ

### フックのユニットテスト

```typescript
describe('AutoFormatterHook', () => {
  let hook: AutoFormatterHook;
  let mockContext: HookContext;

  beforeEach(() => {
    hook = new AutoFormatterHook();
    mockContext = {
      hooks: {
        register: jest.fn()
      }
    } as any;
  });

  it('should register file.before-save hook', async () => {
    await hook.activate(mockContext);

    expect(mockContext.hooks.register).toHaveBeenCalledWith(
      'file.before-save',
      expect.any(Function)
    );
  });

  it('should format TypeScript files', async () => {
    const file: FileInfo = {
      path: '/src/test.ts',
      content: 'const x=1;let y=2;',
      language: 'typescript'
    };

    const result = await hook.formatFile(file);

    expect(result.content).toContain('const x = 1;');
    expect(result.content).toContain('let y = 2;');
  });
});
```

### フックのデバッグ

```typescript
class DebugHook implements TestEditorPlugin {
  name = 'debug-hook';

  async activate(context: HookContext): Promise<void> {
    if (process.env.NODE_ENV === 'development') {
      this.enableDebugging(context);
    }
  }

  private enableDebugging(context: HookContext): void {
    const originalRegister = context.hooks.register.bind(context.hooks);

    context.hooks.register = (event: string, hook: Hook) => {
      const wrappedHook = this.wrapHookWithLogging(event, hook);
      return originalRegister(event, wrappedHook);
    };
  }

  private wrapHookWithLogging(event: string, hook: Hook): Hook {
    return async (data: any) => {
      const startTime = Date.now();
      console.log(`🔧 Hook [${event}] starting...`);

      try {
        const result = await hook(data);
        const duration = Date.now() - startTime;
        console.log(`✅ Hook [${event}] completed (${duration}ms)`);
        return result;
      } catch (error) {
        const duration = Date.now() - startTime;
        console.error(`❌ Hook [${event}] failed (${duration}ms):`, error);
        throw error;
      }
    };
  }
}
```

## フックのパッケージ化と配布

### package.jsonの設定

```json
{
  "name": "test-editor-hook-formatter",
  "version": "1.0.0",
  "description": "Auto-formatter hook for Test Editor",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "keywords": ["test-editor", "hook", "formatter"],
  "peerDependencies": {
    "@test-editor/sdk": "^2.0.0"
  },
  "files": [
    "dist/",
    "README.md"
  ],
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "prepublishOnly": "npm run build"
  }
}
```

### プラグインマニフェスト

```json
{
  "name": "auto-formatter",
  "displayName": "Auto Formatter",
  "description": "Automatically formats code on save",
  "version": "1.0.0",
  "author": "Your Name",
  "homepage": "https://github.com/username/test-editor-hook-formatter",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/test-editor-hook-formatter.git"
  },
  "engines": {
    "test-editor": "^2.0.0"
  },
  "categories": ["Formatters", "Other"],
  "activationEvents": [
    "onStartupFinished"
  ],
  "contributes": {
    "configuration": {
      "title": "Auto Formatter",
      "properties": {
        "autoFormatter.enabled": {
          "type": "boolean",
          "default": true,
          "description": "Enable auto formatting on save"
        }
      }
    }
  }
}
```

## 関連ドキュメント

- [パフォーマンス最適化](performance.md) - フックのパフォーマンス改善
- [セキュリティガイド](security.md) - セキュアなフック開発
- [監視ガイド](monitoring.md) - フックの監視とメトリクス
- [API リファレンス](../../docs/api/endpoints.md) - SDK API詳細