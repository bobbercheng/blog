---
layout: post
title:  "Akeneo PIM architecture"
date:   2025-05-07 10:39:13 -0400
categories: PIM
tags: PIM
---

{% raw %}
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true });
  document.querySelectorAll('pre > code.language-mermaid').forEach((codeBlock) => {
    codeBlock.parentElement.outerHTML = `<pre class="mermaid">${codeBlock.textContent}</pre>`;
  });
</script>
{% endraw %}

Akeneo PIM architecture:

```mermaid
flowchart TD
    %% UI Layer
    subgraph "Web Browser / Micro-Frontend Apps"
        direction TB
        CF["Category Frontend"]:::ui
        ChF["Channel Frontend"]:::ui
        ConF["Connectivity Permission Form"]:::ui
        DQF["DataQualityInsights Frontend"]:::ui
        StrF["Structure Frontend"]:::ui
        click CF "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Category/front"
        click ChF "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Channel/front"
        click ConF "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Connectivity/Connection/workspaces/permission-form"
        click DQF "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Pim/Automation/DataQualityInsights/front"
        click StrF "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Pim/Structure/front"
    end

    %% HTTP Layer
    subgraph "Symfony Kernel / HTTP Layer"
        direction TB
        K["Kernel.php"]:::backend
        Bst["bootstrap.php"]:::backend
        Bnds["bundles.php"]:::backend
        Router["Router"]:::backend
        Ctrl["InternalApi/ExternalApi Controllers"]:::backend
        click K "https://github.com/bobbercheng/pim-community-dev/blob/master/src/Kernel.php"
        click Bst "https://github.com/bobbercheng/pim-community-dev/blob/master/config/bootstrap.php"
        click Bnds "https://github.com/bobbercheng/pim-community-dev/blob/master/config/bundles.php"
    end

    %% Backend Context Bundles
    subgraph "Backend – Context Bundles"
        direction TB
        subgraph "Category Context"
            direction TB
            CatApp["Category Application (Ports)"]:::port
            CatDom["Category Domain"]:::domain
            CatInfra["Category Infrastructure"]:::adapter
            click CatInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Category/back"
        end
        subgraph "Channel Context"
            direction TB
            ChApp["Channel Application (Ports)"]:::port
            ChDom["Channel Domain"]:::domain
            ChInfra["Channel Infrastructure"]:::adapter
            click ChInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Channel/back"
        end
        subgraph "Connectivity Context"
            direction TB
            ConPorts["Connectivity Ports"]:::port
            ConDom["Connectivity Domain"]:::domain
            ConInfra["Connectivity Infrastructure"]:::adapter
            click ConInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Connectivity/Connection"
        end
        subgraph "FreeTrial Context"
            direction TB
            FTApp["FreeTrial Application"]:::port
            FTDom["FreeTrial Domain"]:::domain
            FTInfra["FreeTrial Symfony Infra"]:::adapter
            click FTInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/FreeTrial/back/Infrastructure/Symfony"
        end
        subgraph "DataQualityInsights Context"
            direction TB
            DQApp["DQI Application"]:::port
            DQDom["DQI Domain"]:::domain
            DQInfra["DQI Infrastructure"]:::adapter
            click DQInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Pim/Automation/DataQualityInsights/back"
        end
        subgraph "Structure Context"
            direction TB
            StrApp["Structure Application"]:::port
            StrDom["Structure Domain"]:::domain
            StrInfra["Structure Infrastructure"]:::adapter
            click StrInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/Pim/Structure/Bundle"
        end
        subgraph "UserManagement Context"
            direction TB
            UMApp["UserMgmt Application"]:::port
            UMDom["UserMgmt Domain"]:::domain
            UMInfra["UserMgmt Infrastructure"]:::adapter
            click UMInfra "https://github.com/bobbercheng/pim-community-dev/tree/master/src/Akeneo/UserManagement/back"
        end
        subgraph "Akeneo Core Bundles"
            Core["Oro/* & Pim Core"]:::core
        end
    end

    %% Infrastructure Services
    subgraph "Infrastructure Services"
        direction TB
        DB["MySQL Database"]:::db
        ES["Elasticsearch"]:::db
        Bus["Message Bus / Queue"]:::bus
        click DB "https://github.com/bobbercheng/pim-community-dev/blob/master/docker/initdb.d/create_db_test.sh"
        click ES "https://github.com/bobbercheng/pim-community-dev/blob/master/config/packages/akeneo_elasticsearch.yml"
        click Bus "https://github.com/bobbercheng/pim-community-dev/blob/master/config/packages/messenger.yml"
    end

    %% OAuth2 and CI/CD
    subgraph "Security & Orchestration"
        direction TB
        OAuth["OAuth2 Server (FOSOAuthServer)"]:::infraService
        CI["Docker & CircleCI"]:::infraService
        click OAuth "https://github.com/bobbercheng/pim-community-dev/blob/master/config/packages/fos_auth_server.yml"
    end

    %% Flows
    CF -->|HTTP/JSON| K
    ChF -->|HTTP/JSON| K
    ConF -->|HTTP/JSON| K
    DQF -->|HTTP/JSON| K
    StrF -->|HTTP/JSON| K

    K --> Router
    Router --> Ctrl
    Ctrl --> CatApp
    Ctrl --> ChApp
    Ctrl --> ConPorts
    Ctrl --> FTApp
    Ctrl --> DQApp
    Ctrl --> StrApp
    Ctrl --> UMApp

    CatApp --> CatDom
    ChApp --> ChDom
    ConPorts --> ConDom
    FTApp --> FTDom
    DQApp --> DQDom
    StrApp --> StrDom
    UMApp --> UMDom

    CatDom --> CatInfra
    ChDom --> ChInfra
    ConDom --> ConInfra
    FTDom --> FTInfra
    DQDom --> DQInfra
    StrDom --> StrInfra
    UMDom --> UMInfra

    CatInfra --> DB
    ChInfra --> DB
    ConInfra --> DB
    FTInfra --> DB
    DQInfra --> DB
    StrInfra --> DB
    UMInfra --> DB

    CatInfra --> ES
    DQInfra --> ES

    CatDom --> Bus
    ChDom --> Bus
    ConDom --> Bus
    FTDom --> Bus
    DQDom --> Bus
    StrDom --> Bus
    UMDom --> Bus

    Bus --> CatInfra
    Bus --> ChInfra
    Bus --> ConInfra
    Bus --> FTInfra
    Bus --> DQInfra
    Bus --> StrInfra
    Bus --> UMInfra

    Core --> CatApp
    Core --> ChApp
    Core --> ConPorts
    Core --> FTApp
    Core --> DQApp
    Core --> StrApp
    Core --> UMApp

    OAuth --> Ctrl
    CI --> K

    %% Styling
    classDef ui fill:#D0E8FF,stroke:#3B82F6,color:#1E3A8A;
    classDef backend fill:#F0F9FF,stroke:#0C4A6E,color:#0C4A6E;
    classDef port fill:#D1FAE5,stroke:#10B981,color:#065F46,shape:hexagon;
    classDef domain fill:#FEF3C7,stroke:#D97706,color:#92400E;
    classDef adapter fill:#FEF2F2,stroke:#DC2626,color:#7F1D1D;
    classDef core fill:#EEF2FF,stroke:#6366F1,color:#4338CA;
    classDef db fill:#F5F3FF,stroke:#7C3AED,color:#4C1D95,shape:cylinder;
    classDef bus fill:#EFF6FF,stroke:#2563EB,color:#1E40AF,shape:circle;
    classDef infraService fill:#ECFDF5,stroke:#22C55E,color:#166534;
```


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
