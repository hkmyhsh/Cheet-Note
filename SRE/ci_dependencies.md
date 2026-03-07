# CI基盤依存関係図

## 全体図を見やすくするためのレイヤ分解

```
1. 利用者 / イベント層
   - 開発者、PR、Push、Release

2. 制御層
   - GitHub、GitHub Actions、Jenkins Controller、ARC

3. 実行層
   - Jenkins Agent、Self-hosted Runner、EKS Pod/Node

4. 成果物層
   - Nexus、ECR、S3、Dependency Cache

5. 認証・基盤層
   - IAM、Secret、DNS、VPC、EC2、時刻同期

6. 観測・周知層
   - Monitoring、Log、Slack/Teams、Status Page
```

## 依存関係全体図

この依存関係図は、単なる図ではなく

- **「どこが上流で、どこが下流か」** を揃えるために使う
- 各ノードにRunbookの入り口を紐づけて**依存関係図 + Runbook導線図**として使う

