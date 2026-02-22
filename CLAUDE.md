# AI Chat Bot - プロジェクト仕様書

## プロジェクト概要

エンターテイメント向けAIチャットボットWebアプリケーション。
一般ユーザーがブラウザからAIと対話できるシンプルなチャットサービス。

## 技術スタック

| カテゴリ | 技術 |
|---------|------|
| フレームワーク | Next.js (App Router) |
| API | Hono |
| ORM | Prisma.js |
| データベース | MongoDB |
| AIエージェント | Mastra |
| AIモデル | Claude claude-4-sonnet (Anthropic) |
| Web検索API | Exa (exa-js) |
| デプロイ | Google Cloud Run |

## ディレクトリ構成（推奨）

```
ai-chat/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx
│   │   ├── page.tsx            # チャットUI
│   │   └── api/
│   │       └── [...route]/     # Hono APIルート
│   │           └── route.ts
│   ├── components/             # UIコンポーネント
│   │   ├── Chat.tsx
│   │   ├── MessageList.tsx
│   │   └── MessageInput.tsx
│   ├── lib/
│   │   ├── prisma.ts           # Prismaクライアント
│   │   └── mastra/             # Mastra設定
│   │       ├── index.ts
│   │       ├── agents/
│   │       │   └── chat-agent.ts
│   │       └── tools/
│   │           └── web-search.ts  # Web検索ツール（Exa API）
│   └── types/
│       └── index.ts            # 型定義
├── prisma/
│   └── schema.prisma           # Prismaスキーマ
├── public/
├── .env.local                  # 環境変数（ローカル）
├── .env.example                # 環境変数テンプレート
├── Dockerfile                  # Cloud Run用
├── next.config.js
├── package.json
├── tsconfig.json
└── CLAUDE.md
```

## 機能要件

### 必須機能

- [ ] チャットUI（メッセージ送信・表示）
- [ ] Claude APIとの対話
- [ ] Mastraを使用したエージェント実装
- [ ] レスポンシブデザイン（PC/スマホ対応）

### 追加機能：Web検索 & X(Twitter)バズポスト検索

Mastraの`createTool`を使い、Web検索APIを統合してチャットエージェントの能力を拡張する。

#### 概要

- ユーザーがチャットで「〇〇に関するバズポストを教えて」と聞くと、エージェントがWeb検索ツールを呼び出してX(Twitter)上の関連ポストを取得・要約して返す
- 同じツールでWebサイトの最新記事取得にも対応

#### 検索プロバイダー

| プロバイダー | 用途 | コスト |
|------------|------|-------|
| Exa（推奨） | セマンティック検索、X投稿・Web記事の取得 | 無料枠あり〜従量課金 |
| Tavily（代替） | RAG/AIエージェント向け検索 | $0.008/リクエスト |

#### 機能要件

- [ ] Web検索ツールの実装（`createTool`で Exa API をラップ）
- [ ] X(Twitter)バズポスト検索（`site:x.com` フィルターでキーワード検索）
- [ ] 特定Webサイトの最新記事取得（`site:` フィルターでドメイン指定検索）
- [ ] チャットエージェントへのツール統合（エージェントが文脈に応じてツールを自動呼び出し）

#### ツール実装

```typescript
// src/lib/mastra/tools/web-search.ts
import { createTool } from "@mastra/core/tools";
import { z } from "zod";
import Exa from "exa-js";

const exa = new Exa(process.env.EXA_API_KEY!);

export const webSearchTool = createTool({
  id: "web-search",
  description: "Web検索で最新情報を取得する。X(Twitter)のバズポストやWebサイトの記事を検索できる",
  inputSchema: z.object({
    query: z.string().describe("検索クエリ"),
    siteFilter: z.string().optional().describe("特定サイトに絞る場合のドメイン（例: x.com）"),
  }),
  outputSchema: z.object({
    results: z.array(z.object({
      title: z.string(),
      url: z.string(),
      content: z.string(),
      publishedDate: z.string().optional(),
    })),
  }),
  execute: async ({ context }) => {
    const query = context.siteFilter
      ? `${context.query} site:${context.siteFilter}`
      : context.query;
    const results = await exa.searchAndContents(query, {
      numResults: 5,
      livecrawl: "always",
    });
    return {
      results: results.results.map((r) => ({
        title: r.title ?? "",
        url: r.url,
        content: r.text ?? "",
        publishedDate: r.publishedDate ?? "",
      })),
    };
  },
});
```

#### 取得可能な情報と制限

| 情報 | 取得可否 | 備考 |
|------|:-------:|------|
| ポスト本文 | ○ | 検索結果から取得 |
| 投稿者・投稿日 | ○ | 検索結果に含まれる |
| ポストURL | ○ | 元ポストへのリンク |
| いいね数・RT数 | × | Web検索APIでは取得不可（将来Phase 2で対応検討） |

### 会話履歴

- セッション中のみ保持（ブラウザのstateで管理）
- 永続化は不要
- ページリロードでリセット

### 対象外（実装しない）

- ユーザー認証
- 多言語対応
- 画像認識・音声入力
- 会話履歴の永続化・検索
- X APIによるエンゲージメントデータ取得（Phase 2として将来検討）

## 非機能要件

| 項目 | 内容 |
|------|------|
| 同時接続数 | 5〜10人程度 |
| デザイン | ビジネスライク |
| レスポンシブ | 対応必須 |
| API利用制限 | なし |

## 環境変数

```env
# .env.local
ANTHROPIC_API_KEY=your_api_key_here
MONGODB_URI=mongodb://localhost:27017/ai-chat
EXA_API_KEY=your_exa_api_key_here
```

## API設計

### POST /api/chat

チャットメッセージを送信し、AIの応答を取得する。

**Request:**
```json
{
  "message": "こんにちは",
  "history": [
    { "role": "user", "content": "前のメッセージ" },
    { "role": "assistant", "content": "前の応答" }
  ]
}
```

**Response:**
```json
{
  "response": "こんにちは！何かお手伝いできることはありますか？"
}
```

## Mastra エージェント設定

```typescript
// src/lib/mastra/agents/chat-agent.ts
import { Agent } from '@mastra/core';
import { webSearchTool } from '../tools/web-search';

export const chatAgent = new Agent({
  name: 'chat-agent',
  model: 'claude-4-sonnet',
  instructions: `
    あなたはフレンドリーなAIアシスタントです。
    ユーザーとの会話を楽しみ、適切な応答を返してください。

    以下のツールを利用できます：
    - web-search: X(Twitter)のバズポストやWebサイトの最新記事を検索できます。
      ユーザーが最新情報やトレンド、バズっている話題について質問した場合に使用してください。
      X(Twitter)の投稿を検索する場合は siteFilter に "x.com" を指定してください。
  `,
  tools: {
    webSearchTool,
  },
});
```

## デプロイ

### Google Cloud Run

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### デプロイコマンド

```bash
# ビルド & デプロイ
gcloud run deploy ai-chat \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated
```

## 開発コマンド

```bash
# 依存関係インストール
npm install

# 開発サーバー起動
npm run dev

# ビルド
npm run build

# 本番起動
npm start

# Prisma
npx prisma generate
npx prisma db push
```

## 注意事項

- `ANTHROPIC_API_KEY` は絶対にコミットしない
- `EXA_API_KEY` は絶対にコミットしない
- `.env.local` は `.gitignore` に含める
- MongoDBはセッション管理用に将来拡張可能な設計とする
- Web検索ツールはExa APIの無料枠を超えた場合に課金が発生するため、利用状況を監視すること
