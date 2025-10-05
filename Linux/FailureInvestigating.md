# 事前準備（ツールインストール）
- Debian/Ubuntu
```
sudo apt-get update
sudo apt-get install -y curl ca-certificates iproute2 jq dnsutils lsof tcpdump \
  nftables net-tools procps openssl inetutils-traceroute mtr-tiny
```
- RHEL/Alma/Rocky
```
sudo yum install -y curl ca-certificates iproute jq bind-utils lsof tcpdump \
  nftables net-tools procps-ng openssl traceroute mtr
```
- （任意）ネットワーク調査用コンテナ：nicolaka/netshoot / praqma/network-multitool

# ざっくりとして事象切り分けフロー
```mermaid
flowchart TD
A[外部からアクセス不可] --> B{DNS 想定IP?}
B -- NG --> B1[DNS設定/伝播見直し]
B -- OK --> C[SYN 到達? (tcpdump)]
C -- NG --> C1[上流/LB/ネットワーク経路]
C -- OK --> D[ポートLISTEN?]
D -- NG --> D1[サービス起動/ListenAddr/公開設定]
D -- OK --> E[FW/DOCKER-USER で遮断?}]
E -- YES --> E1[ルール修正]
E -- NO --> F[アプリ応答? (curl localhost)]
F -- NG --> F1[アプリログ/依存先/TLS]
F -- OK --> G[クライアント側/途中装置再確認]
```

# 一次切り分け
- 共通
```
hostname -f; date; uptime
ip -brief addr
ip route show; ip -6 route show
resolvectl status 2>/dev/null || cat /etc/resolv.conf
free -h; df -hT
```
- ポート/プロセス/ファイアウォール
```
# LISTEN しているポートとプロセス
sudo ss -tulpen
# 誰がそのポートを掴んでいるか
sudo lsof -iTCP -sTCP:LISTEN -nP
# nftables（または iptables）ルール確認
sudo nft list ruleset 2>/dev/null || sudo iptables -S
# firewalld / ufw を使っている場合
sudo firewall-cmd --state 2>/dev/null; sudo ufw status 2>/dev/null
```
- ログ収集
```
sudo journalctl -p 0..3 -n 200 --no-pager      # emerg..err 直近
sudo dmesg -T | tail -200                      # カーネル（OOM/ネット）
```
- Docker(基本)
```
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
# ポート公開とネットワーク
docker inspect <container> | jq '.[0].NetworkSettings | {Ports, Networks}'
# コンテナログと再起動回数
docker inspect -f 'RestartCount={{.RestartCount}}' <container>
docker logs --tail=300 <container>
```

