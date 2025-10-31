# SD5818 AWS Infrastructure

This repository contains Terraform configurations for provisioning AWS infrastructure including VPC, EKS cluster, ECR repositories, and a Jenkins CI/CD server.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Resources Created](#resources-created)
- [Modules](#modules)
- [Cleanup](#cleanup)

## 🎯 Overview

This Terraform project provisions a complete AWS infrastructure for containerized applications with the following key components:

- **VPC** with public and private subnets across 3 availability zones
- **EKS Cluster** (Amazon Elastic Kubernetes Service) with managed node groups
- **ECR Repositories** for frontend and backend Docker images
- **Jenkins Server** for CI/CD automation on EC2
- **NAT Gateways** for private subnet internet access
- **Security Groups** with appropriate ingress/egress rules

## 🏗️ Architecture

```
├── VPC (10.0.0.0/16)
│   ├── 3 Public Subnets (for NAT, Jenkins, Load Balancers)
│   └── 3 Private Subnets (for EKS worker nodes)
├── EKS Cluster
│   ├── Control Plane (managed by AWS)
│   └── Managed Node Groups (in private subnets)
├── ECR Repositories
│   ├── bndz/frontend
│   └── bndz/backend
└── Jenkins EC2 Instance (in public subnet)
    ├── Jenkins (port 8080)
    ├── Docker
    ├── kubectl
    └── AWS CLI
```

## ✅ Prerequisites

Before you begin, ensure you have the following installed:

- **Terraform** >= 1.0 ([Download](https://www.terraform.io/downloads.html))
- **AWS CLI** >= 2.x ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html))
- **AWS Account** with appropriate permissions
- **SSH Key Pair** for EC2 access (create in AWS Console or CLI)

### Required AWS Permissions

Your IAM user/role should have permissions to create:
- VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways
- EKS Clusters, Node Groups
- EC2 Instances, Security Groups
- ECR Repositories
- IAM Roles and Policies

## 📁 Project Structure

```
SD5818_aws_infrastructure/
├── README.md
└── Provision/
    ├── main.tf                 # Root module configuration
    ├── variables.tf            # Root module variables
    ├── outputs.tf              # Root module outputs
    ├── sample.tfvars           # Sample variable values
    ├── cluster/
    │   ├── main.tf            # Cluster orchestration
    │   ├── variables.tf
    │   └── outputs.tf
    └── modules/
        ├── eks/               # EKS cluster module
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        └── vpc_and_subnets/   # VPC & networking module
            ├── main.tf
            ├── variables.tf
            └── outputs.tf
```

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/hung-nashtech/SD5818_aws_infrastructure.git
cd SD5818_aws_infrastructure/Provision
```

### 2. Configure AWS Credentials

```bash
# Configure AWS CLI
aws configure

# Or export credentials
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="ap-southeast-1"
```

### 3. Create SSH Key Pair (if not exists)

```bash
# Create key pair in AWS
aws ec2 create-key-pair --key-name my-jenkins-key --query 'KeyMaterial' --output text > my-jenkins-key.pem

# Set permissions (Linux/Mac)
chmod 400 my-jenkins-key.pem
```

## ⚙️ Configuration

### Create Your Variables File

Copy the sample variables file and customize it:

```bash
cp sample.tfvars terraform.tfvars
```

Edit `terraform.tfvars`:

```hcl
# AWS Region
region = "ap-southeast-1"  # Change to your preferred region

# EKS Cluster Configuration
cluster_name = "my-eks-cluster"  # Change to your cluster name

# VPC Configuration (optional - uses defaults if not specified)
vpc_name = "eks-vpc"
vpc_cidr = "10.0.0.0/16"

# Kubernetes Version
k8s_version = "1.32"

# EC2 Key Name for Jenkins Server
ec2_key_name = "my-jenkins-key"  # Use your key pair name
```

### Available Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `region` | AWS region for resources | - | Yes |
| `cluster_name` | EKS cluster name | `devops-eks` | No |
| `vpc_name` | VPC name | `eks-vpc` | No |
| `vpc_cidr` | VPC CIDR block | `10.0.0.0/16` | No |
| `k8s_version` | Kubernetes version | `1.32` | No |
| `ec2_key_name` | SSH key pair name for Jenkins | - | Yes |

## 🚀 Deployment

### Initialize Terraform

```bash
terraform init
```

### Plan the Infrastructure

```bash
terraform plan -var-file="terraform.tfvars"
```

### Apply the Configuration

```bash
terraform apply -var-file="terraform.tfvars"
```

Type `yes` when prompted to create the resources.

### Deployment Time

The complete infrastructure typically takes **15-20 minutes** to provision.

## 📦 Resources Created

### Networking
- 1 VPC with DNS hostnames and support enabled
- 3 Public Subnets (for NAT, Load Balancers, Jenkins)
- 3 Private Subnets (for EKS nodes)
- 1 Internet Gateway
- NAT Gateways (1 per AZ by default)
- Route Tables and associations

### EKS Cluster
- EKS Control Plane (Kubernetes version 1.32)
- Managed Node Groups in private subnets
- EKS Addons: CoreDNS, kube-proxy, vpc-cni
- IRSA (IAM Roles for Service Accounts) enabled
- Public and private endpoint access

### Container Registry
- ECR Repository: `bndz/frontend`
- ECR Repository: `bndz/backend`
- Image scanning on push enabled

### CI/CD Infrastructure
- EC2 Instance (t3.medium) running Amazon Linux 2
- Pre-installed software:
  - Jenkins (on port 8080)
  - Docker
  - kubectl (v1.29)
  - AWS CLI v2
  - Git
  - Java 17 (Amazon Corretto)
- Security Group with ports 22, 80, 8080 open

## 🔧 Modules

### VPC and Subnets Module

**Location**: `modules/vpc_and_subnets/`

Creates VPC infrastructure using the official AWS VPC module.

**Key Features**:
- Automatic subnet calculation (CIDR splitting)
- Multi-AZ deployment
- Configurable NAT Gateway strategy

### EKS Module

**Location**: `modules/eks/`

Creates EKS cluster using the official AWS EKS module.

**Key Features**:
- Managed node groups
- Built-in addons
- IRSA support
- Configurable node group settings

## 📊 Outputs

After successful deployment, Terraform outputs important information:

```bash
# View outputs
terraform output
```

Common outputs include:
- VPC ID
- Subnet IDs
- EKS Cluster name and endpoint
- ECR repository URLs
- Jenkins server public IP

## 🔐 Post-Deployment Steps

### 1. Configure kubectl

```bash
# Update kubeconfig
aws eks update-kubeconfig --region ap-southeast-1 --name my-eks-cluster

# Verify connection
kubectl get nodes
```

### 2. Access Jenkins

```bash
# Get Jenkins initial admin password
ssh -i my-jenkins-key.pem ec2-user@<jenkins-public-ip>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Access Jenkins at: `http://<jenkins-public-ip>:8080`

### 3. Configure Jenkins with AWS

Within Jenkins, you may need to:
- Install required plugins (AWS, Kubernetes, Docker)
- Configure AWS credentials
- Add kubectl configuration for EKS

## 🧹 Cleanup

To destroy all resources:

```bash
terraform destroy -var-file="terraform.tfvars"
```

⚠️ **Warning**: This will permanently delete all resources. Ensure you have backups of any important data.

## 🔒 Security Considerations

### Production Recommendations

1. **Security Groups**: Restrict Jenkins ingress to specific IPs, not `0.0.0.0/0`
2. **IAM Roles**: Use assume roles instead of access keys
3. **S3 Backend**: Enable state locking and encryption
4. **Private Endpoints**: Consider making EKS endpoint private-only
5. **Secrets Management**: Use AWS Secrets Manager or Parameter Store
6. **Network Security**: Implement Network ACLs and WAF

### Example: S3 Backend Configuration

Uncomment in `main.tf` and create backend config:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "eks/terraform.tfstate"
    region         = "ap-southeast-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is part of SD5818 training program.

## 📧 Contact

For questions or issues, please contact the repository owner.

---

**Note**: This infrastructure is designed for development/learning purposes. For production use, additional security hardening and high-availability configurations are recommended.