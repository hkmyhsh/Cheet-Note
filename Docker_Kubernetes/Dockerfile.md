# 基本的な文法
- COPY命令：**コンテキストなどからイメージへ**、ファイルをコピーする（ルートファイルシステムへの変更）
- RUN命令：**イメージ内でシェルコマンドを実行**する（ルートファイルシステムへの変更）
- ENV命令：**後続命令やビルド結果のイメージの実行時に設定される環境変数を指定**する（実行時設定に対する変更）
- ENTRYPOINT命令：**イメージをコンテナとして実行するときにコンテナ内で実行するコマンドを指定**する（実行時設定に対する変更）

# Dockerfile例
- ```
  FROM ubuntu:22.04
  ENV DEBIAN_FRONTEND=noninteractive
  RUN apt-get update && apt-get install -y figlet
  COPY ./hello.sh /hello.sh
  ENTRYPOINT [ "/hello.sh" ]
  ```
- 処理内容
  - **FROM**命令により、ubuntu:22.04というイメージをDocker Hubからpullしベースイメージとして用いる
  - **ENV**命令により、環境変数「DEBIAN_FRONTEND」を設定し、「apt-get」が、Dockerfileのサポートしない対話的機能（ダイアログなど）を実行しないようにする。
  - **RUN**命令により、ベースイメージ上でapt-getコマンドを実行し、figletをインストールする
  - **COPY**命令により、コンテキストからhello.shをコピーする
  - **ENTRYPOINT**命令により、コンテナを起動したらhello.shを実行するよう設定を施す
- ![image](https://github.com/user-attachments/assets/b130459d-75c1-4447-b747-69a4b32bb8e3)

# Dcokerfileからイメージのビルドと削除
- ビルド
  - `docker build -t [コンテナイメージ]:[コンテナタグ] [Dockerファイル配置ディレクトリ]`
- コンテナ実行と削除
  - `docker run --rm --name=[コンテナ名] [コンテナイメージ]:[コンテナタグ]`

