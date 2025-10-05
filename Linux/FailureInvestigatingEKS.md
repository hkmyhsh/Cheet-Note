# 事前前提・準備
- Debian/Ubuntu
```
sudo apt-get update
sudo apt-get install -y curl ca-certificates iproute2 jq dnsutils lsof tcpdump \
  nftables net-tools procps openssl inetutils-traceroute mtr-tiny \
  awscli kubectl ss traceroute
export HTTP_PROXY=http://proxy.local:8080
export HTTPS_PROXY=http://proxy.local:8080
export NO_PROXY=".cluster.local,.svc,.svc.cluster.local,localhost,127.0.0.1,169.254.169.254,\
.eks.amazonaws.com,.amazonaws.com,<vpce-xxx-*.vpce.amazonaws.com>,<your.corp.domains>"
```
- RHEL/Alma/Rocky
```
sudo yum install -y curl ca-certificates iproute jq bind-utils lsof tcpdump \
  nftables net-tools procps-ng openssl traceroute mtr \
  awscli kubectl ss
export HTTP_PROXY=http://proxy.local:8080
export HTTPS_PROXY=http://proxy.local:8080
export NO_PROXY=".cluster.local,.svc,.svc.cluster.local,localhost,127.0.0.1,169.254.169.254,\
.eks.amazonaws.com,.amazonaws.com,<vpce-xxx-*.vpce.amazonaws.com>,<your.corp.domains>"
```
- 社内CAを全レイヤで信頼
  - Deb系
    - /usr/local/certificate
    - `update-ca-certificates`
  - RHEL系
    - /etc/pki/ca-trust/
    - `update-ca-trustextract`
  - コンテナ（ベースイメージにCA追加）
  - （必要なら）containerd/docker
    - /etc/docker/certs.d/<REGISTRY>/配置
  - Java系
    - バージョンにより異なる
# EKS初動トリアージ（共通コマンド）
- クラスタ・コア系
```
# kubeconfig 確立（必要時）
aws eks update-kubeconfig --region <ap-northeast-1> --name <cluster>

kubectl get nodes -o wide
kubectl get pods -n kube-system -o wide
kubectl get events -A --sort-by=.lastTimestamp | tail -n 120

# DNS: CoreDNS 設定とログ
kubectl -n kube-system get cm coredns -o yaml
kubectl -n kube-system logs deploy/coredns --tail=300

# CNI / kube-proxy
kubectl -n kube-system logs ds/aws-node --tail=200
kubectl -n kube-system logs ds/kube-proxy --tail=200

# Ingress / LB コントローラ（使っていれば）
kubectl -n kube-system get deploy aws-load-balancer-controller
kubectl -n kube-system logs deploy/aws-load-balancer-controller --tail=300
```
- サービス公開（ALB/NLB）
```
kubectl get svc,ing -A -o wide | egrep 'LoadBalancer|NodePort|ingressclass'
kubectl -n <ns> describe svc <svc>
kubectl -n <ns> describe ing <ing>
```
- よくある原因
  - SG/NACLで遮断、またはNodePort範囲不一致
  - TargetGroupヘルスチェックパス/ポート不整合
  - Ingress注釈ミス（ALB/NLB向けannotations）
  - CodeDNSの上流解決先が社内DNS/Route53Resolverとズレ
# 経路別チェック
## 外部（社内拠点）→DX→VPC→EKS（内部 ALB/NLB）
- DNS（社内 DNS/Private Hosted Zone）
```
# 社内端末から FQDN -> 内部ALB/NLBの名前/プライベートIPへ解決するか
nslookup app.corp.example.com
```
- LBの状態（ALS CLI）
```
# 既知の DNS 名から LB を特定
DNS="<lb-dns-from-k8s-status>"
LB_ARN=$(aws elbv2 describe-load-balancers \
  --query "LoadBalancers[?DNSName=='${DNS}'].LoadBalancerArn" -o text)

aws elbv2 describe-target-groups --load-balancer-arn "$LB_ARN"
TG_ARN="<上の出力から>"
aws elbv2 describe-target-health --target-group-arn "$TG_ARN"
```
- セキュリティ協会
```
# LB / Node / Pod へ関連する SG の入出力を点検
aws ec2 describe-security-groups --group-ids <sg-ids> \
  --query 'SecurityGroups[*].{id:GroupId,ingress:IpPermissions,egress:IpPermissionsEgress}'
# ルート/NACL
aws ec2 describe-route-tables --filters Name=vpc-id,Values=<vpc-xxx> \
  --query 'RouteTables[*].{id:RouteTableId,routes:Routes}'
```
- Pod 側リスン/Readiness
```
kubectl -n <ns> get pods -o wide -l app=<your-app>
kubectl -n <ns> describe pod <pod>
kubectl -n <ns> logs <pod> --tail=200
kubectl -n <ns> exec -it <pod> -- ss -lntp
```
## Pod→社外（①社内プロキシ経由、②VPCエンドポイント直行）
- podからの実態確認（エフェメラル診断Pod）
```
# 同一NS/ネットワークで起動して試験
kubectl -n <ns> run diag --rm -it --restart=Never \
  --image=nicolaka/netshoot -- bash
# 以降は diag コンテナ内:
env | grep -i _proxy
curl -v -x http://proxy.local:8080 https://www.google.com/
# VPCエンドポイント先の疎通（Private DNS）
dig +short s3.ap-northeast-1.amazonaws.com
aws sts get-caller-identity
aws ecr get-authorization-token --region ap-northeast-1 >/dev/null
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