# ケース別コマンド
## 外部からWebサービスにアクセスできない
- DNS整合性（公開名↔︎期待IP）
```
dig +short A www.example.com @8.8.8.8
dig +short AAAA www.example.com @8.8.8.8
```
- サーバ内ローカル疎通
```
# まずローカルでアプリは応答するか
curl -vk https://127.0.0.1/healthz  # 443 の例（自己署名でも -k でOKか確認）
curl -v  http://127.0.0.1/healthz   # 80 の例
```
- ポートがLISTENしているか/誰が掴んでいるか
```
sudo ss -lpnt 'sport = :80 or sport = :443'
sudo lsof -iTCP:80 -sTCP:LISTEN -nP
sudo lsof -iTCP:443 -sTCP:LISTEN -nP
```
- ローカルFW/DOCKER-USER ルールで落としていないか
```
sudo nft list ruleset 2>/dev/null || sudo iptables -S
sudo iptables -S DOCKER-USER 2>/dev/null
```
- パケット到達確認（SYNが来ているか）
```
# 15秒だけキャプチャ（本番での長時間キャプチャは避ける）
sudo timeout 15 tcpdump -nn -i any 'tcp port 80 or tcp port 443'
```
- Docker公開ポートの実態確認（ホスト側のバインド）
```
docker ps --format '{{.Names}} -> {{.Ports}}'
# 例: 0.0.0.0:443->8443/tcp になっているか。127.0.0.1 バインドだと外部から来ない。
docker inspect -f '{{json .NetworkSettings.Ports}}' <container> | jq
```
- よくあるミス
```
・コンテナが127.0.0.1:443にのみ公開（外部から到達不可）
・ホストのnftables/iptablesでDROP/REJECT
・プロセスは起動しているがListenAdddressがlocalhost固定
```
## 内部から別プロキシ経由で外部へ出られない
- 環境変数・設定確認
```
env | grep -i _proxy
grep -R --line-number -iE '_proxy|http_proxy|https_proxy|no_proxy' /etc 2>/dev/null | head
```
- プロキシ経由の基本疎通
```
# HTTP 先
curl -v -x http://proxy.local:8080 http://example.com/
# HTTPS 先（CONNECT 成功するか）
curl -v -x http://proxy.local:8080 https://www.google.com/
```
- no_proxyの（社内ドメイン/メタデータIP等）
```
echo "$no_proxy"
# 疎通したい FQDN/IP が no_proxy に正規化されて含まれているか（.example.local など）
```
- プロキシ→先サイトのTLSハンドシェイク（詳細）
```
# プロキシで CONNECT してから先の 443 に握手
openssl s_client -proxy proxy.local:8080 -connect target.example.com:443 -servername target.example.com -brief
```
- コンテナでプロキシ名が解決できているか
```
docker exec <container> getent hosts proxy.local
docker exec <container> env | grep -i _proxy
docker exec <container> curl -v -x http://proxy.local:8080 https://www.google.com/
```
## 証明書不正（ハンドシェイク失敗/検証エラー）
- サーバ証明書と検証結果の生確認
```
# verify return code の値と SAN/CN、有効期限/チェーンを確認
openssl s_client -connect target.example.com:443 -servername target.example.com -showcerts </dev/null
```
- 証明書の中身
```
# PEM を保存してから
openssl x509 -in server.pem -noout -text
openssl x509 -in server.pem -noout -dates -issuer -subject
```
- 社内CAの信頼ストア登録（例）
```
# Debian/Ubuntu
sudo install -m0644 corp-rootCA.crt /usr/local/share/ca-certificates/corp-rootCA.crt
sudo update-ca-certificates

#RHEL系
sudo cp corp-rootCA.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
```
- チェーン検証（特定CAで明示的に）
```
openssl verify -CAfile corp-bundle.pem server.pem
```
- Javaランタイムを使うアプリの場合（必要時）
```
keytool -list -keystore "$JAVA_HOME/lib/security/cacerts" -storepass changeit | grep -i corp
# 追加時（要審査）
sudo keytool -importcert -alias corp-root -keystore "$JAVA_HOME/lib/security/cacerts" \
  -storepass changeit -file corp-rootCA.crt
```

## 汎用ログ調査
```
# systemd ユニット単位
sudo systemctl status nginx
sudo journalctl -u nginx --since "2 hours ago" -n 500 --no-pager

# 文字列あたり（大文字小文字無視）
sudo journalctl --since "1 hour ago" | grep -iE 'error|fail|denied|timeout|refused'

# アプリケーションログ（例）
tail -n 200 -F /var/log/nginx/error.log
tail -n 200 -F /var/log/app/*.log

# カーネル/OOM
dmesg -T | grep -iE 'oom|memory|tcp'

# Docker
docker logs --since=2h <container>
docker events --since 2h | grep -iE 'die|oom|kill|restart'
```

# コンテナのネットワークを現場で検査する小技
- 同じネットワークに"検査用"コンテナをアタッチ
```
# コンテナが属するネットワークを確認
docker inspect <app> | jq -r '.[0].NetworkSettings.Networks | keys[]'
# そのネットワークに netshoot を入れてテスト
docker run --rm -it --network <network> nicolaka/netshoot bash
# その中で: curl/openssl/dig/tcpdump などが使える
```
- 対象コンテナのネット名前空間で実行（nsenter）
```
PID=$(docker inspect -f '{{.State.Pid}}' <app>)
sudo nsenter -t "$PID" -n sh -c 'ip addr; ip route; ss -tulpen'
```

# 初動で一括で取る"収集スクリプト"
- 概要
  - ホスト/コンテナの基本情報・ネット・FW・ログ・Docker状態を一式で`incident_bundle_YYYYmmdd_HHMMSS/`に収集します
- 使い方（例）
  - bash incident-quickcheck.sh -p 80,443 -c app1,proxy1 -s "2 hours ago" -t target.example.com:443
- 注意
  - 収集物にはセキュリティ情報（IP/ホスト名/ルール/ログ）が含まれる
  - 外部共有前には目視確認が必要
