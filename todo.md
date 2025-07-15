# ToDo リスト

## 主要タスク

### 1. HonoX 対応（現在の実装）
HonoX フレームワークのサポートを維持・改善する。

### 2. Next.js 対応
Next.js フルスタックフレームワークのサポートを追加する。

### 3. React Router 対応
React Router（Remix の進化版）フルスタックフレームワークのサポートを追加する。

---

## 現在の実装状況

### HonoX 対応（既存）
現在の `Dockerfile.node` と `Dockerfile.node-litestream` は HonoX を想定しています：

**特徴：**
- `npm run build` → `dist/` に出力
- `npm start` で実行
- SQLite + Litestream バックアップ機能

---

## フレームワーク別の特徴と要件

### 1. HonoX
**ビルドプロセス：**
- `npm run build` → `dist/` 出力
- `npm start` で実行

**現在の対応状況：** ✅ 完了

### 2. Next.js
**ビルドプロセス：**
- `npm run build` → `.next/` 出力
- `npm start` または `next start` で実行
- standalone ビルド対応

**必要な対応：** 🔄 要実装

### 3. React Router
**ビルドプロセス：**
- `npm run build` (内部的に `react-router build`) → `build/` 出力
- `npm start` で実行（`react-router-serve` を使用）
- Vite ベースのビルドシステム

**必要な対応：** 🔄 要実装

---

## 統一的なアプローチ

### 1. フレームワーク自動検出
- [ ] package.json の dependencies を分析
- [ ] `honox` または `@honox/vite` → HonoX として処理
- [ ] `next` → Next.js として処理
- [ ] `@react-router/dev` または `react-router` → React Router として処理

### 2. 統一 Dockerfile の作成
#### `Dockerfile.unified` 
各フレームワークに対応する統一的な Dockerfile を作成：

```dockerfile
# ビルドステージ
FROM node:22-alpine AS builder

WORKDIR /app

# package*.jsonをコピー
COPY package*.json ./

# 依存関係をインストール
RUN npm ci || npm install

# アプリケーションのソースをコピー
COPY . .

# フレームワーク検出とビルド
RUN if [ -f package.json ]; then \
      if grep -q '"next"' package.json; then \
        echo "Building Next.js app..." && npm run build; \
      elif grep -q '"@react-router/dev"' package.json || grep -q '"react-router"' package.json; then \
        echo "Building React Router app..." && npm run build; \
      else \
        echo "Building HonoX app..." && npm run build; \
      fi \
    fi

# 実行ステージ
FROM node:22-alpine

WORKDIR /app

# package*.jsonをコピー
COPY package*.json ./

# 本番用の依存関係のみインストール
RUN npm ci --only=production || npm install --production

# フレームワーク別の成果物をコピー
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/build ./build

# 必要に応じて他のファイルもコピー
COPY --from=builder /app/public ./public

# 環境変数
ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3000

EXPOSE 3000

# フレームワーク別の起動コマンド
CMD npm start
```

### 3. Litestream 統合版
#### `Dockerfile.unified-litestream`
統一 Dockerfile に Litestream バックアップ機能を統合：

- [ ] 統一的な起動スクリプトの作成
- [ ] フレームワーク別の実行コマンド対応
- [ ] SQLite データベースパスの自動検出

---

## 詳細実装項目

### Next.js 対応

#### 1. Next.js 設定サポート
- [ ] `next.config.js` の standalone 出力設定の検証
- [ ] 環境変数の適切な処理（NEXT_PUBLIC_ プレフィックス）
- [ ] Image Optimization の考慮

#### 2. 実行環境の最適化
- [ ] standalone ビルドの活用
- [ ] メモリ使用量の最適化
- [ ] 静的アセットの適切な配置

### React Router 対応

#### 1. Vite ベースのビルドシステム対応
- [ ] `react-router.config.ts` の検出
- [ ] Vite プラグインの適切な処理
- [ ] 環境変数の管理

#### 2. 実行環境の構築
- [ ] `npm start` での実行（`react-router-serve` を使用）
- [ ] loaders と actions のサポート
- [ ] データベース接続の考慮

---

## action.yml の拡張

### 新しい入力パラメータ
```yaml
inputs:
  framework-type:
    description: 'Framework type (auto, honox, nextjs, react-router)'
    required: false
    default: 'auto'
  build-command:
    description: 'Custom build command (overrides auto-detection)'
    required: false
  start-command:
    description: 'Custom start command (overrides auto-detection)'
    required: false
  output-directory:
    description: 'Build output directory for custom frameworks'
    required: false
```

### フレームワーク検出ロジック
- [ ] package.json の解析
- [ ] カスタムコマンドの対応
- [ ] デフォルト設定の提供

---

## 使用例（実装後）

### 自動検出（推奨）
```yaml
- uses: meso/sakura-apprun-action@v2
  with:
    framework-type: auto  # デフォルト
    apprun-token: ${{ secrets.SAKURA_APPRUN_TOKEN }}
    apprun-app: my-app
```

### HonoX（既存）
```yaml
- uses: meso/sakura-apprun-action@v2
  with:
    framework-type: honox
    apprun-token: ${{ secrets.SAKURA_APPRUN_TOKEN }}
    apprun-app: my-honox-app
```

### Next.js
```yaml
- uses: meso/sakura-apprun-action@v2
  with:
    framework-type: nextjs
    apprun-token: ${{ secrets.SAKURA_APPRUN_TOKEN }}
    apprun-app: my-nextjs-app
```

### React Router
```yaml
- uses: meso/sakura-apprun-action@v2
  with:
    framework-type: react-router
    apprun-token: ${{ secrets.SAKURA_APPRUN_TOKEN }}
    apprun-app: my-react-router-app
```

---

## 実装の優先順位

1. **高**: フレームワーク自動検出機能
2. **高**: 統一 Dockerfile の作成
3. **高**: Next.js 基本対応
4. **高**: React Router 基本対応
5. **中**: Litestream 統合版の作成
6. **中**: action.yml の拡張
7. **低**: 高度な機能（ISR、複雑な SSR など）

---

## 注意事項

- 既存の HonoX サポートを損なわない
- 後方互換性を維持する
- さくらのクラウド AppRun の制約を考慮
- ポート 3000 での動作を前提
- SQLite バックアップ機能との互換性を維持
- 各フレームワークの特性を尊重した実装

---

## 技術的な差異の整理

### HonoX
- Vite ベースの軽量フレームワーク
- `npm run build` → `dist/`
- `npm start` で実行
- Edge Runtime 対応

### Next.js
- React フルスタックフレームワーク
- `npm run build` → `.next/`
- `npm start` で実行
- App Router/Pages Router

### React Router
- Remix 進化版フレームワーク
- `npm run build` (内部的に `react-router build`) → `build/`
- `npm start` で実行（`react-router-serve` を使用）
- Vite ベースのビルドシステム

すべて Node.js サーバーとして動作し、それぞれ独自の最適化を持つフレームワークです。