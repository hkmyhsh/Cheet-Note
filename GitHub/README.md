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
    permissions:
       contents: read
    packages: write
    ``` 

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
