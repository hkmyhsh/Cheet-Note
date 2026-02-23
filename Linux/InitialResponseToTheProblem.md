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