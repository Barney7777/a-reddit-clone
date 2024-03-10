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

Step1: Run terraform on your local machine
- we will store state file in S3 bucket and lock it in dynamodb table
- create iam role with policy of AdministratorAccess and attach to ec2 instance
- create security group to open necessary ports
- create ec2 instance with t2.large and 30 ebs volume
- we will use user-data to install jenkins, docker, trivy, kubectl, awscli and eksctl while ec2 is creating

```sh
terraform init
terraform fmt
terraform validate
terraform apply --auto-approve
```


<img width="869" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/f1418f0b-38ee-4028-ad0d-c5ecf35599b3">

<img width="1263" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/ef673017-d0f6-48fd-afb3-692f3986fae0">


Step2: install eks cluster by using eksctl on jenkins server

```sh
eksctl create cluster --name reddit-cluster \
--region ap-southeast-2 \
--node-type t2.small \
--nodes 3 \
```
wait 10-20 minutes until cluster is created

<img width="888" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/aa0d991b-c38c-4cd6-be9a-8c0b12319c9f">

Verify Cluster with below command

```sh
kubectl get nodes
kubectl get svc
```

<img width="785" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/62b1a5a0-1cff-4b8b-b8a8-84fcad45aa33">

Step3: On Jenkins UI, install the following plugin

go to jenkins-> plugins-> available plugins

Blue Ocean, Eclipse Temurin Installer, SonarQube Scanner,  NodeJs Plugin, Docker, Docker commons, Docker pipeline, Docker API, Owasp Dependency Check

<img width="1917" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/e0c7edf7-db6d-44ad-9dfe-9f2d484a0f3c">


Step4: Configuration on tool jenkins tool and system, create credentials for sonarqube, docker and github token

go to manage jenkins-> Tools

<img width="1647" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/88efebb9-29e3-49bf-b67e-cda76f89bf54">

<img width="416" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/5df69989-bc79-43c3-bec2-a99ab7b863f8">

<img width="516" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/2ec0df3d-e815-4234-9851-3452391b450b">

<img width="316" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/ecc0d05d-f892-45f0-b90f-04d96fb0dc9b">

go to manage jenkins-> credentials

provide your dockerhub name and password/token

<img width="717" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/e0722fb5-a96d-481c-aa7b-9e5d1c2807b6">

go to sonarqube dashboard, administration->security->users->generate token

put this token on jenkins credentials

<img width="781" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/6816eeca-0812-46f5-917f-7ea441a1d9c9">

go to github-> settings-> developer settings-> personal access tokens-> Token(classic)->generate token

put this token on jenkins credentials

<img width="751" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/7d103d4e-d1fe-43cd-add6-8251a03cab2d">

go to manage jenkins-> system-> sonarqube server

<img width="1580" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/64340a43-6468-4de9-a624-99ed583db473">


Step5: Configure Sonarqube

go to Sonarqube dashboard-> administration-> configuration-> Webhook-> create

<img width="889" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/b5812205-1bd3-42ef-8019-facd29d5a16c">

go to Sonarqube dashboard-> project-> manually

<img width="457" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/a9fb26ee-8cb2-4263-be8d-e62056c87276">

after that, choose manually-> using existing token, put your token which is generated before

<img width="690" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/0d150cf8-d42b-412b-9509-f2abbaf06928">

than, continue-> others->linux-> you will get commands like following

<img width="1298" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/31e069e8-c9b6-40b0-a916-27b50d6d1586">


Step6: Notification

install Email Extention Plugin in jenkins

Go to your Gmail and click on your profile

Then click on Manage Your Google Account --> click on the security tab on the left side panel you will get password(make sure your 2-step verification is enabled)

Click on Manage Jenkins--> credentials and add your mail username and generated password

<img width="776" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/92a861d9-5d63-406d-9a60-b7754b67c37b">


click on manage Jenkins --> configure system there under the E-mail Notification section configure the details as shown in the below image

<img width="1603" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/bceae6a8-953a-4f9a-90cb-bedc7e9f1ac0">

<img width="1621" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/15152e3f-21e0-404d-b027-a61e9e5353b2">

Now under the Extended E-mail Notification section configure the details as shown in the below images

<img width="476" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/02709f9e-4c9a-4bd7-a30e-e7318229df98">

<img width="204" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/cae03574-d0b9-4c75-a0bb-c56baa87cdfb">

<img width="369" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/ca366925-28b9-41b2-97b8-75595b51f23d">

Step7: Monitor

Install Helm Chart

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

<img width="1083" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/a8de62a5-202f-40be-bc48-56fba26b9645">

Install promethues(Grafana will be coming along with Prometheus as the stable version)

```sh
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace prometheus
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
kubectl get pods -n prometheus
kubectl get svc -n prometheus
```
<img width="936" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/6dc7b594-402d-4666-a958-4e0c2fc1aa8a">


let’s expose Prometheus to the external world

```sh
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus //change it from Cluster IP to LoadBalancer.change port & targetport to 9090, save and close
kubectl get svc -n prometheus
```

<img width="582" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/8bd19d9c-8c57-4614-92f0-f3752ff26318">

<img width="1505" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/e598c675-9c95-4a81-a4bc-e6d6444d0236">

copy dns name of LB and browse with 9090

<img width="1749" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/028a42e9-a50e-4a24-9098-552f34de2015">

let’s change the SVC file of the Grafana and expose it to the outer world

```sh
kubectl edit svc stable-grafana -n prometheus // change it from Cluster IP to LoadBalancer
kubectl get svc -n prometheus
```

<img width="320" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/61c0f907-e9ff-4eee-805e-0447d06e499c">

<img width="1489" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/610dacdb-27dc-4daa-82b0-72076589a781">

user name is admin

password

```sh
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

<img width="1098" alt="image" src="https://github.com/Barney7777/a-reddit-clone/assets/122773145/d23da88a-72fd-4ee0-b697-b5e40dcdd13b">












  
  

  
