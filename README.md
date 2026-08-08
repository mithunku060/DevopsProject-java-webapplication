# DevOps Project: Java Web Application

A complete end-to-end DevOps CI/CD pipeline demonstrating automated code checkout, build, containerization, and deployment for a Java web application.

---

## 🏗️ Architecture & Tech Stack
* **OS:** Ubuntu (AWS EC2 Instance)
* **Version Control:** Git & GitHub (`https://github.com/mithunku060/DevopsProject-java-webapplication.git`)
* **CI/CD Automation:** Jenkins (configured with Maven & Java 11 JDK tools)
* **Containerization:** Docker & Docker Hub (`mkumar060/devproject-01`)
* **Web/Application Server:** Apache Tomcat (running inside the Docker container)

---

## 🔄 CI/CD Pipeline Workflow

1. **Source Code Management:** The pipeline pulls the latest code from the GitHub repository using **Git**.
2. **Build Stage:** **Maven** and **JDK 11** (managed via Jenkins tools) compile the project and generate the build artifact at `/webapp/target/webapp.war`.
3. **Containerization:** **Docker** builds an image using the project's `Dockerfile` and tags it as **`mkumar060/devproject-01`**.
4. **Registry Push:** The built image is securely pushed to **Docker Hub**.
5. **Deployment:** A container is launched from the image, mapping **Port 9008** on the host to **Port 8080** inside the container running Tomcat.

### 🔑 Important Jenkins Docker Permission Fix
To allow Jenkins to execute Docker commands without running into permission denied errors, grant the Jenkins user access to the Docker socket by running:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
---

## 🚀 How to Run Locally / Manually

### 1. Clone the Repository
```bash
git clone [https://github.com/mithunku060/DevopsProject-java-webapplication.git](https://github.com/mithunku060/DevopsProject-java-webapplication.git)
cd DevopsProject-java-webapplication
