# CI基盤 非機能要件カタログ

※※CI基盤の品質 → 非機能要件**

| NFR-ID | カテゴリ | 要件 | 指標 / 合格基準 | 検証方法 | 監視 |
|------|------|------|------|------|------|
| NFR-001 | 可用性 | CI基盤は常時利用可能 | 稼働率 99.9% | 障害訓練 | uptime監視 |
| NFR-002 | 回復性 | Runner障害時自動復旧 | Pod再起動 | failover test | pod restart監視 |
| NFR-003 | 性能 | build queueが滞留しない | queue < 5 | 負荷試験 | queue監視 |
| NFR-004 | スケーラビリティ | Runnerは負荷に応じてスケール | autoscale | scale test | runner数監視 |
| NFR-005 | セキュリティ | AWSアクセスはIAM Role | IAM role使用 | IAM test | audit log |
| NFR-006 | 監査 | buildログ保存 | 90日保存 | log確認 | log監視 |
| NFR-007 | 監視 | CI異常を検知 | alert < 5分 | alert test | alert監視 |
| NFR-008 | 運用性 | 手動復旧可能 | Runbook存在 | 手順確認 | Runbook |
| NFR-009 | 変更管理 | CI設定変更はPR経由 | Git管理 | PR確認 | audit |
| NFR-010 | コスト | Runnerコスト制御 | 月額上限 | cost確認 | billing監視 |


| カテゴリ | 意味 |
|------|------|
| 可用性 | CIが止まらない |
| 回復性 | 止まってもすぐ復旧 |
| 性能 |buildが遅くならない |
| スケーラビリティ | 負荷に耐える |
| セキュリティ | 秘密情報漏洩防止 |
| 監査 | 誰が何を実行したか |
| 監視 | CI異常を検知 |
| 運用性 | 人が対応できる |
| 変更管理 | 設定変更の追跡 |
| コスト | クラウド費用 |
