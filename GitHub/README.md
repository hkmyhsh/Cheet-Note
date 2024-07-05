# 定義
- ジョブの定義
  - ```
    jobs:
       cd:
         name: cd
         runs-on: ubuntu-latest
    ```
- Github Actions の permissions 設定
  - ```
    GITHUB_TOKENの権限をデフォルトから変更したい場合、permissionsでワークフロー、ジョブ単位で権限を指定する。contentsなど、それぞれのスコープに対してread、write、noneの3種類の許可を与えられている。
    ```
  - リポジトリで設定した**デフォルトパーミッション**よりも優先される
  - **パーミッション記述時は、ソースコードの読み込みにも明示的な許可が必要**
    - `contents: read`
    - ```
      permissions:
        contents: read
        packages: write
      ```
  - パーミッションの**無効化**
    - `permissions: {}`
  - 主なパーミッションのスコープ
    - `contents`: コード
    - `packages`: パッケージ
    - `actions`: ワークフローの起動
    - `id-token`: OpenID Connect でクラウドプロバイダへのアクセス
  
- ワークフローのタイムアウト
  - ```
    defaults:
      lint:
        timeout-minutes: 5                            # タイムアウト
    ```
- デフォルトシェル
  - パイプエラーを拾えるように、Bash起動オプションを変更
  - ```
    defaults:                                         # デフォルトシェル
      run:
        shell: bash
    ```
- コミット追加時に、古いワークフローの実行を自動キャンセルする
  - ```
    concurrency:                                      # 自動キャンセル
      group: ${{ github.workflow }}-${{ github.ref }} # Concurrencyグループ
      cancel-in-progress: true
    ```

# ステップ間のデータ共有
- GITHUB_OUTPUT環境変数によるデータ共有
  - ```
    steps:
      - id: <step-id>                                     # ステップIDを設定
        run: echo "<key>=<value>" >> "${GITHUB_OUTPUT}" # GITHUB_OUTPUTへ書き出し
      - env:
        RESULT: ${{ steps.<step-id>.outputs.<key> }}   # stepsコンテキストから参照
        run: echo "${RESULT}"
    ```

# ジョブ間のデータ共有
- 環境変数 `outputs` キーを利用
- `needs: [before]` でジョブの逐次実行を制御
  - ```
    jobs:
      before:
        runs-on: ubuntu-latest
        steps:
          - id: generate                                   # ステップのID
            run: echo "result=Hello" >> "${GITHUB_OUTPUT}" # ステップレベルの出力値
        outputs:
          result: ${{ steps.generate.outputs.result }}     # ジョブレベルの出力値
      after:
        runs-on: ubuntu-latest
          needs: [before]                                    # 依存するジョブIDの指定
          steps:
          - env:
            RESULT: ${{ needs.before.outputs.result }}   # 依存ジョブの出力値を参照
            run: echo "${RESULT}"
    ```

# 手動実行ワークフロー
- ```
  name: Manual
  on:
    workflow_dispatch:                       # 手動実行イベント
      inputs:
        greeting:                            # 入力パラメータ名
          type: string                       # データ型（文字列）
          default: Hello                     # 入力パラメータのデフォルト値
          required: true                     # 入力パラメータの必須フラグ
          description: A cheerful word       # 入力パラメータの概要
  jobs:
    run:
      runs-on: ubuntu-latest
      steps:
        - run: echo "${{ inputs.greeting }}" # 入力パラメータ「greeting」の参照
  ```
  - choice 型による**列挙値**の指定
    - ```
      inputs:
        log-level:
          type: choice # 入力パラメータを特定の値に制限
          options:     # 受け付ける入力値を列挙
            - info
            - warn
            - error
      ```

# github コンテキストでよく使うプロパティ
- ワークフローの実行ID
  - `github.run_id`
- プルリクエストのソースブランチ
  - `github.head_ref`
- ステップのワーキングディレクトリ
  - `github.workspace`
- リポジトリ明確に（octcat/hello-world のような値）
  - `github.repository`
- リポジトリのオーナー名（octcat のような値）
  - `github.repository_owner`
- トリガーになったイベント（オブジェクト）
  - `github.event`
