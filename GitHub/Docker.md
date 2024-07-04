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
