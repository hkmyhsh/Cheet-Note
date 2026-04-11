task:
  name: "npm publish 用スクリプト作成"
  objective: "社内NPMレジストリへパッケージ公開するCI用スクリプトを生成する"

requirements:
  functional:
    - "指定ブランチでのみ publish する"
    - "package.json の version を利用する"
    - "publish 前に test を実行する"
  non_functional:
    - "冪等性を考慮する"
    - "失敗時に原因が分かるログを出す"
    - "資格情報をログ出力しない"

inputs:
  user_inputs:
    - name: "branch"
      type: "string"
      required: true
      example: "main"
    - name: "registry_url"
      type: "string"
      required: true
      example: "https://npm.example.co.jp/repository/npm-private/"
    - name: "package_scope"
      type: "string"
      required: false
      example: "@my-org"

trigger:
  type: "github_actions"
  event: "push"
  branches:
    - "main"

outputs:
  artifacts:
    - "公開済み npm package"
  logs:
    - "publish 成否"
    - "失敗理由"
  exit_codes:
    success: 0
    failure: "non-zero"

external_integrations:
  - type: "npm_registry"
    name: "private-npm"
    endpoint: "https://npm.example.co.jp/repository/npm-private/"
    auth:
      method: "token"
      secret_name: "NPM_TOKEN"

security_constraints:
  certificates:
    custom_ca_required: true
    ca_path: "/usr/local/share/ca-certificates/internal-ca.crt"
    verify_tls: true
  secret_handling:
    - "token を echo しない"
    - "set -x を使わない"
    - "認証情報は環境変数または secrets から取得する"

build_tools:
  maven:
    use: false
  npm:
    use: true
    version: "10.x"
    registry: "https://npm.example.co.jp/repository/npm-private/"
    config_file: ".npmrc"
  docker:
    use: true
    base_image: "node:20"
    registry: "docker.example.co.jp"
    certificate_required: true

repositories:
  source_repo: "github.com/example-org/sample-app"
  artifact_repo:
    npm: "private-npm"
    docker: "private-docker"

coding_constraints:
  language: "bash"
  style:
    - "POSIX互換ではなく bash 前提で可"
    - "関数化する"
    - "set -euo pipefail を使う"
  forbidden:
    - "ハードコードされた認証情報"
    - "TLS検証無効化"
    - "--insecure の使用"

acceptance_criteria:
  - "main ブランチ push 時のみ publish される"
  - "社内CA証明書を使ってTLS接続できる"
  - "認証情報がログに出ない"
  - "失敗時にどの工程で失敗したか分かる"

deliverables:
  - "GitHub Actions workflow YAML"
  - ".npmrc のサンプル"
  - "Dockerfile 例"
  - "必要な secrets 一覧"