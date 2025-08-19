# DevSecOps Multi-Node Home Lab 🚀

Multi-Node DevSecOps Pipeline Architecture

This project builds a multi-node DevSecOps pipeline in a home lab using **free and open-source tools**. It demonstrates how software can move from development → testing → security scanning → deployment → monitoring in a **secure, automated, and production-like environment**.

---

## 🔹 High-Level Architecture

* **Developer Node** – Source code & Git repo (GitHub or GitLab)
* **Jenkins Node** – CI/CD Orchestration + SonarQube container
* **Ansible Node** – Configuration management & deployment
* **Docker Node** – Build & run containerized apps
* **Harbor Node** – Private container registry with vulnerability scanning
* **Monitoring Node** – Prometheus + Grafana + Alertmanager

![Architecture Diagram](docs/architecture.png)

---

## 🔹 Features

* ✅ **CI/CD Pipeline** with Jenkins
* ✅ **Code Quality & Security Scanning** (SAST, SCA, DAST)
* ✅ **Container Image Scanning** with Harbor & Trivy
* ✅ **Automated Deployments** with Ansible + Docker
* ✅ **Monitoring & Alerts** with Prometheus + Grafana
* ✅ **Multi-Node Setup** for realistic enterprise simulation

---

## 🔹 Prerequisites

* At least **4–5 Linux VMs** (Ubuntu 22.04, RHEL 9, or CentOS 7/AlmaLinux)
* Installed on each VM:

  * `ssh`, `git`, `curl`, `wget`
  * Docker & Docker Compose (for Jenkins/Harbor/Monitoring nodes)
* GitHub/GitLab account for source control
* Minimum 4 GB RAM for Jenkins VM (SonarQube is resource-heavy)

---

## 🔹 Step 1: Clone Repository

```bash
git clone https://github.com/<your-username>/devsecops-home-lab.git
cd devsecops-home-lab
```

---

## 🔹 Step 2: Start CI/CD Services (Jenkins Node)

```bash
cd docker
docker-compose up -d --build
```

### Services Available

* Jenkins → [http://localhost:8080](http://localhost:8080)
* SonarQube → [http://localhost:9000](http://localhost:9000)
* Harbor → [http://localhost](http://localhost) (admin / Harbor123!)

---

## 🔹 Step 3: Configure Jenkins

1. Unlock Jenkins:

   ```bash
   cat /var/jenkins_home/secrets/initialAdminPassword
   ```
2. Install **suggested plugins**.
3. Add Jenkins agents if you want to distribute builds.
4. Install required plugins:

   * Git
   * AnsiColor
   * SonarQube Scanner
   * Docker
   * Pipeline
   * Blue Ocean

---

## 🔹 Step 4: Integrate SonarQube

1. Log in to SonarQube → create a project.
2. Generate a token.
3. Add SonarQube credentials in Jenkins (`Manage Jenkins → Credentials`).
4. Update Jenkinsfile with analysis stage:

```groovy
stage('Code Analysis') {
  steps {
    script {
      sh 'sonar-scanner -Dsonar.projectKey=myapp -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=$SONAR_TOKEN'
    }
  }
}
```

---

## 🔹 Step 5: Harbor Setup

1. Access Harbor at [http://localhost](http://localhost).
2. Login with `admin / Harbor123!`.
3. Create a project (e.g., `devsecops`).
4. Push images:

   ```bash
   docker login harbor.local
   docker tag myapp:latest harbor.local/devsecops/myapp:latest
   docker push harbor.local/devsecops/myapp:latest
   ```

---

## 🔹 Step 6: Ansible Deployment

* Inventory (`inventory.ini`):

```ini
[docker_nodes]
192.168.56.20 ansible_user=devops
```

* Playbook (`deploy.yml`):

```yaml
- hosts: docker_nodes
  become: true
  tasks:
    - name: Pull image from Harbor
      docker_image:
        name: harbor.local/devsecops/myapp:latest
        source: pull
    - name: Run container
      docker_container:
        name: myapp
        image: harbor.local/devsecops/myapp:latest
        state: started
        ports:
          - "8081:80"
```

Run:

```bash
ansible-playbook -i inventory.ini deploy.yml
```

---

## 🔹 Step 7: Security Scanning

* **SAST (Static)** → SonarQube
* **SCA (Dependencies)** → Snyk / OWASP Dependency Check
* **Container Scans** → Trivy (integrated with Harbor)
* **DAST (Runtime)** → OWASP ZAP

Example Trivy scan:

```bash
trivy image harbor.local/devsecops/myapp:latest
```

---

## 🔹 Step 8: Monitoring (Prometheus + Grafana)

1. Start containers:

   ```bash
   cd monitoring
   docker-compose up -d
   ```
2. Access:

   * Prometheus → [http://localhost:9090](http://localhost:9090)
   * Grafana → [http://localhost:3000](http://localhost:3000) (admin / admin)
3. Import Jenkins & Docker dashboards.
4. Configure alerts in Alertmanager (email, Slack, etc).

---

## 🔹 Example Jenkinsfile

```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { git 'https://github.com/<your-repo>/sample-app.git' }
    }
    stage('Build') {
      steps { sh 'docker build -t myapp:latest .' }
    }
    stage('Code Analysis') {
      steps { sh 'sonar-scanner' }
    }
    stage('Unit Tests') {
      steps { sh 'pytest tests/' }
    }
    stage('Security Scan') {
      steps { sh 'trivy image myapp:latest' }
    }
    stage('Push to Harbor') {
      steps {
        sh '''
          docker tag myapp:latest harbor.local/devsecops/myapp:latest
          docker push harbor.local/devsecops/myapp:latest
        '''
      }
    }
    stage('Deploy with Ansible') {
      steps { sh 'ansible-playbook -i inventory.ini deploy.yml' }
    }
  }
}
```

---

## 🔹 Credentials & Defaults

* Jenkins UI → `http://<jenkins-node>:8080`
* SonarQube → `http://<jenkins-node>:9000` (`admin / admin`)
* Harbor → `http://<harbor-node>` (`admin / Harbor123!`)
* Grafana → `http://<monitor-node>:3000` (`admin / admin`)

---

## 🔹 Why This Project Matters

* **Hands-on simulation** of enterprise DevSecOps
* **Multi-layer security** with SAST + SCA + DAST + container scans
* **CI/CD automation** builds confidence in modern delivery practices
* **End-to-end monitoring** ensures reliability in production
* **Great talking point for interviews & resumes**

---

✅ This README now covers all services, configs, credentials, monitoring, and security scanning for a multi-node DevSecOps home lab.

