---
layout: post
title:  "Twitter recommendation architecture"
date:   2025-04-04 10:39:13 -0400
categories: ML
tags: ML
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

Twitter recommendation architecture:

```mermaid
flowchart TD
    %% Data/Ingestion Layer
    subgraph "Data / Ingestion Layer"
        A1("Tweet Data Provider (Tweetypie)"):::data
        A2("User Action Stream (Unified User Actions)"):::data
        A3("User Signal Service"):::data
    end

    %% Candidate Generation Layer
    subgraph "Candidate Generation Layer"
        B1("Search Index (Text-based)"):::candidate
        B2("Candidate Generation (CR Mixer)"):::candidate
        B3("User Tweet Entity Graph (UTEG)"):::candidate
        B4("Follow Recommendations Candidate Source"):::candidate
        C1("Candidate Pool (Recos Injector)"):::candidate
    end

    %% Ranking Layer
    subgraph "Ranking Layer"
        D1("Light Ranker (Fast Model)"):::ranking
        D2("Heavy Ranker (Neural Network)"):::ranking
    end

    %% For You Timeline Branch
    subgraph "For You Timeline"
        E1("Tweet Mixing & Filtering (Home Mixer)"):::mixing
        E2("Visibility Filters"):::mixing
        E3("For You Timeline Feed"):::mixing
    end

    %% Recommended Notifications Branch
    subgraph "Recommended Notifications"
        F1("Push Notification Recommendation Service"):::push
        F2("Pushservice Light Ranker"):::push
        F3("Pushservice Heavy Ranker"):::push
        F4("Notification Feed"):::push
    end

    %% Model & ML Layer (Auxiliary)
    subgraph "Model & ML"
        G1("SimClusters & Sparse Embeddings"):::ml
        G2("Legacy ML/Model Serving (twml)"):::ml
        G3("Navi Model Serving (Rust based)"):::ml
    end

    %% Shared / Common Libraries Layer
    subgraph "Shared Middleware & Frameworks"
        H1("Product Mixer Shared Framework"):::shared
        H2("Representation Manager"):::shared
        H3("Representation Scorer"):::shared
        H4("Visibility and Compliance Library"):::shared
        H5("Common Realtime Data Processing (ann)"):::shared
        H6("Unified User Actions Library"):::shared
    end

    %% Data flow connections
    %% Data Sources feed Candidate Generation
    A1 -->|"TweetData"| B1
    A1 -->|"TweetData"| B2
    A2 -->|"UserActions"| B3
    A3 -->|"User Signals"| B4

    %% Candidate Generation merging
    B1 --> C1
    B2 --> C1
    B3 --> C1
    B4 --> C1

    %% Candidate Pool feeds Ranking
    C1 -->|"CandidateFeed"| D1
    D1 -->|"Pre-filteredCandidates"| D2

    %% Ranking feeds For You Timeline branch
    D2 -->|"RankedCandidates"| E1
    E1 -->|"MixedCandidates"| E2
    E2 -->|"FinalFeed"| E3

    %% Recommended Notifications branch (parallel pipeline)
    %% Assume similar data ingestion feeds pushservice directly
    A1 -->|"TweetData"| F1
    A2 -->|"UserActions"| F1
    A3 -->|"UserSignals"| F1
    F1 -->|"InitialCandidates"| F2
    F2 -->|"RefinedCandidates"| F3
    F3 -->|"FinalNotifications"| F4

    %% Model & ML influence on Ranking
    G1 ---|"ML scores"| D1
    G2 ---|"Model outputs"| D2
    G3 ---|"Serving data"| D2

    %% Shared Middleware supports multiple layers
    H1 ---|"Framework support"| C1
    H2 ---|"Embedding management"| G1
    H3 ---|"Scoring assistance"| D2
    H4 ---|"Compliance checks"| E2
    H5 ---|"Realtime processing"| A2
    H6 ---|"User actions client"| A2

    %% Click Events for component mapping
    click A1 "https://github.com/twitter/the-algorithm/tree/main/tweetypie/"
    click A2 "https://github.com/twitter/the-algorithm/tree/main/unified_user_actions/"
    click A3 "https://github.com/twitter/the-algorithm/tree/main/user-signal-service/"
    click G1 "https://github.com/twitter/the-algorithm/tree/main/simclusters-ann/"
    click G2 "https://github.com/twitter/the-algorithm/tree/main/twml/"
    click G3 "https://github.com/twitter/the-algorithm/tree/main/navi/"
    click B2 "https://github.com/twitter/the-algorithm/tree/main/cr-mixer/"
    click C1 "https://github.com/twitter/the-algorithm/tree/main/recos-injector/"
    click B4 "https://github.com/twitter/the-algorithm/tree/main/follow-recommendations-service/"
    click E1 "https://github.com/twitter/the-algorithm/tree/main/home-mixer/"
    click F1 "https://github.com/twitter/the-algorithm/tree/main/pushservice/"
    click H1 "https://github.com/twitter/the-algorithm/tree/main/product-mixer/shared-library/"
    click H2 "https://github.com/twitter/the-algorithm/tree/main/representation-manager/"
    click H3 "https://github.com/twitter/the-algorithm/tree/main/representation-scorer/"
    click H4 "https://github.com/twitter/the-algorithm/tree/main/visibilitylib/"
    click H5 "https://github.com/twitter/the-algorithm/tree/main/ann/"
    click H6 "https://github.com/twitter/the-algorithm/tree/main/unified_user_actions/"

    %% Style definitions
    classDef data fill:#AED6F1,stroke:#2874A6,stroke-width:2px;
    classDef candidate fill:#FCF3CF,stroke:#B7950B,stroke-width:2px;
    classDef ranking fill:#D5F5E3,stroke:#27AE60,stroke-width:2px;
    classDef mixing fill:#FADBD8,stroke:#C0392B,stroke-width:2px;
    classDef push fill:#E8DAEF,stroke:#8E44AD,stroke-width:2px;
    classDef ml fill:#FDEBD0,stroke:#CA6F1E,stroke-width:2px;
    classDef shared fill:#D6EAF8,stroke:#3498DB,stroke-width:2px;
```

