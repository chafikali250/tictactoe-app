# 🎮 TicTacToe Web Application - CI/CD Pipeline

Une application web TicTacToe déployée automatiquement via un pipeline CI/CD moderne intégrant **Jenkins**, **SonarQube**, **Trivy**, **Docker Hub**, et **Argo CD** (GitOps).

---

## 🏗️ Architecture du Pipeline CI/CD

```mermaid
graph TD
    Dev[👨‍💻 Developer] -->|Push Code| GitRepo[📦 GitHub App Repo]
    GitRepo -->|Webhook / Trigger| Jenkins[⚙️ Jenkins CI Pipeline]
    
    subgraph CI Pipeline
        Jenkins --> Build[🔨 Build & Test]
        Build --> Sonar[🔍 SonarQube Code Analysis]
        Sonar --> Trivy[🛡️ Trivy Image Scan]
        Trivy --> DockerHub[🐳 Push to Docker Hub]
        DockerHub --> UpdateManifests[✏️ Update Image Tag in Manifests Repo]
    end

    UpdateManifests -->|Git Push| ManifestRepo[📄 GitHub Manifests Repo]
    
    subgraph CD Pipeline (GitOps)
        ManifestRepo -->|Sync Manifests| ArgoCD[🐙 Argo CD Core]
        ArgoCD -->|Deploy & Reconcile| K8s[☸️ Kubernetes Cluster]
    end

    Jenkins -.->|Notification| Slack[💬 Slack Channel]
