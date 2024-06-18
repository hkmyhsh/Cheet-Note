# 既存のPATHに新たなディレクトリを追加
- PATH="$PATH:「追加するPATH名」"
  - `PATH="$PATH:~/bin"`

# 環境変数
- 設定
  - `export prof=~/prof.txt
- 呼び出し
  - `$prof`
- 一覧表示
  - `printenv`

# ファイルの中身表示
- ファイルの先頭10行表示
  - `head num.txt`
- ファイルの末尾10行表示
  - `tail num.txt`
- ファイルの末尾指定行表示
  - `tail -3 num.txt`
- ファイルからキーワードのある行を抜き出す
  - `grep cat animals.txt`
  - 大文字小文字区別なく検索
    - `grep -i cat animals.txt`

# ファイルの圧縮と展開
- ファイルのアーカイブと圧縮
  - `tar -czf 2.tar.gz a.txt b.txt`
  - アーカイブ
    - `tar -cf 2.tar.gz a.txt b.txt`
  - 圧縮
    - `gzip 2.tar`
- 圧縮ファイルの展開
  - `tar -xzf 2.tar.gz`

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

# パイプ機能
- **コマンドの標準出力**を標準入力へ渡す
  - `ls -l | less`

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
