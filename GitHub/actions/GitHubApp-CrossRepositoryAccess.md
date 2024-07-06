# クロスリポジトリアクセス
- ワークフローを実行したリポジトリ以外のリポジトリに `GitHub App` を利用してアクセスする
  - ```
    name: Cross repo
    on: push
    env:
      TARGET_REPO: another-repo # GitHub Appsのインストール時に設定したリポジトリ
    jobs:
      checkout:
        runs-on: ubuntu-latest
        steps:
          - id: create                                 # GitHub Appsトークンの生成
            uses: actions/create-github-app-token@7bfa3a4717ef143a604ee0a99d859b8886a96d00 # v1.9.3
            with:
              app-id: ${{ secrets.APP_ID }}
              private-key: ${{ secrets.PRIVATE_KEY }}
              repositories: ${{ env.TARGET_REPO }}
          - uses: actions/checkout@v4                  # 別リポジトリのチェックアウト
            with:
              repository: ${{ github.repository_owner }}/${{ env.TARGET_REPO }}
              path: ${{ env.TARGET_REPO }}
              token: ${{ steps.create.outputs.token }}
          - run: cat "${TARGET_REPO}/README.md"        # 別リポジトリのREADMEを表示
    ```
- `GitHub Apps` トークン生成アクション
  - `actions/create-github-app-token` アクション
    - 'App ID' と 'GitHub Apps' の秘密鍵を使って、'GitHub Apps' トークンを生成します。
  - ```
    - id: create
      uses: actions/create-github-app-token@v1
      with:
        app-id: ${{ secrets.APP_ID }}           # GitHub AppsのApp ID
        private-key: ${{ secrets.PRIVATE_KEY }} # GitHub Appsの秘密鍵
        repositories: ${{ env.TARGET_REPO }}    # アクセスを許可するリポジトリ
    ```
  - 【備考】シェルスクリプトでもトークン生成実装
    - `GitHub API` の認証には `JWT` が必要
      - Base64URL エンコード関数と署名関数を定義する必要がある
        - 関数の実行には `OpenSSL` が必要
      - **JWT**
        - ヘッダー
          - ```
            {
              "alg": "RS256",
              "typ": "JWT"
            }
            ```
          - Base64UR Lエンコード
            - `header="$(printf '{"alg":"RS256","typ":"JWT"}' | base64url)"`
        - ペイロード
          - ```
            {
             "iss": "<App ID>",
             "iat": "<作成日時>",
             "exp": "<有効期限>"
            }
            ```
          - Base64URL エンコード
            - ```
              template='{"iss":"%s","iat":%s,"exp":%s}'
              payload="$(printf "${template}" "${APP_ID}" "${iat}" "${exp}" | base64url)"
              ```
        - 署名の生成 及び Base64 エンコード
          - `signature="$(printf '%s' "${header}.${payload}" | sign | base64url)"`
        - JWT の組み立て
          - `jwt="${header}.${payload}.${signature}"`
    - 大まかな処理フロー
      1. App IDと秘密鍵を使って、GitHub APIのアクセスに使うJWTを作成する
      2. Installation APIを実行し、Installation IDを取得する
      3. Access Tokens APIを実行し、GitHub Appsトークンを取得する
    - ```
      #!/usr/bin/env bash

      # Base64 へのエンコード関数
      base64url() {
        openssl enc -base64 -A | tr '+/' '-_' | tr -d '='
      }

      # SHA-256 アルゴリズムを利用して GitHub Apps へ登録した秘密鍵を使用して署名する
      sign() {
        openssl dgst -binary -sha256 -sign <(printf '%s' "${PRIVATE_KEY}")
      }

      # JWTの生成
      header="$(printf '{"alg":"RS256","typ":"JWT"}' | base64url)"
      now="$(date '+%s')"
      iat="$((now - 60))"
      exp="$((now + (3 * 60)))"
      template='{"iss":"%s","iat":%s,"exp":%s}'
      payload="$(printf "${template}" "${APP_ID}" "${iat}" "${exp}" | base64url)"
      signature="$(printf '%s' "${header}.${payload}" | sign | base64url)"
      jwt="${header}.${payload}.${signature}"

      # Installation APIの実行
      repo="${GITHUB_REPOSITORY_OWNER}/${TARGET_REPO}"
      installation_id="$(curl --location --silent --request GET \
        --url "${GITHUB_API_URL}/repos/${repo}/installation" \
        --header "Accept: application/vnd.github+json" \
        --header "X-GitHub-Api-Version: 2022-11-28" \
        --header "Authorization: Bearer ${jwt}" \
       | jq -r '.id'
      )"

      # Access Tokens APIの実行
      token="$(curl --location --silent --request POST \
        --url "${GITHUB_API_URL}/app/installations/${installation_id}/access_tokens" \
        --header "Accept: application/vnd.github+json" \
        --header "X-GitHub-Api-Version: 2022-11-28" \
        --header "Authorization: Bearer ${jwt}" \
        --data "$(printf '{"repositories":["%s"]}' "${TARGET_REPO}")" \
        | jq -r '.token'
      )"
      echo "token=${token}" >>"${GITHUB_OUTPUT}"
      ```
