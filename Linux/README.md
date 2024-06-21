# シンプルなコマンド
- 直前のコマンドの実行結果を確認
  - `echo $?`
- 現在のシェルの表示
  - `echo $0`
- OS確認
  - `cat /etc/os-release`
- パッケージの更新とアップグレード
  - `sudo apt update && sudo apt -y upgrade` 
  - `sudo yum update && sudo yum -y upgrade`
- パッケージのインストール
  - `sudo apt -y install [パッケージ名]` 
  - `sudo yum -y install [パッケージ名]`
- エラーメッセージを出したくない時
  - `2> /dev/null` をコマンドの後につける
- エラーメッセージを収集したい場合
  - `2> [エラーを集める用のファイル]`
- １つの文字の置換
  - `echo $PATH | tr : "¥n"`
- 文字列を置換する
  - `sed's/[置換元の文字列]/[置換後の文字列]/g' [置換元ファイル]`
- 直前に作業していたディレクトリに移動する
  - `cd -`

# コマンドを調べる際
- `man [調べたいコマンド]`
- `info [調べたいコマンド]`
  - 「h」で使用方法の説明
  - 「/」または「s」で文書内検索

# 過去のコマンドを掘り起こす
- コマンド履歴の表示
  - `history`
- 最近の3つのコマンドを表示する
  - 'history 3'
- 履歴を一度に1画面ずつ表示する
  - `history | less`
  - `history | sort -nr | less`
- 特定文字だけ含んでいるコマンドだけ表示する
  - `history | grep -w [特定文字]`
- コマンド履歴の削除
  - `history -c`

# ログのERROR調査
- `journalctl --no-pager | grep -i ERROR`
- `grep -iR ERROR /var/log`

# sshコマンド
- 単純な接続
  - `ssh [接続先ユーザ名]@[接続先のIPアドレス or ホスト名]`
- 単純な接続 + コマンド実行
  - `ssh [接続先ユーザ名]@[接続先のIPアドレス or ホスト名] [接続先で実行するコマンド]`
- 単純な接続 + シェルファイル実行
  - `ssh [接続先ユーザ名]@[接続先のIPアドレス or ホスト名] sh [実行させたいスクリプト]`
- 個別にrootパスワードが必要な場合
  - `ssh [接続先ユーザ名]@[接続先のIPアドレス or ホスト名] "su - -c [実行したいコマンド]" <<< [rootパスワード]`

# seqコマンド
- 数値の範囲を指定して範囲全体を出力する
  - `seq 1 5`
- 2つずつ数値を増やす
  - `seq 1 2 10`
- 降順に数値を生成する
  - `seq 3 -1 0`
- 小数も可能
  - `seq 1.1 0.1 2`
- 数値間のセパレータを設定（スラッシュ区切り）
  - `seq -s/ 1 5`
- wオプションを追加すると先頭に「0」が追加されて文字幅が統一される
  - `seq -w 8 10`
    - ```
      08
      09
      10
      ```

# 一連の数値、文字出力（ブレース展開）
- 1から順に
  - `echo {1..10}`
    - `1 2 3 4 5 6 7 8 9 10`
- 10から逆に
  - `echo {10..1}`
    - `10 9 8 7 6 5 4 3 2 1`
- 「0」をつけて幅を統一
  - `echo {01..10}`
    - `01 02 03 04 05 06 07 08 09 10`
- 1から100ずつ
  - `echo {1..1000..100}`
    - `1 101 201 301 401 501 601 701 801 901`
- 1000から逆に
  - `echo {1000..1..100}`
    - `1000 900 800 700 600 500 400 300 200 100`
- 「0」をつけて幅を統一
  - `echo {01..1000..100}`
    - `0001 0101 0201 0301 0401 0501 0601 0701 0801 0901`
- 一連の文字出力
  - `echo {A..Z}`
    - `A B C D E F G H I J K L M N O P Q R S T U V W X Y Z`
- []と{}は違う
  - ls file[2-4]
    - `file2 file4`
  - ls file{2..4}
    - ```
      ls: cannot access `file3` : No such file or directory
      file2 file4
      ```

# dateコマンド
- デフォルトのフォーマット
  - `date`
    - `Mon Jun 28 16:57:33 EDT 2021`
- 「年-月-日」のフォーマット
  - `date +%Y-%m-%d`
    - `2021-06-28`
