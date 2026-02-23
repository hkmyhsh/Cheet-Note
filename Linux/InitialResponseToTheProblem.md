# 実務での判断フロー

	1.	uptime で Load Average 確認
	2.	top で CPU / MEM トップ確認
	3.	vmstat 1 で詰まり種別確認
	4.	必要なら対象PIDを深掘り

## 注意点（誤解しやすい）

	•	CPU 100% = 悪ではない（バッチなら正常）
	•	Load Average 高い = CPU問題とは限らない（I/O待ちの可能性）
	•	VIRTが大きくても問題とは限らない（Javaは特に）

### Load Average の正体

```
Load Average は以下の合計
R (Running) + D (Uninterruptible sleep)

状態 → 意味
R → CPU実行待ち（ランキュー）
D → I/O待ち（ディスク・NFSなど）
```

**CPU待ちとI/O待ちを区別していない**ため、Load Averageが高いことの原因を間違えやすい
**CPUかI/Oのどちらかで詰まっている**ことのシグナル

### 切り分けの体系フロー

#### まず Load を確認

```
uptime

# load average: 12.3, 11.8, 9.2
# → コア数と比較
# 8コアで12なら明らかに過負荷
```

#### CPUが本当に詰まっているか？

```
mpstat -P ALL 1
```

| 指標 | 判断 |
|------------|------------|
| %idle 低い (<10%) | CPUひっ泊 |
| %iowait 高い (>20%) | I/O待ち |
| %steal 高い | 仮想化基盤問題 |

#### 【最重要】vmstatで種別判定

```
vmstat 1

# procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
10  0      0  50000  10000 200000    0    0     0     0  200  300 90  5  5  0  0
```

| 列 | 意味 | 判断 |
|------------|------------|------------|
| r | 実行待ち | 高い＝CPU競合 |
| b | ブロック中 | 高い＝I/O待ち |
| wa | I/O待ちCPU | 高い＝ディスク/NFS |

## 判定パターンまとめ

### CPUボトルネック

```
r 多い
wa 低い
%idle 低い
```

 → 純粋なCPU不足

### I/Oボトルネック

```
b 多い
wa 高い
%idle ある程度ある
```

 → Disk or NFS or EBS詰まり

### 仮想化影響（EKSでよくある）

```
%steal 高い
```

 → 物理ホスト側のCPU取り合い

## D状態プロセスの確認

```
# I/O待ちプロセス確認
ps -eo pid,stat,comm | grep '^ *[0-9]* D'
# STATが D のものがI/O待ち
```

## さらに深掘る場合

```
# ディスクI/O確認
iostat -x 1
```

### 見るべき項目

| 指標 | 意味 |
|------------|------------|
| %util | 100%近いと飽和 |
| await | レイテンシ |
| r/s w/s | IOPS |

## 例

### ビルド時

	•	CPUは余裕
	•	Load高い
	•	wa高い
	•	EBS %util 100%

→ ワークスペースI/O詰まり

### Nexus blob store肥大化

	•	D状態プロセス増加
	•	NFS利用ならNFS遅

## 最短判断Runbook

```
uptime
mpstat -P ALL 1
vmstat 1 5
iostat -x 1 3
```

# ノードが死んだとき

## 運用時の視点

| 症状 | まず疑う |
|------------|------------|
| Pod再起動 | cgroup OOM |
| Javaだけ死ぬ | JVM OOM |
| ノードは生きてる | OOM |
| ノード再起動してる | panic or HW |
| コンソールにstack trace | panic |

## SSH可能？
	•	Yes → OOM
	•	No → Panic可能性

## uptime確認

```
uptime
```

## 前回ブートログ

```
journalctl -b -1
```