# Reusable Workflow
- パーミッション
  - 定義をスキップすることも可能
    - その場合は呼び出し側ワークフローのパーミッションを暗黙的に継承する
    - 利用者の認知負荷が上がるため**省略は非推奨**
- GIRHUB_TOKEN のパーミッション
  - **呼び出し側ワークフローに制約される**
- `github.event` プロパティ
  - 呼び出し側のワークフローがどのイベントで起動されているか制御できないため、**`github.event` プロパティで意図した値が取れる保証がない**
-Secrets
  - **呼び出し側ワークフローの `Secrets`** を直接参照できない
    - 例外: `GITHUB_TOKEN`
    - `inherit` キーワードを使うと、呼び出し側の `Secrets` を Reusable Workflow がまとめて継承できる
      - ```
        uses: ./.github/workflows/reusable-inherit.yml
        secrets: inherit # Secretsをまとめて継承できる
        ```
  - **Ruesable Workflows の環境変数**
    - デフォルト環境変数を除き、呼び出し側ワークフローの環境変数は Reusable Workflow から参照できない
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

# Reusable Workflows のジョブ定義
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

# Reusable Workflows 呼び出しワークフロー
- 記載例
  - ```
    name: Call
    on: pull_request
    jobs:
      call:
        uses: ./.github/workflows/reusable-workflows.yml # Reusable Workflowsの指定
        with:                                            # 平文の入力パラメータ指定
          pr-number: ${{ github.event.pull_request.number }}
        secrets:                                         # Secretsの入力パラメータ指定
          token: ${{ secrets.GITHUB_TOKEN }}
        permissions:
          contents: read
          pull-requests: write
      print:
        needs: [call]
        runs-on: ubuntu-latest
        steps:
          - run: echo "Result> ${MESSAGE}"
            env:
              MESSAGE: ${{ needs.call.outputs.message }} # 出力値の参照
    ```
