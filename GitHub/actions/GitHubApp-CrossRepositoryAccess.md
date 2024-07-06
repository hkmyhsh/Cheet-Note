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
            uses: actions/create-github-app-token@v1
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