ML train/reference architecture:
```mermaid
flowchart TD
    %% Environment & Setup
    subgraph "Environment & Setup"
        EnvSetup["Virtual Environment Setup"]:::env
        EnvManagement["Environment Management"]:::env
        AuxUtilities["Auxiliary Utilities"]:::env
    end

    %% Shared Infrastructure
    subgraph "Shared Infrastructure"
        Common["Common Utilities"]:::shared
        Core["Core Training Pipeline & Configuration"]:::shared
        Optimizer["Optimizer Configurations"]:::shared
        MLLogging["ML Logging"]:::shared
        Metrics["Metrics Computation"]:::shared
        Reader["Data Ingestion / Reader"]:::shared
    end

    %% Model-Specific Modules
    subgraph "Model-Specific Modules"
        Recap["For You Heavy Ranker (Recap)"]:::project
        TwHIN["TwHIN Embeddings"]:::project
    end

    %% External Dependency
    External["torchrec/Nvidia GPU"]:::external

    %% Connections from Environment to Shared Infrastructure
    EnvSetup -->|"env_setup"| Common
    EnvManagement -->|"hardware_detection"| Common
    AuxUtilities -->|"data_prep"| Reader

    %% Internal Shared Infrastructure Data Flow
    Reader -->|"data_flow"| Common
    Common -->|"training_control"| Core
    Core -->|"optimizer_update"| Optimizer
    Core -->|"logging"| MLLogging
    Core -->|"metrics"| Metrics
    External -->|"runtime_dependency"| Core

    %% Shared to Project-Specific Flow
    Core -->|"model_training"| Recap
    Core -->|"model_training"| TwHIN

    %% Click Events for Environment & Setup
    click EnvSetup "https://github.com/twitter/the-algorithm-ml/tree/main/images/"
    click EnvManagement "https://github.com/twitter/the-algorithm-ml/tree/main/machines/"
    click AuxUtilities "https://github.com/twitter/the-algorithm-ml/tree/main/tools/"

    %% Click Events for Shared Infrastructure
    click Common "https://github.com/twitter/the-algorithm-ml/tree/main/common/"
    click Core "https://github.com/twitter/the-algorithm-ml/tree/main/core/"
    click Optimizer "https://github.com/twitter/the-algorithm-ml/tree/main/optimizers/"
    click MLLogging "https://github.com/twitter/the-algorithm-ml/tree/main/ml_logging/"
    click Metrics "https://github.com/twitter/the-algorithm-ml/tree/main/metrics/"
    click Reader "https://github.com/twitter/the-algorithm-ml/tree/main/reader/"

    %% Click Events for Model-Specific Modules
    click Recap "https://github.com/twitter/the-algorithm-ml/tree/main/projects/home/recap/"
    click TwHIN "https://github.com/twitter/the-algorithm-ml/tree/main/projects/twhin/"

    %% Styles
    classDef env fill:#fdd,stroke:#333,stroke-width:2px;
    classDef shared fill:#dfd,stroke:#333,stroke-width:2px;
    classDef project fill:#ddf,stroke:#333,stroke-width:2px;
    classDef external fill:#ffd,stroke:#333,stroke-width:2px;
```

{% raw %}
<script src="https://giscus.app/client.js"
        data-repo="bobbercheng/blog"
        data-repo-id="R_kgDOLrBZsw"
        data-category="Ideas"
        data-category-id="DIC_kwDOLrBZs84Coy7e"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
{% endraw %}


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
