# ノードが死んだとき

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