```
#!/usr/bin/env bash
# incident-quickcheck.sh
# 主に読むのは生成される TXT/LOG。sudo 推奨（なくても動く範囲で収集）

set -euo pipefail

PORTS=""
CONTAINERS=""
SINCE="2 hours ago"
TLS_TARGET=""     # 例: target.example.com:443
OUTDIR="incident_bundle_$(date +%Y%m%d_%H%M%S)"

usage() {
  cat <<USAGE
Usage: $0 [-p ports] [-c containers] [-s "since"] [-t host:port]
  -p  例: -p 80,443
  -c  例: -c web,proxy
  -s  journalctl --since の範囲（既定: "2 hours ago"）
  -t  TLS チェックの対象 (host:port)
USAGE
}

while getopts "p:c:s:t:h" opt; do
  case "$opt" in
    p) PORTS="$OPTARG" ;;
    c) CONTAINERS="$OPTARG" ;;
    s) SINCE="$OPTARG" ;;
    t) TLS_TARGET="$OPTARG" ;;
    h|*) usage; exit 0 ;;
  esac
done

mkdir -p "$OUTDIR"

run() { echo "\$ $*" | tee -a "$OUTDIR/commands.log"; eval "$@" 2>&1 | tee -a "$OUTDIR/output.log"; }

echo "#== System ==#" | tee "$OUTDIR/sys.txt"
{ date; hostname -f; uname -a; uptime; } >> "$OUTDIR/sys.txt" 2>&1

echo "#== Network ==#" | tee "$OUTDIR/net.txt"
{ ip -brief addr; echo; ip route show; ip -6 route show; echo; \
  (resolvectl status || cat /etc/resolv.conf) 2>/dev/null; } >> "$OUTDIR/net.txt"

echo "#== Listening ==#" | tee "$OUTDIR/listen.txt"
{ ss -tulpen || true; echo; lsof -iTCP -sTCP:LISTEN -nP || true; } >> "$OUTDIR/listen.txt" 2>&1

echo "#== Firewall ==#" | tee "$OUTDIR/fw.txt"
{ (nft list ruleset || iptables -S) 2>&1; } >> "$OUTDIR/fw.txt"

echo "#== Resources ==#" | tee "$OUTDIR/res.txt"
{ free -h; df -hT; } >> "$OUTDIR/res.txt"

echo "#== Logs ==#" | tee "$OUTDIR/logs.txt"
{ journalctl -p 0..3 --since "$SINCE" -n 200 --no-pager; echo; dmesg -T | tail -200; } >> "$OUTDIR/logs.txt" 2>&1

if command -v docker >/dev/null 2>&1; then
  echo "#== Docker Summary ==#" | tee "$OUTDIR/docker.txt"
  { docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}' || true; } >> "$OUTDIR/docker.txt"
  if [ -n "$CONTAINERS" ]; then
    IFS=',' read -ra CS <<<"$CONTAINERS"
    for c in "${CS[@]}"; do
      echo -e "\n## $c" >> "$OUTDIR/docker.txt"
      { docker inspect "$c" | jq '.[0] | {Name:.Name, RestartCount:.RestartCount, NetworkSettings:{Ports,Networks}}' || true; } >> "$OUTDIR/docker.txt"
      docker logs --since "$SINCE" "$c" > "$OUTDIR/docker_${c}_logs.txt" 2>&1 || true
    done
  fi
fi

if [ -n "$PORTS" ]; then
  IFS=',' read -ra PS <<<"$PORTS"
  for p in "${PS[@]}"; do
    echo -e "\n# Port $p" >> "$OUTDIR/listen.txt"
    { ss -lpnt "sport = :$p" || true; lsof -iTCP:$p -sTCP:LISTEN -nP || true; } >> "$OUTDIR/listen.txt" 2>&1
  done
fi

if [ -n "$TLS_TARGET" ]; then
  echo "#== TLS ($TLS_TARGET) ==#" | tee "$OUTDIR/tls.txt"
  printf "openssl s_client -connect %s -servername %s -showcerts\n" "$TLS_TARGET" "${TLS_TARGET%:*}" >> "$OUTDIR/tls.txt"
  echo | openssl s_client -connect "$TLS_TARGET" -servername "${TLS_TARGET%:*}" -showcerts >> "$OUTDIR/tls.txt" 2>&1 || true
fi

# 簡易グリップ：典型エラーの拾い上げ
echo "#== Grep (error/fail/timeout/refused/denied) ==#" | tee "$OUTDIR/greps.txt"
journalctl --since "$SINCE" --no-pager 2>/dev/null | \
  grep -iE 'error|fail|timeout|refused|denied' | tail -n 500 >> "$OUTDIR/greps.txt" || true

echo "=== Collected to: $OUTDIR ==="
```

- aaa
```

```
- aaa
```

```
- aaa
```

```

- aaa
```

```
- aaa
```

```
- aaa
```

```

- aaa
```

```
- aaa
```

```
- aaa
```

```

- aaa
```

```
- aaa
```

```
- aaa
```

```
