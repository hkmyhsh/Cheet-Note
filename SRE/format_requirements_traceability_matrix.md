# トレーサビリティ表の目的と必要性

- 要件は全部実装されたか
- 実装したものは全部検証されたか
- 検証されてない要件はないか

## レビュー者視点

```
要件 → 設計 → 検証
```

が成立しているかを見る

# トレーサビリティ表フォーマット

トレーサビリティは **全工程を横断します。**

**要件 → 設計 → 検証観点 → テスト → リリース判定**
をつなぐ

| 要件ID | 要件 | 基本設計 | 詳細設計 | 実装 | 検証 | リリース |
|------|------|------|------|------|------|------|
| R1 | Runner自動スケール | ARC autoscaling | RunnerDeployment | Helm | scale test | OK |
| R2 | AWS認証 | IRSA | ServiceAccount | IAM Role | IAM test | OK |
| R3 | GitHub接続 | ARC Controller | controller config | Helm | workflow test | OK |
| R4 | 監視 | Prometheus | metrics exporter | Helm | monitoring test | OK |

```markdown
# 要件トレーサビリティ

| 要件ID | 要件 | 基本設計 | 詳細設計 | 実装 | 検証 | リリース |
|------|------|------|------|------|------|------|
| R1 | Runner自動スケール | ARC autoscaling | RunnerDeployment | Helm | scale test | OK |
| R2 | AWS認証 | IRSA | ServiceAccount | IAM Role | IAM test | OK |
| R3 | GitHub接続 | ARC Controller | controller config | Helm | workflow test | OK |
| R4 | 監視 | Prometheus | metrics exporter | Helm | monitoring test | OK |
```

# トレーサビリティ表作成タイミング

**後から作ると失敗します。**

```
要件定義
↓
要件ID確定
↓
トレーサビリティ表作成（空で作る）
↓
設計
↓
設計列を更新
↓
実装
↓
実装列を更新
↓
検証
↓
検証列を更新
```

# 実務の流れ

要件定義
↓
トレーサビリティ作成

|要件ID|要件|
|R1|Runner autoscale|
|R2|IAM auth|

↓

設計

|要件ID|要件|設計|
|R1|Runner autoscale|ARC|

↓

実装

|要件ID|要件|設計|実装|
|R1|Runner autoscale|ARC|Helm|

↓

検証

|要件ID|要件|設計|実装|検証|
|R1|Runner autoscale|ARC|Helm|scale test|
```

