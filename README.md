# さくらのAppRunにNode.jsアプリをデプロイするAction

Node.jsアプリケーションをさくらのクラウドのAppRunへ自動ビルド・デプロイするGitHub Actionです。

SQLite のデータベースファイルを Litestream を使って自動でオブジェクトストレージにレプリケーションすることができるので、簡単な設定だけで **コンテナ+SQLite環境で永続化を実現** します。

## 特徴

- 🚀 複数のNode.jsフレームワークに対応（HonoX、Next.js、React Router）
- 🔍 package.jsonから自動フレームワーク検出
- 🌸 さくらのAppRunへ直接デプロイ
- 📦 npm startで起動するアプリに対応（デフォルト3000、ポート指定可能）
- 🏗️ TypeScriptビルド対応（マルチステージビルド）
- 🔄 既存アプリケーションの自動更新
- 🏷️ Git SHAベースの自動バージョニング
- 🔐 安全な認証情報の管理
- 🗄️ Litestreamを使ったSQLiteのレプリケーション機能
- 📁 自前のDockerfileを使うことも可能

## 対応フレームワーク

このActionは以下のNode.jsフレームワークを自動で検出し、適切なビルドプロセスを実行します：

- **[HonoX](https://github.com/honojs/honox)** - [サンプル](https://github.com/meso/honox-node)
- **[Next.js](https://nextjs.org/)** - [サンプル](https://github.com/meso/next-sample)
- **[React Router](https://reactrouter.com/)** - [サンプル](https://github.com/meso/reactrouter-todo-app)

フレームワークの検出は`package.json`の依存関係から自動的に行われます。

## 事前準備

1. さくらのクラウドアカウント（AppRun有効化済み）
2. さくらのコンテナレジストリ
3. 各種APIキー・認証情報：
   - さくらのクラウドAPIキー（AppRun権限付き）
   - コンテナレジストリの認証情報
   - （SQLiteレプリケーション使用時）さくらのオブジェクトストレージの認証情報

## 使い方

### 基本的な使用例

```yaml
- name: さくらのAppRunへデプロイ
  uses: meso/sakura-apprun-action@v3
  with:
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
    container-registry: MYREGISTRY.sakuracr.jp
    container-registry-user: ${{ secrets.REGISTRY_USER }}
    container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
```

### 自前のDockerfileを使用する場合

```yaml
- name: さくらのAppRunへデプロイ
  uses: meso/sakura-apprun-action@v3
  with:
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
    container-registry: MYREGISTRY.sakuracr.jp
    container-registry-user: ${{ secrets.REGISTRY_USER }}
    container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
    use-repository-dockerfile: 'true'
```


## 入力パラメータ

| 名前 | 説明 | 必須 | デフォルト値 |
|------|------|------|------------|
| `sakura-api-key` | さくらのクラウドAPIキー | はい | - |
| `sakura-api-secret` | さくらのクラウドAPIシークレット | はい | - |
| `container-registry` | コンテナレジストリのURL | はい | - |
| `container-registry-user` | レジストリのユーザー名 | はい | - |
| `container-registry-password` | レジストリのパスワード | はい | - |
| `app-name` | アプリケーション名 | いいえ | リポジトリ名 |
| `port` | アプリケーションのポート番号 | いいえ | `3000` |
| `max-cpu` | 最大CPU | いいえ | `0.4` |
| `max-memory` | 最大メモリ | いいえ | `256Mi` |
| `timeout-seconds` | リクエストタイムアウト（秒） | いいえ | `300` |
| `use-repository-dockerfile` | リポジトリのDockerfileを使用する | いいえ | `true` |
| `object-storage-bucket` | SQLiteレプリケーション用のオブジェクトストレージバケット名 | いいえ | - |
| `object-storage-access-key` | オブジェクトストレージのアクセスキー | いいえ | - |
| `object-storage-secret-key` | オブジェクトストレージのシークレットキー | いいえ | - |
| `sqlite-db-path` | SQLiteデータベースファイルのパス | いいえ | - |
| `litestream-replicate-interval` | Litestreamレプリケーション間隔 | いいえ | `10s` |

## Dockerfile設定

### use-repository-dockerfile パラメータについて

- `true`（デフォルト）: リポジトリのルートに`Dockerfile`が存在する場合、それを使用します
- `false`: 常に最適化されたDockerfileを使用します

リポジトリにDockerfileが存在しない場合は、自動的に最適化されたDockerfileが使用されます。

## フレームワーク自動検出

本Actionは`package.json`の依存関係を分析して、使用するフレームワークを自動的に検出します：

1. **Next.js**: `"next"`の依存関係が検出された場合
2. **React Router**: `"@react-router/dev"`または`"react-router"`の依存関係が検出された場合
3. **HonoX**: 上記以外の場合（デフォルト）

全てのフレームワークで`npm run build`によるビルドと`npm start`による起動をサポートしています。

## LitestreamによるSQLiteレプリケーション

本Actionは、SQLiteデータベースのレプリケーション機能を提供します。以下の全てのパラメータが設定されている場合、Litestreamが自動的に有効になります：

- `object-storage-bucket`
- `object-storage-access-key` 
- `object-storage-secret-key`
- `sqlite-db-path`

### バックアップファイルのバケット構造

データベースのバックアップは以下の形式で整理されます：
```
{bucket}/{app-name}/{database-filename}
```

これにより、複数のアプリケーションが同じバケットを使用してもファイルが衝突することはありません。

### Litestreamを使用した例

```yaml
- name: さくらのAppRunへデプロイ
  uses: meso/sakura-apprun-action@v3
  with:
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
    container-registry: MYREGISTRY.sakuracr.jp
    container-registry-user: ${{ secrets.REGISTRY_USER }}
    container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
    # SQLiteレプリケーション設定
    object-storage-bucket: my-backup-bucket
    object-storage-access-key: ${{ secrets.S3_ACCESS_KEY }}
    object-storage-secret-key: ${{ secrets.S3_SECRET_KEY }}
    sqlite-db-path: ./database.sqlite
    litestream-replicate-interval: 5s
```

## 出力

| 名前 | 説明 |
|------|------|
| `public-url` | デプロイされたアプリケーションの公開URL |
| `app-id` | AppRunアプリケーションID |

## 完全なワークフローの例

### 基本的なワークフロー

```yaml
name: 本番環境へデプロイ

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: チェックアウト
      uses: actions/checkout@v4
      
    - name: さくらのAppRunへデプロイ
      id: deploy
      uses: meso/sakura-apprun-action@v3
      with:
        sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
        sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
        container-registry: MYREGISTRY.sakuracr.jp
        container-registry-user: ${{ secrets.REGISTRY_USER }}
        container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
        port: '3000'
        
    - name: デプロイURLを表示
      run: |
        echo "デプロイ先: ${{ steps.deploy.outputs.public-url }}"
```

### SQLiteバックアップ付きワークフロー

```yaml
name: SQLiteレプリケーション付きデプロイ

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: チェックアウト
      uses: actions/checkout@v4
      
    - name: さくらのAppRunへデプロイ（SQLiteバックアップ付き）
      id: deploy
      uses: meso/sakura-apprun-action@v3
      with:
        sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
        sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
        container-registry: MYREGISTRY.sakuracr.jp
        container-registry-user: ${{ secrets.REGISTRY_USER }}
        container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
        # SQLiteレプリケーション設定
        object-storage-bucket: my-backup-bucket
        object-storage-access-key: ${{ secrets.S3_ACCESS_KEY }}
        object-storage-secret-key: ${{ secrets.S3_SECRET_KEY }}
        sqlite-db-path: ./data/app.db
        litestream-replicate-interval: 10s
        
    - name: デプロイURLを表示
      run: |
        echo "デプロイ先: ${{ steps.deploy.outputs.public-url }}"
        echo "アプリID: ${{ steps.deploy.outputs.app-id }}"
```

## シークレットの設定方法

GitHubリポジトリの Settings > Secrets and variables > Actions で以下を追加：

**基本的なデプロイに必要なシークレット:**
- `SAKURA_API_KEY`: さくらのクラウドAPIキー（AppRun権限付き）
- `SAKURA_API_SECRET`: さくらのクラウドAPIシークレット
- `REGISTRY_USER`: さくらのコンテナレジストリのユーザー名
- `REGISTRY_PASSWORD`: さくらのコンテナレジストリのパスワード

**SQLiteバックアップに必要な追加シークレット:**
- `S3_ACCESS_KEY`: さくらのオブジェクトストレージアクセスキー
- `S3_SECRET_KEY`: さくらのオブジェクトストレージシークレットキー

## よくある質問

### Q: 対応しているフレームワークは？

A: HonoX、Next.js、React Routerに対応しています。フレームワークはpackage.jsonから自動的に検出されます。

### Q: 自分のDockerfileを使いたい場合は？

A: `use-repository-dockerfile: 'true'`（デフォルト）を設定し、リポジトリのルートにDockerfileを配置してください。

### Q: 最適化されたDockerfileを使いたい場合は？

A: `use-repository-dockerfile: 'false'`を設定するか、リポジトリのルートにDockerfileを置かないでください。

### Q: npm start以外のコマンドで起動したい

A: 現在はnpm startのみ対応しています。package.jsonのscriptsにstartコマンドを定義してください。

### Q: デプロイが失敗する

A: 以下を確認してください：
- APIキーにAppRun権限があるか
- コンテナレジストリの認証情報が正しいか
- アプリケーション名が既に使用されていないか（v1以降は自動的に更新されます）

### Q: TypeScriptプロジェクトは対応していますか？

A: はい、対応しています。package.jsonに`build`スクリプトが定義されていれば自動的にビルドされます。

### Q: 既存のアプリケーションを更新できますか？

A: はい、同名のアプリケーションが存在する場合は自動的に更新されます。

### Q: SQLiteレプリケーションはどのように動作しますか？

A: Litestreamを使用して、SQLiteデータベースに変更があった場合に自動的にさくらのオブジェクトストレージにレプリケーションされます。変更がない場合はレプリケーションは実行されないため、効率的です。全ての必要なパラメータが設定されている場合にのみ有効になります。

### Q: Litestreamのレプリケーション間隔を変更できますか？

A: はい、`litestream-replicate-interval`パラメータで設定可能です（例：1s、10s、1m）。これは変更をチェックする頻度で、実際のレプリケーションはデータベースに変更があった時にのみ実行されます。デフォルトは10秒です。

### Q: 複数のアプリケーションが同じバケットを使用する場合はどうなりますか？

A: バックアップファイルはアプリケーション名で自動的に整理されるため、ファイルが衝突することはありません。バックアップパスは`{bucket}/{app-name}/{database-filename}`の形式になります。

### Q: データベースの復元はどのように行われますか？

A: アプリケーション起動時に、指定されたパスにSQLiteデータベースファイルが存在しない場合、Litestreamが自動的にバックアップから復元を試みます。バックアップが存在しない場合は、新しいデータベースファイルが作成されます。

## ライセンス

MIT