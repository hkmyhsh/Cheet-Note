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
ポート] [イメージ名]:[タグ]`