- プルリクエストのイベントタイトル
  - `github.event.pull_request.title`
    - 実際の書き方例
      - `${{ github.event.pull_request.title }}`
- プルリクエストの本文
  - `github.event.pull_request.body`
- Issue のタイトル
  - `github.event.issue.title`
- ブランチ名
  - `github.head_ref`
- ランナー名
  - `runner.name`
- OS（Linux / Windows / macOS）
  - `runner.os`
- CPUアーキテクチャ
  - `runner.arch`
- 一時ディレクトリのパス
  - `runner.temp`
- インストール済みツールのディレクトリパス
  - `runner.tool_cache`
- デバッグログが有効な場合のみ「1」をセット
  - `runner.debug`
- セッション名の設定例
  - `SESSION_NAME: gh-oidc-${{ github.run_id }}-${{ github.run_attempt }}`

# ワークフローをトリガーしたイベントやリポジトリの取得
- ワークフローが実行された
  - プルリクエストのブランチ名
    - `${{ github.head_ref }}`
  - プルリクエストの番号
    - `${{ github.event.pull_request.number }}`
  - プルリクエストの作成者
    - `${{ github.event.pull_request.user.login }}`
  - ステップのステータス表示
    - `${{ steps.[ステップのID].conclusion }}`
  - Organization に登録した変数の参照
    - `${{ vars.SOME_VALUE }}`
  - Organization に登録した変数の参照
    - `${{ secrets.SECRET_TOKEN }}`

# 環境変数の定義と参照
- 環境変数の定義
  - ```
    env:           # 環境変数の定義
      BRANCH: main # 「BRANCH」という名前の環境変数へ、「main」という値をセット
    ```
- 環境変数の参照
  - ```
    steps:
      - run: echo "${BRANCH}"     # シェルコマンドからジョブレベルの環境変数を参照
      - uses: actions/checkout@v4
        with:
          ref: ${{ env.BRANCH }}  # envコンテキスト経由でジョブレベルの環境変数を参照
    ```
- 環境変数のオーバーライド
  - ```
    steps:
      - run: echo "${EXAMPLE}"           # ワークフローレベルの環境変数を出力
      - env:
        EXAMPLE: Defined by step level # ステップレベルで環境変数をオーバーライド
        run: echo "${EXAMPLE}"           # オーバーライドされた環境変数を出力
    ```
- GITHUB_ENV 環境変数を利用
  - ```
    steps:
      - run: echo "RESULT=hello" >> "${GITHUB_ENV}" # GITHUB_ENVへ書き出し
      - run: echo "${RESULT}"                       # 通常の環境変数として参照
    ```

# Environments のデータ参照
- `settings` で `Environment viriables' と 'Environment secrets' の登録は別で必要
- Environments は `environment` キーを使ってジョブレベルで指定する
  - `environment: <environment-name>`
- Environments の利用例
  - ```
    name: Environments
    on:
      workflow_dispatch:
        inputs:
          environment-name:
            type: environment             # 入力パラメータでEnvironmentsを切り替え
            default: test
            required: false
            description: Environment name
    jobs:
      print:
        runs-on: ubuntu-latest
        environment: ${{ inputs.environment-name }} # 利用するEnvironmentsを指定
        env:
          USERNAME: ${{ vars.USERNAME }}            # Environment variablesの参照
          PASSWORD: ${{ secrets.PASSWORD }}         # Environment secretsの参照
        steps:
          - run: echo "${USERNAME}"
          - run: echo "${PASSWORD:0:1} ${PASSWORD#?}"
    ```

# GitHub API を実行するワークフロー
- GItHub CLI を利用する
- ```
  permissions:           # GITHUB_TOKENの権限を指定
    pull-requests: write # プルリクエストの書き込みを許可
    contents: read       # ソースコードの読み込みを許可
  steps:
    - uses: actions/checkout@v4
    - run: gh pr comment "${GITHUB_HEAD_REF}" --body "Hello, ${GITHUB_ACTOR}"
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub CLI用クレデンシャル
  ```

# 関数
- 文字列系
  - ```
    - run: printenv | grep '_FUNC'
        env:
          CONTAINS_FUNC: ${{ contains('Hello', 'ell') }}     # 「ell」を含むか
          STARTS_WITH_FUNC: ${{ startsWith('Hello', 'He') }} # 「He」で始まるか
          ENDS_WITH_FUNC: ${{ endsWith('Hello', 'lo') }}     # 「lo」で終わるか
          FORMAT_FUNC: ${{ format('{0}, {1}.', 'Hi', 'world') }} # フォーマット
          JOIN_FUNC: ${{ join(github.event.*.html_url, ', ') }}  # カンマで結合
    ```
- JSON
  - ```
    steps:
      - run: echo "${CONTEXT}"
        env:
          CONTEXT: ${{ toJSON(github) }} # githubコンテキストをJSON文字列でダンプ
    ```
- ハッシュ生成
  - `hashFiles()`: 引数のパスから、ファイルのハッシュ値を生成する
  - ```
    - run: echo "${HASH}"
        env:
          HASH: ${{ hashFiles('.github/workflows/*.yml') }} # ハッシュ値の生成
    ```

# ジョブフロー制御
- 特定条件でのみジョブを実行したい
  - ```
    steps:
      - run: echo "Hello"
        if: ${{ contains(github.run_id, '1') }} # ワークフロー実行IDで分岐
    ```
- 文字列比較結果で**trueの場合**ジョブを実行したい
  - `if: ${{ github.actor == 'octocat' }} # octocatアカウント以外は実行をスキップ`
