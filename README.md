![image](https://github.com/dv-sharma/three-tier-architecture-handson/assets/65087388/aff06220-ba09-4fff-adb5-a3f6a512f253)

# 1.Three Tier Architecture Deployment on AWS EKS using helm charts
# 2.Observability using Prometheus & Grafana
# 3.Logging with EFK Stack
# 4.CI Pipeline for Microservices
# 5.CD with ArgoCD

#### PROJECT IDEA

I came across this project developed by IBM Instana team ,"Stan's Robot Shop" is a sample microservice application you can use as a sandbox to test and learn containerised application orchestration and monitoring techniques. I picked up this application to practise handson skills for deploying a microservices application on Kubernetes Managed Cluster. This project is perfect to practise building docker containers,Deployment with helm charts, orchestrating with Kubernetes,Montoring of the application & cluster, and demonstrate CI pipeline for microservices & CD with ArgoCD skills.

This sample microservice application has been built using these technologies:
- NodeJS ([Express](http://expressjs.com/))
- Java ([Spring Boot](https://spring.io/))
- Python ([Flask](http://flask.pocoo.org))
- Golang
- PHP (Apache)
- MongoDB
- Redis
- MySQL ([Maxmind](http://www.maxmind.com) data)
- RabbitMQ
- Nginx
- AngularJS (1.x)
More information about the development of this project here: [blog post](https://www.instana.com/blog/stans-robot-shop-sample-microservice-application/)

### TASKS 
- **PART1 DEPLOYING ON KUBERNETES EKS CLUSTER MANUALLY USING HELM CHARTS** First part of the project will be to deploy this application on EKS Cluster.
- **PART2 : SETUP OF MONITORING WITH PROMETHES & GRAFANA**
- **PART3: LOGGING WITH EFK (Elasticsearc,FluentD,Kibana) STACK SETUP**
- **PART4: Build an automation CI pipeline to build & publish images to ECR repo.**
- **PART5: Deploy the microservices application using ArgoCD.**

### PART1 DEPLOYING ON KUBERNETES CLUSTER MANUALLY (USE THE EKS DIRECTORY)
###  ENVIRONMENT
CLOUD & INFRA: AWS EKS
APPLICATION ARCHITECTURE:
The web page is a Single Page Application written using AngularJS (1.x), its resources are served by Nginx which also acts as a reverse proxy for the backend microservices. Those microservices are written in different languages using various frameworks, providing a wide range of example scenarios. MongoDB is used as the data store for the product catalogue and the registered users. MySQL is used for the look up of the shipping information. Redis is used to hold the active shopping carts. The order pipeline is processed via RabbitMQ.
The code already has any required Instana components installed, making it very easy to start monitoring the application with Instana, you just have to install the agent on the platform. Instana End User Monitoring is also preconfigured, you just need to add your unique key to the environment of the Nginx container.




### BUILDING THE PROJECT

**CREATING EKS CLUSTER**

`eksctl create cluster --name demo-cluster-three-tier-1 --region us-east-1`

**LISTING OIDC PROVIDERS**

`aws iam list-open-id-connect-providers`

**ASSOCIATING WITH CLUSTER**

`eksctl utils associate-iam-oidc-provider --cluster demo-cluster-three-tier-1 --approve`

**SETTING UP ALB ADDONS**

DOWNLOAD THE POLICY FROM HERE: https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

**CREATING POLICY**

`aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json`

**INSTALL LOADBALANCER**

`helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --setclusterName=demo-cluster-three-tier-1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0d5bd0e9558c3fc6a`

**CREATE SERVICE ACCOUNT**

`eksctl create iamserviceaccount --cluster=demo-cluster-three-tier-1 --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<AWS-ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy --approve`

**ADD HELM REPO**

`helm repo add eks https://aws.github.io/eks-charts
helm repo update eks`

**INSTALL LOADBALANCER**

`helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=demo-cluster-three-tier-1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0d5bd0e9558c3fc6a`

**CSI-PLUGIN-EBS-CONFIGURATION**

`eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster demo-cluster-three-tier-1 --role-name AmazonEKS_EBS_CSI_DriverRole --role-only --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve`

`eksctl create addon --name aws-ebs-csi-driver --cluster demo-cluster-three-tier-1  --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force`

`eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster demo-cluster-three-tier-1 --role-name AmazonEKS_EBS_CSI_DriverRole --role-only --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve`

`eksctl create addon --name aws-ebs-csi-driver --cluster demo-cluster-three-tier-1  --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/`

**DEPLOY YAMLS WITH HELM CHART**

`helm install robot-shop --namespace robot-shop .
`
**CREATING INGRESS LOAD BALANCER**

`kubectl apply -f .\ingress.yaml`

**VIEW THE PODS**
![image](https://github.com/dv-sharma/three-tier-architecture-handson/assets/65087388/0510be0a-6070-4174-81c8-0d7c2ce067ba)


kubectl get pods -n robotshop


#### ACCESSING THE APPLICATION AT REMOTE HOST PUBLICIP:PORT

The application can be accessed at DNS of Ingress Load Balancer that we have created.

#### Application deployed on EKS Cluster and accessible through instance IP and port 🎉



**FIRST PART ACCOMPLISHED!**
![image](https://github.com/dv-sharma/three-tier-architecture-handson/assets/65087388/343a9b94-bbd4-40fd-9993-16eeaffdfe1d)


## PART2 : SETUP OF MONITORING WITH PROMETHES & GRAFANA

### STEPS
**ADDING METRICS SERVER**
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

**VIEW METRICS**
`kubectl get deployment metrics-server -n kube-system`

`kubectl top nodes`

`kubectl top nodes -n kube-system`

**INSTALL WITH HELM PACKAGE MANAGER**

`helm repo add stable https://charts.helm.sh/stable`

`helm install stable prometheus-community/kube-prometheus-stack`

**VIEW PODS**

`kubectl --namespace default get pods -l "release=stable"`

**MAKE THE SERVICE TYPE OF GRAFANA & PROMETHUES TO LOADBALANCER**

`edit svc stable-kube-prometheus-sta-prometheus`

`kubectl edit svc stable-grafana`

**ACCESS THE GRAFANA & PROMETHEUS THROUGH THE SERVICES EXTERNAL IP**

`kubectl get svc`

**LOGIN INTO GRAFANA:**

USER: admin 
PASSWORD: prom-operator

![image](https://github.com/dv-sharma/three-tier-architecture-handson/assets/65087388/017dfafa-f7a1-4b11-8bdf-fb3478ce113b)
![image](https://github.com/dv-sharma/three-tier-architecture-handson/assets/65087388/849e5cc7-614e-4524-a717-e0a3e56210db)

## PART3: LOGGING WITH EFK STACK SETUP

STEPS FOR SETTING UP EFK STACK : [HERE](https://github.com/dv-sharma/three-tier-architecture-handson/tree/master/EFKStack)

## PART4: Build an automation CI pipeline to build & publish images to ECR repo. & PART5:Deploy the microservices application using ArgoCD.**  
[HERE](https://github.com/dv-sharma/three-tier-architecture-handson/tree/master/ArgoCD)







