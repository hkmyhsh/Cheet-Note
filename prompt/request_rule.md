# copilotに書いてもらいやすくするコツ

良い制約

	•	使うもの
	•	使わないもの
	•	接続先
	•	認証方法
	•	証明書の扱い
	•	失敗条件
	•	完成条件

弱い書き方

	•	「よしなにやって」
	•	「社内向けなので考慮して」
	•	「証明書も必要」
	•	「Nexusとかnpmとかdockerも使う」

## 特に重要な追加項目

	•	実行環境: GitHub Actions / Linux / self-hosted runner / container内実行 など
	•	認証情報の受け渡し方法: GitHub Secrets / env / mounted file / settings.xml / .npmrc / docker login
	•	証明書の適用範囲: OS trust store / Java truststore / Node/npm / Docker daemon / Maven settings.xml
	•	依存取得先と公開先の分離: 例: Maven download先は mirror、publish先は releases repository
	•	失敗時の扱い: fail fast / retry / warning only / continue-on-error の可否
	•	禁止事項: 社内環境ではここが重要です: 例: curl -k, NODE_TLS_REJECT_UNAUTHORIZED=0, mvn -Dmaven.wagon.http.ssl.insecure=true

## シンプルな形式

```
# Copilot 実装依頼テンプレート

## 1. 背景 / 目的
## 2. 作成対象
## 3. 入力
## 4. トリガー
## 5. 出力
## 6. 外部連携
## 7. 実行環境
## 8. 証明書 / TLS 制約
## 9. パッケージ管理 / レジストリ制約
## 10. 認証方式
## 11. リポジトリ / ブランチ制約
## 12. 実装ルール
## 13. 禁止事項
## 14. 受け入れ条件
## 15. 期待する成果物
```

# リポジトリ共通ルール

.github/copilot-instructions.md

# Repository Instructions

- このリポジトリは社内閉域向けCI資材を管理する
- すべての外部通信は社内CA証明書を使ってTLS検証を行う
- npm は private registry を使用する
- Maven は private Nexus を使用する
- Docker image pull/push は社内registryを使用する
- 認証情報は GitHub Actions secrets または環境変数から取得する
- 認証情報をログ出力してはいけない
- TLS検証無効化、--insecure、strict-ssl=false は禁止
- 変更時は build/test/validate 方法も合わせて示す

# パス別ルール

.github/instructions/*.instructions.md

パス別ルール。

たとえば Dockerfile 系、github/workflows 系、pom.xml 系で分ける。GitHubは path-specific instructions をサポートしています。 ￼

例:
	•	.github/instructions/workflows.instructions.md
	•	.github/instructions/docker.instructions.md
	•	.github/instructions/java.instructions.md

# 案件ごとの依頼テンプレート

# 実装依頼

## 目的
GitHub Actions 上で、main ブランチ push 時に npm package を社内registryへ publish する workflow を作成してください。

## 入力
- registry_url: https://npm.example.co.jp/repository/npm-private/
- npm_token: GitHub Secrets の NPM_TOKEN
- ca_cert_path: /usr/local/share/ca-certificates/internal-ca.crt

## トリガー
- push: main

## 出力
- .github/workflows/publish.yml
- 必要であれば .npmrc のサンプル
- 必要 secrets の一覧

## 外部連携
- 社内NPM registry に token 認証
- TLS は社内CA証明書を使って検証必須

## 技術制約
- Node 20
- npm 10
- bash 利用可
- set -euo pipefail を使う
- ログに secret を出力しない

## 禁止事項
- strict-ssl=false
- npm config set registry をグローバル汚染する書き方
- token の直書き
- --insecure
- 証明書未設定のまま接続すること

## 受け入れ条件
- main 以外では publish しない
- 社内CA証明書前提で成功する
- 失敗時に原因が分かるログになる
- 認証情報が露出しない

## 出力時の要望
- まず workflow 全体を提示
- 次に各 step の意図を説明
- 最後に必要 secrets と注意点を列挙


```
# Repository Instructions

- このリポジトリは社内閉域向けCI資材を管理する
- すべての外部通信は社内CA証明書を使ってTLS検証を行う
- npm は private registry を使用する
- Maven は private Nexus を使用する
- Docker image pull/push は社内registryを使用する
- 認証情報は GitHub Actions secrets または環境変数から取得する
- 認証情報をログ出力してはいけない
- TLS検証無効化、--insecure、strict-ssl=false は禁止
- 変更時は build/test/validate 方法も合わせて示す
```

```
# 実装依頼

## 目的
GitHub Actions 上で、main ブランチ push 時に npm package を社内registryへ publish する workflow を作成してください。

## 入力
- registry_url: https://npm.example.co.jp/repository/npm-private/
- npm_token: GitHub Secrets の NPM_TOKEN
- ca_cert_path: /usr/local/share/ca-certificates/internal-ca.crt

## トリガー
- push: main

## 出力
- .github/workflows/publish.yml
- 必要であれば .npmrc のサンプル
- 必要 secrets の一覧

## 外部連携
- 社内NPM registry に token 認証
- TLS は社内CA証明書を使って検証必須

## 技術制約
- Node 20
- npm 10
- bash 利用可
- set -euo pipefail を使う
- ログに secret を出力しない

## 禁止事項
- strict-ssl=false
- npm config set registry をグローバル汚染する書き方
- token の直書き
- --insecure
- 証明書未設定のまま接続すること

## 受け入れ条件
- main 以外では publish しない
- 社内CA証明書前提で成功する
- 失敗時に原因が分かるログになる
- 認証情報が露出しない

## 出力時の要望
- まず workflow 全体を提示
- 次に各 step の意図を説明
- 最後に必要 secrets と注意点を列挙
```