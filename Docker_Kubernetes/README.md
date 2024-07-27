# dockerコマンド操作
- Dockerfileの作成例
  - ```
    FROM ubuntu:22.04 ←――❶
    COPY ./hello.sh /hello.sh  ←――❷
    ENTRYPOINT ["/hello.sh"]  ←――❸
    EOF
    ```
    - FROM ubuntu:22.04で、ubuntu:22.04イメージを土台として新たなイメージを作成することを記述している
    - ❷COPY ./hello.sh /hello.shで、コンテキスト中のhello.shファイルをコンテナ中の/hello.shにコピーしている
    - ❸ENTRYPOINT [ "/hello.sh" ]で、コンテナを起動したらhello.shを実行するよう指示している
- イメージのビルド
  - `docker build -t [イメージ名]:[タグ] [Dockerfileのあるディレクトリ]`
- イメージに新たな名前を付与
  - `docker tag [元のイメージ名]:[元のタグ] [新しいイメージ名]:[新しいタグ]
- docker イメージの一覧表示
  - `docker image ls ([イメージ名]:[タグ])`
- イメージをレジストリからプル
  - `docker pull [イメージ名]:[イメージタグ]`
- イメージを Dockerレジストリ にプッシュ
  - `docker push [イメージ名]:[タグ]`
- コンテナ実行
  - `docker run -d --name [コンテナ名] [イメージ名]:[タグ]`
- コンテナの中身をのぞく
  - `docker exec -it [コンテナ名] /bin/bash`
- コンテナの停止
  - `docker stop [コンテナ名]`
- コンテナの削除
  - `docker rm [コンテナ名]`

# コンテナの実行方法
- ホストとコンテナ間での共有やデータの永続化
  - `docker run -it --name [コンテナ名] -v [ホスト側マウント元ディレクトリ]:[コンテナ側先ディレクトリ](:ro *読み取り専用) [イメージ名]:[タグ]`
  - `docker run -it --name [コンテナ名] -v [ホスト側マウント元ディレクトリ]:[コンテナ側先ディレクトリ] [イメージ名]:[タグ]　/bin/bash`
    - `:ro`指定しなければ書き込み可能
- 作成したボリュームを表示
  - `docker volume ls`
- コンテナのポートをホスト上で公開
  - `docker run -d --name [コンテナ名] -p [ホスト側のIP]:[ホスト側のポート]:[コンテナ側の
ポート] nginx:1.25
    - **コンテナへ紐付けされたホストのポート**を通じてコンテナ内で稼働するnginxにアクセスできる
      - `curl [ホスト側のIP]:[ホスト側のポート]`

# **docker compose** による複数のコンテナをまとめて管理
- **Compose ファイル** で指定されたビルドを実行する
  - `docker compose build`
- **Compose ファイル** に記載されたコンテナを起動する
  - `docker compose up`
- **Compose ファイル** に記載されたコンテナを停止し削除する
  - `docker compose down`
- **Compose ファイル** の例
  - ```
    services:
      wordpress:  # wordpressコンテナの定義 
        image: wordpress:6.3
        restart: always
        ports:
          - 127.0.0.1:8080:80  # コンテナのポートをホストに紐付け 
        environment:  # 環境変数の指定 
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: exampleuser
          WORDPRESS_DB_PASSWORD: examplepass
          WORDPRESS_DB_NAME: exampledb
        volumes:
          - wordpress:/var/www/html  # ボリュームをコンテナ内にマウント 
      db:  # dbコンテナの定義 
        image: mariadb:11.1
        restart: always
        environment:  # 環境変数の指定 
          MYSQL_DATABASE: exampledb
          MYSQL_USER: exampleuser
          MYSQL_PASSWORD: examplepass
          MYSQL_RANDOM_ROOT_PASSWORD: '1'
        volumes:
          - db:/var/lib/mysql  # ボリュームをコンテナ内にマウント 
    volumes:  # 各コンテナが使うボリュームの定義 
      wordpress:
      db:
    EOF
    ```
  - ![image](https://github.com/user-attachments/assets/050c1cd8-5abf-49bf-b593-c8ee315e3ac8)
