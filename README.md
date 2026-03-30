🚀 CI/CD Pipeline for Event Application

This project implements a complete CI/CD pipeline using Jenkins to build, test, scan, containerise, and analyse a Java-based application.

It demonstrates DevOps best practices including automation, security scanning (SAST & DAST), and container security.

📌 Project Overview
The pipeline automates the following workflow:
Source Code Retrieval from GitHub
Build & Test using Maven
Security Scanning (SAST) using Snyk
Secrets Detection using Trivy
Docker Image Build
Dynamic Security Testing (DAST) using OWASP ZAP
Container Vulnerability Scanning using Trivy & Snyk

🧱 Pipeline Architecture
GitHub → Jenkins → Maven → Snyk → Trivy → Docker → ZAP → Trivy Image Scan

⚙️ Technologies Used
CI/CD: Jenkins
Build Tool: Maven
Programming Language: Java
Containerisation: Docker
SAST: Snyk
DAST: OWASP ZAP
Security Scanning: Trivy
Version Control: GitHub

▶️ How to Run
Prerequisites
Jenkins installed with:
Pipeline plugin
Snyk plugin
Docker installed
Trivy installed
Snyk CLI configured
Steps
1/Create a Jenkins Pipeline job
2/Add your Jenkinsfile
3/Configure credentials:
4/Snyk Token
5/Run the pipeline
