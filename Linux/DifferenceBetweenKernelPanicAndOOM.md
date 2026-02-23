# OOM と Kernel Panic の本質的な違い

| 観点 | OOM (Out Of Memory) | Kernel Panic |
|------|---------------------|--------------|
| 発生レイヤ | 主にユーザ空間（メモリ不足） | カーネル空間（致命的例外） |
| トリガー | 物理メモリ枯渇・cgroup制限超過 | NULL参照・メモリ破壊・FS崩壊など |
| 主体 | OOM Killer | panic() |
| 対象 | 特定プロセス | OS全体 |
| システム継続性 | 継続する | 停止する |
| 自動回復 | あり（プロセスkill） | **基本なし（再起動必要）** |
| ログ取得 | 稼働中に確認可能 | 次回ブート後に確認 |
| Kubernetesでの挙動 | Pod再起動（OOMKilled） | Node NotReady → 再作成 |
| 予測可能性 | 高い（メモリ傾向で検知可能） | 低い（多くは突発） |
| 監視アプローチ | リソース数値監視 | カーネルログ監視 |
| 代表ログ例 | `Out of memory: Kill process 1234 (java)` | `Kernel panic - not syncing: Fatal exception` |
| 運用設計での制御性 | 高い（制限・チューニング可能） | 低〜中（HW/バグ依存） |

# OOM と Panic の決定的な違い

## OOMの場合

	•	ログは残る
	•	再起動不要
	•	journalctlで確認可能

```
journalctl -k | grep -i oom
```

## KernelPanicの場合

	•	そのブートは終了
	•	前回ログを見る必要あり

```
journalctl -k -b -1
# もしくは
/var/crash/
```