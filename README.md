# 🚀 Create CI/CD Pipeline with Jenkins, Docker, and Google Kubernetes Engine (GKE)

In this blog, I’ll walk you through how I built a **fully automated CI/CD pipeline** using Jenkins that integrates with **GitHub, Maven, Docker, DockerHub, Tomcat, and GKE**. This pipeline takes code from source, builds & tests it, packages it as a Docker image, pushes it to DockerHub, and finally deploys it to a Kubernetes cluster running on Google Kubernetes Engine (GKE). All the tools were installed and configured on a **Google Compute Engine (GCE) VM**, which served as my Jenkins controller and execution environment.

---

## 🔗 Tools & Integrations

Here’s the tech stack we’re using:

* **Jenkins** → Automation server to orchestrate the pipeline.
* **GitHub** → Source code repository.
* **Maven** → Build & dependency management tool for Java.
* **Docker** → Containerization.
* **DockerHub** → Image registry.
* **Tomcat** → Application server to run the Java web app.
* **GKE (Google Kubernetes Engine)** → Kubernetes cluster where we deploy the app.

Each of these tools is connected to Jenkins with proper authentication:

* **GitHub** → Integrated via webhook or Jenkins Git plugin with a Personal Access Token (PAT).
* **Maven** → Configured as a global tool in Jenkins.
* **DockerHub** → Credentials (username & password/token) stored in Jenkins Credentials Manager.
* **GCP/GKE** → Service account JSON key uploaded to Jenkins credentials, used by the GKE Jenkins plugin.

--------------

### 🔍 Explanation of Jenkinsfile(Line by Line)

* **Scm Checkout** → Pulls code from GitHub.
* **Build** → Compiles and packages the application using Maven.
* **Test** → Runs unit tests to verify build correctness.
* **Build Docker Image** → Uses Dockerfile to build a container image.
* **Push Docker Image** → Pushes the image to DockerHub securely with Jenkins credentials.
* **Deploy to GKE** → Updates Kubernetes manifests and deploys the container to GKE.

---

## 📊 Jenkins Pipeline in Action

Here’s the execution of my pipeline in Jenkins:

*"Here’s the Jenkins pipeline execution for my end-to-end CI/CD flow. Each stage (SCM checkout, build, test, Docker image build & push, and deployment to GKE) completed successfully — fully automated from code to Kubernetes!"*

---

## ✅ Key Takeaways

* Jenkins integrates seamlessly with GitHub, Maven, Docker, DockerHub, Tomcat, and GKE.
* Store all secrets securely in Jenkins credentials.
* Automating with Jenkinsfile ensures reproducibility & traceability.
* Each code commit → Docker image pushed → App deployed on GKE.
* Installing and running all tools on a **GCE VM** provides a consistent, cloud-based CI/CD environment.

This is **true DevOps in action** 💡 — transforming the way we deliver software.
