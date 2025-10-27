# End-to-End DevSecOps CI/CD Pipeline Project

## Overview 🚀

This project showcases a complete DevSecOps CI/CD pipeline built using Jenkins, Maven, Docker, Kubernetes, SonarQube, Nexus, Trivy, Prometheus, and Grafana, hosted on AWS EC2. It demonstrates automating the build, test, security scanning, deployment, and monitoring of a Java Spring Boot web application. When code is pushed to GitHub, the pipeline automatically processes it through various quality, security, and deployment stages.

---
## Architecture 🏗️



* **(Developer Push -> GitHub -> Jenkins Trigger)**
* **Jenkins Pipeline Stages:**
    * Checkout -> Compile -> Test (Maven)
    * Dependency Scan (Trivy)
    * Code Analysis -> Quality Gate Check (SonarQube)
    * Package (Maven JAR) -> Publish Artifact (Nexus)
    * Build Docker Image -> Scan Image (Trivy) -> Push Image (Docker Hub)
    * Deploy (Kubernetes) -> Verify Deployment
    * Notification (Email)
* **Monitoring:** Prometheus scrapes metrics from Jenkins (Node Exporter) & Application (Blackbox Exporter), visualized in Grafana.

---
## Tools Used 🛠️

* **Cloud:** AWS EC2
* **CI/CD:** Jenkins
* **Build:** Maven
* **Code Quality:** SonarQube
* **Artifact Repository:** Nexus Repository OSS
* **Containerization:** Docker
* **Security Scanning:** Trivy
* **Container Registry:** Docker Hub
* **Orchestration:** Kubernetes (kubeadm)
* **Monitoring:** Prometheus, Grafana, Node Exporter, Blackbox Exporter
* **Language/Framework:** Java, Spring Boot
* **Source Control:** Git, GitHub

---
## Pipeline Stages Explained ⚙️

1.  **Git Checkout:** Pulls the latest code from the specified GitHub repository and branch.
2.  **Compile:** Cleans the workspace and compiles Java source code using Maven (`mvn clean compile`).
3.  **Test:** Runs unit tests using Maven (`mvn test`).
4.  **File System Scan:** Scans project dependencies for known vulnerabilities using Trivy (`trivy fs`).
5.  **SonarQube Analysis:** Sends the compiled code and test coverage reports to SonarQube for static analysis (`sonar-scanner`).
6.  **Quality Gate:** Pauses the pipeline to check if the SonarQube analysis meets the defined quality standards (e.g., coverage, bugs, vulnerabilities).
7.  **Build:** Packages the application into an executable JAR file (`mvn package`).
8.  **Publish to Nexus:** Deploys the JAR artifact (as a `-SNAPSHOT` version during development) to the Nexus Repository (`mvn deploy`).
9.  **Build Docker Image:** Creates a Docker image containing the application JAR and runtime environment based on the `Dockerfile` (`docker build`).
10. **Docker Image Scan:** Scans the built Docker image for OS and library vulnerabilities (`trivy image`).
11. **Push Docker Image:** Pushes the tagged Docker image to Docker Hub (`docker push`).
12. **Deploy to Kubernetes:** Applies the `deployment-service.yaml` manifest to the Kubernetes cluster using `kubectl apply`.
13. **Verify Deployment:** Checks the status of the newly created pods and service within the Kubernetes namespace (`kubectl get pods/svc`).
14. **Notification:** Sends an HTML email summarizing the build status (Success/Failure) and linking to the Jenkins build log.

---
## Setup & Configuration Notes 📝

* **AWS EC2:** The pipeline requires multiple EC2 instances (Jenkins Master, SonarQube, Nexus, K8s Master/Workers, Monitoring). Ensure Security Groups allow necessary traffic (e.g., Jenkins to SonarQube on port 9000, Jenkins to Nexus on 8081, Jenkins to K8s API on 6443, Prometheus to exporters). Consider using **Elastic IPs** for stable public addresses.
* **Jenkins:**
    * **Plugins:** Requires `Pipeline`, `Docker Pipeline`, `Kubernetes CLI`, `SonarQube Scanner`, `Maven Integration`, `Email Extension`, `Timestamper`, etc.
    * **Global Tool Configuration:** JDK 17, Maven, and SonarQube Scanner installations must be configured with specific names (`jdk17`, `maven`, `sonarqube` used in the `Jenkinsfile`).
    * **Credentials:** Requires configured credentials in Jenkins for:
        * GitHub (ID: `github-token` - Use a Personal Access Token)
        * Docker Hub (ID: `docker-cred`)
        * SonarQube (ID: `sonar-token` - Use a SonarQube User Token)
        * Kubernetes (ID: `k8-cred` - Use the Service Account Token generated via `kubectl describe secret`)
        * Email (ID: `mail-credentials` - Use Gmail username and App Password if using Gmail)
    * **Managed Files:** Requires a "Global Maven settings.xml" file (ID: `global-settings`) containing Nexus server credentials.
* **Kubernetes:** Assumes a cluster set up via `kubeadm`. RBAC is configured using a Service Account, Role, and RoleBinding to grant Jenkins deployment permissions. A namespace (e.g., `webapps`) is used for deployment.
* **Nexus:** Requires `maven-releases` (Policy: Disable redeploy) and `maven-snapshots` (Policy: Allow redeploy) repositories. Credentials (`admin`/`your_password`) needed for deployment.
* **SonarQube:** Default Quality Gate is used unless customized. A Webhook needs to be configured pointing to `[Jenkins_URL]/sonarqube-webhook/` for the `waitForQualityGate` step.
* **Prometheus:** `prometheus.yml` needs scrape configurations for Jenkins (`/prometheus`), Node Exporter (`:9100`), and Blackbox Exporter (`:9115`), targeting the correct instance IPs.
