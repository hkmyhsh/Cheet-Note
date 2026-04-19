# リリース管理

## 実務向けの統合版（1枚で見たい場合）

```mermaid
flowchart TD
    A[ソースコード / VCS] --> B[CIビルド]
    B --> C[Build Info / Artifact登録]
    C --> D[テスト]
    D --> E[セキュリティ / ライセンス / 品質ゲート]
    E --> F{自動ゲート通過?}

    F -- No --> G[rejected / failed-gate 記録]

    F -- Yes --> H[Promotion API to Staging]
    H --> I[Staging Repository]
    I --> J[ステージング検証]
    J --> K{承認OK?}

    K -- No --> L[差し戻し]
    K -- Yes --> M[Promotion API to Release]
    M --> N[Immutable Release Repository]
    N --> O[Release Bundle v2作成]
    O --> P[ターゲットへ配布]
    P --> Q[Checksum / Digest 検証]
    Q --> R[Production Deploy]
    R --> S[監視 / インシデント対応]

    S --> T[Bundle]
    T --> U[Build Info]
    U --> V[Source / Dependencies]

    W[Promotion Identity 分離] -.-> H
    W -.-> M
    X[Copyベース昇格] -.-> H
    X -.-> M
    Y[長期保持ポリシー] -.-> N
    Y -.-> O
```

## 全体像

```mermaid
flowchart TD
    A[ソースコード / VCS] --> B[CIビルド]
    B --> C[Build Info登録]
    C --> D[テスト実行]
    D --> E[セキュリティスキャン / ライセンスチェック / 品質ゲート]
    E --> F{ゲート通過?}

    F -- No --> G[failed-gate / rejected として記録]
    F -- Yes --> H[プロモーション to Staging]

    H --> I[ステージングリポジトリ]
    I --> J[ステージング検証]
    J --> K{承認 / 検証OK?}

    K -- No --> L[差し戻し / 再ビルド]
    K -- Yes --> M[プロモーション to Release]

    M --> N[不変なRelease Repository]
    N --> O[Release Bundle v2作成]
    O --> P[ターゲット環境へ配布]
    P --> Q[配布後検証]
    Q --> R[本番デプロイ]
    R --> S[監視 / インシデント対応]

    S --> T[Bundle → Build → Source へ追跡可能]
```

## 「Promotionはビルド工程ではなくライフサイクル遷移」であることの構造図

```mermaid
flowchart LR
    subgraph Wrong["誤った考え方: Release = 再ビルド"]
        A1[同じソースコミット] --> B1[再ビルド]
        B1 --> C1[依存解決差異]
        B1 --> D1[環境差異]
        B1 --> E1[ツール挙動差異]
        C1 --> F1[検証済みと異なるバイト列]
        D1 --> F1
        E1 --> F1
    end

    subgraph Right["正しい考え方: Release = ライフサイクル遷移"]
        A2[一度だけビルド]
        A2 --> B2[検証]
        B2 --> C2[同一バイト列をPromotion]
        C2 --> D2[Staging]
        D2 --> E2[Release]
        E2 --> F2[本番]
    end
```

## リポジトリ境界とライフサイクル

```mermaid
flowchart LR
    A[Development Repository<br/>頻繁な書き込み / 変動許容]
    B[Staging Repository<br/>候補の検証用]
    C[Release Repository<br/>承認済み / 不変]
    D[Production Consumption<br/>本番はここだけ参照]

    A -->|Promotion| B
    B -->|Promotion| C
    C --> D

    X[書込権限制御] -.-> A
    Y[候補保持ポリシー] -.-> B
    Z[上書き禁止 / 削除制限] -.-> C
```

## Copy と Move の判断図

```mermaid
flowchart TD
    A[プロモーション方式を決める] --> B{証跡を残したいか?}

    B -- Yes --> C[Copyを選択]
    B -- No --> D{厳格な単一所在モデルか?}

    D -- Yes --> E[Moveを検討]
    D -- No --> C

    C --> F[元を保持]
    C --> G[ロールバック容易]
    C --> H[調査・比較が可能]
    C --> I[推奨デフォルト]

    E --> J[重複削減]
    E --> K[保存容量削減]
    E --> L[ただし調査柔軟性低下]
    E --> M[証跡消失リスク]
```

## 自動ゲートと REST API プロモーション

```mermaid
flowchart TD
    A[CI Pipeline] --> B[ビルド]
    B --> C[Build Info公開]
    C --> D[テスト]
    D --> E[セキュリティ / ライセンス / 品質チェック]
    E --> F[承認確認]
    F --> G{ゲート判定}

    G -- No --> H[Promotionしない]
    H --> I[failed-gate / rejected 記録]

    G -- Yes --> J[REST APIでBuild Promotion]
    J --> K[Status更新]
    J --> L[ArtifactsをtargetRepoへCopy]
    K --> M[監査証跡]
    L --> M
```

## 権限分離の構造図

```
flowchart LR
    A[Build Identity<br/>通常CI権限]
    B[Promotion Identity<br/>昇格専用権限]
    C[Development Repo]
    D[Staging Repo]
    E[Release Repo]

    A -->|publish| C
    B -->|copy + status update| D
    B -->|copy + status update| E

    X[最小権限] -.-> A
    Y[Build Info参照 + targetRepo書込のみ] -.-> B
    Z[管理者トークン禁止] -.-> B
```

## Release Bundle v2 の概念図

```mermaid
flowchart TD
    A[Build Aの成果物]
    B[Build Bの成果物]
    C[Docker Image]
    D[Maven Artifact]
    E[npm Package]
    F[Python Wheel]
    G[共有ライブラリ / 共通資産]

    A --> H[Release Bundle v2]
    B --> H
    C --> H
    D --> H
    E --> H
    F --> H
    G --> H

    H --> I[Bundle Name]
    H --> J[Bundle Version]
    H --> K[不変なリリースオブジェクト]
```

## Release Bundle の配布と検証

```mermaid
flowchart LR
    A[Release Bundle v2<br/>name + version] --> B[配布]
    B --> C[ターゲット環境 / 別Artifactory / Edge]

    C --> D[Checksum検証]
    C --> E[Digest検証]
    C --> F[内容完全性確認]

    D --> G[配布成功の証跡]
    E --> G
    F --> G

    G --> H[本番利用可]
```

## 証跡チェーン（監査・インシデント対応）

```mermaid
flowchart TD
    A[Source Commit]
    B[Build Record / Build Info]
    C[Produced Artifacts]
    D[Promotion Event]
    E[Release Bundle]
    F[Distribution Event]
    G[Deployed Artifact]
    H[Monitoring / Incident Response]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H

    H --> I[何が出たか説明可能]
    H --> J[誰が承認したか説明可能]
    H --> K[依存関係を追跡可能]
    H --> L[再現可能]
```

## 本番への流入制御

```mermaid
flowchart TD
    A[Development Content]
    B[Experimental Content]
    C[Approved Release Content]
    D[Release Bundle / Release Repo]
    E[Production]

    A --> X[本番に直接流さない]
    B --> X
    C --> D
    D --> E

    X[遮断 / 非公開 / 非参照]
```

## 保持ポリシーと証跡保全

```mermaid
flowchart LR
    A[Development Builds] -->|短期保持| B[削除対象]
    C[Staging Candidates] -->|候補期間で保持| D[削除対象]
    E[Released Builds] -->|長期保持| F[証跡保全]
    G[Release Bundle Versions] -->|長期保持| F

    F --> H[監査対応]
    F --> I[障害調査]
    F --> J[旧版追跡]
```

