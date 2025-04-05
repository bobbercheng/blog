---
layout: post
title:  "Agones as GKE Orchestrator for few or millions of players"
date:   2025-03-14 10:39:13 -0400
categories: Game
tags: Game GKE Orchestrator Gemini
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

Use Agones as GKE Orchestrator for few or millions of players.
Which architecture diagram you prefer?

Draw by humam experts:
![Draw by humam](https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/gke_lexitrail-2-architecture.max-1100x1100.png) or interactive draw from code repository as following:

```mermaid
flowchart TD

    subgraph "Frontend Layer"
        UI["UI (React, port:3000)"]:::ui
    end

    subgraph "Backend Layer"
        BE_LOGIC["Backend API Logic (Flask, port:5001)"]:::backend
        BE_START["Backend API Startup (run.py)"]:::backend
        UI -->|"REST:5001"| BE_LOGIC
        BE_LOGIC -->|"CRUD:3306"| DB[(MySQL Database)]
        BE_LOGIC -->|"Gemini:8080"| GEMINI["Gemini Service"]:::external
    end

    subgraph "Middle Layer"
        ML["Middle Layer (Game Server Orchestration)"]:::middle
        AGONES["Agones Library"]:::agones
        ML -->|"Orchestration"| AGONES
        AGONES -->|"FleetMgmt"| KC["Kubernetes Cluster"]:::kube
    end

    subgraph "Matchmaker Service"
        MM_DIRECTOR["Director (Go)"]:::matchmaker
        MM_FRONTEND["Frontend (Go)"]:::matchmaker
        MM_MMF["MMF (Go)"]:::matchmaker
        MM_DIRECTOR -->|"Coordination"| MM_FRONTEND
        MM_FRONTEND -->|"Coordination"| MM_MMF
        MM_MMF -->|"Coordination"| MM_DIRECTOR
        MM_FRONTEND -->|"RealtimeMatch"| BE_LOGIC
    end

    subgraph "Infrastructure & Deployment"
        INFRA["Terraform & K8s Templates"]:::infra
        INFRA --> KC
    end

    %% Click Events
    click UI "https://github.com/bobbercheng/gke_lexitrail/tree/main/ui/"
    click BE_LOGIC "https://github.com/bobbercheng/gke_lexitrail/tree/main/backend/app/"
    click BE_START "https://github.com/bobbercheng/gke_lexitrail/blob/main/backend/run.py"
    click ML "https://github.com/bobbercheng/gke_lexitrail/tree/main/middle_layer/"
    click AGONES "https://github.com/bobbercheng/gke_lexitrail/tree/main/middle_layer/agones"
    click MM_DIRECTOR "https://github.com/bobbercheng/gke_lexitrail/tree/main/matchmaker/director"
    click MM_FRONTEND "https://github.com/bobbercheng/gke_lexitrail/tree/main/matchmaker/frontend"
    click MM_MMF "https://github.com/bobbercheng/gke_lexitrail/tree/main/matchmaker/mmf"
    click INFRA "https://github.com/bobbercheng/gke_lexitrail/tree/main/terraform/"
    click DB "https://github.com/bobbercheng/gke_lexitrail/tree/main/terraform/k8s_templates/mysql-*"

    %% Styles
    classDef ui fill:#cce5ff,stroke:#004085,stroke-width:2px;
    classDef backend fill:#d4edda,stroke:#155724,stroke-width:2px;
    classDef matchmaker fill:#f8d7da,stroke:#721c24,stroke-width:2px;
    classDef middle fill:#ffeeba,stroke:#856404,stroke-width:2px;
    classDef agones fill:#fff3cd,stroke:#856404,stroke-width:2px;
    classDef database fill:#d1ecf1,stroke:#0c5460,stroke-width:2px;
    classDef infra fill:#e2e3e5,stroke:#6c757d,stroke-width:2px;
    classDef kube fill:#c3e6cb,stroke:#155724,stroke-width:2px;
    classDef external fill:#f5c6cb,stroke:#721c24,stroke-width:2px;
```


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
