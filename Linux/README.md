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

# リダイレクトをファイルに保存
- `ls > ~/ls.txt`
- 追加保存
  - `ls april >> ~/ls.txt`
- 標準エラー出力
  - `ls /abcdefg 2 > ~/error.txt`
