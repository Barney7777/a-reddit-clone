# Reddit Clone App on Kubernetes
This project is about to deploy Reddit-Clone-App on EKS by using Jenkins and ArgoCD with security check.
## Workflow
<img width="452" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/cb55f006-8d8f-4416-be49-6fffa62f901e">

- Version Control System: Git
- Git Repo: Github. we will use Github to store our source code and webhook with Jenkins so whevever there are code changes, Jenkins pipeline will be triggered automatically.
- Iac: Terraform. we will use terraform to provision EC2 instance to install Jenkins, Docker, Trivy and Run SonaeQube container in docker.
- For CI Job(Jenkins)

  Dependencies installation -> NPM

  Code Quality Check -> SonarQube(Sonar scanner and Quality Gate)

  Dependencies Check -> OWASP

  Filse Scan -> Trivy

  Container tool  -> Docker. we will use docker to build docker image with different tags.

  Container Registry -> Dockerhub. we will push docker image to Dockerhub.

  Image Scan -> Trivy

  Image updater -> shell script. whenever new image with different tag is built, this step will update the image and tag on the Gitops Repository where there are   manifest files for the Kubernetes and this change will trigger the ArgoCD.

- For CD job(ArgoCD)

  We will use ArgoCD to performance Gitops mechanism. it will continuously watching for the Gitops repo, whenever new image is created, ArgoCD will automaticaly deploy this new image to Kubernetes Cluster.

- For Notificaion -> Gmail. Whenever pipeline is finished, this step will send notifications like building results, logs, reports related to Trivy File scan, image scan and Dependencies check via Gmail.

- For Monitoring -> Promethues and Granafa. we will use these tools to monitor Kubernetes Cluster.

## Stpes
- Step1: Run terraform on your local machine
  

  
  
  

  
