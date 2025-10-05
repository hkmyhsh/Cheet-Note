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

