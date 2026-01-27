# Copilot Instructions

このファイルには、GitHub Copilotや他のLLMアシスタントがこのプロジェクトでコードを生成する際に従うべき指示が含まれています。

## 地図の作成

### ライブラリ

地図を作成する際は、必ず **Leaflet** を使用してください。

- 公式サイト: https://leafletjs.com/
- Leafletは軽量でモバイルフレンドリーなインタラクティブ地図のためのJavaScriptライブラリです

### 緯度経度データの取得

施設や地点の緯度経度を使用する場合は、以下のガイドラインに従ってください：

1. **モデルに保存されている値を使用しないでください**
2. **必ずWeb等を通じて最新の情報を調査してください**
3. 信頼できる情報源：
   - Google Maps
   - OpenStreetMap
   - 各施設の公式ウェブサイト
   - 地理情報システム (GIS) データベース

### 実装例

```javascript
// Leafletを使用した地図の基本的な実装例
const map = L.map('map').setView([緯度, 経度], ズームレベル);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
}).addTo(map);

// マーカーの追加（緯度経度は事前に調査したもの）
L.marker([緯度, 経度]).addTo(map)
    .bindPopup('施設名や説明');
```

## 一般的なガイドライン

- データの正確性を重視してください
- 最新の情報を使用してください
- 適切な情報源を常に確認してください

## デプロイメントプラットフォーム

### Webアプリケーションのデプロイ

Webアプリケーションを作成・デプロイする際は、以下のプラットフォームを使用してください：

- **推奨**: **Cloudflare Pages** または **Cloudflare Workers**
  - 公式サイト: https://pages.cloudflare.com/ / https://workers.cloudflare.com/
  - 高速なグローバルCDN
  - エッジでの実行による低レイテンシ
  - 無料プランでも十分な機能

#### Cloudflare Pagesの特徴

- 静的サイトとフルスタックアプリケーションの両方に対応
- Git連携による自動デプロイ
- カスタムドメインとSSL証明書の自動設定
- プレビュー環境の自動生成
- 用途: 静的サイト、JAMstack、フロントエンドアプリケーション

#### Cloudflare Workersの特徴

- サーバーレス関数のエッジでの実行
- 低レイテンシかつスケーラブル
- KV、Durable Objects、R2などのストレージオプション
- 用途: APIエンドポイント、ミドルウェア、サーバーサイドロジックが必要なアプリケーション

### 以前のプラットフォーム

- **Vercel**: 以前使用していましたが、Cloudflare Pages/Workersへ移行中です

## セキュリティガイドライン

### データ読み込み時のセキュリティチェックリスト

外部データソースからデータを読み込む際、LLMを騙して悪意のあるスクリプトを入力させようとする攻撃があります。以下のチェックリストに従って、セキュリティ対策を実装してください。

#### 1. 入力検証 (Input Validation)

- [ ] **型検証**: すべての入力データの型を検証する
- [ ] **範囲チェック**: 数値データは期待される範囲内であることを確認する
- [ ] **長さ制限**: 文字列データに適切な長さ制限を設ける
- [ ] **許可リスト**: 可能な限り、許可された値のリストに対して検証する
- [ ] **拒否リスト**: 既知の悪意のあるパターンを拒否する

```javascript
// 良い例
function validateInput(data) {
  const errors = [];
  
  if (typeof data.name !== 'string') {
    errors.push('Invalid name type');
  } else {
    if (data.name.length > 100) {
      errors.push('Name too long');
    }
    if (!/^[a-zA-Z0-9\s]+$/.test(data.name)) {
      errors.push('Name contains invalid characters');
    }
  }
  
  return {
    valid: errors.length === 0,
    errors: errors
  };
}
```

#### 2. サニタイゼーション (Sanitization)

- [ ] **HTMLエスケープ**: HTML出力前に特殊文字をエスケープする
- [ ] **SQLインジェクション対策**: プリペアドステートメントを使用する
- [ ] **スクリプトタグの除去**: ユーザー入力からスクリプトタグを除去する
- [ ] **コマンドインジェクション対策**: シェルコマンドに渡す前に入力をサニタイズする
- [ ] **パスインジェクション対策**: ファイルパスの検証と正規化を行う

```javascript
// 良い例: HTMLエスケープ
function escapeHtml(unsafe) {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#x27;")
    .replace(/\//g, "&#x2F;")
    .replace(/`/g, "&#x60;");
}

// より推奨: 既存のライブラリを使用
// import DOMPurify from 'dompurify';
// const clean = DOMPurify.sanitize(dirty);

