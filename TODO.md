# TODO

## 将来的な機能追加

### Dockerfile拡張

- [ ] **Supervisord対応のDockerfile** - 複数プロセスを管理する必要があるアプリケーション用
  - 例：アプリケーションサーバー + ワーカープロセス
  - ファイル名案：`Dockerfile.node-supervisor`

- [ ] **Litestream対応のDockerfile** - SQLiteレプリケーション機能付き
  - SQLiteをさくらのオブジェクトストレージに自動バックアップ
  - リアルタイムレプリケーション対応
  - ファイル名案：`Dockerfile.node-litestream`

### その他の改善案

- [ ] Docker Compose対応（複数コンテナのアプリケーション）
- [ ] 環境変数の自動設定機能
- [ ] ヘルスチェックの詳細設定