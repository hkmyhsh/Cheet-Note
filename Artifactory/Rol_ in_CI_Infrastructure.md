# CI基盤における位置づけ

```mermaid
flowchart TB

    %% ===== 開発者・入力側 =====
    Dev[開発者]
    SCM[GitHub / GitLab / SCM]
    Hook[Webhook / Push / PR / Release Trigger]

    %% ===== CI制御 =====
    Orchestrator[CI Orchestrator<br/>Jenkins / GitHub Actions / GitLab CI]
    Runner[Build Runner / Agent<br/>Self-hosted Runner / Jenkins Agent]

    %% ===== 依存取得 =====
    subgraph ArtifactRepo["Artifact Repository Manager<br/>Artifactory / Nexus"]
        direction TB
        Virtual[Virtual Repository / Group]
        Local[Local Repository<br/>自組織成果物]
        Remote[Remote Proxy Repository<br/>外部レジストリのキャッシュ]
    end

    %% ===== 外部取得元 =====
    InternetRepo[外部レジストリ / Upstream<br/>Maven Central / npmjs / Docker Hub / PyPI]

    %% ===== Build/Scan/Test =====
    Build[Build / Package]
    UnitTest[Test]
    Scan[Security / License Scan<br/>Xray / OSS Scan / 外部Scanner]
    Quality[Quality Gate<br/>合否判定]

    %% ===== 出力・配布 =====
    ImageReg[Container Registry]
    ReleaseRepo[Release Repository]
    DeployCtl[CD / Deploy Controller<br/>Argo CD / Deploy Job]
    Runtime[実行環境<br/>EKS / Kubernetes / VM]

    %% ===== 運用監視 =====
    LogMon[監視 / ログ / 監査]
    Ops[運用Runbook / 障害対応]
    IAM[認証認可<br/>LDAP / SSO / Token / Permission]

    %% ===== main flow =====
    Dev --> SCM
    SCM --> Hook
    Hook --> Orchestrator
    Orchestrator --> Runner

    Runner -->|依存取得| Virtual
    Virtual --> Local
    Virtual --> Remote
    Remote --> InternetRepo

    Runner --> Build
    Build --> UnitTest
    UnitTest --> Scan
    Scan --> Quality

    Build -->|成果物Publish| Local
    Quality -->|合格成果物を利用| ReleaseRepo
    Quality -->|コンテナPush| ImageReg

    ReleaseRepo --> DeployCtl
    ImageReg --> DeployCtl
    DeployCtl --> Runtime

    %% ===== control / audit =====
    IAM --> ArtifactRepo
    IAM --> Orchestrator
    LogMon --> ArtifactRepo
    LogMon --> Orchestrator
    LogMon --> Runner
    LogMon --> DeployCtl
    Ops --> LogMon
    Ops --> ArtifactRepo
    Ops --> Orchestrator

    %% ===== notes by relation =====
    classDef control fill:#f6f6f6,stroke:#555,stroke-width:1px;
    classDef core fill:#eef7ff,stroke:#1d5fa7,stroke-width:1.2px;
    classDef runtime fill:#eefaf0,stroke:#2f7d32,stroke-width:1.2px;
    classDef risk fill:#fff6e8,stroke:#b26b00,stroke-width:1.2px;

    class ArtifactRepo,Orchestrator,Runner core;
    class DeployCtl,Runtime runtime;
    class LogMon,Ops,IAM control;
    class InternetRepo,Scan,Quality,ReleaseRepo,ImageReg risk;
```

## 入力の制御点

Runner はビルド時に外部の依存を直接取りに行くのではなく、
原則として Artifactory / Nexus 経由で取得する。

つまり、

- npmjs
- Maven Central
- Docker Hub
- PyPI

へのアクセスを、直接ではなく
Remote Proxy Repository を通して統制する位置です。

これにより、

- キャッシュ
- 外部通信先の限定
- 取得物の再現性確保
- 障害時の切り分け簡易化

が可能。