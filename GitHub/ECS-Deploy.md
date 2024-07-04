# コンテナビルドアクション
1. Amazon ECR へログイン
2. デプロイ対象コンテナイメージのメタデータ生成
3. デプロイ対象コンテナイメージをプッシュ
```
name: Container Build
description: コンテナイメージをビルドし、ECRへプッシュします。
inputs:
  ecr-repository-uri:
    required: true
    description: ECRリポジトリのURI
  dockerfile-path:
    required: true
    description: Dockerfileのパス
outputs:
  container-image:
    value: ${{ steps.meta.outputs.tags }}
    description: ビルドしたコンテナイメージ
runs:
  using: composite
  steps:
    - uses: aws-actions/amazon-ecr-login@v2 # Amazon ECRへのログイン
    - uses: docker/metadata-action@v5       # コンテナイメージのメタデータ生成
      id: meta
      with:
        images: ${{ inputs.ecr-repository-uri }}
        tags: type=sha,format=long
    - uses: docker/build-push-action@v5     # コンテナイメージのビルドとプッシュ
      with:
        push: true
        context: ${{ inputs.dockerfile-path }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
```

# コンテナデプロイアクション
1. 現在のタスク定義を取得
2. タスク定義の image 部分を、デプロイ対象のコンテナイメージに書き換え
3. 書き換えられたタスク定義を使うよう、ECSサービスを更新
```
name: Container Deploy
description: ECSサービスを更新し、コンテナをデプロイします。
inputs:
  ecs-cluster:
    required: true
    description: ECSクラスター
  ecs-service:
    required: true
    description: ECSサービス
  task-definition:
    required: true
    description: タスク定義
  container-name:
    required: true
    description: コンテナ名
  container-image:
    required: true
    description: コンテナイメージ
runs:
  using: composite
  steps:
    - run: |                                                 # タスク定義の取得
        aws ecs describe-task-definition --task-definition "${TASK_DEFINITION}" \
        --query taskDefinition --output json > "${RUNNER_TEMP}/task-def.json"
      env:
        TASK_DEFINITION: ${{ inputs.task-definition }}
      shell: bash
    - uses: aws-actions/amazon-ecs-render-task-definition@v1 # タスク定義の修正
      id: render
      with:
        task-definition: ${{ runner.temp }}/task-def.json
        container-name: ${{ inputs.container-name }}
        image: ${{ inputs.container-image }}
    - uses: aws-actions/amazon-ecs-deploy-task-definition@v1 # ECSサービスの更新
      with:
        cluster: ${{ inputs.ecs-cluster }}
        service: ${{ inputs.ecs-service }}
        task-definition: ${{ steps.render.outputs.task-definition }}
```

# デプロイワークフロー
1. ソースコードのチェックアウト
2. AWS の一時クレデンシャルを取得
3. コンテナビルドアクションを実行し、コンテナイメージをプッシュ
4. コンテナデプロイアクションを実行し、コンテナを入れ替え
```
name: Deploy
on: workflow_dispatch
env:
  ROLE_ARN: arn:aws:iam::${{ secrets.AWS_ID }}:role/${{ secrets.ROLE_NAME }}
  SESSION_NAME: gh-oidc-${{ github.run_id }}-${{ github.run_attempt }}
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v4                      # チェックアウト
      - uses: aws-actions/configure-aws-credentials@v4 # 一時クレデンシャルの取得
        with:
          role-to-assume: ${{ env.ROLE_ARN }}
          role-session-name: ${{ env.SESSION_NAME }}
          aws-region: ap-northeast-1
      - uses: ./.github/actions/container-build/       # コンテナイメージのビルド
        id: build
        with:
          ecr-repository-uri: ${{ vars.ECR_REPOSITORY_URI }}
          dockerfile-path: docker/ecs/
      - uses: ./.github/actions/container-deploy/      # コンテナのデプロイ
        with:
          ecs-cluster: ${{ vars.ECS_CLUSTER_NAME }}
          ecs-service: ${{ vars.ECS_SERVICE_NAME }}
          task-definition: ${{ vars.TASK_DEFINITION_NAME }}
          container-name: ${{ vars.CONTAINER_NAME }}
          container-image: ${{ steps.build.outputs.container-image }}
```