- 「時-分-秒」のフォーマット
  - `date +%H:%M:%S`
    - `16:57:33`
- 「曜日」のフォーマット
  - `date +%A`
    - `Tuesday`

# 日数の計算
- 1日後
  - `date -d "1 day"`
  - `date -d tomorrow`
- n日後
  - `date -d "n days"`
- n週後
  - `date -d "n weeks"
- n月後
  - `date -d "n months"
- n日前
  - `date -d "-n days"`
  - `date -d "n days ago"`
- n週前
  - `date -d "-n weeks"
  - `date -d "n weeks ago"`
- n月前
  - `date -d "-n months"
  - `date -d "n months ago"`
- 特定日からn日前
  - `date -d "n days ago　yyyy/mm/dd"`
- 特定日からn週前
  - `date -d "n weeks ago　yyyy/mm/dd"`
- 特定日からn月前
  - `date -d "n months ago　yyyy/mm/dd"`
- UNIX時間の確認
  - `date +%s`
- 特定日のUNIX時間
  - `date +%s -d yyyy/mm/dd`
- 特定日まで後何日か計算
  - `echo $((($(date +%s -d 'yyyy/mm/dd')-$(date +%s))/(60*60*24)))`

# 環境変数
- 設定
  - `export prof=~/prof.txt
- 既存のPATHに新たなディレクトリを追加
  - PATH="$PATH:「追加するPATH名」"
    - `PATH="$PATH:~/bin"`
- $PATHを見やすくする
  - echo $PATH | tr : "¥n"
- 呼び出し
  - `$prof`
- 一覧表示
  - `printenv`
- 個別表示
  - `printenv HOME`

# awk および sedコマンド
- ファイルの10行を表示して終了する（q）
  - `sed 10q myfile`
- 行番号が10以下の間だけ表示する
  - `awk "FNR<=20" myfile`
- 各行の最後の単語をそのまま表示する（指定されたアクションを実行する）
  - `awk "{print $NF}" members`
- アクションを持たない場合はデフォルトのアクション`{print}'を実行する
  - echo efficient linux | awk `/efficient/`
- 入力セパレーターをスペースからタブ（¥t）に変更して列を並び替える
  - `awk -F"¥t" "{print $4, '(' $3 ').', '¥"' $2 '¥"'}"` animals.txt
  - 特定の文字列で始まる行だけを処理する（例: horse）
    - `awk -F"¥t" "/^horse/{print $4, '(' $3 ').', '¥"' $2 '¥"'}"` animals.txt
  - 特定のカラムが特定の文字列で始まる行だけを処理する
    - `awk -F"¥t" "$3/^201/{print $4, '(' $3 ').', '¥"' $2 '¥"'}"` animals.txt

# テキストの変換
- 文字を別の文字に変換する（例: コロンを改行する）
  - `echo $PATH | tr : "¥n"`
- 特定の文字セットを別の文字セットに変換する（例: アルファベットを大文字 → 小文字）
  - `echo Efficient | tr A-Z a-z`
    - `efficient`
- 特定文字を削除する（例: 空白文字の削除）
  - `echo efficient linux | tr -d " ¥t"`
- 文字列の置換（例: .jpgを.pngに置き換える）
  - `echo image.jpg | sed"s/¥.jpg/¥.png/"`
- 標準入力された単語を入れ替える
  - 'echo "linux efficient" | awk "{print $2 $1}"'


# ファイルの中身表示
- ファイルの**先頭10行**表示
  - `head num.txt`
- ファイルの**先頭3行**表示
  - `head -n3 num.txt`
- ファイルの**末尾10行**表示
  - `tail num.txt`
- ファイルの**末尾指定行**表示
  - `tail -3 num.txt`
- ファイルの**特定行数より下**を表示
  - `tail -n+25 num.txt`
- ファイルの**特定行数のみ**を表示
  - `head -n4 num.txt | tail -n1`
- ファイルの**6〜8行目のみ**を表示
  - `head -n8 num.txt | tail -n3`
- ファイルの2列目を表示する(タブ区切り)
  - `cut -f2 num.txt`
- ファイルの2列目を表示する(カンマ区切り)
  - `cut -f2 -d, num.txt`
- ファイルの1, 3列目を表示する(タブ区切り)
  - `cut -f1,3 num.txt`
- ファイルの2~4列目を表示する(タブ区切り)
  - `cut -f2-4 num.txt`
