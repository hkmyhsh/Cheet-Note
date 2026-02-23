# 実務上の重要ポイント（CI基盤向け）

特に優先すべきは：

	1.	BUG: / WARNING: の検知
	2.	I/O error の連続発生
	3.	soft lockup / RCU stall
	4.	ECCエラーの増加

# Kernel Panic の前兆監視設計

## 前提

panicは“突然死”に見えますが、
多くは兆候ログが事前に出ています。

## 監視すべきカテゴリ

| カテゴリ | 監視対象・指標例 | 具体的なログ・確認例 | 意味・リスク | 推奨監視方法 |
|-----------|------------------|----------------------|--------------|--------------|
| ハードウェア異常 | ECCエラー、MCE | `Machine Check Exception`<br>`EDAC MC0: CE` | メモリ劣化・CPU障害の前兆 | dmesgログ監視 / ハードウェア監視エージェント |
| カーネル警告 | BUG, WARNING | `BUG:`<br>`WARNING:`<br>`kernel NULL pointer` | カーネル内部異常。panic予備軍 | journalctl -k 監視 / ログアラート |
| ソフトロックアップ | soft lockup | `BUG: soft lockup - CPU# stuck` | CPUハング・スケジューラ停止前兆 | dmesg監視 / watchdog有効化 |
| RCU stall | RCU stall | `RCU stall detected` | カーネル同期異常 | カーネルログ監視 |
| I/Oエラー | ブロックデバイスエラー | `blk_update_request: I/O error` | ディスク障害→FS破損 | dmesg / smartctl定期チェック |
| ファイルシステム異常 | EXT4/XFS error | `EXT4-fs error`<br>`XFS (sda): Metadata corruption` | ルートFS破損→panic | カーネルログ監視 |
| カーネルメモリ逼迫 | Slab増大 | slabtopで特定cache肥大 | カーネルメモリ枯渇→panic | node_exporter + slab監視 |
| 割り込み異常 | IRQ storm | `irq XX: nobody cared` | 割り込み暴走→システム不安定 | カーネルログ監視 |
| ドライバ異常 | モジュールエラー | `tainted kernel`<br>`segfault in module` | 非互換ドライバ→panic | カーネルログ監視 |

## 監視

### dmesg 監視（最重要）

#### 監視ワード例

```
BUG:
WARNING:
soft lockup
RCU stall
kernel NULL pointer
I/O error
EXT4-fs error
```

#### 検知 / 発報

- Promtail / FluentBit でログ収集し、
- Loki / CloudWatch Logs にてアラート。

### ハードウェア前兆

#### ECCエラー

```
dmesg | grep -i ecc
# 繰り返される場合はpanic予備軍
```

#### ディスク異常

```
smartctl -a /dev/sda
```

監視項目：

	•	Reallocated_Sector_Ct
	•	Current_Pending_Sector

### ソフトロックアップ検知

```
watchdog: BUG: soft lockup - CPU#0 stuck
# これはpanic一歩手前
```

node_exporter で：

	•	CPU Steal
	•	context switch異常増加
	•	load average異常

も補助指標

## Panic前兆の構造

```
HW劣化
  ↓
カーネル警告
  ↓
I/O失敗増加
  ↓
メモリ破壊
  ↓
panic()
```

# OOM の前兆監視設計

## 監視対象レイヤ

| レイヤ | 指標 |
|-----------|------------------|
| ノード | MemAvailable |
| コンテナ | memory.limit / usage |
| カーネル | Slab使用量 |
| swap | in/out |
| PSI | Memory Pressure |

### 【最重要】MemAvailable

```
cat /proc/meminfo

# 監視すべきは：MemAvailable(単なるfreeではない)
```

### 【最重要】PSI（Pressure Stall Information）

```
cat /proc/pressure/memory

# 例
# some avg10=12.50
# full avg10=3.25
## avg10が上昇し続ける → OOM前兆
## node_exporterで取得可能。
```

### Slab肥大

- カーネルメモリ枯渇はpanic原因にも。

```
slabtop
# 特定cacheが異常増大 → リーク疑い
```

### swapの異常増加

```
vmstat 1
# si/so が連続増加 → OOM接近
```

### Kubernetes特有

```
# 監視
container_memory_working_set_bytes
container_oom_events_total
# 重要項目
# memory.limit - usage
```

## OOM前兆の典型パターン

### パターンA：Javaリーク

```
Heap使用率右肩上がり
GC時間増大
RSS増大
```

### パターンB：CIビルド集中

```
Pod急増
node memory急減
swap増加
```