// 良い例: SQLインジェクション対策
const query = 'SELECT * FROM users WHERE id = ?';
db.execute(query, [userId]); // プリペアドステートメント使用
```

#### 3. プロンプトインジェクション対策

- [ ] **システムプロンプトの保護**: ユーザー入力がシステムプロンプトを上書きできないようにする
- [ ] **入力の明確な区切り**: システム命令とユーザー入力を明確に分離する
- [ ] **メタ文字のエスケープ**: LLMプロンプトで特別な意味を持つ文字をエスケープする
- [ ] **出力の検証**: LLMの出力が期待される形式であることを確認する
- [ ] **コンテキストの制限**: 必要最小限の権限とコンテキストのみを提供する

```javascript
// 良い例: プロンプトインジェクション対策
function sanitizeForPrompt(userInput) {
  // 危険な文字とパターンを除去（順序が重要）
  return userInput
    .replace(/```/g, '')  // コードブロックを除去
    .replace(/[<>]/g, '')  // 危険な特殊文字を除去
    .replace(/ignore\s+(previous|above|prior|all)\s+(instructions?|prompts?|commands?)/gi, '[filtered]')  // 命令の上書き試行を除去
    .replace(/\b(system|user|assistant|human)\s*:/gi, 'user-')  // ロール指定を無効化
    .replace(/(?<!user-)\b(system|assistant)\b/gi, 'user-$1')  // 未処理のロール言及を無効化（既に変換されたものは除外）
    .substring(0, 1000);  // 長さ制限
}

function buildPrompt(userInput) {
  const sanitizedInput = sanitizeForPrompt(userInput);
  return `
System: You are a helpful assistant. Only respond to the following user query.
User Input Starts Here:
---
${sanitizedInput}
---
User Input Ends Here.
Provide a helpful response.`;
}
```

#### 4. データソースの検証

- [ ] **信頼できるソースのみ使用**: 検証済みのデータソースからのみデータを読み込む
- [ ] **HTTPS使用**: 外部APIとの通信には必ずHTTPSを使用する
- [ ] **証明書の検証**: SSL/TLS証明書を適切に検証する
- [ ] **レート制限**: 外部APIへのリクエストにレート制限を実装する
- [ ] **タイムアウト設定**: すべての外部リクエストにタイムアウトを設定する

```javascript
// 良い例
// 信頼できるドメインを小文字で事前処理
const trustedDomains = ['api.example.com', 'data.trustedsite.com'].map(d => d.toLowerCase());

function validateDataSource(url) {
  try {
    const urlObj = new URL(url);
    
    // HTTPSのみ許可
    if (urlObj.protocol !== 'https:') {
      throw new Error(`Only HTTPS sources are allowed, got: ${urlObj.protocol}`);
    }
    
    // 正確なドメインマッチング（サブドメイン攻撃を防ぐ）
    const hostname = urlObj.hostname.toLowerCase();
    const isExactMatch = trustedDomains.includes(hostname);
    
    if (!isExactMatch) {
      throw new Error(`Untrusted data source: ${hostname}`);
    }
    
    // 国際化ドメイン名（Punycode）の検証
    // より厳密な検証には専用ライブラリの使用を推奨
    const domainParts = hostname.split('.');
    if (domainParts.some(part => part.startsWith('xn--'))) {
      throw new Error(`Internationalized domain names are not allowed: ${hostname}`);
    }
    
    return true;
  } catch (error) {
    if (error instanceof TypeError) {
      throw new Error(`Invalid URL format: ${url}`);
    }
    throw error;
  }
}
```

#### 5. 出力エンコーディング

- [ ] **コンテキスト適切なエンコーディング**: 出力先に応じて適切なエンコーディングを使用する
- [ ] **JSON出力の検証**: JSON出力が適切にエスケープされていることを確認する
- [ ] **XSS対策**: クライアント側での動的なDOM操作時にエスケープする
- [ ] **CSP実装**: Content Security Policyを実装する

```javascript
// 良い例: コンテキスト別のエスケープ
// 注意: escapeHtml関数は上記のサニタイゼーションセクションで定義されています
function renderData(data, context) {
  switch(context) {
    case 'html':
      return escapeHtml(data);  // 上記のescapeHtml関数を使用
    case 'js':
      return JSON.stringify(data);
    case 'url':
      return encodeURIComponent(data);
    default:
      throw new Error('Unknown context');
  }
}
```

#### 6. エラーハンドリング

- [ ] **詳細なエラー情報の非表示**: 本番環境でシステムの詳細を露出しない
- [ ] **ログ記録**: セキュリティ関連のイベントを適切にログに記録する
- [ ] **グレースフルな失敗**: 攻撃を受けても、安全に失敗する
- [ ] **監視とアラート**: 異常なパターンを検知してアラートする

```javascript
// 良い例
function sanitizeForLogging(input) {
  // ログ用にデータをサニタイズ（個人情報や機密情報を除去）
  if (typeof input === 'string') {
    return input.substring(0, 100) + (input.length > 100 ? '...' : '');
  }
  return '[complex data]';
}

try {
  processUserInput(input);
} catch (error) {
  // 詳細なエラーはログに記録（入力はサニタイズ）
  logger.error('Input processing failed', { 
    error: error.message,
    inputPreview: sanitizeForLogging(input)
  });
  // ユーザーには一般的なメッセージのみ
  return { success: false, message: 'Invalid input' };
}
```

#### 7. アクセス制御

- [ ] **認証の実装**: すべての機密操作に認証を要求する
- [ ] **認可の確認**: ユーザーが操作を実行する権限があることを確認する
- [ ] **最小権限の原則**: 必要最小限の権限のみを付与する
- [ ] **セッション管理**: セキュアなセッション管理を実装する

### 定期的なセキュリティレビュー

- コード変更時には、必ず上記チェックリストを確認してください
- 外部ライブラリを使用する場合は、既知の脆弱性がないか確認してください
- セキュリティパッチは速やかに適用してください
- 定期的にセキュリティスキャンツールを実行してください