- ファイルの2列目を先頭3行だけ表示する(タブ区切り)
  - `cut -f2 num.txt | head -n3`
- 各行の特定範囲の文字を表示
  - `cut -c1-3 num.txt`
- ファイルからキーワードのある行を抜き出す
  - `grep cat animals.txt`
  - 大文字小文字区別なく検索
    - `grep -i cat animals.txt`
- ファイルを横結合する
  - `paste -d, num.txt animals.txt`
  - `paste -d "¥n" num.txt animals.txt`
- 各ファイルの結合した行を生成する
  - `paste -d, -s num.txt animals.txt`

# ファイル一覧表示 / 検索
- ファイルのみ表示する
  - `find . -type f -print`
- ディレクトリのみ表示する
  - `find . -type d -print`
- 出力結果のそれぞれのファイルパスに対して操作を実施
  - `find /etc -exec echo @ {} @ ";"`
  - `find /etc -type f -name "*.conf" -exec ls -l {} ";"`
- 特定のファイルを探す
  - `find [検索対象のディレクトリ] -name '[検索対象のファイル]'`
- 特定文字をファイル名に含むファイルの検索
  - `find [検索対象のディレクトリ] -name '*infomation*.csv'`
- 上記に加えてn日前までの間に更新されたファイルだけを表示
  - `find [検索対象のディレクトリ] -mtime -7 -name '*infomation*.csv'`
- 任意の単語が含まれるファイルを探す
  - `grep -r [検索対象の言語] [検索対象のディレクトリ]`

# ファイルの中身を操作
- ファイルの行数確認
  - `wc -l [対象のファイル]`
- ファイルの単語数確認
  - `wc -w [対象のファイル]`
- ファイルの文字数確認（マルチバイト対応）
  - `wc -m [対象のファイル]`
- ファイルの文字数確認（マルチバイト非対応）
  - `wc -c [対象のファイル]`
- ファイル内の文字列を置換する
  - `sed's/[置換元の文字列]/[置換後の文字列]/g' [置換元ファイル] > [置換後ファイル]`
- 行の並び替え(アルファベット昇順)
  - 'sort num.txt'
- 行の並び替え(アルファベット降順)
  - 'sort -r num.txt'
- 行の並び替え(数値昇順)
  - 'sort -n num.txt'
- 行の並び替え(数値降順)
  - 'sort -nr num.txt'

# ファイルの圧縮と展開
- ファイルのアーカイブと圧縮
  - `tar -czf 2.tar.gz a.txt b.txt`
  - アーカイブ
    - `tar -cf 2.tar.gz a.txt b.txt`
  - 圧縮
    - `gzip 2.tar`
- 圧縮ファイルの展開
  - `tar -xzf 2.tar.gz`
- zipファイルの作成
  - `zip -r [作成されるZIPファイル名] [圧縮対象のディレクトリ]`
- 解凍せずにZIPファイルの中身を確認
  - `unzip -l [ZIPファイル名]`
- ZIPファイルの解凍
  - `unzip [ZIPファイル名]`

# ファイル / ディレクトリの検索
- カレントディレクトリ下のファイルを検索する
  - `find . -name nikki.htmi -print`
  - ワイルドカード検索
    - `find . -name *.htmi -print`
- カレントディレクトリ下のディレクトリを検索する
  - `find . -type d -print`
  - 空のディレクトリのみを検索
    - `find . -type d -empty -print`
- カレントディレクトリ下の3日前から現在までに作成したファイルを検索する
  - `find . -mtime -3 -print`

# プロセス関連
- プロセスの一覧
  - `ps -aux`
- プロセス終了
  - `kill {プロセスID}`

# ユニットとサービス（デーモン）の管理
- 起動
  - `systemctl start ユニット名`
- 終了
  - `systemctl stop ユニット名`
- 強制終了
  - `systemctl kill -s 9 ユニット名`
- 再起動
  - `systemctl restart ユニット名`
- サービス一覧の表示
  - `systemctl -t service`
  - `systemctl --type service`

# リダイレクトをファイルに保存
- `ls > ~/ls.txt`
- 追加保存
  - `ls april >> ~/ls.txt`
- 標準エラー出力
  - `ls /abcdefg 2 > ~/error.txt`

# シンボリックリンク
- シンボリックリンクの作成
  - `ln -s リンク元のファイル リンク先のファイル` 
  - `ln -s file1.txt file2.txt`
