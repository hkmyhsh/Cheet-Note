# デバッグの仕方
- 文法チェック
  - `bash -n [対象のシェルスクリプト]`
- 実行内容を表示しながら実行する
  - `bash -x [対象のシェルスクリプト]`

# 複数のサーバに接続する
```
#!/usr/bin/env bash

# 接続先の設定
LIST="user@192.168.1.1
user@192.168.2.2
user@192.168.3.3"

# 引数のチェック
if [ -z "$1" ] # 引数があるかどうかをチェック
then
  echo "引数がありません" # 引数がない場合のメッセージ
  echo "例 $0 ls -l"
  exit 1 # シェルスクリプトの終了
fi

# 「接続してコマンドを実行する」をループで繰り返す
# パスワードがない場合
for TARGET in $LIST
do
  echo "----- $TARGET -----" # 接続先の表示
  ssh $TARGET "$@" # 接続先に対して、接続後コマンドを実行
done
```

# サービスの稼働状況を確認
```
#!/usr/bin/env bash

# チェック先の設定
# チェック先の@[IPアドレス（またはホスト名）]:[ポート番号]の形式で列挙する
# ポート番号443: httpサービス
LIST="www.shoeisha.co.jp:443
raintrees.net:80
raintrees.net:8080
192.168.1.1:443
sample.example.com:443"

#「チェック先に接続できるか確認する」をチェック先分ループで繰り返す
for TARGET in $LIST
do
  echo "----- $TARGET -----" # 接続先の表示
  nc -w 1 -z ${TARGET//:/ } # チェック先に接続できるかncで確認
  if [$? -eq 0 ] # ncの実行結果を判定、0が正常終了
    then
      echo "◯ サービスが稼働しています"
    else
      echo "× サービスが稼働していません"
  fi
done
```
