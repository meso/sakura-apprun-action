# Node.js さくらのAppRunデプロイAction

Node.jsアプリケーションをさくらのクラウドAppRunへ自動ビルド・デプロイするGitHub Actionです。

## 特徴

- 🚀 Node.jsアプリの自動コンテナ化
- 🌸 さくらのAppRunへ直接デプロイ
- 📦 npm startでポート3000で起動するアプリに対応
- 🏗️ TypeScriptビルド対応（マルチステージビルド）
- 🔄 既存アプリケーションの自動更新
- 🏷️ Git SHAベースの自動バージョニング
- 🔐 安全な認証情報の管理

## 事前準備

1. さくらのクラウドアカウント（AppRun有効化済み）
2. さくらのコンテナレジストリ
3. AppRun権限を持つAPIキー

## 使い方

### 基本的な使用例

```yaml
- name: さくらのAppRunへデプロイ
  uses: meso/sakura-apprun-action@v1
  with:
    sakura-api-key: ${{ secrets.SAKURA_API_KEY }}
    sakura-api-secret: ${{ secrets.SAKURA_API_SECRET }}
    container-registry: MYREGISTRY.sakuracr.jp
    container-registry-user: ${{ secrets.REGISTRY_USER }}
    container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
```


## 入力パラメータ

| 名前 | 説明 | 必須 | デフォルト値 |
|------|------|------|------------|
| `dockerfile-url` | Node.js用DockerfileのURL | いいえ | デフォルトのNode.js用Dockerfile |
| `sakura-api-key` | さくらのクラウドAPIキー | はい | - |
| `sakura-api-secret` | さくらのクラウドAPIシークレット | はい | - |
| `container-registry` | コンテナレジストリのURL | はい | - |
| `container-registry-user` | レジストリのユーザー名 | はい | - |
| `container-registry-password` | レジストリのパスワード | はい | - |
| `app-name` | アプリケーション名 | いいえ | リポジトリ名 |
| `port` | アプリケーションのポート番号 | いいえ | `3000` |
| `max-cpu` | 最大CPU | いいえ | `0.5` |
| `max-memory` | 最大メモリ | いいえ | `256Mi` |
| `timeout-seconds` | リクエストタイムアウト（秒） | いいえ | `300` |

## 出力

| 名前 | 説明 |
|------|------|
| `public-url` | デプロイされたアプリケーションの公開URL |
| `app-id` | AppRunアプリケーションID |

## 完全なワークフローの例

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
      uses: meso/sakura-apprun-action@v1
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

## シークレットの設定方法

GitHubリポジトリの Settings > Secrets and variables > Actions で以下を追加：

- `SAKURA_API_KEY`: さくらのクラウドAPIキー
- `SAKURA_API_SECRET`: さくらのクラウドAPIシークレット  
- `REGISTRY_USER`: コンテナレジストリのユーザー名
- `REGISTRY_PASSWORD`: コンテナレジストリのパスワード

## よくある質問

### Q: Dockerfileがリポジトリにない場合は？

A: `dockerfile-url`パラメータで外部のDockerfileを指定できます。

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

A: はい、同名のアプリケーションが存在する場合は自動的に更新されます

## ライセンス

MIT