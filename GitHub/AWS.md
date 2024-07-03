# OpenID Connect による GitHub と AWS 連携
- 前提
  - AWS が GitHub OIDC Provider を信頼するよう設定
    - OIDC トークンと一時クレデンシャルを交換するために必須
  - IAM ロールの作成
    - Assume Role ポリシー: アクセス元を規定するコンポーネント
    - IAM ポリシー: アクセス先を規定するコンポーネント
    - IAM ロール: `Assume Role ポリシー` と　`IAM ポリシー` を繋ぐコンポーネント
      - ```
        export GITHUB_REPOSITORY="<OWNER>/<REPO>"
        export PROVIDER_URL=token.actions.githubusercontent.com
        export AWS_ID=$(aws sts get-caller-identity --query Account --output text)
        export ROLE_NAME=github-actions
        ```
  - `AWS_ID`: AWS アカウント
  - `ROLE_NAME`: IAMロール名
  - AWS連携ワークフロー
    - ```
      name: OpenID Connect
      on: push
      env: # AWSの認証アクションへ指定する入力パラメータを環境変数として定義
        ROLE_ARN: arn:aws:iam::${{ secrets.AWS_ID }}:role/${{ secrets.ROLE_NAME }}
        SESSION_NAME: gh-oidc-${{ github.run_id }}-${{ github.run_attempt }}
      jobs:
        connect:
          runs-on: ubuntu-latest
          permissions:
            id-token: write                                  # OIDCトークンの取得を許可
          steps:
            - uses: aws-actions/configure-aws-credentials@v4 # AWSの認証アクション
              with:
                role-to-assume: ${{ env.ROLE_ARN }}
                role-session-name: ${{ env.SESSION_NAME }}
                aws-region: ap-northeast-1
            - run: aws iam list-users                        # 一時クレデンシャルの利用
            - run: aws iam create-user --user-name invalid || true
      ```

- AWS 連携のトラブルシューティングの例
  - `Credentials could not be loaded`
    - 事象
      - `id-token` スコープの書き込み権限がない
    - エラーメッセージ
      - ```
        Error: Credentials could not be loaded, please check your action inputs:
        Could not load credentials from any providers
        ```
    - 対応内容
      - ワークフローへパーミッション定義を追加する
  - `No OpenIDConnect provider found in your account`
    - `OpenID Connect Provider` が存在しない
    - エラーメッセージ
      - ```
        Error: Could not assume role with OIDC:
        No OpenIDConnect provider found in your account for ......
        ```
    - 対応内容
      - OpenID Connect Providerを作成する
