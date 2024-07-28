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

# マルチステージビルド
- 1つのDockerfileに複数の**FROM**命令を記述すること
  - **複数のビルド**をまとめることもできる
  - 各**FROM**命令から始まるひとまとまりのビルド手順は**ステージ**と呼ばれ、**それぞれ独立にビルド**される
  - 各ステージには名前を付与でき、これには「**FROM イメージ名 AS 名前**」という命令を使います。
  - **ステージ間を連携させる**こともできる

# マルチステージビルドを使用するDockerfile例
- `hello` を出力する Goソースコード `hell.go`
  - ```
    package main
    import "fmt"
    func main() {
        fmt.Println("hello"）  「hello」を出力 
    }
    ```
- Goプログラムコンパイル用の `go.mod`
  - ```
    module hello
    go 1.21.3
    ```
- **特定のステージを指定してビルド**
  - `docker build -t hello-go-dev --target dev .`
- ```
  FROM golang:1.21.3 AS dev  ソースコードをコンパイルするためのステージ 
  COPY . /root/hello/
  RUN go build -o /hello /root/hello/hello.go
  FROM scratch  コンパイル結果の実行ファイルだけを含む、実行用のステージ 
  COPY --from=dev /hello .
  ENTRYPOINT [ "/hello" ]
  ```
  - 「**COPY --from=ステージ名**」という命令で、**あるステージからそのCOPY命令が実行されるステージへファイルをコピー**する
  - ![image](https://github.com/user-attachments/assets/46a4672e-a714-48c1-86a6-fe34534dc77a)

# 並列化されるDockerfile例
- ```
  FROM golang:1.21.3 AS dev
  COPY . /root/hello/
  RUN go build -o /hello  /root/hello/hello.go

  FROM ubuntu:22.04
  ENV DEBIAN_FRONTEND=noninteractive
  RUN apt-get update && apt-get install -y figlet
  COPY --from=dev /hello .
  ENTRYPOINT [ "/bin/sh", "-euc", "/hello | figlet" ]
  ```
  - **BuildKit**はDockerfileに記載の命令群の依存関係を分析し、**依存関係にない命令群を可能な限り並列に実行する**ことで、**ビルド時間を短縮**しようとする
    - **BuildKit**は Docker**v23.0.0** からdocker buildのバックエンドとして用いられているビルダ
  - ![image](https://github.com/user-attachments/assets/95e56c09-6ed9-4383-b813-1212a128489d)
