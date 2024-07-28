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

# kubectlコマンド使用例
- nginx のデプロイ
  - `kubectl create deployment nginx-deployment --image=nginx:1.25`
    - ![image](https://github.com/user-attachments/assets/ed7ca520-5d86-4192-8bb0-cc9ea040490d)
- Deploymentが作成されるか確認
  - `kubectl get deployments`
- Deploymentの削除
  - `kubectl delete deployment [Deployment名]`
- コンテナ（Pod）が1つ実行される
  - `kubectl get pods`
- pod作成
  - `kubectl apply -f [podのマニフェストファイル]`
- pod内のファイルを確認
  - `kubectl exec -it example-pod -c alpine -- wget -qO - localhost:80/date.json | head -n 3`
- podの削除
  - `kubectl delete -f [podのマニフェストファイル]`

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
          app: nginx # ラベルでPodを絞り込み
      template:
        metadata:
          labels:
            app: nginx # 各Podラベル付与
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

# Pod
- Kubernetesにおける基本的なデプロイ単位
- 一つ以上のコンテナ群をまとめたもの
- ![image](https://github.com/user-attachments/assets/ea7ce874-1a15-40e1-b5bc-4413425e9b4d)
- マニフェスト例
  - ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: example-pod
    spec:
      containers:
      # 共有ボリュームのデータを80番ポートで公開するnginxコンテナ 
    - name: nginx
      image: nginx:1.25
      ports:
      - containerPort: 80
      # コンテナ間で共有するボリュームを「/usr/share/nginx/html/」にマウント 
      volumeMounts:
      - mountPath: /usr/share/nginx/html/
      name: docroot
      # 共有ボリュームにデータを書き込むコンテナ 
      - name: alpine
        image: alpine:3.18
        command: ["sh"]
        args:
        - -euc
        - |
          for i in $(seq 1 10) ; do
            echo '{"date": "'$(date)'"}' >> /mnt/date.json
            sleep 3
            done ; sleep infinity
        # コンテナ間で共有するボリュームを「/mnt」にマウント 
        volumeMounts:
        - mountPath: /mnt/
          name: docroot
        # コンテナ間で共有するボリューム(後述するemptyDirを利用) 
      volumes:
      - name: docroot
        emptyDir:
          sizeLimit: 500Mi
    ```
  - 1つめは nginx を実行するコンテナ: 80番ポートでHTTP接続を受ける
  - 2つめは alpine コンテナ: シェルスクリプトを実行し、一定時間毎にタイムスタンプをファイルに書き込む
    - 2つのコンテナは `docroot` と名付けたボリュームを共有する* Kubernetesを管理するストレージ領域
    - このボリュームを介して、alpineが書き込むデータがnginxコンテナに共有され、nginxサーバがそれを80番で公開する

# ラベルとアノテーション
- PodなどのKubernetesの**管理対象をグルーピング / 追加情報を付与**する機能
- 例:![image](https://github.com/user-attachments/assets/59095450-aec9-4b98-9af5-87ceb0799289)
