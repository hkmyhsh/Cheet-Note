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
  - grep cat animals.txt
