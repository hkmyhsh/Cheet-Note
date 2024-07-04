# Reusable Workflow
- Reusable Workflowsを起動するイベント
  - `workflow_call` を指定するとReusable Workflowsとして自動的に認識される。
  - ```
    on:
      workflow_call
    ```
- Reusable Workflowsの入力パラメータ定義
  - 入力パラメータが2種類ある
    - `inputs`：平文を入力パラメータで指定
    - `secrets`：Secretsを入力パラメータで指定
  - `inputs` キー
    - プロパティ
      - type：データ型（boolean/number/string）
      - default：入力パラメータ省略時のデフォルト値
      - required：入力パラメータの指定は必須か（trueなら必須）
      - description：入力パラメータの概要
    - 利用例
      - ```
        inputs:
          pr-number:
            type: string                                     # 入力パラメータのデータ型
            default: ${{ github.event.pull_request.number }} # 入力パラメータのデフォルト値
            required: false                                  # 入力パラメータの必須フラグ
            description: プルリクエスト番号                      # 入力パラメータの概要
        ```
  - `secrets` キー
    - プロパティ
      - required：Secrets入力パラメータの指定は必須か（trueなら必須）
      - description：Secrets入力パラメータの概要
    - 利用例
      - ```
        secrets:
          token:
            required: true             # Secrets入力パラメータの必須フラグ
            description: GitHubトークン  # Secrets入力パラメータの概要
        ```
  - `outputs` キー
    - プロパティ
      - value：出力する値
      - description：出力値の概要
    - 利用例
      - ```
        outputs:
          message:
            value: ${{ jobs.comment.outputs.pr-comment }} # 出力する値
            description: メッセージ                         # 出力値の概要
        ```

# Reusable Workflowsのジョブ定義
- 定義
  - 下記のような項目を通常通り定義可能
    - ランナー
    - パーミッション
- ロジック
  - ステップレベルで出力したい値を、GITHUB_OUTPUT環境変数へ書き出す
  - ジョブレベルのoutputsキーへ、ステップレベルの出力値をセットする
  - ワークフローレベルのoutputsキーへ、ジョブレベルの出力値をセットする
- 設定例
  - ```
    name: Reusable Workflows
    on:
      workflow_call: # Reusable Workflowsを起動するイベント
        inputs:      # Reusable Workflowsの入力パラメータ定義（平文）
          pr-number:
            type: string
            default: ${{ github.event.pull_request.number }}
            required: false
            description: プルリクエスト番号
        secrets:     # Reusable Workflowsの入力パラメータ定義（Secrets）
          token:
            required: true
            description: GitHubトークン
        outputs:     # Reusable Workflowsの出力値定義
          message:
            value: ${{ jobs.comment.outputs.pr-comment }}
            description: メッセージ
    jobs:            # Reusable Workflowsのジョブ定義
      comment:
        runs-on: ubuntu-latest
        permissions:
          contents: read
          pull-requests: write
        steps:
          - uses: actions/checkout@v4
          - id: pr-comment
            run: |
              body="Welcome, ${GITHUB_ACTOR}"
              gh pr comment "${PR_NUMBER}" --body "${body}"
              echo "body=${body}" >>"${GITHUB_OUTPUT}"
            env:
              PR_NUMBER: ${{ inputs.pr-number }}
              GITHUB_TOKEN: ${{ secrets.token }}
        outputs:
          pr-comment: ${{ steps.pr-comment.outputs.body }}
    ```
