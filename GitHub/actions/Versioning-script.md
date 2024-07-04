# バージョニングスクリプト
- セマンティックバージョニング
  - フルバージョンタグ：「v1.2.3」のようにパッチバージョンまで含めた書式
  - メジャーバージョンタグ：「v1」のようなメジャーバージョンのみの書式

- アクション例の仕様
  - 新しいバージョンを算出し、2種類のバージョンタグを作成する
  - バージョンアップレベルを指定するため、'major' / 'minor' / 'patch' のいずれかをスクリプト引数として渡す

- アクションのロジック
  - GitHubから現在のバージョンを取得
  - 次のバージョンを算出し、Gitタグを作成
  - GitHubへGitタグをプッシュ
 
- スクリプトの例
```
#!/usr/bin/env bash

 # GitHubからGitタグをまとめてフェッチして、最新バージョンを取り出す
 git fetch --tag 2>/dev/null
 version="$(git tag --sort=-v:refname | head -1 | sed 's/^v//')"

 # 指定されたバージョンアップレベルに基づいて、新しいバージョンを算出
 IFS='.' read -ra tokens <<<"${version:-0.0.0}"
 major="${tokens[0]}"; minor="${tokens[1]}"; patch="${tokens[2]}"
 case "$1" in
   major) major="$((major + 1))"; minor=0; patch=0 ;;
   minor) minor="$((minor + 1))"; patch=0 ;;
   patch) patch="$((patch + 1))" ;;
 esac

 # GitHubへフルバージョンタグとメジャーバージョンタグをプッシュ
 git tag "v${major}.${minor}.${patch}"
 git tag --force "v${major}" >/dev/null 2>&1
 git push --force --tags >/dev/null 2>&1
 echo "v${major}.${minor}.${patch}"
```
