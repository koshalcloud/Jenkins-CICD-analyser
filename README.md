 AI-Powered CI/CD Failure Analysis Pipeline on Kubernetes
A production-style DevSecOps CI/CD pipeline that doesn't just build and deploy — when a pipeline stage fails, it automatically analyzes the logs using a local AI model (Ollama) and posts a full Root Cause Analysis report to Slack.

No manual log-diving. No external AI API calls — everything runs privately inside the Kubernetes cluster.

<img width="1672" height="941" alt="Architecture" src="https://github.com/user-attachments/assets/fe721d3f-308c-44ed-afef-b1ca6779c5c4" />



🔗 Pipeline Flow

GitHub → Jenkins (Dynamic K8s Agents) → Kaniko Build → Trivy Security Gate
        → Kubernetes Deploy → Prometheus/Grafana
        → [on failure] → Ollama AI RCA → Slack Alert
✨ Key Features
Feature	Description
🔄 Dynamic Jenkins Agents	Ephemeral Kubernetes pods per build — no static agents
🐳 Daemonless Builds	Docker images built with Kaniko — no docker.sock mounted, safer for K8s
🛡️ Security Gate	Trivy scans every image and blocks deployment on HIGH/CRITICAL vulnerabilities
☸️ Kubernetes Deploy	Rolling deployment with readiness/liveness probes + HPA autoscaling
🧠 Local AI Root Cause Analysis	Ollama (tinyllama) analyzes failure logs entirely inside the cluster — private, no external API
💬 Slack Notifications	Success + AI-generated failure reports (Root Cause, Evidence, Fix, Next Action) sent automatically
📊 Monitoring	Prometheus scrapes Jenkins + app metrics, visualized in Grafana
🧰 Tech Stack
CI/CD: Jenkins (Kubernetes plugin, dynamic pod agents) Containers: Docker, Kaniko Security: Trivy (vulnerability scanning, exit-code gated) Orchestration: Kubernetes (kops-provisioned cluster), Helm AI: Ollama (tinyllama model) Notifications: Slack Incoming Webhooks Monitoring: Prometheus, Grafana (via kube-prometheus-stack) Application: Node.js (Express) + prom-client for metrics Automation: Python (AI analyzer + Slack integration)

🏗️ Architecture
The cluster is organized into three namespaces:

jenkins — Jenkins controller + dynamic build agent pods (node, kaniko, trivy, kubectl, python containers)
ai-cicd — Ollama LLM deployment + the deployed Node.js application (HPA-enabled)
monitoring — Prometheus + Grafana stack
On every git push:

Jenkins spins up a temporary agent pod with 5 purpose-built containers
Code is tested (npm ci + npm test)
Image is built and pushed with Kaniko (no Docker daemon)
Trivy scans the image — HIGH/CRITICAL CVEs block the pipeline
If scan passes, the app is rolled out to Kubernetes
If any stage fails, logs are collected and sent to Ollama for RCA, then posted to Slack
Prometheus scrapes Jenkins + app metrics; Grafana visualizes them
📸 Screenshots
Jenkins Pipeline — Stage View
<img width="1903" height="919" alt="Screenshot 2026-07-19 160830" src="https://github.com/user-attachments/assets/ce28030c-cc03-4c04-b004-8cd99eaab5f9" />


AI Root Cause Analysis in Slack
<img width="1919" height="672" alt="Screenshot 2026-07-19 160735" src="https://github.com/user-attachments/assets/a4fafb03-1c59-40f2-8d99-d66c64e64178" />
<img width="1914" height="675" alt="Screenshot 2026-07-19 160801" src="https://github.com/user-attachments/assets/7363e36c-4ee6-43c5-8c73-5f1046460df4" />


Prometheus Targets
<img width="1919" height="910" alt="Screenshot 2026-07-19 160400" src="https://github.com/user-attachments/assets/afc46ae7-80ef-4eaf-b9d2-1b3a4bdc49d6" />


Grafana Dashboard
<img width="1916" height="915" alt="Screenshot 2026-07-19 160448" src="https://github.com/user-attachments/assets/7782af13-f45b-4960-8e59-037e23039b72" />


🚦 Security Gate in Action
This isn't a passive scan — the pipeline actively blocks unsafe deployments:


Trivy Security Scan   ✅  Working — blocks deployment on HIGH/CRITICAL CVEs
Kubernetes Deploy     ⏭️  Skipped when the security gate fails
The Trivy image is pinned to a specific version (public.ecr.aws/aquasecurity/trivy:0.71.0) instead of latest, following the March 2026 Trivy Docker Hub supply-chain compromise disclosure — a deliberate production-hardening choice.

📁 Project Structure

ai-cicd-failure-analyzer/
├── app/                    # Node.js application
│   ├── package.json
│   ├── server.js
│   └── server.test.js
├── k8s/                    # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── ai-analyzer/
│   └── analyze_failure.py  # Ollama-based RCA + Slack integration
├── Dockerfile
├── Jenkinsfile
└── README.md
⚙️ How It Works — Failure Path

Pipeline stage fails
        ↓
Logs collected from workspace
        ↓
Python script sends logs to Ollama (local LLM)
        ↓
Ollama generates: Root Cause / Evidence / Fix / Next Action
        ↓
Report posted to Slack
If Ollama is unreachable, the analyzer falls back to a rule-based pattern match (npm errors, Jest failures, Docker build issues, Trivy CVEs, K8s rollout failures) so a report is always delivered — no silent failures.

🚀 Getting Started
Provision a Kubernetes cluster (this project uses kops)
Install kube-prometheus-stack and Jenkins via Helm
Deploy Ollama inside the cluster and pull the tinyllama model
Add Jenkins credentials: dockerhub-creds (DockerHub) and slack-webhook (Slack)
Create a Jenkins Pipeline job pointing at this repo's Jenkinsfile
Push code and watch the pipeline run 🎉
Full setup notes and troubleshooting steps are in docs/SETUP_NOTES.md.

🔮 Possible Improvements
Swap tinyllama for a larger local model for richer RCA (tradeoff: latency/resources)
Add Loki for centralized log aggregation alongside Prometheus/Grafana
GitHub Actions webhook trigger instead of poll SCM
Slack thread replies instead of new messages per build

👤 Author
kaushal karma 🔗 GitHub
