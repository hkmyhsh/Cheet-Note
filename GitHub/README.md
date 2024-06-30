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

# ジョブフロー制御
- ジョブが失敗しても後続のジョブを実行したい
  - `if: ${{ always() }}'

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
