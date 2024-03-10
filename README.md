# Reddit Clone App on Kubernetes
This project is about to deploy Reddit-Clone-App on EKS by using Jenkins and ArgoCD
## Workflow
<img width="452" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/cb55f006-8d8f-4416-be49-6fffa62f901e">

- Version Control System: Git
- Git Repo: Github. we will use Github to store our source code and webhook with Jenkins so whevever there are code changes, Jenkins pipeline will be triggered automatically.
- Iac: Terraform. we will use terraform to provision EC2 instance to install Jenkins, Docker, Trivy and Run SonaeQube container in docker.
- For CI part

  Code Quality Check -> SonarQube