- シンボリックリンクの確認
  - `ls -l file2.txt`

# ファイルシステムの使い方
- パーティションを作成する
  -`fdisk /dev/sdb`
    - `/dev/sdb1`みたいに作成される 
- ファイルシステム作成する
  - `mke2fs -t ext4 /dev/sdb1`
- ディレクトリを作成してマウントする
  - `mkdir /datadisk1`
  - `mount /dev/sdb1 /datadisk1`
- （FATでフォーマットされたUSBメモリやポータブルHDD/SSDの場合）`/mnt/`下のディレクトリをマウントする
  - `mkdir /mnt/usbmem1`
  - `mount -t vfat /dev/sdf1 /mnt/usbmem1`
- 自動マウント
  - `/etc/fstab`を更新

# パイプ機能
- **コマンドの標準出力**を標準入力へ渡す
  - `ls -l | less`

# パイプを利用した各種コマンド
- ファイルの行数を数える
  - `cat [任意のテキスト形式のファイル] | wc -l`
- 大量の出力結果を表示したい
  - `ls --help | less`

# `grep`コマンド利用方法
- [基本]特定の文字列が含まれる行を探す
  - `grep [探したい文字列] [ファイル名]`
- **マッチする行を含むファイル名だけ**を表示した検索
  - `grep -l [探したい文字列] [ファイル名]`
- 特定の文字列が**含まれない**行を探す
  - `grep -v [探したくない文字列] [ファイル名]`
- **厳密**な単語の検索
  - `grep -w [探したい文字列] [ファイル名]`
- **大文字小文字を無視**した単語の検索
  - `grep -i [探したい文字列] [ファイル名]`
- **特殊文字**はエスケープが必要
  - `grep "w¥." [ファイル名]`

# `grep`コマンドの活用
- 大文字からはじまるすべての行にマッチ
  - `grep "^[A-Z]" [ファイル名]`
- 空出ないすべての行にマッチ
  - `grep -v "^$" [ファイル名]`
- 「cookie」または「cake」を含んでいるすべての行にマッチ
  - `grep "cookie¥|cake" [ファイル名]`
- 5文字以上の長さのすべての行にマッチ
  - `grep "....." [ファイル名]`
- HTMLコードの行にマッチ
  - `grep "<.*>" [ファイル名]`

# `grep`コマンドでエラーログを捜索
- 検索結果の行数が長い場合
  - 'less'で表示
    - `cat access.log | grep " 503 " | less`
- 検索結果の行数をカウント
  - `cat access.log | grep " 503 " | wc -l`
- 正規表現で複数ファイルを対象に検索
  - grep " 503 " *.txt

# awkコマンド
- 基本的な使い方
  - `echo Efficient fun Linux | awk "{print $1 $3}"` # スペースなし
    - `EfficientLinux`
  - `echo Efficient fun Linux | awk "{print $1, $3}"` #スペースあり
    - `Efficient Linux`
- **整然とした列から逸脱した列から逸脱したコマンド出力**を処理するのに最適
  - `df / /data | awk "{print $4}"`
- セパレータを任意の**正規表現**に変更可能
  - `echo efficient::::::linux | awk -F":*" "{print $2}"`
    - `linux`

# 重複ファイルの検出
- 特定ファイルのチェックサム
  - 'md5sum image001.jpg'
- 複数ファイルのチェックサム
  - 'md5sum image001.jpg image002.jpg image003.jpg'
  - 'md5sum *.jpg'
- 重複しているファイルのチェックサムを多い順から並べる
  - 'md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr'
- 重複しているファイルのチェックサムのみを表示する
  - 'md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr | grep -v "      1 "'
  - **これをawkで簡略化する**
    - ```
      md5sum *.jpg ¥
      | awk `{counts[$1]++; names[$1]=names[$1] " " $2} ¥
             END {for (key in counts) print counts[key] " " key ":" names[key]}` ¥
      | grep -v "^1 " ¥
      | sort -nr
      ```
- 特定のチェックサムのファイルを表示
  - `m5sum *.jpg | grep [チェックサム番号] | cut -c35-`

# テキストの生成
- すべてのユーザ名を表示して、ソートする
  - `cut -d: -f1 /etc/passwd | sort`
- 複数のファイルの内容を別のコマンドにパイプで渡す
  - `cat *.txt | wc -l`

