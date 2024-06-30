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
  - ```
    permissions:
       contents: read
    packages: write
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
    name: Environment variables
    on: push
    jobs:
      run:
        runs-on: ubuntu-latest
        env:
          BRANCH: main                # ジョブレベルで環境変数を定義
        steps:
          - run: echo "${BRANCH}"     # シェルコマンドからジョブレベルの環境変数を参照
          - uses: actions/checkout@v4
            with:
              ref: ${{ env.BRANCH }}  # envコンテキスト経由でジョブレベルの環境変数を参照
    ```
- 環境変数のオーバーライド
  - ```
    name: Override environment variables
    on: push
    env:
      EXAMPLE: Defined by workflow level     # ワークフローレベルで環境変数を定義
    jobs:
      print:
        runs-on: ubuntu-latest
        steps:
          - run: echo "${EXAMPLE}"           # ワークフローレベルの環境変数を出力
          - env:
              EXAMPLE: Defined by step level # ステップレベルで環境変数をオーバーライド
            run: echo "${EXAMPLE}"           # オーバーライドされた環境変数を出力
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

