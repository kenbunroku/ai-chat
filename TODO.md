# AI Chat Bot - 実行計画 TODO

## Phase 1: プロジェクト初期化

- [x] Next.js プロジェクト作成（App Router）
- [x] TypeScript 設定確認
- [x] ESLint / Prettier 設定
- [x] .gitignore 設定
- [x] .env.example 作成

## Phase 2: 依存パッケージのインストール

- [x] Hono インストール (`hono`)
- [x] Prisma インストール (`prisma`, `@prisma/client`)
- [x] MongoDB アダプタ設定
- [x] Mastra インストール (`@mastra/core`)
- [x] Anthropic SDK インストール (`@anthropic-ai/sdk`)
- [x] UI関連パッケージ (`lucide-react`)

## Phase 3: データベース設定

- [x] MongoDB ローカル環境構築（Docker）
- [x] Prisma schema 作成 (`prisma/schema.prisma`)
- [x] Prisma Client 生成 (`npx prisma generate`)
- [x] DB接続確認

## Phase 4: Mastra エージェント設定

- [ ] Mastra 設定ファイル作成 (`src/lib/mastra/index.ts`)
- [ ] チャットエージェント定義 (`src/lib/mastra/agents/chat-agent.ts`)
- [ ] Claude claude-4-sonnet モデル設定
- [ ] エージェント動作確認

## Phase 5: API 実装（Hono）

- [ ] Hono ルーター設定 (`src/app/api/[...route]/route.ts`)
- [ ] POST /api/chat エンドポイント実装
- [ ] リクエスト/レスポンス型定義
- [ ] エラーハンドリング実装
- [ ] API動作確認（curl / Postman）

## Phase 6: フロントエンド実装

- [ ] レイアウト作成 (`src/app/layout.tsx`)
- [ ] チャットページ作成 (`src/app/page.tsx`)
- [ ] コンポーネント作成
  - [ ] `Chat.tsx` - メインコンテナ
  - [ ] `MessageList.tsx` - メッセージ一覧表示
  - [ ] `MessageInput.tsx` - 入力フォーム
  - [ ] `Message.tsx` - 個別メッセージ
- [ ] 会話履歴のstate管理
- [ ] API呼び出し処理
- [ ] ローディング状態表示
- [ ] エラー表示

## Phase 7: スタイリング

- [ ] グローバルスタイル設定
- [ ] ビジネスライクなデザイン適用
- [ ] レスポンシブ対応（PC/タブレット/スマホ）
- [ ] ダークモード対応（任意）

## Phase 8: テスト・動作確認

- [ ] ローカル環境での動作確認
- [ ] 会話フローのテスト
- [ ] レスポンシブ表示確認
- [ ] エラーケースの確認

## Phase 9: デプロイ準備

- [ ] Dockerfile 作成
- [ ] next.config.js に `output: 'standalone'` 設定
- [ ] 環境変数の整理
- [ ] Cloud Run 用設定確認

## Phase 10: デプロイ

- [ ] Google Cloud プロジェクト設定
- [ ] Cloud Run へデプロイ
- [ ] 環境変数設定（Secret Manager or 直接）
- [ ] 本番動作確認
- [ ] ドメイン設定（任意）

---

## 補足: コマンドリファレンス

```bash
# Phase 1: プロジェクト作成
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir

# Phase 2: パッケージインストール
npm install hono @hono/node-server
npm install prisma @prisma/client
npm install @mastra/core
npm install @anthropic-ai/sdk

# Phase 3: Prisma初期化
npx prisma init --datasource-provider mongodb

# Phase 9-10: デプロイ
gcloud run deploy ai-chat --source . --region asia-northeast1 --allow-unauthenticated
```

## 進捗管理

| Phase | 状態 | 備考 |
|-------|------|------|
| 1. 初期化 | 完了 | |
| 2. パッケージ | 完了 | |
| 3. DB設定 | 完了 | |
| 4. Mastra | 未着手 | |
| 5. API | 未着手 | |
| 6. フロント | 未着手 | |
| 7. スタイル | 未着手 | |
| 8. テスト | 未着手 | |
| 9. デプロイ準備 | 未着手 | |
| 10. デプロイ | 未着手 | |
