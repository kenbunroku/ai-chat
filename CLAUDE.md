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
│   │       └── agents/
│   │           └── chat-agent.ts
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

### 会話履歴

- セッション中のみ保持（ブラウザのstateで管理）
- 永続化は不要
- ページリロードでリセット

### 対象外（実装しない）

- ユーザー認証
- 多言語対応
- 画像認識・音声入力
- 会話履歴の永続化・検索

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

export const chatAgent = new Agent({
  name: 'chat-agent',
  model: 'claude-4-sonnet',
  instructions: `
    あなたはフレンドリーなAIアシスタントです。
    ユーザーとの会話を楽しみ、適切な応答を返してください。
  `,
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
- `.env.local` は `.gitignore` に含める
- MongoDBはセッション管理用に将来拡張可能な設計とする
