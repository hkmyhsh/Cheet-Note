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
- '.github/depemdabot.yml' にDependabot の設定を記載する
- 記載例
  - ```
    version: 2
    updates:
      - package-ecosystem: github-actions # パッケージエコシステム
        directory: /                      # パッケージマニフェストの配置先ディレクトリ
        schedule:                         # バージョンアップスケジュール
          interval: daily
        ignore:                                      # バージョンアップの除外設定
          - dependency-name: actions/upload-artifact # 除外する依存関係の名前
            versions:                                # 除外対象のバージョン
              - 4.3.0
              - 4.3.1
          - dependency-name: 'actions/*'             # アスタリスクは任意文字列にマッチ
            update-types:                            # 除外するバージョンアップの種類
              - version-update:semver-major
    ```
    - version: 設定ファイル自体のバージョン
    - updates: 依存関係のアップデート設定
    - package-ecosystem: パッケージマネージャーやツールの識別子
      - 例: `github-actions` を指定すると `Reusable Workflow` が自動バージョンアップされる
    - directory: パッケージマニフェストの配置先ディレクトリを指定
      - リポジトリの**ルートディレクトリ**が起点
      - 例
        - package.json
        - Gemfile
        - Dockerfile
      - **ただし、GitHub Actions のみ固定で「/」と指定する**
    - schedule: バージョンアップのスケジュール設定
      - 例
        - daily
        - weekly
        - monthly
    - ignore: バージョンアップの除外設定
      - dependency-name：除外する依存関係の名前（* を使い0文字以上の任意の文字列を表現可能）
      - versions：除外対象のバージョン
      - update-types：除外するバージョンアップの種類
        - version-update:semver-major：メジャーバージョン（X.Y.ZのX部分）
        - version-update:semver-minor：マイナーバージョン（X.Y.ZのY部分）
        - version-update:semver-patch：パッチバージョン（X.Y.ZのZ部分）

# Dependabotコマンド
- プルリクエスト上で `@dependabot<command>` のようなコメントを投稿すると Dependabot が操作出来る
  - `@dependabot merge`: ステータスチェックがすべて成功ならマージする
  - `@dependabot close`: プルリクエストをクローズする
  - `@dependabot recreate`: プルリクエストを再作成する

# 【参考】ブランチプロテクションルールの3つのケース
- ブランチプロテクションルールなし
- ステータスチェックが必須（Require status checks to pass before merging）
- 自分以外の承認が必須（Require approvals）

# GitHub Actions による自動マージ
- Dependabot の役割: `依存関係のバージョンアップを検知` して、`プルリクエストを出す` こと
- ユーザ の役割: プルリクエストをレビューして、マージすること
  - 依存関係が多いとプルリクエストが多くなり、日々の運用負荷が高まる
    - 自動マージするワークフローを実装（下記コマンドをワークフローに組み込む）
      - `gh pr merge <NUMBER | URL | BRANCH> --merge`
      - ```
        name: Auto merge
        on: pull_request
        jobs:
          merge:
            if: ${{ github.actor == 'dependabot[bot]' }} # Dependabotのプルリクエストのみ
            runs-on: ubuntu-latest
            permissions:                                 # マージに必要なパーミッション
              contents: write
              pull-requests: write
            env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # GitHub CLIのクレデンシャル
            steps:
              - uses: actions/checkout@v4
              - run: gh pr merge "${GITHUB_HEAD_REF}" --merge # GitHub CLIでマージ
        ```
    - **自動マージとブランチプロテクションルールの両立**
      - ワークフローの実行を **Dependabot** が作成したプルリクエストに制限する
        - `if: ${{ github.actor == 'dependabot[bot]' }}`
      - 対象のソースブランチ名 `${GITHUB_HEAD_REF}` を取得し、マージ対象のプルリクエストを識別する
        - `- run: gh pr merge "${GITHUB_HEAD_REF}" --merge`
