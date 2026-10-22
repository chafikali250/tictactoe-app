```mermaid
graph TD
    Dev[Developer] -->|Push Code| GitRepo[GitHub App Repo]
    GitRepo -->|Trigger / Pull| Jenkins[Jenkins CI Pipeline]
    
    subgraph CI_Pipeline
        Jenkins --> Build[Build & Test npm]
        Build --> Sonar[SonarQube Code Analysis]
        Sonar --> Trivy[Trivy Image Scan]
        Trivy --> DockerHub[Push to Docker Hub]
        DockerHub --> UpdateManifests[Update Image Tag in Manifests Repo]
    end

    UpdateManifests -->|Git Push| ManifestRepo[GitHub Manifests Repo]
    
    subgraph CD_Pipeline
        ManifestRepo -->|Pull Manifests| ArgoCD[Argo CD Core]
        ArgoCD -->|Deploy & Sync| K8s[Kubernetes Cluster]
    end

    Jenkins -.->|Status Notification| Slack[Slack Channel]