- 手前のジョブが**成功したら**ジョブを実行したい
  - `if: ${{ success() }}`
- 手前のジョブが**失敗したら**ジョブを実行したい
  - `if: ${{ failure() }}`
- 手前のジョブが**キャンセルされたら**ジョブを実行したい
  - `if: ${{ cancelled() }}`
- 手前のジョブの**実行結果に関わらず**ジョブを実行したい
  - `if: ${{ always() }}`
- **パッチバージョン変更**の場合のみ実行したい
  - `if: ${{ steps.meta.outputs.update-type == 'version-update:semver-patch' }}`
- **開発環境向け**の場合のみ実行したい
  - ```
    if: >-
      ${{ steps.meta.outputs.dependency-type == 'direct:development' &&
      steps.meta.outputs.update-type != 'version-update:semver-major' }}`
    ```

# コンテキストによるフロー制御
- ジョブやステップの終了状態
  - success: 実行が成功した
  - failure: 実行が失敗した
  - cancelled: 実行をキャンセルした
  - skipped: 実行をスキップした
  - * コンテキストからもこの終了状態を取得できる
    - `steps` コンテキスト：ステップの終了状態を保持
      - `steps` コンテキストのオブジェクト
        - ```
          {
            "outcome": "failure",
            "conclusion": "success",
            "outputs": {
              "foo": "A step value"
            }
          }
          ```
    　　　　　- `outcome プロパティ`: **Continue on Error** 適用前の状態
    　　　　　- Continue on Error の有無に左右されない
      　　　　　- 実行結果の生情報と考えられる
    　　　　　- `conclusion` プロパティ: **Continue on Error** 適用前の状態
      　　　　　- Continue on Error が有効な場合、エラー発生時も「success」がセットされる
    - `needs` コンテキスト：（依存している）ジョブの終了状態を保持
      - `needs` コンテキストのオブジェクト
        - ```
          {
            "result": "success",
            "outputs": {
              "bar": "A job value"
            }
          }
          ```
- コンテキストとステータスチェック関数の併用
  - 実際のフロー制御の例
    - 2パターンのエラーが発生します。ステップ1でエラーになるか、ステップ2でエラーになるか
    - ロジック
      - ステップ1：終了ステータスが50%の確率でゼロ以外になり、エラー終了する
      - ステップ2：ステップ1が正常終了した場合のみ実行され、必ずエラー終了する
      - ステップ3：ステップ2が実行された場合のみ、メッセージを標準出力する
    - ```
      name: Flow control
      on: push
      jobs:
        run:
          runs-on: ubuntu-latest
          steps:
            - run: exit "$(( RANDOM % 2 ))"
            - run: exit 1
              id: error
            - run: echo "Catch error step"
              if: ${{ failure() && steps.error.outcome == 'failure' }}
      ```
      
# ジョブの並列実行と逐次実行
- **並列**実行
  - ```
    jobs: # jobsキーへ複数のジョブを定義すれば、並列に実行される
      first:
        runs-on: ubuntu-latest
        steps:
          - run: sleep 10 && echo "First job"
      second:
        runs-on: ubuntu-latest
        steps:
          - run: sleep 10 && echo "Second job"
      third:
        runs-on: ubuntu-latest
        steps:
          - run: sleep 10 && echo "Third job"
    ```
- **逐次**実行
  - `needs` キーがポイント
  - ```
    jobs:
      first:                   # 依存ジョブがないので最初に実行される
        runs-on: ubuntu-latest
        steps:
          - run: sleep 10 && echo "First job"
      second:                  # firstジョブのあとに実行される
        runs-on: ubuntu-latest
        needs: [first]         # needsキーへ依存するfirstジョブのIDを指定
        steps:
          - run: sleep 10 && echo "Second job"
      third:                   # secondジョブのあとに実行される
        runs-on: ubuntu-latest
        needs: [second]        # needsキーへ依存するsecondジョブのIDを指定
        steps:
          - run: sleep 10 && echo "Third job"
    ```
  - 複数ジョブへの依存定義も可能
    - `needs: [<job-id-1>, <job-id-2>, ...]`


# docker 関連で使いそうなコマンド
- Container registry の認証
  - `- run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin`
- docker ビルド
  - `- run: docker build -t ghcr.io/${{ github.repository }}:latest .`
- docker へのプッシュ
  - `- run: docker push ghcr.io/${{ github.repository }}:latest`

# Amazon ECS にDocker イメージをデプロイするワークフロー
- OIDC 連携により AWS の認証情報を取得する
  - ```
    uses: aws-actions/configure-aws-credentials@v4
       with:
         aws-region: ap-northeast-1
         role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    ```
- ECR に認証する
  - ```
    uses: docker/login-action@v3
       with:
         registry: ${{ secrets.AWS_ECR_URL }}
    ```
- Docker ビルドおよびプッシュを実行する
  - ```
    uses: docker/build-push-action@v5
       with:
         push: true
         tags: ${{ secrets.AWS_ECR_URL }}/${{ secrets.IMAGE_NAME }}:latest
    ```
- 新しいタスク定義を作る
  - ```
    id: render-task-def
       uses: aws-actions/amazon-ecs-render-task-definition@v1
       with:
         task-definition: task-definition.json
         container-name: ${{ secrets.CONTAINER_NAME }}
         image: ${{ secrets.AWS_ECR_URL }}/${{ secrets.IMAGE_NAME }}:latest
    ```
- ECS にデプロイする
  - ```
    uses: aws-actions/amazon-ecs-deploy-task-definition@v1
       with:
         task-definition: ${{ steps.render-task-def.outputs.task-definition }}
         service: ${{ secrets.ECS_SERVICE_NAME }}
         cluster: ${{ secrets.ECS_CLUSTER_NAME }}
    ```

# OIDCを用いてクラウドプロバイダーと接続する
- GitHub Actions から利用したいクラウドプロバイダーがOIDCに対応している場合に利用可能
- ```
  あらかじめワークフローがクラウドプロバイダーにアクセスした際に使用するクラウド内の権限と、
  クラウドプロバイダーがどういった条件のワークフローを受け入れるかをクラウドプロバイダー側で設定しておく
  ```
  - 1. GitHub OIDC Provider はジョブの開始時に OIDC トークンを生成
    2. ワークフローから GitHub OIDC Provider へリクエストを送信。
    3. ワークフローがレスポンスとしてOIDCトークンを取得
    4. ワークフローはクラウドプロバイダーへOIDCトークンを含むリクエストを送信
    5. クラウドプロバイダーは送られてきたリクエストを検証
    6. クラウドプロバイダーからあらかじめ設定された権限を持つアクセストークンを生成しワークフローへ送信
  - 「クラウドプロバイダーは送られてきたリクエストを検証」について
    - ここで検証するのはリクエストに含まれる **Subject claim** という文字列
      - 例: `repo:OWNER/REPO:ref:refs/heads/main` という Subject claim を含むリクエストが送信された場合
        - クラウドプロバイダーはリポジトリが `OWNER/REPO` であるか、かつmainブランチであるかを検証できる
- **Subject claim** に何を指定するかと、最後にワークフローへ渡されるアクセストークンが扱える
**クラウドプロバイダー内での権限の設定** が重要

# ジョブにおけるコンテナの利用
- ジョブコンテナ
  - `runs-on` と同じ階層で `container` 以下に設定を記述する
  - ```
    jobs:
      container-test-job:
        runs-on: ubuntsu-latest
        container:
         image: golang:1.22.2
        steps:
          - run: (..略..)
    ```
- サービスコンテナ
  - ランナーがジョブの開始時に立ち上げるコンテナ
    - サービスコンテナはDBへのアクセスを必要とするようなテストを実施する際に便利
  - ```
    jobs:
      test:
        runs-on: ubuntu-latest
        services:
          postgres:
            image: postgres
            ports:
              - 5432:5432
            credentials:
              username: ${{ secrets.registry_username }}
              password: ${{ secrets.registry_password }}
        steps:
          run: (..略..)
    ```

# キャッシュの保存と復元
- 例: トピックブランチではキャッシュを保存しないようにする
```
steps:
   - uses: actions/checkout@v4
   - uses: actions/cache/restore@v4
     with:
       path: |
         ./path-to-cache
       key: ${{ runner.os }}-${{ runner.arch }}-${{ github.sha }}
       restore-keys: |
         ${{ runner.os }}-${{ runner.arch }}-
   - run: echo "Do something"
   - uses: actions/cache/save@v4
     if: github.ref == 'refs/heads/main'
     with:
       path: |
         ./path-to-cache
       key: ${{ runner.os }}-${{ runner.arch }}-${{ github.sha }}
