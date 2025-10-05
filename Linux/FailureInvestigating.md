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
- 外部からWebサービスにアクセスできない
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
