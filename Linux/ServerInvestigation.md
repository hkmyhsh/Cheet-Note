# 大方針

🎯 実務での判断フロー（障害初動）
	1.	uptime で Load Average 確認
	2.	top で CPU / MEM トップ確認
	3.	vmstat 1 で詰まり種別確認
	4.	必要なら対象PIDを深掘り

⸻

⚠ 注意点（誤解しやすい）
	•	CPU 100% = 悪ではない（バッチなら正常）
	•	Load Average 高い = CPU問題とは限らない（I/O待ちの可能性）
	•	VIRTが大きくても問題とは限らない（Javaは特に）

# 即自確認

## top（最も基本）

```
top
操作
	•	P → CPU使用率順にソート
	•	M → メモリ使用率順にソート
	•	Shift + m でも可
	•	1 → CPUコア別表示

見るべき列
	•	列 → 意味
	•	%CPU → CPU使用率
	•	%MEM → メモリ使用率
	•	RES → 実メモリ使用量
	•	VIRT → 仮想メモリ量
	•	PID → プロセスID
```

## htop（視認性が高い）

```
htop

	•	F6 → ソート選択
	•	マウス操作可能
	•	スレッド表示切替も簡単

※ 入っていない場合
sudo yum install htop   # RHEL系
sudo apt install htop   # Debian系
```

# 1回のスナップショットで確認

## CPU使用率トップ

```
ps aux --sort=-%cpu | head -10
```

## メモリ使用率トップ

```
ps aux --sort=-%mem | head -10
```

## RSS（実メモリ量）順にソート（より実務的）

```
ps -eo pid,comm,%mem,rss --sort=-rss | head -10
	•	rss = 実際に使っている物理メモリ（KB）
```

# 実務寄りの確認（SRE視点）

## システム全体が本当に逼迫しているか

```
free -m
vmstat 1 5
```

## CPUボトルネック確認

```
mpstat -P ALL 1
	•	%idle が低いCPUコアがあるか
	•	%iowait が高いならI/O待ち
```

# Javaプロセスの場合（CI基盤でよくある）

```
ps -ef | grep java
# PIDを取得して：
top -Hp <PID>
→ スレッド単位でCPU確認可能
```

# コンテナ環境の場合（Docker）

```
docker stats
```

# Kubernetes (EKSなど)

```
kubectl top pod -A
kubectl top node
※ metrics-server が必要
```