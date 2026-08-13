# DevOps CI Project – Java Web Application (Tomcat + Docker on AWS EC2)

A CI pipeline that builds a Java web application with Maven, packages it as a WAR file, deploys it into a Tomcat-based Docker image, and pushes that image to Docker Hub — fully automated through Jenkins running on a single AWS EC2 instance.

**Application repo:** [DevopsProject-java-webapplication](https://github.com/mithunku060/DevopsProject-java-webapplication)

---

## Architecture Overview

```
GitHub (source) ─▶ Jenkins (build/CI) ─▶ Docker (build image) ─▶ Docker Hub (registry)
                        │
                        ├── Maven compile
                        ├── Maven package (WAR)
                        ├── Docker image build (Tomcat + WAR)
                        ├── Run container (local test, port 9008)
                        └── docker login + push to Docker Hub
```

**Infrastructure:**
| Node | Instance Type | OS | Role |
|------|---------------|-----|------|
| Project2 (Project2-master) | c7i-flex.large | Ubuntu 24.04 | Jenkins, Docker, build & containerization |

---

## Tech Stack

- **Source Control:** GitHub
- **CI:** Jenkins (Pipeline, Eclipse Temurin Installer plugin)
- **Build Tool:** Maven
- **Language/Runtime:** Java 11 (JDK), deployed on Apache Tomcat
- **Containerization:** Docker
- **Container Registry:** Docker Hub
- **Cloud:** AWS EC2

---

## AWS EC2 Setup

A single EC2 instance (`Project2`) was provisioned with the following security group rules:

| Port | Purpose |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 8080 | Jenkins UI |

> Note: the app container is run locally on the host with port `9008` mapped to the container's `8080` (see Jenkinsfile) — open port `9008` in the security group as well if you need to reach the running container from outside the instance.

---

## Setup Steps

### 1. Base OS update
Connected to the EC2 instance (`Project2-master`) as root:
```bash
sudo apt update
sudo apt upgrade
```

### 2. Install Jenkins
```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins
```
Jenkins UI is then available at `http://<instance-ip>:8080`.

### 3. Install Docker
```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo docker run hello-world
systemctl status docker
```

### 4. Give Jenkins permission to run Docker
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### 5. Configure Jenkins
- Access Jenkins at `http://<instance-ip>:8080` and complete initial setup.
- Add **GitHub** and **Docker Hub** credentials under **Manage Jenkins → Credentials → Global domain** (Docker Hub credential ID used in the pipeline: `dockerHub`).
- Install the **Eclipse Temurin Installer** plugin.
- Configure **Maven** and **Java 11 JDK** tools under **Manage Jenkins → Tools** (tool names used in the pipeline: `java-11`, `maven`).

### 6. Pipeline (Jenkinsfile)
The pipeline (stored in the repo as `Jenkinsfile`) runs the following stages:

1. **Git-checkout** — pulls the `main` branch from the [application repo](https://github.com/mithunku060/DevopsProject-java-webapplication.git).
2. **code compile** — `mvn compile`
3. **code package** — `mvn clean install` (produces the WAR artifact)
4. **Build and tag** — builds the Docker image: `docker build -t mkumar060/devproject-01 .`
5. **Containerization** — runs the image locally for testing: `docker run -it -d --name cont1 -p 9008:8080 mkumar060/devproject-01`
6. **Log in to Docker Hub** — authenticates using the `dockerHub` Jenkins credential.
7. **Pushing image to repository** — pushes the image: `docker push mkumar060/devproject-01`

### 7. Dockerfile
The application image (in the repo as `Dockerfile`) is based on Tomcat and deploys the built WAR file:
```dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY webapp/target/webapp.war /usr/local/tomcat/webapps
```

---

## Accessing the Application

Once the pipeline runs, the container is reachable on the EC2 instance at:
```
http://<instance-public-ip>:9008
```

The built image is also available on Docker Hub at `mkumar060/devproject-01`.

---

## Notes / Possible Improvements

- Restrict security group rules to specific IP ranges instead of open access.
- Add automated tests (`mvn test`) as a pipeline stage before packaging.
- Stop/remove any existing container (`cont1`) before re-running the pipeline, or use a rolling deployment approach, to avoid "container name already in use" errors on repeated builds.
- Add a container image scanner (e.g. Trivy) as a pipeline stage before pushing to Docker Hub.
- Consider deploying the pushed image to a separate host or orchestrator (e.g. Kubernetes) rather than running it on the same EC2 instance as Jenkins.
