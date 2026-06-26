# StreamingApp — DevOps Orchestration & Scaling Project
## HeroVired DevOps Programme — Graded Capstone Project

## Architecture Overview
A MERN-based streaming application deployed via a full DevOps pipeline on AWS.

## Tech Stack
| Component | Technology |
|---|---|
| Frontend | React + Nginx |
| Auth Service | Node.js/Express |
| Streaming Service | Node.js/Express |
| Admin Service | Node.js/Express |
| Chat Service | Node.js/Express (Socket.io) |
| Database | MongoDB 6 |
| Container Registry | Amazon ECR |
| CI Pipeline | Jenkins |
| Orchestration | Amazon EKS (Kubernetes 1.34) |
| Package Manager | Helm v4 |
| Monitoring | Amazon CloudWatch |

## Repository Structure
## Services & Ports
| Service | Port | Kubernetes Type |
|---|---|---|
| Frontend | 80 | LoadBalancer |
| Auth Service | 3001 | ClusterIP |
| Streaming Service | 3002 | ClusterIP |
| Admin Service | 3003 | ClusterIP |
| Chat Service | 3004 | ClusterIP |
| MongoDB | 27017 | ClusterIP |

## Step-by-Step Deployment

### Prerequisites
- AWS CLI configured with IAM user credentials
- Docker Desktop
- kubectl, eksctl, helm installed

### 1. Fork and Clone
```bash
git clone https://github.com/sanojaix-debug/StreamingApp.git
cd StreamingApp
cp .env.example .env
```

### 2. Local Testing with Docker Compose
```bash
docker-compose build
docker-compose up -d
docker-compose ps
# Frontend: http://localhost:3000
```

### 3. Push Docker Images to Amazon ECR
```bash
aws ecr create-repository --repository-name streamingapp-frontend --region ap-south-1
aws ecr create-repository --repository-name streamingapp-auth --region ap-south-1
aws ecr create-repository --repository-name streamingapp-streaming --region ap-south-1
aws ecr create-repository --repository-name streamingapp-admin --region ap-south-1
aws ecr create-repository --repository-name streamingapp-chat --region ap-south-1

aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 431445330886.dkr.ecr.ap-south-1.amazonaws.com

docker-compose build
docker tag streamingapp-frontend:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:latest
docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:latest
# Repeat for all services
```

### 4. Jenkins CI Pipeline
- URL: https://jenkinsacademics.herovired.com/job/sanoj-streamingapp-ci/
- Triggers: GitHub webhook on push to main branch
- Stages: Checkout → Build Images → Push to ECR

### 5. Create EKS Cluster
```bash
eksctl create cluster \
  --name streamingapp-cluster \
  --region ap-south-1 \
  --nodegroup-name standard-nodes \
  --node-type t3.small \
  --nodes 2 \
  --managed \
  --zones ap-south-1a,ap-south-1b
```

### 6. Deploy with Helm
```bash
aws eks update-kubeconfig --name streamingapp-cluster --region ap-south-1
helm install streamingapp ./streamingapp --namespace default
kubectl get pods
kubectl get svc
```

### 7. Monitoring
```bash
# Enable EKS cluster logging
aws eks update-cluster-config \
  --name streamingapp-cluster \
  --region ap-south-1 \
  --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enabled":true}]}'

# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "StreamingApp-HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --region ap-south-1
```

## Live Application
- Frontend: http://a6a6596b9f9ce462d9f12296d3cf70e6-966217773.ap-south-1.elb.amazonaws.com

## ECR Repositories
| Repository | URI |
|---|---|
| Frontend | 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend |
| Auth | 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-auth |
| Streaming | 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-streaming |
| Admin | 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-admin |
| Chat | 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-chat |

## Author
Sanoj Thanasekaran
HeroVired DevOps Programme