```

# 静的解析ワークフロー例
- ```
  - run: |                                    # 静的解析の実行
          docker run --rm -v "$(pwd):$(pwd)" -w "$(pwd)" rhysd/actionlint:latest
  ```

# ワークフローコマンド
- `echo` コマンド経由でランナーへ特殊な操作を指示できる
  - `::workflow-command parameter1=<data1>,parameter2=<data2>::<command value>`

# ログの出力
- デバッグログの出力（Enable debug logging が必要）
  - ```
    - run: echo "::debug::This is a debug log" # デバッグログ有効化時にのみ出力
    ```
- Bash のトレーシングオプション（**どんなコマンドを実行したか** / **その結果はどうなったか**）
  - `set -x` により有効化
  - 利用例
    - ```
      - run: |   # Bashのトレーシングオプションを有効化し、実行したコマンドをログ出力
          set -x
          date
          hostname
      ```
- ログのグループ化（実行結果画面で**ログが折り畳まれた状態**になる）
  - ワークフローコマンドによりログをグループ化可能
    - ```
      ::group::<group-name>
      ::endgroup::
      ```
    - ログのグループ化利用例
      - ```
        steps:
          - run: |
            echo "::group::Show environment variables" # ロググループ化の開始
            printenv
            echo "::endgroup::"                        # ロググループ化の終了
        ```
- ログの手動マスク（**クレデンシャルを動的に生成するケースで使われる**）
  - Secrets へ登録した値は、ログ出力時に自動でマスクされる
  - これと同じように**任意の値に対してマスクを実現する方法**
    - `::add-mask::<secret-value>`
    - 値のマスク例（PASSWORD環境変数の値が「***」へマスク）
      - ```
        steps:
          - run: echo "::add-mask::${PASSWORD}" # ログ出力時にマスク
          - run: echo "${PASSWORD}"
        ```

# ジョブのレポーティングについて
- ワークフローコマンドを利用する
- アノテーション（**シンプルなメッセージの出力向き**）
  - ジョブページにアノテーションが表示される
  - ```
    ::error::<message>
    ::warning::<message>
    ::notice::<message>
    ```
    - 利用例
      - ```
        - run: echo "::error::This is an error"    # Errorレベルのアノテーション
        - run: echo "::warning::This is a warning" # Warningレベルのアノテーション
        - run: echo "::notice::This is a notice"   # Noticeレベルのアノテーション
        ```
- ジョブサマリー（**テーブルやリストなどで出力したい場合**）
  - 例) マークダウン形式での出力
    - ```
      steps:
      - run: | # マークダウンテキストをジョブサマリーへ出力
          echo "## Example Title :rocket:" >> "${GITHUB_STEP_SUMMARY}"
          echo "- first line" >> "${GITHUB_STEP_SUMMARY}"
          echo "- second line" >> "${GITHUB_STEP_SUMMARY}"
      ```

# マトリックス
- 1つのジョブ定義で、複数のジョブを実行できる
  - `複数のOSでビルドしたい`場合などに効果的
- マトリックスは `matrix` キーで定義する
  - ```
    jobs:
      print:
        strategy:
          matrix:                                             # マトリックスの定義
            os: [ubuntu-latest, windows-latest, macos-latest] # osプロパティの定義
        runs-on: ${{ matrix.os }}                             # osプロパティの参照
        steps:
          - run: echo "${RUNNER_OS}"
            shell: bash
    ```
- **多次元マトリックス**
  - ```
    jobs:
      print:
        strategy:
          matrix:                                 # 多次元マトリックスの定義
            os: [ubuntu-latest, macos-latest]     # osプロパティの定義
            version: [18, 20]                     # versionプロパティの定義
        runs-on: ${{ matrix.os }}                 # osプロパティの参照
        steps:
          - uses: actions/setup-node@v4
            with:
            node-version: ${{ matrix.version }} # versionプロパティの参照
          - run: echo "${RUNNER_OS}" && node --version
    ```
- **組み合わせ条件の手動定義**
  - `include` キーを利用する
  - ```
    jobs:
      print:
        strategy:
          matrix:                  # 多次元マトリックスの定義
            include:               # 組み合わせ条件を手動で列挙
              - os: ubuntu-latest  # パターン1
                version: 20
              - os: macos-latest   # パターン2
                version: 18
        runs-on: ${{ matrix.os }}
        steps:
          - uses: actions/setup-node@v4
            with:
              node-version: ${{ matrix.version }}
          - run: echo "${RUNNER_OS}" && node --version
    ```
  - **動的なマトリックスの生成**
    - ```
      name: Dynamic matrix
      on: push
      jobs:
        prepare:
          runs-on: ubuntu-latest
          steps:
            - id: dynamic
              run: | # マトリックス定義をJSON文字列で記述
                json='{"runner": ["ubuntu-latest", "macos-latest"]}'
                echo "json=${json}" >> "${GITHUB_OUTPUT}"
          outputs:   # マトリックス定義のJSON文字列をジョブ出力値へセット
            matrix-json: ${{ steps.dynamic.outputs.json }}
        print:
          needs: [prepare]
          strategy:  # JSON文字列からfromJSON関数で動的にマトリックスを生成
            matrix: ${{ fromJSON(needs.prepare.outputs.matrix-json) }}
          runs-on: ${{ matrix.runner }}
          steps:
            - run: echo "${RUNNER_OS}"
      ```

# エラーハンドリング
- エラーを握りつぶして次の処理に進みたいとき
  - `continue-on-error` キーを使う
  - この時、途中でエラーが発生しても、**ワークフロー自体は正常終了扱いされる**
    - ```
      name: Continue on Error
      on: push
      jobs:
        failure:
          runs-on: ubuntu-latest
          steps:
            - run: exit 1             # 終了ステータスがゼロ以外なので、エラーが発生する
              continue-on-error: true # Continue on Errorが発生したエラーを握りつぶす
            - run: echo "Success"     # 素知らぬ顔で実行され、ワークフローは正常終了する
      ```
- マトリックスのフェイルファストの無効化
  - マトリックスを使うと、複数のジョブが並列に起動する
    - 途中でエラーが発生し、ジョブが停止することがあるが、この時、**他のジョブも止まる**
      - 他のジョブに影響を受けたくない場合
        - ```
          strategy:
            fail-fast: false
          ```
        - ```
          name: Fail-fast matrix
          on: push
          jobs:
            run:
              runs-on: ubuntu-latest
              strategy:
                fail-fast: false     # マトリックスのフェイルファストを無効化する
                matrix:
                  time: [10, 20, 30] # ジョブごとに異なる時間スリープさせる
              steps:
                - run: sleep "${SLEEP_TIME}" && exit 1
                  env:
                    SLEEP_TIME: ${{ matrix.time }}
          ```
