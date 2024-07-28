# Kubernetes の役割
- 理想状態をファイルで宣言
- 理想状態に合致するようKubernetesがコンテナを管理
- 過去の状態を宣言し状態を宣言し状態を戻すことが可能
- ![image](https://github.com/user-attachments/assets/d87fe606-00dd-44a0-91d8-4c1f332064d1)
- ![image](https://github.com/user-attachments/assets/3e9048fd-abc8-4bbb-8a2f-aa75eb281e16)

# コントロールプレーン例
- kube-controller-manager: **コンテナの実行数維持はじめ基本的な管理**を担うコントローラ
- kube-scheduler: **コンテナのデプロイ時にそのスケジューリング**（実行ノードの選択）を行うスケジューラ
- kube-apiserver: **管理情報の紹介や変更要求**を HTTP API で受ける

# kubectl コマンド使用例
- nginx のデプロイ
  - `kubectl create deployment nginx-deployment --image=nginx:1.25`
    - ![image](https://github.com/user-attachments/assets/ed7ca520-5d86-4192-8bb0-cc9ea040490d)
- Deploymentが作成されるか確認
  - `kubectl get deployments`
- Deploymentの削除
  - `kubectl delete deployment [Deployment名]`
- コンテナ（Pod）が1つ実行される
  - `kubectl get pods`

# kubernetes マニフェストを使ってデプロイ
- マニフェストファイル
  - ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 1
      selector:
       matchLabels:
         app: nginx
      template:
       metadata:
         labels:
          app: nginx
       spec:
         containers:
         - name: nginx
          image: nginx:1.25  # nginxイメージをコンテナとして実行 
          ports:
          - containerPort: 80
    ```
- マニフェストを使ってデプロイ
  - `kubectl app;y -f nginx-deployment.yaml`
- 稼働状況を確認
  - `kubectl get deploy,pods`
  - ```
    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-deployment   1/1     1            1           7s
    NAME                                    READY   STATUS    RESTARTS   AGE
    pod/nginx-deployment-79b55879bb-wj464   1/1     Running   0          7s
    ```
