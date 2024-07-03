# 依存関係のバージョンアップ
1. 検知：新しいバージョンがリリースされたことを知る
2. 把握：具体的にどのような変更が行われたか理解する
3. 実装：新しいバージョンを自身のコードへ組み込む
4. テスト：リグレッションテストを実施し、なにかが壊れた場合は対応する

# Dependabot
- 依存関係の管理をサポートする
- 機能
  - Dependabot version updates：最新バージョンへの自動アップデート
  - Dependabot security updates：脆弱性を含むバージョンの自動アップデート
  - Dependabot alerts：脆弱性が含まれるバージョンのアラート通知

# Dependabot version updates
- パッケージマネージャーと連携して依存関係を最新バージョンに保つ
  1. 新しいバージョンが出ていないか、定期的にチェック
  2. 新しいバージョンを検知したら、バージョンアップのプルリクエストを作成する
    - プルリクエストトリガーによるワークフローを用いたリグレッションテストも実行可能

# Dependabot がサポートする主要な言語とパッケージマネージャー
- パッケージマネージャーのあるツール
| 言語 | パッケージマネージャー |
----|------------------ 
| JavaScript | npm / yarn / pnpm |
| Python | pip / poetry / pip-compile / pipenv |
| Java | Maven / Gradle |
| Ruby | Bundler |
| Go | Go modules |
| .NET | NuGet |
| PHP | Composer |
| Rust | Cargo |
- パッケージマネージャーの無いツール
  - GitHub Actions
  - Docker
  - Terraform

# Dependabot の設定ファイル
