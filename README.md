# DevSecOps Multi-Node Home Lab (Open Source)

Multi-Node DevSecOps Pipeline Architecture

A production-style, **multi-node** DevSecOps pipeline using free/open-source tools.

## 📐 Architecture (High-Level)
See `docs/architecture.png` for a visual.
- **Jenkins Node**: Jenkins (CI/CD), SonarQube (SAST)
- **Docker Node**: Harbor (image registry), App runtime
- **Ansible Node**: Infrastructure-as-Code and deployments
- **Monitoring Node**: Nagios Core (or Prometheus + Grafana)

```
Developer → GitHub → Jenkins → (SAST) SonarQube
                         ↓
                     Build Image → Harbor (push)
                         ↓
                 Ansible → Deploy to Docker node
                         ↓
                  Monitoring watches all nodes
```

## 🧱 Prerequisites
- 4 Linux VMs/servers (Ubuntu 22.04 or similar). Suggested specs:
  - Jenkins+SonarQube: 4 vCPU / 8GB RAM
  - Docker+Harbor: 4 vCPU / 8GB RAM
  - Ansible: 2 vCPU / 4GB RAM
  - Monitoring: 2 vCPU / 4GB RAM
- SSH connectivity between nodes, and DNS or `/etc/hosts` entries for easy names.
- Docker & Docker Compose on Jenkins and Docker nodes.
- Git on all nodes. Ansible on the Ansible node.

## 🌐 Example Host Mapping
Update IPs to your environment and keep consistent across files.
```
192.168.1.10  jenkins-node
192.168.1.11  docker-node
192.168.1.12  ansible-node
192.168.1.13  monitor-node
```

## 🚀 Step-by-Step Setup

### 1) Clone this repo (on all nodes where applicable)
```bash
git clone https://github.com/YOURUSERNAME/devsecops-multinode-home-lab.git
cd devsecops-multinode-home-lab
```

### 2) Jenkins + SonarQube (on jenkins-node)
```bash
cd docker
docker compose -f jenkins-sonarqube.yml up -d
# Jenkins  : http://jenkins-node:8080
# SonarQube: http://jenkins-node:9000 (admin/admin → then change password)
```
**Jenkins initial unlock:**
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
**Install plugins:** Git, Pipeline, Docker, Docker Pipeline, Credentials, Ansible, SonarQube Scanner, Warnings NG.

**Credentials to add in Jenkins:**
- `github-pat` (Secret text) – for private repos/webhooks
- `harbor-creds` (Username/Password or robot)
- `ansible-ssh` (SSH Username with private key)
- `sonarqube-token` (Secret text)

**Configure SonarQube in Jenkins:**
Manage Jenkins → System → *SonarQube servers* → add `http://jenkins-node:9000` + token.

### 3) Harbor (on docker-node) — Summary
Harbor is a multi-container stack. Quick outline:
```bash
wget https://github.com/goharbor/harbor/releases/download/v2.11.0/harbor-online-installer-v2.11.0.tgz
tar xzf harbor-online-installer-v2.11.0.tgz && cd harbor
cp harbor.yml.tmpl harbor.yml
# Edit harbor.yml:
#   hostname: docker-node (or IP)
#   harbor_admin_password: StrongPasswordHere
./install.sh
```
Create a **project** (e.g., `library`) and a **robot account** (push/pull).

### 4) Ansible node
Update inventory and vars:
```bash
cd ansible
# edit inventory.ini and group_vars/all.yml with real IPs and Harbor robot creds
ansible all -i inventory.ini -m ping
```
Deploy app later via pipeline or manually:
```bash
ansible-playbook -i inventory.ini playbooks/deploy_app.yml
```

### 5) Jenkins Pipeline
Create a Pipeline job that uses `jenkins/Jenkinsfile`. Update the environment section with your node IPs and credentials IDs. Then commit/push a change to trigger the pipeline:
- Checkout → SAST (SonarQube) → SCA → Build → Trivy Scan → Push to Harbor → Ansible Deploy → (optional) ZAP DAST

### 6) Monitoring (Monitoring node)
- **Nagios Core (quick OSS):**
  - Install packages, add hosts `jenkins-node`, `docker-node`, `ansible-node`, your app endpoint.
- **Prometheus + Grafana (alternative):**
  - Run Prometheus/Grafana in Docker and add exporters (`node_exporter`, `cAdvisor`, Jenkins exporter).

## 🔧 Files of Interest
- `docker/jenkins-sonarqube.yml` — Compose for Jenkins + SonarQube
- `jenkins/Jenkinsfile` — CI/CD pipeline (multi-node)
- `jenkins/zap-scan.sh` — DAST baseline runner
- `ansible/inventory.ini` — node mapping
- `ansible/group_vars/all.yml` — registry/image variables
- `ansible/playbooks/deploy_app.yml` — deploys app container on docker-node
- `docker/Dockerfile` + `src/index.html` — demo application (static)

## 🧹 Teardown
```bash
# On jenkins-node
cd docker && docker compose -f jenkins-sonarqube.yml down -v
# On docker-node
cd harbor && docker compose down -v   # if installed via compose
# Containers on docker-node
docker rm -f demoapp || true
```

## 📈 Future Enhancements
- Replace Harbor HTTP with HTTPS (self-signed or real certs)
- Add GitOps (ArgoCD) and Kubernetes (k3s) target
- Add Terraform for infra provisioning
- Expand security gates (policy-as-code, OPA/Conftest)

---

**Author**: Your Name — Built for educational and portfolio use.
