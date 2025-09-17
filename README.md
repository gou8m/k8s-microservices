# Kubernetes Microservices Deployment on AWS EKS

This repository contains the setup and deployment of a microservices-based application on **AWS EKS** using **Docker**, **Jenkins**, and **Terraform**.

---

## Prerequisites

- AWS account with necessary permissions.
- EC2 instance (t2.medium) with:
  - IAM role allowing ECR push and EKS access.
  - Installed tools:
    - Java 17
    - Jenkins
    - Docker
    - Git
    - kubectl
    - eksctl

---

## Setup Instructions

### 1. Install Required Tools on EC2

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### eksctl

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Java 17

```
#Java Installation
sudo dnf install java-17-amazon-corretto -y
```

### Jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### Docker

```
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -aG docker jenkins #Adding Jenkins to Docker group to perform docker action on behalf of us
sudo systemctl restart docker
```

### Git

```
#Git Installation
sudo yum install git -y
```

### 2. Create ECR Repositories (Terraform)

Terraform file: ecr-main.tf creates the following repositories:

- `emailservice`
  - `checkoutservice`
  - `recommendationservice`
  - `frontend`
  - `paymentservice`
  - `productcatalogservice`
  - `cartservice`
  - `loadgenerator`
  - `currencyservice`
  - `shippingservice`
  - `adservice`

Run Terraform:
```terraform init```
```terraform apply```

---

### 3. Build and Push Docker Images via Jenkins

- Log in to Jenkins at http://<EC2_PUBLIC_IP>:8080.
- Create a pipeline for each microservice (e.g., checkoutservice) using its Dockerfile at src/checkoutservice/Dockerfile.
- Configure AWS credentials in Jenkins (do not hardcode in pipeline).
- Pipeline builds the Docker image and pushes it to the respective ECR repository.

---

### 4. Create EKS Cluster

Attach required IAM permissions to the EC2 instance and create the cluster:

```
eksctl create cluster --name <cluster-name> --region us-east-1 --node-type t2.medium --nodes 4
```

---

### 5. Deploy Microservices to EKS

Clone this repo to EC2:

```
git clone <repo-url>
cd <repo>
```

Apply Kubernetes manifests:

```
# Apply Manifest File
kubectl apply -f .
```

Verify services:

```
kubectl get pods
kubectl get svc
```

The LoadBalancer service exposes the frontend application.
Security group (Node Group) allows HTTP (port 80) access.

---

### 6. Access the Application

```
# Load balancer endpoint
kubectl get svc frontend
```

- Open in browser: http://<LOAD_BALANCER_DNS>

---

### Notes

- All microservice images are stored in AWS ECR.
- Jenkins uses credentials securely; images are not hardcoded.
- EKS cluster runs in us-east-1 region with 4 t2.medium nodes.
- HTTP access configured for frontend via node group security group.

---

### References

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Amazon ECR](https://aws.amazon.com/ecr/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)







