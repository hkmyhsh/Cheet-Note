# Artifactory 全体像

- **Artifactory は単なる保存庫ではなく、成果物のライフサイクルを制御する中核**
- Artifactory を「ただの置き場」ではなく**品質保証の制御点**にしている

```mermaid
flowchart TB
    %% -------------------------
    %% Sources / Build
    %% -------------------------
    A[開発者 / ソースコード / VCS] --> B[CI / Build Pipeline]
    B --> C[依存解決]
    B --> D[ビルド成果物生成]
    B --> E[Build Info登録]

    %% -------------------------
    %% Dependency resolution
    %% -------------------------
    C --> F[Virtual Repository]
    F --> G[Remote Repository<br/>外部レジストリのキャッシュ/代理取得]
    F --> H[Local Repository<br/>社内成果物の保存]
    G --> I[外部OSS / 外部Registry]
    H --> J[社内標準ライブラリ / 内製成果物]

    %% -------------------------
    %% Publish / Dev lifecycle
    %% -------------------------
    D --> K[Development Repository]
    E --> L[Build Record / Metadata]

    %% -------------------------
    %% Gates and promotion
    %% -------------------------
    K --> M[テスト / スキャン / 品質ゲート]
    L --> M
    M --> N{ゲート通過?}

    N -- No --> O[rejected / failed-gate<br/>として記録]
    N -- Yes --> P[Promotion API]

    %% -------------------------
    %% Lifecycle repos
    %% -------------------------
    P --> Q[Staging Repository<br/>候補の検証領域]
    Q --> R[ステージング検証 / 承認]
    R --> S{承認OK?}

    S -- No --> T[差し戻し / 再ビルド]
    S -- Yes --> U[Promotion API]

    U --> V[Release Repository<br/>承認済み / 不変]
    V --> W[Release Bundle v2]
    W --> X[配布 / Distribution]
    X --> Y[本番環境 / ターゲット拠点]

    %% -------------------------
    %% Verification / Evidence
    %% -------------------------
    Y --> Z[Checksum / Digest検証]
    Z --> AA[本番デプロイ]
    AA --> AB[監視 / 障害対応]

    AB --> AC[Bundle → Build → Source へ追跡]
    V --> AC
    W --> AC
    L --> AC

    %% -------------------------
    %% Governance / Controls
    %% -------------------------
    AD[権限制御<br/>誰が読めるか / 書けるか] -.-> F
    AD -.-> K
    AD -.-> Q
    AD -.-> V

    AE[不変性制御<br/>上書き禁止 / 削除制限] -.-> V
    AE -.-> W

    AF[保持ポリシー<br/>開発は短期 / リリースは長期] -.-> K
    AF -.-> Q
    AF -.-> V
    AF -.-> W

    AG[証跡 / 監査対応<br/>Promotion記録 / 承認参照 / Build Info] -.-> L
    AG -.-> W
    AG -.-> AC
```

## 1. 入口：依存解決と成果物保存

- CI は Artifactory の Virtual Repository を入口として依存解決する
- 外部OSSは Remote Repository でキャッシュし、社内成果物は Local Repository に置く
- これにより、依存解決の入口が統制される

## 2. ビルド：成果物と Build Info を残す

- ビルド成果物だけでなく、Build Info も登録する
- 何から作られたか、どの依存を使ったかを後から追える

## 3. 判定：テスト・スキャン・承認

- Development Repository に置かれた成果物を、そのままゲートにかける
- 合格したものだけを Promotion する
- ここで重要なのは、再ビルドせず、同じバイト列を昇格すること

## 4. 境界：Development → Staging → Release

- Development は頻繁更新を許容
- Staging は候補の検証
- Release は承認済みで不変
- この境界そのものが統制点になる

## 5. 配布：Release Bundle と Distribution

- Release Repository 上の承認済み成果物を、必要に応じて Release Bundle v2 として束ねる
- それをターゲット環境へ配布する
- 本番は広い入口を見ず、承認済みの供給源だけを見る

## 6. 検証：Checksum / Digest / 追跡

- 配布後に整合性を確認する
- 障害時には Bundle → Build → Source と逆追跡できる
- これが監査や障害対応で効く

## 7. 下支え：権限・不変性・保持

- 誰が書けるか
- 上書きできるか
- どれだけ保持するか