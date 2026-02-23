# OOM について

**プロセスを犠牲にしてOSを守る**

## 内部動作（Linux）

メモリ不足時：

	1.	ページ回収
	2.	swap使用
	3.	それでも不足
	4.	OOM Killer発動

```
Out of memory: Kill process 1234 (java) score 987
Killed process 1234 (java) total-vm:4096kB
```

# OOMのログの読み方

## どのプロセスが殺されたか

```
Killed process 1234 (java)
```

## なぜそのプロセスか

```
cat /proc/1234/oom_score
# score 987
# → oom_score が高い＝メモリ大量消費
```

# Kubernetesの場合

```
OOMKilled
```

これは：

	•	コンテナのcgroup制限超過
	•	ノード自体のOOMとは別

| 種類 | 内容 |
|------|---------------------|
| Container OOM | Pod再起動 |
| Node OOM | プロセスkill |
| Kernel Panic | ノード停止 |