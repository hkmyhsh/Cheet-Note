# プルリクエスト作成アクション
- inputs: message（コミットメッセージ）
- outputs: branch（プルリクエストのブランチ）
- アクション例の仕様
  - ワークフロー実行中に変更したファイルを、まとめてコミットする
  - どこで実行してもいいように、コミットにはボットアカウントを利用する
  - プルリクエストのタイトルやブランチ名は自動生成する
- アクションのロジック
  - Git設定ファイル: ボットアカウントで Git を操作出来るよう設定する
  - リポジトリ操作: 変更したファイルをすべてコミットしてプッシュする
  - プルリクエスト作成: GitHub CLI でプルリクエストを作成する
- 【注意】**ボットアカウント**
  - `github-actions[bot]`: 実在するのボットアカウント
  - ```
    USERNAME: github-actions[bot]
    EMAIL: 41898282+github-actions[bot]@users.noreply.github.com
    ```
```
name: Create PR
description: |
  変更したファイルをコミットし、プルリクエストを作成します。
  プルリクエストのタイトルや本文には、コミットメッセージがそのまま利用されます。
  パーミッションに「contents: write」と「pull-requests: write」が必要です。
inputs:
  message:                                # アクションの入力
    required: true
    description: コミットメッセージ
outputs:
  branch:                                 # アクションの出力
    value: ${{ steps.pr.outputs.branch }}
    description: プルリクエストのブランチ
runs:
  using: composite
  steps:
    - id: pr
      shell: bash
      env:
        USERNAME: github-actions[bot]
        EMAIL: 41898282+github-actions[bot]@users.noreply.github.com
        MESSAGE: ${{ inputs.message }}
        BRANCH: auto/${{ github.run_id }}/${{ github.run_attempt }}
        GITHUB_TOKEN: ${{ github.token }}
      run: |
        echo "branch=${BRANCH}" >> "${GITHUB_OUTPUT}"
        git config --global user.name "${USERNAME}"   # Gitのユーザー設定
        git config --global user.email "${EMAIL}"
        git switch -c "${BRANCH}"                     # コミットとプッシュ
        git add .
        git commit -m "${MESSAGE}"
        git push origin "${BRANCH}"                   # プルリクエストの作成（↓）
        gh pr create --head "${BRANCH}" --title "${MESSAGE}" --body "${MESSAGE}"
```
- アクションのテスト
  1. Setupフェーズ：テストの準備
  2. Exerciseフェーズ：テスト対象コードの実行
  3. Verifyフェーズ：実行結果の検証
  4. Teardownフェーズ：テストの後始末
- 今回使うテスト
  1. Setupフェーズ：プッシュ対象のダミーファイルを作成
  2. Exerciseフェーズ：「プルリクエスト作成アクション」を実行
  3. Verifyフェーズ：プルリクエストが正しく作成されたか検証
  4. Teardownフェーズ：プルリクエストのクローズとブランチの削除
- アクションのテストフロー
```
name: Test action
on: pull_request
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - name: Setup         # テストの準備
        run: date > foo.md
      - name: Exercise      # テスト対象コードの実行
        id: exercise
        uses: ./
        with:
          message: Test
      - name: Verify        # 実行結果の検証
        run: |
          set -x
            test "$(gh pr view "${BRANCH}" --json title --jq .title)" = "Test"
        env:
          BRANCH: ${{ steps.exercise.outputs.branch }}
      - name: Teardown      # テストの後始末
        if: ${{ always() }}
        run: |
          gh pr close "${BRANCH}" || true
          git push origin "${BRANCH}" --delete || true
        env:
          BRANCH: ${{ steps.exercise.outputs.branch }}
```
